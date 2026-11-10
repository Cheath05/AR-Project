# Coding from zero

Read this once before Stage 0. It assumes you have never really coded. It explains how the pieces fit together, what you need to learn and when, and the routine to follow every time you sit down.

## The two languages in this roadmap

| Language | File ending | Where it runs | Used in |
|---|---|---|---|
| **JavaScript** (inside an HTML page) | `.html`, `.js` | A web browser | Stage 0 only |
| **C#** (say "C sharp") | `.cs` | Inside Unity | Stages 1–12 |

They look alike and share the same core ideas, so what you learn in Stage 0 is not wasted. From Stage 8 you will also write a few short **Swift** files (`.swift`) to reach iPhone features Unity does not include.

You will never create a `.cpp` file here. That is C++, a different language.

## The ideas every language shares

You need these seven before any brief will make sense. The tutorials below teach them; this table tells you what to look for.

| Idea | What it is | JavaScript | C# |
|---|---|---|---|
| **Variable** | A named box holding a value | `const size = 2;` | `float size = 2f;` |
| **Type** | What kind of value: number, text, true/false | Worked out for you | You must state it: `int`, `float`, `string`, `bool` |
| **Function / method** | A named set of steps you can run | `function grow() { ... }` | `void Grow() { ... }` |
| **Calling** | Running a function, giving it inputs | `grow(2);` | `Grow(2);` |
| **If** | Do something only when a condition is true | `if (size > 3) { ... }` | Same |
| **Loop** | Repeat steps | `for (const card of cards) { ... }` | `foreach (var card in cards) { ... }` |
| **Object and dot** | A thing with its own data and functions, reached with `.` | `cube.position.x` | `cube.transform.position.x` |

Two more arrive early and confuse everyone at first:

- **Callback / event:** "when *this* happens, run *that* function." You hand over a function to be called later, for example when a card is found.
- **Class:** a blueprint for making objects. `new Thing()` makes one from the blueprint. In C#, every script you write is a class.

## What to learn, and when

Do not try to learn everything first. Learn just enough, then build.

### Before Stage 0 (about 3–4 hours)

1. MDN, **"What is JavaScript?"** through **"What went wrong?"** in the Scripting module. Type every example yourself; do not copy and paste.
   https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting
2. javascript.info, **"JavaScript Fundamentals"**, the sections on variables, functions, and conditionals. Use it as a second explanation when MDN does not click.
   https://javascript.info/first-steps

Stop there. Then open the [Stage 0 guide](../projects/00-web-image-tracking/GUIDE.md).

### During Stage 1 (about 10–12 hours, already on the schedule)

1. Microsoft's **beginner C# tutorials**, which run in the browser with nothing to install:
   https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/
2. If you prefer video, **C# Fundamentals for Absolute Beginners**:
   https://learn.microsoft.com/en-us/shows/csharp-fundamentals-for-absolute-beginners/
3. **Unity Essentials**, which teaches the editor and your first scripts:
   https://learn.unity.com/pathway/unity-essentials

The [Stage 1 guide](../projects/01-hello-phone/GUIDE.md) says which parts to do in which order.

### Later, as each stage needs it

Each stage's `GUIDE.md` starts with a "New coding ideas" list and a link for each. Learn them then, not before.

## How a Unity project is put together

This is the part no C# tutorial explains, and the reason "where do I even start" is hard.

- A **Scene** is one screen or level. It holds **GameObjects**.
- A **GameObject** is a thing in the scene: a cube, a camera, a light, an empty holder. On its own it does nothing.
- A **Component** is a behaviour attached to a GameObject. A cube has a component that draws it and one that sets its position (its **Transform**).
- A **script** is a component *you* write. It is a `.cs` file containing one class. You attach it to a GameObject, and Unity runs it.
- The **Inspector** is the panel showing the selected GameObject's components and their settings.
- A **Prefab** is a saved GameObject you can make copies of, such as a creature that appears on a card.

So the pattern for nearly every task in stages 1–12 is:

1. Create a script file in `Assets/Scripts/`.
2. Write a class in it with the behaviour you want.
3. Attach the script to a GameObject in the scene.
4. Fill in its settings in the Inspector.
5. Run it and read the Console.

Unity calls certain functions in your script automatically. The two you will use most:

- `Start()` runs once, when the object first appears.
- `Update()` runs every frame, about 60 times a second.

Read these two short pages when you reach Stage 1:
https://docs.unity3d.com/Manual/creating-scripts.html and https://docs.unity3d.com/Manual/class-MonoBehaviour.html

## The routine for every task

Follow this each time. It is how working programmers operate too.

1. **Say it in plain English first.** Write the steps as comments before any code. "When the screen is tapped, find where the tap hits the floor, put a cube there." If you cannot write the steps in English, you are not ready to write them in code, and that is the thing to ask about.
2. **Find the one thing you do not know how to do.** Usually it is a single line: "how do I detect a tap?" Search the docs linked in the guide for that one thing.
3. **Write the smallest piece and run it.** One or two lines, then test. Never write twenty lines before running anything.
4. **Print things.** When unsure what your code is doing, print a value and look. JavaScript: `console.log(value)`. C#: `Debug.Log(value)`.
5. **Read the error.** It tells you the file, the line number and what went wrong. Go to that line. Most errors are a typo, a missing bracket, or using a name before it exists.
6. **Commit when it works.** Then start the next piece. A commit is a save point you can return to.

## When you are stuck for more than 20 minutes

Bring three things and ask:

1. What you are trying to make happen, in plain English.
2. The code you wrote (paste it).
3. What happened: the error text, or what you saw compared with what you expected.

Then ask for one of these, in order: an explanation of the idea, a hint, a review of your code, or a small example of the one line you are missing. Being stuck is normal. Staying stuck silently is the only mistake.

## Words you will keep seeing

| Word | Meaning |
|---|---|
| **Syntax** | The spelling and punctuation rules of a language |
| **Bug** | Code that runs but does the wrong thing |
| **Compile error** | Code the computer cannot understand at all; nothing runs until it is fixed |
| **Console** | The window where printed messages and errors appear |
| **API** | The list of functions someone else's code offers you |
| **Docs** | The written reference for an API |
| **Library / package** | Someone else's code you add to your project (three.js, AR Foundation) |
| **Build** | Turning your project into an app that runs on the phone |
| **Repo** | The project folder that Git tracks |
