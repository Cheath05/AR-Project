# Stage 10 — Capstone: idea board

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity, AR Foundation, your iPhone · **Builds on:** Stages 2, 3 and 5

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

A spatial whiteboard for treating ideas as physical things: a board on your wall that you draw on, cover in post-its and papers, and come back to tomorrow.

## What you will learn

- **Gestures:** turning touches into move, resize, draw and drag, and telling them apart.
- **Drawing:** two ways to put ink on a surface (painting into a texture, or building line meshes) and what each costs.
- **Documents as textures:** getting a photo or a PDF page onto an object in the scene.
- **Serialisation:** saving a whole board to a file and rebuilding it exactly.
- **A graph data model:** notes as nodes, connections as edges, which is what the organising quests need.

## The one rule: data first, rendering second

The board is **data**: a list of strokes, notes, papers and connections, with positions measured relative to the board. The Unity objects you see are a **view** of that data.

- The data classes must not reference AR Foundation, cameras or touch input.
- Every change goes: input → change the data → the view updates.
- Saving writes the data. Loading rebuilds the view from it.

Get this right and Stage 13 is a port: on a headset you replace touch input with hands and keep everything else. Get it wrong and Stage 13 is a rewrite.

## Read first

- Your Stage 3 code for anchors and persistence.
- Touch input: https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/Touch.html
- Raycasts: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/raycasts.html
- Render Texture: https://docs.unity3d.com/Manual/class-RenderTexture.html
- Line Renderer: https://docs.unity3d.com/Manual/class-LineRenderer.html
- World-space UI: https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/HOWTO-UIWorldSpace.html
- JsonUtility: https://docs.unity3d.com/ScriptReference/JsonUtility.html
- Where to save files on the device: https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html
- For milestone 4, PDFKit and native plug-ins: https://developer.apple.com/documentation/pdfkit · https://docs.unity3d.com/Manual/PluginsForIOS.html

## Core build: six milestones

Each milestone is a usable tool on its own. Milestones 1–5 are scheduled; milestone 6 is backlog.

Before milestone 1, sketch the data model on paper: what is a board, a stroke, a note, a paper? What fields does each have? You will revise it, but start with a guess.

### Milestone 1 — Board (week of Jun 20)

**Goal:** a whiteboard you can put on a wall or float in the air, move, resize and lock.

1. Place a flat board on a detected vertical plane. What changes if there is no wall and you want it floating in front of you?
2. Drag it along the wall with one finger.
3. Pinch to resize. Should the board scale, or should its width and height change? Think about what happens to things drawn on it later.
4. Add a lock toggle so you stop moving it by accident.
5. Define the board's own coordinate system. Where is (0, 0)? What are the units? Everything in later milestones is stored in these coordinates.

**Done when:** you can hang a board, resize it, lock it, walk away and it stays put.

### Milestone 2 — Draw (week of Jun 27)

**Goal:** ink on the board with colours, widths, an eraser and undo.

1. Turn a touch on the screen into a point on the board, in board coordinates.
2. Draw a stroke as the finger moves. Research both approaches (texture painting and line meshes), pick one and write down why. Which makes undo easier? Which survives resizing the board?
3. Store each stroke as data: its points, colour and width.
4. Add a colour and width picker.
5. Add undo. What does your data model need for undo to be simple?
6. Add an eraser. Does it remove whole strokes or parts of them? Start with whichever is easier.
7. One finger now means both "draw" and "move the board". How will the app know which you meant?

**Done when:** you can sketch a diagram, undo three strokes and erase one, and the strokes stay in place when you resize the board.

### Milestone 3 — Post-its (week of Jul 4)

**Goal:** sticky notes you can add, write on, colour, drag, stack and delete.

1. Add a note at the point you tap.
2. Edit its text with the keyboard (see [TouchScreenKeyboard](https://docs.unity3d.com/ScriptReference/TouchScreenKeyboard.html)). Stretch: dictate it.
3. Change its colour.
4. Drag it across the board. It should follow your finger and stay on the surface.
5. Let notes overlap. Which one is on top, and how does the user change that?
6. Delete a note. What gesture is hard to do by accident?
7. Add notes to the data model and to undo.

**Done when:** you can brainstorm ten notes for a real assignment and rearrange them without fighting the controls.

### Milestone 4 — Papers (week of Jul 11)

**Goal:** photos and PDF pages as sheets you can pin up, flip through and annotate.

1. Put a photo from your library on the board as a sheet. How does an image file become a texture?
2. Keep the sheet's proportions correct for any image.
3. Bring in a PDF. Unity cannot read PDFs itself, so this needs a native plugin that renders a page to an image with PDFKit. If Stage 8's plugin bridge is done, reuse the pattern.
4. Flip between pages of a multi-page PDF.
5. Draw on top of a sheet. Do the strokes belong to the sheet or to the board? What should happen to them when the sheet moves?
6. Decide what the save file stores for a paper: the file itself, or a reference to it?

**Done when:** you can pin up a lecture slide PDF, flip to page 3 and circle something on it.

### Milestone 5 — Save (week of Jul 18)

**Goal:** boards reload exactly where you left them, and you can keep several.

1. Serialise one board (strokes, notes, papers) to a JSON file and rebuild it from that file. Test by saving, clearing the scene and loading.
2. Put the board back in the same place in the room after a relaunch, using what you learned in Stage 3.
3. What should happen if the room is not recognised? Give the user a way to re-place the board without losing its contents.
4. Support several named boards, with a way to switch between them.
5. Add a version number to the save format. Why will you want it later?
6. Decide when saving happens. On a button, on every change, or on a timer? What is lost in each case if the app crashes?

**Done when:** you plan a real assignment on the board, force-quit the app, relaunch the next day and everything is where you left it.

### Milestone 6 — Off the board (backlog)

**Goal:** pull a note off the wall and leave it on your desk or the door as its own object.

1. Detect a drag that leaves the board's edge.
2. Turn the note into a free object with its own anchor.
3. Let it be dropped back onto a board.
4. Extend the data model: a note now belongs to either a board or the room.

**Done when:** a "buy milk" note lives on your door and survives a relaunch.

### Done when (whole core build)

- [ ] Milestones 1–5 work together in one app on your iPhone.
- [ ] The data classes have no reference to AR Foundation or touch input.
- [ ] A saved board reloads with nothing missing or moved.
- [ ] You have used it to plan something real.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Why store positions in board coordinates and not world coordinates? What would break if you used world coordinates and then moved the board?
2. Texture painting or line meshes: which did you choose, and what did you give up?
3. How does your app decide whether a touch means draw, drag a note, or move the board?
4. Open your save file in a text editor. Could a different program rebuild the board from it? What is missing?
5. Which parts of your code would change on a headset with hand tracking, and which would not change at all?

## Study quests

All backlog unless you finish a milestone early.

### Mind-map mode

**Goal:** notes arrange themselves around a central idea, and branches collapse and expand.

- Notes now need parents and children. Extend the data model into a tree.
- Research a simple radial layout. How do you stop branches overlapping?
- Animate notes to their new positions when the layout changes.
- Collapsing a branch hides its children. Are they deleted from the data, or only from the view?

### Kanban wall

**Goal:** three columns, To do / Doing / Done. Flick a note from one to the next.

- Columns are regions of the board. How does a note know which it is in?
- Detect a flick as distinct from a drag.
- Snap notes into a tidy stack within a column.
- Stretch: a count at the top of each column.

### Timeline rail

**Goal:** a rail along the wall where notes snap to dates, for deadlines and history revision.

- Map a date to a position on the rail, and back.
- Decide the scale. A semester of deadlines and a thousand years of history need different ones.
- Notes snap to the nearest date when dropped. Stretch: a "today" marker that moves.

### Room as workspace

**Goal:** one wall per subject or project. Walking to a wall is how you switch context.

- Several boards in one room, each on its own anchor.
- Work out which board the user is facing or nearest to, and make that one active.
- Stretch: boards you are not near fade or simplify to save performance.

## Play quests

### Connections

**Goal:** draw string between notes, corkboard style. The string follows when a note moves.

- A connection is data: which two notes, what colour. This is the "edge" in your graph.
- Draw the line between the two notes' current positions every time either moves.
- What happens to a connection when one of its notes is deleted?

### Idea shelf

**Goal:** notes become small 3D tokens you can sort into trays, bins or a "parking lot" shelf.

- Place a shelf on a horizontal plane using Stage 2 skills.
- Convert a flat note into a token and back.
- Trays are containers in the data model. How is this like the Kanban columns?

### Sketch to shape

**Goal:** draw a rough box, circle or arrow and it snaps to a clean one.

- Look at a finished stroke's points. What distinguishes a rough circle from a rough rectangle?
- Start with one shape. Replace the stroke with a clean version of the same size.
- Stretch: research the "$1 recogniser" and compare it to your approach.

### Photo capture

**Goal:** photograph a real whiteboard or notebook page and it becomes a sheet on your board.

- Capture a camera frame while the AR session is running. See [image capture](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/camera/image-capture.html).
- The photo is taken at an angle. Research perspective correction to flatten it.
- Stretch: with Stage 8's text recognition, turn the handwriting into editable notes.

### Sync

**Goal:** see your boards on an iPad, and in a web page on your laptop.

- Your save file from milestone 5 is the thing to sync. Choose between CloudKit and your own server; see [docs/apple-services.md](../../docs/apple-services.md).
- Two devices edit the same board while offline. What should happen? Start with "last save wins" and note its flaw.
- The web view only needs to draw the data, read-only. This is a second renderer for the same data, which is exactly the skill Stage 13 needs.

### Shared board

**Goal:** a friend's phone sees and edits the same board at the same time.

- Reuse what you learned in Stage 9's *Shared table* about shared space and shared state.
- Send changes, not the whole board. What is the smallest message for "note 7 moved"?
- Distribute through TestFlight.

## On a headset

On iPhone the input is touch on glass. On a headset the same board becomes hand-driven: pinch to grab a note, draw with a fingertip or a controller, push the board back with a palm. That is where it starts to feel physical.

If you kept the rule at the top of this page, the port in Stage 13 is: write new input code for hands, swap the anchor and persistence calls for the headset's versions, and keep your data model, save format and layout logic unchanged.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
