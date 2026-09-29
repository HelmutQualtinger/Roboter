# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file, dependency-free 5-axis robot arm simulator built with three.js. Everything — 3D scene, kinematics, collision detection, grasp logic, UI, and the intro demo — lives in `robot-arm.html`. There is no build step, no package manager, and no test suite.

`prompt.md` is a running log of the German-language prompts that shaped this project's requirements (axes, collision behavior, gripper camera, floor texture, demo sequence). Treat it as the spec/changelog: when a new feature request arrives, check whether it's already described there before re-deriving requirements from the code.

## Running / testing

Open `robot-arm.html` directly in a browser (`open robot-arm.html` on macOS), or serve it with any static file server (avoid port 5000). Three.js is loaded from a CDN (`cdn.jsdelivr.net/npm/three@0.128.0`), so an internet connection is required. There is no automated test suite — verify changes by loading the page and exercising the sliders/demo manually (or via the `run` skill / browser automation tools).

For debugging in the browser console, `window.__demoDebug()` returns the current grasp state, TCP-to-block distance, joint angles, and block position. `window.__demoEvents` accumulates a log of collision/grasp/release events with timestamps.

## Architecture

Everything is one IIFE in `robot-arm.html`. Key pieces, in dependency order:

**Kinematic chain**: a nested `THREE.Group` hierarchy — `axis1Group` (base yaw) → `shoulderGroup` (a2, upper arm) → `elbowGroup` (a3, forearm) → `wristRoll` (a4, gripper roll) → `jawPivotL`/`jawPivotR` (a5, jaw open/close). `state = {a1..a5}` holds the current joint values; `applyStateToRobot()` writes them onto the actual `THREE.Object3D` rotations each frame. A separate `applyCandidatePose(s)` does the same for a *hypothetical* state without mutating `state`, used by the collision/grab checks below.

**Collision system**: `computeClearance(candidateState)` approximates the gripper as a few spheres (`collisionPoints`: gripper body, both jaw tips, TCP) and returns the minimum signed clearance against the floor plane and (if not yet grasped) the block. Negative means penetration. Slider input handlers and the demo's `tweenAxis()` both call this before committing a candidate value: a move into deeper collision is rejected unless it would also trigger a valid grasp (`wouldTriggerGrab`) — this is why grasping always takes priority over the collision lock even when jaw tips are geometrically closer to the block than the TCP.

**Grasp logic**: `updateGrabLogic()` runs every frame, not just on slider input (this matters for the autonomous demo, where the block must be detected as grabbed while joints are still tweening). Grasping re-parents the block mesh onto `wristRoll` via `reparentPreservingWorld()` (decomposes world matrix into the new parent's local space) so it moves rigidly with the gripper; releasing re-parents back to `scene` and starts a short `settling` tween that drops the block to the floor.

**Grasp object**: `block` is a dodecahedron (`DodecahedronGeometry`, one vertex color per pentagon face, rotated so a face points down and it rests flat at y = `BLOCK_SIZE`/2). Its face-to-face width equals `BLOCK_SIZE`; the gripper-roll solver still assumes cube-like 90° symmetry.

**Saturn**: `saturnSystem` (planet, ring, three moons on individually tilted orbit planes with line loops) sits at the zenith (`SATURN_Y`); `updateSaturn()` animates the moons each frame. All materials are `MeshBasicMaterial` with `fog:false`.

**Gripper camera**: `gripperCam` is a second `THREE.PerspectiveCamera` parented to `wristRoll`, positioned at the jaw hinge line looking down the gripper's local +Y (same direction as the TCP), rendered into a second `THREE.WebGLRenderer` targeting the small `#cam-canvas` monitor. Both renderers render the same `scene` every frame in `animate()`.

**Intro demo**: `buildDemoSteps()` computes the sequence of single-axis moves (approach → hover → descend → grasp → move → release → return → point at Saturn) using numerical inverse kinematics (`solveA2A3ForPoint`, a coordinate-descent search over a2/a3 against the real forward kinematics — not a closed-form solution) rather than hand-derived angles. `runIntroDemo()` then plays these steps one axis at a time via `tweenAxis()`, which respects the same collision-clearance rule as manual slider input, disables the sliders, shows the demo banner, and drives a Web Audio warning tone (`startWarningTone`/`stopWarningTone`) — audio only starts after a user gesture unlocks the `AudioContext`, per browser autoplay policy.

**Demo finale / camera zoom**: after returning home, `solvePointAtSaturn()` (coarse + fine grid search over a1–a3) aims the gripper axis at Saturn, then `zoomGripperCam()` tweens the gripper-camera zoom until the outermost moon orbit fills `SATURN_FRAME_FILL` of the view. `setGripperZoom()` is the single place that sets `gripperCam.fov` and syncs the `#cam-zoom` slider (log scale 1×–10×); reset and demo start call `resetGripperCamZoom()`.

## Conventions specific to this file

- All UI-facing strings and code comments are German; keep new ones consistent with that.
- Physical/geometric constants (`BLOCK_SIZE`, `COLLIDE_R_*`, `GRAB_DISTANCE`, arm lengths) are tuned together — several existing comments explain *why* a given radius or distance was chosen (e.g. why the block-collision radius is smaller than its diagonal, why the gripper-body collision radius is smaller than its bounding sphere). Read the surrounding comment before changing one of these in isolation, since they were tuned against each other to make the gripper reach the floor/block without false collision locks.
- `HOME` defines the reset pose; `blockStartPos` defines where the block resets to. Both are referenced by the reset button and the demo.
