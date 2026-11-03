# Clinic Booking

Version 3 · 2026-10-28

## Goal

Patients of a small clinic book visits online, and the front desk runs each clinician's day: books visits for callers, moves and cancels them, and sets hours and time off. A reminder goes out the day before each visit, by text or email, to cut no-shows.

It leaves out payments, insurance, video visits, patient accounts, and clinics with more than one location.

## Constraints

* TypeScript throughout: Node.js with Express, React for the two web apps, PostgreSQL through Prisma.
* One container service and a managed database on AWS. One deployable service, no microservices.
* One deployment serves every clinic. Each clinic has its own subdomain, like `harbor.clinicbook.app`, and every record belongs to one clinic.
* About 40 clinics at launch, each with 2 to 8 clinicians and up to 150 visits a day.
* Two engineers. The first clinics go live in January.
* Text messages go through Twilio and email through Postmark. We already have accounts with both.
* Our team sets up each clinic and its clinicians at onboarding, so there are no screens for that yet.
* Assumption: a visit is 15 to 60 minutes long and starts on a quarter hour.

## Modules

### Front desk app

* **Does:** Shows each clinician's day. Staff book visits for callers, move and cancel visits, and set clinicians' weekly hours and time off. Visits that fall outside open hours stand out, so staff can move them.
* **Depends on:** Web API, over HTTP.
* **Code:** `web/front-desk/`

### Booking page

* **Does:** Lets a patient pick a clinician and a free slot, enter their name and phone or email, and book.
* **Depends on:** Web API, over HTTP.
* **Code:** `web/booking-page/`

### Web API

* **Does:** Serves both apps over HTTP. Finds the clinic from the subdomain, checks staff sign-in on front desk routes, and turns each request into one call to a module. Holds no business rules.
* **Depends on:** `Bookings`, `Schedules`, `PatientDirectory`.
* **Code:** `src/api/`

### Booking

* **Does:** Creates, moves, and cancels visits. Rejects a visit that overlaps another one for the same clinician, or falls outside the clinician's open hours. Lists free slots and a clinician's visits for a day. Announces every change as a booking event.
* **Owns:** visits.
* **Depends on:** `OpenHours`, `VisitStore`, `Clock`.
* **Code:** `src/booking/`

### Availability

* **Does:** Keeps each clinic's clinicians, with their weekly hours and time off, and works out which hours are open on a given day.
* **Owns:** clinicians, weekly hours, and time off.
* **Depends on:** `ScheduleStore`.
* **Code:** `src/availability/`

### Patients

* **Does:** Keeps each patient's name, phone, email, and how they want reminders: text, email, or none. Finds a returning patient by phone or email, so booking again doesn't create a duplicate.
* **Owns:** patients.
* **Depends on:** `PatientStore`.
* **Code:** `src/patients/`

### Reminders

* **Does:** Schedules a reminder 24 hours before each visit, moves it when the visit moves, and drops it when the visit is cancelled. Every minute, sends the reminders that are due.
* **Owns:** reminder jobs.
* **Depends on:** `BookingEvents`, `ContactLookup`, `Notifier`, `ReminderStore`, `Clock`.
* **Code:** `src/reminders/`

### Notifications

* **Does:** Sends one message to one patient through the channel they chose, and reports whether the provider accepted it.
* **Depends on:** `MessageChannel`, implemented by `SmsChannel` with Twilio and `EmailChannel` with Postmark.
* **Code:** `src/notifications/`

## Interfaces

```ts
// Shared types
type LocalDate = string; // "2026-11-03", in the clinic's time zone
type TimeRange = { start: Date; end: Date }; // UTC
type Slot = { clinicianId: string; start: Date; end: Date };
type Visit = {
  id: string;
  clinicId: string;
  clinicianId: string;
  patientId: string;
  start: Date;
  end: Date;
  reason?: string;
};
type Clinician = { id: string; clinicId: string; name: string };

class SlotTaken extends Error {}
class OutsideOpenHours extends Error {}
class VisitNotFound extends Error {}

// Booking, used by the Web API
type DayVisit = Visit & { outsideOpenHours: boolean };

interface Bookings {
  freeSlots(clinicianId: string, date: LocalDate, minutes: number): Promise<Slot[]>;
  book(slot: Slot, patientId: string, reason?: string): Promise<Visit>; // SlotTaken, OutsideOpenHours
  move(visitId: string, to: Slot): Promise<Visit>; // SlotTaken, OutsideOpenHours, VisitNotFound
  cancel(visitId: string): Promise<void>; // VisitNotFound
  day(clinicianId: string, date: LocalDate): Promise<DayVisit[]>;
}

// Booking, for modules that react to visits
type BookingEvent =
  | { type: "visit-booked"; visit: Visit }
  | { type: "visit-moved"; visit: Visit }
  | { type: "visit-cancelled"; visitId: string };

interface BookingEvents {
  subscribe(handler: (event: BookingEvent) => Promise<void>): void;
}

// Availability, used by Booking
interface OpenHours {
  openHours(clinicianId: string, date: LocalDate): Promise<TimeRange[]>; // time off already taken out
}

// Availability, used by the Web API
type Day = "mon" | "tue" | "wed" | "thu" | "fri" | "sat" | "sun";
type WeeklyHours = Record<Day, { from: string; to: string }[]>; // "09:00", in the clinic's time zone

interface Schedules {
  clinicians(clinicId: string): Promise<Clinician[]>;
  weeklyHours(clinicianId: string): Promise<WeeklyHours>;
  setWeeklyHours(clinicianId: string, hours: WeeklyHours): Promise<void>;
  addTimeOff(clinicianId: string, range: TimeRange, note?: string): Promise<string>; // returns its id
  removeTimeOff(timeOffId: string): Promise<void>;
}

// Patients, used by the Web API
type ReminderChoice = "text" | "email" | "none";
type Patient = {
  id: string;
  clinicId: string;
  name: string;
  phone?: string;
  email?: string;
  reminders: ReminderChoice;
};

interface PatientDirectory {
  findOrCreate(clinicId: string, details: { name: string; phone?: string; email?: string }): Promise<Patient>;
  setReminderChoice(patientId: string, choice: ReminderChoice): Promise<void>;
}

// Patients, used by Reminders
type Contact = { name: string; channel: "text" | "email"; address: string };

interface ContactLookup {
  contactFor(patientId: string): Promise<Contact | null>; // null when the patient chose no reminders
}

// Notifications, used by Reminders
type Delivery = { accepted: boolean; providerId?: string; error?: string };

interface Notifier {
  send(to: Contact, text: string): Promise<Delivery>;
}

// Notifications, one per provider: SmsChannel (Twilio) and EmailChannel (Postmark)
interface MessageChannel {
  readonly channel: "text" | "email";
  send(address: string, text: string): Promise<Delivery>;
}

// Stores: each owned by the module that uses it. src/db/ implements them with Prisma.
interface VisitStore {
  get(visitId: string): Promise<Visit | null>;
  forDay(clinicianId: string, range: TimeRange): Promise<Visit[]>;
  save(visit: Visit): Promise<void>; // SlotTaken when the database catches an overlap
  remove(visitId: string): Promise<void>;
}

interface ScheduleStore {
  clinicians(clinicId: string): Promise<Clinician[]>;
  weeklyHours(clinicianId: string): Promise<WeeklyHours>;
  saveWeeklyHours(clinicianId: string, hours: WeeklyHours): Promise<void>;
  timeOff(clinicianId: string, range: TimeRange): Promise<TimeRange[]>;
  addTimeOff(clinicianId: string, range: TimeRange, note?: string): Promise<string>;
  removeTimeOff(timeOffId: string): Promise<void>;
}

interface PatientStore {
  get(patientId: string): Promise<Patient | null>;
  findByContact(clinicId: string, phone?: string, email?: string): Promise<Patient | null>;
  save(patient: Patient): Promise<void>;
}

type ReminderJob = { id: string; visitId: string; patientId: string; visitStart: Date; sendAt: Date };

interface ReminderStore {
  schedule(job: Omit<ReminderJob, "id">): Promise<void>; // replaces the visit's pending reminder
  drop(visitId: string): Promise<void>;
  takeDue(now: Date, limit: number): Promise<ReminderJob[]>; // locks what it takes, so two copies never send twice
  markSent(jobId: string, delivery: Delivery): Promise<void>;
}

interface Clock {
  now(): Date;
}
```

## Key flows

### A patient books a visit

1. The booking page asks the Web API for free slots. The Web API calls `Bookings.freeSlots`. Booking gets the clinician's open hours from `OpenHours.openHours` and takes out the visits already booked.
2. The patient picks a slot and enters their details. The Web API calls `PatientDirectory.findOrCreate`, then `Bookings.book`.
3. Booking checks the slot against open hours and other visits, and saves it with `VisitStore.save`. If someone else took the slot a moment earlier, the database rejects it, and the patient sees `SlotTaken` and picks again.
4. Booking announces `visit-booked`. Reminders schedules a reminder for 24 hours before the visit.

### The front desk moves or cancels a visit

1. Staff drag the visit to a new time in the front desk app, or cancel it. The Web API calls `Bookings.move` or `Bookings.cancel`.
2. A move goes through the same checks as a new booking.
3. Booking announces `visit-moved` or `visit-cancelled`. Reminders moves the reminder or drops it.

### The front desk blocks time off

1. Staff add time off in the front desk app. The Web API calls `Schedules.addTimeOff`.
2. Those hours stop showing up in `Bookings.freeSlots`. Visits already booked in that time stay, and come back from `Bookings.day` with `outsideOpenHours: true`, so the front desk app can flag them for staff to move.

### A reminder goes out

1. Every minute, Reminders takes the due jobs with `ReminderStore.takeDue`.
2. For each job, it asks `ContactLookup.contactFor` for the patient's channel and address. If the patient chose no reminders, it drops the job.
3. It calls `Notifier.send`, and Notifications sends it through the `MessageChannel` for that channel. Reminders records the result with `ReminderStore.markSent`.

## Decisions

* **One service, one folder per module:** It's the house rule, and two engineers can't run more. Modules call each other only through the interfaces above, so one could become its own service later without changing its callers. Not chosen: separate services for booking and reminders.
* **Each module owns its store interface:** `src/db/` implements `VisitStore`, `ScheduleStore`, `PatientStore`, and `ReminderStore` with Prisma. Business logic never imports Prisma, and tests use in-memory stores. Not chosen: one shared repository layer.
* **Double bookings are stopped twice:** Booking checks for overlaps, and an exclusion constraint on clinician and time range in PostgreSQL catches two bookings made at the same moment. Not chosen: a lock per clinician.
* **Times are stored in UTC:** Weekly hours are kept in the clinic's local time and turned into UTC for each day, so a daylight saving change doesn't shift anyone's hours. Not chosen: storing local times.
* **Booking announces events instead of calling Reminders:** Booking stays the same when something new reacts to visits, like a waitlist. Events are handled in the same process, after the visit is saved. If a handler fails, the visit still stands and the error is logged. Not chosen: Booking calling Reminders directly.
* **Reminders run from a table, not a queue:** About 6,000 reminders a day fit in one PostgreSQL table, checked every minute. Moving a visit is one row update, and `takeDue` locks the rows it takes, so two running copies never send the same reminder. Not chosen: a queue like SQS. It's one more thing to run, and a queued message can't be moved when the visit moves.
* **One channel per provider, behind `MessageChannel`:** Clinics have already asked about WhatsApp. Adding it means one new channel and no change to Reminders. Not chosen: calling Twilio and Postmark from Reminders.
* **Availability is its own module:** The front desk changes hours and blocks time off every week, while booking rules rarely change. Booking only reads open hours, through `OpenHours`. Not chosen: keeping hours inside Booking, as in v2.

## Open questions

* **Patients moving their own visits:** Today they call the front desk. Doing it online needs a link in the reminder and a way for patients to prove who they are, so it waits until clinics ask for it.
* **Reminder timing:** Assumed one reminder, 24 hours before, for every clinic. Some clinics may want 48 hours, or a second one on the morning of the visit.
* **Clinicians at two clinics:** Assumed each clinician works at one clinic. If not, Availability needs hours per clinic.

## Versions

| Version | Date | What changed | Why |
| --- | --- | --- | --- |
| 1 | 2026-10-20 | First design. | |
| 2 | 2026-10-23 | Added text and email reminders: new Reminders and Notifications modules, and Booking now announces booking events. | Clinics lose about one visit in ten to no-shows, and reminders were the first thing they asked for. |
| 3 | 2026-10-28 | Split Availability out of Booking: clinicians, weekly hours, and time off moved to their own module. | The front desk now blocks time off and changes hours every week, while booking rules rarely change. Two reasons to change, so two modules. |
