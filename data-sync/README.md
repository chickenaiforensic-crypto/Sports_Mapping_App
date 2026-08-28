# Tennis Data Sync Artifacts

Files here are regenerated directly from `the_creation_2` repo, branch `arena/01a015bb-the-creation-2`, the confirmed single source of truth for tennis data.

## Current: tennis-data-regenerated-20260828-from-arena-edbef0a.json

204 editions / 14,946 matches. Includes:
- ATP/WTA naming reconciliation (Indian Wells/Miami/Madrid/Rome/Cincinnati/Canada)
- Ningbo 2024/2025
- WTA1000 depth added by concurrent work on the arena branch (Doha, Guadalajara, Beijing, Wuhan)
- Player name canonicalization: 7 reversed-word-order pairs found and fixed (e.g. "Ma Yexin"/"Yexin Ma" were splitting one real player into two identities). All traced to inconsistent naming in the Ningbo builds; standardized to the dataset's dominant Given-Family convention. Zero non-canonical occurrences remain, verified programmatically.

Replaces the app's original stale embedded tennis-data.js (119 editions / 8,555 matches).

To use: replace the `<script id="tennis-data-json" type="application/json">...</script>` block's contents in the app with this file's contents, then rebundle.

Source commit: arena/01a015bb-the-creation-2 @ edbef0a
