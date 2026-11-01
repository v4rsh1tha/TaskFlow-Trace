<h1 align="center">🧠➡️✅ TaskFlow Trace</h1>

<p align="center">
  <b>Dump your messy brain-notes. Get a tidy to-do list. Then watch the app show its homework! 📝✨</b>
</p>

<p align="center">
  <a href="https://task-flow-trace.replit.app"><img src="https://img.shields.io/badge/🚀_Live_demo-task--flow--trace.replit.app-4fd0e6?style=for-the-badge" alt="Live demo"></a>
  <a href="https://github.com/YOUR-USERNAME/taskflow-trace"><img src="https://img.shields.io/badge/💻_Source_code-GitHub-8b5cf6?style=for-the-badge" alt="Source code"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black" alt="React">
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript">
  <img src="https://img.shields.io/badge/Express-111111?style=flat-square&logo=express&logoColor=white" alt="Express">
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/Drizzle-C5F74F?style=flat-square&logo=drizzle&logoColor=black" alt="Drizzle">
  <img src="https://img.shields.io/badge/OpenAI-10A37F?style=flat-square&logo=openai&logoColor=white" alt="OpenAI">
  <img src="https://img.shields.io/badge/Replit-F26207?style=flat-square&logo=replit&logoColor=white" alt="Replit">
</p>

<p <img width="1344" height="896" alt="image" src="https://github.com/user-attachments/assets/d0ca6bb8-6769-480b-b9dc-2a80cdfcfc47" />

</p>
<p align="center"><sub>🎨 Overview illustration of a run with a pretend database hiccup. Open the live app to see a real one!</sub></p>

---

## 🎬 Try it in 30 seconds

1. 🚪 Open the **[live app](https://task-flow-trace.replit.app)** and hit **Explore the live demo**
2. ✍️ Go to **Smart Organizer** and use the example note
3. 💥 Flip on **Simulate DB Drop** (yes, we break things on purpose!)
4. ▶️ Press **Generate trace**
5. 🔍 Open **Execution Studio** and smash **Replay Run**

### 📸 A real run

<!-- Replace with your own screenshot of Execution Studio after a real run -->
<p align="center">
  <img src="./docs/real-run.png" alt="How it works diagram after a real run" width="100%">
</p>

---

## 🪄 What does it do?

You write something like:

> *"I have a Java interview on Monday, complete my React project by Friday, update my resume today, and apply to five companies."* 🤯

And TaskFlow Trace turns it into:

- ✅ **Clean tasks** with a title, deadline, priority and confidence score
- 🚩 **Missing-info flags** (no deadline? It says so instead of guessing!)
- 🔗 **Dependencies** between tasks
- 🙋 **You stay in charge.** Tasks wait as *pending* until you approve, edit, complete or delete them

<p align="center"><img src="./docs/02-smart-organizer.png" alt="Smart Organizer with a sample note, failure simulation toggles and extracted tasks" width="90%"></p>

---

## 🗺️ The six steps of every run

Every run walks through the same six steps. Those steps are the real events the server sends **and** the boxes on the diagram, so they can never disagree. 🤝

| # | | Step | Who does the work | What happens |
|---|---|------|-------------------|--------------|
| 1 | 📥 | **Input Received** | 🌐 Page → Server | Your note travels to the server |
| 2 | 🛡️ | **Validation** | 🖥️ Server | The note is checked against the request schema (Zod) |
| 3 | 🤖 | **AI Task Extraction** | ☁️ Outside service | An OpenAI model (`gpt-5-nano`) pulls out tasks, and its JSON is validated |
| 4 | 🎯 | **Prioritization & Scoring** | 🖥️ Server | Duplicates merged, priority and confidence assigned |
| 5 | 💾 | **Save Tasks** | 🗄️ Database | Everything is saved in one Postgres transaction |
| 6 | 🎉 | **Response Rendered** | 🖥️ Server → Page | Tasks and trace come back to your screen |

Each step records `started` and then `succeeded`, `failed`, `retrying` or `recovered`. The server pushes them to your browser live using **Server-Sent Events**. ⚡

⏱️ **Honest timings:** durations are measured with `performance.now()`. We **never** add fake delays to the pipeline. If a run is too fast to follow, the replay slows down instead (0.25x, 0.5x, 1x or 2x). 🐢🐇

<p align="center"><img src="./docs/03-live-execution-diagram.png" alt="Six-step How it works diagram with durations and replay speeds" width="90%"></p>

---

## 🛟 How it handles failures

Things break. We show it on screen and fix it, instead of hiding it. 🙈➡️👀 A **Failure Recovery Simulation** panel lets you break things on purpose and watch the diagram react.

| 💥 What goes wrong | 🦸 What the app does | 🏁 Final status |
|---|---|---|
| 🤖 **AI error or bad response** (empty, broken or invalid JSON) | Logs `failed`, switches to a rule-based backup extractor, logs `recovered`, and carries on | 🟡 `recovered` |
| 🔌 **Temporary database drop** | Logs `failed`, then `retrying` (attempt 2), retries the save, logs `recovered` | 🟡 `recovered` |
| 🧱 **Hard database outage** | Logs `failed` (`retryable: false`), saves nothing, and still shows you a clear failure | 🔴 `failed` |
| 🚫 **Invalid input** | Turned away with a `400` before the run even starts, and checked again in Validation | 🔴 `400` / `failed` |
| 🌀 **Dependency loop in AI output** | The save step checks the task graph has no cycles, and a loop rolls the whole save back | 🔴 `failed` |
| 😱 **Unexpected crash** | A safety net marks the run `failed` and records a final event | 🔴 `failed` |

### 🧰 Extra safety nets

- ⚛️ **All-or-nothing saves:** tasks and dependencies go in one transaction, so no half-saved mess
- 🕵️ **No made-up deadlines:** if you didn't give one, it says *Unspecified* and lists "Exact deadline" as missing
- 🛡️ **Prompt-injection guard:** the model is told to ignore sneaky instructions hidden in notes
- 📡 **Resumable live stream:** the event stream remembers where it stopped (`Last-Event-ID`) and sends heartbeats, so a dropped connection doesn't lose or repeat events
- 🧬 **Duplicates handled gently:** merged inside one extraction, linked (not deleted) across runs, so history stays intact
- 🔒 **Private by design:** every query is scoped to the current user

<p align="center"><img src="./docs/05-failure-recovery-event-stream.png" alt="Failure recovery: failed, retrying and recovered events with the recovery log" width="90%"></p>

---

## 🎁 Features

| 🧭 Page | 🎈 What you get |
|---|---|
| **Command Center** & **Smart Organizer** | Paste notes, arm a failure if you're feeling spicy 🌶️, and run |
| **Execution Studio** | The live "How it works" diagram, a payload inspector (input and output JSON of any step) and a recovery log |
| **Replay** | Replays recorded events using their real time gaps, at 0.25x–2x speed |
| **Execution History** | Every past run with status, duration, source and task count, all replayable |
| **Task Dashboard** | Stats, a dependency graph, and approve / edit / complete / delete |
| **Sign in** | Clerk accounts, or a one-click isolated **demo session** (30 days). The demo comes pre-loaded with sample runs marked `seeded`, and runs you start are 100% real |

<p align="center">
  <img src="./docs/01-branding-sidebar.png" alt="TaskFlow AI branding and sidebar" width="48%">
  <img src="./docs/04-task-dependency-flow.png" alt="Task dependency flowchart" width="48%">
</p>

---

## 🏗️ Under the hood

```
 🌐 Browser (React + Vite)  ──HTTP──▶  🖥️ Express 5 API  ──▶  🗄️ PostgreSQL (Drizzle ORM)
          ▲                                  │
          └────── ⚡ SSE: live run events ───┤
                                             └──▶  🤖 OpenAI model (task extraction)
```

- 🎨 **Page:** React, Vite, Tailwind, shadcn/ui, TanStack Query, wouter, Clerk
- ⚙️ **Server:** Express 5, TypeScript, Pino logging
- 🗄️ **Database:** PostgreSQL + Drizzle ORM (`users`, `tasks`, `task_dependencies`, `execution_runs`, `execution_events`)
- ☁️ **Outside services:** OpenAI via Replit AI Integrations, and Clerk for auth
- 📜 **One contract:** an OpenAPI spec generates the typed React Query client and the Zod schemas with Orval, so page and server always agree
- 📦 **Monorepo:** pnpm workspaces, Node.js 24, TypeScript 5.9

<details>
<summary>📂 <b>Project layout</b> (click to open)</summary>

```
artifacts/
  api-server/        Express API: routes, run engine, SSE bus, auth middleware
    src/lib/run-engine.ts         the six-step pipeline and failure handling
    src/lib/task-intelligence.ts  AI extraction, fallback extractor, scoring
    src/lib/run-bus.ts            in-memory pub/sub feeding the SSE stream
  taskflow-ai/       React front end (diagram, replay, dashboard)
lib/
  api-spec/          OpenAPI spec + Orval codegen
  api-zod/           generated Zod schemas
  api-client-react/  generated React Query hooks
  db/                Drizzle schema and connection
  task-utils/        title normalization, dedup, deadline text (with tests)
  integrations-openai-ai-server/  OpenAI client
```

</details>

<details>
<summary>🔌 <b>API routes</b> (click to open)</summary>

| Method | Route | Purpose |
|--------|-------|---------|
| `POST` | `/api/runs` | Start a run (returns `202`, work continues on the server) |
| `GET` | `/api/runs` | List past runs |
| `GET` | `/api/runs/:runId` | Run detail with all events and tasks |
| `GET` | `/api/run-events?runId=` | Live SSE stream (resumable) |
| `GET` `POST` `PATCH` `DELETE` | `/api/tasks…` | List, create, edit, approve, complete, delete tasks |
| `GET` | `/api/dashboard/summary` | Dashboard stats |
| `POST` `DELETE` | `/api/demo/session` | Start or end the demo session |
| `GET` | `/api/healthz` | Health check |

</details>

---

## 🧑‍💻 Run it yourself

Don't feel like setting things up? Just use the hosted version: **https://task-flow-trace.replit.app** 🎈

Otherwise you need Node.js 24, pnpm and a PostgreSQL database.

```bash
pnpm install

# 🔑 Environment
#   DATABASE_URL                      Postgres connection string
#   AI_INTEGRATIONS_OPENAI_BASE_URL   OpenAI-compatible endpoint
#   AI_INTEGRATIONS_OPENAI_API_KEY    API key for the endpoint
#   CLERK_SECRET_KEY, CLERK_PUBLISHABLE_KEY, VITE_CLERK_PUBLISHABLE_KEY  (auth)

pnpm --filter @workspace/db run push              # 🗄️ create tables
pnpm --filter @workspace/api-server run dev       # 🚀 start the API
pnpm --filter @workspace/task-utils run test      # 🧪 run unit tests
pnpm run typecheck                                # ✅ typecheck everything
pnpm --filter @workspace/api-spec run codegen     # 🔄 regenerate API client + schemas
```

On Replit, the database and AI integration are set up for you, and the app is published with Autoscale. 🌈

---

## 🔮 What's next?

- 📅 **Calendar sync:** push approved tasks to Google Calendar, shown as a new "Outside service" step
- 🔁 **Smarter retries:** exponential backoff and a retry budget, plus one AI retry before falling back (today the AI step falls back right away and the database step retries once)
- 🗓️ **Real dates:** turn "Friday" into an actual due date in your time zone, and sort by it
- 🔗 **Shareable traces:** a read-only link so anyone can watch a replay
- 📬 **Sturdier event delivery:** swap the in-memory event bus for Postgres `LISTEN/NOTIFY` or Redis, so live streams work across many servers
- ⚖️ **Run comparison:** two runs side by side to spot where timings or failures differ
- 🧪 **More tests:** end-to-end tests for every simulated failure

---

<p align="center">
  Made with ☕, 🧠 and a healthy fear of things breaking.<br>
  <sub>🎨 Pictures in <code>docs/</code> are illustrations based on the app's design and a sample run. <code>docs/real-run.png</code> is a screenshot from the live app.</sub>
</p>
