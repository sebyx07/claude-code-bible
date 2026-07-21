# claude-code-bible

"The Claude Code Bible" — a prose book about working effectively with Claude Code, distributed as
plain Markdown on GitHub. There is no source code, no build, no tests and no CI: edits ship by
committing `.md` files, and readers consume the chapters straight from the GitHub UI. It serves
anyone trying to get more out of Claude Code, and it deliberately eats its own dog food — the
compressed-writing rules in chapter 11 govern the repo's own prose.

- **Stack:** Markdown only. MIT licensed. No package manager, no toolchain, no deploy target.
- **Key commands:** none. Validation is manual — preview the Markdown, click every link you touched,
  and re-check against the self-check list at the bottom of `docs/11-compressed-config.md`.
- **Layout:**
  - `README.md` — landing page and TOC table (chapter index, quick wins, license).
  - `docs/NN-<slug>.md` — the numbered chapters, 01 through 12, read top-to-bottom as a book.
  - `LICENSE` — MIT.
  - `.gitignore` excludes `.claude/`, so local agent config here is never committed.
- **Editing conventions:** adding a chapter = new `docs/NN-<slug>.md` + a row in the README TOC +
  cross-links. Renumbering means renaming files and rewriting every inter-chapter link — avoid.
  Chapters 01, 03 and 11 are the representative examples for tone.
- **State as of 2026-07-21:** branch `main`, working tree clean (no uncommitted work) when this note
  was written. Remote `origin` → `git@github.com:sebyx07/claude-code-bible.git`.
