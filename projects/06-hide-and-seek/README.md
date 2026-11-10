# Stage 6 — Depth, occlusion, meshing, lighting

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity 6, AR Foundation · **Best with:** a LiDAR iPhone (Pro models), but not required

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **Depth:** how far away each pixel of the camera image is, and where that estimate comes from with and without LiDAR.
- **Occlusion:** hiding the parts of a virtual object that should be behind something real.
- **Meshing:** a live 3D model of your room that physics and rendering can use.
- **Light estimation:** matching virtual lighting to the real room so objects look like they belong.

This is the stage where AR stops looking like a sticker on the camera feed.

## Read first

- Occlusion: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/occlusion.html
- Meshing: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/meshing.html
- Occluding virtual content with people (Apple): https://developer.apple.com/documentation/arkit/occluding-virtual-content-with-people
- The occlusion and meshing scenes in the AR Foundation samples: https://github.com/Unity-Technologies/arfoundation-samples

## Core build: hide and seek

**Goal:** a virtual character placed in your room that is correctly hidden when it is behind real furniture or a real person, and lit like the room.

Start from a copy of your Stage 2 project so you already have tap-to-place.

### Tasks

1. **Turn depth on and look at it.** Before hiding anything, find out what depth data your phone provides and display it somehow (as a debug image, or by colouring something by distance). Does your iPhone have LiDAR? What does the depth image look like at the edges of objects, and at 5 metres?
2. **Environment occlusion.** Place the character, then walk so a chair or table is between you and it. Make the real object hide it. Try each quality setting the occlusion manager offers. What do you gain and what does it cost in frame rate?
3. **People occlusion.** Have someone walk in front of the character, or hold your hand in front of the phone. Make them hide it. Is this the same mechanism as task 2 or a different one? When does it fail?
4. **Meshing, with a fallback.** With LiDAR: generate the room mesh, show it as a wireframe, then make it invisible but still able to hide and collide with virtual objects. Without LiDAR: there is no mesh, so work out how to get as close as you can using the planes you already detect. What can planes not represent?
5. **Light estimation.** Make the character's lighting follow the room: dim the lights and it should get darker. Which values does ARKit give you, and which of them does the rear camera actually support?

### Done when

- [ ] The character is hidden behind a real chair and reappears as you walk around.
- [ ] A person walking in front hides it.
- [ ] Something virtual collides with real geometry (mesh or planes).
- [ ] Its brightness changes when the room lights change.
- [ ] You have noted the frame rate with everything on.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Where does depth come from on a phone with no LiDAR? Why is it worse on blank walls?
2. Why are occlusion edges fuzzy or flickery? What would a perfect depth sensor change?
3. A mesh and a set of planes both describe the room. When is each the better tool?
4. The Quest 3 has a depth sensor and the Quest 3S does not. Having done this stage, what difference would you expect between them?

## Study quest: Life-size models

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md).

**Goal:** a full-scale skeleton, organ, planet or machine standing in your room that you can walk around and behind.

- Find a model for something you are studying (see "Free 3D models" in [docs/resources.md](../../docs/resources.md)) and get its real-world scale right. How do you check a model is truly 1.7 m tall?
- With occlusion on, walk behind furniture and look back at it. Does it feel present?
- Add labels that always face you. What should happen to a label when the part it names is hidden?
- Stretch: tap a part to isolate it, borrowing from *Exploded diagrams*.

**Done when** you have used it to learn at least five named parts, then add it as a mode in study buddy.

## Play quest: Portal door

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md).

**Goal:** a doorway on your wall. Through it you see another world; walk through and you are inside it, looking back at your room.

- The trick is the stencil buffer: draw the other world only where the door's shape has been drawn first. Read: https://docs.unity3d.com/Manual/SL-Stencil.html
- Get it working from the outside first: world visible only through the door.
- Then the hard part: what has to flip at the moment the camera passes through the doorway? How do you detect that moment?
- Stretch: real furniture occludes the door frame.

## Play quest: Marble run

**Backlog.** Needs the room mesh, so best with LiDAR.

**Goal:** drop virtual marbles that roll across a real table, fall off the edge and bounce on the real floor.

- The mesh needs colliders. What does that cost, and how often should it update?
- Marbles that fall through the floor are the classic bug. Why does it happen with fast, small objects?
- Stretch: place virtual ramps and funnels to build a course that uses your real furniture.

## Play quest: Room scanner

**Backlog.** LiDAR only.

**Goal:** scan your room and export it as a 3D file you can open on your Mac.

- The mesh is already in memory from task 4. What file format is simplest to write by hand?
- How do you get a file off the phone?
- Stretch: bring the scan into the Unity editor as a test room for XR Simulation, so you can test at your desk.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
