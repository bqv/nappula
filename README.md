# resources

An orphan root: the commit on this branch has no parent, so it shares no
history with `master`. One repository, two histories. Nothing here is source -
it is published assets and published dictionaries, kept in version control
beside the code that fetches them.

    v1/         the dictionary channel the app fetches
    releases/   this fork's pre-release assets, verbatim, one directory per tag

## v1/

`v1/` is a **drop-in replacement** for the dictionary channel the app reads
today. The app builds `<base>/v<format version>/<name>.dict`
(`DictionaryListView.urls_of_dictionary`), where the base is currently
`https://raw.githubusercontent.com/Julow/Unexpected-Keyboard-dictionaries/refs/heads/main`.
Point the base at this branch and the same files answer to the same names.

- **88 upstream dictionaries**, byte-identical copies of
  `Julow/Unexpected-Keyboard-dictionaries` at commit `9e1cf30` (2026-09-13),
  with their `.dict.sha256` sidecars and `substitutions.json`.
- **`ja.dict`, `ko.dict`, `zh.dict`** - this fork's CJK dictionaries. The bytes
  are exactly those the `ja-dict-v1`, `ko-dict-v1` and `zh-dict-v1`
  pre-releases publish (also kept under `releases/`), so they can be served
  from the same base as everything else. That is what lets the app drop the
  three `FALLBACK_URLS` that reach back to the releases.
- Every `.dict` is a **gzip stream whose payload is the cdict format**, as
  upstream's are; the app decompresses whatever the URL returns
  (`DictionaryListView.download_dictionary` calls `Download.gzip`), so the
  files must stay gzipped here.
- **`dictionaries.xml`** is upstream's file with the three fork dictionaries
  added. Every locale, label, size and checksum in it was re-derived from the
  files in this directory, and for the 88 upstream entries the derived values
  match upstream's declared ones exactly.
- Each `.dict.sha256` sidecar holds the SHA-256 of the **decompressed**
  content, then two spaces, then the path - upstream's convention. All 91 were
  checked against the files.

The three CJK dictionaries belong in the shared dictionary repository; until
upstream takes them, this branch carries them.

## releases/

The fork's pre-release assets, verbatim, so the bytes a release published stay
in version control even if a release is deleted.

| Directory | Release | Files (download size) |
| --- | --- | --- |
| `releases/ja-dict-v1/` | `ja-dict-v1` | `ja.dict.gz` 1,049,325 B, `ja_dict.txt.gz` 481,419 B |
| `releases/ko-dict-v1/` | `ko-dict-v1` | `ko.dict.gz` 221,823 B, `ko_wordlist.combined.gz` 233,301 B |
| `releases/zh-dict-v1/` | `zh-dict-v1` | `zh.dict.gz` 3,046,233 B, `zh_words.combined.gz` 2,315,077 B |
| `releases/hanwriting-model-v1/` | `hanwriting-model-v1` | `ja.model.gz` 5,296,996 B, `zh_TW.model.gz` 10,585,780 B, `COPYING` 26,430 B |

`releases/*/…dict.gz` are byte-identical to `v1/<name>.dict`; Git stores one
blob for both paths, so the duplication costs nothing. The word lists are the
inputs of the dictionary builds, not files the app fetches.

The two HanWriting models are what `ModelStore` downloads and verifies: an
installed file counts only when it is exactly 8,156,708 B (`ja.model`) or
16,327,356 B (`zh_TW.model`), the sizes of the decompressed models. `COPYING`
is the LGPL-2.1 text they ship with and belongs beside them.

## Verifying

    sha256sum -c SHA256SUMS                 # the bytes stored in this branch
    cd v1 && sha256sum -c *.dict.sha256     # the decompressed dictionaries

`SHA256SUMS` lists the stored bytes, so it also covers the gzip container and
the assets in `releases/`.

## Known skew

The app's own `res/values/dictionaries.xml` declares 556,287 B for `bn` and
887,655 B for `hy`, while the files here (and upstream's) are 556,196 B and
887,663 B. The app uses those numbers only for the "n MB" label in the
dictionary list - installation validates a dictionary by loading it, not by
its size - so this is a wrong figure in the UI, not a failed download.
Regenerate the app's copy from `v1/dictionaries.xml` when the base URL is
repointed.

## Refreshing

This branch is a snapshot, so it carries one commit rather than a history of
each re-publish. To take a newer upstream dictionary set, re-export
`Julow/Unexpected-Keyboard-dictionaries` into `v1/`, re-add the three fork
dictionaries and their sidecars, rebuild `dictionaries.xml` from the files,
regenerate `SHA256SUMS`, and amend.

Sizes, checksums and the upstream commit above were taken on 2026-10-15.
