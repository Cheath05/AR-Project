# Stage 5 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, what kind of code, in what order, and how to tell each step worked. It does not contain the finished code.

This stage has no new AR. The new skill is organising code and data so several things can live in one app.

## Before you write anything

1. Reread your log entries for *Flip cards*, *Memory palace* and *Textbook triggers*. Write down three things that annoyed you. Those are your requirements.
2. Do task 1 (the paper sketch) before opening Unity. It takes 30 minutes and saves hours.
3. Create a new Unity project in this folder called `StudyBuddy`, set up for AR exactly as in [docs/setup.md](../../docs/setup.md). Build the empty project to your phone once, so you know the pipeline works before there is anything to debug.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| `List<T>`: a resizable list of things | Every data class | https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/list-collection |
| `Dictionary<K, V>`: look something up by a key | Finding a card's progress by its id | https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2 |
| Plain classes marked `[Serializable]` | Task 2 | https://docs.unity3d.com/Manual/script-Serialization.html |
| JSON, and turning objects into text and back | Tasks 2 and 5 | https://docs.unity3d.com/Manual/JSONSerialization.html |
| `enum`: a fixed list of named choices | Which mode is selected | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum |
| Canvas, buttons and layout groups | Task 3 | https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/UICanvas.html |
| Interfaces: a promise that a class has certain methods | Task 4 | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces |
| Reading and writing files | Task 5 | https://learn.microsoft.com/en-us/dotnet/api/system.io.file |
| Dates and times | Spaced repetition | https://learn.microsoft.com/en-us/dotnet/api/system.datetime |
| Web requests (server route only) | Task 5 | https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html |

## The one big idea: data, logic, display

Until now each script did everything. Here you split into three kinds of file:

- **Data classes** describe things: a subject, a card. No `MonoBehaviour`, no Unity scene. They are just boxes of fields.
- **Managers** hold the data and the rules: load it, save it, say which subject is selected.
- **Views and modes** are the `MonoBehaviour` scripts that draw things and react to taps. They ask the managers for data.

If a data class ever needs `transform` or `GameObject`, something is in the wrong file.

## Files you will add

```
05-study-buddy/
├── sketch.jpg                         (task 1: photo of your paper sketch)
└── StudyBuddy/                        (the Unity project)
    └── Assets/
        ├── Data/
        │   └── cards.json             (task 2: your cards, written by hand)
        ├── Prefabs/
        │   └── SubjectButton.prefab   (task 3)
        └── Scripts/
            ├── Data/
            │   ├── Card.cs            (task 2, plain class)
            │   ├── Subject.cs         (task 2, plain class)
            │   ├── CardProgress.cs    (task 2, plain class)
            │   └── StudyData.cs       (task 2, plain class: the whole file's contents)
            ├── DataStore.cs           (tasks 2 and 5: loads and saves)
            ├── AppState.cs            (task 3: what is selected right now)
            ├── MenuController.cs      (task 3: builds the menu)
            ├── IStudyMode.cs          (task 4: the interface)
            ├── ModeManager.cs         (task 4: starts and stops modes)
            ├── Modes/
            │   ├── MemoryPalaceMode.cs
            │   ├── TextbookTriggersMode.cs
            │   └── (one Stage 2 quest)Mode.cs
            └── SyncClient.cs          (task 5: second half)
```

**Making a plain class file:** in the Project window, right-click the folder → **Create → Scripting** and pick the empty C# script option (not the MonoBehaviour one). If you only see the MonoBehaviour option, use it and delete `: MonoBehaviour`, `Start()` and `Update()`.

## Task by task

### Task 1 — Sketch on paper

No code. Draw a box for every screen and an arrow for every button. Label each box "2D" or "AR". Photograph it and save it as `sketch.jpg` in this folder.

**Check:** you can trace with your finger from launching the app to studying one card, and back to the menu.

### Task 2 — The data model

**Plain English:** describe a card, a subject and "how well I know this card" as classes. Then write real cards as a JSON file by hand and prove Unity can read it.

**Write the JSON first, by hand.** Make `Assets/Data/cards.json`. Its shape is one outer object holding a list of subjects, each holding a list of cards:

```
{
  "subjects": [
    {
      "id": "...",
      "name": "...",
      "cards": [
        { "id": "...", "question": "...", "answer": "..." }
      ]
    }
  ]
}
```

Add the fields *you* need. Does a card have a hint? A chapter? An image name for *Textbook triggers*?

**Then write classes that match it, field for field.** The shape of each data file:

```csharp
using System;
using System.Collections.Generic;

[Serializable]
public class Card
{
    public string id;
    // one public field per piece of data in your JSON card
}
```

`Subject.cs` has the same shape and includes a `public List<Card> cards;`. `StudyData.cs` is the outer object: it has one field, `public List<Subject> subjects;`.

Three rules of Unity's `JsonUtility` that catch everyone:

- The class must have `[Serializable]` above it, and fields must be `public`.
- Field names must match the JSON names exactly, including capital letters.
- It cannot read a bare list or a `Dictionary`. That is why `StudyData` exists: a wrapper object with a list inside.

**`CardProgress.cs`** records how well you know one card. Keep it separate from `Card` on purpose: cards are content you write, progress is something the app changes. Give it the card's id and whatever your scheduler will need (times seen, times correct, when it is next due). `JsonUtility` cannot store a `DateTime`, so store dates as a `string` or a `long`; look up `DateTime.ToString("o")` and `DateTime.Parse`.

**`DataStore.cs`** is a `MonoBehaviour` on an empty GameObject named `DataStore`:

```csharp
using System.Collections.Generic;
using UnityEngine;

public class DataStore : MonoBehaviour
{
    [SerializeField] private TextAsset cardsFile;   // drag cards.json here in the Inspector

    public StudyData Data { get; private set; }

    void Awake()
    {
        // 1. Turn cardsFile.text into a StudyData object with JsonUtility
        // 2. Store it in Data
        // 3. Debug.Log how many subjects and cards were loaded
    }

    public Subject FindSubject(string id)
    {
        // 1. Loop over Data.subjects
        // 2. Return the one whose id matches, or null
    }
}
```

**Check:** press Play in the editor (no phone needed). The Console prints the right number of subjects and cards.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Count is 0 or `Data` is null, no error | Field names do not match the JSON, or `[Serializable]` is missing |
| `ArgumentException: JSON parse error` | A missing comma, quote or bracket in `cards.json`. Paste it into an online JSON checker |
| `NullReferenceException` on `cardsFile` | You did not drag the file into the Inspector slot |
| The list of cards is null but subjects load | The inner class is missing `[Serializable]` |

Commit.

### Task 3 — Subject menu and the first mode

**Plain English:** show one button per subject, made from the data. Tapping one remembers the choice and starts *Memory palace* with only that subject's cards.

**Editor steps:**

1. **GameObject → UI → Canvas.** Under it add a **Panel** named `MenuPanel`.
2. Inside the panel add an empty object named `SubjectList` with a **Vertical Layout Group** component. Children line up automatically.
3. **GameObject → UI → Button - TextMeshPro.** Style it, drag it into `Assets/Prefabs/` to make `SubjectButton.prefab`, then delete it from the scene.

**`AppState.cs`** answers "what did the user choose?". Put it on the same GameObject as `DataStore`:

```csharp
using UnityEngine;

public class AppState : MonoBehaviour
{
    public Subject CurrentSubject { get; private set; }

    public void SelectSubject(Subject subject)
    {
        // 1. Store it
        // 2. Debug.Log its name so you can see it worked
    }
}
```

**`MenuController.cs`** goes on `MenuPanel`:

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class MenuController : MonoBehaviour
{
    [SerializeField] private DataStore dataStore;
    [SerializeField] private AppState appState;
    [SerializeField] private ModeManager modeManager;     // exists after task 4; leave empty for now
    [SerializeField] private Button subjectButtonPrefab;
    [SerializeField] private Transform listParent;        // drag SubjectList here

    void Start()
    {
        // 1. For each subject in dataStore.Data.subjects:
        // 2.   Instantiate the button prefab as a child of listParent
        // 3.   Set its label text to the subject's name
        // 4.   Tell the button what to do when clicked (see the note below)
    }

    void OnSubjectChosen(Subject subject)
    {
        // 1. Tell appState which subject was chosen
        // 2. Hide the menu panel
        // 3. Start the Memory palace mode
    }
}
```

Step 4 is the new piece. A button has `onClick.AddListener(...)`, which takes a function to run later. To pass *which* subject, you wrap the call in an arrow function, the same idea as the callbacks in Stage 0. There is a classic trap when doing this inside a loop; search "C# closure in loop" if every button opens the last subject.

**Bringing in Memory palace:** copy its scripts from your Stage 3 project into `Scripts/Modes/`. Wherever the old code had a hard-coded list of cards, replace it with a public method that is *given* a subject and uses that subject's cards. The mode should never decide which subject; it is told.

**Check:** two subjects in your JSON show as two buttons. Tapping one hides the menu and starts the palace with only its cards. Add a third subject to the JSON and a third button appears with no code change.

**If it fails:**

| What you see | Likely cause |
|---|---|
| No buttons | `listParent` or the prefab is not assigned; or `Start` ran before `DataStore.Awake` loaded anything |
| Every button opens the same subject | The closure-in-loop trap |
| Buttons overlap | No layout group on the parent |
| Taps do nothing on the phone | The scene has no **EventSystem** (created with the Canvas; do not delete it) |

Commit.

### Task 4 — Two more modes sharing one AR session

**Plain English:** every mode can be started and stopped the same way, and only one runs at a time.

**`IStudyMode.cs`** is the promise each mode makes:

```csharp
public interface IStudyMode
{
    string DisplayName { get; }
    void Begin(Subject subject);   // turn on what this mode needs, build its content
    void End();                    // remove everything it created, turn its managers off
}
```

Each mode script then declares `public class MemoryPalaceMode : MonoBehaviour, IStudyMode` and must contain those three members.

**`ModeManager.cs`**:

```csharp
using UnityEngine;

public class ModeManager : MonoBehaviour
{
    [SerializeField] private AppState appState;
    [SerializeField] private GameObject menuPanel;

    private IStudyMode current;

    public void StartMode(IStudyMode mode)
    {
        // 1. If a mode is already running, end it
        // 2. Remember the new one
        // 3. Call its Begin with the current subject
    }

    public void BackToMenu()
    {
        // 1. End the current mode, set current to null
        // 2. Show the menu panel
    }
}
```

**What goes in each mode's `Begin` and `End`:** make a list for each mode of (a) which AR managers it needs (plane manager, image manager, anchor manager) and (b) which objects it creates. `Begin` enables (a) and creates (b). `End` destroys (b) and disables (a). Managers are components, so they have an `enabled` setting. Keep one XR Origin and one AR Session for the whole app.

Add a **Back** button on a second, always-visible Canvas panel that calls `ModeManager.BackToMenu`.

**Check:** go menu → mode A → back → mode B → back → mode A, three times round. Nothing from the previous mode is left floating, and nothing slows down.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Old objects remain after leaving | `End` does not destroy everything `Begin` created. Keep created objects in a `List<GameObject>` and loop over it |
| Planes still show in an image-tracking mode | The plane manager was not disabled, or existing planes were not hidden |
| `NullReferenceException` in a copied script | It was looking for an object that existed in its old project. Find the line and pass that object in |
| Second visit to a mode shows double content | `Begin` adds without `End` having cleared |

Commit after each mode.

### Task 5 — Save, then sync

**Part 1, local save. Plain English:** write the progress list to a file on the phone; read it back at launch.

Add to `DataStore.cs`:

```csharp
    public void SaveProgress()
    {
        // 1. Turn the progress wrapper object into JSON text
        // 2. Build the file path: Application.persistentDataPath plus a file name
        // 3. Write the text to that path
    }

    void LoadProgress()
    {
        // 1. Build the same path
        // 2. If the file does not exist, start with an empty progress list and return
        // 3. Read the text and turn it back into objects
    }
```

You need a wrapper class for progress too (one field: a list of `CardProgress`), for the same reason as `StudyData`. Call `LoadProgress` from `Awake`. Call `SaveProgress` whenever progress changes and from Unity's `OnApplicationPause`, which runs when the app goes to the background (look it up in the `MonoBehaviour` reference).

**Check:** answer a card, force-quit the app from the app switcher, reopen. The progress is still there.

**Part 2, sync.** Pick one route and write your reason in the log.

- **Your own server (recommended first):** `SyncClient.cs` has two methods, one that sends your progress JSON with a web request and one that fetches it. Web requests take time, so they run in a *coroutine*; search the Unity manual for "coroutines" and read the `UnityWebRequest` examples. The server itself can be a few lines in any language; you will rebuild it properly in Stage 11.
- **CloudKit:** needs a Swift plugin. Read the [Stage 8 guide](../08-text-lens/GUIDE.md) section "How a native plugin is laid out" first; the file layout is identical.

Either way, decide the conflict rule before coding it: when the phone and the server disagree about one card, which wins? Giving each `CardProgress` a "last changed" time is the usual starting point.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Progress resets every launch | `LoadProgress` is not called, or load and save build different paths |
| Works in the editor, not on the phone | You wrote into `Assets/` and not `persistentDataPath` |
| Request fails with a security error | iOS blocks plain `http://`. Use `https://`, or research "App Transport Security" for local testing |
| Request never finishes | The method was called directly and not started with `StartCoroutine` |

Commit.

## Spaced-repetition room: how to start

1. Make `Scripts/Scheduler.cs` as a **plain static class**: no `MonoBehaviour`. One method, shaped like this:

   ```csharp
   public static class Scheduler
   {
       public static void RecordAnswer(CardProgress progress, int quality, System.DateTime now)
       {
           // 1. Update the counts
           // 2. Work out the new gap in days (start with: wrong = 1 day, right = double the last gap)
           // 3. Set the next due date to now plus the gap
       }

       public static bool IsDue(CardProgress progress, System.DateTime now)
       {
           // compare the stored due date with now
       }
   }
   ```

2. Test it with no AR: a temporary `MonoBehaviour` whose `Start` makes one `CardProgress`, calls `RecordAnswer` five times with made-up answers and fake dates, and prints the due date each time. Passing `now` in, not reading the clock inside, is what makes this testable.
3. Only then make `Modes/SpacedRepetitionMode.cs`. In `Begin`, loop over the subject's cards and give due ones a different look (scale, colour, distance). Start with one difference, such as colour.
4. Once the simple rule works, read about SM-2 and replace step 2 of `RecordAnswer`.

## Study reminders: how to start

1. **Window → Package Manager → Unity Registry**, install **Mobile Notifications**. Manual: https://docs.unity3d.com/Packages/com.unity.mobile.notifications@2.3/manual/index.html
2. Make `Scripts/ReminderScheduler.cs` with one method that (1) counts cards due tomorrow, (2) if zero, cancels any pending reminder and stops, (3) otherwise schedules one local notification for a time you choose. The manual's iOS page has the class and property names; follow its example.
3. Call it from the same places you call `SaveProgress`.
4. The app must ask permission once. The manual covers the request.

Test by scheduling one for a minute ahead and locking the phone.

## Siri shortcuts: how to start

Leave this until after Stage 8. It needs a Swift file inside the Xcode project, and the bridge from Swift back into Unity that Stage 8 teaches. The receiving end is small: a public method on a GameObject that takes a subject id as a string, selects it in `AppState` and starts a mode.

## Committing your work

```bash
git add -A
```

```bash
git commit -m "Stage 5: task 3, subject menu built from data"
```

```bash
git push
```

Your real course notes are yours to commit. Scanned textbook pages are not; keep those in a `private/` folder.
