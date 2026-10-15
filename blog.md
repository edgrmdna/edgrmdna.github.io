---
layout: "default"
title: "Posts"
---

<style>
  .post-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 24px;
    max-width: 960px;
    margin: 40px auto;
    align-items: start;
  }
  .post-tile img {
    width: 100%;
    aspect-ratio: 1 / 1;
    object-fit: cover;
    display: block;
    border-radius: 4px;
  }
  .post-tile h4 {
    text-align: center;
    font-size: 0.95em;
    margin: 8px 0 0;
  }
  .bw-image {
    filter: grayscale(100%);
  }
  @media (max-width: 800px) { .post-grid { grid-template-columns: repeat(2, 1fr); } }
  @media (max-width: 500px) { .post-grid { grid-template-columns: 1fr; } }
</style>

<div class="post-grid">
  <div class="post-tile">
    <a href="{{ site.baseurl }}{% post_url 2022-01-29-address-scrape %}"><img class="bw-image" src="{{ site.baseurl }}/assets/thumbnails/thumbnail-address-scrape.png" alt="Address Scraping"></a>
    <h4><a href="{{ site.baseurl }}{% post_url 2022-01-29-address-scrape %}">Workflow: Address Scraping</a></h4>
  </div>
  <div class="post-tile">
    <a href="{{ site.baseurl }}{% post_url 2022-01-18-LidarVolume %}"><img src="{{ site.baseurl }}/assets/thumbnails/thumbnail-lidar-volumes.png" alt="Lidar Volume Calculations"></a>
    <h4><a href="{{ site.baseurl }}{% post_url 2022-01-18-LidarVolume %}">Workflow: Lidar Volume Calculations</a></h4>
  </div>
  <div class="post-tile">
    <a href="{{ site.baseurl }}{% post_url 2022-02-10-gee %}"><img src="https://raw.githubusercontent.com/edgrmdna/edgrmdna.github.io/master/assets/cdl.png" alt="Landsat Composites"></a>
    <h4><a href="{{ site.baseurl }}{% post_url 2022-02-10-gee %}">Code Sample: Download Landsat Composites from Google Earth Engine</a></h4>
  </div>
  <div class="post-tile">
    <a href="https://edgrmdna.github.io/leaflet-inaturalist-map/"><img src="{{ site.baseurl }}/assets/thumbnails/thumbnail-la-river.png" alt="iNaturalist LA River map"></a>
    <h4><a href="https://edgrmdna.github.io/leaflet-inaturalist-map/">Webmap Sample: iNaturalist Observations - Los Angeles River</a></h4>
  </div>
  <div class="post-tile">
    <a href="https://raw.githubusercontent.com/edgrmdna/edgrmdna.github.io/refs/heads/master/maps/Lake_Cachuma_Decadal_Fire_History.png"><img src="{{ site.baseurl }}/assets/thumbnails/thumbnail-lake-cachuma-fire.png" alt="Lake Cachuma fire history map"></a>
    <h4><a href="https://raw.githubusercontent.com/edgrmdna/edgrmdna.github.io/refs/heads/master/maps/Lake_Cachuma_Decadal_Fire_History.png">Map Sample: Lake Cachuma Decadal Fire History</a></h4>
  </div>
  <div class="post-tile">
    <a href="https://raw.githubusercontent.com/edgrmdna/edgrmdna.github.io/refs/heads/master/maps/SLO_County_2026_Land_Ownership.png"><img src="{{ site.baseurl }}/assets/thumbnails/thumbnail-zoning.png" alt="SLO County land ownership map"></a>
    <h4><a href="https://raw.githubusercontent.com/edgrmdna/edgrmdna.github.io/refs/heads/master/maps/SLO_County_2026_Land_Ownership.png">Map Sample: San Luis Obispo County Public Land Ownership</a></h4>
  </div>
  <div class="post-tile">
    <a href="{{ site.baseurl }}{% post_url 2026-09-11-split-line-at-vertices-basic %}"><img class="bw-image" src="{{ site.baseurl }}/assets/thumbnails/thumbnail-split-vertices.png" alt="Split By Vertices"></a>
    <h4><a href="{{ site.baseurl }}{% post_url 2026-09-11-split-line-at-vertices-basic %}">Code Sample: Split By Vertices Using a basic ArcGIS Pro license</a></h4>
  </div>
</div>
