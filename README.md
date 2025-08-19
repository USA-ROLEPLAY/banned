<div align="center">
  <p align="center">
    <a href="#">
      <img src="https://raw.githubusercontent.com/USA-ROLEPLAY/repo-resources/main/Org/logo.png" width="144" height="144" />
    </a>
  </p>
</div>

<div style="border: 2px solid #d1d5db; padding: 20px; border-radius: 8px; background-color: #f9fafb;">
  <h2 align="center">USARoleplay • Global Bans & Word Lists</h2>
  <p align="center">Single source of truth for organization-wide bans, ban appeals, and banned word lists — across all current and future game servers.</p>
  <p align="center">
    <a href="https://usa-roleplay.org">
      <img src="https://img.shields.io/badge/Website-0077b5?style=for-the-badge&logo=github&logoColor=white" alt="Website" />
    </a>
    <a href="https://discord.gg/YURaDVRdet">
      <img src="https://img.shields.io/badge/Discord-7289da?style=for-the-badge&logo=discord&logoColor=white" alt="Discord" />
    </a>
    <a href="https://github.com/USA-ROLEPLAY/Ark_Servers">
      <img src="https://img.shields.io/badge/Game%20Repos-6c5ce7?style=for-the-badge&logo=game&logoColor=white" alt="Games" />
    </a>
  </p>
</div>

---

## 🤖 Workflow Status

> These badges will light up as we add/enable automation.

[![Ban Appeal Guard](../../actions/workflows/ban-appeal-guard.yml/badge.svg)](../../actions/workflows/ban-appeal-guard.yml)
[![Ban Appeal Status Update](../../actions/workflows/ban-appeal-status-update.yml/badge.svg)](../../actions/workflows/ban-appeal-status-update.yml)
[![Ban Registry Validate](../../actions/workflows/ban-registry-validate.yml/badge.svg)](../../actions/workflows/ban-registry-validate.yml)
[![Ban Registry Build Exports](../../actions/workflows/ban-registry-build.yml/badge.svg)](../../actions/workflows/ban-registry-build.yml)
[![Ban Registry Publish](../../actions/workflows/ban-registry-publish.yml/badge.svg)](../../actions/workflows/ban-registry-publish.yml)

---

## 🔍 What This Repo Is

This repository is the **global registry** for moderation across **all** USARoleplay servers:

- 🧑‍⚖️ **Ban Registry**: A unified list of banned players with cross-game identifiers (e.g., SteamID, EpicID, Xbox, PSN, Discord snowflake).
- 🚫 **Banned Word Lists**: Organization-wide lists and per-game overlays for chat/username moderation.
- 📝 **Ban Appeals**: Players submit appeals here (one open at a time). Automation enforces cooldowns and blocks spam.
- 📦 **Per-game Exports**: Automation compiles game-specific ban/word exports that each game server can consume.

> Today we actively export **ARK** lists. This design scales to additional titles as we add them.

---

## 🧭 How It Works (High Level)

1. **Moderators** add/edit entries in the **global registry** (`registry/bans.yml`).
2. **Validation workflow** lints and deduplicates identifiers, checks schema, and prevents unsafe edits.
3. **Build workflow** compiles **per-game exports** under `exports/<game>/…` (e.g., ARK SteamID lists, global word lists + game overrides).
4. **Publish workflow**:
   - Commits exports back to this repo (versioned),
   - Optionally opens PRs to game repos or updates a storage bucket/API endpoint that game servers pull from on restart/cron.
5. **Game servers** consume their **export** files (raw URL or mirrored copy) and apply bans/filters.
6. **Appeals**:
   - Players open a “Ban Appeal” issue here.
   - Automation enforces **1 open appeal**, **14-day cooldown** between appeals, **6-month** wait after a denial.
   - Moderators mark **approved/denied**, automation updates state and (optionally) opens a PR removing or annotating the ban.

---

## 📁 Repository Layout

```text
.github/
  ISSUE_TEMPLATE/
    ban_appeal.yml          # Issue form for appeals
    config.yml              # Disables blank issues & routes users to forms
  workflows/
    ban-appeal-guard.yml         # Enforce 1-open appeal, cooldowns, blocklist
    ban-appeal-status-update.yml # Extend/clear cooldown on deny/approve
    ban-appeal-prune.yml         # Daily cleanup of expired cooldowns
    ban-registry-validate.yml    # Lint/validate schema & identifiers
    ban-registry-build.yml       # Build per-game exports from global registry
    ban-registry-publish.yml     # Publish exports (commit/PR/artifacts)

registry/
  bans.yml                 # 🔑 Global ban registry (source of truth; schema below)
  notes/                   # Optional case notes (no PII), referenced by ID

wordlists/
  global.txt               # Words/phrases banned org-wide (one per line)
  exceptions.txt           # Allowlist to reduce false positives
  ark.txt                  # ARK-specific extras (optional)
  <future-game>.txt        # Game-specific overlays

exports/
  ark/
    steamids.txt           # Built file: one SteamID64 per line (ARK consumers)
    banned-words.txt       # Built word list (global + ark overlay − exceptions)
  <future-game>/
    ...                    # Built files tailored to that game’s needs

state/
  appeal_state.json        # ⚠️ Automation-managed cooldowns (do not edit manually)

lists/
  appeal_blocklist.txt     # Hard block list (GitHub usernames or IDs; one per line)
