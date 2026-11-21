# Ennovatec changes to imap-mcp-server

Author: stavros.vrakas@ennovatec.com, Ennovatec Technologies LLC
Upstream: nikolausm/imap-mcp-server (MIT license). Our fork: github.com/svrakas74-a11y/imap-mcp-server
Base: upstream **main f1c4426** (after v2.2.0; includes the security dependency updates, npm audit clean)
Running branch: `ennovatec` (our three changes + this note). Local clone on each PC at `C:\Users\svrak\imap-mcp-server`
Started: 2026-11-20. Rebuilt 2026-11-21 after the author's review of PR #231.

## Why

The daily mailbox cleanup task moves mail to Trash without anyone watching. These changes make it safer
and easier to check, and they replace the manual PowerShell step for DMARC reports.

## What changed (all optional, off unless configured)

Each change is also its own branch and its own upstream pull request, so the author can decide on each separately.

| # | Change | Branch / PR | Files |
|---|---|---|---|
| 1 | `IMAP_MCP_MOVE_ALLOWED_FOLDERS`: allowed move destinations, exact name match (INBOX case-insensitive). We use `Trash,INBOX`; INBOX is listed so a move can be undone | `feat/move-allowlist` | `src/utils/move-policy.ts`, `src/tools/email-tools.ts` |
| 2 | `IMAP_MCP_MOVE_LOG`: CSV log of every move, optional `reason` field. New file is owner-only, header created atomically | `feat/move-log` | `src/utils/move-log.ts`, `src/tools/email-tools.ts`, `src/services/imap-service.ts` (`fetchEnvelopesForUids`) |
| 3 | Read-only tool `imap_get_dmarc_report` with size, entry-count, per-call and output limits | `feat/dmarc-report` | `src/utils/dmarc.ts`, `src/tools/dmarc-tools.ts`, `src/tools/index.ts`, `src/services/imap-service.ts` (`getAllAttachments`) |
| 4 | Run from this local build on both PCs instead of `npx imap-mcp-server@2.2.0` | (config only) | `%APPDATA%\Claude\claude_desktop_config.json` |
| 5 | Tool `imap_save_reply_draft` (2026-11-20, Stavros's request): reply drafts, never sent, quoted like Outlook Reply All and built on the server (header block, the original's formatted HTML + head styles, inline images re-attached by Content-ID), so long threads keep their formatting | `feat/reply-draft-quote` (no PR yet) | `src/utils/reply-quote.ts`, `src/services/imap-service.ts` (`getReplyQuoteSource`), `src/tools/email-tools.ts` |

Tests: `tests/email-tools-move-allowlist.test.ts`, `tests/email-tools-move-log.test.ts`, `tests/dmarc.test.ts`, `tests/reply-quote.test.ts`.
README and CHANGELOG are updated in each branch.

## Claude desktop config (titan-imap entry)

```json
"titan-imap": {
  "command": "C:\\Program Files\\nodejs\\node.exe",
  "args": ["C:\\Users\\svrak\\imap-mcp-server\\dist\\index.js"],
  "env": {
    "IMAP_MCP_ENABLED_TOOLS": "<read tools>,imap_save_draft,imap_save_reply_draft,imap_move_email,imap_get_dmarc_report",
    "IMAP_MCP_MOVE_ALLOWED_FOLDERS": "Trash,INBOX",
    "IMAP_MCP_MOVE_LOG": "C:\\Users\\svrak\\Qsync\\Personal\\Personal Backup Dirs\\ennovatec\\Governance\\Email\\mailbox-moves.csv"
  }
}
```

Both PCs write the same move log through Qsync. Moves from both PCs at nearly the same time could create a
Qsync conflicted copy; the daily task runs on the desktop only.

## Updating a PC to the latest `ennovatec` branch

```powershell
cd C:\Users\svrak\imap-mcp-server
git fetch fork
git checkout ennovatec
git reset --hard fork/ennovatec        # the branch was rebuilt on 2026-11-21, so a plain pull won't work
npm install --include=dev               # npm omit=dev is set on these PCs
npm run build; npm test                 # all tests must pass
```

Then quit Claude from the tray icon and reopen it.

## Moving to a new upstream version

Rebuild the three feature branches on the new upstream main (`git rebase upstream/main` on each), then
recreate `ennovatec` from upstream main by cherry-picking the three feature commits and this note. If the author
merges one of the changes, drop it from the list. Run `npm audit`, build, tests and the type check before deploying.

## Upstream submission

- 2026-11-20: PR #231 (all three in one). The author asked for a split, two fixes (exact allowlist matching;
  complete DMARC limits), CSV permission and concurrency review, and a rebase onto current main for the
  dependency updates.
- 2026-11-21: all of the above done in the three feature branches. New PRs pending Stavros's OK.
