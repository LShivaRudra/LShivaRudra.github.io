---
title: Automated 3D Reconstruction of Buildings
summary: Reconstructed Bonn roofs from airborne LiDAR and built a LoD3 façade prototype that uses street imagery and laser returns to recover windows and doors.
tags:
  - 3D Reconstruction
  - LiDAR
  - Deep Learning
# Portfolio publication date; the presentation does not specify project dates.
date: '2026-09-27T00:00:00Z'
external_link: ''
thumbnail_video: lod2_transition.mp4
image:
  filename: video-poster.jpg
  preview_only: true
  focal_point: Center
links: []
url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''
---

<div class="recon-story">

<header class="recon-hero">
  <div class="recon-hero-copy">
    <p class="recon-kicker">Bonn, Germany · Urban LiDAR reconstruction</p>
    <h2>From roofline<br>to <em>doorway.</em></h2>
    <p class="recon-intro">From above, LiDAR can trace a roof. At street level, a laser return behind a wall can hint at a window. We used both views to reconstruct buildings in Bonn, from roof planes to individual façade openings.</p>
    <nav aria-label="Project chapters">
      <a href="#lod2">LoD2 · Roof geometry</a>
      <a href="#lod3">LoD3 · Façade openings</a>
      <a href="#findings">Findings</a>
    </nav>
  </div>

{{< project-video src="lod2_transition.mp4" poster="video-poster.jpg" autoplay="false" label="Point cloud to LoD2 reconstruction along the Bonn mapping corridor" caption="The Bonn corridor as a point cloud and as reconstructed LoD2 buildings." >}}

</header>

<div class="recon-role" role="note" aria-label="My contribution">
  <p><strong>My contribution</strong> I focused on LoD2 and LoD3. I built the Roofer tuning and evaluation workflow and implemented the LoD3 stages and photo-based experiments, building on Poorna Sasank Sesetti’s conflict-map fusion method. The wider team worked on point-cloud preparation, colourisation and building segmentation.</p>
</div>

<section class="recon-chapter" aria-labelledby="lod2">

<p class="recon-kicker">LoD2 · Airborne laser scanning</p>

## Give the skyline its shape {#lod2}

A cadastral footprint defines where a building sits. The airborne points supply the heights and slopes that turn that outline into a roof. I used Roofer to fit those planes and export the models to CityJSON, then used Optuna to tune the plane detector and the amount of geometric detail Roofer would retain.

### A better roof, by which measure?

The early 43-building comparison recovered **nine additional valid models** with tuned settings. The semester-two evaluation used a separate 30-trial tuning study and a much larger roof comparison against NRW LoD2 references, with height interpolation bounded to each reference triangle.

<div class="recon-metrics">
  <div><strong>0.533 → 0.492 m</strong><span>Mean roof-height MAE against NRW · 7.9% lower</span></div>
  <div><strong>0.893 → 0.801 m</strong><span>Mean roof-height RMSE against NRW</span></div>
</div>

Across **13,489 buildings valid in both runs**, 4,525 improved by more than 1 cm, 1,958 worsened by more than 1 cm, and 7,006 changed by at most 1 cm. The tuned run produced **48 fewer valid roof evaluations** overall.

<div class="recon-case">

{{< project-figure src="roof-case-008025.jpg" alt="Rendered model of building 008025 with a distinct raised feature on the roof" caption="Building 008025 from the semester-two presentation." >}}

<div>
  <h3>When the reference simplifies a roof</h3>
  <p>A roof feature visible in the airborne points and Roofer models is simplified in the NRW model. For this building, the default model still fit the airborne points best: <strong>0.376 m MAE</strong>, versus 0.400 m tuned and 0.502 m for NRW. Visual topology and point-wise error capture different aspects of the fit.</p>
</div>
</div>

<details class="recon-details">
<summary>Earlier LoD2 comparison · 43 attempted buildings</summary>

| Criterion | Default Roofer | Optuna · 100 trials |
| --- | ---: | ---: |
| Valid reconstruction | 30/43 (70%) | 39/43 (91%) |
| Volumetric IoU ≥ 0.90 | 25/43 (58%) | 33/43 (77%) |
| Overall RMSE ≤ 0.50 m | 21/43 (49%) | 28/43 (65%) |
| Roof RMSE ≤ 0.50 m | 16/43 (37%) | 25/43 (58%) |

The denominator includes failed reconstructions. IoU measures volume overlap with the reference, and RMSE measures geometric error.

{{< project-figure src="reconstruction-comparison.jpg" alt="Two examples with airborne input, default reconstruction, tuned reconstruction and NRW reference" caption="Two building-level comparisons from the early LoD2 study." >}}

</details>

</section>

<section class="recon-chapter" aria-labelledby="lod3">

<p class="recon-kicker">LoD3 · Mobile LiDAR and imagery</p>

## Find what the wall is hiding {#lod3}

LoD2 describes the roof and the building envelope. To recover windows and doors, I used a separate LoD3 workflow that starts from **official NRW LoD2 walls** and aligns them to a mobile mapping drive. When a laser ray hits the wall, it supports the wall being solid at that spot. A return behind the wall can indicate an opening. Accumulating those observations on a 10 cm grid gives a conflict map.

Trees, parked cars and sparse upper-floor returns leave gaps in that evidence. The same vehicle also captured fisheye photographs. I used its calibrated camera poses to build straightened façade views, detect openings with SAM 3, and merge detections across views. The photographs locate openings where they have coverage. The conflict map fills gaps, and laser returns estimate recess depth where available.

{{< project-figure src="facade-opening-detections.jpg" alt="Rectified façade view beside the same view with blue window and green door detections" caption="A rectified mobile-mapping image before and after opening detection." >}}

### What the photos recovered

<div class="recon-table">

| Opening detection | Laser only | Photo-assisted LoD3 |
| --- | ---: | ---: |
| Recall | 47.5% | **84.0%** |
| Precision | 56.8% | **84.5%** |
| Instance F1 | 0.517 | **0.842** |
| Doors matched | 0/16 | **9/16** |

</div>

The pilot covers **8 façades and 455 hand-reviewed openings**, with a match defined at box IoU ≥ 0.50. The labels started from SAM drafts, so the photo-assisted scores may be optimistic. Independent labels and unseen drives would give a better estimate of how well the method generalises.

{{< project-figure src="facade-laser-photo-comparison.jpg" alt="A single façade with laser-only opening matches above and photo-assisted matches below" caption="A single pilot façade, with 13 matched openings from laser evidence alone and 75 after adding photos." >}}

### From detections to geometry

The pipeline cuts openings and recesses into the wall geometry and exports CityJSON and CityGML. In the experimental photo branch, I also tested adding LiDAR-supported balconies as separate installations and projecting imagery onto the reconstructed façade.

{{< project-figure src="facade-geometry-and-texture.jpg" alt="Two renders of the same Bonn building showing semantic openings and balconies at left and the textured façade at right" caption="A Bonn façade rendered with semantic geometry and with projected imagery. Windows, doors and balconies are visible in the geometry view." >}}

</section>

<section class="recon-chapter" aria-labelledby="findings">

## What the two views leave unresolved {#findings}

Roof tuning lowered average error against the city model, while some individual buildings fit the airborne points better with default settings. The façade pilot recovered many more openings with imagery, while occlusion and the lack of independent labels still limit what we can conclude. The next validation step is to test on unseen roofs and independently annotated façades, and measure how LoD2 wall alignment errors carry through to LoD3.

<details class="recon-details">
<summary>Team workflow · building instances</summary>

Upstream of my work, the team aligned and colourised airborne and mobile point clouds, isolated building points, and explored how to separate neighbouring buildings. FaçadeSAM-Track used image segmentation and tracking to assign building instances to the point cloud. The images below show a qualitative example.

{{< project-comparison before="segmentation-street-input.jpg" after="segmentation-street-instances.jpg" caption="Colourised LiDAR and predicted building instances along a Bonn street." >}}

</details>

<p class="recon-credits"><strong>Team</strong> · Shiva Rudra Lolla, Ravikumar Chaurasia, Poorna Sasank Sesetti, Pranesh Gopal and Carine Allen.</p>

</section>

</div>
