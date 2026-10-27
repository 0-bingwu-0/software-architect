# My Profile

Read this before designing or checking a system. Below is an example setup. Say "Set me up." in Chat to replace it with yours, or edit it directly.

## About me

I'm a full-stack engineer at a 12-person startup that builds scheduling software for small clinics. I design most new features myself, then build them with Claude Code.

## Stack

* TypeScript and Node.js with Express on the backend, React on the frontend.
* PostgreSQL, through Prisma.
* Runs on AWS: one container service and a managed database.

## House rules

* One deployable service until there's a clear reason to split. No microservices.
* Business logic never imports Prisma or Express directly.
* Name modules after what they do for the business, like `booking`, not after layers, like `services`.

## Doc settings

* **Language:** English

## Avoid

* Patterns added "for flexibility" with no change in sight.
* Words like "robust", "scalable", and "seamless" in place of saying what something does.
