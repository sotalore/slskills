# sl-skills export format (v0.2.0)

The contract between `sl-skills` (SotaLore Skills, the add-on for players) and the app in `sotalore/sotalore`, which
reads the skills code (`app/models/skill_code.rb`). **Only this repo changes this file**, in the same PR as the
export change; see "Contracts" in `AGENTS.md` (`sotalore/hq`). Lua API docs:
https://catnipgames.net/lua/reference.html (sl-skills needs Lua API 14+).

Until 2026-10-18 the skills window lived in the single add-on `sl-recipes`, which became `sl-admin`; `sl-skills`
counts its versions from 0.1.0. Every game skill, trained or not, is in `sl-admin`'s `sotalore-game-skills.json`,
which uses the same skill `id`s; see `sotalore/sladmin`'s `EXPORT-FORMAT.md`.

## Skills: the code and the file (since v0.1.0, in use)

Written by the players' add-on, **`sl-skills`** (SotaLore Skills), not by `sl-admin`. The site keeps, per avatar,
each skill's current level and a target level. The target exists only on the site;
the current level comes from `ShroudGetSkills()`, which lists only the skills the character has **trained** (an
untrained skill is simply absent: read it as level 0). The level used is `trainedLevel` (what the experience has
bought), not `level`, which a scene's skill cap can lower. `/slskills` (or its key binding) opens the "SotaLore
Skills" window; `/slskills file` writes `sotalore-skills.json`. (In sl-admin v0.9.0,
which was never released, the same window lived in the admin add-on.) sl-skills v0.2.0 changed only the window's wording and dropped its "Save file" button; the code and the file are unchanged.

**The file** (`"type": "skills"`) has the header, `character` (the avatar's name) and `skills`, one entry per trained
skill, sorted by `id`: `id` (the game's stable number), `key` (the game's stable name, e.g. "Fireball"), `name`,
`school`, `schoolKey`, `subcategory` (the names the skills window shows, in the player's language), `trainedLevel`,
`level`, `effectiveLevel`, `experience`, `grandmaster`, `mode`. The site does not read it (its skills already
carry the game's ids, from the development team's skill list); it is for checking.

**The code** is for the player to copy and paste without saving a file. A plugin text box holds at most 1024
characters and Lua cannot reach the clipboard, so the code is cut into pages of up to 1000 characters; the window
shows one page at a time and the player copies each in turn (click the box, Ctrl+A, Ctrl+C). A page reads

    SLS1|<page>/<pages>|<id digits>|<o or ->|<character>|<entries>|<check>

`SLS1` is the format. `<entries>` is a run of fixed-width entries, each a skill `id` then its `trainedLevel`, both
**base 36, lower case**: the id in `<id digits>` characters, the level in 2. If any id is negative the plugin adds
2147483648 to every id and writes `o` in the fourth field (subtract it again); otherwise it writes `-`.
`<check>` is two base 36 digits: with `sum = 0`, for each byte `b` of `<entries>`: `sum = (sum * 31 + b) mod 1296`;
the check is `sum` in 2 digits. The pages of one code have the same `<pages>` and `<id digits>`, and the
site should accept them in any order. The character name has `|` and control characters removed.

**In use.** Players paste the code on an avatar's skills page on the site ("Import from game"), all pages into
one box. Measured on 2026-10-18: the game's skill ids are the same as the development team's skill list
(`data/skill-costs.csv`, "Skill ID"), the highest is 13310 (`a9q`), so `<id digits>` is 3 and an entry is 5
characters; a character with 338 trained skills needs **two pages**. Copying from the plugin's text box with
Ctrl+A, Ctrl+C works in the game. The format stays as described; a change to it would be `SLS2`.

## Every file starts the same way

Every file the plugins write begins with the same fields (`plugin` added in sl-admin v0.10.0 / sl-skills v0.1.0,
the others in v0.7.1):

```json
{ "apiVersion": 40, "plugin": "sl-admin", "pluginVersion": "0.10.0", "type": "recipes", ... }
```

| Field | Meaning |
|-------|---------|
| `apiVersion` | The game's Lua API version when the file was written. |
| `plugin` | Which add-on wrote it: `"sl-admin"` or `"sl-skills"`. **Missing means the old single add-on, `sl-recipes`**, which is `sl-admin`'s past (versions 0.7.1 to 0.9.0). |
| `pluginVersion` | The version of that add-on (`major.minor.patch`), its `Version` in `core.lua` and `manifest.json`. Compare versions only within one `plugin`. |
| `type` | What kind of file it is: `"recipes"` (`sotalore-recipes-new.json`, `-all.json` and `-one.json`, which also have a `scope`), `"items"` (`sotalore-item-catalog.json`), `"finds"` (`sotalore-scene-finds.json`) or `"skills"` (`sotalore-skills.json`, from `sl-skills`). Use it to refuse a file uploaded to the wrong place. |

**Files written before 0.7.1 have no `pluginVersion` or `type`** (the file name says the kind). Their plugin
version is unknown, so treat a missing `pluginVersion` as "older than 0.7.1", not as any particular version. This
matters for one thing: a `"scope": "all"` recipe file from before **0.5.2** is missing the recipes the game would
not hand over (about 100), and cannot be told apart from a complete one without the version.


## Changelog

- 2026-11-23: moved here from `sl-lua/EXPORT-FORMAT.md` in `sotalore/sotalore` with the split into separate repos.
  No format changed.
