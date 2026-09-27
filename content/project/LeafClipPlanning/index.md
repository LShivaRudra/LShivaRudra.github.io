---
title: Leaf-Clip Pose Planning for Robotic Plant Sensing
summary: Developed a geometry-based planner to find and rank leaf-clip placements with supported contact and clear insertion paths. Results include pose-quality heatmaps, clip-placement visualizations, and cuRobo motion demonstrations on a robot model with payload and B2 in Isaac Sim.
tags:
  - Motion Planning
  - Robotic Manipulation
  - Plant Sensing
# Portfolio publication date.
date: '2026-09-27T01:00:00Z'
external_link: ''
thumbnail_video: leaf-clip-poses.webm
image:
  filename: leaf-clip-poses-poster.jpg
  preview_only: true
  focal_point: Center
links: []
url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''
---

{{< project-video src="leaf-clip-poses.webm" poster="leaf-clip-poses-poster.jpg" label="Candidate leaf-clip placements" width="2485" height="1322" caption="Candidate clip placements visualized on a leaf mesh. Multiple alternatives are displayed together to inspect contact locations and clip orientation." >}}

## Overview

Placing a measuring clip on a leaf requires more than reaching a point in space. The measuring aperture needs to sit on the leaf blade, the contacts need support, and the clip needs a clear path through surrounding foliage.

My work combines **geometry-based leaf-clip pose planning** with **cuRobo motion-planning demonstrations**. The planner searches a leaf mesh, rejects mechanically unsuitable placements, and ranks the survivors. Heatmaps make it possible to inspect where good placements occur and why other regions are rejected. Alongside this, I implemented end-effector tracking on a robot model with payload and cuRobo integration on B2 in Isaac Sim.

This project is funded by [PhenoRob](https://www.phenorob.de/).

<div class="project-facts" aria-label="Planner configuration">
  <div><strong>38</strong><span>orientations per candidate spot</span></div>
  <div><strong>6</strong><span>mandatory geometric checks</span></div>
  <div><strong>24 mm</strong><span>minimum spacing between selected sites</span></div>
</div>

These settings define the current search. Its outputs are ranked candidate poses and diagnostic heatmaps; the cuRobo recordings show the accompanying robot motion work.

## Finding valid clip poses

### 1. Filter candidate sites

I first screen every vertex on the leaf mesh, before generating any clip poses. A candidate must satisfy all four conditions:

| Filter | Requirement |
| --- | --- |
| Surface edge margin | At least **7 mm** from the rim, measured along the leaf surface |
| Organ mask | Petiole and shoot regions excluded |
| Local flatness | Plane-fit RMS deviation below **0.3 mm** over a **10 mm** window |
| Neighbor clearance | At least **5 mm** from neighboring organs |

This removes unsuitable regions early. The later contact-support check applies a stricter **12 mm** rim clearance to account for the modeled upper contact ring.

### 2. Sample both leaf faces

At each surviving site, I align the jaw-closing axis with the local surface normal and place the approach direction in the tangent plane. Sweeping yaw from **−90° to +90° in 10° steps** gives 19 orientations per face, or **38 poses across both faces**.

Each pose stores a contact configuration and a pre-insertion configuration **30 mm** back along the approach direction.

### 3. Enforce mechanical constraints

A pose survives only if it passes all six checks:

| Check | What the planner requires |
| --- | --- |
| Measuring aperture | The full **10 mm diameter** measuring window lies on the lamina, avoiding background in the aperture. |
| Contact support | At least **98%** support beneath the modeled upper and lower contacts, with **12 mm and 9 mm radii**. The center must be at least **12 mm** from the rim. |
| Attachment exclusion | No stem, petiole, or midrib inside the contact footprint. |
| Jaw alignment | Closing axis within **15°** of the local surface normal. |
| Insertion clearance | Approach line remains at least **5 mm** from other organs. |
| Swept-body collision | Three solid boxes approximate the clip body and extend **30 mm** along the approach sweep. Collision checks use zero margin against the target leaf, preserving the jaw gap, and **5 mm** against other organs. |

### 4. Rank and spread out the results

Surviving poses receive a score, with lower values preferred. **Crowding** penalizes clearance near the 5 mm limit, while **uncertainty** penalizes sites near the curvature and edge limits. The trajectory-length term is currently the same **30 mm standoff** for every pose, so it does not affect their order.

The batch driver selects the best few poses with at least **24 mm between sites**, producing spatially distinct alternatives.

Four additional score terms—deformation energy, patch rotation, patch translation, and petiole moment—are defined but currently return zero. They depend on a deformable-leaf physics rollout that is not connected yet. **Current ranking is geometry only.**

## Results and visualizations

### Pose-quality heatmaps

Each mesh vertex accumulates information from the pose search. I can visualize rejection reasons, the best score at each site, clearance, curvature, edge margin, and vote density weighted toward better poses.

{{< project-video src="grasp-heatmaps.webm" poster="grasp-heatmaps-poster.jpg" label="Leaf-clip grasp-pose heatmaps" width="2486" height="1316" autoplay="false" caption="The recording shows score-weighted candidate regions and a diagnostic rejection map for a leaf with no feasible pose." >}}

Together with the clip-placement view at the top, these visualizations help inspect both the search results and the physical placement implied by a candidate pose.

### cuRobo end-effector tracking with a payload model

I also implemented cuRobo end-effector tracking on the robot model with its payload represented. This recording shows the robot and target-pose visualization used to inspect the tracking behavior.

{{< project-video src="curobo-payload-tracking.webm" poster="curobo-payload-tracking-poster.jpg" label="cuRobo end-effector tracking on a robot model with payload" width="2493" height="1326" autoplay="false" caption="Model-based end-effector tracking with the payload represented in the robot scene." >}}

### B2 integration in Isaac Sim

The simulation recording below shows cuRobo running with the B2 robot model in Isaac Sim. It provides a supporting view of the robot integration; the leaf-level geometry is easier to inspect in the dedicated planner visualizations above.

{{< project-video src="b2-isaac-sim.mp4" poster="b2-isaac-sim-poster.jpg" label="cuRobo integration on B2 in Isaac Sim" width="1280" height="672" autoplay="false" caption="B2 and manipulator simulation in Isaac Sim. The recording is cropped to the simulation viewport to keep the robot visible." >}}

## Hardware dimensions and modeling assumptions

The model uses the MINI-PAM-II's documented **10 mm measuring aperture** and **170 × 57 × 80 mm overall envelope**. Several other dimensions are planner assumptions and need validation against the physical clip:

| Assumption | Effect on the planner |
| --- | --- |
| Contact radii of 12 mm and 9 mm | These are modeled values. The upper contact requires a leaf blade at least **24 mm wide**, rejecting narrower leaves. |
| Fixed 20 mm jaw opening | The real support adjusts vertically for different leaf thicknesses; the model uses a fixed gap. |
| 7 mm initial edge margin | A chosen planner parameter. It overrides a component-based margin calculation that sums to **13 mm**. |
| Three solid collision boxes | A hand-built approximation of the clip body, rather than manufacturer CAD. |

## Next steps

I’m extending the workflow toward automated leaf measurements in agricultural environments, with two areas of focus:

<div class="project-roadmap">
  <section aria-labelledby="leaf-segmentation-roadmap">
    <span class="roadmap-label">Perception</span>
    <h3 id="leaf-segmentation-roadmap">Leaf instance segmentation</h3>
    <p>I’m developing a segmentation pipeline to identify individual leaves and provide targets for automated measurement.</p>
  </section>
  <section aria-labelledby="hardware-roadmap">
    <span class="roadmap-label">Hardware</span>
    <h3 id="hardware-roadmap">B2 with a Z1 arm</h3>
    <p>I plan to implement the workflow on a physical B2 robot with a Z1 arm in an agricultural setting, combining partial autonomy with a human in the loop.</p>
  </section>
</div>
