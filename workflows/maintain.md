# Keep the Dashboard Current

The dashboard is `index.html`. For each system, it draws the modules and what each one depends on, lists every version with what changed, and shows the decisions and open questions. Every version opens its document. The page can't read the documents, so what it shows comes from `assets/systems.json`, and keeping that file in line with the architecture is your job. The documents stay the source: everything in the JSON is a short version of something already in `architecture.md` or `versions/`. StashBase shows the change within a couple of seconds, even while the dashboard is open.

## When to update it

* **After writing a system's first version:** add the system, with version 1.
* **After updating `architecture.md`:** update the current version's modules and links if they changed, and the decisions and open questions.
* **After saving a version, or going back to one:** add the new version, as below.
* **At setup:** replace the example with my first system, and leave out `example`.
* **When I ask you to check the design:** compare the JSON with `architecture.md`, and fix every mismatch.

## Versions

Each version in the JSON holds its own copy of the modules and links, the way `versions/` holds a copy of each document. The dashboard compares each version with the one before it, and marks what's new, changed, or removed.

* **Save a version:** copy the current version's entry, add the copy at the end of the list, and change the copy: its `version`, `date`, `change`, `why`, modules, and links. The copy is now the current version.
* **Go back:** add a new version at the end, with the modules and links copied from the old one and a `change` like "Back to v2".
* **Anything smaller:** update the current version, the last in the list, in place.
* Never change an earlier version's entry, the same as the files in `versions/`.

## Editing rules

* Keep it valid JSON. Change only what you need, and leave every other field as it is, including fields not listed here.
* A system's `id` is its folder name in `systems/`. The dashboard opens `systems/<id>/architecture.md` for the current version and `systems/<id>/versions/v<N>.md` for earlier ones, so ids and version numbers must match the files.
* Don't change `schemaVersion` or `index.html`.
* The file can hold up to 500 KB. If it gets close, tell me, and offer to keep only the last few versions of each system in it. The documents in `versions/` stay as they are.

## Format

```json
{
  "example": true,
  "systems": [{
    "id": "clinic-booking",
    "name": "Clinic Booking",
    "goal": "Patients book visits online, the front desk runs each clinician's day, and ...",
    "versions": [{
      "version": 3,
      "date": "2026-10-28",
      "change": "Split Availability out of Booking",
      "why": "The front desk now blocks time off and changes hours every week, ...",
      "modules": [
        { "id": "booking", "name": "Booking", "kind": "module", "does": "Creates, moves, and cancels visits, ...", "code": "src/booking/" },
        { "id": "postgres", "name": "PostgreSQL", "kind": "outside", "role": "Database" }
      ],
      "links": [{ "from": "booking", "to": "postgres", "via": "VisitStore" }]
    }],
    "decisions": [{ "title": "Availability is its own module", "why": "..." }],
    "openQuestions": [{ "title": "Reminder timing", "detail": "..." }]
  }]
}
```

* `example` is `true` only while the template's example is in place; the dashboard then also tells people to say "Set me up." Setup removes it.
* `name` is the document's title, and `goal` is its Goal, shortened to one sentence.
* `version` and `date` match the top of that version's document. `change` and `why` are its row in the Versions table, shortened: `change` to a short title, `why` to a sentence or two. Version 1's `change` is "First design", with an empty `why`.
* `modules` follow the document's Modules section, in order, then the outside services and databases the system uses.
  * `kind` is `app` for what people use, like a web or mobile app; `module` for the rest of what we build; and `outside` for what the system uses but we don't build, like a database or an outside API.
  * `id` is short, lowercase, with hyphens, like `web-api`. It stays the same across versions: that's how the dashboard knows a module is the same one.
  * `does` is the module's Does line, shortened to one sentence. Change it only when what the module does changes, because the dashboard marks a module as changed when its `does` or its links change.
  * `code` is the Code line, for apps and modules. `role` is what an outside service is for, in a word or two, like "Database" or "Text messages".
* `links` are the Depends on lines: a link from a module to each module or outside service it depends on, with `via` the interface it goes through. The link goes to the module that owns the interface. For an outside service, the link goes from the module that uses it, through the interface that wraps it, like `MessageChannel`. Apps link to what they call, with `via` like "HTTP". One link per pair: when a module uses two interfaces of another, put both in `via`, like "Bookings, BookingEvents".
* `decisions` are the current document's Decisions: `title` is the bold part, and `why` is the rest, shortened to a sentence or two.
* `openQuestions` are its Open questions, the same way: `title` and `detail`.
