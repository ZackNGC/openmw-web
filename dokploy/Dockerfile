# Dokploy deployment image for openmw-web.
#
# Uses the prebuilt engine from the upstream GitHub release (the engine cannot be
# rebuilt here: the source build needs the maintainer's prebaked openmw-builder
# image and the cross-compiled dependency stack, neither of which is published).
# The release zip is self-contained: content-versioned engine dir (e/<hash>/),
# launcher.html, index.html, moddata/, and the ready-to-run server.py which
# implements the full serving contract (COOP/COEP isolation headers, HTTP Range,
# brotli siblings, the mwdata manifest, and the launcher flag).
#
# Game data is NOT baked in — mount your own legally-owned Data Files (plus any
# mods such as Tamriel Rebuilt) at /app/mwdata. See dokploy/README.md.
FROM python:3.12-slim

# Bump this when upstream publishes a new release.
ARG OPENMW_WEB_VERSION=v1.1.0

ADD https://github.com/Virtastic/openmw-web/releases/download/${OPENMW_WEB_VERSION}/openmw-web-${OPENMW_WEB_VERSION}.zip /tmp/openmw-web.zip
RUN python -m zipfile -e /tmp/openmw-web.zip /app && rm /tmp/openmw-web.zip

# Overlay this branch's patched game page (server-curated load order support) onto the
# release. The release build stamps __ENGINE_VERSION__ with the content hash of the
# engine dir (e/<hash>/) and rewrites the streamfs.js script src to the versioned path;
# reproduce both here so the patched page addresses the same engine files. Any stale
# precompressed sibling must go, or server.py would serve the unpatched .br instead.
COPY play/index.html /tmp/index.html.in
RUN set -e; \
    ENGVER=$(ls /app/e); \
    sed -e "s/__ENGINE_VERSION__/${ENGVER}/g" \
        -e "s|src=\"streamfs.js\"|src=\"e/${ENGVER}/streamfs.js\"|" \
        /tmp/index.html.in > /app/index.html; \
    rm -f /app/index.html.br /tmp/index.html.in; \
    grep -q "e/${ENGVER}/streamfs.js" /app/index.html; \
    grep -q "__momwCfg" /app/index.html

WORKDIR /app

# OPENMW_HOST: bind all interfaces (container). OPENMW_OPEN: don't try to open a browser.
# OPENMW_LAUNCHER: serve the data chooser at /; with mwdata mounted it offers
# "This server's copy" so players boot straight into the staged game data.
ENV OPENMW_HOST=0.0.0.0 \
    OPENMW_OPEN=0 \
    OPENMW_LAUNCHER=1 \
    PORT=8910

EXPOSE 8910

CMD ["python3", "server.py"]
