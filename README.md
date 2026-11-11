# OpsDesk

OpsDesk is an internal operations desk. Staff raise support tickets, move them through a workflow (Open → In progress → Resolved → Closed), and see what each team is working on.

It is made of two applications, each in its own repository:

- [opsdesk-backend](https://github.com/Stormmzyy/opsdesk-backend): a Spring Boot REST API backed by PostgreSQL.
- [opsdesk-frontend](https://github.com/Stormmzyy/opsdesk-frontend): a React 19 + TypeScript app. It also holds [API_CONTRACT.md](https://github.com/Stormmzyy/opsdesk-frontend/blob/main/API_CONTRACT.md), the contract the two are built against.

## What this repository is for

This repository holds what belongs to the system as a whole rather than to either application: the Docker Compose file that runs the database, backend and frontend together, and the main documentation. Clone it next to the other two, so the three sit side by side:

```
Wirepick/
  opsdesk-backend/
  opsdesk-frontend/
  opsdesk-system/
```

The full README, with how to run everything, comes later this week.

## Development password

**`devpassword123` is seeded development data.** It is the password of the six sample users that the backend creates in a local development database. It is not a real password, and it must never be used anywhere else.
