# Stage 9 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, what kind of code, in what order, and how to tell each step worked. It gives you empty skeletons. Filling them in is the stage.

## Before you write anything

1. Copy your finished Stage 4 Unity project into this folder and rename it `CardBattler`. You are extending it, not starting again. Open the copy in Unity Hub with **Add → Add project from disk**.
2. Build the copy to your phone once, unchanged, to confirm it still tracks cards. If it does not, fix that before adding anything.
3. Do task 1 (the rules page) with pen and paper, away from the computer.

The big idea of this stage: **the game is plain C# that knows nothing about Unity scenes. The AR part only shows what the game says.** Tasks 1–2 involve no AR at all, and that is deliberate.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| `enum` (a fixed list of named choices) | Task 2, the match phase | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum |
| State machine (code that is always in exactly one phase and only allows certain moves between them) | Task 2 | Draw it on paper; an `enum` plus `if` checks is all it is |
| C# `event` (your own "when this happens, tell whoever is listening") | Tasks 2 and 5 | https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/ |
| Unit tests (code that checks your code) | Task 2 | https://docs.unity3d.com/Packages/com.unity.test-framework@1.4/manual/workflow-create-test.html |
| Assembly definitions (splitting a project into separate code libraries) | Task 2 | https://docs.unity3d.com/Manual/assembly-definition-files.html |
| ScriptableObject (data saved as an asset: one per card) | Task 3 | https://docs.unity3d.com/Manual/class-ScriptableObject.html |
| Coroutines (doing something over several frames) | Task 5 | https://docs.unity3d.com/Manual/Coroutines.html |

## Files you will add

```
09-card-battler/
├── RULES.md                      (task 1)
└── CardBattler/                  (Unity project, copied from Stage 4)
    └── Assets/
        ├── GameLogic/            plain C#: no "using UnityEngine" anywhere in here
        │   ├── GameLogic.asmdef
        │   ├── MatchPhase.cs
        │   ├── AttackData.cs
        │   ├── CreatureState.cs
        │   └── Match.cs
        ├── Tests/
        │   └── EditMode/
        │       ├── EditModeTests.asmdef
        │       └── MatchTests.cs
        ├── Cards/                one CardDefinition asset per card
        ├── Scripts/
        │   ├── CardDefinition.cs     (task 3)
        │   ├── MatchController.cs    (task 3)
        │   ├── AttackMenu.cs         (task 4)
        │   ├── CreatureView.cs       (task 5)
        │   └── HpBar.cs              (task 5)
        └── Prefabs/
```

Your Stage 4 scripts stay where they are.

## The work loop

Tasks 1–2: write code → **Window → General → Test Runner** → run tests → read which failed. No phone, no build. This loop takes seconds, which is the point.

Tasks 3–6: change → build → run on phone, as in earlier stages.

## Task by task

### Task 1 — Rules on paper

No code. Create `RULES.md` in this folder. Use these headings so nothing is forgotten:

```
# Rules
## What a card has        (name, max HP, attacks: each with a name and damage)
## Setup                  (who goes first?)
## A turn                 (what may the active player do? exactly one attack?)
## Ending                 (when is the match over? who wins?)
## Things that are not allowed
## What happens when a card disappears from view   (fill in during task 3)
```

**Check:** hand the page to someone else. Could they play the game with two pieces of paper and a pencil? If they have to ask you a question, the answer belongs on the page.

Commit.

### Task 2 — Game state in plain C#

**Plain English:** a `Match` object holds two creatures and whose turn it is. You call `match.Attack(...)`. It either changes HP and passes the turn, or refuses. Nothing here draws anything.

**Step A: make the GameLogic folder its own library.**

1. Create `Assets/GameLogic/`.
2. Inside it, create an Assembly Definition asset named `GameLogic` (right-click in the Project window → **Create**; check the manual page linked above for the exact menu path in your Unity version).
3. Select it and look in the Inspector for an option that removes engine references. Turn it on. Now any file in this folder that tries to use `UnityEngine` will fail to compile, which enforces the rule for you.

**Step B: the four files.** Each file holds one type. Skeletons:

`MatchPhase.cs`
```
public enum MatchPhase
{
    // list every phase a match can be in, e.g. waiting to start,
    // player one's turn, player two's turn, finished
}
```

`AttackData.cs`
```
public class AttackData
{
    // fields: a name, a damage number
    // a constructor that takes both
}
```

`CreatureState.cs`
```
public class CreatureState
{
    // fields: name, max HP, current HP, a list of AttackData
    // a property that answers "is this creature knocked out?"
    // a method that takes damage and never lets HP go below zero
}
```

`Match.cs`
```
using System;

public class Match
{
    // fields: the two creatures, the current MatchPhase

    // events the outside world can listen to. Declare, do not call yet:
    //   public event Action<...> AttackResolved;
    //   public event Action<...> TurnChanged;
    //   public event Action<...> MatchEnded;

    public Match(CreatureState first, CreatureState second)
    {
        // 1. store the creatures
        // 2. set the starting phase
    }

    public bool TryAttack(int playerIndex, int attackIndex)
    {
        // 1. is the match still running? if not, return false
        // 2. is it this player's turn? if not, return false
        // 3. is attackIndex a real attack? if not, return false
        // 4. apply the damage to the other creature
        // 5. raise AttackResolved
        // 6. is the target knocked out? then end the match and raise MatchEnded
        // 7. otherwise pass the turn and raise TurnChanged
        // 8. return true
    }

    public void Reset()
    {
        // used by the rematch in task 6
    }
}
```

Notice `TryAttack` returns `bool`: true if the move happened, false if it was refused. That is how the rest of the app finds out whether a move was legal.

**Step C: tests.**

1. Open **Window → General → Test Runner**. It offers to create a test assembly folder; put it at `Assets/Tests/EditMode/`.
2. Select the new `.asmdef` in that folder. In the Inspector, add `GameLogic` to its list of assembly references, or the tests cannot see your classes.
3. With the folder selected, **Assets → Create → Testing → C# Test Script**, named `MatchTests`.

A test is a method that sets something up, does one thing, and states what should now be true:

```
using NUnit.Framework;

public class MatchTests
{
    [Test]
    public void Attack_ReducesTargetHp()
    {
        // 1. make two creatures and a match
        // 2. call TryAttack once
        // 3. Assert.AreEqual(expectedHp, actualHp);
    }
}
```

Write the test **before** the code it tests. It will fail (red). Then fill in `TryAttack` until it passes (green). Tests to write, one at a time:

- an attack reduces the target's HP by the right amount
- the turn passes after an attack
- attacking out of turn returns false and changes nothing
- an attack index that does not exist returns false
- HP never goes below zero
- knocking out a creature ends the match with the right winner
- attacking after the match is over returns false
- a whole match played to the end (a loop of attacks)
- `Reset` puts everything back

**Check:** every test in the Test Runner has a green tick, and no file in `GameLogic/` contains the word `UnityEngine`.

**If it fails:**

| What you see | Likely cause |
|---|---|
| `The type or namespace 'Match' could not be found` in the test file | The test `.asmdef` does not reference `GameLogic` |
| `The type or namespace 'UnityEngine' could not be found` inside `GameLogic/` | Working as intended: remove the Unity dependency from that file |
| Test Runner shows no tests | The script is not inside the test assembly folder, or there is a compile error elsewhere (check the Console) |
| A test passes when you expected it to fail | The test has no `Assert` line, so it checks nothing |

Commit after each test goes green.

### Task 3 — Hook state to tracked cards

**Plain English:** each printed card needs to be linked to a creature's stats. When two known cards are visible, start a match.

**`CardDefinition.cs`** describes one card as data. It is a ScriptableObject, so each card becomes an asset file you fill in through the Inspector with no code:

```
using UnityEngine;
using System.Collections.Generic;

[CreateAssetMenu(menuName = "Card Battler/Card Definition")]
public class CardDefinition : ScriptableObject
{
    // public fields, filled in from the Inspector:
    //   the reference image name this card matches (a string)
    //   display name, max HP
    //   a list of attacks (name + damage). To show in the Inspector, the
    //   attack type needs [System.Serializable]. Decide: reuse AttackData,
    //   or make a small Unity-side copy and convert? Why might a copy be cleaner?
    //   the creature prefab to spawn

    public CreatureState CreateState()
    {
        // build a fresh CreatureState from these fields
    }
}
```

Then **right-click in `Assets/Cards/` → Create → Card Battler → Card Definition** once per card, and fill each in.

**`MatchController.cs`** is the one MonoBehaviour that owns the `Match`. Attach it to an empty GameObject in the scene.

```
using UnityEngine;
using System.Collections.Generic;

public class MatchController : MonoBehaviour
{
    [SerializeField] private List<CardDefinition> cards;   // drag your assets in
    // a reference to your Stage 4 tracking script or the tracked image manager

    public Match Match { get; private set; }

    private void OnEnable()
    {
        // subscribe to the same tracked-image event you used in Stage 4
    }

    private void OnDisable()
    {
        // unsubscribe
    }

    private void OnTrackedImagesChanged(/* same parameter as Stage 4 */)
    {
        // 1. for each added image: find the CardDefinition whose image name matches
        // 2. remember which cards are currently visible
        // 3. if exactly two known cards are visible and no match is running, start one
        // 4. for removed or lost images: follow the decision you wrote in RULES.md
    }

    private void StartMatch(CardDefinition a, CardDefinition b)
    {
        // 1. Match = new Match(a.CreateState(), b.CreateState());
        // 2. Debug.Log that a match started and who goes first
    }
}
```

Before coding step 4, fill in the last heading of `RULES.md`: one card visible, a third card appears, a card lost mid-match. Simplest rule that works: the match lives in `MatchController` and does not care whether cards are visible; losing a card only hides its creature.

**Check:** show two cards. The Xcode console (or an on-screen text) says a match started and names both creatures. Hide one and show it again: no second match starts.

### Task 4 — Attacks

**Plain English:** on your turn, buttons show your creature's attacks. Tapping one asks the match to do it.

`AttackMenu.cs`, on a Canvas:

```
using UnityEngine;

public class AttackMenu : MonoBehaviour
{
    [SerializeField] private MatchController controller;
    // references to your button objects and their labels

    private void OnEnable()
    {
        // listen to the match's TurnChanged and MatchEnded events
    }

    private void Refresh()
    {
        // 1. whose turn is it?
        // 2. set each button's label to that creature's attack names
        // 3. hide the menu if the match is over
    }

    public void OnAttackButton(int attackIndex)
    {
        // 1. call controller.Match.TryAttack(currentPlayer, attackIndex)
        // 2. if it returned false, show why (or just Debug.Log for now)
    }
}
```

Wire each button's **On Click** list in the Inspector to `OnAttackButton`, typing `0` or `1` as the argument.

Start with buttons fixed to the screen. Try world-space buttons beside the card afterwards and compare, as the brief asks.

**Check:** tap an attack. The log shows HP dropping and the turn changing. The buttons then show the *other* creature's attacks. Tapping fast twice does not give one player two moves.

### Task 5 — HP UI and animation

**Plain English:** the views listen to the match's events and react. They never change HP themselves.

`HpBar.cs` (on a world-space Canvas above each creature) and `CreatureView.cs` (on each creature prefab):

```
public class CreatureView : MonoBehaviour
{
    // which player index this view represents
    // a reference to its HpBar and Animator

    public void Bind(Match match, int playerIndex)
    {
        // 1. remember the index
        // 2. subscribe to match.AttackResolved
        // 3. set the HP bar to full
    }

    private void OnAttackResolved(/* your event's parameters */)
    {
        // if I was the attacker: play my attack animation
        // if I was the target: play my hit animation, update my HP bar
    }
}
```

For the animation itself, start with the cheapest thing that reads clearly: a coroutine that moves the creature forward and back over half a second. Look up `IEnumerator`, `yield return null` and `Vector3.Lerp` (https://docs.unity3d.com/ScriptReference/Vector3.Lerp.html). Swap in real animation clips later if you want.

The brief asks what breaks if a move arrives mid-animation. Simplest fix: `AttackMenu` disables its buttons while an animation plays and re-enables them when it ends. Decide who tells it.

**Check:** HP bars drop at the moment of the hit. Then delete the views from the scene and run the tests again: they still pass, because the game never depended on the views.

### Task 6 — Full match loop

1. Add a result panel that listens to `MatchEnded` and shows the winner with a **Rematch** button.
2. Rematch calls `Match.Reset()` (or starts a new `Match`) and every view rebinds.
3. Play five full matches. Keep a list of every odd thing in `docs/log.md` and fix them one at a time, adding a test for any that turn out to be rule bugs.

**If it fails:**

| What you see | Likely cause |
|---|---|
| After a rematch, each attack hits twice | Views subscribed to events again without unsubscribing from the old match |
| HP bars stay empty after rematch | Views were not told to rebind |
| A new match starts by itself mid-game | `OnTrackedImagesChanged` starts a match without checking one is already running |
| `NullReferenceException` in `AttackMenu` | It ran before a match existed; check for `null` first |

Commit.

## Shared table: how to start

Do this only after the core build is solid.

1. **Notice what you already have.** Both phones look at the same physical cards, and each creature sits on its card. So the phones already agree on *where* things are without sharing any map. You only need to share *state*.
2. **Pick one phone as host.** Only the host's `Match` is real. The other phone sends "I want to use attack 1" and waits to be told what happened.
3. **Define the messages on paper first.** Probably three: a move request, a move result, and a full snapshot for when a phone joins late. Write each as a small `[System.Serializable]` class and turn it into text with `JsonUtility`.
4. **Sending bytes between phones** needs Apple's Multipeer Connectivity (https://developer.apple.com/documentation/multipeerconnectivity), which is a Swift API. That means a native plugin like the one you wrote in Stage 8: one function to send a string, one callback when a string arrives. Get two phones exchanging the word "hello" before sending any game data.
5. Because your rules are plain C#, the guest phone can run the same `Match` class and apply the host's results to it. That is the payoff for task 2.
6. Ship to a friend with TestFlight ([docs/setup.md](../../docs/setup.md#6-testflight-from-stage-5)).

## Committing your work

```bash
git add -A
```

```bash
git commit -m "Stage 9: task 2, match rules pass tests"
```

```bash
git push
```

Do not commit card photos or published card art. Test images go in a `private/` folder, which Git ignores.
