# Stage 7 — Faces, hands and sound

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity 6, AR Foundation, an iPhone with Face ID (TrueDepth camera) · **Kept short on purpose**

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **Face tracking:** a live mesh of your face plus its pose, from the front camera.
- **Blend shapes:** around fifty numbers from 0 to 1 describing your expression (jaw open, left eye blink, brow up).
- **Hand pose:** finding finger joints in a camera image with Apple's Vision framework.
- **Audio input:** reading the microphone as data you can react to.

These are the input methods beyond touch. On a headset, hands and voice become the main ones.

## Read first

- Unity Learn, AR face filters (the first units are enough): https://learn.unity.com/course/create-with-ar-face-filters
- Face tracking: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/face-tracking.html
- Tracking and visualising faces (Apple): https://developer.apple.com/documentation/arkit/tracking-and-visualizing-faces
- For the hand quests: https://developer.apple.com/documentation/vision/detecting-hand-poses-with-vision

## Core build: a face mask

**Goal:** a mask or accessory that follows your face and reacts when you make one chosen expression.

### Tasks

1. **Set up a face tracking project.** New AR Foundation project using the front camera. What is different in the session setup compared with every project so far? Can world tracking and face tracking run at the same time on your phone?
2. **Show the face mesh.** Render the mesh ARKit gives you with a plain or wireframe material. Move your face around. How many vertices is it, and does it change shape or only move?
3. **Attach something.** Put an object on your face: glasses, a hat, a nose. It should stay attached as you turn your head. Which point do you attach to, and how do you find it?
4. **Read the blend shapes.** Display a few live values on screen (jaw open, a blink, a smile). AR Foundation exposes these through the ARKit-specific face subsystem rather than the generic one. Find out how to reach it.
5. **Trigger on an expression.** Pick one expression and make something happen when you do it: the mask changes colour, particles fire, a sound plays. What threshold feels right, and how do you stop it firing fifty times while your mouth stays open?

### Done when

- [ ] An object stays attached to your face as you move and turn.
- [ ] At least three blend-shape values are shown live.
- [ ] One expression reliably triggers one effect, once per expression.
- [ ] It works on a second person's face without changes.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Face tracking uses a different camera and sensor than world tracking. Which, and why does that limit what you can combine?
2. A blend shape is a number, not a gesture. How did you turn a noisy number into a clean "it happened" event?
3. Why does the same mesh work on different faces?
4. Which of these inputs (face, hands, voice) would you want for controlling a whiteboard on a headset, and why?

## Study quest: Finger quiz

**Backlog.**

**Goal:** answer multiple-choice questions by holding up one to four fingers to the camera.

- Hand pose comes from Vision, which is a Swift framework with no Unity wrapper. You need a native plugin that passes camera frames to Vision and returns joint positions to C#. Stage 8 teaches this bridge properly; doing this quest first means meeting it early. See [docs/apple-services.md](../../docs/apple-services.md).
- Given the joint positions of one hand, how do you decide a finger is "up"? Work it out on paper before coding.
- How long must the hand be held steady before it counts as an answer?
- Feed it questions from your study buddy data.

**Done when** you can get through ten questions without touching the phone.

## Play quest: Invisible instruments

**Backlog.**

**Goal:** air drums or a theremin played by moving your hand in front of the camera.

- Theremin: map hand height to pitch and horizontal position to volume. How do you generate a continuous tone in Unity?
- Drums: detect a fast downward movement crossing a line. What is the difference between position and velocity here, and which do you need?
- Latency decides whether this feels good. Measure the delay between moving and hearing.

## Play quest: Audio visualiser

**Backlog.**

**Goal:** music playing in the room makes shapes pulse across your walls.

- Microphone input: https://docs.unity3d.com/ScriptReference/Microphone.html
- Start with loudness only: one object that scales with volume.
- Then split the sound into frequency bands (research Unity's spectrum data) and drive a row of bars.
- Place the bars on a real wall using your Stage 2 plane code.
- Stretch: with LiDAR, ripple the room mesh from Stage 6.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
