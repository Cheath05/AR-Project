# Stage 10 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, what kind of code, in what order, and how to tell each step worked. It gives you empty skeletons. Filling them in is the stage.

## Before you write anything

1. Create a new Unity project named `IdeaBoard` in this folder, set up for AR Foundation as in [docs/setup.md](../../docs/setup.md). Build the empty AR scene to your phone to confirm the camera shows.
2. Have your Stage 2 project (plane detection, raycasts) and Stage 3 project (anchors, saving a world map) open for reference. You will reuse both.
3. On paper, sketch the data model the brief asks for: what a board, stroke, note and paper each store.

This stage uses the same split as Stage 9, and it matters even more here:

```
finger → Input scripts → change the DATA → Views redraw from the data
                              ↓
                         save / load
```

Input scripts never move a Unity object directly. They change the data and the view follows.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| Touch phases (began, moved, ended) and two-finger gestures | Milestones 1–3 | https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/Touch.html |
| Local vs world coordinates (`InverseTransformPoint`) | Milestones 1–2 | Search the Scripting API for `Transform.InverseTransformPoint` |
| Line Renderer (drawing a line through a list of points) | Milestone 2 | https://docs.unity3d.com/Manual/class-LineRenderer.html |
| Painting into a texture, the other way to draw | Milestone 2 | https://docs.unity3d.com/Manual/class-RenderTexture.html |
| A stack for undo (last in, first out) | Milestone 2 | Search the C# docs for `Stack<T>` |
| The on-screen keyboard | Milestone 3 | https://docs.unity3d.com/ScriptReference/TouchScreenKeyboard.html |
| Loading an image file into a texture | Milestone 4 | https://docs.unity3d.com/ScriptReference/ImageConversion.LoadImage.html |
| Serialisation: objects to JSON text and back | Milestone 5 | https://docs.unity3d.com/ScriptReference/JsonUtility.html |
| Reading and writing files | Milestone 5 | https://learn.microsoft.com/en-us/dotnet/api/system.io.file |
| Where an app may save files on the phone | Milestone 5 | https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html |

## Files you will add

```
10-idea-board/
└── IdeaBoard/                         (Unity project)
    └── Assets/
        ├── BoardData/                 data only: no AR Foundation, no touch input
        │   ├── BoardData.cs
        │   ├── StrokeData.cs
        │   ├── NoteData.cs
        │   ├── PaperData.cs           (milestone 4)
        │   ├── BoardDocument.cs       the list of changes + undo
        │   └── BoardStore.cs          save and load (milestone 5)
        ├── Scripts/
        │   ├── Input/
        │   │   ├── BoardPlacer.cs     (milestone 1)
        │   │   ├── TouchRouter.cs     decides what a touch means
        │   │   ├── DrawTool.cs        (milestone 2)
        │   │   └── NoteTool.cs        (milestone 3)
        │   └── View/
        │       ├── BoardView.cs       rebuilds everything from BoardData
        │       ├── StrokeView.cs
        │       ├── NoteView.cs
        │       └── PaperView.cs
        ├── Plugins/iOS/               (milestone 4: PDF bridge, Stage 8 pattern)
        └── Prefabs/                   Board, Stroke, Note, Paper
```

The data classes may use simple Unity value types such as `Vector2` and `Color`, because `JsonUtility` saves those for you. They must not mention `Transform`, `GameObject`, AR Foundation or touches.

## The data skeletons

Create these before milestone 1, even though most stay empty for a while. `[Serializable]` plus **public fields** is what lets `JsonUtility` save a class.

```
using System;
using System.Collections.Generic;
using UnityEngine;

[Serializable]
public class StrokeData
{
    // an id (string), a list of Vector2 points in BOARD coordinates,
    // a colour, a width
}

[Serializable]
public class NoteData
{
    // an id, text, colour, position in board coordinates, a stacking order number
}

[Serializable]
public class BoardData
{
    // a format version number (see milestone 5)
    // a name, a width and height in metres
    // List<StrokeData>, List<NoteData>, later List<PaperData>
}
```

`BoardDocument.cs` is the only door through which the board changes:

```
public class BoardDocument
{
    public BoardData Data { get; private set; }
    public event Action Changed;          // views listen to this

    public void AddStroke(StrokeData stroke)   { /* 1. add  2. record for undo  3. raise Changed */ }
    public void RemoveStroke(string id)        { }
    public void AddNote(NoteData note)         { }
    public void MoveNote(string id, Vector2 p) { }
    public void Undo()                         { /* reverse the most recent recorded change */ }
}
```

Two JsonUtility limits to know now, because they shape your classes: it cannot save a `Dictionary`, and it cannot save a bare list as the top-level thing. Both are avoided by keeping lists as fields inside `BoardData`.

## The work loop

Change → build → run on phone. Where you can, test the data without the phone: `BoardDocument` and `BoardStore` are plain C#, so the Edit Mode tests you learned in Stage 9 work on them. Undo and save/load are the two parts worth testing that way.

## Milestone by milestone

### Milestone 1 — Board

**Plain English:** tap a wall to hang a flat rectangle. Drag to slide it, pinch to resize, toggle a lock.

**Unity setup:** make a `Board` prefab: an empty parent with a flat child (a Quad) scaled to, say, 1.2 × 0.8 m. Give the Quad a collider so raycasts can hit it. Set your plane manager to detect vertical planes.

**`BoardPlacer.cs`**, on a GameObject in the scene:

```
public class BoardPlacer : MonoBehaviour
{
    // references: the AR raycast manager, the anchor manager, the Board prefab
    // state: the placed board (or null), a "locked" bool

    private void Update()
    {
        // 1. no touches? return
        // 2. no board yet and one finger tapped: raycast against planes,
        //    place the board at the hit, attach it to an anchor (Stage 3)
        // 3. board exists, not locked, one finger moving: slide it along the plane
        // 4. board exists, not locked, two fingers: compare the distance between
        //    them now to the distance last frame, and resize by that ratio
    }

    public void ToggleLock() { }   // wire to a UI button
}
```

For step 4, follow the brief's hint: change `BoardData.width` and `height` and let the view resize the Quad. Do not scale the parent, or everything drawn later will stretch.

**Board coordinates (task 5).** Decide now: origin at the board's centre, units in metres, x to the right, y up. Then a point on the board is just the hit point converted into the board's local space. Write one helper and use it everywhere:

```
public Vector2 WorldToBoard(Vector3 worldPoint) { /* use the board transform's InverseTransformPoint */ }
public Vector3 BoardToWorld(Vector2 boardPoint) { /* the reverse */ }
```

**Check:** hang the board, resize it, lock it, walk to another room and back. It has not moved. Log `WorldToBoard` of a tap at the board's centre: it prints close to (0, 0).

**If it fails:**

| What you see | Likely cause |
|---|---|
| Taps never place a board | No vertical planes detected yet (visualise planes as in Stage 2), or the raycast is set to the wrong trackable type |
| Board faces the wrong way or lies flat | Use the hit pose's rotation, and check which way your Quad faces in the prefab |
| Board drifts | It is not parented to an anchor |
| Pinch resizes wildly | You are comparing to the starting distance every frame instead of the previous frame's |

### Milestone 2 — Draw

**Plain English:** while a finger moves across the board, collect points in board coordinates. When it lifts, that list becomes a stroke in the data.

**Recommendation for a first version: line meshes with a Line Renderer.** One stroke is one object, so undo and erase mean "remove that object", and strokes stay crisp when the board resizes. Texture painting is worth trying afterwards for comparison, as the brief asks.

**Hitting the board:** a touch is a 2D screen position. Turn it into a ray from the camera (`Camera.ScreenPointToRay`), raycast against the board's collider with `Physics.Raycast`, and convert the hit point with your `WorldToBoard` helper. This is ordinary Unity physics, not the AR raycast.

**`DrawTool.cs`:**

```
public class DrawTool : MonoBehaviour
{
    // references: the camera, the board, the BoardDocument
    // settings: current colour, current width
    // state: the StrokeData being drawn right now (or null)

    public void OnTouchBegan(Vector2 screenPos) { /* start a new StrokeData if the ray hits the board */ }
    public void OnTouchMoved(Vector2 screenPos) { /* add a point, but only if it moved far enough from the last one */ }
    public void OnTouchEnded()                  { /* hand the finished stroke to document.AddStroke */ }
}
```

**`StrokeView.cs`**, on a `Stroke` prefab that has a Line Renderer: one method, `Show(StrokeData data)`, which sets the Line Renderer's point count, positions, colour and width from the data. Turn off "use world space" on the Line Renderer so points are relative to the board.

**`BoardView.cs`** listens to `document.Changed` and makes the scene match the data. The simplest correct version destroys all stroke objects and recreates them from the list. It is wasteful, and it is fine for now.

**Undo:** inside `BoardDocument`, keep a `Stack` of "how to reverse the last change". Adding a stroke pushes "remove stroke X". `Undo()` pops one and does it.

**Draw or move? (task 7).** Put the decision in one place, `TouchRouter.cs`, which reads touches and forwards them to exactly one tool. Start with a mode button (Draw / Move / Notes). Refine later.

**Check:** draw a house. Undo the roof. Resize the board: the drawing keeps its shape and position on the board.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Lines appear somewhere else in the room | Line Renderer is in world space, or you stored world points instead of board points |
| Lines hidden inside the board | Lift strokes a millimetre off the surface |
| Jagged or very heavy strokes | Adding a point every frame; add only after a minimum distance |
| Undo removes it from the data but it stays on screen | `Changed` was not raised, or the view is not listening |

### Milestone 3 — Post-its

**Plain English:** a note is a small coloured square with text. Its position, text and colour live in a `NoteData`.

1. Make a `Note` prefab: a Quad with a collider and a world-space text label.
2. `NoteView.cs`: `Show(NoteData data)` sets position, colour and text. Give it a public `Id` so a raycast hit can be traced back to its data.
3. `NoteTool.cs` has the same three touch methods as `DrawTool`. On touch-began, raycast: did you hit a note or bare board? Bare board → `document.AddNote`. A note → remember its id and start dragging; each move calls `document.MoveNote`.
4. Editing text: open `TouchScreenKeyboard` on a double-tap, and copy its text into the data when it closes. Read the linked page for how to tell that it has closed.
5. Stacking: the note you touch gets the highest order number. The view uses that number to lift it slightly off the board.
6. Delete: pick something deliberate, such as dragging the note to a bin in the corner.
7. Every one of these goes through `BoardDocument` and records an undo entry.

**Check:** ten notes for a real assignment, rearranged, with one edited and one deleted, then three undos that each do what you expect.

### Milestone 4 — Papers

**Plain English:** a paper is a note that shows an image instead of text.

1. **First with a bundled image.** Put a JPEG in the project, show it on a `Paper` prefab, and size the quad from the texture's width ÷ height so it is not stretched. This proves the view before any file picking.
2. **Then from a file.** Read the file's bytes and load them into a `Texture2D` (see the `ImageConversion.LoadImage` link).
3. **Picking from the photo library** needs iOS's picker, which Unity does not expose: a native plugin, same shape as Stage 8 (one C# call in, one callback out carrying a file path).
4. **PDF pages** need a second small plugin function: "render page N of this PDF to a PNG file and give me the path". PDFKit (https://developer.apple.com/documentation/pdfkit) does the rendering. Unity then loads that PNG exactly as in step 2.
5. **`PaperData`:** an id, position, size, the file name, and the current page number. Copy picked files into the app's own storage folder and save only the file *name*. Why not the full path? (Print `Application.persistentDataPath` on two different launches and compare.)
6. **Drawing on a paper:** the simplest rule is that strokes always belong to the board. Note what goes wrong when the paper moves, then decide whether to give strokes an optional "parent paper id".

**Check:** a lecture PDF on the wall, flipped to page 3, with a circle drawn on it.

### Milestone 5 — Save

**Plain English:** turn `BoardData` into text, write it to a file, and later read it back and hand it to the view.

**`BoardStore.cs`:**

```
public static class BoardStore
{
    public static void Save(BoardData data, string boardName)
    {
        // 1. turn data into a JSON string
        // 2. build the file path inside Application.persistentDataPath
        // 3. write the text to the file
    }

    public static BoardData Load(string boardName)
    {
        // 1. does the file exist? if not, return a new empty board
        // 2. read the text
        // 3. turn it back into BoardData
        // 4. check the version number
    }

    public static List<string> ListBoards() { /* the names of every saved board */ }
}
```

Because the view already rebuilds itself from data (milestone 2), loading is: `Load`, hand the result to `BoardDocument`, raise `Changed`. If you find yourself writing special "restore" code in the views, something has leaked out of the data.

**Putting the board back in the room** is your Stage 3 world-map code. Save the anchor's identifier alongside the board. When the room is not recognised, fall back to milestone 1's placing flow with the saved contents.

**Version number:** set `version = 1` now. When you add a field later, old files will load with that field empty. The version tells you which files need fixing up.

**Check:** save, then look at the actual file. In Xcode: **Window → Devices and Simulators**, select the app, download its container, and open the JSON in VS Code. Can you read your notes in it? Then force-quit, relaunch, and confirm nothing is missing or moved.

**If it fails:**

| What you see | Likely cause |
|---|---|
| File is `{}` | Fields are private, or the class lacks `[Serializable]` |
| Loads fine in the editor, empty on the phone | A hard-coded Mac path; use `Application.persistentDataPath` |
| Papers are blank after relaunch | You saved a full path that changed, not a file name |
| Board loads but in the wrong place | Positions were saved in world coordinates somewhere |

### Milestone 6 — Off the board (backlog)

A note gains one more field: which board it belongs to, or none. In `NoteTool`, when a drag's ray stops hitting the board, switch to placing the note on whatever plane the ray does hit, and give it its own anchor.

## Quests: how to start

**Mind-map mode.** Add `parentId` to `NoteData`. Write a plain function that takes the notes and returns a new position for each, placing children evenly on a circle around their parent. Test it on paper with five notes before animating anything.

**Kanban wall.** A column is a range of x values in board coordinates. A note's column is worked out from its x, so nothing new needs saving. A flick is a drag that ends while the finger is still moving fast: keep the last two touch positions and their times.

**Timeline rail.** Two plain functions: date → x position, and x position → date. Write and test those first with no visuals.

**Room as workspace.** A list of boards, each with its own anchor and file. Each frame, the active board is the one closest to the centre of the camera's view.

**Connections.** Add `List<ConnectionData>` to `BoardData`: two note ids and a colour. The view draws a two-point Line Renderer between the notes' current positions. Deleting a note must also remove its connections; do that inside `BoardDocument`.

**Idea shelf.** A shelf is a second container beside the board in the data. Moving a note to it is the same "change which container owns it" step as milestone 6.

**Sketch to shape.** When a stroke ends, examine its points: does it end near where it started? How close are its points to its bounding box, or to a circle around its centre? Start by recognising only a rectangle.

**Photo capture.** `ARCameraManager.TryAcquireLatestCpuImage` gives you the current camera frame; convert it, save it as a JPEG, and add it as a paper. Dispose of the image when done, as the linked manual page stresses.

**Sync.** Your save file is the thing to sync. The quickest route is the server you build in Stage 11: one endpoint to upload a board's JSON, one to download it. Come back to this quest after that stage.

**Shared board.** Send the same small change messages `BoardDocument` already uses internally ("add stroke", "move note"), using the phone-to-phone bridge from Stage 9's *Shared table*.

## Committing your work

```bash
git add -A
```

```bash
git commit -m "Stage 10: milestone 2, strokes with undo"
```

```bash
git push
```

Keep lecture slides, textbook scans and personal photos out of the repo. Put test files in a `private/` folder, which Git ignores.
