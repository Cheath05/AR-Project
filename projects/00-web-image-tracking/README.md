# Stage 0 — First AR: image targets

**Scheduled:** Nov 9 – Nov 22, 2026 · **Tools:** a browser, a text editor, your iPhone · **No Unity yet**

## What you will learn

- **Image target:** a flat picture the software knows in advance and can find in the camera feed.
- **Pose:** the position and rotation of that picture relative to the camera, recomputed every frame.
- **Coordinate frames:** why "put the model at (0, 0, 0)" means "on the card" and not "in the corner of the room".

These three ideas come back in every later stage.

## Read first

- MindAR docs, the "Image Tracking → Quick Start" pages: https://hiukim.github.io/mind-ar-js-doc/
- three.js manual, "Fundamentals" and "Scene graph": https://threejs.org/manual/
- Apple on what makes an image easy to detect (applies to any tracker): https://developer.apple.com/documentation/arkit/detecting-images-in-an-ar-experience

## Core build: a model on a card

**Goal:** point your iPhone at a card you printed and see a 3D model sitting on it in Safari.

The starter is [index.html](index.html). It loads three.js and MindAR and has an empty `start()` function with numbered TODOs. It does nothing visible until you fill them in.

### Tasks

1. **Serve the page to your phone.** Safari only allows the camera on HTTPS (or `localhost`). Find a way to open the starter on your iPhone over HTTPS. Options to research: GitHub Pages for this repo, a tunnelling tool, or a local dev server with a self-signed certificate. Check that the page's "modules loaded" message appears on the phone.
2. **Compile an image target.** Pick an image, run it through the MindAR compiler (link in `docs/resources.md`) and save the `.mind` file next to `index.html`. Look at the feature points the compiler shows you. Why did it pick those?
3. **Start tracking.** Fill in TODO 1 and 2 so the camera feed appears and MindAR starts. Nothing is drawn yet.
4. **Put a cube on the card.** Fill in TODO 3 and 4. Before you run it, predict: if the cube is 1 unit wide, how big will it look relative to the card? Then check.
5. **React to found and lost.** Fill in TODO 5. Show a message when the card is detected and when it is lost.
6. **Make it yours.** Replace the cube with a real model (glTF) and the test image with a card you designed. Fix its scale and rotation so it stands upright on the card.

### Done when

- [ ] The model appears on your own printed card on your iPhone.
- [ ] It stays attached as you move the phone and tilt the card.
- [ ] You get a visible signal when tracking is found and lost.
- [ ] The code is committed, with no copyrighted card art in the repo.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. What makes an image a good target? Try a plain logo and a busy photo and compare.
2. The model is a child of the "anchor". What does that mean, and what would happen if you added it to the scene root?
3. What happens to tracking when you cover half the card? When the light is dim? Why?
4. MindAR gives you a pose for the card relative to the camera. Does it know where the card is in the room?

Question 4 is the gap that Stage 2 fills.

## Study quest: Flip cards

**Scheduled:** Nov 16 – Nov 22. This is your first real study tool; build it for a class with a December final.

**Goal:** a set of printed question cards. Point the phone at one and the answer appears above it.

- Design at least 5 cards. Each needs to look different enough for the tracker to tell apart. What could you add to a text-only card to make that true?
- MindAR can compile several images into one target file. Find out how, and how your code knows which one was found.
- Decide where answers live. Hard-coded is fine to start; a JSON file is better. What would the format look like?
- Show the answer as text in 3D. Research how three.js draws text, and pick the simplest approach that is readable on a phone.
- Stretch: tap the screen to flip between a hint and the full answer. Stretch further: a "got it / missed it" tap that keeps score.

**Done when** you have revised with it at least once and noted in the log what was annoying. Those notes feed Stage 5.

## Play quest: Holo stat sheet

**Goal:** a trading card with stats floating above it, like a hologram.

- Lay out name, HP and two attacks on a panel above the card.
- Make it feel holographic: transparency, a slow bob or rotation, a glow. Research three.js materials and the render loop.
- Stretch: animate the panel unfolding when the card is first detected.

Use a card you drew yourself for anything you commit.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
