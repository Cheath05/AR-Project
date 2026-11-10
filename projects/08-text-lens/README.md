# Stage 8 — Seeing and reading

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity 6, AR Foundation, Xcode, Swift · **First stage with a native plugin at its centre**

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **Native plugins:** calling Swift from C# and getting results back, the bridge to every iOS feature Unity does not wrap.
- **Computer vision alongside AR:** running text recognition and image classification on the camera frames ARKit is already capturing.
- **2D to 3D:** turning "the text is at these pixels" into "the text is at this point in the room".

Tracking tells you where things are. This stage is about knowing *what* they are.

## Read first

- Building plug-ins for iOS: https://docs.unity3d.com/Manual/PluginsForIOS.html
- Vision (start with the text recognition articles): https://developer.apple.com/documentation/vision
- Core ML overview, for the object-recognition quests: https://developer.apple.com/documentation/coreml
- Raycasts, again: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/raycasts.html
- How Apple services fit with Unity: [docs/apple-services.md](../../docs/apple-services.md)

Vision is a Swift framework and Unity has no wrapper for it. There is no way around writing some Swift here. Expect the first two tasks to be slow and unglamorous; everything after them is quick.

## Core build: text lens

**Goal:** point the phone at printed text, and a label appears on the text in AR and stays there as you move.

### Tasks

1. **Build the bridge.** Before any vision work, write the smallest possible native plugin: a Swift function that takes a number or string from C# and returns a changed one. Get it to build and run on the phone. What has to be true about a Swift function for C# to call it, and where does the file live in the Unity project?
2. **Send a camera frame to Vision.** Get the current camera image from AR Foundation on the CPU and pass it across the bridge to a Vision text recognition request. For now, just print what Vision found to the Xcode console. How often should you send a frame: every frame, or less? What is the image's orientation compared with the screen?
3. **Get results back into Unity.** Return the recognised strings and their bounding boxes to C#. Vision works off the main thread and finishes later. How does the answer get back, and what format do you send it in? Show the recognised text in a plain 2D debug overlay.
4. **Map the 2D box to a 3D position.** You have a rectangle in image coordinates. Work out the matching screen point, then find where in the room that point lands. Vision's coordinate origin is not the same as Unity's screen origin. Draw both on paper before coding.
5. **Place a label in AR.** Put a world-space label at that 3D position. It should stay on the text when you move the phone, not slide with the screen. What do you do when the same text is recognised again next frame: a new label, or update the old one?

### Done when

- [ ] A C# call reaches Swift and gets an answer back.
- [ ] Printed text is recognised from the live AR camera.
- [ ] A label sits on the text and stays put as you move.
- [ ] Re-recognising the same text does not create duplicate labels.
- [ ] The app stays responsive while recognition runs.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Why can't C# call a Swift method directly? What sits in between?
2. Vision returned normalised coordinates with a different origin. What were the steps to turn that into a point in the room?
3. Recognition takes time and the phone moves meanwhile. How could that put the label in the wrong place, and how might you correct for it?
4. Image tracking (Stage 4) and text recognition both "find something in the picture". What does each know that the other does not?

## Study quest: Homework lens

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md). Add it to study buddy in time for finals.

**Goal:** point at a printed problem and get a hint, then the next step when you ask, never the whole answer at once.

- How does the app know which problem it is looking at? Options: recognise a problem number or a unique phrase with your text lens, or fall back to a Stage 4 marker. Which is more robust on a real worksheet?
- Design the data format: a problem has an ordered list of steps. Write three real problems from a real class by hand.
- Reveal one step per tap. Why is that better for learning than showing the solution?
- You write the hints yourself in this version. Writing them is the studying.
- Stretch (after Stage 12): the assistant generates the next hint.

**Done when** it is a mode in study buddy and loaded with problems for one final.

## Study quest: Vocabulary labels

**Backlog.**

**Goal:** point at everyday objects and see their name in the language you are learning.

- This is image classification, not text recognition. Vision can run a Core ML model on the same frames. Research which ready-made models Apple offers for download.
- The model returns an English label and a confidence. Where does the translation come from: a hand-written dictionary for the words you are studying, or something else?
- What confidence is high enough to show a label? What do you show when unsure?
- Stretch: tap a label to hear it pronounced.

## Play quest: Deck scanner

**Backlog.**

**Goal:** scan your trading cards one by one to build a digital collection list.

- Read the card's name with your text lens. Which part of the card is the name, and how do you ignore the rest?
- Store the collection in the same kind of data file as study buddy.
- Stretch: show collection stats in AR above the deck. This list becomes the roster for Stage 9.

Keep scans and photos of real cards out of the repo (the `private/` folder is git-ignored).

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
