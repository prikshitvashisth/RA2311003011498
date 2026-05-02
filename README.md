# Campus Notifications — Assessment Submission

## Repository Structure

```
<your-roll-number>/                        ← GitHub repo root
│
├── logging_middleware/                    ← Reusable log package (Pre-Test)
│   ├── index.js
│   └── package.json
│
├── notification_app_be/                   ← Stage 1: Priority Inbox backend
│   ├── index.js
│   ├── package.json
│   └── .env                              ← AUTH_TOKEN goes here
│
├── notification_app_fe/                   ← Stage 2: Next.js frontend
│   ├── src/
│   │   ├── app/
│   │   │   ├── layout.jsx
│   │   │   ├── page.jsx                  ← All Notifications page
│   │   │   ├── priority/
│   │   │   │   └── page.jsx              ← Priority Inbox page
│   │   │   └── globals.css
│   │   ├── components/
│   │   │   └── NotificationCard.jsx
│   │   └── lib/
│   │       ├── api.js
│   │       └── logger.js
│   ├── package.json
│   ├── next.config.js
│   └── .env.local                        ← NEXT_PUBLIC_AUTH_TOKEN goes here
│
└── notification_system_design.md         ← Architecture doc (Stage 1)
```

---

## Step 0 — Register & Get Your Token

**Register (POST):**
```
http://20.207.122.201/evaluation-service/register
```
Body:
```json
{
  "email": "your@college.edu",
  "name": "Your Name",
  "mobileNo": "9999999999",
  "githubUsername": "your-github-username",
  "rollNo": "your-roll-number",
  "accessCode": "your-access-code-from-email"
}
```
Save the `clientID` and `clientSecret` from the response — you only get them once.

**Get auth token (POST):**
```
http://20.207.122.201/evaluation-service/auth
```
Body:
```json
{
  "email": "your@college.edu",
  "name": "Your Name",
  "rollNo": "your-roll-number",
  "accessCode": "your-access-code",
  "clientID": "from-register-response",
  "clientSecret": "from-register-response"
}
```
Copy the `access_token` from the response.

---

## Step 1 — Run the Backend (Stage 1)

```bash
cd notification_app_be
npm install
```

Paste your `access_token` into `.env`:
```
AUTH_TOKEN=eyJh...your_token_here
```

Run:
```bash
node index.js
```

You should see the top 10 priority notifications printed in the terminal along with the
result of simulating a new incoming Placement notification.

---

## Step 2 — Run the Frontend (Stage 2)

```bash
cd notification_app_fe
npm install
```

Paste your `access_token` into `.env.local`:
```
NEXT_PUBLIC_AUTH_TOKEN=eyJh...your_token_here
```

Start the dev server:
```bash
npm run dev
```

Open **http://localhost:3000** in your browser.

### Pages:
- **/** — All notifications with type filter (All / Placement / Result / Event) and pagination
- **/priority** — Priority Inbox: top-N notifications ranked by type weight + recency

---

## Taking Screenshots (for Submission)

1. Open Chrome, open DevTools → Toggle device toolbar (Ctrl+Shift+M)
2. Screenshot desktop view at full width
3. Switch to a mobile preset (e.g. iPhone 12) and screenshot again
4. For the backend, screenshot your terminal showing the printed top-10 output

---

## Key Design Decisions

- **No database needed** — Stage 1 fetches and ranks in memory on each call
- **Auth is environment-variable-based** — tokens never appear in committed code
- **Read/unread state** uses sessionStorage — persists across page navigation but resets on a new browser session (appropriate for a campus tool with no login)
- **Priority formula**: `typeWeight × 10^13 + timestampMs` ensures type always dominates recency
- **Efficient updates**: new incoming notifications are compared against only the current minimum score — no full re-sort needed
