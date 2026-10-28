# Getting a mod listed

1. Make and test your mod with the mod template (`npx carina-modkit check`, `run`, `pack`). The catalog runs
   the same checks the game runs on import.
2. Make sure it follows [CONTENT_POLICY.md](CONTENT_POLICY.md): your own work (or work whose licence allows it),
   nothing taken from other games, a licence in mod.json.
3. Then either
   - open a **Submit a mod** issue with a link to your `.carinamod` (or your mod's page) and the sha256 that
     `pack` printed, or
   - open a pull request that runs `npx carina-modkit@^0.2.0 catalog add <your file>` on this repository (add
     `--page <url>` if you host the file yourself). The `validate` check must pass.

The keeper reviews the mod, merges it and signs the catalog; the game shows it within minutes.

Packages the catalog stores are at most 20 MB. Bigger mods, and mods you prefer to host yourself (for example on
itch.io), are listed as external pages: players open your page from the game, download the file there and
import it; give the sha256 so the game can tell them whether their download is the listed file.

Updates: pack a new version (a listed file never changes) and submit it the same way. Players see an update
notice in the game.
