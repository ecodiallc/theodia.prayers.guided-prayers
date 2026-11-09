# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository is a static catalog of `.prayer.zip` "guided prayer" packages consumed by Theodia (Theodia.app). There is no application code, build system, or test suite here — the repo contains prayer data packages plus metadata (`index.json`), a human-facing catalog (`README.md`), and an authoring spec (`notes/guide.md`).

## Common commands

Validate all packages (JSON, zip shape, tag limits/matching, duration sums):

```bash
python3 - <<'PY'
import json, zipfile, os
all_tags = set()
for f in sorted(os.listdir('.')):
    if not f.endswith('.prayer.zip'):
        continue
    with zipfile.ZipFile(f) as z:
        assert len(z.namelist()) == 1, f'{f}: expected one JSON file'
        data = json.loads(z.read(z.namelist()[0]))
    assert len(data['tags']) <= 3, f'{f}: too many tags'
    assert data['tags'] == data['payload']['data']['tags'], f'{f}: tag mismatch'
    expected = sum(s['durationSeconds'] for s in data['payload']['data']['steps'])
    assert data['payload']['data']['durationSeconds'] == expected, f'{f}: duration mismatch'
    all_tags.update(data['tags'])
print(f'OK — {len(all_tags)} unique tags')
PY
```

Cross-check `index.json` against the zip files (existence, id, tags, duration, recorded size):

```bash
python3 - <<'PY'
import json, os, zipfile
for p in json.load(open('index.json'))['packages']:
    assert os.path.exists(p['path']), p['path']
    with zipfile.ZipFile(p['path']) as z:
        data = json.loads(z.read(z.namelist()[0]))
    assert data['id'] == p['id']
    assert data['tags'] == p['tags']
    assert data['payload']['data']['durationSeconds'] == sum(s['durationSeconds'] for s in data['payload']['data']['steps'])
    assert os.path.getsize(p['path']) == p['size']
print('index OK')
PY
```

Create a zip from an authored JSON (zip name and inner JSON name must follow the naming conventions in `notes/guide.md`):

```bash
zip <Title-Cased-Name>.prayer.zip <kebab-id>.prayer-guided.json
```

## Architecture and conventions

- Each package is one `.prayer.zip` containing exactly one `<id>.prayer-guided.json`.
- Package id is kebab-case and must match the zip base name (Title-Cased with hyphens) and the inner JSON filename.
- Top-level `tags` and `payload.data.tags` must be identical and <= 3 tags, drawn from the existing vocabulary in `notes/guide.md`.
- `payload.data.durationSeconds` must equal the sum of all step `durationSeconds`.
- `payload.data.folderId` is always `null`, `isBuiltIn`/`isPinned` always `false`, `mode` always `"default"`, `source` always `"repo"`, `type` always `"single"` (top-level `type` is `"single"`; index `type` is `"guided"`).
- `index.json` registers each package with matching `id`, `path`, `tags`, and `size` (zip byte size).
- `README.md` catalog table is alphabetically sorted by id; Sources entries describe each package.
- `notes/guide.md` is the full authoring spec, including JSON schema, step shape, tag vocabulary, timestamp formats, and the end-to-end workflow for adding a package. Read it before authoring or modifying packages.
- Commit messages follow the format documented in `notes/guide.md` (`Add <Name> guided prayer`, blank line, description, `Co-Authored-By: Claude <noreply@anthropic.com>`).

## Authoring / modifying packages

Follow the workflow in `notes/guide.md`:

1. Read `notes/guide.md` and inspect one existing package to confirm the current JSON shape.
2. Author the `.prayer-guided.json`, register it in `index.json` and `README.md`.
3. Zip it as `.prayer.zip`.
4. Run both validation commands above.
5. Commit and push to `main`.
