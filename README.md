# Throne of Carina mod catalog

The curated list of mods the game's mod library shows under BROWSE. Every mod here was reviewed by the keeper
and checked by the game's own importer and compiler; the list is signed, and the game shows nothing that fails
the signature.

- Players: open MODS in the game, then BROWSE. Mods install with one click (or, for mods hosted elsewhere, open
  the mod's page, download the file there and use IMPORT).
- Modders: see [SUBMITTING.md](SUBMITTING.md) to get a mod listed.
- Problems with a listed mod: the REPORT button in the game, the [report form](../../issues/new?template=report-mod.yml),
  or write to **melds.melee.4f@icloud.com**.
- What may be listed: [CONTENT_POLICY.md](CONTENT_POLICY.md).

## How the game reads this repository

| File | Read from | Notes |
|---|---|---|
| `index.json` | `https://raw.githubusercontent.com/jkirsteins/carina-mods/main/index.json` | `carina-catalog/1`; each package entry lists the mod's required `dependencies` (`id` and `range`), so BROWSE installs what a mod needs. GitHub serves it with `Cache-Control: max-age=300`, so a change shows within about five minutes |
| `index.json.sig` | next to it | Ed25519 over the exact bytes of `index.json`; the game holds the public keys (edition data) |
| `packages/<id>/<version>.carinamod` | `https://cdn.jsdelivr.net/gh/jkirsteins/carina-mods@<packagesAt>/<path>` | pinned to the commit the signed index names: jsDelivr serves a commit URL as immutable for a year, while a branch URL (`@main`) is cached 12 hours on the CDN and 7 days in the browser, so a new package could 404 for hours; length and sha256 checked against the index before the game's importer opens it |

Packages are files in this repository, never GitHub Release assets (release downloads redirect without CORS
headers and fail in a browser). A package is at most 20,000,000 bytes: jsDelivr does not serve GitHub files
over 20 MB. A bigger mod is listed as an external page.

An index entry is either a package (`path`, `bytes`, `sha256`, `packageHash`) or an external page (`page` and,
when the keeper recorded it, the file's `sha256`; the game's import report then says whether a player's
download matches). Both carry the mod's `id`, `version`, `title`, `summary`, `authors`, `license`,
`capabilities`, `game` ranges (the game lists only mods for its own version), `added` and `tags`. One entry
per mod id; the newest listed version. A removed entry is deleted: players keep their installed copy, and the
game marks it "no longer listed".

## Keeping the catalog

You need Node 24 and the signing key (a file you keep offline; never in any repository).

Once: make the key and give the game its public half.

```sh
npx carina-modkit@^0.2.0 catalog keygen          # writes ~/.config/carina/catalog-signing.key (mode 600)
# prints {"id": "...", "ed25519": "..."}: add it to the catalog block's keys in content/editions/*.json
# (and set the block's contact, which the game refuses to ship as the TODO placeholder once a key is listed)
```

Set the repository variables `CATALOG_KEY_ID` and `CATALOG_PUBLIC_KEY` to the same id and key, so the
`validate` workflow checks the signature on main.

Each mod (minutes):

```sh
npx carina-modkit@^0.2.0 catalog add path/to/mod.carinamod --tag balance            # a package the catalog stores
npx carina-modkit@^0.2.0 catalog add mod.carinamod --page https://someone.itch.io/a-mod   # an external page
git add packages index.json && git commit -m "Add <mod> <version>" && git push
npx carina-modkit@^0.2.0 catalog sign --key ~/.config/carina/catalog-signing.key       # names the pushed commit
git add index.json index.json.sig && git commit -m "Sign the catalog" && git push
```

`catalog add` runs the game's checks and the review screens (no file of another game, no reserved names) and
refuses a mod that fails them; `--allow-term <term>` passes a name hit you reviewed. `catalog sign` refuses
while `packages/` has changes that are not committed, because the index names the commit that holds them.

To remove a mod: delete its entry from `index.json`, sign, push. Leave its file in `packages/`.

Key rotation: add the new public key to the game's edition data and ship that build; sign with the new key;
remove the old key in a later build.

Pull requests: the `validate` workflow runs `catalog check` (the same checks as `catalog add`). Merge only
what you reviewed against CONTENT_POLICY.md, then sign.
