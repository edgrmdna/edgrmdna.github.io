---
title:  "LiDAR Volume Calculations"
mathjax: true
layout: post 
categories: 
  - github
  - website
---

[Cerro San Luis](/assets/cerro-san-luis.png)

Step 1: Cleanse your lidar

You gotta get rid of the trees and buildings before you do any sort of volumetric calc. So we want only the bare-ground points out of your lidar dataset.

I cleaned my lidar with pdal. I used its built-in SMRF (Simple Morphological Filter) via the osgeo4w command line interface. It looked something like this:
{% highlight bash %}
pdal translate in.las out.las smrf -v 4
{% endhighlight %}

This outputs a .las file that contains the same points as the input but now with a classification value for each point:

1 - Not ground
2 - Ground

Next step is to output a file with just the ground points. We can use the translate command again but this time flag (-f) a range filter of the 'classification' type:

{% highlight bash %}
pdal translate in.las out.las -f range --filters.range.limits="Classification[2:2]"
{% endhighlight %}
[Tree Removal](/assets/tree-classification.gif)


__________
Step 2: Clip your data


I arbitrarily clipped Cerro San Luis Obispo Mountain to elevations at or above 145m, using the contours as a guideline. I ended up converting the point cloud into a raster with PDAL's pipeline function. (The raster helps extract contours). Here's the pipeline I used:

{% highlight c %}
[
    "cerro_san_luis_ground_only.laz",
    {
        "type":"writers.gdal",
        "filename":"cerro_sluis_ground_only.tif",
        "output_type":"min",
        "gdaldriver":"GTiff",
        "window_size":3,
        "resolution":0.5
    }
]
{% endhighlight %}


Then run the pipeline via command line using PDAL2:


pdal pipeline -i dtm.json




__________
Step 4: Calculate the Volume

At this point, CloudCompare has a function called 'Compute 2.5 D Volume'. Specify your base height and a report will display the calculations. I went ahead and made a map to show my results:
