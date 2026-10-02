# KOReader highlights for YAOS

Automatically send KOReader highlights, quotes, attached notes, and bookmarks to an Obsidian vault synchronized by [YAOS](https://github.com/kavinsood/yaos). Each book becomes a Markdown note in `Reading Highlights/`. Uploads originate on the Kobo over Wi-Fi; a Mac or desktop importer is not required.

This is a [Tomedown](https://github.com/imanubdesigner/tomedown.koplugin) v0.7.1 derivative. Tomedown provides the Markdown export and KOReader cloud upload; this fork adds change-triggered uploads, first-connection history import, last-annotation deletion, and a YAOS connection screen. It retains Tomedown's GPL-3.0 license and attribution.

## Requirements

- KOReader with its bundled **Cloud storage** plugin enabled.
- A YAOS v2.1.1 Cloudflare Worker with an R2 `YAOS_BUCKET` binding and the [Kobo inbox patch](server/yaos-v2.1.1-kobo-inbox.patch) below. The standard YAOS server does not provide this route yet.
- YAOS Obsidian plugin connected to the same Worker and vault on each Obsidian device.

## Configure the YAOS Worker

Use a checkout of [kavinsood/yaos](https://github.com/kavinsood/yaos) at commit `2174eb89538ce7e213e9e33a0e75ecaf1a3f1f58` (v2.1.1). From that checkout:

```sh
git apply /path/to/koreader-yaos-highlights/server/yaos-v2.1.1-kobo-inbox.patch
cd server
npm ci
npm run typecheck
npx wrangler secret put KOBO_SYNC_TOKEN
npx wrangler secret put KOBO_VAULT_ID
npx wrangler deploy
```

Use a strong, unique random value for `KOBO_SYNC_TOKEN`. Enter the **same value** on the Kobo. Use the **Vault ID** shown in YAOS settings in Obsidian for `KOBO_VAULT_ID`. This vault ID must match the devices where you want the notes. The Kobo token is separate from YAOS's regular sync token. Keep both secrets private. Deployment requires your own Cloudflare account, Worker, and R2 bucket. See the [YAOS deployment guide](https://github.com/kavinsood/yaos#quick-start) for base setup.

The patch writes each upload to YAOS's shared vault document and keeps an R2 backup. It accepts authenticated Markdown uploads only under `/kobo/`; it does not provide a general WebDAV service. This patch is pinned to YAOS v2.1.1 and must be reviewed or rebased when YAOS changes.

## Install on KOReader

1. Download the release ZIP and extract its `tomedown.koplugin` folder into `koreader/plugins/` on your Kobo. Restart KOReader.
2. Open **Tomedown for YAOS → Settings → Cloud → Connect to YAOS Worker**.
3. Enter your Worker HTTPS URL (for example, `https://your-worker.example.workers.dev`) and the `KOBO_SYNC_TOKEN` created above.
4. Keep Wi-Fi enabled when you want uploads. The first connection exports your existing library. Later highlight, note, and bookmark changes export after a one-second debounce. Changes made offline queue locally and upload after reconnection.

Tomedown also writes local Markdown exports under KOReader's `clipboard/tomedown/` directory. On YAOS devices, synced files appear under `Reading Highlights/`. Deleting a book's final annotation removes its managed note from the vault; deleting one of several annotations updates the note. Edit or add personal material in a separate Obsidian note, because subsequent Kobo exports replace the generated note.

### What is exported?

The generated note identifies the book and contains highlighted passages/quotes, attached annotation text, and bookmarks when enabled. Sync occurs when KOReader is running and the Kobo is online; an offline device cannot upload until it reconnects. The one-second interval only groups rapid edits, rather than polling continuously.

## Safety and attribution

The Worker writes only to notes marked as managed by this integration. It refuses to overwrite an unmarked note at the same path. It stores a backup of each uploaded Markdown file in R2. Kobo credentials are saved in KOReader's local plugin settings; no user credentials are included in this repository or release.

Original Tomedown by [imanubdesigner](https://github.com/imanubdesigner/tomedown.koplugin), licensed under GPL-3.0. YAOS by [kavinsood](https://github.com/kavinsood/yaos); the included server patch is distributed under YAOS's license.
