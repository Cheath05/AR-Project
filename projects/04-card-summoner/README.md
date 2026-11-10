# Stage 4 — Image tracking in Unity

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity 6, AR Foundation, your iPhone, printed cards

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **Reference image libraries:** how the app is told in advance which pictures to look for, and how big they are in real life.
- **Tracked-image lifecycle:** added, updated and removed, and the tracking states in between.
- **Multi-image tracking:** following several cards at once, and its limits.
- **Image targets in a tracked world:** unlike Stage 0, the phone now knows where the card is in the room.

This is Stage 0 again with the tools from Stages 2 and 3, and it is the base of the card battler.

## Read first

- AR Foundation, image tracking: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/image-tracking.html
- Apple on detecting images, and what makes a good one: https://developer.apple.com/documentation/arkit/detecting-images-in-an-ar-experience
- Unity Learn, AR markers and planes (the markers half): https://learn.unity.com/course/create-with-ar-markers-and-planes

## Core build: card summoner

**Goal:** lay three or more printed cards on a table and see a different creature standing on each.

There is no starter. Create the Unity project inside this folder. The image tracking scenes in the AR Foundation samples are a working reference to compare against, after you have tried: https://github.com/Unity-Technologies/arfoundation-samples

Use cards you drew yourself. Photos of real trading cards go in a `private/` folder, which is git-ignored.

### Tasks

1. **Build a reference image library.** Add three or more card images and give each a physical size. Measure a printed card with a ruler. What happens to the creature's position and scale if the size you entered is wrong? Predict, then try doubling it.
2. **Track one card.** Show a simple shape on one card when it is detected. Read the tracked image's name and tracking state and log them. How does this compare with the found and lost events in Stage 0?
3. **Tell the cards apart.** When a card is detected, work out which one it is. Design a data structure that maps a card to its creature. Where should that mapping live so that adding a card does not mean editing code?
4. **A creature per card.** Spawn the right creature on each card, standing upright and sized sensibly. Find or make three simple models (free sources are in `docs/resources.md`). Each needs its own scale and rotation fix; where do you store those?
5. **Several at once.** Put all your cards on the table together. How many are tracked at the same time? Find the setting that controls this and the limit on your phone. Move one card while the others stay still.
6. **Handle lost tracking.** Cover a card, flip it over, slide it out of view. Decide what the creature does in each case: vanish at once, linger, fade? The tracking state changes before the image is ever removed. Use it. What happens to a creature whose card is out of view but still known to the world tracker?

### Done when

- [ ] Three or more cards each show their own creature at the same time.
- [ ] Creatures follow their cards as you slide them around the table.
- [ ] Covering or removing a card does something deliberate, not a frozen or floating creature.
- [ ] Adding a fourth card needs a new image and a data entry, and no new code.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Why does the library need the physical size of each image? Stage 0 did not ask for it.
2. A card is out of view but its creature is still drawn in the right place. How is that possible here and not in Stage 0?
3. Two cards share the same border and layout and differ only in a small picture. What will the tracker do, and how would you design cards to avoid it?
4. Tracking a moving card is harder than a still one. What do you see when you move a card quickly, and why?

## Study quest: Textbook triggers

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md). One chapter first, then a second class.

**Goal:** point the phone at a marker on a textbook page or a page of your notes and get your summary, or a 3D model, for that chapter.

- A full page of text is a poor image target. Why? Design a small marker to stick on the page instead. What must be true of a set of markers so they are never confused?
- Each marker maps to content: a summary panel, a diagram, or a model from Stage 2. Reuse your card-to-creature data structure.
- Show the content beside the page, not on top of the text you are reading.
- Decide where the content lives so you can add a chapter without rebuilding the app.
- Stretch: once a page has been seen, pin its content to an anchor so it stays up after you look away.

**Done when** two classes have at least one working marker each and you have used them while reading. Do not commit scans of textbook pages.

## Study quest: Molecule builder

**Scheduled:** backlog.

**Goal:** printed atom cards that bond when you slide them together.

- One card per element, each showing a labelled sphere.
- Every frame you know where each tracked card is. When are two atoms "close enough"? Draw a bond between them.
- Atoms have limits on how many bonds they make. Where do those rules live?
- Recognise one or two simple molecules and show their names.
- Stretch: show bond angles, or snap the atoms into the molecule's real shape.

## Play quest: Card extras

**Scheduled:** backlog.

**Goal:** make the cards feel like a game. Pick two.

- **Evolve:** place a second card next to the first, or stack it, and the creature changes. How do you detect "next to"?
- **AR dice:** tap to roll a physics die on the table (Stage 2 planes) and read which face landed up.
- **Damage counters:** tap a creature to add or remove counters that float above its card.
- **Idle life:** creatures turn to face each other, or face you.

Whatever you build here carries straight into Stage 9.

## Play quest: Tabletop fog of war

**Scheduled:** backlog.

**Goal:** a printed map on the table, hidden under fog that clears around player tokens as they move.

- The map is one large image target; tokens are small ones. What size and detail does a small token need to track at all?
- Fog is a layer over the map with holes in it. Research ways to do this: a texture you paint into, or a shader.
- Token positions are in world space. How do you convert them to positions on the map?
- Stretch: fog that stays cleared where tokens have been, shown dimmer.

## Play quest: Instant card

**Scheduled:** backlog. Needs a TestFlight build, so do it after Stage 5.

**Goal:** scan a code on a card with the iPhone camera and its AR experience opens with no app installed.

- This is an App Clip: a small slice of an app that launches from a code or NFC tag. Start with [docs/apple-services.md](../../docs/apple-services.md).
- App Clips have a strict size limit. A full Unity build may not fit. Find out the limit and how big your build is, then decide what the clip can realistically contain.
- A simpler first version to research: a QR code that opens a `.usdz` model in AR Quick Look, with no app at all.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
