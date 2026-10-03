# League of Legends census history

Every live build of the League of Legends game client from patch 8.20 (October 2018) to now, as a
git history: **one commit per build, one file per fact**, so a patch reads as a diff. The builds
up to 16.19.8207193 were written by an export from a database of past builds; each build after it
is appended from its own bytes by [ltk-census-sync](https://github.com/LeagueToolkit/ltk-census-sync).
Nothing here is edited by hand.

**This branch holds only this file and the license.** The history is on the branch `history`,
whose newest tree has 1.4 million files -- one small file per WAD entry or bin entry -- which is
why it is not what a plain clone checks out.

## Getting it

A plain `git clone` downloads every branch, history included, and checks out only this README.
To work with the data:

```sh
# the history only, without checking out a million files
git clone --single-branch --branch history --no-checkout https://github.com/LeagueToolkit/ltk-census-history.git census
cd census

# then read it without a working tree
git log --oneline                                  # one line per build
git show 16.19:build.yaml                          # a build's version, date, realms
git diff --stat 16.18 16.19                        # what a patch changed
git diff 16.18 16.19 -- '*.rito'                    # ... in bins, entry by entry, as ritobin text
git log --oneline -- files/data/final/champions/ahri.wad.client/a8/a847c7a46bc6730e.yaml   # one entry's versions

# or check out a few kinds of file only
git sparse-checkout set --no-cone /build.yaml /census.yaml /schema/ '*.rito' '*/_wad.yaml'
git checkout history
```

Each patch is tagged with its name (`8.20` ... `16.19`) at its newest build when the next patch
began, so `git diff 16.18 16.19` is one patch. A tag never moves: a hotfix that came after it lands
after it, and `build.yaml` says which build a commit is. Each commit message is the build's version followed by
trailers, so `git log --grep 'Census-Source: rads'` finds builds without reading a tree:

```
16.19.8207193

Census-Patch: 16.19
Census-Manifest: ad6896ce5cf475a1
Census-Source: rman
Census-Realms: BR1 EUN1 EUW1 JP1 KR LA1 LA2 ME1 NA1 OC1 RU SG2 TR1 TW2 VN2
```

## Layout

```
census.yaml                                           format 2; the kinds of file and their schemas
schema/<name>.schema.json                             a JSON Schema per kind of YAML file
build.yaml                                            this build: version, patch, manifest, source, date, realms
files/<manifest path>/_wad.yaml                       one WAD: format version, the manifest's file id and tags
files/<manifest path>/xx/<hash>.yaml                  one WAD entry: sha256, checksum, kind, and what its bytes
                                                      say -- skeleton, mesh, texture, sound bank, or a bin's
                                                      links and its objects' keys
files/<manifest path>/xx/<hash>.bin/yy/<entry>.rito   one entry of a bin, as ritobin text (as C++ ritobin writes it)
```

- **Every file is about one WAD or one WAD entry.** A WAD that did not change is a directory that
  did not change, and `git log -- files/<manifest path>/` is that WAD's history.
- **`<manifest path>`** is the file's path in the game's manifest, lowercased. The name says which
  locale a WAD is (`ahri.en_us.wad.client`); the manifest's tags in `_wad.yaml` say which realms
  consume it.
- **An entry** is named for its path hash -- XXH64 of the lowercased path -- in the directory of the
  hash's first byte. Its `.yaml` holds the SHA-256 of its bytes (the same anywhere means the same
  bytes: "did the game ever ship this file" is a search for it), the checksum the WAD's table holds
  for it, its kind by magic, and then what the bytes say under a key per kind: `skeleton`, `mesh`,
  `texture`, `bank`, or a bin's `links` and `objects` (each object's class and the SHA-256 keys of
  its binary bytes, which the ritobin text cannot give back).
- **A bin** is a directory beside its entry's file, `<hash>.bin/`, one file per bin entry named for
  its entry hash, in the directory of the hash's first byte. Each is the ritobin text of a bin
  holding that one entry; the whole bin is its `links` and the entries of every file under
  `<hash>.bin/` in path order, which is exactly what C++ ritobin writes for it.
- **Hashes stay hex.** Bin entry, class and field names are FNV-1a 32 and paths XXH64; naming them is
  the job of the community hash lists, not of this repository.
- **The YAML** is a strict subset: every string double-quoted, every integer plain, block style,
  one space of indentation. Any YAML reader loads it the same, and every file validates against
  the schema `census.yaml` names for it. Every number is unsigned; a bin's links are sorted and
  its entries go by hash.
- **8.20 to 9.1** were read from Riot's RADS distribution and have `source: rads` in
  `build.yaml`: their `manifest` is a census id (no RMAN manifest exists for them), their date the
  client executable's link time. Builds before 8.20 shipped game data partly as loose files and are
  not in this history.
- **Realms.** Up to 16.19.8207193, `realms` lists the live realms that shipped a build. After it,
  the history follows NA1 alone, and `realms` is `NA1`.

The format and the measurements behind it are described in ltk-census-sync, `docs/FORMAT.md` and
`docs/RUNS.md`.

## License

The data is dedicated to the public domain, without warranty, and grants no right in Riot Games'
own work: [LICENSE](LICENSE). ltk-census-history isn't endorsed by Riot Games.
