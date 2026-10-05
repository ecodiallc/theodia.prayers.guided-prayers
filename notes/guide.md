# Theodia Guided Prayers — Authoring Guide

This guide is for developers and AI assistants creating new `.prayer.zip` guided prayer packages for the Theodia app.

## What is a guided prayer package?

Each package is a single `.prayer.zip` file containing one JSON payload with a guided prayer session. The app downloads the package, unzips it, and renders the `steps` as a timed, step-by-step prayer experience.

## Package lifecycle

1. Author a `.prayer-guided.json` file.
2. Zip it into a `.prayer.zip` using the same base name (e.g., `Peace-And-Rest.prayer.zip`).
3. Register it in `index.json`.
4. Add a one-line entry to `README.md`.
5. Validate JSON and zip structure.
6. Commit and push to `main`.

## File structure

```
/Volumes/external/theodia.prayers.guided-prayers/
├── Faith-In-Gods-Faithfulness.prayer.zip
├── Go-Make-Disciples.prayer.zip
├── index.json
├── README.md
└── notes/
    └── guide.md     <-- this file
```

Each `.prayer.zip` contains exactly one JSON file:

```
Pray-For-Leaders.prayer.zip
└── pray-for-leaders.prayer-guided.json
```

The JSON file name inside the zip should match the package id with `.prayer-guided.json` suffix.

## JSON schema

### Top-level fields

| Field | Required | Description |
|---|---|---|
| `id` | yes | Kebab-case unique identifier. Must match the file base name. |
| `name` | yes | Human-readable title shown in the app. |
| `description` | yes | One-sentence summary. |
| `author` | yes | Usually `"Theodia"`. |
| `traditions` | yes | Array with at least one tradition, e.g. `["General"]`, `["Catholic"]`, `["Jewish"]`. |
| `tags` | yes | **Maximum 3 tags.** Choose from the existing tag vocabulary when possible. |
| `version` | yes | Semantic version, usually `"1.0.0"`. |
| `createdAt` | yes | ISO 8601 UTC timestamp. |
| `source` | yes | Always `"repo"` for packages in this repository. |
| `type` | yes | Always `"single"`. |
| `payload` | yes | Contains the app-facing session data. |

### `payload.data` fields

| Field | Required | Description |
|---|---|---|
| `id` | yes | Same as top-level `id`. |
| `name` | yes | Same as top-level `name`. |
| `description` | yes | Same as top-level `description`. |
| `traditionCategory` | yes | Usually mirrors the primary tradition (e.g., `"General"`, `"Catholic"`). |
| `traditionTag` | yes | Same as `traditionCategory`. |
| `folderId` | yes | Always `null` for downloadable packages. |
| `tags` | yes | **Same array as top-level `tags`. Must be ≤ 3 items.** |
| `steps` | yes | Array of prayer step objects. |
| `durationSeconds` | yes | Sum of all step `durationSeconds`. Must be accurate. |
| `mode` | yes | Always `"default"`. |
| `passageReference` | no | A scripture reference anchoring the session, e.g. `"Psalm 51:10"`. Use `null` if none. |
| `passageBible` | no | Bible module for the passage, usually `null` because the app does not auto-load it. |
| `isBuiltIn` | yes | Always `false` for downloadable packages. |
| `isPinned` | yes | Always `false`. |
| `createdAt` | yes | JavaScript-style timestamp in milliseconds since epoch (UTC). |
| `updatedAt` | yes | Same as `createdAt` for new packages. |

### Step object

| Field | Required | Description |
|---|---|---|
| `id` | yes | Unique within the session, e.g. `step-0`, `step-1`. |
| `order` | yes | Zero-based integer, sequential. |
| `prompt` | yes | Spoken/displayed guidance. Keep it warm, direct, and ecumenical unless tradition-specific. |
| `durationSeconds` | yes | How long the step runs. Typical: 30–90 seconds. |
| `background` | yes | Usually `""` unless a specific background asset is provided. |
| `allowExtend` | yes | `true` if the user may extend this step; `false` for the final step. |
| `allowSkip` | yes | Usually `true`. Set `false` only if skipping would break the session. |
| `showProgress` | yes | Usually `true`. |
| `prayerListSegment` | yes | Usually `false`. Set `true` only if this step is a user prayer-list interaction. |

## Writing guidelines

### Keep it ecumenical unless tradition-specific

General packages should avoid denominational assumptions. Tradition-specific packages (Catholic, Orthodox, Protestant, Jewish) may use their own prayers and language.

### Anchor in Scripture

Most sessions should have a clear `passageReference` and weave 1–3 supporting verses into the steps. Quote verses directly; the app may use TTS to read them aloud.

### Use a warm, invitational tone

Phrases like "Listen to...", "Reflect...", "Pray for...", "Receive...", and "Go in peace..." work well. Avoid lecturing or heavy theology.

### Structure a clear arc

A good session usually follows this shape:

1. **Centering** — become still, remember God's presence.
2. **Scripture** — hear the anchor passage.
3. **Reflection** — let the word sink in.
4. **Response** — confession, thanks, intercession, resolve, etc.
5. **Sending** — close with peace and a simple commission.

### Step durations

- Centering/closing: 30 seconds
- Scripture hearing: 60–90 seconds
- Reflection or intercession: 60–90 seconds
- Total session: 5–12 minutes is a comfortable range.

### Step count

7–12 steps is typical. Each step should have one clear focus.

## Tag rules

- **Maximum 3 tags per package.**
- Use the same 3 tags in both top-level `tags` and `payload.data.tags`.
- Prefer existing tags from the vocabulary below. Only add a new tag when it fills a real gap and is likely to be reused.

### Current tag vocabulary

```
anxiety, beginner, blessing, catholic, confession, courage, daily,
devotion, encouragement, enemies, faith, forgiveness, general,
great-commission, habakkuk, intercession, jerusalem, jewish,
leaders, love, mercy, mission, orthodox, peace, persecuted-church,
protestant, psalm-51, renewal, rest, scripture, stillness
```

Choose tags that describe the **primary action or theme**, not every scripture reference. For example, a prayer on Psalm 51 should use `confession`, `psalm-51`, `renewal` rather than `psalm-139`, `1-john`, `ezekiel`, etc.

## Naming conventions

- Package id: kebab-case, descriptive, e.g. `pray-for-the-persecuted-church`.
- Zip file: Title-Case with hyphens, e.g. `Pray-For-The-Persecuted-Church.prayer.zip`.
- JSON file inside zip: `<id>.prayer-guided.json`, e.g. `pray-for-the-persecuted-church.prayer-guided.json`.
- Name/title: sentence-case title, e.g. "Pray for the Persecuted Church".

## Timestamps

- `createdAt` top-level: ISO 8601 UTC string, e.g. `"2026-10-05T13:10:00.000Z"`.
- `createdAt` / `updatedAt` in `payload.data`: milliseconds since Unix epoch in UTC. Compute with:

```python
import datetime
ms = int(datetime.datetime(2026, 10, 5, 13, 10, 0, tzinfo=datetime.timezone.utc).timestamp() * 1000)
```

## Updating `index.json`

Append a package object to the `packages` array:

```json
{
  "id": "confess-and-be-forgiven",
  "type": "guided",
  "path": "Confess-And-Be-Forgiven.prayer.zip",
  "name": "Confess and Be Forgiven",
  "description": "A guided prayer of honest confession, receiving God's pardon and forgiveness, grounded in 1 John 1:9 and Psalm 32:5.",
  "author": "Theodia",
  "traditions": ["General"],
  "tags": ["confession", "forgiveness", "mercy"],
  "version": "1.0.0"
}
```

Ensure `id`, `path`, and tags match the zip package exactly.

## Updating `README.md`

Add one row to the table:

```markdown
| `Confess-And-Be-Forgiven.prayer.zip` | General | A guided prayer of honest confession and receiving God's pardon, grounded in 1 John 1:9 and Psalm 32:5. |
```

## Validation checklist

Before committing, verify:

- [ ] JSON is valid.
- [ ] Zip contains exactly one `.prayer-guided.json` file.
- [ ] Top-level tags and `payload.data.tags` are identical.
- [ ] Tags count ≤ 3.
- [ ] `durationSeconds` equals the sum of all step durations.
- [ ] `createdAt` and `updatedAt` are in milliseconds; top-level `createdAt` is ISO 8601.
- [ ] Package is registered in `index.json` with matching id, path, and tags.
- [ ] `README.md` table includes the new package.

A quick validation script:

```bash
python3 - <<'PY'
import json, zipfile, os
all_tags = set()
for f in sorted(os.listdir('.')):
    if not f.endswith('.prayer.zip'):
        continue
    with zipfile.ZipFile(f) as z:
        data = json.loads(z.read(z.namelist()[0]))
    assert len(data['tags']) <= 3, f'{f}: too many tags'
    assert data['tags'] == data['payload']['data']['tags'], f'{f}: tag mismatch'
    expected = sum(s['durationSeconds'] for s in data['payload']['data']['steps'])
    assert data['payload']['data']['durationSeconds'] == expected, f'{f}: duration mismatch'
    all_tags.update(data['tags'])
print(f'OK — {len(all_tags)} unique tags')
PY
```

## Commit message format

```
Add <Name> guided prayer

Brief description of anchor passage and supporting verses.

Co-Authored-By: Claude <noreply@anthropic.com>
```

## For AI assistants

When asked to create a new guided prayer:

1. Read this guide.
2. Inspect one existing package to confirm the current JSON shape.
3. Choose a clear scripture anchor and 1–3 supporting verses.
4. Draft 7–12 steps with a centering → scripture → reflection → response → sending arc.
5. Limit tags to 3 and prefer the existing vocabulary.
6. Register in `index.json` and `README.md`.
7. Validate, commit, and push to `main`.

If the user asks for a prayer that could be interpreted as harmful, divisive, or tradition-exclusive, default to a general/ecumenical framing that emphasizes love, peace, and shared devotion.
