# AR-Project

Learning how AR works by building it, one concept at a time, on the way to a personal AR assistant.

**Where this is heading**

1. Study tools that make revision something you walk around in.
2. A card game where the cards come to life.
3. A spatial whiteboard for treating ideas like physical things.
4. A personal HUD and a Jarvis-style assistant, first on iPhone, later on a headset.

**How it works**

- Everything is built on an iPhone first, with Unity 6 + AR Foundation (C#). The same skills carry to Quest and Android XR later.
- Each stage teaches one AR concept through a *core build*, then reuses it in *study quests* and *play quests*.
- The briefs in `projects/` describe problems to solve, not solutions. The code is mine to write.

## Start here

| Doc | What it is |
|---|---|
| [docs/schedule.md](docs/schedule.md) | What to do this week |
| [docs/roadmap.md](docs/roadmap.md) | Every stage, with "done when" criteria |
| [docs/setup.md](docs/setup.md) | Cloning the repo, choosing an editor, installing Unity and Xcode |
| [docs/resources.md](docs/resources.md) | Reading and tutorials, grouped by stage |
| [docs/apple-services.md](docs/apple-services.md) | What the Apple Developer account unlocks |
| [docs/hardware.md](docs/hardware.md) | Headsets and glasses, for later |
| [docs/log.md](docs/log.md) | Weekly progress log |

## Progress

- [ ] **Stage 0** — [First AR: image targets](projects/00-web-image-tracking/README.md)
- [ ] **Stage 1** — [Foundations](projects/01-hello-phone/README.md)
- [ ] **Stage 2** — [World tracking, planes, raycasts](projects/02-tap-to-place/README.md)
- [ ] **Stage 3** — [Anchors and persistence](projects/03-sticky-notes/README.md)
- [ ] **Stage 4** — [Image tracking in Unity](projects/04-card-summoner/README.md)
- [ ] **Stage 5** — [Capstone: study buddy](projects/05-study-buddy/README.md)
- [ ] **Stage 6** — [Depth, occlusion, meshing, lighting](projects/06-hide-and-seek/README.md)
- [ ] **Stage 7** — [Faces, hands and sound](projects/07-face-mask/README.md)
- [ ] **Stage 8** — [Seeing and reading](projects/08-text-lens/README.md)
- [ ] **Stage 9** — [Capstone: card battler](projects/09-card-battler/README.md)
- [ ] **Stage 10** — [Capstone: idea board](projects/10-idea-board/README.md)
- [ ] **Stage 11** — [Capstone: personal HUD](projects/11-pocket-hud/README.md)
- [ ] **Stage 12** — [Capstone: assistant](projects/12-assistant/README.md)
- [ ] **Stage 13** — Port to a headset (see [docs/hardware.md](docs/hardware.md))

## Repo layout

```
docs/        roadmap, schedule, setup, resources, log
projects/    one folder per stage: a brief (README.md) plus the code I write
```

Unity projects live inside their stage folder (for example `projects/02-tap-to-place/TapToPlace/`). The root `.gitignore` and `.gitattributes` are already set up for Unity and Git LFS.

## Ground rules

- No secrets in this repo. API keys live on a server or in untracked local files.
- No copyrighted card art. Test with my own drawings or cards I own, and keep photos of them out of the repo.
