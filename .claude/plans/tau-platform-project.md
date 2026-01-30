# Plan: TAU Platform Project Infrastructure

## Goal

Establish the GitHub project management infrastructure for the TAU Platform ecosystem:
1. Create a GitHub Project board with phase tracking
2. Create two well-structured tau-core issues (Audio protocol, Whisper provider)
3. Create a `tau-platform` coordination repository with Discussions enabled

---

## Step 1: Create `tau-platform` Repository

Create a new public repository at `tailored-agentic-units/tau-platform` to serve as:
- **Organization Discussions source** — GitHub org-level discussions require a backing repo
- **Coordination artifacts** — Cross-repo planning docs, ADRs, ecosystem vision
- **Project home** — README describing the TAU Platform ecosystem and linking to all repos

```bash
gh repo create tailored-agentic-units/tau-platform \
  --public \
  --description "TAU Platform coordination, planning, and cross-repo discussions" \
  --clone=false
```

Clone locally:

```bash
gh repo clone tailored-agentic-units/tau-platform ~/tau/tau-platform
```

Then enable Discussions on the repo and configure it as the org's discussion source.

---

## Step 2: Create GitHub Project "TAU Platform"

```bash
PROJECT_NUM=$(gh project create --owner tailored-agentic-units \
  --title "TAU Platform" --format json --jq '.number')
```

### Link repositories

```bash
gh project link "$PROJECT_NUM" --owner tailored-agentic-units \
  --repo tailored-agentic-units/tau-core

gh project link "$PROJECT_NUM" --owner tailored-agentic-units \
  --repo tailored-agentic-units/tau-platform
```

### Create Phase field

```bash
gh project field-create "$PROJECT_NUM" --owner tailored-agentic-units \
  --name "Phase" \
  --data-type SINGLE_SELECT \
  --single-select-options "Backlog,Phase 1 - Foundation"
```

---

## Step 3: Create tau-core Issues

### Issue 1: Audio Protocol Support

**Title:** Add Audio protocol for speech-to-text transcription

**Body:** Describes the new Audio protocol following the existing protocol extension pattern from the tau-core-admin skill:
- Add `Audio` constant to `pkg/protocol/protocol.go`
- Create `pkg/request/audio.go` request type
- Create `pkg/response/audio.go` response type with `ParseAudio()`
- Add `AudioData` struct to `pkg/providers/data.go`
- Update `BaseProvider.Marshal()` with audio case
- Add `Audio()` method to agent API
- Black-box tests in `tests/`

**Labels:** `enhancement`, `protocol`

### Issue 2: Whisper Provider

**Title:** Add Whisper provider for audio transcription

**Body:** Describes the new Whisper provider following the existing provider extension pattern:
- Create `pkg/providers/whisper.go` embedding `BaseProvider`
- Implement `Provider` interface (endpoint routing to `/v1/audio/transcriptions`)
- Multipart form data handling for audio file upload
- Bearer token authentication
- Register via `Register("whisper", NewWhisper)` in `pkg/providers/registry.go`
- Black-box tests in `tests/providers/whisper_test.go`

**Labels:** `enhancement`, `provider`

**Blocked by:** Issue 1 (Audio protocol must exist first)

---

## Step 4: Add Issues to Project & Assign Phase

```bash
# Add both issues to the project
gh project item-add "$PROJECT_NUM" --owner tailored-agentic-units \
  --url <issue-1-url>

gh project item-add "$PROJECT_NUM" --owner tailored-agentic-units \
  --url <issue-2-url>

# Resolve IDs and assign both to "Phase 1 - Foundation"
PROJECT_ID=$(gh project view "$PROJECT_NUM" --owner tailored-agentic-units --format json --jq '.id')
FIELDS_JSON=$(gh project field-list "$PROJECT_NUM" --owner tailored-agentic-units --format json)
FIELD_ID=$(echo "$FIELDS_JSON" | jq -r '.fields[] | select(.name == "Phase") | .id')
OPTION_ID=$(echo "$FIELDS_JSON" | jq -r '.fields[] | select(.name == "Phase") | .options[] | select(.name == "Phase 1 - Foundation") | .id')

# Set phase on each item
gh project item-edit --id <item-id> --project-id "$PROJECT_ID" \
  --field-id "$FIELD_ID" --single-select-option-id "$OPTION_ID"
```

---

## Step 5: Enable Organization Discussions

Configure the `tau-platform` repo as the org's discussion source:

```bash
# Enable discussions on the tau-platform repo
gh repo edit tailored-agentic-units/tau-platform --enable-discussions

# Set as org discussion source (via GitHub web UI or API)
gh api --method PUT /orgs/tailored-agentic-units \
  -f discussion_source_repository_id=<repo-id>
```

> Note: Org-level discussion source configuration may require the web UI.
> Navigate to: Organization Settings → Discussions → Enable and select tau-platform.

---

## Verification

- [ ] `tau-platform` repo exists at `github.com/tailored-agentic-units/tau-platform`
- [ ] Discussions enabled on `tau-platform`
- [ ] "TAU Platform" project visible at org project board
- [ ] Phase field has "Backlog" and "Phase 1 - Foundation" options
- [ ] tau-core and tau-platform linked to project
- [ ] Issue #3: "Add Audio protocol for speech-to-text transcription" exists on tau-core
- [ ] Issue #4: "Add Whisper provider for audio transcription" exists on tau-core
- [ ] Both issues assigned to "Phase 1 - Foundation" in the project
