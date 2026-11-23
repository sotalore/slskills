# CLAUDE.md (slskills)

@../hq/AGENTS.md
@../hq/LUA-PLUGINS.md

Notes for the `slskills` agent, working on `sl-skills` (SotaLore Skills), the in-game Lua add-on for players: it
copies a character's trained skills into the site's skill planner. The rules every SotA Lore plugin follows
(naming, how the game runs a package, saved data, testing) are in `LUA-PLUGINS.md` above; this file is what's
particular to `sl-skills`.

## What this is

- The package is the `sl-skills/` folder (flat, its name is the slug); everything at the top of the repo is
  outside the package, so the store never sees it. Its global table is `SotaLoreSkills`, its command `/slskills`.
- **Players install it on its own**, so it needs nothing from `sl-admin` and only Lua API 14 (`min_api_version`
  in the manifest). Check that every game call it adds exists at the API version it claims.
- Its helpers in `core.lua` (`jstr`, `jnum`, `header`, `writeFile`) have a twin in `sotalore/sladmin`'s
  `sl-admin/core.lua`. A fix to one of those here is checked there, and the other way round.
- `EXPORT-FORMAT.md`: **the contract with the app, and yours to keep accurate.** The app reads the skills code
  (`SLS1`, `app/models/skill_code.rb` in `sotalore/sotalore`). Change it in the same PR as any change to the
  code or the file, and add a line to its changelog. A change to the code's layout is `SLS2`, and the app must
  read it before the plugin writes it (see "Versions and breaking changes" in `AGENTS.md`).
- **Requests from the app are issues in this repo** (`gh issue list -R sotalore/slskills`); answer in the issue.
- `.gitignore` ignores `sotalore-*.json` so game output is never committed here by accident.

## Code organisation

Each file does one job and starts with a comment saying what it is for.

| File | Job |
|------|-----|
| `skills.lua` | `SotaLoreSkills.Skills`: the trained skills (`ShroudGetSkills`), the skills file, and the paged base 36 code the player copies into the site |
| `skills_ui.lua` | `SotaLoreSkills.SkillsUI`: the skills window: what the add-on is for (with the site address), how to copy, the code in a text box, pages, Refresh. Every text in it is for a player who only copies and pastes (no ids or sizes) |
| `main.lua` | the `ShroudOn...` callbacks, the `/slskills` command and the key binding |

`sl-skills/icon.png` is the add-on's icon in the add-on manager and the store (named by the manifest's `"icon"`):
the site's favicon scaled to 256x256, the size other store add-ons use.

## Environment

- Both game clients' `Lua\sl-skills` folders are **Windows junctions** to this repo's package folder,
  `C:\dev\sotalore\slskills\sl-skills`. Edit here, never through the game's path. Remove a junction with
  `cmd /c rmdir <path>`, which removes only the link.
- `/slskills` opens the skills window; `/slskills file` writes `sotalore-skills.json` to the client's `Lua/`
  folder (not mentioned to players; the window has no button for it since 0.2.0). `sl-skills` keeps no saved data,
  and owns one window.
- **Every text in the window is for a player who only copies and pastes**: no ids, sizes or jargon.

## Working style

- One topic per commit. Do not commit or push unless Darren asks; when he does, push straight to `main` (no
  branches or PRs). Say in the commit message what is verified in game and what is not.
- Before starting work, find or open its issue in this repo (`bug` or `enhancement`), and put "Closes #N" in the commit message.
