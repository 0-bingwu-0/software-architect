# Software Architect

Before you build anything sizable, you need a design: the modules, what each one does, and how they talk to each other. Talking it through with AI works well, until the design is spread across a long chat. Change your mind halfway, and the old design and the new one get mixed up. A week later, nobody remembers why it changed.

This project keeps the design in one document while you talk. AI works as your software architect: it designs with SOLID, and tells you when something is over-engineered. Each time you decide something, it updates the architecture. Before a big change, it saves the old version, with the reason. A dashboard draws the design as a diagram, so you can see every version and what it changed. When you're done, Claude Code or Codex can build from it.

## Getting started

Open this project in StashBase and tell the Agent:

1. **"Set me up."** It asks you a few questions, one at a time: what you do, what you usually build with, any rules your designs follow, the language for your docs, and what you want to design first. Then it replaces the example with the first version of your design.
2. **"Design an inventory app for a bike shop."** Say what it does, who uses it, and the few things they do most. It makes `systems/<name>/`, asks if the goal or the main use cases are missing, and writes the first version of `architecture.md`: modules, interfaces, key flows, and the decisions behind them.
3. **"What if we add online payments?"** Ask why, push back, change your mind. Each time something is decided, it updates `architecture.md`. If the change reshapes the design, it tells you and saves the current version to `versions/` first.
4. **"What changed since v2?"** It tells you module by module, with the reasons. Changed your mind? Say "Go back to v2."

When the design is ready, copy `architecture.md` into your code project and ask Claude Code or Codex to build from it. Interfaces are written as code, so it doesn't have to guess.

Want a second look? Say **"Check the design."** It walks each use case through the modules, checks SOLID, and tells you what to fix.

The template comes with an example: Clinic Booking, the design of online booking for small clinics, now at version 3. Version 1 was the first design, version 2 added text and email reminders, and version 3 moved clinicians' hours out of booking into a module of their own. Step through the versions on the dashboard, or ask "What changed since v1?", before running setup. Setup replaces it with yours.

## The dashboard

The architecture is all text, and you never have to open anything else. The dashboard, `index.html`, draws it as a diagram, and the Agent keeps it current each time the design changes.

* **Architecture:** every module, above what it depends on, down to the database and outside services. Click a module to see what it depends on, and through which interface. Switch versions to see what each one added, changed, or removed.
* **Versions:** what changed in each version, and when. Each one opens its document.
* **Decisions and open questions:** what's been decided, and what's still open. Hover over one for the reason.

## Make it yours

Setup covers most of it. You can also change things anytime:

* **Your role, your stack, and your team's rules** are in `context/profile.md`. Add a rule anytime, like "no microservices", or just tell the Agent.
* **How the architecture is written, and how SOLID is applied** are in `workflows/design.md`.
* **When a version is saved** is in `workflows/versions.md`.
* **What the dashboard shows,** and how the Agent keeps it current, is in `workflows/maintain.md`.
* **The Agent's persona and rules** are in `AGENTS.md`.

## Files

* `AGENTS.md`: the Agent's persona, the design rules, and the task index.
* `CLAUDE.md`: points Claude Code to `AGENTS.md`.
* `context/profile.md`: your role, stack, house rules, and doc language.
* `workflows/setup.md`: the setup Q&A.
* `workflows/design.md`: starting a system, keeping it current, checking it, and the format of `architecture.md`.
* `workflows/versions.md`: when and how versions are saved, compared, and restored.
* `workflows/maintain.md`: how the Agent keeps the dashboard in line with the architecture, and the format of its data.
* `systems/`: one folder per system, with `architecture.md` and `versions/`. The example is in `systems/clinic-booking/`.
* `index.html`: the dashboard.
* `assets/systems.json`: everything the dashboard shows, a short version of each system and its versions.
* `assets/banner.png`: the dashboard's banner.
