# Semantic Persuasive Copywriter — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude Teams skill that produces Bencivenga-direct-response + SRO copy for CAKE agency work, with deterministic intake routing, auto-grading, medical/legal compliance, and client-brand voice fetching.

**Architecture:** Claude skill (SKILL.md + references/ + scripts/ + tests/). Three modes routed by intent classification. Progressive disclosure via decision matrix. Pre-delivery auto-grading with adversarial enumeration. Python scripts (Code-only) for URL fetch + GitHub brand-guide fetch + client-index sync. Graceful degradation in chat/cowork.

**Tech Stack:** Python 3.11+, `uv` or `pip`, `httpx`, `trafilatura`, `pytest`, `pyyaml`. Markdown for all skill content. YAML for fixture properties.

**Spec reference:** `docs/superpowers/specs/2026-04-20-semantic-persuasive-copywriter-design.md` (this folder).

**Working directory assumption:** All paths in this plan are relative to the skill's root folder, which will be `semantic-persuasive-copywriter/` after Task 1 (currently `sro-bencivenga/`).

---

## Phase 0 — Setup

### Task 1: Rename workbench folder to match skill name

**Files:**
- Rename directory: `sro-bencivenga/` → `semantic-persuasive-copywriter/`

**Why:** Skill `name` in frontmatter must match the parent directory name per Anthropic standards. Fixing at start prevents every subsequent path reference from needing later rewrites.

- [ ] **Step 1: Verify current location**

Run: `ls c:/Users/clark/Code/skills-workbench/`
Expected: includes `sro-bencivenga/` directory.

- [ ] **Step 2: Rename the folder**

```bash
mv c:/Users/clark/Code/skills-workbench/sro-bencivenga c:/Users/clark/Code/skills-workbench/semantic-persuasive-copywriter
```

- [ ] **Step 3: Verify rename succeeded**

Run: `ls c:/Users/clark/Code/skills-workbench/`
Expected: `semantic-persuasive-copywriter/` present, `sro-bencivenga/` absent.

- [ ] **Step 4: Initialize git in the renamed folder (if not already)**

Run:
```bash
cd c:/Users/clark/Code/skills-workbench/semantic-persuasive-copywriter
git init
git add -A
git commit -m "chore: initial import of semantic-persuasive-copywriter workbench"
```

Expected: clean initial commit.

---

### Task 2: Scaffold directory structure and dependency files

**Files:**
- Create: `references/` (empty dir)
- Create: `scripts/` (empty dir)
- Create: `tests/scripts/` (empty dir)
- Create: `tests/skill/` (empty dir)
- Create: `tests/fixtures/` (empty dir)
- Create: `tests/references/` (empty dir)
- Create: `scripts/requirements.txt`
- Create: `scripts/README.md`
- Create: `.gitignore`

- [ ] **Step 1: Create empty subdirectories**

```bash
cd c:/Users/clark/Code/skills-workbench/semantic-persuasive-copywriter
mkdir -p references scripts tests/scripts tests/skill tests/fixtures tests/references
```

- [ ] **Step 2: Write `scripts/requirements.txt`**

```
httpx>=0.27.0
trafilatura>=1.10.0
pyyaml>=6.0
pytest>=8.0.0
```

- [ ] **Step 3: Write `scripts/README.md`**

```markdown
# Scripts

Code-only helpers for the semantic-persuasive-copywriter skill. These run in Claude Code only; chat/cowork fall back to asking the user to paste content.

## Install

```bash
pip install -r requirements.txt
```

## GITHUB_TOKEN setup (required for fetch-brand-guide.py and sync-client-index.py)

Create a **fine-grained** personal access token at https://github.com/settings/personal-access-tokens/new:

1. **Repository access:** Only select repositories → `cake-websites/cake-brand-guidelines`
2. **Permissions:** Contents → Read-only
3. **Expiration:** 90 days (rotate quarterly)

Set the token:

```bash
export GITHUB_TOKEN="github_pat_..."
```

Broad-scope classic PATs are rejected by the scripts' pre-flight scope check.

## Scripts

- `fetch-url.py <url>` — fetch and clean main content from a public URL. Blocks private/reserved IPs (SSRF guard).
- `fetch-brand-guide.py <client-slug>` — fetch concatenated brand guideline files for a client.
- `sync-client-index.py` — regenerate `references/client-index.md` from the brand repo README.
```

- [ ] **Step 4: Write `.gitignore`**

```
__pycache__/
*.pyc
.pytest_cache/
.venv/
venv/
.env
.env.*
```

- [ ] **Step 5: Commit**

```bash
git add scripts/ tests/ references/ .gitignore
git commit -m "chore: scaffold directory structure and dependencies"
```

---

### Task 3: SKILL.md skeleton with frontmatter

**Files:**
- Create: `SKILL.md`

This is a minimal skeleton — just enough for `name` + `description` + `when_to_use` to be valid. Full body comes in Task 17.

- [ ] **Step 1: Write `SKILL.md`**

```markdown
---
name: semantic-persuasive-copywriter
description: Writes persuasive, AI-retrievable copy using Gary Bencivenga's direct-response methods fused with Semantic Retrieval Optimization (SRO). Use whenever you need to rewrite an existing page, draft a new blog post, or write a client newsletter — especially for medical, legal, or other YMYL businesses. Auto-detects the job family from input and routes to the right workflow. Always runs a pre-delivery grading pass (skippable on explicit user override).
when_to_use: "Trigger phrases: 'improve this copy,' 'rewrite this page,' 'make this landing page better,' 'polish this draft,' 'write a blog post about [X],' 'draft an article about [X],' 'need a newsletter for [client],' 'write the [month] newsletter,' 'update this service page,' 'SEO-optimize this,' 'make this more persuasive,' 'apply Bencivenga/SRO to this,' 'produce copy for [client].' Verticals: medical, dental, chiropractic, optometry, veterinary, plastic surgery, med spa, mental health, law firm, immigration law, family law, financial services, insurance — any YMYL or professional-services business. Do NOT use for: technical documentation, API reference material, internal runbooks, developer-facing content, code comments, release notes, internal team communications, or anything where Bencivenga's direct-response voice would be inappropriate. For those tasks, prefer a general-purpose writing approach or a dedicated technical-writing skill."
---

# Semantic Persuasive Copywriter

*Body to be completed in Task 17. See docs/superpowers/specs/2026-04-20-semantic-persuasive-copywriter-design.md §6 for the target shape.*

## Placeholder — do not ship this version
```

- [ ] **Step 2: Verify frontmatter parses**

Create `tests/skill/test_frontmatter.py`:

```python
import yaml
from pathlib import Path


def test_skill_md_frontmatter_parses():
    skill_md = Path(__file__).resolve().parents[2] / "SKILL.md"
    content = skill_md.read_text(encoding="utf-8")
    assert content.startswith("---\n"), "SKILL.md must start with YAML frontmatter"
    _, fm, _ = content.split("---\n", 2)
    data = yaml.safe_load(fm)
    assert data["name"] == "semantic-persuasive-copywriter"
    assert "description" in data
    assert "when_to_use" in data
    combined = len(data["description"]) + len(data["when_to_use"])
    assert combined <= 1536, f"description + when_to_use = {combined}, exceeds 1536 char cap"


def test_skill_md_name_matches_directory():
    skill_md = Path(__file__).resolve().parents[2] / "SKILL.md"
    content = skill_md.read_text(encoding="utf-8")
    _, fm, _ = content.split("---\n", 2)
    data = yaml.safe_load(fm)
    parent_dir = skill_md.parent.name
    assert data["name"] == parent_dir, f"name '{data['name']}' != parent dir '{parent_dir}'"
```

- [ ] **Step 3: Run tests**

Run: `pytest tests/skill/test_frontmatter.py -v`
Expected: both tests PASS.

- [ ] **Step 4: Commit**

```bash
git add SKILL.md tests/skill/test_frontmatter.py
git commit -m "feat: add SKILL.md skeleton with valid frontmatter"
```

---

## Phase 1 — Reference-file linter

### Task 4: Build reference-file property linter

**Files:**
- Create: `tests/references/__init__.py`
- Create: `tests/references/test_reference_properties.py`
- Create: `tests/references/properties.yaml`

**Why:** Reference files are prose, but their *structure* is checkable. Authoring each reference file against a property spec (required sections, minimum citations, word-count bounds) keeps them aligned with the design spec and prevents drift.

- [ ] **Step 1: Write `tests/references/properties.yaml` (initial schema, populated as files are authored)**

```yaml
# Each entry describes assertable properties for a reference file.
# Properties are checked by test_reference_properties.py.
# Add an entry when you author a reference file in later tasks.

# Example shape (no real entries yet — added as files are written):
# client-index.md:
#   min_words: 50
#   required_sections: ["## Clients"]
#   required_header_keys: ["last_synced"]

{}
```

- [ ] **Step 2: Write the linter test file**

```python
# tests/references/test_reference_properties.py
import re
from pathlib import Path

import pytest
import yaml

REPO_ROOT = Path(__file__).resolve().parents[2]
REFERENCES_DIR = REPO_ROOT / "references"
PROPERTIES_FILE = Path(__file__).parent / "properties.yaml"


def load_properties():
    if not PROPERTIES_FILE.exists():
        return {}
    return yaml.safe_load(PROPERTIES_FILE.read_text(encoding="utf-8")) or {}


def get_reference_files():
    if not REFERENCES_DIR.exists():
        return []
    return sorted([p.name for p in REFERENCES_DIR.glob("*.md")])


@pytest.mark.parametrize("filename", list(load_properties().keys()))
def test_reference_file_exists(filename):
    assert (REFERENCES_DIR / filename).exists(), f"{filename} referenced in properties.yaml but not found in references/"


@pytest.mark.parametrize("filename", list(load_properties().keys()))
def test_reference_file_properties(filename):
    props = load_properties()[filename]
    path = REFERENCES_DIR / filename
    if not path.exists():
        pytest.skip(f"{filename} not yet authored")
    content = path.read_text(encoding="utf-8")

    if "min_words" in props:
        word_count = len(content.split())
        assert word_count >= props["min_words"], f"{filename}: {word_count} words < min {props['min_words']}"

    if "max_words" in props:
        word_count = len(content.split())
        assert word_count <= props["max_words"], f"{filename}: {word_count} words > max {props['max_words']}"

    if "required_sections" in props:
        for section in props["required_sections"]:
            assert section in content, f"{filename}: missing required section '{section}'"

    if "min_citations" in props:
        # Count parenthetical citations like "(FTC 16 CFR §255.1)" or "[FDA 21 CFR §101]"
        citation_pattern = r"\([A-Z][A-Za-z ]+(?: \d+)? ?(?:CFR|§|USC)[^\)]*\)|\[[A-Z][^\]]*(?:CFR|§|USC)[^\]]*\]"
        citations = re.findall(citation_pattern, content)
        assert len(citations) >= props["min_citations"], f"{filename}: {len(citations)} citations < min {props['min_citations']}"

    if "required_header_keys" in props:
        # Header block of form: **key:** value
        for key in props["required_header_keys"]:
            pattern = rf"\*\*{re.escape(key)}:\*\*"
            assert re.search(pattern, content), f"{filename}: missing header key '{key}'"
```

- [ ] **Step 3: Run linter — expect all parameterized tests to be skipped (no entries in properties.yaml yet)**

Run: `pytest tests/references/test_reference_properties.py -v`
Expected: 0 failures, 0 tests actually run (empty parametrize).

- [ ] **Step 4: Commit**

```bash
git add tests/references/
git commit -m "feat: add reference-file property linter framework"
```

---

## Phase 2 — Scripts (TDD)

### Task 5: `sync-client-index.py`

**Files:**
- Create: `scripts/sync_client_index.py` (underscore name for importability)
- Create: `scripts/__init__.py` (empty)
- Create: `tests/scripts/__init__.py` (empty)
- Create: `tests/scripts/test_sync_client_index.py`
- Create: `tests/scripts/fixtures/brand_repo_readme_sample.md`

- [ ] **Step 1: Install deps**

```bash
pip install -r scripts/requirements.txt
```

- [ ] **Step 2: Capture a real sample of the brand repo README**

Run in Code (has GitHub MCP): fetch `cake-websites/cake-brand-guidelines/README.md` and save first 2KB to `tests/scripts/fixtures/brand_repo_readme_sample.md`. If GitHub MCP unavailable, use this minimal sample for testing:

```markdown
# Brand Guidelines

> Auto-generated by `generate-index.py` — last updated 2026-02-26 at 14:59 UTC

---

## Clients

- [Aesthetic Nirvana](clients/aesthetic-nirvana/README.md) — Look Good. Feel Good.
- [Berks Plastic Surgery](clients/berks-plastic-surgery/README.md) — It's all about confidence.
- [Kirby Plastic Surgery](clients/kirby-plastic-surgery/README.md) — [TO BE CONFIRMED]

---
```

- [ ] **Step 3: Write the failing test for README parsing**

```python
# tests/scripts/test_sync_client_index.py
from pathlib import Path

import pytest

from scripts.sync_client_index import parse_clients_from_readme

SAMPLE = Path(__file__).parent / "fixtures" / "brand_repo_readme_sample.md"


def test_parse_clients_extracts_slug_name_tagline():
    content = SAMPLE.read_text(encoding="utf-8")
    clients = parse_clients_from_readme(content)
    assert {"slug": "aesthetic-nirvana", "name": "Aesthetic Nirvana", "tagline": "Look Good. Feel Good."} in clients
    assert {"slug": "berks-plastic-surgery", "name": "Berks Plastic Surgery", "tagline": "It's all about confidence."} in clients


def test_parse_clients_handles_tbc_tagline():
    content = SAMPLE.read_text(encoding="utf-8")
    clients = parse_clients_from_readme(content)
    kirby = next(c for c in clients if c["slug"] == "kirby-plastic-surgery")
    assert kirby["tagline"] == "[TO BE CONFIRMED]"


def test_parse_clients_empty_input_returns_empty_list():
    assert parse_clients_from_readme("") == []
```

- [ ] **Step 4: Run tests — expect import error**

Run: `pytest tests/scripts/test_sync_client_index.py -v`
Expected: FAIL with `ModuleNotFoundError: No module named 'scripts.sync_client_index'`.

- [ ] **Step 5: Write minimal `scripts/sync_client_index.py`**

```python
"""Regenerate references/client-index.md from cake-brand-guidelines README."""

from __future__ import annotations

import os
import re
import sys
from datetime import datetime, timezone
from pathlib import Path

import httpx

BRAND_REPO_OWNER = "cake-websites"
BRAND_REPO_NAME = "cake-brand-guidelines"
README_PATH = "README.md"

CLIENT_LINE_PATTERN = re.compile(
    r"^- \[(?P<name>[^\]]+)\]\(clients/(?P<slug>[^/]+)/README\.md\)(?: — (?P<tagline>.+))?$"
)


def parse_clients_from_readme(content: str) -> list[dict]:
    """Extract client entries from the brand repo README.

    Each entry: {"slug": ..., "name": ..., "tagline": ...}.
    Missing tagline becomes empty string.
    """
    clients: list[dict] = []
    for line in content.splitlines():
        match = CLIENT_LINE_PATTERN.match(line.strip())
        if not match:
            continue
        clients.append({
            "slug": match.group("slug"),
            "name": match.group("name"),
            "tagline": (match.group("tagline") or "").strip(),
        })
    return clients


def fetch_readme(token: str) -> str:
    url = f"https://api.github.com/repos/{BRAND_REPO_OWNER}/{BRAND_REPO_NAME}/contents/{README_PATH}"
    headers = {
        "Authorization": f"Bearer {token}",
        "Accept": "application/vnd.github.raw",
        "X-GitHub-Api-Version": "2022-11-28",
    }
    response = httpx.get(url, headers=headers, timeout=15.0)
    response.raise_for_status()
    return response.text


def render_client_index(clients: list[dict]) -> str:
    now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")
    lines = [
        "# Client Index",
        "",
        f"**Last synced:** {now}",
        "",
        "> If a recent client is not in this index, run `scripts/sync_client_index.py` (Claude Code) or fetch `cake-websites/cake-brand-guidelines/README.md` directly.",
        "",
        "| Slug | Client Name | Tagline |",
        "|------|-------------|---------|",
    ]
    for c in clients:
        tagline = c["tagline"].replace("|", "\\|") or "—"
        lines.append(f"| `{c['slug']}` | {c['name']} | {tagline} |")
    lines.append("")
    return "\n".join(lines)


def main() -> int:
    token = os.environ.get("GITHUB_TOKEN")
    if not token:
        print("ERROR: GITHUB_TOKEN not set. See scripts/README.md for setup.", file=sys.stderr)
        return 1

    try:
        readme = fetch_readme(token)
    except httpx.HTTPError as exc:
        print(f"ERROR: failed to fetch brand repo README: {exc}", file=sys.stderr)
        return 1

    clients = parse_clients_from_readme(readme)
    output = render_client_index(clients)

    target = Path(__file__).resolve().parent.parent / "references" / "client-index.md"
    target.parent.mkdir(parents=True, exist_ok=True)
    target.write_text(output, encoding="utf-8")
    print(f"Wrote {len(clients)} clients to {target}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

- [ ] **Step 6: Run parser tests — expect PASS**

Run: `pytest tests/scripts/test_sync_client_index.py -v`
Expected: 3 tests PASS.

- [ ] **Step 7: Add render test**

Append to `tests/scripts/test_sync_client_index.py`:

```python
from scripts.sync_client_index import render_client_index


def test_render_client_index_has_header_and_table():
    clients = [{"slug": "foo", "name": "Foo Clinic", "tagline": "Better health."}]
    output = render_client_index(clients)
    assert "# Client Index" in output
    assert "**Last synced:**" in output
    assert "| `foo` | Foo Clinic | Better health. |" in output


def test_render_client_index_empty_tagline_shows_dash():
    clients = [{"slug": "foo", "name": "Foo Clinic", "tagline": ""}]
    output = render_client_index(clients)
    assert "| `foo` | Foo Clinic | — |" in output
```

- [ ] **Step 8: Run all sync tests — expect PASS**

Run: `pytest tests/scripts/test_sync_client_index.py -v`
Expected: 5 tests PASS.

- [ ] **Step 9: Commit**

```bash
git add scripts/sync_client_index.py scripts/__init__.py tests/scripts/ tests/scripts/fixtures/
git commit -m "feat: add sync_client_index script with README parser and tests"
```

---

### Task 6: Generate initial `references/client-index.md`

**Files:**
- Create: `references/client-index.md` (generated by script)
- Modify: `tests/references/properties.yaml`

- [ ] **Step 1: Run the sync script**

```bash
export GITHUB_TOKEN=<your fine-grained PAT>
python scripts/sync_client_index.py
```

Expected output: `Wrote <N> clients to references/client-index.md` where N matches the brand repo's current client count (~25).

- [ ] **Step 2: Inspect the generated file**

Run: `head -20 references/client-index.md`
Expected: `# Client Index`, `**Last synced:** <timestamp>`, staleness note, table header, at least 20 rows.

- [ ] **Step 3: Add properties.yaml entry for client-index.md**

Replace the `{}` placeholder in `tests/references/properties.yaml` with:

```yaml
client-index.md:
  min_words: 50
  required_sections:
    - "# Client Index"
    - "| Slug | Client Name | Tagline |"
  required_header_keys:
    - "Last synced"
```

- [ ] **Step 4: Run linter against client-index.md**

Run: `pytest tests/references/test_reference_properties.py -v`
Expected: 2 tests PASS (exists + properties).

- [ ] **Step 5: Commit**

```bash
git add references/client-index.md tests/references/properties.yaml
git commit -m "feat: generate initial client-index.md"
```

---

### Task 7: `fetch-brand-guide.py` with token-scope guard

**Files:**
- Create: `scripts/fetch_brand_guide.py`
- Create: `tests/scripts/test_fetch_brand_guide.py`

- [ ] **Step 1: Write failing test — missing token exits 1**

```python
# tests/scripts/test_fetch_brand_guide.py
import subprocess
import sys
from pathlib import Path

SCRIPT = Path(__file__).resolve().parents[2] / "scripts" / "fetch_brand_guide.py"


def run_script(args, env=None):
    return subprocess.run(
        [sys.executable, str(SCRIPT), *args],
        capture_output=True,
        text=True,
        env=env or {},
    )


def test_missing_token_exits_1():
    result = run_script(["some-slug"], env={"PATH": ""})
    assert result.returncode == 1
    assert "GITHUB_TOKEN" in result.stderr


def test_missing_slug_arg_exits_1():
    result = run_script([], env={"GITHUB_TOKEN": "fake"})
    assert result.returncode == 1
    assert "slug" in result.stderr.lower() or "usage" in result.stderr.lower()
```

- [ ] **Step 2: Run tests — expect import error (script doesn't exist)**

Run: `pytest tests/scripts/test_fetch_brand_guide.py -v`
Expected: both FAIL (script not found).

- [ ] **Step 3: Write minimal `scripts/fetch_brand_guide.py`**

```python
"""Fetch brand guideline files for a CAKE client from cake-brand-guidelines repo."""

from __future__ import annotations

import os
import sys
from datetime import datetime, timezone

import httpx

BRAND_REPO_OWNER = "cake-websites"
BRAND_REPO_NAME = "cake-brand-guidelines"

REQUIRED_FILES = [
    ("Overview (README.md)", "README.md"),
    ("Personality & Tone", "guidelines/personality-and-tone.md"),
    ("Target Audiences", "guidelines/target-audiences.md"),
    ("Foundations", "guidelines/foundations.md"),
]
OPTIONAL_FILES = [
    ("Compliance", "guidelines/compliance.md"),
]


def check_token_scope(token: str) -> tuple[bool, str]:
    """Pre-flight check: token must read the brand repo and not have broader write/admin."""
    headers = {"Authorization": f"Bearer {token}", "X-GitHub-Api-Version": "2022-11-28"}

    # Basic reachability + read permission on the target repo
    repo_url = f"https://api.github.com/repos/{BRAND_REPO_OWNER}/{BRAND_REPO_NAME}"
    try:
        repo_resp = httpx.get(repo_url, headers=headers, timeout=10.0)
    except httpx.HTTPError as exc:
        return False, f"network error contacting GitHub: {exc}"
    if repo_resp.status_code == 401:
        return False, "token rejected by GitHub (401). Check token validity."
    if repo_resp.status_code == 404:
        return False, "token cannot read cake-websites/cake-brand-guidelines (404). Grant Contents: Read-only."
    if repo_resp.status_code != 200:
        return False, f"unexpected status {repo_resp.status_code} from GitHub"

    # If permissions block is present, verify push/admin are false
    perms = repo_resp.json().get("permissions", {})
    if perms.get("push") or perms.get("admin"):
        return False, (
            "token has more access than required. Use a fine-grained PAT scoped to "
            "cake-brand-guidelines, Contents: Read-only."
        )
    return True, ""


def fetch_file(token: str, slug: str, path_in_repo: str) -> tuple[int, str]:
    url = (
        f"https://api.github.com/repos/{BRAND_REPO_OWNER}/{BRAND_REPO_NAME}/"
        f"contents/clients/{slug}/{path_in_repo}"
    )
    headers = {
        "Authorization": f"Bearer {token}",
        "Accept": "application/vnd.github.raw",
        "X-GitHub-Api-Version": "2022-11-28",
    }
    response = httpx.get(url, headers=headers, timeout=15.0)
    return response.status_code, response.text


def main(argv: list[str]) -> int:
    if len(argv) < 1:
        print("Usage: python fetch_brand_guide.py <client-slug>", file=sys.stderr)
        return 1
    slug = argv[0]

    token = os.environ.get("GITHUB_TOKEN")
    if not token:
        print(
            "ERROR: GITHUB_TOKEN not set. See scripts/README.md for fine-grained PAT setup.",
            file=sys.stderr,
        )
        return 1

    ok, err = check_token_scope(token)
    if not ok:
        print(f"Refused: {err}", file=sys.stderr)
        return 3

    now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")
    output_blocks = [f"# Brand Guide: {slug}", "", f"**Slug:** `{slug}`", f"**Fetched:** {now}", ""]

    for section_title, path_in_repo in REQUIRED_FILES:
        status, body = fetch_file(token, slug, path_in_repo)
        if status == 404 and path_in_repo == "README.md":
            print(f"Client slug '{slug}' not found in repo", file=sys.stderr)
            return 1
        if status == 200:
            output_blocks.extend([f"## {section_title}", "", body.rstrip(), ""])
        else:
            output_blocks.extend([f"## {section_title}", "", f"_(missing: HTTP {status})_", ""])

    for section_title, path_in_repo in OPTIONAL_FILES:
        status, body = fetch_file(token, slug, path_in_repo)
        if status == 200:
            output_blocks.extend([f"## {section_title}", "", body.rstrip(), ""])
        else:
            output_blocks.extend([
                f"## {section_title}",
                "",
                "_No client-specific compliance file. Apply baseline medical-legal guardrails._",
                "",
            ])

    print("\n".join(output_blocks))
    return 0


if __name__ == "__main__":
    sys.exit(main(sys.argv[1:]))
```

- [ ] **Step 4: Run initial tests — expect PASS**

Run: `pytest tests/scripts/test_fetch_brand_guide.py -v`
Expected: 2 tests PASS.

- [ ] **Step 5: Add broad-scope token test (integration, gated by env var)**

Append to the test file:

```python
import os
import pytest


@pytest.mark.skipif(
    "GITHUB_TOKEN_BROAD_SCOPE" not in os.environ,
    reason="requires a known-broad-scope PAT for the scope guard test; skipped by default",
)
def test_broad_scope_token_exits_3():
    result = run_script(["aesthetic-nirvana"], env={
        "PATH": os.environ.get("PATH", ""),
        "GITHUB_TOKEN": os.environ["GITHUB_TOKEN_BROAD_SCOPE"],
    })
    assert result.returncode == 3
    assert "more access than required" in result.stderr


@pytest.mark.skipif(
    "GITHUB_TOKEN" not in os.environ,
    reason="requires a valid fine-grained PAT; skipped in CI unless set",
)
def test_real_fetch_succeeds_for_known_slug():
    result = run_script(["aesthetic-nirvana"], env={
        "PATH": os.environ.get("PATH", ""),
        "GITHUB_TOKEN": os.environ["GITHUB_TOKEN"],
    })
    assert result.returncode == 0
    assert "# Brand Guide: aesthetic-nirvana" in result.stdout
    assert "## Personality & Tone" in result.stdout


@pytest.mark.skipif(
    "GITHUB_TOKEN" not in os.environ,
    reason="requires a valid fine-grained PAT",
)
def test_unknown_slug_exits_1():
    result = run_script(["this-slug-does-not-exist-xyz"], env={
        "PATH": os.environ.get("PATH", ""),
        "GITHUB_TOKEN": os.environ["GITHUB_TOKEN"],
    })
    assert result.returncode == 1
    assert "not found" in result.stderr.lower()
```

- [ ] **Step 6: Run full test suite**

Run: `pytest tests/scripts/test_fetch_brand_guide.py -v`
Expected: 2 PASS, 3 SKIP (or 4–5 PASS if env vars set). If `GITHUB_TOKEN` is set and valid, the real-fetch and unknown-slug tests must PASS.

- [ ] **Step 7: Commit**

```bash
git add scripts/fetch_brand_guide.py tests/scripts/test_fetch_brand_guide.py
git commit -m "feat: add fetch_brand_guide script with token-scope guard"
```

---

### Task 8: `fetch-url.py` with SSRF guard

**Files:**
- Create: `scripts/fetch_url.py`
- Create: `tests/scripts/test_fetch_url.py`

- [ ] **Step 1: Write failing test — scheme + private IP guards**

```python
# tests/scripts/test_fetch_url.py
import subprocess
import sys
from pathlib import Path

SCRIPT = Path(__file__).resolve().parents[2] / "scripts" / "fetch_url.py"


def run_script(args):
    return subprocess.run(
        [sys.executable, str(SCRIPT), *args],
        capture_output=True,
        text=True,
    )


def test_missing_url_arg_exits_1():
    result = run_script([])
    assert result.returncode == 1


def test_non_http_scheme_exits_1():
    result = run_script(["file:///etc/passwd"])
    assert result.returncode == 1
    assert "scheme" in result.stderr.lower() or "http" in result.stderr.lower()


def test_loopback_url_exits_2_ssrf_guard():
    result = run_script(["http://127.0.0.1/admin"])
    assert result.returncode == 2
    assert "private" in result.stderr.lower() or "reserved" in result.stderr.lower()


def test_private_rfc1918_exits_2_ssrf_guard():
    result = run_script(["http://10.0.0.1/"])
    assert result.returncode == 2


def test_link_local_exits_2_ssrf_guard():
    result = run_script(["http://169.254.169.254/latest/meta-data/"])
    assert result.returncode == 2


def test_hostname_resolving_to_private_exits_2():
    # localhost resolves to 127.0.0.1
    result = run_script(["http://localhost/"])
    assert result.returncode == 2
```

- [ ] **Step 2: Run tests — expect all FAIL (script missing)**

Run: `pytest tests/scripts/test_fetch_url.py -v`
Expected: all FAIL.

- [ ] **Step 3: Write `scripts/fetch_url.py`**

```python
"""Fetch and clean main content from a public URL (with SSRF guard)."""

from __future__ import annotations

import ipaddress
import socket
import sys
from datetime import datetime, timezone
from urllib.parse import urlparse

import httpx
import trafilatura

USER_AGENT = "CAKE-SemanticCopywriter/1.0"
TIMEOUT_SECONDS = 15.0


def is_private_ip(ip_str: str) -> bool:
    try:
        ip = ipaddress.ip_address(ip_str)
    except ValueError:
        return True  # fail closed on malformed addresses
    return (
        ip.is_private
        or ip.is_loopback
        or ip.is_link_local
        or ip.is_multicast
        or ip.is_reserved
        or ip.is_unspecified
    )


def resolve_all_ips(hostname: str) -> list[str]:
    try:
        infos = socket.getaddrinfo(hostname, None)
    except socket.gaierror:
        return []
    return sorted({info[4][0] for info in infos})


def validate_url_or_die(url: str) -> str:
    """Return the URL if it passes scheme + SSRF checks; otherwise sys.exit with the right code."""
    parsed = urlparse(url)
    if parsed.scheme not in ("http", "https"):
        print(f"ERROR: refused unsupported scheme '{parsed.scheme}'. Use http or https.", file=sys.stderr)
        sys.exit(1)
    hostname = parsed.hostname
    if not hostname:
        print("ERROR: URL has no hostname", file=sys.stderr)
        sys.exit(1)
    ips = resolve_all_ips(hostname)
    if not ips:
        print(f"ERROR: could not resolve hostname '{hostname}'", file=sys.stderr)
        sys.exit(1)
    for ip in ips:
        if is_private_ip(ip):
            print(
                f"Refused: target '{hostname}' resolves to a private/reserved IP range ({ip})",
                file=sys.stderr,
            )
            sys.exit(2)
    return url


def fetch_and_extract(url: str) -> str | None:
    headers = {"User-Agent": USER_AGENT}
    try:
        with httpx.Client(follow_redirects=False, timeout=TIMEOUT_SECONDS) as client:
            response = client.get(url, headers=headers)
            # Manually follow redirects with SSRF re-check at each hop
            redirects = 0
            while response.is_redirect and redirects < 5:
                new_location = response.headers.get("location", "")
                if not new_location:
                    break
                if new_location.startswith("/"):
                    parsed = urlparse(url)
                    new_location = f"{parsed.scheme}://{parsed.netloc}{new_location}"
                validate_url_or_die(new_location)  # re-check each redirect target
                response = client.get(new_location, headers=headers)
                url = new_location
                redirects += 1
        response.raise_for_status()
        html = response.text
    except httpx.HTTPError as exc:
        print(f"ERROR: fetch failed: {exc}", file=sys.stderr)
        return None

    extracted = trafilatura.extract(
        html,
        include_comments=False,
        include_tables=False,
        output_format="markdown",
        favor_precision=True,
    )
    return extracted


def main(argv: list[str]) -> int:
    if len(argv) < 1:
        print("Usage: python fetch_url.py <url>", file=sys.stderr)
        return 1
    url = argv[0]
    url = validate_url_or_die(url)  # exits 1 or 2 if invalid

    body = fetch_and_extract(url)
    if body is None:
        return 1

    now = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M UTC")
    print(f"**URL:** {url}")
    print(f"**Fetched:** {now}")
    print()
    print(body.strip())
    return 0


if __name__ == "__main__":
    sys.exit(main(sys.argv[1:]))
```

- [ ] **Step 4: Run guard tests — expect PASS**

Run: `pytest tests/scripts/test_fetch_url.py -v`
Expected: all 6 tests PASS.

- [ ] **Step 5: Add integration test for a real public URL (network-required, gated)**

Append:

```python
import os
import pytest


@pytest.mark.skipif(
    os.environ.get("SKIP_NETWORK_TESTS") == "1",
    reason="network test skipped by SKIP_NETWORK_TESTS=1",
)
def test_real_public_url_extracts_content():
    # example.com is stable, public, no paywall.
    result = run_script(["https://example.com/"])
    assert result.returncode == 0
    assert "Example Domain" in result.stdout or "example" in result.stdout.lower()
    assert "**URL:** https://example.com/" in result.stdout
    assert "**Fetched:**" in result.stdout
```

- [ ] **Step 6: Run full suite**

Run: `pytest tests/scripts/test_fetch_url.py -v`
Expected: 7 tests PASS (or 6 PASS + 1 SKIP if `SKIP_NETWORK_TESTS=1`).

- [ ] **Step 7: Commit**

```bash
git add scripts/fetch_url.py tests/scripts/test_fetch_url.py
git commit -m "feat: add fetch_url script with SSRF guard and redirect re-check"
```

---

## Phase 3 — Reference files (content with property-driven TDD)

For each content file: first add the assertable properties to `tests/references/properties.yaml`, run the linter (expect fail), author the file, run the linter (expect pass), commit.

### Task 9: `grading-rubric.md`

**Files:**
- Modify: `tests/references/properties.yaml`
- Create: `references/grading-rubric.md`

- [ ] **Step 1: Add properties entry**

Append to `tests/references/properties.yaml`:

```yaml
grading-rubric.md:
  min_words: 700
  max_words: 2000
  required_sections:
    - "# Grading Rubric"
    - "## Bencivenga (7 checks)"
    - "## SRO (7 checks)"
    - "## E-E-A-T (4 checks)"
    - "## Adversarial enumeration protocol"
    - "## Per-item subset (newsletter mode)"
    - "## Iteration cap"
    - "## Scoring and grade scale"
    - "## Auto-revise protocol"
    - "## Worked example"
    - "## Grade report output template"
```

- [ ] **Step 2: Run linter — expect FAIL (file missing)**

Run: `pytest tests/references/test_reference_properties.py -k grading-rubric -v`
Expected: FAIL (file doesn't exist).

- [ ] **Step 3: Author `references/grading-rubric.md`**

Write the file covering:
- 18 total checks numbered 1–18, grouped: Bencivenga 1–7, SRO 8–14, E-E-A-T 15–18.
- Each check framed as a **mechanical enumeration instruction**, not a yes/no vibe check. Examples provided inline (from spec §7):
  - Entity clarity → "List every pronoun + antecedent; count ambiguous; pass if zero."
  - Semantic triplets → "List every core claim; extract S-P-O; count subject-less; pass if zero."
  - Proof visibility → "List every proof element + location; pass if ≥1 in headline or lead."
  - Query-aligned headings → "Copy every heading; for each state the likely search query; pass if all answer a query directly."
- Per-item subset for newsletter: exactly 7 checks (urgent problem, specific promise, proof element, clear CTA, named entity, triplet structure, query-aligned headline). Whole-letter remaining 11 checks listed separately.
- Iteration cap rule stated explicitly: "Revise once. If second grade still has Required No items, deliver with gap notes. Never loop a third time."
- Scoring: `score = (pass_count / 18) × 100`. Grade scale: A 90+, B 80–89, C 70–79, D/F <70.
- Auto-revise protocol: for each Required-No, show the prescribed fix action (e.g., "missing proof in headline → add a named testimonial or credential anchor before the lead").
- Worked example: weak draft shown → enumerated grading showing Nos → revised draft → re-graded showing all Yeses. Example is ~150 words of copy so the worked example stays concrete.
- Grade report output template: a literal markdown block showing exactly how the report appears in delivered copy.

- [ ] **Step 4: Run linter — expect PASS**

Run: `pytest tests/references/test_reference_properties.py -k grading-rubric -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add references/grading-rubric.md tests/references/properties.yaml
git commit -m "feat: add grading-rubric reference with adversarial enumeration protocol"
```

---

### Task 10: `sro-principles.md`

**Files:**
- Modify: `tests/references/properties.yaml`
- Create: `references/sro-principles.md`

- [ ] **Step 1: Add properties entry**

```yaml
sro-principles.md:
  min_words: 600
  max_words: 900
  required_sections:
    - "# SRO Principles"
    - "## The Persuasion Equation anchored to semantic search"
    - "## Entity clarity"
    - "## Semantic triplets"
    - "## Query-aligned headings"
    - "## Atomic chunks"
    - "## E-E-A-T essentials"
    - "## Descriptive anchor text"
```

- [ ] **Step 2: Run linter — expect FAIL**

Run: `pytest tests/references/test_reference_properties.py -k sro-principles -v`
Expected: FAIL.

- [ ] **Step 3: Author `references/sro-principles.md`**

Per spec §7 authoring requirements:
- 600–900 words total. Keep examples terse.
- 7 required sections in the order specified.
- Source: distill `context/semantic-copywriting-project/semantic-copywriting-guidelines.md`.
- Remove every mention of Cora. Every example must be medical or legal.
- For "Semantic triplets" section include a 4-row table: vague → retrievable, all medical/legal examples.

- [ ] **Step 4: Run linter — expect PASS**

Run: `pytest tests/references/test_reference_properties.py -k sro-principles -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add references/sro-principles.md tests/references/properties.yaml
git commit -m "feat: add sro-principles reference (always-loaded baseline)"
```

---

### Task 11: `compliance-medical-legal.md` (with citation linter)

**Files:**
- Modify: `tests/references/properties.yaml`
- Create: `references/compliance-medical-legal.md`

- [ ] **Step 1: Add properties entry**

```yaml
compliance-medical-legal.md:
  min_words: 800
  max_words: 2500
  min_citations: 8
  required_sections:
    - "# Compliance — Medical & Legal"
    - "## FTC testimonial rules"
    - "## Medical advertising rules"
    - "## Legal advertising rules"
    - "## E-E-A-T hardening for YMYL"
    - "## Author bio block requirements"
    - "## Red-flag phrases"
    - "## Client-specific compliance override"
```

- [ ] **Step 2: Run linter — expect FAIL**

Run: `pytest tests/references/test_reference_properties.py -k compliance-medical-legal -v`
Expected: FAIL (file missing, also citations=0 < 8).

- [ ] **Step 3: Author `references/compliance-medical-legal.md`**

Required content:
- Header callout at the very top: *"This file encodes regulatory guardrails for copywriting, not legal advice. Consult counsel for jurisdiction-specific questions."*
- Below the callout: reviewer + review-date lines as placeholders for attorney sign-off (see Task 21):
  - `**Reviewed by:** _(pending attorney review — see Task 21)_`
  - `**Review date:** _(pending)_`
- Every rule cites its source inline. Examples of required citations (at minimum 8 total):
  - FTC Endorsement Guide (16 CFR §255)
  - FTC typical-results disclosure guidance
  - FDA 21 CFR §101 (food/supplement claims) or §201 (prescription drug labeling) depending on the rule
  - ABA Model Rule 7.1 (communications concerning a lawyer's services)
  - ABA Model Rule 7.2 (advertising), 7.3 (solicitation), 7.4 (fields of practice) as applicable
  - State-bar examples: cite at least 2 states relevant to CAKE clients (e.g., Texas Disciplinary Rule 7.02, California Rule 1-400).
- Citation format: parenthetical with statute/rule name and § marker, e.g., `(FTC 16 CFR §255.1)` or `(ABA Model Rule 7.1)`. Linter matches on that pattern — if citations are formatted differently, update the linter regex.
- Red-flag phrases list: "guaranteed," "miracle," "no risk," "cure," "100% safe," "always wins," "the best," plus medical-specific ("FDA-approved" without actual approval, "clinically proven" without a cited study).
- Client-specific override note: if the client's `guidelines/compliance.md` in the brand repo exists, its rules SUPPLEMENT this baseline. Conflicts resolve in favor of the stricter rule.
- Date-stamp each rule with its source-document year; flag rules older than 3 years for review.

- [ ] **Step 4: Run linter — expect PASS**

Run: `pytest tests/references/test_reference_properties.py -k compliance-medical-legal -v`
Expected: PASS (8+ citations, all sections present).

- [ ] **Step 5: Commit**

```bash
git add references/compliance-medical-legal.md tests/references/properties.yaml
git commit -m "feat: add compliance-medical-legal reference with cited guardrails (attorney review pending)"
```

---

### Task 12: `mode-rewrite.md`

**Files:**
- Modify: `tests/references/properties.yaml`
- Create: `references/mode-rewrite.md`

- [ ] **Step 1: Add properties entry**

```yaml
mode-rewrite.md:
  min_words: 400
  max_words: 1200
  required_sections:
    - "# Mode: Rewrite"
    - "## Intake"
    - "## Source fetch"
    - "## Diagnostic pass"
    - "## Pre-write mini-checklist"
    - "## Produce"
    - "## Output template"
    - "## Keep-vs-cut guidance"
```

- [ ] **Step 2: Run linter — expect FAIL**

Run: `pytest tests/references/test_reference_properties.py -k mode-rewrite -v`

- [ ] **Step 3: Author `references/mode-rewrite.md`**

Per spec §7 `mode-rewrite.md` scope:
- Intake behavior: just-go, minimal questions (unless critical info missing).
- Source fetch: Claude Code uses `scripts/fetch_url.py`; chat/cowork uses WebFetch or paste.
- Diagnostic: inline notes on source weaknesses (proof gaps, weak headline, missing CTA, entity ambiguity).
- Pre-write mini-checklist: audience, primary CTA, proof assets, unique promise.
- Produce flow: headline → lead → body → proof blocks → CTA → P.S.
- Output template: literal markdown showing H1 / H2 / H3 / meta title / meta description / image alt / grade report block.
- Keep-vs-cut: respect existing proof, strip weak copy, preserve brand-specific disclaimers.

- [ ] **Step 4: Run linter — expect PASS**

- [ ] **Step 5: Commit**

```bash
git add references/mode-rewrite.md tests/references/properties.yaml
git commit -m "feat: add mode-rewrite reference"
```

---

### Task 13: `mode-blog-post.md`

**Files:**
- Modify: `tests/references/properties.yaml`
- Create: `references/mode-blog-post.md`

- [ ] **Step 1: Add properties entry**

```yaml
mode-blog-post.md:
  min_words: 400
  max_words: 1200
  required_sections:
    - "# Mode: Blog Post"
    - "## Smart intake"
    - "## Outline"
    - "## Produce"
    - "## Output template"
    - "## Conditional archive loading"
```

- [ ] **Step 2: Run linter — expect FAIL**

- [ ] **Step 3: Author `references/mode-blog-post.md`**

Per spec §7:
- Smart intake: scan brief for audience, angle, keyword, CTA goal. If 3+ missing, ask up to 3 targeted questions in one batched message. If rich, skip.
- Outline: search-intent-matched H2/H3 structure. Answer-first paragraphs.
- Produce: semantic triplets, named-entity proof, CTA integration.
- Output template: H1 / H2 / H3 / meta title / meta description / image alt / suggested URL slug / grade report.
- Conditional archive loading: if post is health/financial/legal advice-heavy AND user or routing signal requests pattern reference, load a relevant `example-*.md`. Default: don't load archives.

- [ ] **Step 4: Run linter — expect PASS**

- [ ] **Step 5: Commit**

```bash
git add references/mode-blog-post.md tests/references/properties.yaml
git commit -m "feat: add mode-blog-post reference"
```

---

### Task 14: `mode-newsletter.md` (with per-item grading split)

**Files:**
- Modify: `tests/references/properties.yaml`
- Create: `references/mode-newsletter.md`

- [ ] **Step 1: Add properties entry**

```yaml
mode-newsletter.md:
  min_words: 500
  max_words: 1500
  required_sections:
    - "# Mode: Newsletter"
    - "## Guided intake"
    - "## Brand guide load"
    - "## Per-item structure"
    - "## Envelope elements"
    - "## Grading split"
    - "## Output template"
```

- [ ] **Step 2: Run linter — expect FAIL**

- [ ] **Step 3: Author `references/mode-newsletter.md`**

Per spec §7:
- Guided intake (5 prompts batched in one question): client slug (confirm), audience, month/theme, topic ideas (offer to generate 5), offers/events to highlight.
- Brand guide load: always fetch — voice consistency critical.
- Per-item structure: curiosity headline (8–14 words) → 100–250 word body → single-action CTA.
- Envelope elements: subject line (≤50 chars), preview text (≤90 chars), header block, footer block.
- Grading split:
  - Per-item (7 checks from grading-rubric.md §"Per-item subset"): urgent problem, specific promise, proof element, clear CTA, named entity, triplet structure, query-aligned headline.
  - Whole-newsletter (remaining 11 checks): subject-line strength, preview-text complement, author block, citations present, update date, anchor-text quality, voice consistency, compliance pass, cross-item topic coherence, schema-ready structure, final grade.
  - Auto-revise only items that fail. Do NOT re-grade the whole newsletter after a per-item fix.
- Output template: subject + preview + header + per-item blocks + footer + grade report.

- [ ] **Step 4: Run linter — expect PASS**

- [ ] **Step 5: Commit**

```bash
git add references/mode-newsletter.md tests/references/properties.yaml
git commit -m "feat: add mode-newsletter reference with grading split"
```

---

### Task 15: `bencivenga-methods.md`

**Files:**
- Modify: `tests/references/properties.yaml`
- Create: `references/bencivenga-methods.md`

- [ ] **Step 1: Add properties entry**

```yaml
bencivenga-methods.md:
  min_words: 1200
  max_words: 3500
  required_sections:
    - "# Bencivenga Methods"
    - "## The Persuasion Equation"
    - "## Fascinations"
    - "## Headlines"
    - "## Proof hierarchy"
    - "## Emotional triggers matrix"
    - "## Bencivenga voice rules"
```

- [ ] **Step 2: Run linter — expect FAIL**

- [ ] **Step 3: Author `references/bencivenga-methods.md`**

Merge + dedupe content from both `bencivenga_copywriting_methods_report.md` and `bencivenga_copywriting_methods_analysis.md` in `context/semantic-copywriting-project/`. Structure:
- **The Persuasion Equation** — each of the 4 elements (urgent problem / unique promise / unquestionable proof / user-friendly proposition) expanded with execution guidance. Medical/legal examples.
- **Fascinations** — 7 types (how-to / secret / never / why / number+benefit / specific outcome / warning) with formula + 2 examples per type.
- **Headlines** — 4 U's (urgent, unique, useful, ultra-specific). 25-headline rule. Proof-enhancement patterns (authority, guarantee, timeframe, specificity). 5 headline structure templates.
- **Proof hierarchy** — 7 layers in order of credibility.
- **Emotional triggers matrix** — 7 triggers (fear, greed, curiosity, hope, security, empowerment, pride) with when-to-lead and how-to-activate guidance.
- **Bencivenga voice rules** — admit a minor flaw, use long copy when proof is available, descriptive anchor text, etc.

- [ ] **Step 4: Run linter — expect PASS**

- [ ] **Step 5: Commit**

```bash
git add references/bencivenga-methods.md tests/references/properties.yaml
git commit -m "feat: add bencivenga-methods reference (merged + deduped from source files)"
```

---

## Phase 4 — Example files

### Task 16: Rename source promos and add teaching headers

**Files:**
- Rename + modify (10 files):
  - `context/semantic-copywriting-project/01_Little_Black_Book.md` → copied to `references/example-complete-package.md`
  - `02_Newsletter.md` → `references/example-subscription-continuity.md`
  - `03_Get_Rich_Slowly.md` → `references/example-financial-headlines.md`
  - `04_Lies_Lies_Lies.md` → `references/example-expose-structure.md`
  - `05_Charles_Givens.md` → `references/example-expert-authority.md`
  - `06_Merrill_Lynch.md` → `references/example-institutional-trust.md`
  - `07_Look_Younger_Now_MAGALOG.md` → `references/example-health-magalog.md`
  - `08_Hands_Off_Washington.md` → `references/example-outrage-urgency.md`
  - `Gary_Bencivenga___Famous__Olive_Oil__Sales_Letter_Breakdown__11_100_.md` → `references/example-olive-oil-breakdown.md`
  - `complete_interview.md` → `references/interview-bencivenga.md`
- Modify: `tests/references/properties.yaml`

Source files stay in `context/` (archival provenance); the copies in `references/` get teaching headers.

- [ ] **Step 1: Copy source files to references/ with new names**

```bash
cp "context/semantic-copywriting-project/01_Little_Black_Book.md" references/example-complete-package.md
cp "context/semantic-copywriting-project/02_Newsletter.md" references/example-subscription-continuity.md
cp "context/semantic-copywriting-project/03_Get_Rich_Slowly.md" references/example-financial-headlines.md
cp "context/semantic-copywriting-project/04_Lies_Lies_Lies.md" references/example-expose-structure.md
cp "context/semantic-copywriting-project/05_Charles_Givens.md" references/example-expert-authority.md
cp "context/semantic-copywriting-project/06_Merrill_Lynch.md" references/example-institutional-trust.md
cp "context/semantic-copywriting-project/07_Look_Younger_Now_MAGALOG.md" references/example-health-magalog.md
cp "context/semantic-copywriting-project/08_Hands_Off_Washington.md" references/example-outrage-urgency.md
cp "context/semantic-copywriting-project/Gary_Bencivenga___Famous__Olive_Oil__Sales_Letter_Breakdown__11_100_.md" references/example-olive-oil-breakdown.md
cp "context/semantic-copywriting-project/complete_interview.md" references/interview-bencivenga.md
```

- [ ] **Step 2: Prepend a teaching header to each of the 10 files**

For each file, insert this header block above the existing content. Fill in the bracketed slots per-file based on what the promo actually demonstrates (see slot notes below the template).

```markdown
---
type: example
source_file: <original filename from context/>
---

# [Descriptive Title — e.g., "Health Magalog: Look Younger Now"]

> **Teaching header (~100 words)**
>
> **What this exemplifies:** [3–5 techniques shown — e.g., "Fascination-heavy magalog; benefit-specificity bullet formulas; aspirational before/after framing; 88-page long-form anatomy; proof-laden subheads."]
>
> **When to load this:** [Task triggers — e.g., "Anti-aging, aesthetics, skincare, dermatology, or any health promise requiring rich curiosity bullets."]
>
> **Key moments:** [Section pointers into the text below — e.g., "Opening spread (fear + hope transition). Mid-piece 'secret list' fascinations (rich pattern library). Final offer page (risk reversal + urgency stack)."]

---

<existing promo content follows unchanged>
```

Per-file slot content (reference these when authoring headers):

- **example-complete-package.md** (`01_Little_Black_Book`): complete direct-mail package (OE + letter + lift note + BRC); package anatomy; teaser-to-letter flow.
- **example-subscription-continuity.md** (`02_Newsletter`): continuity product, value-stacking, subscriber psychology; premium-framing on a recurring offer.
- **example-financial-headlines.md** (`03_Get_Rich_Slowly`): financial headline mechanics; proof-enhanced headline patterns; retirement/wealth audience.
- **example-expose-structure.md** (`04_Lies_Lies_Lies`): expose/reveal long-form arc; credibility-building by calling out industry falsehoods; proof ladder structure.
- **example-expert-authority.md** (`05_Charles_Givens`): expert authority positioning; seminar-style proof stacking; spokesperson-led offer.
- **example-institutional-trust.md** (`06_Merrill_Lynch`): institutional trust-first; credential-heavy; soft CTA.
- **example-health-magalog.md** (`07_Look_Younger_Now_MAGALOG`): fascination mastery; benefit-specificity formulas; long-form magalog anatomy.
- **example-outrage-urgency.md** (`08_Hands_Off_Washington`): outrage/patriot emotion triggers; urgency framing; advocacy-pitch structure.
- **example-olive-oil-breakdown.md**: line-by-line annotated persuasion analysis of a famous Bencivenga letter.
- **interview-bencivenga.md** (`complete_interview`): Bencivenga's direct voice on his methodology — searchable for specific techniques.

- [ ] **Step 3: Add properties.yaml entries for all 10 example files**

```yaml
example-complete-package.md:
  required_sections: ["## Teaching header"]
  required_header_keys: ["What this exemplifies", "When to load this", "Key moments"]
example-subscription-continuity.md:
  required_sections: ["## Teaching header"]
  required_header_keys: ["What this exemplifies", "When to load this", "Key moments"]
# ... same pattern for the remaining 8 files
```

Note: the properties linter's `required_header_keys` pattern is `**key:**`. Teaching headers use `> **What this exemplifies:**` (inside a blockquote). Update the linter regex to be blockquote-aware, or change the teaching-header format to not use blockquotes. Prefer the latter — rewrite the template without the `>` prefix so the linter works uniformly. Update Step 2's template accordingly.

- [ ] **Step 4: Run linter against all example files — expect PASS**

Run: `pytest tests/references/test_reference_properties.py -v`
Expected: all tests PASS (or 20 pass: 10 exists + 10 properties).

- [ ] **Step 5: Commit**

```bash
git add references/example-*.md references/interview-bencivenga.md tests/references/properties.yaml
git commit -m "feat: add 10 Bencivenga example references with teaching headers"
```

---

## Phase 5 — Full SKILL.md body

### Task 17: Complete SKILL.md body with routing, override, decision matrix

**Files:**
- Modify: `SKILL.md`
- Modify: `tests/skill/test_frontmatter.py` (add body-structure tests)

- [ ] **Step 1: Add structural tests for the full body**

Append to `tests/skill/test_frontmatter.py`:

```python
def test_skill_md_under_500_lines():
    skill_md = Path(__file__).resolve().parents[2] / "SKILL.md"
    lines = skill_md.read_text(encoding="utf-8").splitlines()
    assert len(lines) <= 500, f"SKILL.md is {len(lines)} lines, must be ≤500"


def test_skill_md_has_required_body_sections():
    skill_md = Path(__file__).resolve().parents[2] / "SKILL.md"
    content = skill_md.read_text(encoding="utf-8")
    required = [
        "## User-override protocol",
        "## Step 1",
        "## Step 2",
        "## Step 3",
        "## Step 4",
        "## Step 5",
        "## Step 6",
        "## The Persuasion Equation",
        "## Writing Rules",
        "## Progressive disclosure",
        "## Output format",
    ]
    for section in required:
        assert section in content, f"SKILL.md missing section '{section}'"


def test_skill_md_has_intent_classification_not_keyword_match():
    skill_md = Path(__file__).resolve().parents[2] / "SKILL.md"
    content = skill_md.read_text(encoding="utf-8")
    # Intent classification language per spec §6 Step 2
    assert "Classify" in content or "classify" in content
    # Explicit negative: keyword-based routing should be flagged as NOT reliable
    assert "URL in the input is NOT a reliable signal" in content or "intent" in content.lower()
```

- [ ] **Step 2: Run tests — expect FAIL (SKILL.md still has placeholder)**

Run: `pytest tests/skill/ -v`
Expected: frontmatter tests PASS, body-structure tests FAIL.

- [ ] **Step 3: Replace SKILL.md body with the full version**

Replace the placeholder body with the shape defined in spec §6. Keep under 500 lines. Required top-level elements in this order:
1. Frontmatter (unchanged from Task 3).
2. `# Semantic Persuasive Copywriter` title + 2-sentence framing.
3. `## User-override protocol` — standing instructions, 3 overrides (skip grading / use my rubric / skip intake), per-request scope.
4. `## Step 1 — Load baseline` — always load `references/sro-principles.md`. Also always load `references/client-index.md`.
5. `## Step 2 — Classify intent, then route to a mode` — the 3-question classifier from spec, ambiguity fallback, load `references/mode-*.md`.
6. `## Step 3 — Check client index, then vertical compliance` — client lookup via `client-index.md`; if client matched, auto-load `compliance-medical-legal.md`; if no client matched, entity-class check.
7. `## Step 4 — Brand context` — Code (run `scripts/fetch_brand_guide.py`), chat/cowork (MCP or paste), graceful degradation.
8. `## Step 5 — Execute the mode` — follow loaded mode file.
9. `## Step 6 — Grade before delivery` — load `grading-rubric.md`, adversarial enumeration, iteration cap (revise once, then gap notes).
10. `## The Persuasion Equation` — 1-line mnemonic.
11. `## Writing Rules` — condensed DO/DON'T lists.
12. `## Progressive disclosure — when to load example files` — the decision-matrix table mapping tasks to `example-*.md` / `bencivenga-methods.md` / `interview-bencivenga.md`.
13. `## Output format (v1)` — H1, H2/H3, body, meta title, meta description, image alt, grade report.

Target: 150–300 lines. Stay under 500.

- [ ] **Step 4: Run tests — expect PASS**

Run: `pytest tests/skill/ -v`
Expected: all tests PASS.

- [ ] **Step 5: Commit**

```bash
git add SKILL.md tests/skill/test_frontmatter.py
git commit -m "feat: complete SKILL.md body with intent-classification routing and user overrides"
```

---

## Phase 6 — Fixtures + harness

### Task 18: Write 5 fixture inputs and assertable-properties spec

**Files:**
- Create: `tests/fixtures/rewrite-medical-thin-source.md`
- Create: `tests/fixtures/rewrite-legal-rich-source.md`
- Create: `tests/fixtures/blog-medical-brief.md`
- Create: `tests/fixtures/blog-legal-topic-only.md`
- Create: `tests/fixtures/newsletter-plastic-surgery.md`
- Create: `tests/fixtures/properties.yaml`

Each fixture is a complete "user input" as it would arrive in chat. The properties YAML lists what must be true of the skill's output for each input.

- [ ] **Step 1: Write `tests/fixtures/rewrite-medical-thin-source.md`**

```markdown
---
mode: rewrite
vertical: medical
client_slug: berks-plastic-surgery
input_type: url
---

# User input

Rewrite this page for Berks Plastic Surgery's tummy tuck service:
https://berksplasticsurgery.com/tummy-tuck/ (assume body content below)

---

# Source body (simulated — thin/weak page)

## Tummy Tuck Surgery

A tummy tuck can help you look and feel better. Dr. Jenkins has performed many tummy tuck surgeries and can help you achieve your goals. Contact us today to schedule a consultation.

### What is a tummy tuck?

A tummy tuck is a surgery that removes excess skin and fat from the abdomen.

### Why choose us?

We are experienced and our patients are happy.
```

- [ ] **Step 2: Write `tests/fixtures/rewrite-legal-rich-source.md`**

```markdown
---
mode: rewrite
vertical: legal
client_slug: mw-brady-law-firm
input_type: url
---

# User input

Rewrite the estate-planning service page for M.W. Brady Law Firm: https://mwbradylaw.com/estate-planning/

---

# Source body (simulated — richer existing page)

## Estate Planning

At M.W. Brady Law Firm, P.C., we have guided over 1,200 Pennsylvania families through estate planning since 1998. M. Winston Brady, Esq. — a member of the Pennsylvania Bar since 1992 — leads our estate-planning practice.

### Our services

- Wills and trusts
- Powers of attorney
- Advance healthcare directives
- Estate administration

### Testimonials

> "Mr. Brady made a difficult process feel manageable. Our family plan is finally in place." — Sandra K., Reading, PA

### Next step

Call (610) 555-0188 to schedule a consultation.
```

- [ ] **Step 3: Write `tests/fixtures/blog-medical-brief.md`**

```markdown
---
mode: blog
vertical: medical
client_slug: aesthetic-nirvana
input_type: brief
---

# User input

Write a blog post for Aesthetic Nirvana about recovery expectations after CoolSculpting. Target audience: women 35–55 considering non-surgical body contouring. Keyword target: "CoolSculpting recovery what to expect." CTA: book a consultation. Length: ~800 words.
```

- [ ] **Step 4: Write `tests/fixtures/blog-legal-topic-only.md`**

```markdown
---
mode: blog
vertical: legal
client_slug: craig-associates
input_type: topic_only
---

# User input

Write a blog post for Craig Associates about medical malpractice statutes of limitation. No other details — figure out audience, angle, and CTA from the firm's brand guide.
```

- [ ] **Step 5: Write `tests/fixtures/newsletter-plastic-surgery.md`**

```markdown
---
mode: newsletter
vertical: medical
client_slug: movassaghi-plastic-surgery
input_type: newsletter
---

# User input

Write the May newsletter for Movassaghi Plastic Surgery. Audience: past patients and prospective leads on the email list. Theme: summer-ready body + face. Suggest 4 items. Highlight the Memorial Day promo on CoolSculpting (20% off through May 31).
```

- [ ] **Step 6: Write `tests/fixtures/properties.yaml`**

```yaml
rewrite-medical-thin-source.md:
  expected_mode: rewrite
  compliance_loaded: true
  brand_guide_loaded: true
  grade_total_min: 80
  output_must_contain:
    - "Berks Plastic Surgery"
    - "Dr. Jenkins"
  output_structure:
    h1: required
    h2_count_min: 2
    meta_title_max_chars: 60
    meta_description_max_chars: 160
    image_alt_count_min: 1
    grade_report: required
  output_must_not_contain:
    - "guaranteed results"
    - "miracle"
    - "100% safe"

rewrite-legal-rich-source.md:
  expected_mode: rewrite
  compliance_loaded: true
  brand_guide_loaded: true
  grade_total_min: 85  # rich source should grade better than thin
  output_must_contain:
    - "M.W. Brady"
    - "Pennsylvania"
  output_must_not_contain:
    - "guaranteed outcome"
    - "win your case"

blog-medical-brief.md:
  expected_mode: blog-post
  compliance_loaded: true
  brand_guide_loaded: true
  grade_total_min: 80
  output_structure:
    h1: required
    h2_count_min: 3
    suggested_slug: required
    grade_report: required

blog-legal-topic-only.md:
  expected_mode: blog-post
  compliance_loaded: true
  brand_guide_loaded: true
  grade_total_min: 75  # topic-only is thin input; allow lower floor
  intake_questions_asked_max: 3
  output_must_not_contain:
    - "we will win"
    - "guaranteed"

newsletter-plastic-surgery.md:
  expected_mode: newsletter
  compliance_loaded: true
  brand_guide_loaded: true
  grade_total_min: 80
  output_structure:
    subject_line: required
    preview_text: required
    item_count_min: 3
    item_count_max: 5
    grade_report: required
  output_must_contain:
    - "Memorial Day"
    - "CoolSculpting"
```

- [ ] **Step 7: Commit**

```bash
git add tests/fixtures/
git commit -m "test: add 5 fixture inputs with assertable properties"
```

---

### Task 19: Build `run_fixtures.py` harness

**Files:**
- Create: `tests/skill/run_fixtures.py`
- Create: `tests/skill/test_run_fixtures.py`

The harness invokes the Claude API with the skill loaded, runs each fixture as a user message, parses the response, and asserts properties. Requires `ANTHROPIC_API_KEY`.

- [ ] **Step 1: Write failing tests for the response parser (unit-testable without API)**

```python
# tests/skill/test_run_fixtures.py
from tests.skill.run_fixtures import (
    parse_grade_report,
    parse_structural_elements,
    check_property,
)


SAMPLE_OUTPUT = """
# How Dr. Jenkins Helps Berks Patients Flatten the Stomach — Without Regret

**Meta title:** Tummy Tuck in Berks County | Dr. Jenkins, Berks Plastic Surgery
**Meta description:** Dr. Jenkins has performed 800+ abdominoplasties in Berks County since 2011. Schedule a consultation today.

## Who this page is for

[body paragraphs...]

## What Dr. Jenkins does differently

[body paragraphs...]

![Before and after — patient after abdominoplasty](alt: "Berks Plastic Surgery tummy tuck before and after — 6 months post-op")

---

## Grade report

- Bencivenga: 7/7
- SRO: 6/7
- E-E-A-T: 4/4
- **Total: 94% (Grade: A)**

### Notes
- SRO item 10 (query-aligned headings): one H2 is a soft label ("What we offer"); consider rewriting as a direct-answer phrase.
"""


def test_parse_grade_report_extracts_total_and_grade():
    result = parse_grade_report(SAMPLE_OUTPUT)
    assert result["total_percent"] == 94
    assert result["grade"] == "A"
    assert result["bencivenga"] == "7/7"
    assert result["sro"] == "6/7"
    assert result["eeat"] == "4/4"


def test_parse_structural_elements_counts_h1_h2_meta():
    result = parse_structural_elements(SAMPLE_OUTPUT)
    assert result["h1"] == 1
    assert result["h2_count"] >= 2
    assert result["meta_title_length"] <= 60
    assert result["meta_description_length"] <= 160
    assert result["image_alt_count"] >= 1


def test_check_property_grade_total_min_pass():
    elements = {"grade_total": 94}
    ok, msg = check_property(elements, "grade_total_min", 80)
    assert ok, msg


def test_check_property_grade_total_min_fail():
    elements = {"grade_total": 70}
    ok, msg = check_property(elements, "grade_total_min", 80)
    assert not ok
    assert "70" in msg and "80" in msg


def test_check_property_output_must_contain_pass():
    elements = {"raw_output": SAMPLE_OUTPUT}
    ok, msg = check_property(elements, "output_must_contain", ["Dr. Jenkins"])
    assert ok


def test_check_property_output_must_not_contain_fail():
    elements = {"raw_output": "We guarantee results."}
    ok, msg = check_property(elements, "output_must_not_contain", ["guarantee"])
    assert not ok
    assert "guarantee" in msg
```

- [ ] **Step 2: Run tests — expect FAIL (imports don't exist)**

Run: `pytest tests/skill/test_run_fixtures.py -v`
Expected: FAIL with `ModuleNotFoundError`.

- [ ] **Step 3: Write `tests/skill/run_fixtures.py`**

```python
"""Run skill fixtures against the Claude API and assert properties."""

from __future__ import annotations

import os
import re
import sys
from pathlib import Path
from typing import Any

import yaml

REPO_ROOT = Path(__file__).resolve().parents[2]
FIXTURES_DIR = REPO_ROOT / "tests" / "fixtures"
PROPERTIES_FILE = FIXTURES_DIR / "properties.yaml"
SKILL_MD = REPO_ROOT / "SKILL.md"


def parse_grade_report(output: str) -> dict[str, Any]:
    """Extract grade report fields from skill output."""
    result: dict[str, Any] = {}
    for line in output.splitlines():
        m = re.match(r"^- Bencivenga:\s*(\S+)", line)
        if m:
            result["bencivenga"] = m.group(1)
        m = re.match(r"^- SRO:\s*(\S+)", line)
        if m:
            result["sro"] = m.group(1)
        m = re.match(r"^- E-E-A-T:\s*(\S+)", line)
        if m:
            result["eeat"] = m.group(1)
        m = re.match(r"^- \*\*Total:\s*(\d+)%.*Grade:\s*([A-F])", line)
        if m:
            result["total_percent"] = int(m.group(1))
            result["grade"] = m.group(2)
    return result


def parse_structural_elements(output: str) -> dict[str, Any]:
    """Count H1/H2, meta title/description length, image alt count."""
    h1_count = sum(1 for line in output.splitlines() if re.match(r"^#\s+\S", line))
    h2_count = sum(1 for line in output.splitlines() if re.match(r"^##\s+\S", line))

    meta_title_match = re.search(r"\*\*Meta title:\*\*\s*(.+)", output)
    meta_desc_match = re.search(r"\*\*Meta description:\*\*\s*(.+)", output)
    meta_title_len = len(meta_title_match.group(1).strip()) if meta_title_match else 0
    meta_desc_len = len(meta_desc_match.group(1).strip()) if meta_desc_match else 0

    image_alt_count = len(re.findall(r"!\[[^\]]*\]", output))

    subject_line = bool(re.search(r"\*\*Subject(?: line)?:\*\*", output, re.IGNORECASE))
    preview_text = bool(re.search(r"\*\*Preview(?: text)?:\*\*", output, re.IGNORECASE))
    suggested_slug = bool(re.search(r"\*\*(?:Suggested |URL )?Slug:\*\*", output, re.IGNORECASE))
    grade_report_present = "Grade report" in output or "Grade:" in output

    return {
        "h1": h1_count,
        "h2_count": h2_count,
        "meta_title_length": meta_title_len,
        "meta_description_length": meta_desc_len,
        "image_alt_count": image_alt_count,
        "subject_line": subject_line,
        "preview_text": preview_text,
        "suggested_slug": suggested_slug,
        "grade_report": grade_report_present,
    }


def check_property(elements: dict[str, Any], key: str, expected: Any) -> tuple[bool, str]:
    """Apply a single property check. Returns (ok, message)."""
    raw = elements.get("raw_output", "")

    if key == "grade_total_min":
        actual = elements.get("grade_total", 0)
        return actual >= expected, f"grade_total {actual} < min {expected}"
    if key == "output_must_contain":
        missing = [s for s in expected if s not in raw]
        return not missing, f"missing required strings: {missing}"
    if key == "output_must_not_contain":
        present = [s for s in expected if s in raw]
        return not present, f"found forbidden strings: {present}"
    if key == "output_structure":
        failures = []
        for subkey, requirement in expected.items():
            if requirement == "required" and not elements.get(subkey):
                failures.append(f"{subkey} missing")
            if isinstance(requirement, int):
                if subkey.endswith("_max_chars"):
                    base = subkey.replace("_max_chars", "_length")
                    actual = elements.get(base, 0)
                    if actual > requirement:
                        failures.append(f"{subkey}: {actual} > {requirement}")
                elif subkey.endswith("_min"):
                    base = subkey.replace("_min", "")
                    actual = elements.get(base, 0)
                    if actual < requirement:
                        failures.append(f"{subkey}: {actual} < {requirement}")
                elif subkey.endswith("_max"):
                    base = subkey.replace("_max", "")
                    actual = elements.get(base, 0)
                    if actual > requirement:
                        failures.append(f"{subkey}: {actual} > {requirement}")
        return not failures, "; ".join(failures)
    return True, f"unknown property '{key}' ignored"


def invoke_skill(fixture_body: str) -> str:
    """Send the fixture to the Claude API with the skill content as system prompt."""
    try:
        import anthropic  # noqa: F401
    except ImportError:
        print("ERROR: `anthropic` package not installed. Run: pip install anthropic", file=sys.stderr)
        sys.exit(1)
    from anthropic import Anthropic

    api_key = os.environ.get("ANTHROPIC_API_KEY")
    if not api_key:
        print("ERROR: ANTHROPIC_API_KEY not set", file=sys.stderr)
        sys.exit(1)

    skill_content = SKILL_MD.read_text(encoding="utf-8")
    system_prompt = (
        "You are running the semantic-persuasive-copywriter skill. "
        "The skill content follows. Execute it exactly as written.\n\n"
        + skill_content
    )

    client = Anthropic(api_key=api_key)
    response = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=8000,
        system=system_prompt,
        messages=[{"role": "user", "content": fixture_body}],
    )
    return response.content[0].text


def run_all_fixtures() -> int:
    properties = yaml.safe_load(PROPERTIES_FILE.read_text(encoding="utf-8")) or {}
    failures: list[str] = []

    for fixture_name, props in properties.items():
        fixture_path = FIXTURES_DIR / fixture_name
        if not fixture_path.exists():
            failures.append(f"{fixture_name}: fixture file not found")
            continue
        print(f"\n--- {fixture_name} ---")
        fixture_content = fixture_path.read_text(encoding="utf-8")
        # Strip the fixture's own frontmatter (it's metadata about expected behavior)
        body = re.sub(r"^---\n.*?\n---\n", "", fixture_content, count=1, flags=re.DOTALL)
        output = invoke_skill(body)

        elements = parse_structural_elements(output)
        grade = parse_grade_report(output)
        elements["grade_total"] = grade.get("total_percent", 0)
        elements["grade_letter"] = grade.get("grade", "?")
        elements["raw_output"] = output

        fixture_failures: list[str] = []
        for key, expected in props.items():
            if key.startswith("expected_") or key.endswith("_loaded") or key == "intake_questions_asked_max":
                # Behavioral properties — not yet machine-checkable from output alone;
                # flag in a TODO log, don't fail the fixture on them in v1.
                continue
            ok, msg = check_property(elements, key, expected)
            if not ok:
                fixture_failures.append(f"  {key}: {msg}")

        if fixture_failures:
            failures.append(f"{fixture_name}\n" + "\n".join(fixture_failures))
            print(f"FAIL: {fixture_name}")
            for f in fixture_failures:
                print(f)
        else:
            print(f"PASS: {fixture_name} (grade {elements['grade_total']}% / {elements['grade_letter']})")

    if failures:
        print(f"\n{len(failures)} fixture(s) failed.")
        return 1
    print(f"\nAll {len(properties)} fixtures passed.")
    return 0


if __name__ == "__main__":
    sys.exit(run_all_fixtures())
```

- [ ] **Step 4: Run unit tests — expect PASS**

Run: `pytest tests/skill/test_run_fixtures.py -v`
Expected: 6 tests PASS.

- [ ] **Step 5: Run the harness end-to-end against the real skill (requires `ANTHROPIC_API_KEY`)**

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export GITHUB_TOKEN=<fine-grained PAT>
python tests/skill/run_fixtures.py
```

Expected: 5 fixtures processed. Each either PASS with grade ≥ its minimum, or a FAIL with specific property names and actual values. Iterate on SKILL.md and reference files until all 5 pass.

- [ ] **Step 6: Commit**

```bash
git add tests/skill/run_fixtures.py tests/skill/test_run_fixtures.py
git commit -m "test: add fixture harness with assertable properties"
```

---

## Phase 7 — Ship gate (non-code)

### Task 20: Pre-ship coordination and documentation

**Files:**
- Modify: `references/compliance-medical-legal.md` (reviewer + date lines)
- Modify: `docs/superpowers/specs/2026-04-20-semantic-persuasive-copywriter-design.md` (§13 resolutions)
- Create: `docs/ship-readiness.md`

This task has no code — it's the non-automated gates from the v1 ship-gate list in spec §10. Each item below must be signed off before Teams distribution.

- [ ] **Step 1: Attorney review of `compliance-medical-legal.md`**

Send `references/compliance-medical-legal.md` to an attorney with healthcare/legal-advertising experience. Incorporate feedback. Update the header:

```markdown
**Reviewed by:** [Attorney Name, Bar Admission, Date]
**Review date:** YYYY-MM-DD
```

Commit:

```bash
git add references/compliance-medical-legal.md
git commit -m "docs(compliance): incorporate attorney review; lock baseline for v1"
```

- [ ] **Step 2: Produce three real-client deliverables across modes + environments**

Pick one each:
- Rewrite (e.g., a Berks Plastic Surgery service page) — run in Claude Code.
- Blog post (e.g., for Aesthetic Nirvana) — run in Claude.ai chat (Teams).
- Newsletter (e.g., Movassaghi monthly) — run in Claude.ai cowork or chat.

For each, document in `docs/ship-readiness.md`:
- Input used
- Environment
- Output summary (or link to the deliverable)
- Grade result (total %, grade letter)
- Gap notes (if any)
- Clark's sign-off (date)

Pass criteria per spec §10 ship gate: grade ≥ 85%, compliance rules applied, brand voice matched, no unresolved gap notes.

- [ ] **Step 3: Resolve spec §13 open questions**

Update `docs/superpowers/specs/2026-04-20-semantic-persuasive-copywriter-design.md` §13 with resolutions:

1. **Source location promotion path** — decide: stay in workbench? Promote to dedicated GitHub repo? Upload to Teams admin? Document the chosen path and who executes it.
2. **GITHUB_TOKEN for chat/cowork** — confirm: not applicable (GitHub MCP or paste handles these). Close this item.
3. **Newsletter subject-line rules** — confirm with team any CAKE-specific length/style constraints; update `mode-newsletter.md` if needed.
4. **Bencivenga copyright for Teams distribution** — make one of the choices: (a) reduce to fair-use quotations, (b) obtain license, (c) Code-only distribution, (d) replace raws with pattern distillations. Document the decision with rationale.
5. **Observability for v1.1** — decide: local JSONL log, copy-paste telemetry block, or nothing. Document with a deferred-to date.

Convert each open question to a "Resolved" entry with the decision and date.

- [ ] **Step 4: Write `docs/ship-readiness.md`**

```markdown
# v1 Ship Readiness

**Target ship date:** <date>

## Ship-gate checklist

- [ ] All 8 new reference files written and committed
- [ ] Attorney-reviewed compliance file (reviewer + date documented)
- [ ] 10 example files renamed with teaching headers
- [ ] SKILL.md ≤ 500 lines, passes frontmatter + body tests
- [ ] 3 scripts pass unit tests (including SSRF + token-scope guards)
- [ ] All 5 fixtures pass every assertable property via run_fixtures.py
- [ ] Graceful degradation verified on each documented failure mode
- [ ] 3 real-client deliverables produced across modes + environments
- [ ] Copyright decision made (§13 resolution)
- [ ] Promotion path decided (§13 resolution)

## Deliverable evidence

### Rewrite deliverable
- **Client:** ...
- **Environment:** ...
- **Grade:** ... / ...
- **Gap notes:** ...
- **Signed off:** Clark, YYYY-MM-DD

### Blog deliverable
- ...

### Newsletter deliverable
- ...

## Distribution plan

<ship plan per §13 resolution 1>
```

- [ ] **Step 5: Commit**

```bash
git add docs/ship-readiness.md docs/superpowers/specs/2026-04-20-semantic-persuasive-copywriter-design.md
git commit -m "docs: finalize ship-readiness checklist and spec open questions"
```

---

## Appendix A — Dependencies & commands

| Need | Install | Command |
|------|---------|---------|
| Python deps | `pip install -r scripts/requirements.txt` | — |
| Anthropic SDK (for harness) | `pip install anthropic` | — |
| Run all tests | — | `pytest tests/ -v` |
| Run the fixture harness | — | `python tests/skill/run_fixtures.py` |
| Regenerate client index | — | `python scripts/sync_client_index.py` |
| Fetch a brand guide | — | `python scripts/fetch_brand_guide.py <slug>` |
| Fetch a URL for rewrite | — | `python scripts/fetch_url.py <url>` |

## Appendix B — Failure-mode matrix (from spec)

| Script | Exit | Meaning |
|--------|------|---------|
| `fetch_url.py` | 1 | Usage / scheme / fetch error |
| `fetch_url.py` | 2 | SSRF guard — private/reserved IP |
| `fetch_brand_guide.py` | 1 | Usage / missing token / slug not found / network |
| `fetch_brand_guide.py` | 3 | Token has broader scope than required |
| `sync_client_index.py` | 1 | Missing token / network / parse error |

## Appendix C — Self-review notes

This plan was self-reviewed against the spec; every requirement in spec §§6–13 maps to a task in Phases 1–7. The ship-gate items in spec §10 map to Task 20. Scripts with guards in spec §8 map to Tasks 5, 7, 8. Reference-file scope in spec §7 maps to Tasks 9–16. SKILL.md body shape in spec §6 maps to Task 17.
