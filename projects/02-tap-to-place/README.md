# Stage 2 — World tracking, planes, raycasts

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity 6, AR Foundation, your iPhone

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **World tracking (SLAM):** how the phone works out its own position in the room from the camera and motion sensors, with no marker.
- **Plane detection:** how flat surfaces such as floors, tables and walls are found and grow over time.
- **Raycasts:** turning a tap on a 2D screen into a point on a real 3D surface.
- **World-space UI:** labels and buttons that live in the room, not on the screen.

In Stage 0 the card was the origin. Here the room is.

## Read first

- AR Foundation, plane detection: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/plane-detection.html
- AR Foundation, raycasts: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/raycasts.html
- Unity Learn, AR markers and planes (the planes half): https://learn.unity.com/course/create-with-ar-markers-and-planes
- This repo's [setup guide](../../docs/setup.md), section 3.

## Core build: tap to place

**Goal:** scan your room, tap a real surface, and an object appears there and stays put as you walk around it.

There is no starter. Create the Unity project inside this folder. The `SimpleAR` scene in the AR Foundation samples is a working reference to compare against, after you have tried: https://github.com/Unity-Technologies/arfoundation-samples

### Tasks

1. **Set up an AR scene.** Create a project with AR Foundation and the ARKit plugin, and build a scene that shows the camera feed on your phone. Which two objects does an AR scene need that a normal scene does not, and what is each one for?
2. **See the planes.** Make detected planes visible. Walk around and watch them appear, grow and merge. Which surfaces are found quickly and which never are? Why might a plain white wall be hard?
3. **Cast a ray.** When you touch the screen, find out whether that touch lands on a detected plane, and log where. A physics raycast and an AR raycast are different things. Which do you need here, and why?
4. **Place an object.** Put a cube at the point you hit, sitting flat on the surface. Before you run it, predict: if the cube is 1 unit wide, how big will it be in your room? Compare with the same question in Stage 0.
5. **Move and rotate.** Select a placed object and drag it along the surface with one finger; rotate it with two. How does your code decide whether a touch means "place a new one" or "move this one"?
6. **Delete, and tidy up.** Add a way to remove an object, and a button that hides the plane visuals once you are done placing. Does hiding the visuals stop plane detection? Should it?

### Done when

- [ ] Objects appear where you tap on a real floor or table.
- [ ] They stay in place when you walk around them and come back.
- [ ] You can move, rotate and delete them.
- [ ] Plane visuals can be switched off without breaking placement.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. With no marker in view, how does the phone know it has moved 30 cm to the left?
2. Where is the world origin (0, 0, 0) in your scene, and when was it decided?
3. Cover the camera for five seconds, then uncover it. What happens to your objects, and why?
4. Your objects are positioned in world space. Detected planes shift slightly as the phone learns more. What could go wrong?

Question 4 is the gap that Stage 3 fills.

## Study quest: Exploded diagrams

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md). Do this one **or** *Graph in your room*; pick whichever fits a spring class.

**Goal:** place a 3D model of something you need to learn (a cell, a heart, an engine) on your desk, pull it apart, and quiz yourself on the parts.

- Find a model made of separate named parts. Free sources are listed in `docs/resources.md`; check the licence.
- "Explode" it: each part moves outward from the centre. What is the simplest rule that gives each part a direction and a distance? Drive it with a slider.
- Tap a part to see its label in world space, facing you. World-space UI: https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/HOWTO-UIWorldSpace.html
- Quiz mode: the app names a part and you tap it. Keep score.
- Stretch: load part names and descriptions from a JSON file so the same app works for any model.

**Done when** you can name every part of one model for a real class without the labels.

## Study quest: Graph in your room

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md).

**Goal:** a 3D surface such as z = f(x, y) standing on your table that you can walk around.

- A surface is a grid of points joined into triangles. Research how Unity builds a `Mesh` from vertices and triangles in code.
- Start with one hard-coded function. Then add sliders for its parameters and rebuild the mesh as they change.
- Colour it by height so the shape reads clearly.
- Stretch: draw the axes with tick labels. Stretch further: tap a point on the surface to see its coordinates and slope.

**Done when** you have used it to look at a function from a class and noticed something a flat plot hid.

## Study quest: Physics sandbox

**Scheduled:** backlog.

**Goal:** throw virtual balls in your room with gravity and friction you control, and see the forces drawn live.

- Detected planes need colliders for balls to bounce on them. Find out how.
- Launch a ball from the phone in the direction it is pointing. Unity physics: https://docs.unity3d.com/Manual/PhysicsSection.html
- Add sliders for gravity, bounciness and friction. Try the Moon.
- Draw each ball's velocity as an arrow that updates every frame.
- Stretch: trace the path and compare it with the parabola you would calculate by hand.

## Play quest: Measuring tape

**Scheduled:** backlog.

**Goal:** tap two points in the room and read the distance between them.

- Two raycast hits give two points. Draw a line between them: https://docs.unity3d.com/Manual/class-LineRenderer.html
- Show the distance at the midpoint, facing the camera.
- Measure something you can check with a real ruler. How far off is it, and what makes the error better or worse?
- Stretch: chain points to measure a perimeter or an area.

## Play quest: AR graffiti / light painting

**Scheduled:** backlog.

**Goal:** draw in mid-air by moving the phone, then walk around your strokes.

- The phone's position every frame is a stream of points. Which object in the scene gives you that?
- Turn the points into a line while a finger is held down. How often should you add a point: every frame, or only after moving some distance?
- Add colours, widths and undo.
- Stretch: draw on surfaces instead, using raycasts.

## Play quest: Desk tower defence / floor is lava

**Scheduled:** backlog.

**Goal:** a small game where your real room is the level.

- Pick one. Tower defence: enemies walk across a detected table toward a goal and you tap to place towers. Floor is lava: the detected floor is deadly and you place safe platforms.
- The plane's boundary gives you the play area. How do you get it, and how do you keep things inside it?
- Keep the rules tiny: one enemy type, one tower, a score.
- Stretch: difficulty that scales with the size of the surface.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
