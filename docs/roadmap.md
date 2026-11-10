# Roadmap

New to coding? Read [coding-basics.md](coding-basics.md) first.

Thirteen stages on iPhone, then a port to a headset. Each stage has a **core build** that teaches one concept, plus **study quests** and **play quests** that reuse it. The full brief for each stage is in its `projects/` folder; dates are in [schedule.md](schedule.md).

## The stages at a glance

| # | Stage | Concept | Core build | Done when |
|---|---|---|---|---|
| 0 | [First AR](../projects/00-web-image-tracking/README.md) | Image targets, pose, coordinate frames | `00-web-image-tracking` | A model sits on your own printed card in Safari on your iPhone, and you can explain why it stays attached |
| 1 | [Foundations](../projects/01-hello-phone/README.md) | C#, Unity editor, Git, iOS builds | `01-hello-phone` | A Unity scene you scripted runs on your iPhone, committed with LFS working |
| 2 | [World tracking](../projects/02-tap-to-place/README.md) | SLAM, planes, raycasts | `02-tap-to-place` | You can place, move and delete objects on real surfaces |
| 3 | [Anchors](../projects/03-sticky-notes/README.md) | Anchors, drift, persistence | `03-sticky-notes` | Notes are where you left them after you force-quit and relaunch |
| 4 | [Image tracking](../projects/04-card-summoner/README.md) | Reference libraries, multi-image tracking | `04-card-summoner` | Three or more cards each show their own creature at once |
| 5 | [Study buddy](../projects/05-study-buddy/README.md) | App architecture, synced data | `05-study-buddy` | One app with a subject menu and at least three study quests you actually use |
| 6 | [Depth](../projects/06-hide-and-seek/README.md) | Occlusion, meshing, light estimation | `06-hide-and-seek` | A virtual object is correctly hidden behind a real chair and a real person |
| 7 | [Faces and hands](../projects/07-face-mask/README.md) | Face tracking, hand pose, audio input | `07-face-mask` | A mask follows your face and reacts to one expression |
| 8 | [Seeing and reading](../projects/08-text-lens/README.md) | Vision, Core ML | `08-text-lens` | Printed text is recognised and a label appears on it in AR |
| 9 | [Card battler](../projects/09-card-battler/README.md) | Game state, multi-user AR | `09-card-battler` | Two cards play a full match to a win or loss |
| 10 | [Idea board](../projects/10-idea-board/README.md) | Drawing, gestures, documents, serialisation | `10-idea-board` | You plan a real assignment on it and it reloads the next day |
| 11 | [Personal HUD](../projects/11-pocket-hud/README.md) | UI locking modes, your own API | `11-pocket-hud` | Your real to-dos and timers show in AR from your own server |
| 12 | [Assistant](../projects/12-assistant/README.md) | Speech, LLM tool use | `12-assistant` | You ask out loud for a timer or a board note and it happens |
| 13 | Headset port | Hand input, passthrough | — | The board, HUD and assistant run on the headset |

## How the concepts build on each other

```
0 image targets ──────────────► 4 image tracking ──► 9 card battler
1 foundations ─► 2 planes ─► 3 anchors ─┬─► 5 study buddy ◄── 8 vision
                              │         ├─► 10 idea board ─┐
                              └─► 6 depth                  ├─► 13 headset
                    7 faces/hands       └─► 11 HUD ─► 12 assistant ─┘
```

Stages 6 and 7 are independent of the capstones, so they can move if the schedule slips.

## How to work a stage

1. **Read** the "Read first" links in the brief (Tuesday or Thursday evening).
2. **Open the stage's `GUIDE.md`.** It lists the new coding ideas to learn first, the files to create, and how to start and check each task.
3. **Build** the core build, working through the tasks in order. The brief says what to build, the guide says how to go about it, and the code is yours to write.
4. **Check** yourself against "Done when" and answer the "Check your understanding" questions in your own words in the log.
5. **Quest**: do the quests the schedule names. Leave the rest as backlog.
6. **Commit** and tick the box in the root README.

## Getting help without getting the answer

When you are stuck, ask in this order, so you still do the thinking:

1. "Explain the concept behind task N again, differently."
2. "Give me a hint for task N" (a nudge, no code).
3. "Review my code for task N" (paste it; get feedback, not a rewrite).
4. "Show me a minimal example of X" (only the one API call you are missing).

## Backlog

Quests not on the schedule, for any week you finish early. They are described in the stage briefs.

- Stage 0: Holo stat sheet
- Stage 1: Constellation finder
- Stage 2: Physics sandbox, Measuring tape, AR graffiti, Desk tower defence
- Stage 3: Formula graffiti, Scale walks, Quiz hunt, Focus pet, Ghost replay, Miniature weather, Geocache notes
- Stage 4: Molecule builder, Card extras, Tabletop fog of war, Instant card (after Stage 5; it needs a TestFlight build)
- Stage 5: iCloud sync, Study reminders, Siri shortcuts
- Stage 6: Marble run, Room scanner
- Stage 7: Finger quiz, Invisible instruments, Audio visualiser
- Stage 8: Vocabulary labels, Deck scanner
- Stage 10: milestone 6 and all quests
- Stage 11: Lock-screen twin
- Stage 12: Theme my board, Presence
