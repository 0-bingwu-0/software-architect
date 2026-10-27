# Versions

`architecture.md` always holds the current version. Its number is at the top, and the Versions table at the bottom says what changed in each version and why. Old versions are copies in the system's `versions/` folder: `v1.md`, `v2.md`, and so on.

## When to save a version

Save one before a change that reshapes the design:

* Adding, removing, splitting, or merging a module.
* Changing an interface, or what depends on what.
* Reversing a decision.
* A new goal or constraint that changes any of the above.

Don't save one for filling in detail, fixing a mistake, rewording, or updating Open questions. Just update the file.

One decision is one version, even if it touches several modules. Several decisions in one message of mine are one version too.

## Save a version

1. Tell me first, at the start of your reply, in a sentence: what's changing, and the version you're saving. Like: "This splits Booking into Booking and Scheduling, so I'm saving the current design as v2." Don't wait for my OK.
2. Copy `architecture.md` as it is to `versions/v<N>.md`, where N is its current version number.
3. Update `architecture.md`. At the top, change the version to N+1 and the date to today. Add a row to the Versions table: what changed and why, a line each.

The first version needs no copy. Write version 1 with one row in the table: "First design."

## Compare versions

When I ask what changed, like "What changed since v2?", compare that version with the current one. Tell me in Chat, module by module, with the reasons from the Versions table. Leave out wording changes.

## Go back

When I ask to go back to an old version:

1. Save the current version first, as above, so nothing is lost.
2. Copy the old version into `architecture.md` under the next version number, not its old one. Keep the full Versions table, and add a row like "Back to v2", with my reason.

Never change or delete a file in `versions/`.
