---
title: Vision-Guided Brick Sorting with a Franka Arm
summary: Led perception for Team KalmanKrew at the robo.innovate Hackathon 2026 (Track 08, Marmon). I designed an RGB-D pipeline that finds the reachable top layer of a brick stack with SAM 3 and depth geometry, classifies each brick by colour, and hands 3D pick targets to a Franka Research 3 for sorting.
tags:
  - Computer Vision
  - Robotic Manipulation
  - Deep Learning
  - Python
  - ROS
date: '2026-10-28T00:00:00Z'
external_link: ''
image:
  filename: featured-team-at-robot.jpg
  caption: 'Explaining the perception pipeline at the robot cell.'
  preview_only: true
  focal_point: Center
links: []
url_code: 'https://github.com/LShivaRudra/vision-guided-brick-palletizer'
url_pdf: ''
url_slides: ''
url_video: 'https://www.youtube.com/watch?v=UwtO5sTqreg'
---

<div class="bp-story">

<section class="bp-hero" aria-label="Project introduction">
  <div class="bp-hero-copy">
    <span class="bp-eyebrow">robo.innovate Hackathon 2026 · Munich · Track 08 by Marmon</span>
    <p class="bp-lede">One brick at<br> <em>a time.</em></p>
    <p class="bp-deck">Marmon’s example kiln car held 28,800 bricks. In six days, our team built a tabletop prototype that could find a reachable brick, judge its colour, and tell a Franka arm where to pick it.</p>
    <dl class="bp-meta">
      <div><dt>My role</dt><dd>Perception lead · grasp testing · final pitch</dd></div>
      <div><dt>Team</dt><dd>KalmanKrew, 5 people</dd></div>
      <div><dt>Hardware</dt><dd>Franka Research 3 + RealSense D435</dd></div>
      <div><dt>Timeline</dt><dd>6 days, 23–28 Oct 2026</dd></div>
    </dl>
    <a class="bp-hero-link" href="#one-frame">Follow one camera frame <span aria-hidden="true">↓</span></a>
  </div>
  <figure class="bp-hero-video">
    {{< bp-video src="robot-demo.mp4" poster="robot-poster.jpg" label="Franka arm picking and sorting LEGO bricks" >}}
    <figcaption>Live on the Franka Research 3</figcaption>
  </figure>
</section>

<nav class="bp-chapter-nav" aria-label="Case study chapters">
  <a href="#the-challenge">The challenge</a>
  <a href="#one-frame">The perception pipeline</a>
  <a href="#on-the-robot">The robot demo</a>
  <a href="#the-final-pitch">The final pitch</a>
</nav>

<section class="bp-chapter" aria-labelledby="the-challenge">

## From kiln car to lab table {#the-challenge}

After firing, every brick on a kiln car has to be inspected, sorted by shade, and restacked onto pallets. Marmon challenged us to automate this unloading step, known as dehacking.

On our lab table, a woven stack of LEGO bricks stood in for a kiln car. The first decision was the same at either scale. **Which brick can the robot pick without disturbing the stack?**

<div class="bp-map" role="table" aria-label="How the factory problem maps to our tabletop demo">
  <div class="bp-map-head" role="row"><span role="columnheader">In the brickyard</span><span aria-hidden="true"></span><span role="columnheader">On our table</span></div>
  <div class="bp-map-row" role="row"><span role="cell">A kiln car of fired bricks</span><i aria-hidden="true">→</i><span role="cell">A woven LEGO stack on a baseplate</span></div>
  <div class="bp-map-row" role="row"><span role="cell">Unload layer by layer</span><i aria-hidden="true">→</i><span role="cell">Pick <b>only the exposed top layer</b></span></div>
  <div class="bp-map-row" role="row"><span role="cell">Inspect and grade by shade</span><i aria-hidden="true">→</i><span role="cell">Classify <b>5 LEGO colours</b></span></div>
  <div class="bp-map-row" role="row"><span role="cell">Palletize and scrap rejects</span><i aria-hidden="true">→</i><span role="cell">One pile per good colour, plus a scrap bin</span></div>
</div>

<figure class="bp-figure">
  {{< bp-img src="real-brick-sample.jpg" alt="A real brick sample labelled Bad, with LEGO bricks in front of it" w="1200" >}}
  <figcaption>A real "bad" sample from the sponsor next to the LEGO bricks that stood in for it.</figcaption>
</figure>

### My role on KalmanKrew

<div class="bp-role">
  <figure>
    {{< bp-img src="featured-team-at-robot.jpg" alt="Shiva explaining the perception pipeline to teammates at the Franka robot cell" w="900" >}}
  </figure>
  <div>
    <p class="bp-role-lead">I led perception for the LEGO demo, taking the pipeline from raw camera frames to 3D targets for the motion planner.</p>
    <ul class="bp-checks">
      <li><b>Perception:</b> top-layer detection, size-based mask cleanup, colour classification and 3D pose export.</li>
      <li><b>Tooling:</b> RGB-D capture, live detection and a modular runner for replaying recorded frames.</li>
      <li><b>On the robot:</b> tuned grasps with the manipulation team and tested the full sorting loop.</li>
    </ul>
  </div>
</div>

</section>

<section class="bp-chapter" aria-labelledby="one-frame">

## From 61 masks to 22 pick targets {#one-frame}

The camera sees the top layer, parts of lower layers, and loose bricks around the stack. Select a stage below to see how one RealSense frame becomes a set of 3D pick targets. You can also use the arrow keys after focusing a stage.

{{< brick-stepper >}}

<p class="bp-note">One real capture from the RealSense D435, top-down over the LEGO stack. Stage images from the team's annotated pipeline figure.</p>

### Which bricks are reachable?

SAM 3 is very good at finding brick-shaped regions, but it has no notion of physical size. Two touching bricks can come back as one mask. A brick half-hidden under the layer above still looks like a brick, and so does the side face of an edge brick.

To fix this, I project each top-layer mask onto the table plane and measure its **footprint in millimetres**. I then compare that against a known brick size: L × W × H ≈ 30 × 15 × 10 mm. The prior then makes one of four calls:

<div class="bp-prior">
  <div class="bp-prior-card is-keep">
    <div class="bp-prior-viz"><span class="bp-brick" style="--l:4;--w:2;--c:#c9332b"></span></div>
    <h4>≈ L × W</h4><p>One clean brick</p><span class="bp-tag">keep</span>
  </div>
  <div class="bp-prior-card is-split">
    <div class="bp-prior-viz"><span class="bp-brick is-split" style="--l:8;--w:2;--c:#d27a24"></span></div>
    <h4>W × n·L</h4><p>Bricks SAM merged</p><span class="bp-tag">split into n</span>
  </div>
  <div class="bp-prior-card is-reject">
    <div class="bp-prior-viz"><span class="bp-brick is-partial" style="--l:4;--w:2;--c:#7a1f26"></span></div>
    <h4>W × &lt; L</h4><p>Mostly hidden by the layer above</p><span class="bp-tag">reject</span>
  </div>
  <div class="bp-prior-card is-reject">
    <div class="bp-prior-viz"><span class="bp-brick is-sliver" style="--l:4;--w:2;--c:#6b3a22"></span></div>
    <h4>&lt; W wide</h4><p>Side face of an edge brick</p><span class="bp-tag">reject</span>
  </div>
</div>

Every brick that survives gets a fitted L × W footprint. That gives its **centre in the camera frame, its yaw on the table and its layer index**, which is exactly what a grasp planner needs.

### Which pile takes each brick?

The lab had bright desk lamps, glare on glossy plastic and shifting exposure. Matching raw RGB values would have mislabelled bricks constantly. The classifier corrects for that in three steps:

<div class="bp-colours">
  <ol class="bp-colour-steps">
    <li><b>White-balance on the table.</b> The plane fit already says which pixels are table, so the colour model uses them as a grey reference in linear RGB.</li>
    <li><b>Measure the clean interior.</b> Colour is sampled inside the fitted footprint, away from the edges. The darkest 15 % (shadows) and brightest 10 % (glare) are dropped.</li>
    <li><b>Match in CIELAB with a shared offset.</b> One bounded Lab offset is fitted jointly across all bricks in the frame to absorb exposure drift. Then each brick takes its nearest reference colour, with a confidence from distance and margin.</li>
  </ol>
  <div class="bp-palette" aria-label="Colour classes and destinations in the example capture">
    <div class="bp-palette-row"><i style="--c:#c91a09"></i><span>Red</span><em>×7</em><b class="is-good">Pile A</b></div>
    <div class="bp-palette-row"><i style="--c:#720e0f"></i><span>Dark red</span><em>×5</em><b class="is-good">Pile B</b></div>
    <div class="bp-palette-row"><i style="--c:#a95500"></i><span>Dark orange</span><em>×6</em><b class="is-good">Pile C</b></div>
    <div class="bp-palette-row"><i style="--c:#582a12"></i><span>Reddish brown</span><em>×3</em><b class="is-bad">Scrap</b></div>
    <div class="bp-palette-row"><i style="--c:#cc8e69"></i><span>Medium nougat</span><em>×1</em><b class="is-bad">Scrap</b></div>
    <p>Counts from the capture above.</p>
  </div>
</div>

### Where should the gripper go?

Perception ends at a clean, documented contract. For every brick, the pipeline writes a 3D box in the **camera optical frame** (x right, y down, z forward, metres). The robotics side applies the hand-eye transform and plans the motion.

<div class="bp-handoff">
  <svg viewBox="0 0 300 190" role="img" aria-label="A brick drawn as a 3D box with numbered corners, centre point, yaw arrow and camera axes">
    <g fill="none" stroke="currentColor" stroke-width="1.6" stroke-linejoin="round">
      <path d="M60 70 L160 40 L250 70 L150 100 Z" class="bp-svg-top"/>
      <path d="M60 70 L60 110 L150 140 L150 100 M150 140 L250 110 L250 70"/>
      <path d="M60 110 L160 80 L250 110 M160 80 L160 40" stroke-dasharray="4 4" opacity=".45"/>
    </g>
    <path d="M159 94 L200 158" stroke="currentColor" stroke-width="1" opacity=".5" fill="none"/>
    <circle cx="155" cy="90" r="5" class="bp-svg-dot"/>
    <path d="M155 70 L205 55" class="bp-svg-yaw" stroke-width="2.4" fill="none" marker-end="url(#bp-arrow)"/>
    <defs><marker id="bp-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M0 0 L10 5 L0 10 z" class="bp-svg-yaw-fill"/></marker></defs>
    <g font-size="11" font-family="inherit" fill="currentColor">
      <text x="44" y="68">0</text><text x="160" y="34">1</text><text x="256" y="68">2</text><text x="146" y="114">3</text>
      <text x="44" y="118">4</text><text x="256" y="118">6</text><text x="146" y="154">7</text>
      <text x="198" y="168" class="bp-svg-label">box centre</text><text x="208" y="52" class="bp-svg-label">yaw</text>
    </g>
    <g stroke="currentColor" stroke-width="1.6" fill="currentColor" font-size="11" opacity=".75">
      <path d="M18 172 L58 172" marker-end="url(#bp-arrow)"/><text x="62" y="176" stroke="none">x</text>
      <path d="M18 172 L18 186" marker-end="url(#bp-arrow)"/><text x="24" y="188" stroke="none">y</text>
      <circle cx="18" cy="172" r="3.5" fill="none"/><circle cx="18" cy="172" r="1" stroke="none"/><text x="6" y="164" stroke="none">z</text>
    </g>
  </svg>
  <pre class="bp-schema"><code><span># per brick → bricks.json, poses.csv</span>
id, colour, quality, confidence
layer           <span># index in the stack</span>
centre_xyz      <span># box centre [m]</span>
top_centre_xyz  <span># top face   [m]</span>
yaw             <span># on the table</span>
size_lwh        <span># fitted size</span>
corners[8]      <span># top 0–3, bottom 4–7</span></code></pre>
</div>

I also wrapped the stages in a **modular pipeline runner**. It can capture a fresh frame or replay a recording, run segmentation, classification and pose export as separate processes, and give every run its own timestamped folder. If any stage fails, it stops and writes an explicit `INCOMPLETE.json`, so stale results are never silently reused.

</section>

<section class="bp-chapter" aria-labelledby="on-the-robot">

## Pick, place, look again {#on-the-robot}

Even a successful pick can nudge the next brick out of position. The Franka picks a few bricks, returns to its observation pose, and **rescans** before committing to the next set of targets.

<div class="bp-loop" aria-label="Pick and rescan loop">
  <span>Observation pose</span><i aria-hidden="true">→</i>
  <span>Perceive<small>pipeline above</small></span><i aria-hidden="true">→</i>
  <span>Pick ≤ N bricks<small>top layer only</small></span><i aria-hidden="true">→</i>
  <span>Place<small>colour pile or scrap</small></span>
  <em aria-hidden="true">↺ rescan · also the recovery path after a motion failure</em>
</div>

<div class="bp-split">
  <figure class="bp-figure bp-tall">
    {{< bp-img src="sorting-setup.jpg" alt="The Franka arm above the LEGO stack, with sorted bricks on the tabletop" w="700" >}}
    <figcaption>The working cell: stack on the left, sorted colour piles on the right.</figcaption>
  </figure>
  <div>
    <p>I helped tune and test the grasping side alongside our manipulation teammates. Most of the work went into details that only show up on real hardware:</p>
    <ul class="bp-checks">
      <li>Grasp width and force for small, glossy 2×4 bricks</li>
      <li>Pre-grasp clearance and offsets so the fingers don't hit neighbouring bricks</li>
      <li>3D-printed gripper fingers</li>
      <li>Varying layer heights as the stack shrinks</li>
      <li>A camera-to-gripper transform taken from CAD and refined on the robot</li>
    </ul>
    <p class="bp-honest">We demonstrated sorting on the Franka. Measuring pick success across repeated trials is the next step.</p>
  </div>
</div>

</section>

<section class="bp-chapter" aria-labelledby="the-final-pitch">

## The final pitch {#the-final-pitch}

On day six, I presented three parts of the same idea to the jury. The Franka showed how to unload a stack, the real-brick inspector showed how to grade fired bricks, and the simulation showed how those steps could fit into a factory line.

{{< bp-youtube id="UwtO5sTqreg" title="Team KalmanKrew final hackathon presentation demo" >}}

### Beyond the lab table

My teammates built two companion prototypes.

<div class="bp-cards">
  <article>
    {{< bp-video src="real-brick-classifier.mp4" poster="real-brick-classifier-poster.jpg" label="Live real-brick inspection: good and bad predictions" >}}
    <span class="bp-card-kicker">Real-brick inspection</span>
    <h3>Grading real fired bricks</h3>
    <p>SAM 3 masks, hand-crafted colour and texture features, and DINOv2 patch features, fused through two linear SVMs. On 258 photos of the 10 sponsor samples, <b>9 of 10 bricks</b> were graded correctly: all 4 good bricks accepted, 5 of 6 bad ones rejected.</p>
  </article>
  <article>
    {{< bp-youtube id="kOvg-SudFjg" title="Factory simulation: dehacking, inspection, reject removal and palletizing" >}}
    <span class="bp-card-kicker">Factory simulation</span>
    <h3>Scaling the idea up</h3>
    <p>A simulated line covering dehacking, inspection, reject removal and palletizing, with a nominal throughput of <b>≈ 6,400 bricks per hour</b>. The throughput estimate comes from scripted motion and simulated shade labels.</p>
  </article>
</div>

### Six days in Munich

<div class="bp-gallery">
  <figure class="is-wide">{{< bp-img src="final-presentation.jpg" alt="Shiva presenting with a microphone during the final pitch, team behind him" w="1400" >}}<figcaption>Pitching on the final day.</figcaption></figure>
  <figure>{{< bp-img src="team-introduction.jpg" alt="Team KalmanKrew at the opening-day introductions" w="900" >}}<figcaption>Day 1: meeting the team.</figcaption></figure>
  <figure>{{< bp-img src="team-discussion.jpg" alt="Shiva explaining an idea to teammates over a real brick sample" w="900" >}}<figcaption>Working out the approach over a real brick.</figcaption></figure>
  <figure>{{< bp-img src="answering-questions.jpg" alt="Shiva answering jury questions with a microphone, Team KalmanKrew standing beside him" w="900" >}}<figcaption>Questions from the jury.</figcaption></figure>
  <figure>{{< bp-img src="teams-and-marmon.jpg" alt="Teams 8A and 8B with Jeff Garascia and Michael Ostrow from Marmon" w="900" >}}<figcaption>Both Marmon teams with our challenge hosts.</figcaption></figure>
</div>

</section>

<section class="bp-chapter" aria-labelledby="what-comes-next">

## What I’d take into the next project {#what-comes-next}

<div class="project-roadmap bp-lessons">
  <section>
    <span class="roadmap-label">Geometry first</span>
    <h3>Use depth to narrow the search</h3>
    <p>A plane fit and a height histogram narrowed the candidate masks to the working layer. That made it easier to trace failed picks back to the geometry or the mask.</p>
  </section>
  <section>
    <span class="roadmap-label">Calibrate to the scene</span>
    <h3>Use the world as a reference</h3>
    <p>The table doubled as a white reference, and one Lab offset per frame absorbed exposure drift. Both were more reliable than tuning camera settings for every lighting change.</p>
  </section>
  <section>
    <span class="roadmap-label">Close the loop</span>
    <h3>Rescan beats predict</h3>
    <p>Rescanning after a few picks handled disturbed bricks and failed motions far better than trying to plan the whole stack from one frame.</p>
  </section>
  <section>
    <span class="roadmap-label">Next step</span>
    <h3>Validate on unseen bricks</h3>
    <p>The inspector was evaluated on ten physical samples. Testing on unseen bricks and measuring pick success across repeated runs would show how well the system generalises.</p>
  </section>
</div>

<div class="bp-team">
  <p><b>Team KalmanKrew</b>: Shiva Rudra (perception, pitch), <a href="https://www.linkedin.com/in/varun-teja-988359142/">Varun Teja</a> (perception and real-brick classification), <a href="https://tanayranjan.github.io/">Tanay Ranjan</a> (perception–manipulation integration, pick-and-rescan loop), <a href="https://www.linkedin.com/in/ameed-robotics/">Ameed Nazhurudeen</a> (manipulation), and <a href="https://www.linkedin.com/in/ahmet-sinan-erdem/">Ahmet Sinan Erdem</a> (vision and simulation). Mentored by <a href="https://www.petermjso.com/">Peter So</a>; challenge hosted by <a href="https://www.marmon.com/about-us/the-marmon-team/jeff-garascia/">Jeff Garascia</a> and Michael Ostrow from Marmon.</p>
</div>

</section>

</div>
