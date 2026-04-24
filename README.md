![DBMS_02 – ER Schema as Code](assets/header.svg)

# DBMS_02 – ER Schema as Code: PlantUML & GitHub Releases

**Module:** Databases · THGA Bochum  
**Lecturer:** Stephan Bökelmann · <sboekelmann@ep1.rub.de>  
**Repository:** <https://github.com/MaxClerkwell/DBMS_02>  
**Prerequisites:** DBMS_01, Lecture 02 (Entity-Relationship Model)  
**Duration:** 90 minutes

---

## Learning Objectives

After completing this exercise you will be able to:

- Write an ER diagram as a text file using **PlantUML**
- Render the diagram locally as an **SVG**
- Explain the concept of **Git tags** and use them deliberately
- Write a **GitHub Actions workflow** that triggers on a tag push
- Have CI automatically publish a **GitHub Release** with a diagram artifact

**After completing this exercise you should be able to answer the following
questions independently:**

- What is the difference between a diagram stored as a binary file and one
  described as plain text in a version-controlled repository?
- Why is a Git tag better suited for triggering a release than a push to `main`?
- What happens in a GitHub Actions workflow when the `permissions: contents: write`
  field is missing?

---

## Check Prerequisites

Run each of the following commands and verify the output:

```bash
plantuml -version
```

> You should see a line starting with `PlantUML version` followed by a version
> number and a date, e.g. `PlantUML version 1.2024.06 (...)`.
> If the command is not found, install PlantUML:
> ```bash
> sudo apt-get install -y plantuml   # Debian / Ubuntu
> brew install plantuml              # macOS
> ```
> PlantUML is also available as a standalone JAR if your package manager offers
> only an outdated version: `java -jar plantuml.jar -tsvg schema.puml`

```bash
java -version
```

> You should see a line such as `openjdk version "21.0.3" ...` or similar.
> Any Java 11 or later is sufficient. PlantUML requires a Java runtime.

```bash
git --version
```

> You should see `git version 2.x.x`. Any version from 2.28 onward is sufficient.

> **Screenshot 1:** Take a screenshot of your terminal showing all three
> successful version checks and insert it here.
>
> `[insert screenshot]`

---

## 0 – Fork and Clone the Repository

This exercise lives in its own repository. You will work in your own **fork**
so that GitHub Actions can create releases under your account.

**Step 1 – Fork on GitHub:**
Navigate to <https://github.com/MaxClerkwell/DBMS_02> and click **Fork** in the
top-right corner. Keep the default settings and confirm.

**Step 2 – Clone your fork:**

```bash
git clone git@github.com:<your-username>/DBMS_02.git
cd DBMS_02
ls
```

> You should see three entries: `README.md`, `assets/`, and `schema.puml`.
> The repository already contains the entity definitions. Your tasks are to
> complete the schema, render it locally, and set up the release pipeline.

---

## 1 – Scenario: Bochum City Library

The Bochum City Library wants to digitize its lending process. From a
requirements interview with the head librarian, the following picture emerges:

> "We maintain a catalogue of all **books** – each book has a unique ISBN, a
> title, and a year of publication. A book can be written by one or more
> **authors**; an author may of course have written several books.
>
> For each book we hold several physical **copies**, tracked by an internal copy
> number and a shelf location (e.g. 'B-12'). Each copy belongs to exactly one
> book.
>
> **Members** receive a membership number and are recorded with their name,
> e-mail address, and enrolment date.
>
> A **loan** is created when a member borrows a specific copy. We store the
> loan date and – once returned – the return date. A copy can be loaned out
> multiple times over its lifetime."

### Entity overview

| Entity | Meaning                      | Key      |
|--------|------------------------------|----------|
| Book   | Title in the catalogue       | ISBN     |
| Author | Writer of a book             | AuthorID |
| Copy   | Physical object on the shelf | CopyNo   |
| Member | Registered library member    | MemberNo |
| Loan   | Event: member borrows a copy | LoanID   |

---

## 2 – PlantUML: Schema as Text

### What is PlantUML?

PlantUML is a text-based diagramming language. Instead of dragging boxes and
arrows with a mouse, you describe the diagram in a readable syntax – similar to
writing code. Three advantages matter here:

1. **Version-controlled** – the `.puml` file lives in the Git repository
   alongside the rest of the course material.
2. **Diff-friendly** – every schema change appears as a readable text diff.
3. **Automatable** – CI can regenerate an up-to-date image from the text on
   every release.

### Crow's Foot notation in PlantUML

PlantUML uses **Crow's Foot notation** for ER diagrams:

```
EntityA [left-side]--[right-side] EntityB : "relationship label"
```

| Symbol | Meaning      |
|--------|--------------|
| `\|\|` | exactly one  |
| `\|{`  | one or more  |
| `o{`   | zero or more |
| `o\|`  | zero or one  |

Read a relationship left to right:

```plantuml
Department ||--|{ Employee : "employs"
' A department employs 1..N employees;
' an employee belongs to exactly one department.
```

### Task 2a – Model the relationships

Open `schema.puml`. The five entities and their attributes are already defined.
Add the **four relationships** from the scenario to the marked `TODO` section.
For each relationship decide:

- What is the cardinality on each side?
- Is participation mandatory (at least one) or optional (zero or more)?

> **Hint:** The relationship between `Author` and `Book` is N:M – both sides
> can have multiple partners.

<details>
<summary>Solution – PlantUML relationships</summary>

```plantuml
A }|--|{ B : "writes"
B ||--|{ C : "is available as"
M ||--o{ L : "places"
C ||--o{ L : "is borrowed in"
```

**Relationships explained:**

| Line | Meaning |
|------|---------|
| `A }\|--\|{ B : "writes"` | An Author writes 1..N Books; a Book is written by 1..N Authors. Both sides are mandatory – a book with no author and an author with no book are not meaningful in this system. |
| `B \|\|--\|{ C : "is available as"` | A Book has 1..N physical Copies; a Copy belongs to exactly one Book. A book with no physical copy cannot be lent. |
| `M \|\|--o{ L : "places"` | A Member places 0..N Loans; a Loan belongs to exactly one Member. Zero loans is valid – a new member has not borrowed anything yet. |
| `C \|\|--o{ L : "is borrowed in"` | A Copy appears in 0..N Loans over its lifetime; a Loan involves exactly one Copy. Zero loans is valid for a copy that has never been checked out. |

> **Caveat:** The `Author`–`Book` relationship is N:M. In a relational schema
> this would require an intermediate table (e.g. `writes(AuthorID, ISBN)`).
> PlantUML models it at the conceptual level and represents N:M directly –
> the physical decomposition is a separate step.

</details>

### Task 2b – Stage the file

```bash
git add schema.puml
git commit -m "feat: complete ER schema for library management"
```

> You should see output ending with a line such as
> `[main 3f7a2c1] feat: complete ER schema for library management`.
> The exact hash will differ.

### Questions for Task 2

**Question 2.1:** In Lecture 02 you used Chen notation (rectangles, diamonds,
ellipses). PlantUML uses Crow's Foot notation. Describe one concrete difference
in how an N:M relationship is represented in each notation.

> *Your answer:*

**Question 2.2:** What would happen if you wrote `@startuml Library` instead of
`@startuml` at the top of `schema.puml`? Try it locally (`plantuml -tsvg schema.puml`)
and observe the output filename. Why would this break the workflow?

> *Your answer:*

**Question 2.3:** The `Author`–`Book` relationship is N:M. Does your PlantUML
diagram require you to model the intermediate join table explicitly, or does
PlantUML abstract it away? At which stage of the design process would the join
table appear?

> *Your answer:*

---

## Excursus: Docs as Code – Why text-based diagrams?

Diagrams are documentation. Like source code, documentation becomes stale,
diverges between team members, and needs to be reviewed when it changes. The
**Docs as Code** principle treats documentation with the same discipline as
source code: it lives in the same repository, is written in a text format,
goes through the same review process, and is built automatically.

A diagram saved as a `.drawio` or `.png` file is opaque to version control.
`git diff` cannot tell you what changed between two versions of a binary file.
A diagram written in PlantUML is transparent:

```diff
-B ||--|{ C : "is available as"
+B ||--o{ C : "is available as"
```

This single line tells a reviewer precisely what changed: the minimum cardinality
on the right side of the `Book`–`Copy` relationship was relaxed from mandatory
(`|{`) to optional (`o{`). A reviewer can comment on it, approve it, or request
a change – exactly as they would with a code diff.

The combination of PlantUML + Git + CI takes this one step further: the
repository always contains both the source of truth (the `.puml` file) and a
rendered artifact (the `.svg` in a Release). Readers never need to install
PlantUML to see the current diagram – they download the latest release. Authors
never need to manually export a new image – CI does it on every tag.

This pattern generalises far beyond diagrams: API documentation generated from
OpenAPI specs, architecture decision records committed as Markdown, database
migration files checked into the same repository as the application code.
The common thread is that **anything that describes the system should live
alongside the system, in a format that version control understands**.

---

## 3 – Render Locally

Before pushing to CI, verify that your schema is syntactically correct:

```bash
plantuml -tsvg schema.puml
```

> PlantUML exits silently on success – no output is printed to the terminal.
> Verify that the output file was created:
>
> ```bash
> ls -lh schema.svg
> ```
>
> You should see a line such as `-rw-r--r-- 1 user group 11K Apr 22 10:41 schema.svg`.

Open `schema.svg` in a browser or SVG viewer.

> You should see five entity boxes connected by four lines with Crow's Foot
> notation at each end. Count the connecting lines: there should be exactly
> four. If you see five unconnected boxes, the relationships are still missing
> from `schema.puml`.

> **Screenshot 2:** Take a screenshot of `schema.svg` open in your browser,
> showing all five entities and all four relationships, and insert it here.
>
> `[insert screenshot]`

Once the diagram looks correct, tell Git to ignore the generated artifact.
The workflow will recreate it on every release:

```bash
echo "schema.svg" >> .gitignore
git add .gitignore
git commit -m "chore: ignore generated SVG artifact"
```

> You should see a commit confirmation. Verify with:
> ```bash
> git status
> ```
> You should see `nothing to commit, working tree clean` – `schema.svg` is
> no longer tracked, even if it exists on disk.

> **Caveat:** We write `schema.svg` specifically rather than `*.svg`. A wildcard
> would also exclude `assets/header.svg`, which is a static committed asset, not
> a generated file. Always be as specific as possible in `.gitignore` entries.

### Questions for Task 3

**Question 3.1:** PlantUML exits without printing anything when it succeeds.
Name one shell command you could use to check the exit code of the last command
and verify that the render succeeded, without opening the SVG file.

> *Your answer:*

**Question 3.2:** Delete `schema.svg` and run `plantuml -tsvg schema.puml` again.
Then run `git status`. Is `schema.svg` shown as an untracked file? Explain why
or why not.

> *Your answer:*

---

## 4 – Git Tags: Versioned Snapshots

### What is a tag?

A **tag** in Git is a named, immutable pointer to a specific commit – like a
bookmark pinned to a point in history. Unlike a branch, a tag **never moves**:
it always refers to the same commit regardless of how many commits follow.

```
... ── A ── B ── C ── D   ← HEAD (main)
                ↑
             v1.0.0        ← tag: immutable pointer
```

### Lightweight vs. annotated

| Type        | Command                            | Contains                       |
|-------------|------------------------------------|--------------------------------|
| Lightweight | `git tag v1.0.0`                   | Commit hash only               |
| Annotated   | `git tag -a v1.0.0 -m "message"`  | Hash + author + date + message |

For releases, always use an **annotated tag** – it carries metadata and is
recognised by GitHub as a proper release point.

### Semantic versioning

Releases conventionally follow **`vMAJOR.MINOR.PATCH`**:

| Part  | Increment when …                        |
|-------|-----------------------------------------|
| MAJOR | the schema changes incompatibly         |
| MINOR | new entities or relationships are added |
| PATCH | typos, comments, or minor corrections   |

Use **`v1.0.0`** for your first release.

### Why do tags matter for CI?

GitHub Actions can bind workflows to specific events. The trigger
`on: push: tags: ['v*']` fires **only** when a tag whose name starts with `v`
is pushed – not on every ordinary commit. This makes releases explicit,
deliberate actions rather than side effects of daily work.

Create the tag now:

```bash
git tag -a v1.0.0 -m "First complete version of the library schema"
git tag
```

> You should see `v1.0.0` listed. Confirm that the tag points to the correct
> commit:
>
> ```bash
> git log --oneline -5
> ```
>
> The topmost commit should be `ci: render PlantUML schema and publish GitHub
> Release on tag` (added in Task 5 – create it before tagging if you have not
> done so yet).

> **Screenshot 3:** Take a screenshot of `git log --oneline -5` showing your
> commits in order, and insert it here.
>
> `[insert screenshot]`

> **Caveat:** Tags are not pushed automatically with `git push origin main`.
> You must push them explicitly. Forgetting this step means the workflow never
> fires, and no release is created. The push command is covered in Task 6.

### Questions for Task 4

**Question 4.1:** Run `git push origin main`. Then open the **Actions** tab in
your fork on GitHub. Did any workflow run trigger? Explain why or why not.

> *Your answer:*

**Question 4.2:** Run `git tag -v v1.0.0`. What information is shown that
`git tag` alone does not display? What does the `-v` flag verify?

> *Your answer:*

---

## 5 – GitHub Actions Workflow

### What is GitHub Actions?

GitHub Actions is a CI/CD platform built into GitHub. You describe automation
steps in a YAML file; GitHub executes them on a virtual machine whenever a
defined event occurs (push, pull request, tag, …).

### Task 5 – Create the workflow file

Create the following file in your repository:

```
.github/
└── workflows/
    └── release.yml
```

```bash
mkdir -p .github/workflows
```

Below is a skeleton of the workflow. Fill in the four `# TODO` lines before
looking at the solution:

```yaml
name: Render Schema & Publish Release

on:
  push:
    tags:
      - 'v*'

jobs:
  render-and-release:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Check out repository
        uses: # TODO: which action checks out the repository?

      - name: Install PlantUML
        run: # TODO: shell command to install plantuml

      - name: Render SVG from PlantUML source
        run: # TODO: plantuml command to produce schema.svg

      - name: Create GitHub Release and upload artifact
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          # TODO: gh release create command with correct flags and artifact
```

<details>
<summary>Solution – complete release.yml with explanation</summary>

```yaml
name: Render Schema & Publish Release

# Trigger: only when a tag starting with "v" is pushed
on:
  push:
    tags:
      - 'v*'

jobs:
  render-and-release:
    runs-on: ubuntu-latest

    # The workflow needs write access to create releases
    permissions:
      contents: write

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Install PlantUML
        run: sudo apt-get install -y plantuml

      - name: Render SVG from PlantUML source
        run: plantuml -tsvg schema.puml

      - name: Create GitHub Release and upload artifact
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          gh release create "${{ github.ref_name }}" \
            --title "DBMS_02 Schema ${{ github.ref_name }}" \
            --notes "Auto-generated ER diagram for tag \`${{ github.ref_name }}\`." \
            schema.svg
```

**Workflow explained line by line:**

| Element | Meaning |
|---------|---------|
| `on: push: tags: ['v*']` | Trigger: the job runs only when a tag whose name matches `v*` is pushed. Ordinary commits to `main` do not trigger this workflow. |
| `runs-on: ubuntu-latest` | The virtual machine image to use. Ubuntu LTS is the most common choice; it comes with `git`, `curl`, and the GitHub CLI pre-installed. |
| `permissions: contents: write` | Grants the workflow token write access to the repository's contents, which includes creating releases. Without this line `gh release create` fails with HTTP 403. |
| `uses: actions/checkout@v4` | A pre-built action maintained by GitHub that clones the repository at the triggering ref (here: the tag) into the runner's working directory. |
| `sudo apt-get install -y plantuml` | Installs PlantUML from Ubuntu's package repositories. The `-y` flag suppresses the interactive confirmation prompt. |
| `plantuml -tsvg schema.puml` | Renders `schema.puml` to `schema.svg` in the same directory. `-t` selects the output type; `svg` produces a scalable vector graphic. |
| `GH_TOKEN: ${{ github.token }}` | Makes the automatically generated workflow token available as the environment variable `GH_TOKEN`. The GitHub CLI reads this variable to authenticate API calls without any manual secret configuration. |
| `gh release create "${{ github.ref_name }}"` | Creates a GitHub Release named after the tag (e.g. `v1.0.0`). `${{ github.ref_name }}` is a GitHub Actions expression that resolves to the pushed tag name at runtime. |
| `--title` / `--notes` | Sets the release title and the body text shown on the release page. |
| `schema.svg` | The file to attach to the release as a downloadable artifact. Multiple files can be listed, separated by spaces. |

> **Caveat:** `permissions: contents: write` is the most common reason a
> first-time workflow fails with a cryptic 403 error. GitHub repositories
> default to read-only token permissions for Actions; write access must be
> granted explicitly for any step that modifies the repository or creates
> releases.

</details>

### Commit the workflow

```bash
git add .github/workflows/release.yml
git commit -m "ci: render PlantUML schema and publish GitHub Release on tag"
```

> You should see a commit confirmation. Verify that the file is in the right
> place:
> ```bash
> cat .github/workflows/release.yml
> ```
> You should see the complete YAML content you just wrote.

### Questions for Task 5

**Question 5.1:** The trigger is `on: push: tags: ['v*']`. What would change
if you replaced it with `on: push: branches: ['main']`? Would the release
workflow still make sense? Why or why not?

> *Your answer:*

**Question 5.2:** The step `apt-get install plantuml` takes roughly 20–30 seconds
on every run. In a larger team with many releases per day, this adds up. Name
one GitHub Actions mechanism that could eliminate this installation time on
repeated runs.

> *Your answer:*

---

## 6 – Push the Tag and Verify the Release

All commits are in place. Push everything to your fork:

```bash
git push origin main
git push origin v1.0.0
```

> After `git push origin v1.0.0` you should see a line ending in
> `* [new tag]         v1.0.0 -> v1.0.0`.
> This push triggers the workflow.

Open your fork on GitHub and navigate to the **Actions** tab.

> You should see a workflow run named `v1.0.0` in progress or completed.
> Each step is listed with a green checkmark once it succeeds. The total
> runtime is typically 60–90 seconds, dominated by the PlantUML installation.

> **Screenshot 4:** Take a screenshot of the completed GitHub Actions run
> showing all four steps with green checkmarks, and insert it here.
>
> `[insert screenshot]`

Once the workflow has completed, navigate to **Releases** in the right sidebar.

> You should see a release named `DBMS_02 Schema v1.0.0` with `schema.svg`
> listed as a downloadable asset. Download the file and open it in a browser.
> It should be identical to the SVG you rendered locally in Task 3.

> **Screenshot 5:** Take a screenshot of the GitHub Release page showing the
> release title, the release notes, and the `schema.svg` download link, and
> insert it here.
>
> `[insert screenshot]`

### Questions for Task 6

**Question 6.1:** In the Actions run, compare the duration of the
"Install PlantUML" step with the "Render SVG from PlantUML source" step.
Which takes longer, and by approximately what factor? What does this suggest
about where optimisation effort should be directed?

> *Your answer:*

**Question 6.2:** Download `schema.svg` from the Release page and compare it
to the `schema.svg` you rendered locally with `plantuml -tsvg schema.puml`.
Are they identical? What does this tell you about the reproducibility of the
build process?

> *Your answer:*

---

## Reflection

After completing all six tasks, answer the following questions:

**Question A – Version history of a diagram:**
Run `git log --oneline` in your repository. Each commit represents a state of
your schema. What would be different if you had stored the diagram as a
`.drawio` file or a PNG instead of a `.puml` file? What information would you
lose?

> *Your answer:*

**Question B – Collaboration:**
Imagine two people editing `schema.puml` simultaneously on separate branches –
one adds a `Genre` entity, the other corrects a cardinality. When they merge,
Git can show a textual diff of the conflict. Would this be possible with a
binary diagram file? What practical consequence does this have for a team?

> *Your answer:*

**Question C – Tag vs. branch for releases:**
You tagged a specific commit as `v1.0.0` rather than pushing to a branch called
`release`. What guarantee does an annotated tag offer that a branch cannot?
Under what circumstance would someone want to use a branch instead?

> *Your answer:*

**Question D – The value of CI for documentation:**
Before this exercise, updating a diagram meant: edit the source, export an
image, commit the image, hope the export matched the source. Describe in two
sentences what the CI pipeline eliminates, and what new guarantee it provides
instead.

> *Your answer:*

> **Screenshot 6:** Take a screenshot of your terminal showing
> `git log --oneline` with all commits from this exercise visible, then open
> `schema.svg` from the Release in the same browser window alongside it.
> Capture both in one screenshot and insert it here.
>
> `[insert screenshot]`

---

## Bonus Tasks

1. **Second release:** Add a `Condition` attribute (`'good'`, `'damaged'`,
   `'lost'`) to the `Copy` entity. Commit the change and create tag `v1.1.0`:

   ```bash
   git tag -a v1.1.0 -m "Add Condition attribute to Copy"
   git push origin v1.1.0
   ```

   Verify that a second release appears with an updated `schema.svg`.

2. **PNG snapshot:** Extend the workflow to also render and attach a PNG.
   Add the following step before `gh release create`, and add `schema.png`
   to the artifact list:

   ```yaml
   - name: Render PNG from PlantUML source
     run: plantuml -tpng schema.puml
   ```

   > **When is a PNG useful?**
   > SVG is a vector format – it scales to any size without quality loss and
   > remains fully editable as text. As long as the diagram is being developed
   > and iterated upon, SVG is the right choice. A PNG is a rasterised
   > snapshot: once rendered, it cannot be scaled up without losing sharpness
   > and cannot be edited as text. Attach a PNG only when you need a fixed,
   > universally renderable image – for example to embed the diagram in a slide
   > deck, a PDF report, or a system that does not support SVG. In all other
   > contexts, keep working with the SVG.

3. **Provoke a failure:** Introduce a deliberate syntax error into `schema.puml`
   (e.g. replace `||` with `|||`), create tag `v1.0.1`, and push. Inspect the
   Actions tab. Which step fails? What does the error message say? Revert the
   error, create tag `v1.0.2`, and push a clean release.

4. **Own extension:** Add an entity of your choice to the schema – for example
   a `Genre` that a Book can belong to, or a `Fine` that accumulates when a
   Loan is overdue. Model the relationships correctly, render locally, and
   release as `v2.0.0`.

   > **Screenshot 7:** Take a screenshot of your extended `schema.svg` and the
   > corresponding GitHub Release page, and insert it here.
   >
   > `[insert screenshot]`

---

> **Key takeaway:** A diagram described as plain text in a repository is not
> just a picture – it is a versioned, reviewable, automatically renderable
> artifact that evolves alongside the system it describes. The CI pipeline
> removes the gap between the source of truth and the published image: they
> are always identical by construction.

---

## Further Reading

- [PlantUML IE Diagram Documentation](https://plantuml.com/ie-diagram)
- [GitHub Actions – Events that trigger workflows](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#push)
- [Semantic Versioning](https://semver.org)
- [GitHub CLI – `gh release create`](https://cli.github.com/manual/gh_release_create)
- [Docs as Code – Write the Docs](https://www.writethedocs.org/guide/docs-as-code/)
# trigger
