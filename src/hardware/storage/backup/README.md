# Backup <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Backup and deduplication tools and techniques for storage.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
* [Deduplication](#deduplication)
  * [jdupes](#jdupes)
  * [rdfind](#rdfind)
  * [czkawka](#czkawka)
    * [Comparing a source and target with Krokiet](#comparing-a-source-and-target-with-krokiet)
  * [rmlint](#rmlint)
  * [Which one to use](#which-one-to-use)
* [Restic](#restic)
* [BorgBackup](#borgbackup)
* [Restic vs BorgBackup](#restic-vs-borgbackup)

## Overview

## Deduplication
The tools above (Restic/Borg) dedup *at backup time* going forward — they don't help with a pile
of manual backups you already have sitting on disk with countless identical copies of the same
files. For that you want a dedicated file-level dedup scanner: something that hashes files in
bulk, groups identical ones, and lets you auto-delete or hardlink the extras in one pass.

### jdupes
[jdupes](https://github.com/jbruchon/jdupes) is a fast fork of the classic `fdupes`, rewritten
for speed (size-match pre-filter before hashing, multi-threaded I/O). It's the best fit for "scan
a huge tree, find dupes, kill them" workflows.

```bash
# ephemeral shell with the tool
$ nix shell nixpkgs#jdupes

# recursively scan and just list duplicate sets
$ jdupes -r /mnt/backups

# recursively scan and auto-delete all but the first copy in each duplicate set
# (default order is by filename), no prompts
$ jdupes -r -N -d /mnt/backups

# keep only the newest copy in each duplicate set, delete the rest, no prompts
# -o time sorts oldest-first, -i reverses that so the newest file is first/kept
$ jdupes -r -o time -i -N -d /mnt/backups

# summary only, no action — good for a dry-run before -d or -L
$ jdupes -r -m /mnt/backups

# compare a reference directory against a target directory, deleting duplicates only
# from the target — the reference directory is never touched
# -O makes command-line order win the tie-break, so listing the reference dir first
# means it's always the file preserved in each set
$ jdupes -r -O -N -d /mnt/reference /mnt/target
```

* `-N` — no-prompt mode, required for unattended auto-delete
* `-d` — delete duplicates (combine with `-N` to skip the interactive picker), preserving only
  the first file in each set
* `-o time` — sort each duplicate set by modification time (oldest first)
* `-i` — reverse the sort order (combine with `-o time` to put the newest file first, so it's the
  one preserved by `-d`)
* `-O` — param-order: prioritizes command-line directory order over the sort order, so the first
  directory listed wins the tie-break in every match set — used to designate a reference directory
  that should never be deleted from
* `-L` — hardlink duplicates instead of deleting (zero data loss, same space savings)
* `-r` — recurse into subdirectories

jdupes has no dry-run flag, so before adding `-N -d`, run without them first (e.g.
`jdupes -r -o time -i /mnt/backups`) to confirm the printed order in each set is what you expect —
the first file listed is the one that survives.

**Caveat on the reference/target pattern:** `-O` only breaks ties in favor of the reference
directory — it does not exempt the reference directory from deletion. jdupes fully dedupes
*everything* across both directories: if the reference directory itself contains multiple copies
of the same file, `-N -d` will delete all but one of those too, not just the copies living in the
target directory. It doesn't honor a true "never touch this directory" concept the way Krokiet's
reference-folder feature does — only use `-O` this way if the reference directory is already
known to be duplicate-free internally.

**WARNING**: for my actual use case — a reference library that itself has duplicates, where I
only want dupes removed from the target/manual-backups side — `-O` doesn't work. It'll happily
delete extras out of the reference directory too. Don't reach for jdupes here; use Krokiet's
reference-folder feature instead, which genuinely never touches files marked as reference.

**References**
* [jdupes GitHub](https://github.com/jbruchon/jdupes)

### rdfind
[rdfind](https://github.com/pauldreik/rdfind) (redundant file finder) takes a different approach:
instead of blindly picking the "first" file in a duplicate set, it *ranks* candidates using
configurable criteria (directory depth, ctime, path given first on the command line) to decide
which copy is the "original" and which are the "duplicates" — useful when you care which specific
copy survives.

```bash
$ nix shell nixpkgs#rdfind

# scan and just report what it found (dry run, writes results.txt)
$ rdfind -dryrun true /mnt/backups

# replace duplicates with hardlinks to the ranked original
$ rdfind -makehardlinks true /mnt/backups

# actually delete the duplicates instead of hardlinking
$ rdfind -deleteduplicates true /mnt/backups
```

**References**
* [rdfind GitHub](https://github.com/pauldreik/rdfind)

### czkawka
[czkawka](https://github.com/qarmin/czkawka) is the shared core behind a family of Rust dedup
tools that go beyond exact-duplicate hashing — they can also find similar (not just identical)
images, empty folders, broken symlinks, and more. Noticeably faster than fdupes-family tools on
very large trees thanks to parallel hashing.

The project ships two frontends over the same `czkawka_core` library:
* **`czkawka_cli`** — the CLI, still actively maintained and what the example below uses.
* **GUI** — the original GTK4 `czkawka` GUI is now deprecated as of v12.0 (no further binaries
  planned). It's been superseded by **[Krokiet](https://github.com/qarmin/czkawka)**, a newer
  Slint-based GUI. If you want an interactive eyeball-before-delete pass, reach for `krokiet`
  (`nix shell nixpkgs#czkawka` also provides a `krokiet` binary), not the old `czkawka_gui`.

**Gotcha — small files are silently skipped:** `czkawka_cli` defaults `--minimal-file-size` (`-m`)
to `8192` bytes (8 KiB). Any file smaller than that is excluded from scanning entirely, with no
warning — it'll report "Not found any duplicates" even when obvious small duplicates exist, and
`-D <method>` has nothing to delete as a result. Confirmed this is the cause of `-D AEN` appearing
to do nothing: files above 8 KiB deleted correctly, files below it were silently never scanned. If
your duplicates are small (docs, configs, thumbnails), pass `-m 1` to drop the size floor.

```bash
$ nix shell nixpkgs#czkawka

# list duplicates only, no deletion (default --delete-method is NONE)
$ czkawka_cli dup -d /mnt/backups

# delete all but the oldest file in each duplicate group
$ czkawka_cli dup -d /mnt/backups -D aeo

# safer: hardlink duplicates together instead of deleting
$ czkawka_cli dup -d /mnt/backups -D hard

# include small files too (drop the 8 KiB default size floor)
$ czkawka_cli dup -d /mnt/backups -D aeo -m 1
```

**References**
* [czkawka GitHub](https://github.com/qarmin/czkawka)

#### Comparing a source and target with Krokiet
Krokiet has a **reference folder** concept that's exactly suited to "compare a pile of manual
backups against the library I actually want to keep": you mark the trusted/target directory as a
reference folder, and Krokiet only ever proposes deleting duplicates found *outside* it — files in
the reference folder itself are never touched.

1. Launch Krokiet
   ```bash
   $ nix shell nixpkgs#czkawka -c krokiet
   ```
2. Open the **Duplicate Files** tool from the left-hand tool list
3. Under **Included Directories**, add both:
   * the target/master directory you trust and want to keep (e.g. `/mnt/library`)
   * the source directory full of scattered manual backups (e.g. `/mnt/backups`)
4. Select the target/master directory in the list and toggle it as a **Reference folder**. This
   tells Krokiet "files here are the ones to keep" — duplicate sets that include a reference-folder
   file will only ever offer to delete the *other* (non-reference) copies.
5. Pick a check method — `Hash` (full content comparison, most reliable) is the default; `Size`
   alone is faster but only useful as a rough pre-filter.
6. Click **Search** and let it scan both directories
7. Review the results grouped by duplicate set — copies living under the source directory will be
   flagged against their matching file in the reference/target directory
8. Use **Select > Select all except reference folders** (or the equivalent smart-select action) to
   select only the redundant source-side copies, never the reference copies
9. Delete the selection — Krokiet supports sending to trash instead of a hard delete, which is
   worth using the first time through until you trust the results

### rmlint
[rmlint](https://rmlint.readthedocs.io) *does* correctly implement a true reference directory:
files in a "tagged" path (after a bare `//` on the command line, with `-k -m`) are never deleted,
even if that path has internal duplicates — verified this live, and it's the one tool of the four
here that actually gets this right (jdupes' `-O` and rdfind's ranking both fail it, see the
[jdupes WARNING](#jdupes)).

**Struck from consideration anyway:** rmlint doesn't delete anything itself — it generates a
`rmlint.sh` removal script (plus a `rmlint.json`) that you have to inspect and run separately. The
default terminal output is a low-level `ls`/`rm` command dump, not something that's easy to eyeball
and make a keep/delete judgment call from before committing. For a manual, one-off cleanup this
workflow is more friction than it's worth compared to Krokiet's interactive results view.

**References**
* [rmlint Documentation](https://rmlint.readthedocs.io)
* [rmlint GitHub](https://github.com/sahib/rmlint)

### Which one to use
For a large one-off pile of manual backups with no reference/protected directory involved,
**jdupes** is the pragmatic default: fast, simple flags, and `-L` lets you reclaim space via
hardlinks without actually deleting anything — the safest option when you're not 100% sure which
copies matter. Reach for **rdfind** if you need control over *which* copy in each duplicate set
survives.

If you have a **reference directory that must never be touched** (even if it has internal
duplicates) — e.g. deduping scattered manual backups against a trusted master library — reach for
**Krokiet**'s reference-folder GUI. jdupes `-O` and rdfind's ranking both fail this specific case
(see the [jdupes WARNING](#jdupes)); rmlint gets the behavior right but its script-based workflow
and raw output aren't worth the friction for a manual review-and-decide pass.
Reach for **czkawka**/**Krokiet** generally if you also want a GUI to review before committing —
just remember [the 8 KiB minimum file size gotcha](#czkawka) if your duplicates are small.

**Recommended workflow:**
1. Dry run first — `jdupes -r -m /mnt/backups` (or `rdfind -dryrun true`) to see how much space is
   actually recoverable before touching anything.
2. Hardlink, don't delete — `jdupes -r -L /mnt/backups` reclaims the disk space immediately while
   leaving every path in place, so nothing you scripted or bookmarked against those paths breaks.
3. Only delete outright (`-d -N`) once you've confirmed via the dry run that the "kept" copy in
   each set is the one you'd actually want.

## Restic
[Restic](https://restic.net) is a fast, modern backup tool written in Go. It splits data into
content-defined chunks using a rolling hash, so identical chunks are only ever stored once across
snapshots — even if the underlying files moved or were renamed. Every chunk is encrypted
(`AES-256`) and authenticated client-side before it ever leaves the machine, so the backend never
sees plaintext.

Repositories are just directories of encrypted blobs, which makes Restic backend-agnostic. It
supports local paths, SFTP, `rest-server`, and native drivers for `S3`, `B2`, `Azure`, `GCS`, and
others — no extra sync tool required to ship backups off-site.

NixOS has a first-class module for it, `services.restic.backups.<name>`, which lets you declare
the repository, paths, excludes, and a systemd timer schedule entirely in config:

```nix
services.restic.backups.homeserver = {
  paths = [ "/var/lib" "/home" ];
  repository = "s3:s3.amazonaws.com/my-backup-bucket";
  passwordFile = "/etc/nixos/secrets/restic-password";
  timerConfig = {
    OnCalendar = "daily";
    Persistent = true;
  };
  pruneOpts = [
    "--keep-daily 7"
    "--keep-weekly 4"
    "--keep-monthly 6"
  ];
};
```

**References**
* [Restic Documentation](https://restic.readthedocs.io)

## BorgBackup
[BorgBackup](https://www.borgbackup.org) (Borg) is a deduplicating archiver written in Python/C,
older and more established than Restic. Like Restic it uses content-defined chunking so identical
data across snapshots and even across different backed-up machines sharing a repo is stored only
once. It supports compression (`lz4`, `zstd`, `zlib`, `lzma`) and encryption (`AES-256` +
`HMAC-SHA256`) on the client side.

Borg repositories are only reachable over local paths or `SSH` — there's no native `S3`/`B2`/cloud
driver, so shipping a Borg repo off-site means pairing it with something like `rclone` or hosting
it on a remote box that already has Borg installed. `borgmatic` is a popular companion tool that
wraps Borg with YAML-based config, retention policies, and pre/post hooks.

NixOS also has a first-class module, `services.borgbackup.jobs.<name>`:

```nix
services.borgbackup.jobs.homeserver = {
  paths = [ "/var/lib" "/home" ];
  repo = "borg@backup-host:/srv/borg/homeserver";
  encryption = {
    mode = "repokey-blake2";
    passCommand = "cat /etc/nixos/secrets/borg-password";
  };
  compression = "zstd";
  startAt = "daily";
  prune.keep = {
    daily = 7;
    weekly = 4;
    monthly = 6;
  };
};
```

**References**
* [BorgBackup Documentation](https://borgbackup.readthedocs.io)

## Restic vs BorgBackup
Both are solid, mature, content-defined-chunking dedup tools with declarative NixOS modules — the
choice usually comes down to where the backups need to land and how much repo maturity/tooling
matters to you.

| | Restic | BorgBackup |
|---|---|---|
| Language | Go | Python/C |
| Dedup method | Content-defined chunking (rolling hash) | Content-defined chunking (rolling hash) |
| Encryption | AES-256, always on | AES-256 + HMAC-SHA256, optional but default |
| Compression | zstd (since 0.14) | lz4, zstd, zlib, lzma |
| Backends | Local, SFTP, rest-server, native S3/B2/Azure/GCS | Local, SSH only (needs rclone for cloud) |
| Multi-host dedup | Yes, if sharing a repo | Yes, if sharing a repo |
| NixOS module | `services.restic.backups` | `services.borgbackup.jobs` |
| Maturity | Newer, active development | Older, very mature, larger feature set |
| Companion tooling | Built-in `restic` CLI is fairly complete | `borgmatic` widely used for config/scheduling |

**Recommendation:** if you're backing up straight to cloud object storage (`S3`/`B2`), Restic's
native backend support avoids an extra hop. If you're backing up to a remote host over `SSH` (e.g.
another NAS or homelab box) and want the more battle-tested repo format, Borg is the stronger
choice.
