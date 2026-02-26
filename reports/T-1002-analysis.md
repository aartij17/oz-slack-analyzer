# Slack Activity Analysis — Tenant T-1002

## Tenant Summary

- **Tenant ID:** T-1002
- **Date range:** 2026-02-10 to 2026-02-11
- **Total events:** 21
- **Users observed:** charlie@globex.com, diana@globex.com

## Findings

### 1. Off-Hours Login from Anomalous Source with Scripted User Agent — HIGH

**Evidence:**
- Row 24: `charlie@globex.com` logged in at `2026-02-11T03:12:00Z` from source `external-unknown` using user agent `python-requests/2.28.1`.

**Why this is anomalous:**
Charlie's prior activity (rows 18–20) shows logins during business hours (~08:55 UTC) from `office-east-c` using a standard macOS browser (`Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)`). The 03:12 UTC login originates from an unrecognized external location and uses `python-requests/2.28.1`, which indicates automated or scripted access rather than a human browser session. This combination of off-hours timing, unknown source, and non-browser user agent is a strong indicator of compromised credentials being used programmatically.

---

### 2. Bulk File Downloads Across Sensitive Channels — HIGH

**Evidence:**
- Row 25: `file_downloaded` — `roadmap-2026.pdf` from `#product` at `03:14`
- Row 26: `file_downloaded` — `architecture-diagram.png` from `#engineering` at `03:15`
- Row 27: `file_downloaded` — `q4-pipeline.xlsx` from `#sales` at `03:16`
- Row 28: `file_downloaded` — `board-deck-feb.pptx` from `#executive` at `03:17`
- Row 29: `file_downloaded` — `salary-bands-2026.xlsx` from `#hr-confidential` at `03:18`

**Why this is anomalous:**
Five files were downloaded in a 4-minute window from five different channels, including highly restricted ones (`#executive`, `#hr-confidential`). The files include strategic plans, financial data, and compensation information. All downloads used the same scripted user agent (`python-requests/2.28.1`) from `external-unknown`. This pattern is consistent with automated data exfiltration.

---

### 3. Unauthorized Privilege Escalation — HIGH

**Evidence:**
- Row 30: `permission_changed` at `2026-02-11T03:22:00Z` — `charlie@globex.com role changed from member to admin`, performed via `python-requests/2.28.1`.

**Why this is anomalous:**
A user account elevated its own role from `member` to `admin` using an automated tool during the same off-hours session. Self-elevation of privileges is a critical security event, especially when performed programmatically from an unrecognized source. This likely enabled the destructive actions that followed.

---

### 4. Mass Deletion of Sensitive Channels — HIGH

**Evidence:**
- Row 31: `channel_deleted` — `#finance-internal` at `03:25`
- Row 32: `channel_deleted` — `#hr-confidential` at `03:26`
- Row 33: `channel_deleted` — `#executive` at `03:27`
- Row 34: `channel_deleted` — `#legal-holds` at `03:28`

**Why this is anomalous:**
Four channels were deleted in 3 minutes, all containing sensitive or confidential information. The deletion of `#legal-holds` is particularly severe as this channel likely contains data subject to legal preservation requirements; destroying it may constitute spoliation of evidence. All deletions were performed via the scripted session from `external-unknown`, immediately after the privilege escalation.

---

### 5. Removal of Another User from Channels — MEDIUM

**Evidence:**
- Row 35: `member_removed` — removed `diana@globex.com` from `#product` at `03:30`
- Row 36: `member_removed` — removed `diana@globex.com` from `#sales` at `03:31`

**Why this is anomalous:**
Immediately following the mass deletions, `diana@globex.com` was removed from two channels. This may be an attempt to limit the other workspace user's visibility into the attacker's actions or to further disrupt workspace access. The removals were performed via the same automated session.

---

## Attack Sequence Summary

The events from rows 24–36 form a coherent attack chain executed in under 20 minutes:

1. **03:12** — Automated login from external source (credential compromise)
2. **03:14–03:18** — Bulk download of sensitive files (data exfiltration)
3. **03:22** — Self-elevation to admin (privilege escalation)
4. **03:25–03:28** — Deletion of 4 sensitive channels (evidence destruction)
5. **03:30–03:31** — Removal of another user from channels (access disruption)

## Recommendations

1. **Immediately disable charlie@globex.com's account** and revoke all active sessions. Reset credentials and rotate any tokens/API keys associated with this account.
2. **Revert the admin privilege escalation** — restore charlie's role to `member` (or lower) pending investigation.
3. **Attempt to recover deleted channels** (`#finance-internal`, `#hr-confidential`, `#executive`, `#legal-holds`) from backups or Slack's data retention features.
4. **Restore diana@globex.com's channel memberships** in `#product` and `#sales`.
5. **Preserve all audit logs** related to this incident for forensic analysis and potential legal proceedings, especially given the deletion of `#legal-holds`.
6. **Investigate the source IP** behind `external-unknown` to determine the origin of the attack.
7. **Review authentication controls** — consider enforcing MFA, restricting API-token-based access, and alerting on `python-requests` or other non-browser user agents.
8. **Notify legal and compliance teams** about the deletion of `#legal-holds`, as this may have regulatory implications.
