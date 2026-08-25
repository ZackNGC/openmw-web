# Deploying openmw-web on Dokploy

This directory deploys the **prebuilt release** of openmw-web behind Dokploy's
Traefik proxy, serving your own game data (base game + Tamriel Rebuilt + texture
packs) from a host directory.

## One-time setup

1. **Stage game data on the VPS** (never commit it anywhere):

   ```
   /opt/morrowind/mwdata/
   ├── Morrowind.esm / Morrowind.bsa
   ├── Tribunal.esm / Tribunal.bsa
   ├── Bloodmoon.esm / Bloodmoon.bsa
   ├── Fonts/ Music/ Sound/ Splash/ Video/
   ├── Tamriel_Data.esm + PT_Data.bsa / TR_Data.bsa (or loose files)
   ├── TR_Mainland.esm, TR_Factions.esp
   └── ...any other mods (.esm/.esp/.bsa, loose Textures/ etc.)
   ```

   Load order is derived automatically: official masters first, then mod `.esm`
   and `.esp` alphabetically — which is correct for Tamriel Rebuilt
   (`Tamriel_Data.esm` sorts before `TR_Mainland.esm`). `?nomods=1` boots vanilla.

2. **Create the Dokploy service**: Compose type, Git provider → this repo,
   branch `dokploy`, compose path `dokploy/docker-compose.yml`.

3. **Attach a domain** to service `morrowind`, container port `8910`, HTTPS on
   (Let's Encrypt). HTTPS is not optional: the engine is multi-threaded WASM and
   needs cross-origin isolation, which browsers only grant on secure origins.
   The isolation headers themselves (COOP/COEP/CORP) are sent by `server.py`.

4. Deploy. Open the domain in **desktop Chrome/Chromium** and pick
   "This server's copy".

## Updating the engine

Bump `OPENMW_WEB_VERSION` in `dokploy/Dockerfile` when upstream publishes a new
release, then redeploy. Front-end commits on `dev` between releases cannot be
deployed alone — the HTML/JS must match the engine build it shipped with.
