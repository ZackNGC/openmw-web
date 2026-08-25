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
