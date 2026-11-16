# CCIFJ Concierge

An AI concierge for the **Chambre de Commerce et d'Industrie France Japon (CCIFJ)**.
Website visitors ask it about CCIFJ events, membership, business support, recruitment and
training, in English, French or Japanese. It answers from CCIFJ's own content, cites its
sources, and can take real actions, such as booking an event or writing a request to CCIFJ
staff, but only after the user approves them. Staff follow up and keep it current from a
dashboard, without coding.

Built by **Team Irbid** at the 42 Asia Hackathon 2026, with CCIFJ as the partner organisation.

### Highlights

- **Answers and acts:** books events, reserves the CCIFJ room, changes or cancels bookings,
  matches members with partner companies and drafts emails to the right staff contact, each
  after the user approves it.
- **Adapts to the user:** visitors, signed-in users, members and staff each get the features
  that fit their role.
- **Grounded and multilingual:** answers in English, French or Japanese from CCIFJ's website
  and approved documents, with the sources shown under every answer.
- **Run by staff, not developers:** one dashboard for requests and their conversations,
  matching and introductions, member engagement, events, member companies and knowledge
  files; linked sources are kept up to date automatically.
- **Fits CCIFJ's tools:** one `<script>` tag adds it to the TYPO3 website, and the same agent
  runs inside Prisme.ai.
- **Tested:** 19 of 20 realistic scenarios pass, all 5 fallback cases are handled safely, and
  275 backend and 48 frontend automated tests pass.

| I want to… | Go to |
|---|---|
| Run it on my machine | [Quick start](#quick-start) |
| Try it as a visitor, a member or staff | [How to use it](#how-to-use-it) |
| See what it can do | [What the concierge can do](#what-the-concierge-can-do) |
| See how it was tested | [Quality and testing](#quality-and-testing) |
| Understand how it is built | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) |
| Install it for CCIFJ, and see the costs | [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) |
| Use it from Prisme.ai | [From Prisme.ai](#from-prismeai) |
| Follow a guided walkthrough | [docs/HACKATHON.md](docs/HACKATHON.md) |

---

## Quick start

### What you need

- Python **3.12 or later** and [uv](https://docs.astral.sh/uv/)
- Node.js and npm

- An OpenAI API key (or local models through Ollama)

### Start it

1. Get the code:

   ```bash
   git clone https://github.com/mohammedadel2023/CCIFJ-Concierge.git
   cd CCIFJ-Concierge
   ```

2. Create your settings file from the example, then fill in `OPENAI_API_KEY` and
   `CCIFJ_AUTH_SECRET` (the example file shows how to generate the secret):

   ```bash
   cp .env.example .env
   ```

3. Start everything:

   ```bash
   ./start.sh
   ```

   The script installs the dependencies, prepares the database and starts the API on port 8001
   and the website on port 5173. It is a bash script: use macOS, Linux, WSL or Git Bash.

4. Open **http://localhost:5173**.

The agent loads its models in the background when the API starts (about 12–18 seconds, longer
the first time because the search model is downloaded). A message sent before that simply waits.

### Optional: OCR for scans and images

**You can skip this step.** The chat, the three agents and the staff dashboard all work without
it. OCR is used for one thing only: reading scanned PDFs and images that a visitor attaches or
staff upload. Text PDFs, Word files and text files are read without it.

To add it, run one script. It is safe to run again, and it never changes the concierge itself.

macOS, Linux, WSL or Git Bash:

```bash
bash scripts/install_ocr.sh
```

Windows PowerShell:

```powershell
powershell -ExecutionPolicy Bypass -File scripts\install_ocr.ps1
```

What the script does:

1. Installs the Tesseract program with the package manager your system has (Homebrew, apt, dnf,
   yum, pacman, zypper or apk; winget, Chocolatey or Scoop on Windows).
2. If English, French and Japanese are not all available, downloads those three language files
   (under 10 MB) from the official Tesseract repository into `var/tessdata` inside the project.
   This needs no admin rights and works the same on every system.
3. Checks the result and tells you whether OCR is ready.

To see what is installed without changing anything, add `--check` (or `-Check` in PowerShell).
On Windows you do not need to edit the PATH: the concierge also looks in the usual install
folders. Restart the concierge after installing.

If the script cannot install the program, for example because the machine has no package
manager or you have no admin rights, it says so and prints what to do by hand. Nothing else is
affected: the concierge keeps working, and a scan or image simply gets a message that OCR is
unavailable.

### Start it by hand (or in Windows PowerShell)

Create `.env` first, as in step 2 above.

```bash
uv sync
uv run concierge-db upgrade
uv run uvicorn concierge.main:app --app-dir src --reload
```

In a second terminal:

```bash
cd ccifj-frontend
npm ci
npm run dev
```

Open **http://127.0.0.1:5173**. Started this way, the API listens on port 8000.

### Demo accounts

You can chat without an account. To see the member and staff sides, sign in with one of these.
All four use the password `ccifj-demo-2026`.

| Account | What it shows |
|---|---|
| `staff@demo.ccifj.test` | The staff dashboard |
| `claire.member@demo.ccifj.test` | A member with a booking and the newsletter |
| `kenji.member@demo.ccifj.test` | A second member |
| `lucas.guest@demo.ccifj.test` | A signed-in non-member |

The database in the repository already contains them. On a fresh database, create them with
`uv run concierge-db seed-demo`. They have a published password: delete them before the agent
is opened to the public.

### If something goes wrong

| What you see | What to do |
|---|---|
| `Port … is already in use` | Stop the other program, or choose other ports: `API_PORT=8002 WEB_PORT=5174 ./start.sh`. |
| The first answer takes a long time | The models are still loading. `http://localhost:8001/api/v1/ready` answers once they are ready. |
| "OCR is unavailable" on a scanned PDF or image | Run the [optional OCR script](#optional-ocr-for-scans-and-images). Text PDFs and Word files work without it. |
| No **Dashboard** button after signing in | Only staff accounts have it. Sign in as `staff@demo.ccifj.test`. |

---

## How to use it

### As a website visitor

1. Open the site and pick your language at the top right: EN, FR or 日本語.
2. Ask a question in your own words, for example *"Can you help my company find customers in
   Japan?"*. For advice, the concierge may first ask about your company, once; you can skip it.
3. Read the answer. The pages and documents it used are listed under it as **Sources used**,
   and suggested next questions appear under **You could also ask**.
4. Ask it to act, for example *"Help me write an email to the Business Support team."* It asks
   which language the email should use, then shows the full draft in an **Email draft** card.
5. Ask for changes until the draft is right, then tell it to send. An approval card shows
   exactly what will be stored. Choose **Store my message** and you get a reference such as
   `SR-2026-000001`. Nothing is stored unless you approve.
6. Ask about events, for example *"What events are coming up?"*. Booking a ticket, reserving
   the room and subscribing to the newsletter need a sign-in (**Sign in**, top right); your
   conversation continues after you sign in.

### As a member

Sign in as `claire.member@demo.ccifj.test`, then try:

- *"What are my bookings?"* to see your bookings and change or cancel one.
- *"Find a French food company with experience in Japanese distribution."* to get matching
  member companies with the reasons they fit, then ask for an introduction. CCIFJ staff receive
  the request and contact the other company first.
- Any email draft: it is already signed with your name, email and company.

### As CCIFJ staff

Sign in as `staff@demo.ccifj.test` and click **Dashboard** in the header. Nothing here needs
code, and the agent uses every change in its next answer.

| I want to… | Where | What to do |
|---|---|---|
| Follow up a visitor's request | **Requests** | Open **Read message** and **Read the conversation**, then set the status to New, In review or Closed. |
| See what visitors ask | **Requests → Conversations** | Open any conversation to read the questions, the answers and the sources cited. |
| Add a brochure or price list | **Knowledge files → Add files** | Choose one or more files (PDF, Word, text, image) and press **Upload and index**. |
| Keep a Calameo publication or web page up to date | **Knowledge files → Link a source** | Paste the link and press **Add link**. It is re-read every few hours; **Sync now** re-reads it immediately. |
| Add or change an event | **Events** | Edit the event, or import a spreadsheet (a template is provided). |
| Withdraw an event | **Events** | Press **Cancel event**. The agent stops listing and booking it; its bookings are kept, and **Restore** brings it back. |
| Update the member directory | **Member companies** | Import the directory export, or link a person's account to their company. |
| Take a company out of the directory | **Member companies** | Press **Hide**. The agent stops showing it in search and matching; **Show** brings it back. |
| Change who receives requests | **Staff directory** | Add or edit a contact and the topics they handle. |
| Find companies for someone's need | **Matching & intros → Find matches** | Describe the need, press **Search**, select companies, and review the drafted introduction. |
| Welcome and follow up members | **Onboarding & automation** | Run the rules for today, or for a later date to preview them, then open the outbox. |
| Give a colleague staff access | **Accounts & roles** | Change the account's role to staff. |

### On the CCIFJ website

One line in the page template adds a chat button to any page, TYPO3 included:

```html
<script src="https://your-concierge-address/embed.js" defer></script>
```

`ccifj-frontend/public/embed-demo.html` is an example page. Installation in CCIFJ's
environment is described in [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md).

### From Prisme.ai

The concierge also runs as an agent in a Prisme.ai workspace, where CCIFJ staff already work.
The Prisme agent holds no CCIFJ facts of its own: one custom function sends each question to
this backend and shows the reply, so answers are the same as on the website and are updated in
one place.

| What it does | Details |
|---|---|
| Answers in three languages | English, French and Japanese, each in the language of the question. |
| Keeps the conversation | Follow-up questions ("and what is her phone number?") stay in the same conversation, because the agent passes the conversation ID back to the backend. |
| Cites the same sources | The reply comes from the backend, with the same sources as the web chat. |
| Can be reused by another chamber | The exported agent, [`prisme/CCIFJ_Concierge.agents.md`](prisme/CCIFJ_Concierge.agents.md), imports in one step; only the backend address has to be set. |

To try it with the backend on your machine:

1. Start the concierge with `./start.sh`.
2. Give it a public address (needs [`cloudflared`](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/), no account):

   ```bash
   prisme/tunnel.sh
   ```

   It prints the **Endpoint URL**. Run it again to check that the address still answers;
   `prisme/tunnel.sh stop` closes it.
3. In Prisme, **Agent Creator → Create agent → Import AGENTS.md** with
   `prisme/CCIFJ_Concierge.agents.md`, then paste the Endpoint URL into the function
   **Ask the CCIFJ Concierge**.
4. In the **Playground**, ask *"What are the membership fees?"*. "Ran 1 tool" above the answer
   means the backend was called.

Set-up by hand, a fixed address and the tests run in Prisme are in
[prisme/README.md](prisme/README.md).

---

## What the concierge can do

| Area | What it does |
|---|---|
| **Events** | Lists upcoming events (by topic keywords, name, format, city or region such as Kansai, or dates), shows full event details, books tickets for every attendee, and reserves the L'Espace@CCIFJ room. Booking requires login and approval. |
| **My bookings** | Shows the user's own bookings and room reservations. Before booking, checks whether registration is open, whether they already booked, whether it clashes with their schedule, and whether the room is free (suggesting free slots when it isn't). Cancels a booking or reservation, or changes it (another event, add or remove attendees, new dates) in one step, until it starts. Changes need login and approval. |
| **Membership** | Explains membership categories, fees, eligibility, and benefits, shares the application form, and searches the member directory. The directory holds each company with its primary and secondary sectors and its **contact people** (name, job title, email, working language), imported from a CCIFJ directory export. Sector names are understood loosely ("F&B", "food", "agroalimentaire" all reach Food & Beverage). The sample directory holds 50 companies and 116 contacts. |
| **Business support** | Knows the full CCIFJ offer across the 8 sections of ccifj.or.jp (Our Chamber, News, Membership, Events, Business Forum, Commercial Support, Recruitment/Training, Publications) and recommends the services that fit the user's stage: market studies, Business Connect, Expo+, sector missions, company setup, domiciliation, V.I.E, VAT recovery, recruitment, training, publications, and advertising. |
| **Contacts** | Reads the staff directory that CCIFJ maintains in the admin dashboard (department, what each contact handles, email, languages), picks the right contact for the request itself, falls back to the triage contact, and always offers to write the email together. A contact added in the dashboard is used on the next question; no code or topic list to update. |
| **Staff requests** | Writes an email to a CCIFJ contact, or a ticket for the CCIFJ team when no contact fits. It revises the draft with the user and stores it after the user approves. Staff read each request, and the conversation behind it, in the dashboard (**Requests → Conversations**). |
| **Member matching** | Finds the member companies that fit a business need (sector, country, experience) and shows the directory evidence for each match. A member can then ask CCIFJ for an introduction: the request is stored for staff with a reference such as `IR-2026-000012`, and staff contact the other company first. |
| **Member engagement** | Rules welcome new members, suggest a first step, relevant events and people to meet, and follow up with members who have gone quiet. Each message explains why it fits that member and appears in the member's corner of the app; staff review the outbox in the dashboard. |
| **Staff dashboard** | One place for staff to follow up and keep the agent current without coding: requests and the conversations behind them, matching and introductions, onboarding rules, events, member companies, knowledge files and links, accounts and the staff directory. See [Staff dashboard](docs/ARCHITECTURE.md#staff-dashboard). |
| **Knowledge that stays current** | Staff add brochures and procedures by upload, or link a Calameo publication, a web page or a PDF once: linked sources are re-read on a schedule and re-indexed when they change. |
| **Newsletter** | Subscribes the user to general news, training announcements, or the @StudioCCIFJ podcast, after approval. |
| **Files** | Reads attached PDFs, images (PNG, JPEG), Word documents (DOCX), and text files, with OCR in English, French, and Japanese for scans and images. Up to 4 files per conversation, 20 MB each. The text goes to the agent as context; the chat only shows the file name. |
| **Personal touch** | Asks for the details that change its advice (industry, country, B2B or B2C, stage) before answering an advisory question, once, and lets the user skip. Knows the date and the signed-in user's bookings, membership (with the company's industry and location), and newsletter status. Remembers the last few exchanges, so it builds on earlier advice instead of repeating it. |
| **Web search** | With an OpenAI model, the events and general agents can search the web for market context (competitors, market size, trade shows) and recent CCIFJ news. CCIFJ facts (fees, events, members, contacts) still come only from the tools. Switch it off with `CCIFJ_WEB_SEARCH=off`. |
| **Embedding** | `ccifj-frontend/public/embed.js` adds a chat button to any page (TYPO3 included) with one `<script>` tag; `embed-demo.html` is an example host page. |
| **Prisme.ai** | A published Prisme agent passes each question to this backend through one custom function, so the concierge can be used inside CCIFJ's Prisme workspace with the same answers as on the website. The exported agent is in [`prisme/`](prisme/README.md) for another chamber to import. |
| **Safety** | Keeps to CCIFJ topics and refuses unrelated requests. It treats tool results and documents as data, not instructions. Every database write needs explicit approval. It never shows the user its internals (databases, tools, event IDs). |

---

## Quality and testing

- **Real-world scenarios:** on a bank of 20 realistic CCIFJ scenarios, 19 pass and 1 is partly
  answered; all 5 fallback cases (missing or conflicting information, unknown member,
  unavailable event, unsupported request) are handled safely, and every answer cites at least
  one validated source. The questions were asked in English, French and Japanese, and each was
  answered in its own language.
- **Automated tests:** 275 backend tests and 48 frontend tests pass. Run them with
  `uv run python -m unittest discover -s tests -p 'test_*.py'` (backend, no API keys needed) and
  `cd ccifj-frontend && npm test`. See [Testing](#testing).
- **Sources:** answers cite the crawled ccifj.or.jp pages and the documents staff added; the chat
  shows them under each answer, and staff see them in the conversation view.
- **Prisme.ai:** the published Prisme agent answers in English, French and Japanese through the
  backend and keeps follow-up questions in the same conversation. The exported agent imports
  into a new workspace and works once its backend address is set. See
  [prisme/README.md](prisme/README.md).

---

## Built with

**Backend:** Python, FastAPI, SQLModel, LangChain and LangGraph, Chroma and bm25s (hybrid
search), sentence-transformers (`intfloat/multilingual-e5-small`, run locally), pypdf, pypdfium2
and Tesseract OCR.
**Frontend:** React and Vite, plus an embeddable chat widget.

| Service | Used for | Required |
|---|---|---|
| OpenAI API | The chat model, routing, and optional web search | Yes, unless local Ollama models are used |
| Cohere Rerank | Re-ordering search results | No; search falls back to hybrid ranking |
| Hugging Face Hub | One-time download of the embedding model, which then runs locally | Yes, on first start |
| LangSmith | Tracing during development | No; off by default |
| Odoo | Creating a CRM lead for each approved request | No; off by default |
| Calameo | Reading linked publications | No; only when staff link a publication |
| Prisme.ai | Hosting the Prisme agent, which sends each question to this backend | No; only for the Prisme agent |

**Sample data.** The member companies, contacts and accounts in the repository are sample data.
The knowledge base is built from the public ccifj.or.jp website and CCIFJ's source documents.
Requests to staff and introductions are stored for staff to handle in the dashboard, and member
follow-ups go to an outbox that staff review.

---

## Configuration

Settings come from `.env` and are checked at startup (`src/concierge/config.py`). Create it from
`.env.example`, which lists every setting with sensible defaults; this table describes each one.

| Variable | Required | Description |
|---|---|---|
| `CCIFJ_AUTH_SECRET` | yes | Secret for signing login tokens. |
| `model_type` | yes | `openai` (default) or `qwen` for local Ollama models. |
| `model_name` | yes | Main chat model. |
| `router_model_name` | yes | Model for routing and the final policy check. |
| `OPENAI_API_KEY` | with `openai` | OpenAI key. |
| `COHERE_API_KEY` | no | Cohere reranking; missing keys or reranker failures fall back to hybrid fused results. |
| `CCIFJ_STAFF_USER_IDS` | no | Comma-separated existing account IDs allowed to view the protected staff inbox. Empty disables access. |
| `trim_messages_max_token` | no | Conversation history kept per turn (default 1000 tokens). |
| `CCIFJ_AGENT_TIMEOUT_SECONDS` | no | Maximum time for one agent turn (default 180). |
| `CCIFJ_CORS_ORIGINS` | no | Comma-separated frontend origins allowed to call the API. |
| `CCIFJ_LINK_SYNC_HOURS` | no | How often linked knowledge sources (Calameo, web pages, PDFs) are re-read; default 6. `off` disables the schedule; **Sync now** in the dashboard still works. |
| `CALAMEO_API_KEY`, `CALAMEO_API_SECRET` | no | Calameo API keys (Premium/Platinum accounts). When set, linked Calameo publications are read through the official API instead of the public page, which also covers private publications. |
| `CCIFJ_WEB_SEARCH` | no | `on` (default) or `off`: OpenAI's built-in web search for the events and general agents, used for market context and recent CCIFJ news. Ignored with Ollama models. |
| `CCIFJ_WEB_SEARCH_DOMAINS` | no | Comma-separated domains the web search may use (e.g. `ccifj.or.jp`); default `*` is the whole web. |
| `ODOO_URL`, `ODOO_DB`, `ODOO_USER`, `ODOO_API_KEY` | no | When all four are set, every staff request a visitor approves is also created in Odoo CRM as a lead (`src/concierge/integrations/odoo.py`; JSON-2 API on Odoo 19+, XML-RPC on older versions). Off by default; an Odoo outage never fails the answer. `ODOO_TIMEOUT` (seconds, default 10). |
| `LANGSMITH_TRACING`, `LANGSMITH_API_KEY`, … | no | LangSmith tracing. Traces go to the project `<LANGSMITH_AGENT_ID>-<LANGSMITH_AGENT_ENVIRONMENT>` (e.g. `realhackathon-local`). Runs are listed by their start time, taken from this machine's clock: keep it synced, or runs recorded while it was wrong sort above newer ones. |

Frontend settings (`ccifj-frontend/.env.local`): `CCIFJ_API_TARGET` for the
development proxy, and optionally `VITE_API_BASE_URL`. Never put API keys in
frontend variables.

---

## Testing

The contract tests use a fake agent and a throwaway database, so they call no model:

```bash
uv run python -m unittest discover -s tests -p 'test_*.py'
```

`scripts/verify.sh` runs the same plus the corpus check and the frontend build and
tests. Live-model checks are separate: `scripts/evaluate.py --mode api` (100
labelled questions, source hit rate) and `scripts/faq_check.py` (71
recurring CCIFJ questions in EN/FR plus five fallback cases: missing price,
conflicting price, unknown member, unavailable event, unsupported policy). Both
need a running, warmed API and call the model.

Frontend tests:

```bash
cd ccifj-frontend && npm test
```

---

## More documentation

| Document | What it covers |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | How a message is answered, the agents and tools, approvals, search, the staff dashboard, the API, the database and the project structure |
| [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) | Installing it in CCIFJ's environment (TYPO3, Odoo, GetResponse), keeping content current, running costs and who maintains what |
| [docs/HACKATHON.md](docs/HACKATHON.md) | A guided walkthrough: staff access, a demo script and the evaluation commands |
| [prisme/README.md](prisme/README.md) | The Prisme.ai agent: set-up, the backend's address, the exported agent, reuse by another chamber |
| [src/concierge/api/README.md](src/concierge/api/README.md) | The full API reference |
| [src/concierge/db/README.md](src/concierge/db/README.md) | The database tables |
| [ccifj-frontend/README.md](ccifj-frontend/README.md) | The website and dashboard code |

---

## Team

Built by **Team Irbid** at the 42 Asia Hackathon 2026:

| Member | Role |
|---|---|
| Mohammed Adel ([@mohammedadel2023](https://github.com/mohammedadel2023)) | The AI agent and the staff dashboard |
| Sara Ammourh | Database and backend |
| Kanan Lafi | Retrieval (RAG) and search |
| Raad Ali ([@Blitz-adali](https://github.com/Blitz-adali)) | Web scraping and the frontend |

The team's hackathon repository is [Blitz-adali/CCIFJ-CHATBOT](https://github.com/Blitz-adali/CCIFJ-CHATBOT).
