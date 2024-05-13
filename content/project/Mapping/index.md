---
title: Implementing ESDF Mapping with VINS
summary: The ESDF map is made using pointcloud data from the depth camera and rectified odometry from the VINS framework. Voxfield framework is utilized for this purpose
tags:
  - C
  - Python
  - ROS
date: '2023-11-10T00:00:00Z'

# Optional external URL for project (replaces project detail page).
external_link: ''

image:
  caption: 
  focal_point: Smart

links:
  - icon: github
    icon_pack: fab
    name: GitHub Link
    url: https://github.com/LShivaRudra/VINS-Mono
url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
# slides: example
---
## Summary

In this work, pointcloud data from the Realsense D455 camera and odometry from VINS estimator module are fed as inputs to the Voxfield mapping framework. Occupancy and ESDF maps are shown in visualization.

## Work Pipeline

![Pipeline](/images/Mapping/i.png)

## Results

### Output of VINS estimator:

![VINS](/images/Mapping/e.png)

### ESDF map for a small region:

![ESDF](/images/Mapping/f.png)

### Occupancy map of the lab while implementing VINS:

![map1](/images/Mapping/g.png)

![map2](/images/Mapping/h.png)

### Video Results:

<iframe width="560" height="315" src="https://www.youtube.com/embed/nKO_ds5VaFg?si=JZnoUhTXfWRBph9L" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/oRt61GfpLrM?si=BymV5UJi6bKZbzej" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>