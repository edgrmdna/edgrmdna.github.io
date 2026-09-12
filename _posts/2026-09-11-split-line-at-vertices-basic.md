---
title:  "Splitting Lines or Polygons at the Vertices using a Basic ArcGIS Pro License"
mathjax: true
layout: post 
categories: 
  - github
  - website
---

**PURPOSE**<br>
This script will basically split lines and polygons in ArcGIS Pro in the same way that the [Split By Vertices (Data Management)](https://pro.arcgis.com/en/pro-app/3.6/tool-reference/data-management/split-line-at-vertices.htm) tool will. It is much slower since it is not compiled but it gets the job done.
The ArcGIS Split By Vertices tool requires an advanced license. This version requires only a Basic license. I guess this is good for folks that have the basic license ArcGIS  and maybe sometimes need a tool from the advanced suite(though not often enough to justify paying for the Advanced license). 
QGIS also has a version of this, so that is another free option. 

**Steps**<br>
1. Save the code below somewhere on disk as a .pyt file. Something like SplitLineAtVerticesBasic.pyt
2. In ArcGIS Pro, add it in as a toolbox from the catalog panel: Toolboxes > Add Toolbox, browse to the .pyt
3. Run it. It will appear as Split Line At Vertices (Basic) with the tool nested under it.
4. Output: Point it as a file geodatabase.

```python
import json
import os

import arcpy

_PATH_KEYS = ("curvePaths", "paths")
_RING_KEYS = ("curveRings", "rings")

# Field types that are never copy: managed by the geodatabase or the shape itself.
_SKIP_FIELD_TYPES = {"OID", "Geometry", "GlobalID"}

#Return the coordinate key present in an Esri JSON geometry dict
def _geometry_key(geom_json):
    for key in _PATH_KEYS + _RING_KEYS:
        if key in geom_json:
            return key
    return None


def _segment_endpoint(element):
    if isinstance(element, dict):
        # Single-key dict; grab whichever key it uses.
        params = next(iter(element.values()))
        return params[0]
    return element


def _split_path(path):
    if len(path) < 2:
        return

    previous = path[0]
    for element in path[1:]:
        yield [previous, element]
        previous = _segment_endpoint(element)


def _segment_geometries(geometry, preserve_curves=True):
    geom_json = geometry.JSON
    if not geom_json:
        return []

    parsed = json.loads(geom_json)
    key = _geometry_key(parsed)
    if key is None:
        return []

    # Preserve the source's spatial reference and Z/M awareness on every piece.
    template = {}
    for carried in ("spatialReference", "hasZ", "hasM"):
        if carried in parsed:
            template[carried] = parsed[carried]

    # Output is always a polyline

    segments = []
    for path in parsed[key]:
        for segment in _split_path(path):
            piece = dict(template)

            has_curve = any(isinstance(part, dict) for part in segment)
            if has_curve and not preserve_curves:
                # Collapse the arc to its chord: start vertex to end vertex.
                segment = [_segment_endpoint(part) for part in segment]
                has_curve = False

            piece["curvePaths" if has_curve else "paths"] = [segment]

            try:
                segments.append(arcpy.AsShape(piece, True))
            except Exception:
                # Fall back to a straight chord if the JSON round-trip fails
                # on this particular segment rather than losing the feature.
                chord = dict(template)
                chord["paths"] = [[_segment_endpoint(p) for p in segment]]
                segments.append(arcpy.AsShape(chord, True))

    return segments

# transfer field names present in both, excluding gdb-managed and shape fields.
def _transferable_fields(in_features, out_features):
    out_names = {f.name.upper() for f in arcpy.ListFields(out_features)}

    names = []
    for field in arcpy.ListFields(in_features):
        if field.type in _SKIP_FIELD_TYPES:
            continue
        if field.required:
            # Shape_Length, Shape_Area and friends -- recalculated on insert.
            continue
        if field.name.upper() not in out_names:
            continue
        names.append(field.name)

    return names


def split_line_at_vertices(in_features, out_feature_class,
                           preserve_curves=True, drop_zero_length=True):
    """
    Split polyline or polygon features into one feature per vertex-to-vertex
    segment. Returns the output feature class path.
    """
    describe = arcpy.Describe(in_features)
    if describe.shapeType not in ("Polyline", "Polygon"):
        raise ValueError(
            "Input must be polyline or polygon; got {0}.".format(
                describe.shapeType))

    out_path, out_name = os.path.split(out_feature_class)
    if not out_path:
        raise ValueError("Output must be a full path, not just a name.")

    source = describe.catalogPath if hasattr(describe, "catalogPath") \
        else in_features

    arcpy.management.CreateFeatureclass(
        out_path,
        out_name,
        "POLYLINE",
        template=source,
        has_m="SAME_AS_TEMPLATE" if describe.hasM else "DISABLED",
        has_z="SAME_AS_TEMPLATE" if describe.hasZ else "DISABLED",
        spatial_reference=describe.spatialReference,
    )

    if not arcpy.ListFields(out_feature_class, "ORIG_FID"):
        arcpy.management.AddField(out_feature_class, "ORIG_FID", "LONG")

    carried = _transferable_fields(in_features, out_feature_class)
    oid_field = describe.OIDFieldName

    read_fields = ["SHAPE@", oid_field] + carried
    write_fields = ["SHAPE@", "ORIG_FID"] + carried

    total_in = 0
    total_out = 0
    skipped = 0

    arcpy.SetProgressor("default", "Splitting features at vertices...")

    with arcpy.da.SearchCursor(in_features, read_fields) as search, \
            arcpy.da.InsertCursor(out_feature_class, write_fields) as insert:

        for row in search:
            geometry = row[0]
            total_in += 1

            if geometry is None:
                skipped += 1
                continue

            attributes = list(row[1:])

            segments = _segment_geometries(geometry, preserve_curves)

            # Matches the Esri tool: a feature with no intermediate vertices
            # passes through as a single output feature.
            if not segments:
                insert.insertRow([geometry] + attributes)
                total_out += 1
                continue

            for segment in segments:
                if drop_zero_length and segment.length == 0:
                    skipped += 1
                    continue
                insert.insertRow([segment] + attributes)
                total_out += 1

            if total_in % 1000 == 0:
                arcpy.SetProgressorLabel(
                    "Processed {0:,} features -> {1:,} segments".format(
                        total_in, total_out))

    arcpy.AddMessage(
        "Split {0:,} input features into {1:,} segments.".format(
            total_in, total_out))
    if skipped:
        arcpy.AddWarning(
            "Skipped {0:,} null or zero-length segments.".format(skipped))

    return out_feature_class


class Toolbox(object):
    def __init__(self):
        self.label = "Split Line At Vertices (Basic)"
        self.alias = "splitlinebasic"
        self.tools = [SplitLineAtVertices]


class SplitLineAtVertices(object):
    def __init__(self):
        self.label = "Split Line At Vertices (Basic)"
        self.description = (
            "Splits lines or polygon boundaries into one feature per "
            "vertex-to-vertex segment. Equivalent to the Advanced-license "
            "Split Line At Vertices tool, using only Basic-license "
            "functionality.")
        self.canRunInBackground = False

    def getParameterInfo(self):
        in_features = arcpy.Parameter(
            displayName="Input Features",
            name="in_features",
            datatype="GPFeatureLayer",
            parameterType="Required",
            direction="Input")
        in_features.filter.list = ["Polyline", "Polygon"]

        out_fc = arcpy.Parameter(
            displayName="Output Feature Class",
            name="out_feature_class",
            datatype="DEFeatureClass",
            parameterType="Required",
            direction="Output")

        preserve = arcpy.Parameter(
            displayName="Preserve true curves",
            name="preserve_curves",
            datatype="GPBoolean",
            parameterType="Optional",
            direction="Input")
        preserve.value = True

        drop_zero = arcpy.Parameter(
            displayName="Drop zero-length segments",
            name="drop_zero_length",
            datatype="GPBoolean",
            parameterType="Optional",
            direction="Input")
        drop_zero.value = True

        return [in_features, out_fc, preserve, drop_zero]

    def isLicensed(self):
        return True

    def updateMessages(self, parameters):
        out_fc = parameters[1]
        if out_fc.valueAsText:
            name = os.path.basename(out_fc.valueAsText)
            if name and name[0].isdigit():
                out_fc.setErrorMessage(
                    "Feature class names cannot start with a number.")

    def execute(self, parameters, messages):
        split_line_at_vertices(
            parameters[0].valueAsText,
            parameters[1].valueAsText,
            preserve_curves=parameters[2].value,
            drop_zero_length=parameters[3].value,
        )

```
**Toolbox**<br>
<img src="/assets/split-by-vertices/Toolbox.png" alt="drawing" width='75%'/>
<!-- **![Toolbox](/assets/split-by-vertices/Toolbox.png)** -->

**Tool**<br>
<img src="/assets/split-by-vertices/Tool.png" alt="drawing" width='75%'/>
<!-- **![Tool](/assets/split-by-vertices/Tool.png)** -->

**Lines before split**<br>
<img src="/assets/split-by-vertices/Line_Before.png" alt="drawing" width='75%'/>
<!-- **![Lines Before Split](/assets/split-by-vertices/Line_Before.png)** -->

**Lines after split**<br>
<img src="/assets/split-by-vertices/Line_After.png" alt="drawing" width='75%'/>
<!-- **![Lines After Split](/assets/split-by-vertices/Line_After.png)**-->

**Polygon before split**<br>
<img src="/assets/split-by-vertices/Polygon_Before.png" alt="drawing" width='75%'/>
<!-- **![Polygon Before Split](/assets/split-by-vertices/Polygon_Before.png)** -->

**Polygon after split**<br>
<img src="/assets/split-by-vertices/Polygon_After.png" alt="drawing" width='75%'/>
<!-- **![Polygon After Split](/assets/split-by-vertices/Polygon_After.png)** -->
