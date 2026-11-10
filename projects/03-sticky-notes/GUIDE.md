# Stage 3 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which scripts to create, which method each piece goes in, and how to tell each step worked. It does not contain the finished code.

## Before you write anything

1. Finish Stage 2. This stage starts from a copy of it.
2. **Copy the project.** In Finder, duplicate `projects/02-tap-to-place/TapToPlace` into this folder and rename the copy `StickyNotes`. Delete its `Library` and `iOSBuild` folders if they came along (Unity rebuilds them). Open it from Unity Hub with **Add → Add project from disk**.
3. In **Project Settings → Player**, change the **Product Name** to `StickyNotes` and the **Bundle Identifier** to a new one, or it will overwrite your Stage 2 app on the phone.
4. Everything is tested on the phone. Keep using `StatusLabel.Show(...)` from Stage 2.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| `async` / `await` in C# (waiting for something slow) | Task 1: creating an anchor | https://docs.unity3d.com/Manual/async-await-support.html |
| Events (`trackablesChanged`): "call my method when this happens" | Task 5 | https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/ |
| Your own data class marked `[Serializable]` | Task 4 | https://docs.unity3d.com/ScriptReference/JsonUtility.html |
| Turning data into JSON text and back | Task 4, 5 | Same page |
| Reading and writing files | Task 4, 5 | https://learn.microsoft.com/en-us/dotnet/api/system.io.file |
| Where an app may save files | Task 4 | https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html |
| `Dictionary<K, V>` (look something up by a key) | Task 5 | https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2 |
| Text input on the phone | Task 3 | https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/TextMeshPro/index.html |

`async`/`await` is the same idea you met in Stage 0 with `await mindar.start()`: "start this, and carry on with the next line only when it has finished".

## Files you will add

```
03-sticky-notes/
├── README.md
├── GUIDE.md
└── StickyNotes/                     (Unity project, copied from Stage 2)
    └── Assets/
        ├── Prefabs/
        │   └── Note.prefab          (task 2)
        └── Scripts/
            ├── StatusLabel.cs       (from Stage 2)
            ├── NotePlacer.cs        (tasks 1–2: replaces TapToPlace)
            ├── Note.cs              (tasks 2–3: lives on each note)
            ├── NoteData.cs          (task 4: plain data, no MonoBehaviour)
            └── NoteStore.cs         (tasks 4–5: save and load)
```

## Task by task

### Task 1 — See drift, then fix it

**Plain English:** on a tap, place two cubes side by side. One is placed the Stage 2 way. The other is attached to an anchor. Then walk away and come back.

**Setup:** select **XR Origin** → **Add Component → AR Anchor Manager**. Manual: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/anchors/aranchormanager.html

**Script:** rename your `TapToPlace.cs` to `NotePlacer.cs` (rename it in Unity's Project panel, then change the class name inside to match). Add a slot for the anchor manager and drag **XR Origin** into it.

```csharp
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;
using Touch = UnityEngine.InputSystem.EnhancedTouch.Touch;

public class NotePlacer : MonoBehaviour
{
    [SerializeField] ARRaycastManager raycastManager;
    [SerializeField] ARAnchorManager anchorManager;
    [SerializeField] GameObject notePrefab;

    readonly List<ARRaycastHit> hits = new List<ARRaycastHit>();

    // OnEnable, OnDisable and the touch checks in Update are the same as Stage 2.

    void Update()
    {
        // ... same touch and raycast steps as Stage 2 ...
        // When there is a hit, call PlaceAt(hits[0]).
    }

    async void PlaceAt(ARRaycastHit hit)
    {
        // 1. Ask the anchor manager for an anchor at the hit's pose. This takes time: await it.
        // 2. Check whether it succeeded. If not, show a message and return.
        // 3. Get the anchor out of the result.
        // 4. Create the content and make it a CHILD of the anchor.
    }
}
```

**What each step looks like:**

- **Step 1** is one line: `var result = await anchorManager.TryAddAnchorAsync(hit.pose);`. The method that contains an `await` must be marked `async`, which is why `PlaceAt` is.
- **Step 2:** `result.status.IsSuccess()` is `true` or `false`.
- **Step 3:** `result.value` is the `ARAnchor`. An anchor is a component on its own GameObject, so it has a `transform`.
- **Step 4:** `Instantiate` has a form that takes a prefab and a parent Transform. A child moves with its parent; that is the whole trick. When the phone refines its map, it moves the anchor, and your content rides along.

**The "attached to a plane" variant** the brief asks about: if what you hit was a plane, you can ask for an anchor fixed to that plane. `hit.trackable is ARPlane plane` tests it and gives you `plane`; `anchorManager.AttachAnchor(plane, hit.pose)` returns the anchor directly, with no `await`. Try both and note the difference when the plane itself shifts.

For the experiment, also place a second cube 10 cm to the side the old way (plain `Instantiate`, no parent). Give the two different colours.

**Check:** build. Place the pair. Walk to another room, turn around a few times, come back. Compare the two cubes. Cover the camera for a few seconds and try again. Write what you see in the log.

**If it fails:**

| What you see | Likely cause |
|---|---|
| "Add anchor failed" every time | No AR Anchor Manager on XR Origin, or the slot is empty |
| Content appears at the world origin | You instantiated without the parent, or passed a world position *and* a parent |
| Compile error on `await` | The method is not marked `async` |
| No visible difference between the cubes | Common in a small, well-lit room. Try a longer walk, or a bigger space |

Commit.

### Task 2 — Notes on anchors

**Plain English:** replace the cube with a flat coloured square that lies against whatever surface it was placed on.

**Make the prefab (no code):**

1. Create an empty GameObject named `Note`.
2. Add a child: **3D Object → Quad**. Scale it to about `0.1, 0.1, 1` (a 10 cm square). Give it a material with a sticky-note colour.
3. A Quad stands upright facing along Z. A hit pose on a plane has its **Y axis pointing out of the surface**. So the quad must be turned to lie flat in the parent's X–Z plane. Rotate the *child* 90° around X and lift it a millimetre (`0.001` on Y) so it does not flicker against the wall.
4. Add another child for the text: **3D Object → Text - TextMeshPro** (the 3D one, not the UI one). Lay it flat the same way, a little above the quad, and shrink its font size until it fits.
5. Add a **Box Collider** to the quad so taps can hit it.
6. Create `Assets/Scripts/Note.cs`, attach it to the `Note` root, then drag `Note` into `Assets/Prefabs` and delete it from the Hierarchy.

```csharp
using TMPro;
using UnityEngine;

public class Note : MonoBehaviour
{
    [SerializeField] TMP_Text label;          // drag the text child here
    [SerializeField] Renderer background;     // drag the quad here

    public string Text { get; private set; } = "";
    public Color Colour { get; private set; } = Color.yellow;

    public void SetText(string text)
    {
        // 1. Remember the text.
        // 2. Show it on the label.
    }

    public void SetColour(Color colour)
    {
        // 3. Remember the colour.
        // 4. Apply it to the background's material.
    }
}
```

Open the prefab (double-click it) and drag the text child and the quad into the two slots.

**Check:** build. A note on the desk lies flat. A note on the wall lies flat against the wall. If one of them stands on edge, the child's rotation in step 3 is wrong; fix it in the prefab, not in code. The brief's question, "where does the rotation come from?", is answered by looking at `hit.pose.rotation` for a floor hit versus a wall hit.

Commit.

### Task 3 — Editable text

**Plain English:** tap a note to select it. A text box and a few buttons appear at the bottom of the screen. Typing changes the selected note.

Doing the typing in screen-space UI is far easier than typing "into" a 3D object.

**UI (no code):** on the **Canvas**, add a panel at the bottom containing:

- **UI → Input Field - TextMeshPro**
- three small **Buttons** for colours
- a **Delete** button and a **Done** button

Untick the panel's checkbox at the top of the Inspector so it starts hidden.

**Script:** create `Assets/Scripts/NoteEditor.cs` on the **Canvas**.

```csharp
using TMPro;
using UnityEngine;

public class NoteEditor : MonoBehaviour
{
    [SerializeField] GameObject panel;
    [SerializeField] TMP_InputField input;

    Note current;

    public bool IsOpen => current != null;

    public void Open(Note note)
    {
        // 1. Remember the note.
        // 2. Show the panel.
        // 3. Put the note's existing text into the input field.
    }

    // Wire the input field's "On Value Changed" event to this.
    public void OnTextChanged(string text)
    {
        // 4. If a note is open, set its text.
    }

    // One of these per colour button, or one method the buttons call with different values.
    public void SetYellow() { /* 5. */ }

    public void DeleteCurrent()
    {
        // 6. Destroy the note's ANCHOR object (the note's parent), not just the note. Then close.
    }

    public void Close()
    {
        // 7. Forget the note and hide the panel.
    }
}
```

**Wiring:** select the Input Field; under **On Value Changed (String)** press **+**, drag in the **Canvas**, and pick `NoteEditor → OnTextChanged` from the **Dynamic string** section at the top of the dropdown. Buttons are wired through **On Click ()** as in Stage 2.

**Selecting a note:** in `NotePlacer.Update`, before placing, do the physics raycast you wrote for `ObjectMover` in Stage 2. If it hits something with a `Note` component (look up `GetComponentInParent<Note>()`), call `editor.Open(thatNote)` and stop. Also ignore all taps while the editor is open or the touch is over UI.

Step 6 is why deleting matters: removing only the note leaves an invisible anchor behind that the phone keeps tracking, and that still gets saved in task 4. The manual describes `anchorManager.TryRemoveAnchor(anchor)` for removing one cleanly.

**Check:** build. Place a note, tap it, type. The keyboard appears and the note updates as you type. Change its colour. Delete it. Place another.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Tapping a note places a new one on top | The physics check is not running before the placement code |
| Keyboard covers the input field | Move the panel to the top of the screen |
| Typing does nothing | You wired the static `OnTextChanged` entry, not the dynamic one |
| Text is mirrored or upside down | The text child's rotation in the prefab |

Commit.

### Task 4 — Save

**Plain English:** when the Save button is pressed, write two files. One is the phone's map of the room. The other is a list of your notes: which anchor each belongs to, its text and its colour.

ARKit does not support AR Foundation's built-in "save anchor" feature, so on iPhone this goes through ARKit's **world map**. The API is in the ARKit package:

- `ARKitSessionSubsystem`: https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/api/UnityEngine.XR.ARKit.ARKitSessionSubsystem.html
- `ARWorldMap`: https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/api/UnityEngine.XR.ARKit.ARWorldMap.html
- A working example to read *after* you have tried: `Assets/Scripts/Runtime/ARWorldMapController.cs` in https://github.com/Unity-Technologies/arfoundation-samples

**The data.** `Assets/Scripts/NoteData.cs`. These are plain classes, not MonoBehaviours, and never get attached to anything.

```csharp
using System;
using System.Collections.Generic;

[Serializable]
public class NoteData
{
    public string anchorId;     // which anchor this note belongs to
    public string text;
    public float r, g, b;       // colour
}

[Serializable]
public class NoteFile
{
    public List<NoteData> notes = new List<NoteData>();
}
```

`JsonUtility` can only save a class, not a bare list, which is why `NoteFile` wraps the list.

**The store.** `Assets/Scripts/NoteStore.cs` on **XR Origin**, with slots for the `ARSession` (drag the **AR Session** object) and the `ARAnchorManager`.

```csharp
using System.IO;
using Unity.Collections;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARKit;

public class NoteStore : MonoBehaviour
{
    [SerializeField] ARSession session;
    [SerializeField] ARAnchorManager anchorManager;
    [SerializeField] GameObject notePrefab;

    string MapPath   => Path.Combine(Application.persistentDataPath, "room.worldmap");
    string NotesPath => Path.Combine(Application.persistentDataPath, "notes.json");

    public void Save()
    {
        // 1. Get the ARKit session subsystem from `session`. If it is not ARKit, show a message and return.
        // 2. Read its world mapping status. If it is not good enough yet, tell the user to look around more and return.
        // 3. Ask for the world map (asynchronous; use the callback form).
        // 4. In the callback: serialize the map to bytes and write them to MapPath.
        // 5. Dispose the map and the byte array when done.
        // 6. Build a NoteFile: loop over every Note in the scene, and for each one
        //    record its anchor's id, its text and its colour.
        // 7. Turn the NoteFile into JSON text and write it to NotesPath.
    }
}
```

**What each step looks like:**

- **Step 1:** `session.subsystem` is the generic session. The ARKit one is reached with a cast: `session.subsystem as ARKitSessionSubsystem`. `as` gives `null` if it is not that type, so check for `null`.
- **Step 2:** the subsystem has a `worldMappingStatus`. Print it to the status label every second while you walk around and watch it change. That answers the brief's question. Decide which value is your threshold.
- **Step 3:** `GetARWorldMapAsync` has a form that takes a callback with two inputs, a status and the `ARWorldMap`. This is the same "call me when it is ready" pattern as Stage 0's `onTargetFound`.
- **Step 4:** `worldMap.Serialize(Allocator.Temp)` returns a `NativeArray<byte>`. `File.WriteAllBytes` wants a normal `byte[]`; a `NativeArray` has a `ToArray()` method.
- **Step 6:** each `Note` is a child of its anchor, so `note.GetComponentInParent<ARAnchor>()` finds the anchor, and an anchor has a `trackableId`. Call `.ToString()` on it to get text you can save. To find every note: `FindObjectsByType<Note>(FindObjectsSortMode.None)`.
- **Step 7:** `JsonUtility.ToJson(file, true)` then `File.WriteAllText`.

Add a **Save** button wired to `NoteStore.Save`.

**Check:** build with Xcode attached. Place three notes, walk around until the mapping status is good, press Save. The label reports success. To look at the files: in Xcode, **Window → Devices and Simulators** → your phone → select the app → gear icon → **Download Container**, then right-click the download → **Show Package Contents → AppData/Documents**. Open `notes.json` in VS Code and check it has three entries.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Subsystem is `null` | The **Session** slot is empty, or you are in the editor and not on the phone |
| Status never improves | Keep moving slowly around the room, pointing at textured surfaces |
| `notes.json` is `{}` | You tried to serialize a bare list, or the fields are not `public` |
| `anchorId` is empty or all zeros | The note is not a child of its anchor |
| Crash or leak warning about a NativeArray | Step 5: call `Dispose()` on both |

Commit.

### Task 5 — Reload

**Plain English:** on launch, if a saved map exists, hand it to ARKit. ARKit then tries to recognise the room. When it does, the saved anchors reappear. For each one that appears, look up its note data and rebuild the note on it.

Add to `NoteStore`:

```csharp
    // anchorId -> the saved data for that note
    readonly Dictionary<string, NoteData> pending = new Dictionary<string, NoteData>();

    void OnEnable()
    {
        // 1. Subscribe OnAnchorsChanged to the anchor manager's trackablesChanged event.
    }

    void OnDisable()
    {
        // 2. Unsubscribe.
    }

    void Start()
    {
        // 3. If MapPath does not exist, show "No saved room" and return.
        // 4. Read notes.json, turn it back into a NoteFile, and put every NoteData into `pending`, keyed by anchorId.
        // 5. Read the map bytes, deserialize them into an ARWorldMap, and apply it to the ARKit session.
        // 6. Show "Look around the room…".
    }

    void OnAnchorsChanged(ARTrackablesChangedEventArgs<ARAnchor> changes)
    {
        // 7. For each anchor in changes.added:
        //    a. Turn its trackableId into text.
        //    b. If `pending` has data for that id, create a note as a child of the anchor,
        //       set its text and colour, and remove the entry from `pending`.
        // 8. When `pending` is empty, show "All notes restored".
    }
```

You will need `using System.Collections.Generic;` at the top.

**What each step looks like:**

- **Step 1:** `anchorManager.trackablesChanged.AddListener(OnAnchorsChanged);`. Step 2 is the same with `RemoveListener`.
- **Step 4:** `File.ReadAllText` then `JsonUtility.FromJson<NoteFile>(text)`.
- **Step 5:** `File.ReadAllBytes` gives a `byte[]`. `ARWorldMap.TryDeserialize` wants a `NativeArray<byte>`; you can make one with `new NativeArray<byte>(bytes, Allocator.Temp)`. Then `subsystem.ApplyWorldMap(worldMap)`.
- **Step 7:** the event gives you `added`, `updated` and `removed` lists. Here you only need `added`.

**Does the anchor come back with the same id it was saved with?** The whole matching scheme depends on it. The ARKit docs say applying a world map "adds its stored planes and anchors", but do not spell out the ids. Test it first: save, relaunch, and print every added anchor's id next to the ids in `notes.json`. If they match, carry on. If they do not, ask; the fallback is to match by saved position.

**The two "what if" questions in the brief:**

- *While the phone is working out where it is*, the session's tracking is limited. Show a message, and do not let the user place new notes yet.
- *If it never recognises the room* (wrong room, lights off), `pending` never empties. Add a timeout: after, say, 20 seconds, offer a "Start fresh" button that clears the saved files.

**Check:**

- [ ] Place and save three notes. Force-quit the app (swipe it away). Relaunch in the same spot. All three return with their text and colour.
- [ ] Relaunch in a different room. No crash, no notes scattered about, and a sensible message.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Nothing comes back | The map was saved with poor mapping status, or the lighting changed a lot. Save again with more scanning |
| Anchors come back but no notes | Ids did not match (see above), or `pending` was filled *after* the map was applied |
| Notes appear twice | You also recreate notes somewhere else, or `OnAnchorsChanged` is subscribed twice |
| New notes placed after reload are not saved | Works as written; press Save again |

Commit.

## Quests: how to start

Each quest is a new scene in the `StickyNotes` project, reusing `NotePlacer`, `Note` and `NoteStore`.

### Memory palace

- Add to `NoteData`: `back` (the answer text) and `order` (an `int`). Add matching fields to `Note`, plus a `bool showingBack` and a `Flip()` method that swaps which text is shown.
- Tap behaviour changes: a tap on a note flips it; a long press opens the editor. A long press is "the touch has been down for more than half a second": keep the time the touch began and compare with `Time.time`.
- The route: a `LineRenderer` whose points are the notes' positions sorted by `order`.
- Build order: flip works → order numbers show on the notes → line between them → review mode → save and reload still work with the new fields.

### Formula graffiti

- A bigger note prefab with no background, scaled for reading from across the room. Work out the size by trying it.
- TextMeshPro supports rich-text tags such as `<sup>` and `<sub>`: https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/TextMeshPro/RichText.html. Find where it runs out (fractions, integrals) and decide your fallback; an image of the formula on a quad is a fair one.
- "This week" sets: keep the anchors, and store the formulas in a separate JSON file keyed by anchor id that you can swap.

### Scale walks

- One anchor at the start. A script holding a list of names and real distances, and one `float` scale factor.
- In `Start`, loop through the list and place a labelled marker at `startPosition + direction * (distance * scale)`.
- Do the arithmetic on paper first: how many metres of hallway do you have, and what scale makes the farthest item fit?

### Quiz hunt

- `NoteData` gains `question`, `answer`, `clue` and `order`. One `int currentStop` says where the player is; save it with `PlayerPrefs.SetInt`.
- Each note shows itself only when its `order` equals `currentStop` (`gameObject.SetActive`: https://docs.unity3d.com/ScriptReference/GameObject.SetActive.html).

### Focus pet

- One anchor on the desk. `FocusPet.cs` with a timer (`float secondsStudied`, increased by `Time.deltaTime` in `Update`) and a few child models switched on and off as it grows.
- Unity tells you when the app goes to the background through `OnApplicationPause`: https://docs.unity3d.com/ScriptReference/MonoBehaviour.OnApplicationPause.html

### Ghost replay

- `PathRecorder.cs`: while recording, every 0.1 seconds add the camera's position and the current time to two lists.
- `GhostPlayer.cs`: find the two samples either side of the current playback time and blend between them with `Vector3.Lerp`.

### Miniature weather

- One anchor, a cloud model, and a **Particle System** for rain: https://docs.unity3d.com/Manual/ParticleSystems.html
- Start with three buttons that fake sun, cloud and rain. Only then look at real data, starting from [docs/apple-services.md](../../docs/apple-services.md).

### Geocache notes

- Read the Apple page linked in the brief first and check your area is supported before writing anything. This one is research-heavy; ask for help planning it.

## Committing your work

After each task, from the repo root:

```bash
git add -A
```

```bash
git commit -m "Stage 3: task 4, notes and world map save"
```

```bash
git push
```

Saved maps and `notes.json` live on the phone, not in the repo, so there is nothing extra to ignore.
