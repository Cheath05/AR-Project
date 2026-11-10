# Stage 4 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which assets and scripts to create, which method each piece goes in, and how to tell each step worked. It does not contain the finished code.

## Before you write anything

1. Finish Stage 2 at least. Stage 3's events (`trackablesChanged`) and dictionaries are used again here; if you skipped Stage 3, read its "New coding ideas" table.
2. Draw and print three cards. Each needs to look clearly different from the others, with plenty of detail and contrast. Measure the printed width with a ruler, in centimetres.
3. Keep photos of real trading cards in a folder named `private/`, which Git ignores.
4. Everything is tested on the phone, with `StatusLabel.Show(...)` for messages.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| Events with added / updated / removed lists | Task 2 | https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/image-tracking.html |
| `foreach` over a list of results | Task 2 | https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/branches-and-loops |
| `enum` (a fixed set of named values, such as tracking states) | Task 2, 6 | https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/api/UnityEngine.XR.ARFoundation.ARTrackedImage.html |
| ScriptableObject (data saved as an asset, edited in the Inspector) | Task 3 | https://docs.unity3d.com/Manual/class-ScriptableObject.html |
| `Dictionary<K, V>` (look something up by name) | Task 3, 4 | https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2 |
| `SetActive` (show or hide a GameObject) | Task 6 | https://docs.unity3d.com/ScriptReference/GameObject.SetActive.html |

## Files you will add

```
04-card-summoner/
├── README.md
├── GUIDE.md
├── private/                             (git-ignored: photos of real cards)
└── CardSummoner/                        (Unity project)
    └── Assets/
        ├── Cards/
        │   ├── Images/                  (your card pictures)
        │   ├── CardLibrary.asset        (task 1: reference image library)
        │   ├── Flame.asset              (task 3: one CardDefinition per card)
        │   ├── Leaf.asset
        │   └── Wave.asset
        ├── Models/                      (task 4)
        ├── Prefabs/
        │   └── (one creature prefab per card)
        └── Scripts/
            ├── StatusLabel.cs           (copy from Stage 2)
            ├── CardDefinition.cs        (task 3: data, a ScriptableObject)
            └── CardSummoner.cs          (tasks 2–6)
```

## Task by task

### Task 1 — Build a reference image library

No code.

**Project:** new **Universal 3D** project named `CardSummoner` in this folder. Repeat the Stage 2 task 1 setup: AR Foundation and ARKit packages, ARKit ticked, camera usage description, a new bundle identifier, iOS platform, **AR Session** and **XR Origin (Mobile AR)** in the scene, default Main Camera deleted, status label added.

**Library** (manual: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/image-tracking.html):

1. Create `Assets/Cards/Images` and drag your three card pictures in from Finder.
2. Top menu: **Assets → Create → XR → Reference Image Library**. Name it `CardLibrary` and move it to `Assets/Cards`.
3. Select `CardLibrary`. In the Inspector press **Add Image** three times. For each row:
   - Drag a card picture into the texture slot.
   - Set **Name** to something short with no spaces: `flame`, `leaf`, `wave`. Your code will use these exact names.
   - Tick **Specify Size** and enter the printed width **in metres**. A card 6.3 cm wide is `0.063`.
4. Select **XR Origin** → **Add Component → AR Tracked Image Manager**. Drag `CardLibrary` into its reference library slot. Leave its tracked image prefab slot empty: the manual says that field is not meant for your content.

**Check:** build. Nothing visible happens yet, and nothing crashes. If Xcode reports a problem with an image when building, it is usually too small, too plain, or missing its size.

Do the brief's experiment (doubling the size) after task 2, when there is something to see.

Commit.

### Task 2 — Track one card

**Plain English:** whenever the list of tracked images changes, look at what was added and what was updated, and show each image's name and tracking state. Put a small cube on the first card found.

**Script:** `Assets/Scripts/CardSummoner.cs`, attached to **XR Origin**. Drag **XR Origin** into its **Image Manager** slot.

```csharp
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;

public class CardSummoner : MonoBehaviour
{
    [SerializeField] ARTrackedImageManager imageManager;

    void OnEnable()
    {
        // 1. Subscribe OnImagesChanged to the manager's trackablesChanged event.
    }

    void OnDisable()
    {
        // 2. Unsubscribe.
    }

    void OnImagesChanged(ARTrackablesChangedEventArgs<ARTrackedImage> changes)
    {
        // 3. For each image in changes.added: handle a newly seen card.
        // 4. For each image in changes.updated: handle a card whose position or state changed.
        // 5. For each entry in changes.removed: handle a card the tracker has forgotten.
    }

    void OnCardAdded(ARTrackedImage image)
    {
        // 6. Show the image's name.
        // 7. Create a small cube as a child of the image.
    }

    void OnCardUpdated(ARTrackedImage image)
    {
        // 8. Show the image's name and its tracking state.
    }
}
```

**What each step looks like:**

- **Step 1:** `imageManager.trackablesChanged.AddListener(OnImagesChanged);`, exactly as in the manual's example. Step 2 uses `RemoveListener`.
- **Steps 3–4:** two `foreach` loops, each calling the matching method.
- **Step 5:** `removed` is slightly different: each entry is a pair of an id and the image. Loop with `foreach (var pair in changes.removed)`; the image is `pair.Value`. Leave the body empty for now.
- **Step 6:** `image.referenceImage.name` is the name you typed into the library.
- **Step 7:** an `ARTrackedImage` is a component on a GameObject that AR Foundation moves to follow the card. So `image.transform` is the card's position and rotation in the room, and anything you make a child of it rides along. Use `GameObject.CreatePrimitive(PrimitiveType.Cube)`, set its parent, set its `localPosition` to zero and its `localScale` to about 2 cm.
- **Step 8:** `image.trackingState` is one of `None`, `Limited` or `Tracking`.

**Check:** build. Show the phone one card. The label shows its name and `Tracking`, and a small cube sits on the card. Slide the card: the cube follows. Cover the card with your hand and watch the state. Note what you see for each: covered, flipped over, taken out of view. You need those notes for task 6.

Now do the brief's experiment: double the size in the library, rebuild, and compare where the cube sits. Put it back afterwards.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Nothing ever detected | Library not assigned to the manager; the card is too plain or glossy; or the room is dim |
| `NullReferenceException` | **Image Manager** slot is empty |
| Cube is huge | You forgot to set `localScale`; a default cube is 1 metre |
| Cube hovers above or sinks below the card | The physical size in the library does not match the print |
| Detected once, then the label stops changing | You only handled `added`. Ongoing changes arrive in `updated` |

Commit.

### Task 3 — Tell the cards apart

**Plain English:** make a small data asset for each card that says "the image called `flame` gets *this* creature, at *this* scale, turned *this* much". Then the script looks up the right asset by name.

A **ScriptableObject** is a class whose instances are saved as files in your project and edited in the Inspector. It is how you keep data out of code, which is what the brief means by "adding a card does not mean editing code".

**Script:** `Assets/Scripts/CardDefinition.cs`. Do not attach this to anything.

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "NewCard", menuName = "Cards/Card Definition")]
public class CardDefinition : ScriptableObject
{
    public string imageName;        // must match the Name in the reference image library
    public GameObject creaturePrefab;
    public float scale = 1f;
    public Vector3 rotationOffset;  // degrees, to stand the model upright
}
```

That one is complete: it is pure data, with no behaviour to write.

**Create the assets (no code):** in the Project panel, right-click `Assets/Cards` → **Create → Cards → Card Definition**. That menu entry exists because of the `[CreateAssetMenu]` line. Make three, name them `Flame`, `Leaf`, `Wave`, and fill in each one's **Image Name**. Leave the prefab empty until task 4.

**Add to `CardSummoner`:**

```csharp
    [SerializeField] List<CardDefinition> cards;

    // image name -> its definition, built once at startup for quick lookup
    readonly Dictionary<string, CardDefinition> byName = new Dictionary<string, CardDefinition>();

    void Awake()
    {
        // 1. Loop over `cards` and add each one to `byName`, keyed by its imageName.
    }

    CardDefinition Find(ARTrackedImage image)
    {
        // 2. Look up image.referenceImage.name in `byName`.
        // 3. Return the definition, or null if there is none (and show a warning).
    }
```

Select **XR Origin**; the script now shows a **Cards** list. Set its size to 3 and drag the three assets in.

- **Step 2:** dictionaries have `TryGetValue(key, out value)`, which returns `true` or `false` and hands you the value. Look it up on the Dictionary page in the table above.

In `OnCardAdded`, call `Find(image)` and show which definition came back.

**Check:** build. Each card shows its own definition's name on the label. A typo in an **Image Name** shows your warning instead of crashing.

Commit.

### Task 4 — A creature per card

**Plain English:** when a card is added, create its creature as a child of the tracked image, with the scale and rotation from its definition. Remember which creature belongs to which card.

**Models and prefabs (no code):**

1. Download three free models (sources at the bottom of [docs/resources.md](../../docs/resources.md); check each licence) into `Assets/Models`.
2. For each: drag the model into the Hierarchy, then drag that object into `Assets/Prefabs` to make a prefab, and delete it from the Hierarchy.
3. Drag each prefab into its card definition's **Creature Prefab** slot.

**Add to `CardSummoner`:**

```csharp
    // the creature currently shown for each tracked image
    readonly Dictionary<ARTrackedImage, GameObject> creatures = new Dictionary<ARTrackedImage, GameObject>();
```

Then rewrite `OnCardAdded`:

```
    // 1. Find the definition. If there is none, return.
    // 2. Instantiate its prefab as a child of image.transform.
    // 3. Set the creature's localPosition to zero.
    // 4. Set its localRotation from the definition's rotationOffset.
    // 5. Set its localScale from the definition's scale.
    // 6. Store it in `creatures` under this image.
```

- **Step 4:** `Quaternion.Euler(x, y, z)` turns three angles in degrees into a rotation (https://docs.unity3d.com/ScriptReference/Quaternion.html).
- **Step 5:** `Vector3.one * scale`.

**Tuning.** Models come in every size and orientation. A tracked image's Y axis points up out of the card, so a model that is "Y-up" should stand correctly with no rotation; one that lies on its side needs 90° on X or Z. For each creature: guess, build, look, adjust the numbers **in the definition asset** (not in code), and build again. Use the card's physical size as a ruler: a 6 cm card with a 6 cm creature needs the model to end up about `0.06` units tall.

Faster tuning: drag the prefab into an empty scene next to a quad scaled to your card's size, adjust until it looks right in the editor, and copy the numbers across.

**Check:** build. Each card shows its own creature, upright and card-sized. This also answers the brief's "where do you store those?".

Commit.

### Task 5 — Several at once

**Plain English:** put all the cards on the table and find out how many the phone will follow at the same time.

Mostly a settings task.

1. Select **XR Origin** and look at the **AR Tracked Image Manager** in the Inspector for the setting that limits how many *moving* images are tracked. In code it is the `requestedMaxNumberOfMovingImages` property; the Inspector label is similar. Check the manual for the exact name.
2. Set it to the number of cards you have.
3. At runtime the phone may give you fewer than you asked for. In `Start`, print `imageManager.currentMaxNumberOfMovingImages` to the label, and print `imageManager.descriptor.supportsMovingImages`.

**Check:** build. Lay out all the cards. Count how many creatures follow their cards when you slide them one at a time. Slide two at once. Write the limit for your phone in the log.

If only one card updates at a time while the others freeze in place, the moving-images setting is still at its default.

Commit.

### Task 6 — Handle lost tracking

**Plain English:** every time a card is updated, look at its tracking state and decide whether its creature should be shown.

Go back to your notes from task 2. You probably saw that a covered card goes to `Limited` and stays in the `updated` list; it is not `removed`. That is why the creature seemed to freeze in mid-air.

Rewrite `OnCardUpdated`:

```
    // 1. Look up this image's creature in `creatures`. If there is none, return.
    // 2. If the state is Tracking: make sure the creature is shown.
    // 3. If the state is Limited or None: do whatever you decided (hide, linger, fade).
```

And fill in the `removed` loop from task 2:

```
    // 4. Look up the removed image's creature. Destroy it. Remove the entry from `creatures`.
```

**Build it in this order:**

1. **Vanish at once.** `creature.SetActive(state == TrackingState.Tracking)`. One line. Build and try all three situations (covered, flipped, slid away).
2. **Linger.** Vanishing at once flickers when tracking wobbles for a single frame. Keep a `Dictionary<ARTrackedImage, float>` of "time last seen tracking"; hide only when `Time.time` minus that is more than, say, half a second.
3. **Fade** (stretch). Shrink the creature's scale toward zero over the linger time.

**The brief's last question:** the card is out of the camera's view but the creature is still drawn where the card was. That is possible because the tracked image has a position in the *room* (world tracking), not just relative to the camera as in Stage 0. Is that what you want for a card game? Decide, and make the behaviour deliberate.

**Check:** all four boxes in the brief's "Done when". For the last box, prove it: draw a fourth card, add it to the library, create a fourth definition asset and a prefab, add it to the **Cards** list, build. You should not have opened a `.cs` file.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Creature frozen in mid-air after the card is taken away | You are not reacting to `Limited` in `updated` |
| Creature flickers | No linger time |
| Errors about a destroyed object | An entry was left in `creatures` after its creature was destroyed |
| Fourth card ignored | It is in the library but not in the **Cards** list, or the names differ |

Commit.

## Quests: how to start

Each quest is a new scene in the `CardSummoner` project. For an AR scene, duplicate `Main.unity` so the session, origin and label come with it.

### Textbook triggers

- **Markers:** small, bold, high-contrast and all different. A 4 cm square with a distinct shape and a chapter number works. Add them to a second reference image library, `MarkerLibrary`.
- **Data:** a new ScriptableObject, `PageContent`, modelled on `CardDefinition`: `imageName`, a `title`, a long `summary` string (tick the `[TextArea]` attribute above it so the Inspector gives you a big box), and an optional prefab for a model.
- **Display:** a world-space panel prefab with TextMeshPro text. Make it a child of the tracked image, then offset its `localPosition` sideways by a little more than the marker's width so it sits beside the page.
- **"Without rebuilding the app":** the stretch version loads summaries from a JSON file, as in Stage 3. Start with the ScriptableObject version.
- Build order: one marker shows a title → shows your real summary → second marker → second class.

### Molecule builder

- One card per element, each with a `CardDefinition` whose prefab is a coloured sphere with a label.
- New script `BondDrawer.cs`. In `Update`, loop over every pair of currently tracked creatures (`creatures.Values`); `Vector3.Distance` between them; if under a threshold, draw a `LineRenderer` between them.
- Looping over pairs needs two nested `for` loops where the inner one starts at `i + 1`. Work out on paper why.
- Rules ("hydrogen makes one bond") belong in data on the definition: add an `int maxBonds`.

### Card extras

- **Evolve:** in `Update`, check the distance between two specific tracked cards. When it drops under a few centimetres, swap one creature's prefab. Add an `evolvesInto` field (another `CardDefinition`) to the definition.
- **Damage counters:** each creature prefab gets a small script with an `int counters`, a method to add or remove one, and a world-space label. Tap detection is the physics raycast from Stage 2 task 5, so creature prefabs need colliders.
- **Idle life:** in a small script on the creature, every frame turn to face the nearest other creature with `transform.LookAt`, keeping it upright by passing the card's up direction.
- **AR dice:** needs an AR Plane Manager for the table, a die prefab with a **Rigidbody**, and a check of which face points up once it stops moving.

### Tabletop fog of war

- Start without fog: one big map image and one small token image, both tracked, with a marker drawn on the map under the token. Converting the token's world position into the map's local space is one call: `mapImage.transform.InverseTransformPoint(tokenPosition)`.
- Only then add fog. Ask for a hint on the texture-painting approach; it is a jump in difficulty.

### Instant card

- Read [docs/apple-services.md](../../docs/apple-services.md) first. Start with the simple version from the brief: a QR code linking to a `.usdz` file, which opens in AR Quick Look with no app. That needs no Unity at all.

## Committing your work

After each task, from the repo root:

```bash
git add -A
```

```bash
git commit -m "Stage 4: task 4, creature per card"
```

```bash
git push
```

Before each commit, run `git status` and check that no photos of real trading cards or textbook pages are listed.
