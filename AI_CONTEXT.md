# AI Context & Rules

This document serves as a reference for AI assistants working on the Scryscraper project. It outlines critical preferences, architectural decisions, and formatting rules.

## 1. Localisation
- **Locale**: UK English (en-GB).
- **Spelling**: Use UK spellings (e.g., `colour`, `initialise`, `prioritisation`, `behaviour`, `centre`).
- **Scope**: This applies to all documentation, comments, and code identifiers.
- **Exceptions**: US spelling is permitted when strictly required by technical constraints, such as CSS properties (e.g., `background-color`), HTML attributes, or external library APIs.

## 2. Technology Stack
- **Language**: TypeScript.
- **Runtime**: Node.js (Active LTS, currently `^22.0.0`).
- **Package Manager**: `pnpm`.
- **Infrastructure**: Docker (Node.js service + PostgreSQL database).
- **Database**: PostgreSQL (persisted via volume mounts).

## 3. Documentation Structure
All documentation must follow this strict directory hierarchy:

```
docs/
  features/
    overview.md
    <feature-name>/
      <feature-name>--alignments/
        <Ymd-His>--<description>.md
  plans/
    _completed/
      <plan-name>/
        <plan-name>--<final-version>--<Ymd-His>.md
    <plan-name>/
      <plan-name>--<version>--<Ymd-His>.md
      _blockers/
        _cleared/
        <blocker-number>--<description>/
          <blocker-number>--<description>.md
      <priority>--<type>--<name>/
        _history/
          <name>--<previous-version>--<Ymd-His>.md
        <name>--<current-version>--<Ymd-His>.md
```

- **Plan Folders**: Use a folder for each plan item: `<priority>--<type>--<name>/`.
- **Plan Files**: Inside the folder, the file must be named `<name>--<version>--<Ymd-His>.md` (e.g., `pnpm-init--v1--20251122-134500.md`).
- **History**: Move previous versions of plan files to `_history/` within the plan folder.
- **Gitkeep**: Always include a `.gitkeep` file in `_completed`, `_history`, and `_cleared` folders to ensure they are tracked by git even when empty.
- **Main Plan File**: The main plan file must have a timestamp suffix: `<plan-name>--<version>--<Ymd-His>.md`.
  - **Versioning**: Do not edit existing plan files. Create a new file with a new timestamp for every change.
  - **Changelog**: Each new version must include a changelog noting what changed from the previous version.
- **Completed Plans**: Move finished plan folders to `docs/plans/_completed/`.
- **Timestamp Format**: `Ymd-His` (e.g., `20251122-143000`).

## 4. Workflow Preferences
- **Planning Mode**: The AI acts strictly in Planning Mode.
  - **No Execution**: Do not offer to execute commands. The user will manually execute steps.
  - **Fleshed Out Plans**: All `<priority>--<type>--<name>` files must be fully detailed before the user works through them.
- **Blockers**: Use the `_blockers` directory within the specific plan folder for questions or decisions requiring user input.
  - Create a folder `<blocker-number>--<description>/` and a file `<blocker-number>--<description>.md` detailing the question.
  - The user will answer and move the folder to `_cleared/`.
- **Vertical Slices & Atomicity**: Work should be broken down into the smallest possible units. A vertical slice does not need to be a full feature deliverable.
- **Granularity**: Prioritise small, atomic changes to facilitate easy review and preserve history. Avoid bundling unrelated changes.
- **Step-by-Step**: Do not rush. Create a plan, get approval, then wait for the user to execute.
- **Package Selection**: Explicitly justify all new package additions. The user retains veto power.
- **Docker First**: Development workflow should support running within Docker (e.g., using `ts-node` for direct execution).
