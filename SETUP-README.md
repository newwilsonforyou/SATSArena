# SATs Practice Arena — setup

Three files matter:

| File | What it is |
|---|---|
| `SATs-Practice-Arena.html` | The game. Login, 4 subjects, 3 modes, 2,445 questions. |
| `SATs-Dashboard.html` | The full dashboard — charts, strengths, weak topics, SATs Record. |
| `SUPABASE-SETUP.sql` | Run once to create the two database tables. |

---

## Step 1 — Create the tables (once, 30 seconds)

1. Go to **supabase.com** → your existing arcade project.
2. Left menu → **SQL Editor** → **New query**.
3. Open `SUPABASE-SETUP.sql`, copy everything, paste it in, press **Run**.

It creates two new tables — `sats_players` (accounts) and `sats_sessions` (every completed test).
It does **not** touch your arcade's `players` or `scores` tables, so nothing there can break.

## Step 2 — Put the two HTML files where children can reach them

Results live in the cloud, but the **files** still have to be on each computer. Two ways:

**A. Host them (recommended — this is what makes "any computer" actually work)**
Upload `SATs-Practice-Arena.html` and `SATs-Dashboard.html` to the same place as your arcade
(GitHub Pages, Netlify, your site). Then a child on any computer just opens the link, logs in,
and sees their own results.

**B. Copy the files onto each computer**
Both files must sit in the **same folder**, since they link to each other. Reading past papers also
need the `Reading/` folder alongside them for the story-booklet PDFs.

## Step 3 — First login

Open the game → **New account** → name, password (4+ characters), pick an avatar → Create.
After that, the same name and password work on any computer.

---

## How the pieces fit

**Login.** Name + password. The password is never stored — only a SHA-256 hash of it, in
`sats_players.pass_hash`. Children stay logged in on a device until they press **Log out**.

**Saving.** Every finished test is written to `sats_sessions`: who, when, subject, mode, year/paper,
marks, time taken, and a per-topic breakdown. A `client_id` on each row means the same test can never
upload twice.

**Offline.** If there's no internet the test still runs; the result is kept on that computer and
uploads automatically the next time the page opens online. You'll see
"📥 Saved on this computer" instead of "☁️ Saved to your account".

**My Dashboard** (inside the game) — the signed-in child only, three columns: **Time · Topic · Score**.

**Full dashboard** (`SATs-Dashboard.html`) — the same child, with everything: totals, day streak,
progress line, score by subject, 💪 Strengths, 🎯 Needs improvement, 🙂 Getting there, and the
**SATs Record** table (Name · Date · Time · Topic · Score) plus CSV export.

---

## Checking on the children yourself

In Supabase → **Table Editor** → `sats_sessions`, or run in the SQL editor:

```sql
-- everyone's tests, newest first
select player, played_at, subject, mode, got, max, secs
from sats_sessions order by played_at desc limit 100;

-- weakest topics across all children
select key as topic,
       sum((value->>'got')::int)  as marks,
       sum((value->>'max')::int)  as out_of,
       round(100.0*sum((value->>'got')::int)/nullif(sum((value->>'max')::int),0)) as pct
from sats_sessions, jsonb_each(topics)
group by key having sum((value->>'max')::int) >= 10
order by pct asc;
```

## Notes

- The Supabase anon key in these files is the public key — it is meant to be public, and the tables
  only allow reading and inserting, never deleting.
- Passwords are hashed but this is a child's practice app, not a bank. Don't reuse a real password.
- Forgotten password: delete that child's row in `sats_players` and they can sign up again. Their
  past results stay in `sats_sessions` under the same name.
