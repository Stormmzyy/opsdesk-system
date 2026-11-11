# The 15-minute demo

A running order to rehearse with a timer. It covers what the guide asks for: `docker compose up --build` on a clean folder, all four roles and what each can and can't do, the Swagger page with Authorize working, and a green CI run.

**The one thing to remember:** the 403 that comes back when a manager calls `DELETE` with curl, while the Delete button is hidden in the UI. It lands between 7:30 and 9:30, with five and a half minutes to spare.

**Designed around the waiting.** The cold build is about two and a quarter minutes of nothing happening (measured: 2 min 14 s, with a 5 s clone before it). So it starts first, and the talk that has to happen anyway runs over it. Signing in as four people is the other slow part, so nobody signs in during the demo: each person has their own browser profile, signed in beforehand.

## Two profiles: the seam is part of the demo

The same code runs twice, under two profiles, and the switch from one to the other is shown, not apologised for.

| | Clean stack | Development stack |
| --- | --- | --- |
| What | the three fresh clones, `docker compose up --build` | your existing checkouts: the dev backend and Vite |
| Addresses | frontend `localhost:3000`, API `localhost:8080` | frontend `localhost:5173`, API `localhost:8081` |
| Profile | `prod`: no seeded users, no Swagger | `dev`: the six seeded people, twelve tickets, and Swagger |
| Used for | sections 1 to 3, and 8 | sections 4 to 6 |

**What the clean stack doesn't have is the point.**
- **No seeded users** is decision 69 doing its job. The six sample accounts share one password that is published in the README. On a real server, those would be six open doors.
- **That's also why the first admin is made with SQL.** No request to the API can make anyone an admin, not even the first one. So the first admin has to come from someone who already holds the database credentials, not from whoever registers first.
- **No Swagger** is decision 74. A live `/v3/api-docs` would hand a stranger the map of every endpoint.

So the end of section 3 proves it with two curls: the seeded admin can't sign in to the clean stack, and its Swagger isn't there. The same two requests against the development stack then succeed. The difference between the profiles becomes something the audience sees, and it leads straight into the role tour.

The dev backend runs on **8081**, not its usual 8080, because the clean stack's backend takes 8080. Vite is pointed at it when started.

## Before you start

### Once, any day before: four browser profiles

In Chrome: profile icon (top right), **Add**, **Continue without an account**, then name it and pick a colour. Make four:

| Profile | Person | Role |
| --- | --- | --- |
| Fatima | Fatima Bello | USER |
| Daniel | Daniel Mensah | AGENT |
| Kwame | Kwame Asante | MANAGER |
| Priya | Priya Sharma | ADMIN |

**Why profiles:** the frontend keeps the token in `localStorage`, which a browser profile keeps to itself. Four profiles are four people signed in at once, so switching person is switching window. One window with signing out and in would cost about 20 seconds each time, four times.

### 30 minutes before (no earlier: tokens last 60 minutes)

**Every terminal is Git Bash, not PowerShell.** The commands below use `export`, `VAR=value command` and `${!v}`, which PowerShell doesn't have.

**What must already exist on the machine.** Prep doesn't create any of these:
- the `opsdesk-db` container, made once with the backend README's `docker run` command, still holding the seeded data;
- the backend and frontend checkouts, each on an up-to-date `main`;
- the frontend's `node_modules`. If `package-lock.json` has changed since your last `npm ci`, run `npm ci` first. Otherwise `npx vite` fails, or tries to download Vite;
- a JDK 21 at the path in Terminal C;
- the four browser profiles above.

**Terminal A: reset the clean stack.** Run this in your home folder.

```bash
docker compose -p opsdesk down -v
docker image rm opsdesk-backend opsdesk-frontend
docker buildx rm opsdesk-demo
docker buildx create --name opsdesk-demo --driver docker-container
docker buildx inspect --bootstrap opsdesk-demo
rm -rf ~/opsdesk-demo
```

- **`-v` matters.** Every copy of the compose file names its project `opsdesk`, so without it the "clean" folder would reuse the old data volume, and its old accounts.
- **The new builder makes the build cold** without wiping the Docker cache other projects use. It's only there to imitate a machine that has never built OpsDesk. On a real clean machine there's no cache anyway.
- **`--bootstrap` starts the builder now**, about 7 seconds, instead of in the first seconds of section 1. On a machine without the `moby/buildkit` image, this is also where it gets downloaded.
- `docker image rm` and `docker buildx rm` complain if there's nothing to remove ("No such image", "no builder found"). That's fine: the next lines still run.
- **The cold build needs the network.** The new builder doesn't use the images already on this machine, so section 1 downloads the Java, Node and nginx base images from Docker Hub, plus every Maven and npm dependency. The measured 2 min 14 s included all of that, on this connection. Section 1 says what to do if the venue's network is slow.

**Terminal C: the dev backend.** Run this in your opsdesk-backend checkout, on an up-to-date `main`. The `JAVA_HOME` line is the backend README's, under "Requirements": `mvnw` needs a JDK 21. `docker start` fails with "No such container" if `opsdesk-db` was never created. In that case, run the backend README's `docker run` command first, and the seeder fills it when the backend starts.

```bash
docker start opsdesk-db
export JAVA_HOME="/c/Program Files/Eclipse Adoptium/jdk-21.0.4.7-hotspot"
SERVER_PORT=8081 SPRING_PROFILES_ACTIVE=dev ./mvnw spring-boot:run
```

Wait for `Tomcat started on port 8081`.

**Terminal D: Vite, pointed at 8081.** Run this in your opsdesk-frontend checkout, on an up-to-date `main`. Section 4 shows the Users page's permission message, which arrived in opsdesk-frontend#55.

```bash
VITE_API_URL=http://localhost:8081/api/v1 npx vite --port 5173 --strictPort
```

The repo's `.env` says 8080, which during the demo is the clean stack, where none of the seeded people exist. A variable set on the command line wins over Vite's `.env` files.

**Sign the four profiles in.** In each profile, go to `http://localhost:5173`, sign in with the email below and `devpassword123`, then open the page listed:

| Profile | Email | Leave it on |
| --- | --- | --- |
| Fatima | `fatima.bello@opsdesk.com` | `http://localhost:5173/tickets` |
| Daniel | `daniel.mensah@opsdesk.com` | `http://localhost:5173/tickets/1` |
| Kwame | `kwame.asante@opsdesk.com` | `http://localhost:5173/tickets/2` |
| Priya | `priya.sharma@opsdesk.com` | `http://localhost:5173/tickets/2` |

**Terminal B: the tokens, worked out now.** Paste the block below. Each person's token goes into a variable named after them, so nothing is typed during the demo.

The tokens can't be written into this file: each one expires 60 minutes after it's issued, and a token pasted here today would be refused tomorrow.

```bash
API=http://localhost:8081/api/v1
tok() { curl -s -X POST "$API/auth/login" -H 'Content-Type: application/json' -d "{\"email\":\"$1\",\"password\":\"devpassword123\"}" | sed -n 's/.*"token":"\([^"]*\)".*/\1/p'; }
PRIYA=$(tok priya.sharma@opsdesk.com); KWAME=$(tok kwame.asante@opsdesk.com)
DANIEL=$(tok daniel.mensah@opsdesk.com); FATIMA=$(tok fatima.bello@opsdesk.com)
for v in PRIYA KWAME DANIEL FATIMA; do t=${!v}; echo "$v: ${#t} characters"; done
```

Each line should say about 175 characters (measured: 176, 179, 177, 175). A `0` means that login failed: check Terminal C. **Read the four lines, then type `clear` yourself.** Don't put it in the block: it would wipe the check before you could see it.

**Keep Terminal B open until the demo ends.** The four variables, and `API`, exist only in that shell. Sections 3 and 5 use them there.

**A fifth window**, in your normal profile, with three tabs in this order:
1. `http://localhost:8081/swagger-ui.html` (don't press Authorize yet)
2. https://github.com/Stormmzyy/opsdesk-backend/actions/runs/43044400675
3. https://github.com/Stormmzyy/opsdesk-frontend/actions/runs/43060946194

Those are the newest runs on each `main` (backend: #11's merge; frontend: #55's merge). If anything merges before the demo, open the newest run on `main` instead, so nobody asks whether the one shown is current.

**Last checks**, in Terminal B:
- `docker ps` shows no `opsdesk-*` container except `opsdesk-db`.
- `curl -s localhost:8081/actuator/health` says `{"status":"UP"}`.
- Nothing is on 3000 or 8080 yet, for the clean stack: `curl -s -o /dev/null -w '%{http_code}\n' localhost:8080` prints `000`.
- The seeded data is intact, with all 12 tickets: `curl -s -H "Authorization: Bearer $PRIYA" "$API/tickets" | grep -o '"id":[0-9]*,"title"' | wc -l` prints `12`. Anything less means a seeded ticket was deleted in a rehearsal. The seeder won't put it back, so the role tour's tickets may be missing.
- Terminal font is large, and notifications are off.

## The running order

| # | Section | Starts at | Budget |
| --- | --- | --- | --- |
| 1 | Start from nothing | 0:00 | 0:45 |
| 2 | Talk over the build | 0:45 | 2:15 |
| 3 | It's up: one sign-up, and what production leaves out | 3:00 | 1:30 |
| 4 | The four roles | 4:30 | 3:00 |
| 5 | **The 403** | 7:30 | 2:00 |
| 6 | Swagger, with Authorize | 9:30 | 2:00 |
| 7 | A green CI run | 11:30 | 1:30 |
| 8 | Data survives a restart, then questions | 13:00 | 2:00 |

**If the build is slow** and not done by 3:00, go straight on to section 4: the development stack doesn't depend on it. Say "the clean build is still running, so I'll start with the roles on the development profile and come back to it". Come back to section 3 when Terminal A gives its prompt back. The 403 still lands by about 10:00.

### 1. Start from nothing (0:00, 45 s), in Terminal A

Say: "This folder has never seen OpsDesk. Everything comes from the three repositories."

```bash
mkdir ~/opsdesk-demo && cd ~/opsdesk-demo
for r in opsdesk-backend opsdesk-frontend opsdesk-system; do git clone -q https://github.com/Stormmzyy/$r.git; done
cd opsdesk-system
printf 'DB_PASSWORD=%s\nJWT_SECRET=%s\n' "$(openssl rand -hex 16)" "$(openssl rand -base64 48)" > .env
BUILDX_BUILDER=opsdesk-demo docker compose up --build --wait
```

The `printf` line does the README's "copy `.env.example` to `.env` and fill in both values" in one step, with two random values. Say that, so nobody thinks a step was skipped.

**If the network is slow,** with the build still downloading at 3:30, or a pull failing: press Ctrl+C, then run

```bash
docker compose up --build --wait
```

This is the same command without `BUILDX_BUILDER`, so it uses this machine's warm cache. Say so plainly: "the venue's network is slow, so I'm using the cache; the cold build took two and a quarter minutes in rehearsal." Section 3 onwards is unchanged.

### 2. Talk over the build (0:45, 2 min 15 s)

The guide says to be ready to explain these four things, so this is where they go:

- **What each service does.** `db` is PostgreSQL 16, with its data in a named volume, so it outlives the container. It has no published port: only the backend can reach it. `backend` is the Spring Boot API on 8080, built from the backend repo's Dockerfile. `frontend` is nginx serving the built React app on 3000. `--wait` returns only when all three healthchecks pass, and the backend's is Actuator's health, which is UP only when the database answers.
- **Why secrets aren't in Git.** `.env` is ignored. `.env.example` lists every variable with a placeholder. The placeholder `JWT_SECRET` is deliberately too short, so an unedited copy stops the backend at startup instead of running with a published key. Anyone with the real secret can sign a token as anyone.
- **What the build is doing.** Maven runs the backend's build inside the image. The frontend is built with the API's address baked in, which is why `VITE_API_URL` is a build argument and not a setting at runtime.
- **The `prod` profile.** No sample data, no Swagger, and no defaults for secrets. A missing setting stops startup with its name.

Leave CI for section 7.

### 3. It's up, and what production leaves out (3:00, 1 min 30 s)

When Terminal A gives its prompt back:

```bash
docker compose ps
curl -s http://localhost:8080/actuator/health
```

`docker compose ps` shows all three services `Up`, the backend and db `(healthy)`, and the health check says `{"status":"UP"}`.

In the fifth window, open a new tab at `http://localhost:3000`:
1. Click **Create one**.
2. Fill in **Full name**, **Username**, **Email** and **Password** (8 characters or more), with any name you like.
3. Click **Create account**. The login page says "Your account was created. Sign in to continue."
4. Sign in. You land on the Dashboard, and the header shows your name and **User**.

Keep this tab: section 8 uses it.

**Now show what production leaves out.** In Terminal B, try the seeded admin and Swagger against the clean stack, then the same two against the development stack:

```bash
curl -s -w '\nHTTP %{http_code}\n' -X POST http://localhost:8080/api/v1/auth/login -H 'Content-Type: application/json' -d '{"email":"priya.sharma@opsdesk.com","password":"devpassword123"}'
curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://localhost:8080/v3/api-docs
```

```
{"error":"UNAUTHORIZED","message":"The email or password is incorrect."}
HTTP 401
HTTP 401
```

```bash
curl -s -o /dev/null -w 'HTTP %{http_code}\n' -X POST "$API/auth/login" -H 'Content-Type: application/json' -d '{"email":"priya.sharma@opsdesk.com","password":"devpassword123"}'
curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://localhost:8081/v3/api-docs
```

```
HTTP 200
HTTP 200
```

**Say, as you switch:**

> "Same code, two profiles. The build you just watched is production: it has one user, the one I just registered, and no Swagger. That's deliberate. A real server must never ship sample accounts with a published password, or a map of its own API. It's also why the first admin is made with SQL: no request to the API can make anyone an admin. To show the four roles, I'll switch to the development profile, the only place the seeded people exist, designed to hit every cell of the permission table."

### 4. The four roles (4:30, 3 min, about 45 s each)

Go from least power to most.

**Fatima, USER** (her window, on Tickets):
- The list says "Showing 1 to 2 of 2 tickets": only **#4** and **#9**, the two she raised. The other ten exist, but not for her.
- Click **#4**. There's an **Edit** button, but no Assign and no Delete. She may change only the title and description of her own ticket while it's Open.

**Daniel, AGENT** (his window, on #1):
- **#1** is assigned to him: **Edit** only, with no Assign and no Delete.
- In the address bar, go to `http://localhost:5173/tickets/3/edit`. That's #3, which is assigned to Liam.
- Add a word to the **Title**, then click **Save changes**.
- A red notice appears: "Ticket #3 wasn't saved: you don't have permission to make this change…" The page shows the server's 403 properly, and nothing was written.
- Say: "The Edit button isn't hidden here, because whether he may edit depends on who the ticket is assigned to. The server decides."
- Click **Users** in the menu. It says "You don't have permission to see the user list.", with no Retry button, because asking again would get the same answer. The page stays reachable on purpose: hiding it would be the same convenience as hiding a button, and the page would still have to handle the server's refusal.

**Kwame, MANAGER** (his window, on #2):
- **#2**: **Edit**, and an **Assignment** box with **Assign to**. There is **no Delete button**.
- Leave this window here. Section 5 starts from it.

**Priya, ADMIN** (her window, on #2):
- The same ticket, with **Edit**, **Assign** and **Delete ticket**. The only difference from Kwame's view is Delete.
- **Don't click Delete ticket.** The seeder only refills empty tables, so a deleted seeded ticket never comes back.

### 5. The 403 (7:30, 2 min)

**This is the moment.** Put Kwame's window and Terminal B side by side.

Say: "Kwame is a manager. He can't see a Delete button. The question is whether that's the only thing stopping him."

First, prove his token works:

```bash
curl -s -w '\nHTTP %{http_code}\n' -H "Authorization: Bearer $KWAME" "$API/tickets/2"
```

You get his ticket, then `HTTP 200`.

Now the request the UI never offers him:

```bash
curl -s -w '\nHTTP %{http_code}\n' -X DELETE -H "Authorization: Bearer $KWAME" "$API/tickets/2"
```

```
{"error":"FORBIDDEN","message":"You don't have permission to do this."}
HTTP 403
```

Then show it's still there:

```bash
curl -s -o /dev/null -w 'HTTP %{http_code}\n' -H "Authorization: Bearer $KWAME" "$API/tickets/2"
```

`HTTP 200`. Reload Kwame's window too: #2 is still there.

**Say:** "Hiding the button is a convenience. It saves him a click that would fail. The rule lives in the server, and it's checked on every request, for every client: the app, curl, Swagger, or a frontend someone has edited. The role comes from the database, not from the token, and the check runs before the request body is even read."

**If there's time,** one more line shows the other half of the design:

```bash
curl -s -w '\nHTTP %{http_code}\n' -H "Authorization: Bearer $FATIMA" "$API/tickets/2"
curl -s -w '\nHTTP %{http_code}\n' -H "Authorization: Bearer $FATIMA" "$API/tickets/999"
```

Both answer `404` with the same body ("No ticket with id …"). Fatima can't tell a ticket that's hidden from one that doesn't exist, so the API doesn't reveal which ticket ids exist (decision 23).

**Ready if asked:**
- Daniel assigning #1 answers the same 403:

  ```bash
  curl -s -w '\nHTTP %{http_code}\n' -X PATCH -H "Authorization: Bearer $DANIEL" -H 'Content-Type: application/json' -d '{"assigneeId":5}' "$API/tickets/1/assignee"
  ```
- With no token at all, you get `401` "You need to log in first":

  ```bash
  curl -s -w '\nHTTP %{http_code}\n' -X DELETE "$API/tickets/2"
  ```

### 6. Swagger, with Authorize (9:30, 2 min)

In the fifth window, tab 1 (`http://localhost:8081/swagger-ui.html`):

1. Point at the text under the title. Swagger is generated from the running code. The rules it can't show (status codes, who may do what, the check order) are in `API_CONTRACT.md`, and where they differ, the contract wins.
2. Click **GET /api/v1/tickets**, then **Try it out**, then **Execute**. You get **401**, because there's no token yet.
3. In Terminal B, copy Fatima's token to the clipboard:
   ```bash
   printf %s "$FATIMA" | clip
   ```
4. Click **Authorize** (top right). Paste into **Value** with Ctrl+V, the token alone, since Swagger adds "Bearer" itself. Click **Authorize**: it now says **Authorized**. Click **Close**. The padlocks are now closed.
5. Click **Execute** again. You get **200**, with only tickets **4** and **9**: the same rule as her window, through a different client.
6. Point at the **Curl** box: Swagger shows the exact request it sent, `Authorization: Bearer …` included.

Say: "This page exists only under the development profile. In production, the Swagger paths answer exactly like a path that doesn't exist, so nobody can tell they're there."

### 7. A green CI run (11:30, 1 min 30 s)

Tab 2, the backend run:
- Two jobs, **verify** and **image**, both green.
- Click **image**, then the step **Start the image without settings, and expect it to refuse**. The log ends with `DB_URL is not set.`
- Say: "Building an image proves nothing about whether it runs. This starts it, and passes only if it refuses to run unconfigured."

Tab 3, the frontend run:
- **check** (lint, 101 tests and the build) and **image** (the nginx config tested inside the image), both green.

Say: "Branch protection on `main` requires these checks to pass, so nothing reaches `main` red, my own pushes included."

### 8. Data survives a restart, then questions (13:00, 2 min)

In Terminal A:

```bash
docker compose down
docker compose up --wait
```

This takes about 17 seconds. Use **`down` without `-v`**, which keeps the volume.

Back on the `localhost:3000` tab from section 3, reload and sign in again with that account. It's still there, because the database lives in the volume, not in the container.

Then questions.

## Don't, during the demo

- **Don't click Delete ticket in Priya's window**, and don't run `DELETE` with `$PRIYA`. Either would really delete a seeded ticket.
- **Don't use `docker compose down -v` in section 8.** It deletes the account you're about to sign in with.

## Afterwards

```bash
docker compose -p opsdesk down -v
docker buildx rm opsdesk-demo
```

Then stop Terminals C and D with Ctrl+C.
