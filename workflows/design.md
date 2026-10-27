# Design the System

Everything for one system goes in `systems/<name>/`:

* `architecture.md`: the current design. A coding agent should be able to build from it alone.
* `versions/`: old versions, saved as `v1.md`, `v2.md`, and so on. See [versions.md](versions.md).

## 1. Start a new system

Make the system's folder. Then get these from me, from my message or the files I attach:

* **Goal:** what the system does, and for whom.
* **Main use cases:** the few things users do most, like "a patient books an appointment".
* **Constraints:** stack, scale, team, deadlines, and systems it has to work with. Start from my [profile](../context/profile.md).

Without the goal and the main use cases there's nothing to design, so if either is missing, ask me, one question at a time. For anything else that's missing, make the simplest assumption and list it under Open questions.

Then write `architecture.md` as version 1, in the [format](#format) below, and check it as in [step 3](#3-check-the-design). In Chat, tell me the design in a few sentences: the modules, the one decision that matters most, and what's still open.

## 2. Keep it current

Most of our work is talk. I ask why, suggest a change, push back, or change my mind.

* When I ask a question or think out loud, answer in Chat. Don't touch the file.
* When something is decided, because I say so or I agree with what you suggested, update `architecture.md` right away. If the change reshapes the design, follow [versions.md](versions.md) first.
* Keep the whole document consistent. When a module changes, update every interface, flow, and decision that mentions it.
* Write each decision under Decisions, with the reason and what we chose not to do.
* After each update, tell me what changed in a line.

## 3. Check the design

Check the design every time you write or update it, and when I ask. Fix what you can, and tell me about anything you can't fix without me.

* **The goal:** walk through each main use case using only the modules and interfaces in the document. A step no module covers is a gap.
* **SOLID:** follow the rules [below](#solid).
* **Buildable:** a coding agent could build each module without asking. Every name, interface, and dependency is spelled out.

When I ask for something that breaks a rule, tell me what it will cost in a sentence, and suggest a fix. If I still want it, do it, and write it under Decisions with my reason.

## SOLID

Apply each principle where the code is likely to change. Where it isn't, keep it simple.

* **Single responsibility:** each module has one reason to change. If its responsibility needs an "and", check whether the two parts change for different reasons. If they do, split it.
* **Open/closed:** where new variants are expected, like payment providers or notification channels, design it so a new one is added without editing existing code. Where no new variants are coming, don't.
* **Liskov substitution:** every implementation of an interface works wherever the interface is used. No implementation throws "not supported" for part of it.
* **Interface segregation:** keep interfaces small, and shape them by the module that uses them, not the one that implements them.
* **Dependency inversion:** business logic doesn't depend directly on the database, the framework, or outside services. It depends on interfaces it owns, and the outside implements them.

Don't over-apply them:

* No interface for something with one implementation and no reason to swap it, except at the edges (the database, outside APIs, the clock), where tests need a fake.
* Fewer modules is better, until splitting one pays for itself.
* No patterns for their own sake. If you can't say what change a pattern makes easier, leave it out.

## Format

`architecture.md` uses these headings, in this order. Leave out a section with nothing in it, except Open questions.

```markdown
# <System name>

Version <N> · <date>

## Goal
What the system does, for whom, and what it leaves out.

## Constraints
Stack, scale, team, deadlines, and systems it has to work with. Only what I've said, plus assumptions marked as such.

## Modules
### <Module>
* **Does:** its one responsibility.
* **Owns:** the data only it writes.
* **Depends on:** other modules' interfaces, by name.
* **Code:** where it lives, like `src/booking/`.

## Interfaces
The contracts between modules, as code in the project's language: names, inputs, outputs, and errors. Only the ones between modules and at the edges, not every class.

## Key flows
Each main use case, step by step: which module does what, through which interface.

## Decisions
* **<Decision>:** why, and what we chose not to do.

## Open questions
What's not decided yet, and the assumptions we're working with.

## Versions
| Version | Date | What changed | Why |
| --- | --- | --- | --- |
```

Write it so a coding agent can build from it without asking:

* Name each thing once and use that name everywhere: modules, interfaces, methods, and data.
* Say what a module does with concrete verbs. "Manages bookings" is too vague. "Creates, moves, and cancels bookings, and rejects double bookings" is enough.
* Write interfaces as real code signatures, not prose.
* Keep to what shapes the structure. Leave out field-level detail, styling, and setup steps, unless a decision depends on them.
