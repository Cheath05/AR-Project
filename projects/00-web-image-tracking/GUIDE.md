# Stage 0 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, what kind of code, in what order, and how to tell each step worked. It does not contain the finished code.

## Before you write anything

1. Read [docs/coding-basics.md](../../docs/coding-basics.md) and do its "Before Stage 0" tutorials (3–4 hours). Skipping this is the main reason people get lost.
2. Run the starter: open `index.html` in VS Code and click **Go Live** in the bottom-right corner. You should see a blue **Start AR** button.
3. Open the browser console: `Cmd + Option + J` in Chrome. Leave it open the whole time. Errors appear here in red.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| HTML tags, and how a page loads a script | The whole file | https://developer.mozilla.org/en-US/docs/Learn_web_development/Getting_started/Your_first_website |
| `const`, functions, calling functions | Every TODO | https://javascript.info/first-steps |
| `import` (using someone else's code) | Top of the script | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules |
| `new Something({ ... })` (making an object with options) | TODO 1, 4 | Covered under "objects" in javascript.info |
| `async` / `await` (waiting for something slow) | TODO 2 | https://javascript.info/async-await |
| Arrow functions `() => { ... }` used as callbacks | TODO 5, 6 | "Arrow functions, the basics" on javascript.info |

## A tour of the file you already have

Open `index.html`. It has four parts:

- **Lines 1–7, the head.** Page title and settings. Leave alone.
- **The `importmap` block.** A lookup table saying "when the code asks for `three`, download it from this address". This is how the page gets three.js and MindAR without you installing anything. Leave alone.
- **The `<style>` block.** How the button and status bar look. Leave alone.
- **The `<script type="module">` block at the bottom.** This is where you work. Inside it:
  - `import` lines bring in three.js (as `THREE`) and MindAR (as `MindARThree`).
  - `setStatus("text")` is a small helper already written for you. Call it to show a message on screen. You will use it a lot, because a phone has no visible console.
  - `async function start() { ... }` holds the six TODOs. It runs when you tap **Start AR**.

**All your code for the core build goes inside `start()`, between the TODO comments, in order.**

## Files you will add

Everything sits in this folder, next to `index.html`:

```
00-web-image-tracking/
├── index.html        (exists; you edit this)
├── targets.mind      (you make this in task 2)
└── assets/           (you make this folder in task 6, for your 3D model)
```

## The work loop

1. Type a line or two in VS Code.
2. Save with `Cmd + S`. The browser reloads by itself.
3. Tap **Start AR**. Look at the page and at the console.
4. Red text in the console? Read it. It gives a line number. Fix that line.
5. Working? Commit, then move to the next TODO.

## Task by task

### Task 1 — See the page on your phone

No code. You already have it running on your Mac with **Go Live**. Do tasks 2–5 on the Mac using its webcam first, and come back to the phone at the end. The brief lists the options; GitHub Pages is the easiest for this repo.

### Task 2 — Make your image target

No code.

1. Pick any photo or picture with lots of detail and save it on your Mac.
2. Open the compiler: https://hiukim.github.io/mind-ar-js-doc/tools/compile
3. Drop your image in, click **Start**, then **Download compiled**.
4. Move the downloaded file into this folder and rename it `targets.mind`.
5. Keep the original picture open on a second screen or print it. That is what you will point the camera at.

**Check:** `targets.mind` appears in the VS Code file list beside `index.html`.

### Task 3 — TODO 1 and TODO 2: camera on

**What you are doing in plain English:** make the MindAR object, tell it where to draw and which target file to use, pull three tools out of it, and switch it on.

**Read this first, with the file open beside it:** https://hiukim.github.io/mind-ar-js-doc/quick-start/overview and the pages that follow it. They contain a short working example. Your job is to understand each line of it and fit the matching lines into your TODOs. Do not paste the whole example over the starter.

**TODO 1 is one statement** shaped like this:

```
const NAME = new ClassName({
  optionOne: value,
  optionTwo: value,
});
```

- `ClassName` is the thing you imported from MindAR at the top of the script.
- It needs two options. One says which page element to draw into; the starter has a `<div>` with the id `container`, and JavaScript can fetch it with `document.querySelector('#container')`. The other says where your `.mind` file is, written as text: `'./targets.mind'`.
- Find the exact option names in the MindAR example.

**TODO 2 is two statements.**

- The first pulls `renderer`, `scene` and `camera` out of the object you just made. The example does this in one line using curly brackets on the left of the `=`. That pattern is called *destructuring*; look the word up.
- The second starts MindAR. Starting takes time (the browser has to ask for the camera), so the line begins with `await`.

**Check:** tap **Start AR**. The browser asks for camera permission, then you see yourself. No cube yet.

**If it fails:**

| Console says | Likely cause |
|---|---|
| `Failed to fetch` or `404` | `targets.mind` has a different name or is in a different folder |
| `X is not defined` | You used a name before creating it, or misspelled it |
| `Unexpected token` | A missing or extra bracket, comma or quote; look at the line number |
| Nothing happens, no error | You saved but the page did not reload; refresh it |

Commit when the camera shows.

### Task 4 — TODO 3 and TODO 4: a cube on the card

**Plain English:** ask MindAR for an "anchor" that follows target number 0. Build a cube. Attach the cube to the anchor.

**TODO 3 is one statement.** Call a function on your MindAR object that adds an anchor, passing it the number `0` (the first image in your target file). Store the result in a `const`. The function's name is in the MindAR example.

**TODO 4 is four statements.** Read "Fundamentals" in the three.js manual, which builds a cube step by step: https://threejs.org/manual/#en/fundamentals

1. Make a **geometry** (the shape). A box needs a width, height and depth.
2. Make a **material** (the surface). Use `MeshBasicMaterial` with a colour for now; it needs no lights.
3. Make a **mesh** from the geometry and the material. The mesh is the actual object.
4. Add the mesh to the anchor. An anchor has a part called `group`, and groups have an `add` function.

Then do **TODO 6** now, out of order, or you will see nothing: the scene has to be redrawn every frame. The renderer has a function that takes a callback and runs it every frame; inside that callback you tell the renderer to render the scene with the camera. It is the last few lines of the MindAR example.

**Check:** point the webcam at your picture. A coloured cube appears on it and follows it.

**Before you run it, write down your prediction:** a cube of size 1 will be how big compared to the picture? Then look. (In MindAR, 1 unit equals the width of the target image.)

**If the cube does not appear:** is the render loop written? Is the picture well lit and filling a good part of the view? Did you add the mesh to `anchor.group` and not to `scene`?

Commit.

### Task 5 — TODO 5: found and lost

**Plain English:** when the target is found, show "Found". When it is lost, show "Lost".

**Read:** https://hiukim.github.io/mind-ar-js-doc/examples/events-handling

An anchor has two properties you can set to a function. MindAR calls that function at the right moment. Each is one statement shaped like:

```
thing.propertyName = () => {
  // what to do when it happens
};
```

Inside each, call `setStatus('...')` with your message.

**Check:** cover and uncover the picture. The status bar at the bottom changes each time.

Commit.

### Task 6 — Your own model and card

1. Find a free model in `.glb` format (sources are at the bottom of [docs/resources.md](../../docs/resources.md)). Make an `assets` folder here and put it inside.
2. Read "Loading a .GLTF file" in the three.js manual: https://threejs.org/manual/#en/load-gltf
3. You need one new `import` line at the top of the script for `GLTFLoader`. It comes from `'three/addons/loaders/GLTFLoader.js'`.
4. Replace your cube code: create a loader, load the file, and inside the loader's callback add `gltf.scene` to the anchor's group.
5. A model lit with nothing looks black. Add a light to the scene; the three.js manual page shows one.
6. It will probably be the wrong size or lying down. Every three.js object has `scale` and `rotation`. Change the numbers, save, look, repeat.
7. Make your own card image, compile a new `targets.mind`, and replace the old one.

**Check:** your model stands upright on your own card.

Commit, then go back to task 1 and get it running on your iPhone.

## Flip cards: how to start

Do this only after the core build works.

1. **Copy, do not rewrite.** Duplicate `index.html` as `flipcards.html` in the same folder. Work in the copy.
2. **Several targets in one file.** The compiler accepts more than one image at once; the order you add them is their number (0, 1, 2…). Read: https://hiukim.github.io/mind-ar-js-doc/examples/multi-tracks
3. **Store the answers in a list.** At the top of `start()`, make an array where position 0 holds the answer for card 0, and so on:

   ```
   const answers = [
     'text for card 0',
     'text for card 1',
   ];
   ```

4. **Loop instead of repeating yourself.** You need an anchor and an answer for every card. Write a `for` loop that runs once per item in `answers` and, each time round, adds an anchor for that number and attaches that answer to it. Get it working for card 0 first with no loop, then turn it into a loop.
5. **Showing text.** The simplest readable way: draw the text onto a small hidden canvas, turn that canvas into a texture, and put the texture on a flat plane. Search the three.js manual for "canvas textures". Write this as its own function that takes a string and returns a mesh, so the loop can call it for each card.

Build order: one card with a plain coloured plane → the plane shows text → two cards → all five.

## Holo stat sheet: how to start

Start from your finished `index.html` copy again (`holo.html`).

1. A flat plane above the card: same technique as the flip-card text, positioned higher using the mesh's `position`.
2. See-through: materials have `transparent` and `opacity` settings.
3. Movement: inside your render loop callback, change the plane's `position` or `rotation` a tiny amount each frame. Look up `Math.sin` and three.js's `Clock` for a smooth bob.

## Committing your work

From the VS Code terminal, after each task works:

```bash
git add -A
```

```bash
git commit -m "Stage 0: task 3, camera feed shows"
```

```bash
git push
```

Do not commit copyrighted card images. Put test photos in a folder named `private/`, which Git ignores.
