# Build a Walking-Paths App

A start-from-scratch course in **Go, SQLite, and React** — learned by building one real,
useful thing. No prior experience with any of the three is assumed.

The product: a tool for cataloguing walking paths by what they're *good for* — shade,
garbage cans, closed loops, parking, dog-friendliness — that then suggests where to walk
today based on your mood, the weather, and your criteria.

> This is the plain-text companion to `index.html`, which is the same course laid out as a
> hostable web page.

---

## How to use this guide

**This guide is built to be skimmed.** The callouts carry the details that actually
matter — if you read nothing else on a page, read the callouts.

| Callout | Meaning |
|---|---|
| 🎯 **Goal** | What you can do by the end of the unit. |
| 🧠 **Learn this** | A concept to genuinely understand, not just type. |
| 🔑 **Decision** | A choice that's expensive to reverse later. Make it on purpose. |
| ⚠️ **Don't skim past this** | A specific detail that will cost you an evening if you miss it. Rare — so when you see one, stop. |
| ✅ **Try it** | Something to actually do, right now, to make an idea concrete. |
| 💡 **From an older language?** | A note for anyone coming from an older or procedural background. |

### Difficulty markers

| Marker | Meaning |
|---|---|
| 🧱 | **Foundation.** Slow down. Everything later assumes it. |
| 🔍 | **Hard-to-reverse decision.** Think before you commit. |
| 🎁 | **Payoff.** The app visibly gets better. |
| 🌉 | **Bridge.** Two things built separately now connect. |

> **The one rule:** Every unit ends with the app still running, and with a commit. No
> exceptions. The habits only build through repetition.

> ⚠️ **No throwaway exercises.** Everything you build accrues to the app. There is no
> "learn it in a sandbox, then start the real thing" — the first Go you write is the app's
> real domain code, and the first SQL you write is the app's real schema. Each unit's
> practice *is* a piece of the walking app, so nothing is wasted and every session leaves
> the product further along.

---

## What you're building

A **single Go program** that:

- serves the web UI (compiled right into the program)
- **provides** a JSON API — the endpoints your own UI calls to load and save paths
- stores everything in one SQLite database file beside it
- **consumes** outside APIs — a weather service now, a mapping service later — to inform
  its suggestions

That whole program compiles to **one binary**, which you deploy to a small cloud host. You
reach it at a single HTTPS URL and can install it on your phone's home screen like any app
— the same feel as a hosted site, but with a real Go server and database behind it.

> 🧠 **Two kinds of API — this trips everyone up.** "API" appears twice above, in opposite
> roles. Your app **provides** an API: your React UI is the client that calls it to fetch
> and save paths. Your app also **consumes** other people's APIs — the weather service, and
> later a mapping service — where *your Go code* is the client calling out. You build both
> sides in this course: the provider in Phase 2, the consumer in Phase 4.

- **In scope:** paths entered by hand, tagged, browsable, plus a suggestion engine
  answering "where should I walk today?"
- **Deferred, but designed for:** drawing paths on a map, and recording a walk from a
  phone's GPS. Sketched at the end.

> 🔑 **The decision that protects your future.** The database stores real latitude/longitude
> geometry **from the very first schema**, and an early unit builds the form that collects
> it. Those two facts go together. Typing a couple of coordinates by hand now is tedious —
> but it's what makes adding a map later *additive* instead of a re-entry of every path you
> own.

### The stack, and why

| Layer | Choice | Why |
|---|---|---|
| Backend | Go | One static binary, no runtime to install, excellent standard library. |
| HTTP | `net/http`, no framework | Modern Go routes method + path natively. Learn the real thing. |
| Database | SQLite | Zero setup, one file, backed up by copying it. |
| DB access | `database/sql`, hand-written SQL | No ORM. An ORM hides exactly what a database course is meant to teach. |
| Driver | `modernc.org/sqlite` 🔍 | Pure Go, no C compiler needed. See Unit 6. |
| Frontend | React + Vite + TypeScript | A widely used, well-documented modern web stack. |
| Styling | Hand-written CSS, ~12 custom properties | A tiny, legible design system you fully control. |
| Weather | Open-Meteo | Free, no key, no signup, no billing. |
| Hosting | Single binary on a small cloud host | Runs locally in dev; deployed to one HTTPS URL and installable as a phone app (PWA). |

---

# Phase 0 — Bare machine to first commit

*Assume nothing is installed.*

## Unit 0 — Tools 🧱

**In one line:** Get the machine able to compile Go, run Node, use Git, and open an editor
— and prove each one works before moving on.

🎯 **Goal:** A verified toolchain.

On Windows, install everything via `winget` (it ships with Windows 11). Run these one at a
time, reading each one's output:

```powershell
winget install GoLang.Go
winget install Git.Git
winget install Microsoft.VisualStudioCode
winget install OpenJS.NodeJS.LTS
winget install GitHub.cli
winget install SQLite.SQLite
```

> ⚠️ **Don't skim past this.** Close the terminal and open a new one before you verify.
> Installers change your `PATH`, and an already-open terminal won't see the change. This
> trips up everyone exactly once — make it zero times.

Verify — all seven must print a version:

```powershell
go version      # expect 1.26.x or newer
git --version
node --version
npm --version
gh --version
sqlite3 --version
code --version
```

✅ **Editor extensions:**
- **Go** — on first opening a `.go` file it offers to install helper tools (formatter,
  debugger, language server). Say yes to all. The debugger is used for real later.
- **SQLite Viewer** — click a database file and browse it.
- **Prettier** and a **React snippets** pack — for the frontend phase.

> 💡 **From an older language?** A *compiler* (Go) turns source into one runnable file. An
> *interpreter* (Node) reads and runs the source each time. If you know a mainframe compile
> step, Go's model will feel more familiar than JavaScript's.

**Ends with:** nothing to commit yet — the only unit without a commit.

---

## Unit 1 — Git and GitHub from zero 🧱

**In one line:** An account, a repo, and the branch → commit → push → pull-request loop,
done once slowly.

🎯 **Goal:** You can push, and you've made a commit.

**Set up your identity:**

1. Create a GitHub account with an email you'll keep. Turn on 2FA immediately.
2. Tell Git who you are — this stamps every commit:

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

Authenticate the CLI so you never type a password into Git:

```powershell
gh auth login        # GitHub.com → HTTPS → authenticate via browser
```

**Create the repo:**

```powershell
cd ~
gh repo create WalkApp --private --clone
cd WalkApp
```

> 🧠 **Adding a collaborator.** Use the web (*Settings → Collaborators*) or the API — **not**
> `gh repo edit`, which does not manage collaborators:
> ```powershell
> gh api --method PUT repos/<owner>/WalkApp/collaborators/<user> -f permission=push
> ```

**Your first commit.** Write a short `README.md`, then:

```powershell
git status          # read this. It's telling you a story.
git add README.md
git status          # read it again. What changed?
git commit -m "Add project README."
git push
```

> 🧠 **Learn this — the four ideas under Git.**
> - A **commit** is a snapshot plus a message. The message is for humans.
> - A **branch** is a movable label pointing at a commit. Cheap. Not a big deal.
> - **Local vs remote:** `commit` saves on your machine, `push` sends it to GitHub.
> - Version control is a *tree of changes*, not a shared folder. Nothing is overwritten;
>   everything is added. (One later unit shows you the single exception.)

**The loop you'll reuse:**

```powershell
git switch -c my-first-branch
# ...make an edit...
git add -A
git commit -m "Expand the README with the app's purpose."
git push -u origin my-first-branch
gh pr create --fill
# review it on GitHub, then:
gh pr merge --squash --delete-branch
git switch main
git pull
```

> 🧠 **Commit message style:** imperative mood, sentence case, ending in a period.
> *"Add the paths table."* — not "added paths table".

**Ends with:** a merged pull request.

---

## Unit 2 — Go, part 1 🧱

**In one line:** Learn Go's core syntax by writing the app's real domain model — the `Path`
type and a walk-time estimate the suggestion engine will use later.

🎯 **Goal:** Comfort with Go's core syntax — and the first real code in the app.

> 🧠 **This is real app code, not practice.** You'll learn the language by building the
> app's **domain model**: a `Path` type and a function that estimates how long a walk takes.
> It's a command-line program for now (no web, no database), but the `Path` struct you
> define here is the exact type the whole app uses, and the estimate function is what the
> suggestion engine calls in Unit 19. Nothing is thrown away.

Initialise the app's Go module inside the repo, and start the domain file:

```powershell
cd ~/WalkApp
go mod init github.com/<you>/walkapp
code .
# create main.go with a Path struct and an EstimateWalkTime function
```

Work through these, writing each one *into that real code*:

1. **`package main` and `func main()`** — the entry point.
2. **`go run .` vs `go build`** — run from source vs produce a runnable file. Build it and
   double-click it, so "one binary" is concrete.
3. **Variables and types** — `var` vs `:=`. Static typing, inferred.
4. **Functions**, including **multiple return values** — alien at first, then the thing you
   miss elsewhere.
5. **Structs** — define the real `Path` (name, distance, is-loop, surface). If you know
   records, this is a record with better ergonomics.
6. **Slices** — a `[]Path`, the app's list of paths. Note `append` returns a new slice;
   this surprises everyone once.
7. **`if`, `for`** — there's only `for`. No `while`, no `do`. One loop keyword.
8. **Errors** 🔍 — see below. Spend real time here.
9. **`gofmt`** — formatting isn't an opinion in Go. The editor formats on save. No style
   debates. This is a feature.
10. **Reading the docs** — look up a standard-library function online and learn what a Go
    doc page looks like.

> 🧠 **Learn this — errors are values.** Go has **no exceptions**. Functions return errors
> as ordinary values, and you check them:
> ```go
> result, err := doSomething()
> if err != nil {
>     return fmt.Errorf("doing something: %w", err)
> }
> ```
> That block is roughly a third of all Go code. It's repetitive because it *is* — and that's
> the trade: errors can't be silently ignored.

> 🧠 **Public vs private is spelling.** A capitalised name (`Path`) is public; lowercase
> (`path`) is package-private. Capitalisation is *syntax*, not style.

*Deliberately not here yet:* pointers, interfaces, methods, closures — those get their own
unit, placed right where they're needed.

**Ends with:** a committed `main.go` holding the real `Path` type and walk-time estimate.
Unit 5 refactors it into a proper package; the suggestion engine imports it in Unit 19.

---

# Phase 1 — Data

*The foundation. A background in structured data and integrity is an asset here, not a gap.*

## Unit 3 — Designing the schema 🧱🔍

**In one line:** Write the whole database design on paper, driven by the questions you want
answered, before any code.

🎯 **Goal:** A written schema, on paper.

> ✅ **No computers for the first hour.** Get a whiteboard. Write 15–20 real sentences you
> want the app to answer, in plain English:
> - "Show me shaded loops under 2 miles."
> - "Which paths near me have garbage cans?"
> - "I have 40 minutes and it's about to rain — where do I go?"
>
> *Then* design tables that answer them. Schemas designed without the queries in mind are
> always wrong in the same way.

**Roughly where you'll land (argue with it):**

- **paths** — id, name, description, distance, is_loop, surface, timestamps
- **path_points** — the geometry: path_id, sequence, latitude, longitude
- **features** — a point of interest on a path: path_id, kind (bench / garbage can / water
  / parking), latitude, longitude, note
- **tags** + **path_tags** — many-to-many for qualities: shaded, quiet, dog-friendly
- **walks** — walks actually taken, so suggestions can use history

> 🧠 **Learn this — the concepts that pay off forever.**
> - **Primary keys** — stable identity for every row.
> - **Foreign keys** — how tables reference each other. (SQLite needs them explicitly
>   turned on — done in Unit 6.)
> - **One-to-many** (a path has many points) vs **many-to-many** (paths ↔ tags). The join
>   table is the whole trick.
> - **Normalization** — why "shaded" lives in a `tags` table, not a comma-separated column.
>   This is the fix for repeating-group pain in flat files.
> - **Nullable vs required** — say what's optional, on purpose, per column.

> ⚠️ **Don't skim past this — decide delete behaviour NOW.** When a path is deleted, what
> happens to its points, features, tags, and walks? Cascade (they go too)? Restrict
> (refuse)? Soft-delete (mark, never remove)?
>
> Skipping this has a guaranteed consequence: a much later unit ships a delete endpoint, you
> run it, and get `FOREIGN KEY constraint failed` with no idea whether the bug is in the
> code, the SQL, or the schema. The answer would be "in this unit, many sessions ago."
> Decide it here.

> 🧠 **Why `path_points` needs a `sequence` column.** SQL rows have **no inherent order**.
> Without an explicit sequence, "the third point on the path" isn't a meaningful concept, no
> matter what order you inserted them in. This is the biggest surprise coming from
> sequential file processing.

> 🔑 **The geometry decision.** `path_points` exists from day one even though nothing draws
> a map yet. You'll type a start coordinate and a couple of waypoints by hand (right-click
> in an online map to read the lat/lon). It's tedious but bounded — and it's the price of
> not re-entering every path when the map arrives. **Recommendation: pay the tax.**

**Ends with:** commit `docs/schema.md` — tables, columns, types, delete behaviour, and the
queries each table serves. Words and diagrams. No SQL yet.

---

## Unit 4 — SQL, hands-on — no Go 🧱

**In one line:** Get fluent in SQL by writing the app's *real* schema — the CREATE
statements you type here become the app's first migration.

🎯 **Goal:** Comfort writing SQL — and the app's real schema, committed.

> 🧠 **What you write here is the app's schema.** You'll learn SQL by building the real
> thing. Type your `CREATE TABLE` statements straight into
> `migrations/001_initial_schema.sql` and run that file against the app's real database. In
> Unit 8 the migration runner you build will run this exact file — so the schema you design
> today is what ships.

> 🧠 **Why still no Go this unit.** Mixing a new language with a new database means when
> something breaks, you don't know which one broke. So this unit stays pure SQL — but
> against the real database, not a throwaway.

```powershell
cd ~/WalkApp
mkdir migrations
# write your CREATE TABLE statements in migrations/001_initial_schema.sql
sqlite3 walk.db < migrations/001_initial_schema.sql   # run it
sqlite3 walk.db                                       # open it and explore
```

> 🧠 **What's committed vs what isn't.** The migration files and your `docs/queries.sql` are
> committed and accrue. The `walk.db` file itself is git-ignored and disposable *on purpose*
> — the whole point of migrations is that the database is reproducible from the committed
> SQL. Delete it any time and rebuild.

**First: getting data in and out**

1. **CREATE TABLE** — type every table into the migration file, including the
   `REFERENCES ... ON DELETE CASCADE` clauses you decided on.
2. **SQLite's type system** — it's dynamic, with type *affinity* rather than strict types.
   Ten minutes, but a foot-gun if you assume stricter rules.
3. **INSERT** — three or four real paths near your house. These are the app's actual starter
   data (and your test data for every unit that follows).
4. **SELECT** — start with everything, then narrow with `WHERE`, `ORDER BY`, `LIMIT`.

**The exact syntax — CREATE, INSERT, SELECT.** A normal table (note the foreign key with
cascade) and the many-to-many join table (note the composite primary key):

```sql
CREATE TABLE paths (
  id          INTEGER PRIMARY KEY,          -- auto-numbers itself
  name        TEXT    NOT NULL,
  distance_mi REAL    NOT NULL,
  is_loop     INTEGER NOT NULL DEFAULT 0,   -- SQLite has no bool; use 0/1
  surface     TEXT,                         -- 'paved'|'trail'|'mixed'; may be NULL
  created_at  TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE path_points (
  id      INTEGER PRIMARY KEY,
  path_id INTEGER NOT NULL REFERENCES paths(id) ON DELETE CASCADE,
  seq     INTEGER NOT NULL,                 -- ordering; rows have no inherent order
  lat     REAL    NOT NULL,
  lon     REAL    NOT NULL
);

CREATE TABLE path_tags (            -- the many-to-many join
  path_id INTEGER NOT NULL REFERENCES paths(id) ON DELETE CASCADE,
  tag_id  INTEGER NOT NULL REFERENCES tags(id)  ON DELETE CASCADE,
  PRIMARY KEY (path_id, tag_id)     -- a path can't carry the same tag twice
);
```

```sql
INSERT INTO paths (name, distance_mi, is_loop, surface)
VALUES ('River Loop', 2.3, 1, 'paved');

SELECT name, distance_mi FROM paths
WHERE is_loop = 1 AND distance_mi < 3
ORDER BY distance_mi
LIMIT 10;
```

> ⚠️ **Don't skim past this — two habits that save you.**
> - **NULL is not zero and not empty string.** `NULL = NULL` is *not true*. Let this bite
>   you now, on purpose, while it's just your own starter data.
> - **Before any DELETE or UPDATE, run the `WHERE` as a SELECT first.** Confirm it hits
>   exactly the rows you meant, *then* change it to a delete. Every single time. Forever.

**Then: the parts that matter most**

5. **JOIN** 🧱 — the concept that unlocks relational databases, worth most of a session on
   its own. `INNER JOIN` first, then `LEFT JOIN`. The difference must be *felt*, not read —
   that's 45 minutes of experimenting.
6. **GROUP BY and aggregates** — `COUNT`, `AVG`, `SUM`. "How many garbage cans per path?"
7. **Transactions** — `BEGIN` / `COMMIT` / `ROLLBACK`. Break one halfway on purpose and
   roll it back.
8. **Prove the cascade** — turn foreign keys on, delete a path, watch its points vanish.
   Turn them off, try again, watch the orphans survive.
9. **Indexes** — add one, then run `EXPLAIN QUERY PLAN` before and after. Watch `SCAN`
   become `SEARCH`. This is the "why is it slow" tool.

**The exact syntax — joins, grouping, transactions, indexes.**

INNER vs LEFT JOIN — paths with their tags. INNER drops paths with no tags; LEFT keeps them
with a NULL tag:

```sql
-- INNER: only paths that HAVE at least one tag
SELECT paths.name, tags.name AS tag
FROM paths
JOIN path_tags ON path_tags.path_id = paths.id
JOIN tags      ON tags.id = path_tags.tag_id;

-- LEFT: ALL paths; tag is NULL where a path has none
SELECT paths.name, tags.name AS tag
FROM paths
LEFT JOIN path_tags ON path_tags.path_id = paths.id
LEFT JOIN tags      ON tags.id = path_tags.tag_id;
```

GROUP BY — "how many garbage cans per path?" `COUNT` collapses each group to one row:

```sql
SELECT paths.name, COUNT(features.id) AS bins
FROM paths
LEFT JOIN features
  ON features.path_id = paths.id AND features.kind = 'garbage_can'
GROUP BY paths.id
ORDER BY bins DESC;
```

Transaction — both inserts land, or neither does:

```sql
BEGIN;
  INSERT INTO paths (name, distance_mi) VALUES ('River Loop', 2.3);
  INSERT INTO path_points (path_id, seq, lat, lon)
    VALUES (last_insert_rowid(), 0, 42.36, -71.06);
COMMIT;          -- or ROLLBACK; to undo everything since BEGIN
```

Prove the cascade — deleting a path removes its points too (once FKs are on):

```sql
PRAGMA foreign_keys = ON;      -- OFF by default in the sqlite3 shell!
DELETE FROM paths WHERE name = 'River Loop';
SELECT COUNT(*) FROM path_points;   -- that path's points are gone
```

Index + EXPLAIN QUERY PLAN — watch `SCAN` become `SEARCH`:

```sql
EXPLAIN QUERY PLAN
SELECT * FROM path_points WHERE path_id = 3;
-- => SCAN path_points            (reads every row)

CREATE INDEX idx_points_path ON path_points(path_id);

EXPLAIN QUERY PLAN
SELECT * FROM path_points WHERE path_id = 3;
-- => SEARCH path_points USING INDEX idx_points_path (path_id=?)
```

> 💡 **The big mental shift.** SQL is **declarative** — you describe the result you want, not
> the steps to get it. Coming from procedural code, the urge to write the loop yourself
> takes weeks to fade. A join replaces nested loops over two files — and does it better than
> you would.

**Ends with:** commit `docs/queries.sql` — real queries answering your Unit 3 questions,
with comments.

---

## Unit 5 — Go, part 2 🧱🔍

**In one line:** The specific Go — pointers above all — that every remaining unit quietly
requires.

🎯 **Goal:** The language features the database and web code depend on.

> ⚠️ **Don't skim past this — why this unit exists.** Skip it and the next unit opens with
> you typing `rows.Scan(p.Name)` and getting a cryptic error about types and pointers.
> Someone says "put an ampersand there," it compiles, and **nobody learned anything** — you
> were transcribing, not understanding. That failure looks exactly like progress from the
> inside, which is why it runs for weeks unnoticed. Do this unit first.

1. **Pointers** 🧱 — `&` and `*`. See below. This is the wall.
2. **`defer`** — runs when the function exits, no matter how. You'll use
   `defer rows.Close()` constantly.
3. **Methods and receivers** — `func (s *Store) GetPath(id int)`. What that thing before the
   name is.
4. **Interfaces** 🧱 — as a *language feature*. The reveal: `error` is an interface, so
   you've been using one since Unit 2. The idea to land: *a type is a promise about
   behaviour, not a description of a thing.*
5. **Closures** — a function that returns a function. Middleware, later, is exactly this.
6. **Packages** — directories, import paths, how a multi-file program is laid out.
7. **Adding dependencies** — `go get` and what the lock file is for.

**The exact syntax — what these look like in Go.**

Pointers — `&x` is "the address of x"; `*p` reads or writes through the pointer:

```go
func doubleIt(n *float64) { *n = *n * 2 }   // *n = the value n points at

d := 2.3
doubleIt(&d)   // pass the ADDRESS, so the function can change d
// d is now 4.6
```

Method + receiver, and an interface it satisfies — there's no `implements` keyword;
matching the method is enough:

```go
type Path struct {
    Name       string
    DistanceMi float64
}

// (p Path) is the receiver: this method belongs to Path
func (p Path) EstimateWalkTime(mph float64) time.Duration {
    hours := p.DistanceMi / mph
    return time.Duration(hours * float64(time.Hour))
}

// a Pacer is "anything that can estimate a walk time"
type Pacer interface {
    EstimateWalkTime(mph float64) time.Duration
}

var _ Pacer = Path{}   // compile-time proof that Path satisfies Pacer
```

Closure — a function that returns a function, capturing `start`. This is exactly the shape
of the middleware in Unit 13:

```go
func timed(label string) func() {
    start := time.Now()
    return func() { fmt.Printf("%s took %s\n", label, time.Since(start)) }
}

func work() {
    defer timed("work")()   // trailing (): call timed NOW, and defer the
    // ...                   // function it RETURNS until work exits
}
```

> ⚠️ **Spend 40 minutes on pointers, with the debugger open.** Pointers are the thing with
> the fewest analogues in older procedural languages, and they show up *everywhere* from
> here on — in scanning rows, in the database handle, in method receivers, in optional
> fields. Set a breakpoint, watch a struct fill in through a pointer, and make it concrete
> rather than syntactic. Get past this wall and much of what follows is downhill.

> ✅ **Practice target — reshape the real domain code.** Move the `Path` type and walk-time
> estimate from Unit 2's `main.go` into a real `internal/walk` package, turn the estimate
> into a *method* on `Path`, and define the interface the suggestion engine will later
> satisfy. `main.go` now imports and calls it. Small, but it exercises all seven ideas at
> once — and it's the actual package structure the rest of the app builds on.

**Ends with:** the app's domain package (`internal/walk`), imported by `main.go`. Commit.

---

## Unit 6 — Go talks to the database 🌉🔍

**In one line:** The app reads the real paths you entered in Unit 4 out of its own database.
Small, complete, entirely yours.

🎯 **Goal:** The app reads its paths out of SQLite. Migrations come later — the win comes
first.

> ⚠️ **Don't skim past this — the driver choice.** Use **`modernc.org/sqlite`**. *Not*
> `mattn/go-sqlite3`, which nearly every tutorial recommends — it needs a C compiler and is
> a genuine afternoon of Windows pain. The modernc driver is pure Go: `go get` and it works.
>
> **And the driver name is `"sqlite"`, not `"sqlite3"`.** When you copy a snippet off the
> internet, that's the line to change.

```powershell
cd ~/WalkApp
go get modernc.org/sqlite    # the module was already created in Unit 2
```

```go
import (
    "database/sql"
    _ "modernc.org/sqlite"
)

db, err := sql.Open("sqlite",
    "file:walk.db?_pragma=foreign_keys(1)&_pragma=journal_mode(WAL)&_pragma=busy_timeout(5000)")
```

<details>
<summary><b>Why every part of that connection string matters</b></summary>

- **The blank import `_`** — you never call the driver directly. Importing it runs its
  setup, which registers it with `database/sql`. Weird once; a common Go pattern.
- **`foreign_keys(1)`** — SQLite ignores foreign keys unless asked. Your relationships are
  decorative without it.
- **`journal_mode(WAL)`** — write-ahead logging, so readers don't block the writer.
- **`busy_timeout(5000)`** — wait 5s for a lock instead of erroring instantly.
- **`sql.Open` doesn't actually connect.** It's lazy. Call `db.Ping()` to find out whether
  it works.
</details>

> ⚠️ **Put the pragmas in the connection string, not a one-off command.** A pragma set with
> `db.Exec("PRAGMA ...")` applies to *one* connection. But the database handle is a **pool**
> of connections, so your setting would silently apply to some queries and not others.
> Putting it in the connection string applies it to every connection the pool opens. This is
> the kind of bug that works on your machine for a month.

> 🧠 **Learn this — reading rows safely.**
> - `QueryRow` vs `Query` vs `Exec`, and always `defer rows.Close()`.
> - Read the real paths from Unit 4 with `Query` + `rows.Next()` into a `[]walk.Path` —
>   reusing the domain type you already built.
> - Check **`rows.Err()`** after a loop — the one everyone forgets. A loop can end because it
>   finished *or* because it broke, and only this tells you which.
> - **Placeholders, never string-building:** `db.Query("... WHERE id = ?", id)`. Building
>   SQL with string formatting is how injection happens. Never. Not once.

**The exact syntax — the read loop.** The `&` in `Scan` is *why* Unit 5's pointers mattered
— `Scan` writes through those pointers into your struct:

```go
func (s *Store) ListPaths() ([]walk.Path, error) {
    rows, err := s.db.Query(
        "SELECT id, name, distance_mi, is_loop FROM paths ORDER BY name")
    if err != nil {
        return nil, fmt.Errorf("querying paths: %w", err)
    }
    defer rows.Close()

    var paths []walk.Path
    for rows.Next() {
        var p walk.Path
        if err := rows.Scan(&p.ID, &p.Name, &p.DistanceMi, &p.IsLoop); err != nil {
            return nil, fmt.Errorf("scanning path: %w", err)
        }
        paths = append(paths, p)
    }
    return paths, rows.Err()   // the check everyone forgets
}
```

Reading one row, and turning "no such row" into a signal you can map to a 404 later:

```go
var p walk.Path
err := s.db.QueryRow("SELECT id, name FROM paths WHERE id = ?", id).
    Scan(&p.ID, &p.Name)
if errors.Is(err, sql.ErrNoRows) {
    return walk.Path{}, ErrNotFound
}
```

**Ends with:** `go run .` prints a real path. Commit — and add `*.db`, `*.db-wal`,
`*.db-shm` to `.gitignore`. 🔍 **Never commit the database.** It's generated, binary, and
causes merge conflicts you cannot resolve.

---

## Unit 7 — When it breaks 🧱

**In one line:** The debugging skills you'll spend most of your first month using — taught
right when you have a backlog of confusion to spend on them.

🎯 **Goal:** Read errors, drive a debugger, get unstuck.

1. **Read five real compile errors** — induce them on purpose in your own code (unused
   variable, wrong type in an argument, missing return, undefined name). This *is* the Go
   experience for the first month; reading them fluently is the whole difference.
2. **Read a real panic** — force one, then read the message and the stack trace beneath it.
   Reading a stack trace is a skill, learnable in 20 minutes, worth a hundred hours
   downstream.
3. **The debugger** — set a breakpoint, step through your row-scanning code, watch the
   struct fill in through the pointers. If pointers still felt abstract, this is where they
   stop being abstract.
4. **Read a docs page properly** — pick one standard-library function and decode its full
   signature together.
5. **Run the linter** locally, now, not for the first time in CI six units later.

> ✅ **Write your stuck-ladder into the README.** Something like: re-read the error aloud →
> check the docs → look at what you just changed (`git diff`) → use the debugger → explain it
> to someone (or a rubber duck) → walk away for 20 minutes → ask for help. Having a written
> ladder helps more than it sounds like it would, especially late at night when there's no
> one at the next desk.

**Ends with:** a README section with your stuck-ladder. Commit.

---

## Unit 8 — The schema has a history 🔍

**In one line:** Write the runner that automatically applies the migration file you already
wrote in Unit 4 — so the database rebuilds itself from committed SQL.

🎯 **Goal:** Migrations, from scratch, running your real schema.

You already have `migrations/001_initial_schema.sql` from Unit 4, and you've been running it
by hand. This unit automates that:

- Read the numbered files from the `migrations/` folder you already have.
- A `schema_migrations` table recording what's run.
- On startup: find the un-run ones, run each **in a transaction**, record it.
- Read them from disk for now. A later unit swaps that for embedding — as a diff against
  working code, the best way to learn it.

> ✅ **Prove it rebuilds from nothing.** Delete `walk.db` entirely, then run the app. The
> runner recreates the whole schema from your committed migration files. Re-enter (or seed)
> your starter paths. This is the migrations promise made concrete: the database is
> reproducible, so losing it is never a disaster.

> ⚠️ **Don't skim past this.** Write the runner as `func migrate(db *sql.DB) error` — **not
> inline in `main()`**. The obvious first draft buries it in `main`, and then a later unit
> ("test the store against a temp database") has no way to create a schema. Small decision,
> delayed consequence.

> 🧠 **Learn this.**
> - **Why migrations exist:** the schema has a *history*, and every environment must be able
>   to replay it. This is what makes database change safe and repeatable instead of a person
>   running commands by hand at 2am.
> - **Append-only.** You never edit a migration that's already run; you write a new one.
>   Annoying for a week, then it saves you forever.

**The exact syntax — a minimal runner.** About 25 lines. Numbered files apply in order; each
un-run one runs inside its own transaction and gets recorded:

```go
func migrate(db *sql.DB) error {
    if _, err := db.Exec(`CREATE TABLE IF NOT EXISTS schema_migrations (
        version    TEXT PRIMARY KEY,
        applied_at TEXT NOT NULL DEFAULT (datetime('now')))`); err != nil {
        return err
    }

    files, err := filepath.Glob("migrations/*.sql")
    if err != nil {
        return err
    }
    sort.Strings(files)   // 001_, 002_, ... apply in filename order

    for _, f := range files {
        version := filepath.Base(f)

        var seen string
        if err := db.QueryRow(
            "SELECT version FROM schema_migrations WHERE version = ?",
            version).Scan(&seen); err == nil {
            continue   // already applied — skip it
        }

        stmts, err := os.ReadFile(f)
        if err != nil {
            return err
        }
        tx, err := db.Begin()
        if err != nil {
            return err
        }
        if _, err := tx.Exec(string(stmts)); err != nil {
            tx.Rollback()               // a broken migration leaves no trace
            return fmt.Errorf("migration %s: %w", version, err)
        }
        tx.Exec("INSERT INTO schema_migrations (version) VALUES (?)", version)
        if err := tx.Commit(); err != nil {
            return err
        }
    }
    return nil
}
```

**Ends with:** `go run .` creates the database and applies migrations. Commit.

---

# Phase 2 — The API

*Where the app becomes a service rather than a script.*

## Unit 9 — An HTTP server 🧱

**In one line:** A server that answers a browser — using the standard library and no
framework.

🎯 **Goal:** A running server with a health endpoint.

```go
mux := http.NewServeMux()
mux.HandleFunc("GET /api/health", func(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{"status": "ok"})
})
http.ListenAndServe(":8080", mux)
```

That's the entire web framework. There isn't one. This is standard library.

> 🧠 **Learn this.**
> - **What HTTP is:** a request (method, path, headers, body) and a response (status,
>   headers, body). Text over a socket. Much simpler than it looks from outside.
> - **Methods:** GET reads, POST creates, PUT/PATCH updates, DELETE removes. Conventions,
>   but universally expected.
> - **Status codes:** 200 ok, 201 created, 400 you sent nonsense, 404 not found, 500 I
>   broke. Getting these right is most of what makes an API pleasant.
> - **Modern routing:** `"GET /api/paths/{id}"` in the pattern, read it with
>   `r.PathValue("id")`. Older tutorials predate this and reach for extra libraries — you
>   need none.

> ⚠️ **Don't skim past this — every handler runs concurrently.** Each request runs in its
> own lightweight thread. Ten browsers means ten copies of your handler running *at the same
> time*. Anything they share needs protecting. **Prove it:** put a global `counter++` in a
> handler, hammer it with a loop of requests, and watch it print a number that's visibly
> wrong. You just wrote a data race in four lines. A later unit is where it would have bitten
> you for real.

Explore with **both** a browser and `curl`. The browser only does GET easily; `curl` does
everything and is how you'll test the rest.

**Ends with:** a health endpoint. Commit.

---

## Unit 10 — Reading, and the seam 🌉🔍

**In one line:** List and fetch paths over JSON — and introduce the interface that lets two
areas (and two people) work without colliding.

🎯 **Goal:** Real paths as JSON in a browser.

> 🔑 **The layering decision.** Three layers, one job each:
> ```
> handler   (HTTP: parse request, write response, choose status)
>    ↓  a PathStore interface
> store     (SQL: the only place that knows a database exists)
>    ↓
> database
> ```
> Handlers never write SQL. The store never touches the HTTP response. That single rule is
> why a future "swap the database engine" is a lesson and not a rewrite — and why you can
> test each half without the other.

> 🧠 **Learn this.**
> - **Struct tags:** a small annotation maps a Go field name to the JSON name the frontend
>   wants. Go and JavaScript disagree about naming; the tag is the translator.
> - **Marshal / unmarshal** — struct ↔ JSON bytes.
> - **REST:** resources are nouns, methods are verbs. `/api/paths/3`, not
>   `/api/getPath?id=3`.
> - **The big idea:** depend on what something *does*, not what it *is*.

**The exact syntax — interface, tags, handler.** The interface the handlers depend on (and
the SQLite store implements):

```go
type PathStore interface {
    List() ([]walk.Path, error)
    Get(id int) (walk.Path, error)
}
```

Struct tags translate Go field names into the JSON your UI expects:

```go
type Path struct {
    ID         int     `json:"id"`
    Name       string  `json:"name"`
    DistanceMi float64 `json:"distance_mi"`
    IsLoop     bool    `json:"is_loop"`
}
```

A handler that depends on the interface, never on SQLite:

```go
func (s *Server) listPaths(w http.ResponseWriter, r *http.Request) {
    paths, err := s.store.List()
    if err != nil {
        http.Error(w, "internal error", http.StatusInternalServerError)
        return
    }
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(paths)   // the tags decide the field names
}
```

**Ends with:** real paths as JSON in a browser. Commit.

---

## Unit 11 — Writing, and the geometry 🌉🔍

**In one line:** Create a path *with its points*, and delete one — the unit that finally
makes the geometry decision real.

🎯 **Goal:** Paths with geometry, creatable and deletable.

> 🧠 **Learn this — two core database moves.** Collecting nested points forces the two most
> important write patterns in the whole course:
> - **Last-inserted id** — insert the parent path, get its new id, insert the child points
>   with it.
> - **A multi-statement write in one transaction** — the path and its N points commit
>   together or not at all. A path with half its points is worse than no path.

**The exact syntax — parent + children in one transaction:**

```go
func (s *Store) Create(p walk.Path, points []walk.Point) (int64, error) {
    tx, err := s.db.Begin()
    if err != nil {
        return 0, err
    }
    defer tx.Rollback()   // no-op after a successful Commit; safety net otherwise

    res, err := tx.Exec(
        "INSERT INTO paths (name, distance_mi, is_loop) VALUES (?, ?, ?)",
        p.Name, p.DistanceMi, p.IsLoop)
    if err != nil {
        return 0, err
    }
    pathID, _ := res.LastInsertId()   // the id SQLite just assigned

    for _, pt := range points {
        if _, err := tx.Exec(
            "INSERT INTO path_points (path_id, seq, lat, lon) VALUES (?, ?, ?, ?)",
            pathID, pt.Seq, pt.Lat, pt.Lon); err != nil {
            return 0, err   // the deferred Rollback undoes the path AND its points
        }
    }
    return pathID, tx.Commit()
}
```

Then the delete endpoint — and the cascade you decided in Unit 3 and proved in Unit 4 does
its job. If you skipped either, this is where you'd lose 90 minutes to a mystery.

> ✅ **Try to break it with curl.** Send a string where a number goes. Send `{}`. Send
> malformed JSON. Send a path with zero points, and one with 500. Ask for a path that
> doesn't exist. Find where it panics — it will — and that becomes the next unit.

**Ends with:** paths with geometry, creatable and deletable. Commit.

---

## Unit 12 — Updating 🔍

**In one line:** Full-replace updates first (easy), then partial updates — and meet the
pointer as the answer to a problem you just hit.

🎯 **Goal:** Full CRUD.

- **PUT first.** Full replace. Dumb. Send the whole path, overwrite it. Works in twenty
  minutes.
- **Then feel why the UI wants PATCH** — a screen that toggles one field shouldn't have to
  send everything.

> 🔑 **Pointers for optional fields.** A partial update must tell "set this to false" apart
> from "don't touch this". A plain boolean can't — but a *pointer* to a boolean can also be
> empty (nil). That distinction is the whole reason the type exists.

**The exact syntax — nil means "leave it alone".** Pointer fields let the JSON decoder tell
"absent" from "present and false". You then build the `SET` clause only from the fields that
actually arrived:

```go
type PathPatch struct {
    Name   *string `json:"name"`     // nil if the client omitted it
    IsLoop *bool   `json:"is_loop"`
}

func buildUpdate(id int, patch PathPatch) (string, []any) {
    var sets []string
    var args []any
    if patch.Name != nil {
        sets = append(sets, "name = ?")
        args = append(args, *patch.Name)
    }
    if patch.IsLoop != nil {
        sets = append(sets, "is_loop = ?")
        args = append(args, *patch.IsLoop)
    }
    args = append(args, id)
    // column names are OURS (safe to concatenate); values stay parameterised (?)
    return "UPDATE paths SET " + strings.Join(sets, ", ") + " WHERE id = ?", args
}
```

> ⚠️ **Don't skim past this — one subtle SQL rule.** Building a partial update means
> assembling SQL from whichever fields were provided — which brushes against the Unit 6 rule
> "never build SQL from strings." The reconciliation: **interpolating column names you
> control is fine; interpolating values you don't control is an injection.** Talk it through
> until it's obvious.

Ordering is the lesson: pointers-as-optional lands ten times harder as the answer to a
question you asked than as a footnote handed to you.

**Ends with:** full CRUD, proven with `curl`. Commit.

---

## Unit 13 — Making it not fall over

**In one line:** Validation, honest error responses, and middleware.

🎯 **Goal:** The server survives abuse.

1. **Validation** — reject a path with no name, a negative distance, a latitude outside
   −90..90. Return **400** with a message that says what's wrong. Validate at the handler
   boundary, before anything reaches the store.
2. **Error handling** — "no rows" is a **404**, not a leaked database message. A raw SQL
   failure is a **500**, and the detail goes to *your* log, not the user's screen.
3. **Middleware** — a function wrapping a handler. Write two: request logging, and panic
   recovery so one bad request can't kill the server.

> 🧠 **The one rule for errors:** **log the detail, return the category.** The user gets "not
> found"; your log gets the stack trace. And middleware being "just a function returning a
> function" is where Unit 5's closures earn their keep.

**The exact syntax — validation and a middleware.** Validation returns a plain error the
handler turns into a 400:

```go
func validatePath(p walk.Path) error {
    if strings.TrimSpace(p.Name) == "" {
        return errors.New("name is required")
    }
    if p.DistanceMi < 0 {
        return errors.New("distance cannot be negative")
    }
    if p.Lat < -90 || p.Lat > 90 {
        return errors.New("latitude out of range")
    }
    return nil
}
```

Middleware *is* Unit 5's closure — a function that wraps a handler and returns a handler:

```go
func logRequests(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)   // run the real handler
        log.Printf("%s %s  %s", r.Method, r.URL.Path, time.Since(start))
    })
}
```

**Ends with:** it survives the curl abuse from Unit 11. Commit.

---

## Unit 14 — Tests 🧱

**In one line:** Automated tests, the race detector, and a merge conflict faced on purpose
while it's safe.

🎯 **Goal:** A green test suite, and conflicts stop being scary.

1. **Table-driven tests** — the idiomatic Go style. Test the validation logic.
2. **In-process handler tests** — test a handler with no live server. A new mental model: a
   fake server inside the test.
3. **Store tests against a temp database** — and this is why the migration runner was made a
   function back in Unit 8.
4. **The race detector** — run the tests with race detection on. Meet it now, so it's
   familiar before the weather cache needs it.

**The exact syntax — a table-driven test.** One `cases` slice, one loop, a named subtest per
row. Run with `go test ./...` (add `-race` for the detector):

```go
func TestValidatePath(t *testing.T) {
    cases := []struct {
        name    string
        path    walk.Path
        wantErr bool
    }{
        {"ok",         walk.Path{Name: "Loop", DistanceMi: 2},  false},
        {"empty name", walk.Path{Name: "",     DistanceMi: 2},  true},
        {"negative",   walk.Path{Name: "Loop", DistanceMi: -1}, true},
    }
    for _, tc := range cases {
        t.Run(tc.name, func(t *testing.T) {
            err := validatePath(tc.path)
            if (err != nil) != tc.wantErr {
                t.Errorf("got err=%v, wantErr=%v", err, tc.wantErr)
            }
        })
    }
}
```

> ✅ **Manufacture a merge conflict on purpose.** Two branches edit the same function; merge
> them and resolve the clash. Conflicts are terrifying exactly once, and only if the first
> one is real and urgent — meet `<<<<<<< HEAD` on a quiet afternoon in a safe branch instead.
> Note that dependency lock files are conflict magnets.

**Ends with:** the suite passes, with and without race detection. Commit.

---

# Phase 3 — The face

*The web UI, and the moment the two halves become one app.*

## Unit 15 — UI scaffold 🎁

**In one line:** A React app with a tiny, deliberate design system.

🎯 **Goal:** A styled page, no data yet.

```powershell
cd ~/WalkApp
npm create vite@latest web -- --template react-ts
cd web && npm install && npm run dev
```

> 🧠 **A design system of about a dozen variables.** Put ~12 CSS custom properties on
> `:root` and build everything from them. Greens and earth tones suit a walking app.
> ```
> --bg  --ink  --muted  --paper  --line  --accent  --on-accent
> --warn  --info  --topbar  --input  --radius
> ```
> Twelve tokens is the *entire* design system. Resist a thirteenth.

- **Self-host your font** — drop the font files in and declare them with `@font-face`,
  rather than depending on an external service.
- **A small component vocabulary** — pill buttons, cards with a subtle border, an uppercase
  letter-spaced label above each card title, one big display heading that scales with the
  viewport.
- **A flat source folder** — the app is small; deep nesting earns nothing.

> 🧠 **Learn this.** Components, props, and simple state. **JSX is not HTML** (different
> attribute names, curly braces for expressions, one root element). What the build tool
> does. And why theming through CSS variables is nearly free.

**Ends with:** a styled page, no data. Commit.

---

## Unit 16 — Frontend meets backend 🌉🎁

**In one line:** The React app lists paths that came out of the database. The whole thing is
real for the first time.

🎯 **Goal:** Paths on screen, creatable from the browser.

This is the first session where something breaks and you don't know *which side* broke it.
Budget for that — it's normal.

1. **Fetch the list** into component state when the page loads. No extra data libraries —
   plain fetch is right at this size.
2. **Loading and error states** — the request takes time and can fail. A UI with no loading
   state *looks* broken. This is correctness, not polish.
3. **A create form** — POST a path *with its points*, and the list updates. Yes, typing
   coordinates by hand is tedious; this is the tax Unit 3 warned about, and it's why the map
   later is additive.

> 🔑 **Use a dev proxy, not CORS.** The dev server and the API run on different ports, and
> the browser blocks that by default. Rather than write cross-origin code you'll later
> delete, proxy the API path through the dev server — three lines of config, and it matches
> how production works (one server, same origin).

> 💡 **The mental jump.** **Async.** A fetch returns a promise, and the UI renders *before*
> the data arrives. If you're used to code that blocks until it has an answer, give this real
> time.

> ✅ **Trace the full round trip out loud, once.** click → fetch → handler → SQL → row →
> JSON → state → pixels. When something breaks later, debugging *is* walking this path and
> asking where it stopped. Watch the real request and response in the browser's network panel
> — that's where most frontend bugs die.

**The exact syntax — fetch into state, and the dev proxy.** Load the list once when the
component mounts, with the three states a real UI needs:

```tsx
const [paths, setPaths] = useState<Path[]>([])
const [error, setError] = useState<string | null>(null)
const [loading, setLoading] = useState(true)

useEffect(() => {
  fetch("/api/paths")
    .then(r => {
      if (!r.ok) throw new Error(`HTTP ${r.status}`)
      return r.json()
    })
    .then(setPaths)
    .catch(e => setError(String(e)))
    .finally(() => setLoading(false))
}, [])   // [] = run once, on first mount
```

The dev proxy, in `vite.config.ts`, so `/api` reaches your Go server:

```ts
server: {
  proxy: { "/api": "http://localhost:8080" },
}
```

**Ends with:** paths on screen, creatable from the browser. Commit. 🎁

---

## Unit 17 — Tags, features, and walks

**In one line:** The many-to-many end to end — plus the walk log the suggestion engine will
depend on.

🎯 **Goal:** Tagging works; walks are logged.

Tagging a path "shaded" touches every layer: join table, store method, endpoint, UI control.
It's a lap around the whole track, and it should feel much easier than the units before it.
If it doesn't, that's useful information about where to slow down.

Add tag chips on the card, a tag filter on the list, and features (garbage cans, benches,
parking) with their coordinates.

> ⚠️ **Don't skip the walk log.** Add an "I walked this" button → one POST → one insert. It's
> the smallest complete vertical slice, so it's the ideal thing to build end-to-end yourself,
> through every layer. It's also load-bearing: the suggestion engine scores on walk history.
> With no writer, that table stays empty forever and two scoring rules silently do nothing —
> they'd *look* alive and be dead.

**Ends with:** tagging works, walks are logged. Commit.

---

# Phase 4 — The actual idea

*The payoff. Everything so far built the machine; this is the part you wanted.*

## Unit 18 — Consuming someone else's API 🎁

**In one line:** Go fetches the weather — the client side of API work, and where the data
race from Unit 9 finally shows up.

🎯 **Goal:** A weather endpoint, race-free.

Use **Open-Meteo** — no key, no account. Paste the URL in a browser first and read the JSON:

```
https://api.open-meteo.com/v1/forecast?latitude=42.36&longitude=-71.06
    &hourly=temperature_2m,precipitation_probability
    &temperature_unit=fahrenheit&timezone=auto
```

> 🧠 **Two easy-to-miss parameters.** The API defaults to Celsius and GMT. Without
> `temperature_unit` and `timezone=auto`, you get metric temperatures and timestamps that
> don't line up with "what's the weather this afternoon."

> ⚠️ **Don't skim past this — two things that bite in production.**
> - **Give your HTTP client a timeout.** The default client has *none*. If the weather
>   service hangs, your handler hangs forever. This is a real outage cause. Always set a
>   timeout.
> - **The cache needs a lock.** Cache the weather for ~15 minutes so you don't call the API
>   on every request — but that cache is shared state read and written by concurrent
>   handlers. Write it *without* a lock first, run the race detector, and watch it catch the
>   bug you just wrote. Best fifteen minutes in the whole course; it turns an abstract warning
>   into a tool you trust.

> 🧠 **Learn this.**
> - Being an API **client** rather than a server — the half most developers spend more time
>   in.
> - **Unmarshalling JSON you don't control** — define a struct for only the fields you want;
>   the rest is ignored.
> - **Graceful degradation** 🔑 — if the weather call fails, the app must still suggest
>   walks, just without weather input. A dependency being down is not an excuse to be down.

**The exact syntax — client with a timeout, and a locked cache.** A package-level client
with a timeout, and a struct for only the fields you want:

```go
var client = &http.Client{Timeout: 5 * time.Second}   // default client has NONE

type forecast struct {          // fields you skip are simply ignored
    Hourly struct {
        Temperature  []float64 `json:"temperature_2m"`
        PrecipChance []int     `json:"precipitation_probability"`
    } `json:"hourly"`
}

func fetchForecast(ctx context.Context, lat, lon float64) (forecast, error) {
    url := fmt.Sprintf(
        "https://api.open-meteo.com/v1/forecast?latitude=%f&longitude=%f"+
            "&hourly=temperature_2m&temperature_unit=fahrenheit&timezone=auto",
        lat, lon)
    req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
    resp, err := client.Do(req)
    if err != nil {
        return forecast{}, err
    }
    defer resp.Body.Close()

    var f forecast
    return f, json.NewDecoder(resp.Body).Decode(&f)
}
```

The cache is shared across concurrent handlers, so a `sync.Mutex` guards it:

```go
type weatherCache struct {
    mu   sync.Mutex
    at   time.Time
    data forecast
}

func (c *weatherCache) get(ctx context.Context, lat, lon float64) (forecast, error) {
    c.mu.Lock()
    defer c.mu.Unlock()
    if time.Since(c.at) < 15*time.Minute {
        return c.data, nil   // fresh enough — no network call
    }
    f, err := fetchForecast(ctx, lat, lon)
    if err != nil {
        return c.data, err   // degrade: hand back the last good value
    }
    c.data, c.at = f, time.Now()
    return f, nil
}
```

**Ends with:** a weather endpoint, race-free. Commit.

---

## Unit 19 — The suggestion engine 🎁

**In one line:** "Where should I walk today?" gets a real, explainable answer. The most
interesting code in the app, with no single right answer.

🎯 **Goal:** A ranked suggestion from mood, time, weather, and history.

The approach is **scoring**: every path earns points against the request, then you sort.

```
Input:  mood, available time, weather, walk history
Output: paths, ranked, each with a reason
```

**Rules to argue about (the fun part):** Hot and sunny → shaded paths up · Raining → paved
up, muddy trails down · "quick" → distance near your available time · "explore" →
least-recently-walked up · "with the dog" → dog-friendly required, garbage cans up · walked
yesterday → down, for variety.

> 🧠 **Learn this.**
> - **Business logic as its own layer** — testable with no database and no server. This is
>   where Unit 10's layering pays a visible dividend: you can test the whole engine with plain
>   values. It calls the `EstimateWalkTime` you wrote back in Unit 2.
> - **Tuning is guessing, then adjusting.** How do you decide shade is worth 3 and
>   distance-fit is worth 5? You don't, really. You guess, use it, and adjust. Getting
>   comfortable with that *is* the lesson.

> 🔑 **Return the reason, not just the ranking.** Each suggestion should say *why* it scored
> well ("shaded, and it's 85°"). Recommendations without reasons feel arbitrary and people
> stop trusting them — and reasons make the thing debuggable, which is not a coincidence.

**The exact syntax — one scoring pass.** Each rule adds points and, when it fires, a
human-readable reason. Return both:

```go
func score(p walk.Path, req Request, w forecast) (int, []string) {
    points := 0
    var reasons []string

    if w.isHot() && p.HasTag("shaded") {
        points += 3
        reasons = append(reasons, fmt.Sprintf("shaded, and it's %.0f°", w.tempNow()))
    }
    if req.Mood == "quick" && p.DistanceMi <= req.milesFor(req.Minutes) {
        points += 5
        reasons = append(reasons, "fits the time you have")
    }
    if p.WalkedWithin(24 * time.Hour) {
        points -= 2
        reasons = append(reasons, "walked recently — mixing it up")
    }
    return points, reasons
}
```

Then rank: score every path, sort by points descending, return the top few with their
reasons attached.

This is the one place where "we spent a while arguing about weights" is a success, not a
delay.

**Ends with:** a working `/api/suggest` endpoint. Commit.

---

## Unit 20 — The suggestion UI 🎁

**In one line:** The screen you'll actually use, outdoors, one-handed.

🎯 **Goal:** The app becomes genuinely useful.

Mood picker, time slider, a ranked list of cards with their reasons, today's weather in the
corner. Design it **big, glanceable, and phone-first** — you'll read it in sunlight, holding
a leash.

> 🧠 **The design vocabulary already exists.** Because of the tokens and components from Unit
> 15, this unit is assembly, not design. Design for the real context: sunlight and one free
> hand are genuine constraints. Set responsive breakpoints so it reflows cleanly on a phone.

**Ends with:** the app is genuinely useful. Take it for a walk. 🎁

---

# Phase 5 — Making it real

*One file to run, a machine that checks your work, and a memory for the project.*

## Unit 21 — One binary 🎁🔍

**In one line:** The whole app — UI and API — compiled into a single self-contained binary,
the exact thing you'll deploy in Unit 22.

🎯 **Goal:** One self-contained binary — run it locally now, ship it next unit.

Build the web app, embed the output right into the Go program, and serve it from the same
server as the API. One process, one origin — no cross-origin config, no separate frontend
deploy.

```go
//go:embed all:web/dist
var webFS embed.FS
```

Start by swapping the earlier disk-based migration loading for the embedded version — a
small diff against working code, which is the easiest way in.

> 🧠 **Learn this.**
> - Files can be compiled *into* the executable.
> - **Serving a single-page app:** unknown paths must return the app's `index.html`, not a
>   404, or refreshing on a deep link breaks. Classic bug. You'll hit it.
> - **Build order:** the web build must run *before* the Go build, because embedding reads
>   the folder at compile time.

> ⚠️ **Don't skim past this — commit a placeholder.** The web build folder is normally
> git-ignored, but embedding **fails the build** if the folder is empty. So on a fresh clone,
> the program won't even *compile* until someone runs the web build. Fix: commit a placeholder
> `index.html` (with a "run the build" message) so the embed pattern always matches something.
> This is a perfect entry for the project-memory file in Unit 23.

> 🧠 **Why this is remarkable — and why it makes hosting easy.** No runtime to install on the
> server, no `node_modules` to ship, no container required. You hand the host a single file
> and it runs. Most stacks can't do this — and it's exactly why deploying in Unit 22 is short
> instead of a saga.

**Ends with:** one binary that runs the whole app locally, ready to deploy. Commit. 🎁

---

## Unit 22 — CI, and going live 🎁🔍

**In one line:** A machine that builds and tests every push, and the app deployed to a real
HTTPS URL you can open — and install — on your phone.

🎯 **Goal:** Green CI, and the app live on the web.

> ⚠️ **Don't skim past this — the CI step order is load-bearing.** Frontend build **first**:
> ```
> npm ci && npm run build   (in web/)
> go vet ./...
> go test ./...
> go build
> ```
> The Go steps both compile the package, so they hit the embed directive — put them before
> the web build and they fail with "no matching files found" before the frontend even exists.
> Knowing a rule and encoding it are different things, which is itself part of the Unit 23
> lesson.

**Deploy the binary to a small host.** Because the whole app is one self-contained binary
(Unit 21), hosting is mostly "hand this file to a server that keeps it running." Pick a host
that runs a persistent process *with a disk* — Fly.io and Render are the usual picks for a
Go + SQLite app. They give you HTTPS automatically, so the phone-install step below just
works.

- **What you push up:** the single binary (or a tiny container that just copies it in and
  runs it). No Node, no runtime, no dependencies to install on the server.
- **What the host gives you:** a URL like `walkapp.fly.dev`, automatic HTTPS, and a
  persistent volume where `walk.db` lives.

**The exact syntax — a Dockerfile and the deploy.** A two-stage build: compile the UI and
the binary, then ship *only* the binary on a tiny base image:

```dockerfile
# --- build stage ---
FROM golang:1.26 AS build
WORKDIR /src
COPY . .
RUN cd web && npm ci && npm run build   # UI first (embed reads web/dist)
RUN go build -o /walkapp .

# --- run stage ---
FROM debian:stable-slim
COPY --from=build /walkapp /walkapp
ENV WALK_DB=/data/walk.db     # point the DB at the mounted volume
EXPOSE 8080
CMD ["/walkapp"]
```

Then, with Fly.io, three commands — the volume is the step you must not skip:

```
fly launch --no-deploy          # writes fly.toml
fly volumes create walkdata --size 1
# mount that volume at /data in fly.toml, then:
fly deploy
```

> ⚠️ **Don't skim past this — the #1 SQLite-hosting mistake.** Your database is a *file on
> disk*. If that disk isn't a **persistent volume**, every deploy (and every restart) hands
> you a fresh, empty disk — and all your paths are gone. On these hosts you must explicitly
> attach a volume and point `walk.db` at it. This catches nearly everyone once; make sure it
> catches you zero times.

> 🧠 **Make it installable — a PWA, like a real phone app.** To get the "Add to Home Screen"
> behaviour of a native app, add two small static files, served by your Go binary alongside
> the UI:
> - a `manifest.webmanifest` — name, icons, colors, and `display: standalone` so it opens
>   without browser chrome;
> - a **service worker** — a tiny background script; registering one (even a near-empty one)
>   is what makes the app installable, and it can cache the shell so the app opens instantly.
>
> HTTPS is required for both — which your host already provides. Open the URL on your phone,
> "Add to Home Screen," and it lives next to your other apps.

> 🔑 **SQLite's honest constraint.** A file-on-a-disk database means *one* instance, on *one*
> volume. That's the trade you made in Unit 6: enormous simplicity, real limits if you ever
> needed many servers. For a personal walking app that ceiling is miles away — but name the
> trade honestly rather than pretend it isn't there. And back up before you care:
> `sqlite3 walk.db ".backup backup.db"`, or snapshot the host's volume.

**Ends with:** green CI, and the app live at a URL you can install on your phone. Commit. 🎁

---

## Unit 23 — Project memory 🧱

**In one line:** Write the document that makes the project maintainable after you've
forgotten all of it.

🎯 **Goal:** A living project-memory file, and a real README.

> 🧠 **What a project-memory file actually contains.** Not generic boilerplate — an
> operational memory with discipline:
> - **Decisions marked settled**, with a "don't re-litigate this" flag.
> - **Dates and commit references** for why things are the way they are.
> - **Near-misses recorded, with their cost.** What almost went wrong is more valuable than
>   what went right.
> - **Dead code named**, with a date and "ask before deleting."

Write yours with: the tables and why they're shaped that way; the delete-cascade decision;
the driver choice and its `"sqlite"`-not-`"sqlite3"` name; why the pragmas live in the
connection string; the embed placeholder and what breaks without it; the CI step order; the
scoring weights and what you learned tuning them; and every gotcha this course made you hit.

> 🧠 **The point.** Documentation as *memory*, not ceremony. The audience is you, in eight
> months, having forgotten everything. Writing down *why* is worth ten times writing down
> *what* — the code already says what.

**Ends with:** a project-memory file and a real README. Commit.

---

## Course map

| # | Unit | | Teaches |
|---|---|---|---|
| 0 | Tools | 🧱 | PATH, compilers, package managers |
| 1 | Git & GitHub | 🧱 | Commits, branches, pull requests |
| 2 | Go, part 1 | 🧱 | Syntax, structs, errors-as-values |
| 3 | Schema design | 🧱🔍 | Keys, relations, cascade, normalization |
| 4 | SQL hands-on | 🧱 | Joins, transactions, indexes |
| 5 | Go, part 2 | 🧱🔍 | **Pointers**, defer, methods, interfaces, closures |
| 6 | Go + SQLite | 🌉🔍 | Database access, the connection string, scanning |
| 7 | When it breaks | 🧱 | Errors, panics, debugger, docs, getting unstuck |
| 8 | Migrations | 🔍 | Schema history, append-only |
| 9 | HTTP server | 🧱 | Requests, routing, concurrency |
| 10 | Read API | 🌉🔍 | REST, JSON, the interface seam |
| 11 | Write API | 🌉🔍 | Transactions, nested writes, geometry |
| 12 | Update API | 🔍 | Full vs partial updates, pointers-as-optional |
| 13 | Hardening | | Validation, error mapping, middleware |
| 14 | Tests | 🧱 | Automated tests, race detection, merge conflicts |
| 15 | UI scaffold | 🎁 | React, the build tool, the design tokens |
| 16 | Wiring it up | 🌉🎁 | Fetch, async, the full round trip |
| 17 | Tags & walks | | Many-to-many, a full vertical slice |
| 18 | Weather | 🎁 | HTTP clients, timeouts, locking a cache |
| 19 | Suggestions | 🎁 | Business logic, scoring, explainability |
| 20 | Suggestion UI | 🎁 | Designing for the real context |
| 21 | One binary | 🎁🔍 | Embedding, serving a single-page app |
| 22 | CI & going live | 🎁🔍 | Deploy the binary, persistent disk, PWA install |
| 23 | Project memory | 🧱 | Documentation as memory |

---

## Coming from an older or procedural language?

Here's what tends to feel strangest, in rough order:

1. **Pointers.** The one with the fewest analogues in older languages, and the reason Unit 5
   exists. When it feels abstract, open the debugger and *watch* it. This is the wall; after
   it, much is downhill.
2. **Interfaces.** A type as a promise about behaviour, not a description of a thing.
3. **SQL being declarative.** You describe the answer, not the loop.
4. **Async.** Code that continues before it has an answer.
5. **Errors as return values.** No exceptions — just a value you check.
6. **Everything is a text file in version control.** No librarian, no checkout lock.
   Branches are free; nothing is ever really lost.

And what tends to feel *easier* than people warn you: static typing (you already think this
way), structs (they're records), transactions (you already know why they matter), and caring
about data integrity (most people learn that late; a data background learns it first).

---

## What comes later

Deliberately vague — you'll know more by the time you get here, and the right design will be
obvious then, not now.

- **Maps.** Draw a path by clicking instead of typing coordinates; the points flow into the
  geometry table that's been collecting real data since Unit 16. Markers for features. This
  adds another *consumed* API — a mapping/tiles service. **OpenStreetMap** via the **Leaflet**
  library is free and needs no key or billing, so it's the easy way in; **Google Maps** works
  too but requires an API key and a billing account. Concepts: mapping libraries, geographic
  data formats, tile servers.
- **Phone GPS.** Record a walk from a phone, building on the installable-PWA foundation from
  Unit 22. Concepts: offline capture and background sync, the browser location API, smoothing
  GPS noise.
- **A different database.** Swap SQLite for a client/server database — the store interface
  makes this a lesson, not a rewrite. Proving that claim is itself worth doing.
- **Multiple users.** Accounts, sessions, "whose paths are these?" Big, deliberately
  deferred.
- **Import from fitness apps.** Third-party sign-in, other people's APIs and data models.

---

*A self-paced training project in Go, SQLite, and React. Free to copy, adapt, and host.
There is no fixed schedule: work through it when it suits you, and treat "the app still runs"
— not the number of the unit — as your measure of progress.*
