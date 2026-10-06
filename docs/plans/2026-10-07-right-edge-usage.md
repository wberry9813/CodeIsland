# CodeIsland right-edge surface + Codex / Antigravity usage plan

Date: 2026-10-07

## Product decision

Use CodeIsland as the primary app. Keep its existing session, permission, question,
completion, notification, terminal-jump, hook, and OpenCode integration behavior.

Do not continue reimplementing those capabilities in Codenotch.

Codenotch remains a reference/source for usage-reading code, especially Codex and
Antigravity. Do not port its session architecture or provider UI wholesale.

## Target UX

Add a presentation setting:

- Notch (existing behavior, unchanged)
- Right edge

Right-edge mode should feel native to CodeIsland rather than like a second app:

- collapsed state lives against the right screen edge
- active/waiting sessions remain immediately visible
- hover/click expands inward (to the left), never off-screen
- approval, question, completion and session-list surfaces reuse the existing
  IslandSurface/AppState state machine
- existing click-to-jump, sounds, smart suppress, notifications and follow-up
  behavior continue to work
- multi-display selection continues to use the existing chosen-screen logic

The first implementation should not rotate the existing notch view 90 degrees.
Introduce a dedicated right-edge shell that reuses the existing session/card
content and state, so notch-specific geometry stays isolated.

## Architecture

Existing layers already separate well:

- AppState / IslandSurface: behavior and interaction state
- PanelWindowController: AppKit window lifecycle and screen selection
- NotchPanelView: notch-specific presentation

Add:

- IslandPlacement enum (.notch, .rightEdge)
- SettingsKey.islandPlacement
- RightEdgePanelView (presentation shell)
- placement-aware PanelWindowController geometry/hosting

Keep NotchPanelView behavior unchanged when placement == .notch.

## Right-edge geometry

Right-edge window should:

- anchor to chosenScreen.frame.maxX
- vertically center by default
- clamp to the visible screen height
- expand leftward
- reserve a narrow collapsed hit target along the edge
- keep approval/question controls key-capable
- preserve all-Spaces/full-screen behavior from the existing NSPanel

Do not reuse the existing horizontal-drag preference in right-edge mode initially.
A later option may add vertical positioning.

## Usage / quota

CodeIsland already has ClaudeQuotaMonitor and footer rendering, which is the
correct product pattern to extend.

Add independent monitors for:

- Codex usage
- Antigravity usage

Port only the data/auth/parsing behavior needed from Codenotch. Do not import
Codenotch's provider registry, ring UI, or session monitoring.

The expanded session list should gain compact quota rows/footer sections for
enabled services. Quota fetching should remain event-driven / low-wakeup like
ClaudeQuotaMonitor.

### Codex

Read the same local account/profile source Codenotch uses and expose the useful
five-hour / weekly limits (and account identity only where useful). Support the
current active Codex profile first; multi-profile selection can follow.

### Antigravity

Read the same Antigravity account/quota source Codenotch uses and expose the
selected five-hour allowance plus useful Gemini/model allowance data. Existing
AntiGravityView is only the session mascot and should remain unrelated.

## Phases

1. Placement foundation
   - add IslandPlacement setting
   - make PanelWindowController placement-aware
   - introduce RightEdgePanelView
   - preserve notch mode bit-for-bit where possible
   - tests for panel frames and placement switching

2. Right-edge interaction parity
   - session list
   - approval/question
   - completion
   - hover/collapse
   - click-to-jump
   - multi-display/full-screen regression tests

3. Codex quota
   - monitor/client/model
   - settings toggle
   - compact footer UI
   - fixtures/tests

4. Antigravity quota
   - monitor/client/model
   - settings toggle
   - compact footer UI
   - fixtures/tests

## Non-goals

- no Codenotch session architecture port
- no second OpenCode implementation
- no rewrite of CodeIsland's hook/plugin protocol
- no change to existing notch UX unless required for shared abstractions
- no release/signing changes during feature development
