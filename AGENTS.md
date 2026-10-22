# Core Agent Guidelines

This document contains core guidelines for every agent, regardless of the repository, the
user, or the device. Other guidelines build on top of these.

Always follow these guidelines unless otherwise specified. Before completing any task,
verify that all changes adhere to these rules.

## Critical Guidelines (`‼️`)

These guidelines are extra important and absolutely must **not** be broken under any
circumstances.

### Keep Repos Anonymous (`‼️`)

Treat every repo as if it were already public, even if it's currently private. Repos can
be forked, made public later, or accidentally exposed, and their history keeps whatever
was ever committed. **Never** put anything that identifies the user or their
infrastructure into code, comments, examples, commit messages, PR descriptions,
documentation, or any other repo content. That includes...

- Real names and usernames (OS accounts, GitHub logins).
- Public IP addresses, especially home or office IPs.
- Personal hostnames, machine names, and SSH config aliases.
- Machine-specific paths (e.g., `/Users/<real-name>/...` or `C:\Users\<real-name>\...`).
- Real names of production deployments, such as WSL distros, Docker networks, or internal
  app codenames.
- Identifiers tied to a live host, such as MAC addresses, internal IP ranges, Docker
  bridge IDs, or certificate serials.

Use generic placeholders instead (e.g., `<user>`, `<host>`, `myhost`, `MyApp`, `1.2.3.4`,
`my-net`, `~/...`, `C:\Users\<user>\...`). When in doubt, ask yourself whether anything
would identify the user, their home, their employer, or a specific machine if the repo
went public tomorrow. If so, fix it before committing.

If a personal identifier does land in a commit, treat it as a leak. Tell the user right
away. Fixing it means rewriting history (e.g., with `git filter-repo --replace-text`),
force pushing, and rotating anything whose secrecy depended on it. Making the repo private
afterward doesn't undo the leak, since clones, forks, and caches keep the original.

### Solve Problems at the Source (`‼️`)

Absolutely **never** apply per-machine workarounds, local config hacks, or quick fixes
that only work on one computer. Always fix the root cause in the source code so it works
automatically for everyone. Tools and features must function correctly out of the box on
any machine. If something requires manual settings or local file edits to work, it's not a
real solution. Think about the team, since every fix should be a permanent, portable
software engineering solution, not a band-aid.

### Clean Up After Fixing Difficult Bugs (`‼️`)

When a difficult bug requires multiple exploratory attempts, always go back through all
changes after the fix is working and remove anything redundant. Keep only the minimal set
of changes needed for the fix, and verify after each removal that things still work.
Exploratory scaffolding, abandoned approaches, and debug artifacts must **not** be left in
the codebase.

### No Environment Assumptions (`‼️`)

When working on a repository, **never** assume you can rely on context about the user's
personal environment, shell configuration, or other repositories. Each repo must be
self-contained and modular, only using dependencies that are explicitly declared. Do not
reference or depend on tools, configs, or conventions that are not part of the project's
own dependency tree.

### No Hardcoded Paths (`‼️`)

Absolutely **never** hardcode user-specific or machine-specific paths. Here are some
examples of what to avoid...

- Home directories (`/home/username`, `/Users/username`)
- Tool storage paths (`~/.rokit/tool-storage/username/toolname/version/binary`)
- Any absolute path that only exists on one specific machine

Instead, always use one of the following...

- Use environment variables (e.g., `process.env.HOME`).
- Use relative paths.
- Use tool shims that resolve paths dynamically (e.g., just `lune` instead of the full
  path).
- Use standard system paths that exist on all machines.

Code must work on any machine, including CI environments. If you find yourself typing a
username or a path from your local machine, stop immediately.

### No Default Values in Configuration (`‼️`)

Absolutely **never** hardcode default fallback values in configuration files (Docker
compose, env vars, infrastructure configs, etc.). All values must come from environment
variables or `.env` files, so there should be no `${VAR:-default}` patterns, no fallback
constants, and no "sensible defaults".

If a required value is missing, the service must fail to start with a clear error message
rather than silently using a default. This applies to Docker compose `command:` and
`environment:` sections, port numbers, tokens, URLs, database paths, and any other
configuration that varies between environments.

The `.env` file (or GitHub Secrets in CI) is the single source of truth for all
configuration. If a value isn't there, it's a deployment error that should be caught
immediately, not papered over with a default that might be wrong.

## Commits

- Always break commits down into logical parts.
- Do not co-author yourself in commits.
- Use sentence case for commit messages (start with a capital letter).
- Do **not** use conventional commit prefixes like `test:`, `chore:`, `feat:`, `fix:`,
  etc.
- Avoid putting colons `:` in commit messages.

## Pull Requests

When creating a PR, review the actual diff (`git diff main...HEAD`) and all commits on the
branch, not just the latest commit. PRs often combine multiple commits, so the title and
description must describe the aggregate of all changes. Double-check that the PR title and
body accurately reflect what the diff contains.

Follow the same naming conventions of commit messages for PR titles.

For the PR description, provide a clear and concise summary of the changes made in
complete sentences, following our [Markdown Style](#markdown-style) conventions. For
smaller PRs, a small paragraph is fine. For larger PRs, feel free to add headings to
organize the content, but start with h3 as the top-level heading instead of h1 or h2.

As commits get added before the PR is merged, make sure to update the PR title and
description to accurately reflect the current state of the changes.

## Editing Files

Use your dedicated file tools (e.g., Read, Edit, and Write) for every change to a file.
The user can review each edit as a diff, and the tools refuse to overwrite a file that
changed since you last read it, which protects edits the user made in the meantime.

**Avoid Workarounds**

Don't edit files through the shell. That means no `sed -i`, `awk`, `perl -pi`, `echo` or
`cat` redirects, or ad hoc Python or Node scripts that rewrite file contents. These hide
the actual change from the user and skip the safety checks above. If an edit fails because
the file changed since you read it, read it again and retry the edit. Don't route around
the failure with a script.

**Exceptions**

Running a project's own formatter or code generator (e.g., `prettier --write`,
`terraform fmt`, `stylua`) through the shell is fine, since those are the normal way the
project rewrites files. For a large mechanical change across many files, such as renaming
something everywhere, a script is acceptable if you tell the user first and show them the
resulting diff afterward.

## Code Style

### Operators

- Use compound assignment operators (`+=`, `-=`, `*=`, `/=`) instead of the expanded form.

### Print Statements

- Avoid using colons (`:`) in prints for stylistic reasons.
- Structure everything in complete sentences.
- Surround strings of interest in quotation marks (`"`).
- Use `[Usage]` instead of `Usage:` for usage messages.
- Handle singular vs. plural properly when displaying counts (e.g., "1 second" vs. "2
  seconds", "1 test suite" vs. "2 test suites").

## Markdown Style

These rules apply to README files, documentation, agent guidelines, and any other markdown
content. Write in proper capitalization and full sentences unless otherwise specified.

They also apply to your own replies to the user. Write chat responses in the same style
(bold labels, arrows, ellipses before lists, and no dashes as punctuation) so that what
you say reads the same way as what you write in files.

### Headings

- Use Title Case for headings (e.g., `### File & Folder Naming`). Also, generally replace
  `and` with `&`.
- Never start the first paragraph or list item under a heading with the same word as the
  heading. Rearrange the sentence instead (e.g., under `### Never Leave a PR Open`, write
  "Every PR you create must be seen through to merge, so **never** leave one open.").
- For a short labeled block that doesn't warrant its own heading, put the label in bold
  Title Case on its own line, followed by a blank line and then the content (e.g.,
  `**Automatic Reviews**`). Unlike a heading, a bold label only covers the content right
  below it, so the surrounding section can continue afterward. The same-first-word rule
  applies here too.

### Punctuation

- Never use em-dashes (`—`), en-dashes (`–`), or hyphens (`-`, `--`) as grammatical
  punctuation. Use commas, periods, parentheses, or an ellipsis instead, or restructure
  the sentence. Hyphens are fine inside compound words (e.g., `well-known`) and in
  technical use (e.g., CLI flags, file names, minus signs).
- Use an ellipsis (`...`) instead of a colon (`:`) to introduce a list, code block, or
  quote (e.g., "Here are some complete examples...").
- Use a double arrow (`»`) in place of an inline colon (`: `), such as when attaching a
  subtitle to a heading (e.g., `### Heartbeat vs Cron » When to Use Each`).

### Lists

- Use a single arrow (`→`) to pair a term with its description (e.g.,
  ``- `feature` → New features.``).
- Use a single arrow after a bold label for labeled steps (e.g.,
  `1. **Check branch** → Verify you are on main.`).
- Write list items as full sentences that start with a capital letter and end with a
  period. Lists of literal values or examples (branch names, file names, etc.) are the
  exception and get no trailing period.

### Emphasis

- Wrap file names, paths, commands, branch names, and literal values in backticks.

### Wrapping

- Wrap prose at column 90. When Prettier is available (as a dependency of the repo or
  installed on the machine), use it with `--print-width 90 --prose-wrap always`.
  Otherwise, wrap by hand at word boundaries.

## Document Formats

### .gitignore

Write `.gitignore` files verbosely so that it's always clear what each pattern matches.
Follow these rules unless the repo specifies otherwise...

- Group patterns into sections, each starting with a `#` comment that names the section
  (e.g., `# Editor & environment`, `# Generated files`).
- Start a pattern with a slash when it only applies to the repo root (e.g., `/TODO.md`,
  `/build/`). Patterns without a leading slash match at any depth, so only leave it off
  for things that can genuinely appear anywhere (e.g., `.DS_Store`, `node_modules/`).
- End a pattern with a slash whenever it refers to a directory (e.g., `.vscode/`,
  `/.wt/`). Without the slash, the pattern also matches files with that name.
- Separate sections with a single blank line, and sort patterns alphabetically within each
  section where possible.

Here's an example...

```gitignore
# Editor & environment
.claude/
.DS_Store
.env
.vscode/

# Dependencies
node_modules/

# Generated files
/build/

# Worktrees
/.worktrees/
/.wt/

# Terraform
.terraform/
*.tfstate
*.tfstate.*

# Project
/CONTEXT.md
/TODO.md
```

## State Management & Persistence

Always consider what happens to in-memory state across process restarts, crashes, and
session boundaries. If runtime state (caches, selections, active modes) needs to survive a
restart, persist it to disk and load it back on startup. Think through edge cases...

> What if the process restarts mid-operation? What if the user switches modes and the old
> state is still saved? What if the cache is stale or from a previous configuration?

When adding a new stateful feature, always ask yourself, "Does this break if the process
restarts right now?" If yes, persist it. When clearing or changing state, clean up all
related persisted data, not just the in-memory cache. Never leave orphaned state that
causes confusing behavior after a restart.

## Testing

Always write unit tests for new behavior. There is no class of feature that's "too
complex" or "too side-effectful" to test. Authentication flows, disk reads/writes, HTTP
requests, UI rendering, and concurrency are all testable when you separate decision logic
from I/O.

Here's the approach...

1. **Extract pure decision logic into a separate module.** If a component decides which
   onboarding steps to show, or which auth method to use, or how to merge two configs,
   that decision should live in a function that takes its inputs as arguments and returns
   a value, with no I/O, no globals, and no `Date.now()`. Tests then exercise it directly
   without touching the real subsystem.
2. **Mock the boundaries, not the logic.** Stub the secure-storage layer, the network
   client, the filesystem, the OAuth provider, or whatever else sits at the edge. Don't
   mock the function under test or assert on internal call counts. Assert on observable
   outputs instead.
3. **Use state machines to verify truthiness.** When the feature has multiple conditions
   (e.g., "is theme picked AND is Discord authed?"), write a test that walks every
   combination of those booleans and asserts the expected branch. Loop over the 2^N truth
   table rather than picking a few cases. Invariants ("X is always true", "Y never
   duplicates") catch bugs that case-by-case tests miss.
4. **Pin contracts that are easy to break.** If a function must be pure, write a test that
   calls it twice with the same input and asserts both results are equal. If a list must
   stay in a fixed order, assert the order. If an array shrinking would break the caller,
   document that with a regression test.
5. **For UI, test the data the UI consumes.** Don't try to assert on rendered pixels.
   Instead, test the helper that produces the props/state the component renders from. The
   React/Ink component itself becomes a thin shell over tested logic.

If a feature is hard to test, that's a signal to refactor, usually by extracting the
decision into a pure helper and pushing I/O to the edges. "Hard to test" is never an
acceptable reason to skip tests.
