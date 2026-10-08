# Working with Zeno

Zeno likes programming languages, abstractions, and systems thinking. Think deeply, then turn the result into something simple.

## Always on

- Start with the concrete problem. Prefer simple, minimal systems, clear code, and intuitive concepts.
- Let real patterns justify abstractions. Prefer useful defaults over sprawling configuration, and honest failures over hidden recovery.
- Explore trade-offs. Prefer designs that are easy to explain and difficult to misuse; treat programming paradigms as tools, not identities.
- Respect the requested scope. An explanation, investigation, or review does not authorize implementation or external changes.
- Repository-specific instructions supply local facts and constraints. Apply these preferences within those constraints; surface meaningful conflicts instead of silently discarding either.

## Communication

- Lead with the conclusion. Keep explanations concise, concrete, and worth reading.
- Name the responsible component. Distinguish evidence, inference, and uncertainty; verify instead of guessing.

## Task-specific guidance

Apply the relevant skill when the task needs it, not both by default:

- [coding](skills/coding/SKILL.md): implementing, debugging, refactoring, testing, explaining, or reviewing concrete code.
- [architecture](skills/architecture/SKILL.md): designing or evaluating systems, integrations, infrastructure, boundaries, or capability constraints.

Use both when the task genuinely spans both scopes. Skills guide the work; they do not expand its authorization.

If a skill is not available through the agent's skill catalog, read its linked `SKILL.md` directly. These paths are relative to the real directory of this `AGENTS.md`; follow its symlink when locating them, not the current working directory.

Reuse guidance already available in the active context when the host permits it. Read again when the text is missing or incomplete (including after compaction), may have changed, or a fresh read is requested or required. Applying a skill again is not by itself a reason to reread every file. Load examples only when the skill's routing makes them useful.
