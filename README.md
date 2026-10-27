# Software Architect

Before you build anything sizable, you need a design: the modules, what each one does, and how they talk to each other. Talking it through with AI works well, until the design is spread across a long chat. Change your mind halfway, and the old design and the new one get mixed up. A week later, nobody remembers why it changed.

This project keeps the design in one document while you talk. AI works as your software architect: it designs with SOLID, and tells you when something is over-engineered. Each time you decide something, it updates the architecture. Before a big change, it saves the old version, with the reason. When you're done, Claude Code or Codex can build from it.

## Getting started

Open this project in StashBase and tell the Agent:

1. **"Set me up."** It asks you a few questions, one at a time: what you do, what you usually build with, any rules your designs follow, and the language for your docs.
2. **"Design a booking system for a small clinic."** Say what it does, who uses it, and the few things they do most. It makes `systems/<name>/`, asks if the goal or the main use cases are missing, and writes the first version of `architecture.md`: modules, interfaces, key flows, and the decisions behind them.
3. **Talk it through.** Ask why, push back, change your mind: "What if we add SMS reminders?" Each time something is decided, it updates `architecture.md`. If the change reshapes the design, it tells you and saves the current version to `versions/` first.
4. **"What changed since v2?"** It tells you module by module, with the reasons. Changed your mind? Say "Go back to v2."

When the design is ready, copy `architecture.md` into your code project and ask Claude Code or Codex to build from it. Interfaces are written as code, so it doesn't have to guess.

Want a second look? Say **"Check the design."** It walks each use case through the modules, checks SOLID, and tells you what to fix.

## Make it yours

Setup covers most of it. You can also change things anytime:

* **Your role, your stack, and your team's rules** are in `context/profile.md`. Add a rule anytime, like "no microservices", or just tell the Agent.
* **How the architecture is written, and how SOLID is applied** are in `workflows/design.md`.
* **When a version is saved** is in `workflows/versions.md`.
* **The Agent's persona and rules** are in `AGENTS.md`.

## Files

* `AGENTS.md`: the Agent's persona, the design rules, and the task index.
* `CLAUDE.md`: points Claude Code to `AGENTS.md`.
* `context/profile.md`: your role, stack, house rules, and doc language.
* `workflows/setup.md`: the setup Q&A.
* `workflows/design.md`: starting a system, keeping it current, checking it, and the format of `architecture.md`.
* `workflows/versions.md`: when and how versions are saved, compared, and restored.
* `systems/`: one folder per system, with `architecture.md` and `versions/`.
