# Stage 9 — Capstone: card battler

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity, AR Foundation, your iPhone, a second iPhone or iPad for the play quest · **Builds on:** Stage 4

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **Game state:** the single source of truth for a match (whose turn, each creature's HP, what is allowed next), kept apart from anything on screen.
- **Turn logic:** a state machine that moves a match from start to a win or loss and refuses illegal moves.
- **Animation driven by state:** visuals that react to changes in the state, never the other way round.
- **Multi-user AR:** how two devices agree on where things are, using a shared map and shared anchors.

The habit this stage builds, rules in plain code and AR as a view of them, is what makes stages 10–12 portable to a headset.

## Read first

- Your own Stage 4 code. You are extending it, not starting over.
- Image tracking (re-read "Tracked image lifecycle"): https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/image-tracking.html
- ScriptableObject, a tidy way to describe cards as data: https://docs.unity3d.com/Manual/class-ScriptableObject.html
- Animation in Unity: https://docs.unity3d.com/Manual/AnimationSection.html
- Unity Test Framework (for task 2): https://docs.unity3d.com/Packages/com.unity.test-framework@1.4/manual/index.html
- For the play quest, Apple on multi-user AR: https://developer.apple.com/documentation/arkit/creating-a-multiuser-ar-experience

## Core build: two cards fight

**Goal:** put two printed cards on the table. Their creatures appear, take turns attacking, lose HP, and one of them wins.

**Card art:** use your own drawings, or cards you own for personal testing only. Do not commit Pokémon artwork, or photos or scans of any published card, to this public repo. Put personal test images in a `private/` folder, which is git-ignored.

### Tasks

1. **Rules on paper.** Write the rules of your game on one page before touching Unity. What does a card have (HP, attacks, a type)? What can a player do on their turn? When does the match end? Keep it small enough to finish: two creatures, two attacks each is plenty. Commit the page as `RULES.md` in this folder.
2. **Game state in plain C#, with no AR.** Model a match in classes that know nothing about Unity scenes, cameras or tracked images. You should be able to play a whole match by calling methods and reading values. How will you prove it works without seeing anything? Write tests that play a match to a win, and tests that try illegal moves (attacking out of turn, attacking after the match is over).
3. **Hook state to tracked cards.** Connect your Stage 4 tracking to the match. Which tracked image is which creature? What should happen when only one card is visible, when a third appears, or when a card is lost mid-match? Decide and write the decisions in `RULES.md`.
4. **Attacks.** Give the player a way to choose an attack on their turn. Where do the buttons live: on the screen, or in the world next to the card? Try one, note what is awkward. The input must go through your game state, which decides whether the move is legal.
5. **HP UI and animation.** Show each creature's HP above it, and make attacks visible: the attacker moves, the target reacts, the HP bar drops. The visuals should respond to events from the game state. What breaks if an animation is still playing when the next move arrives?
6. **Full match loop.** Start, play, win or lose, rematch, all without restarting the app. Play five full matches and fix what goes wrong.

### Done when

- [ ] Two cards play a complete match to a win or loss on your iPhone.
- [ ] The rules run and pass their tests with no AR session at all.
- [ ] Losing sight of a card mid-match does not corrupt the match.
- [ ] A rematch works without relaunching.
- [ ] `RULES.md` matches what the game actually does.
- [ ] No copyrighted card art is in the repo.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Why keep the rules in classes that do not know about AR? Name two things that became easier because you did.
2. A creature's HP is shown on a bar and stored in the game state. Which one is "true"? What bug appears if both can be changed independently?
3. Draw your turn logic as a state diagram. Which transitions did you forget the first time?
4. Tracking is lost for a second and found again. What did your app do with the creature object, and why?
5. If a friend's phone joined, which parts of your code would need to be shared and which stay local?

Question 5 is the play quest.

## Play quest: Shared table

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md).

**Goal:** two phones look at the same table and see the same battle. A move made on one shows on the other.

- There are two separate problems here. One is **shared space**: both phones agreeing where the creatures are. The other is **shared state**: both phones agreeing on HP and whose turn it is. Which one do image-tracked cards already solve for you, and why?
- Research how ARKit lets nearby devices share a session (collaborative sessions, Multipeer Connectivity). What does each device send?
- Decide who is in charge of the match state. What goes wrong if both phones can change it at once?
- Send as little as possible over the network. What is the smallest message that describes one move?
- Get a build onto a friend's phone with TestFlight (see [docs/setup.md](../../docs/setup.md#6-testflight-from-stage-5)).
- Stretch: one player per phone, each only able to act on their own turn.

**Done when** you and a friend finish a match on two phones, and you can explain in the log what was sent between them.

## Backlog ideas from earlier stages that fit here

If you finish early, these Stage 4 and Stage 8 quests slot straight into the battler: *Card extras* (evolve by stacking, AR dice, damage counters), *Deck scanner* (build a collection by scanning), *Instant card* (an App Clip code that launches a card's AR).

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
