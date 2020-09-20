# vim:set ft=dockerfile:
ARG BASE_IMAGE=node:14.11.0-buster-slim

# ====================
# --- Build Assets ---
# ====================
FROM ${BASE_IMAGE} AS assets

RUN set -ex; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        g++ \
        make \
        python3 \
    ; \
    rm -rf /var/lib/apt/lists/*;

WORKDIR /wiki

COPY ./client ./client
COPY ./dev ./dev
COPY ./package.json ./package.json
COPY ./.babelrc ./.babelrc
COPY ./.eslintignore ./.eslintignore
COPY ./.eslintrc.yml ./.eslintrc.yml

RUN yarn cache clean
RUN yarn --frozen-lockfile --non-interactive
RUN yarn build
RUN rm -rf /wiki/node_modules
RUN yarn --production --frozen-lockfile --non-interactive

# ===============
# --- Release ---
# ===============
FROM ${BASE_IMAGE}

RUN set -ex; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        bash \
        curl \
        git \
        openssh-client \
        ca-certificates \
        gnupg \
    ; \
    rm -rf /var/lib/apt/lists/*;

RUN mkdir -p /wiki && \
    mkdir -p /logs && \
    mkdir -p /wiki/data/content && \
    chown -R node:node /wiki /logs

WORKDIR /wiki

COPY --chown=node:node --from=assets /wiki/assets ./assets
COPY --chown=node:node --from=assets /wiki/node_modules ./node_modules
COPY --chown=node:node ./server ./server
COPY --chown=node:node --from=assets /wiki/server/views ./server/views
COPY --chown=node:node ./dev/build/config.yml ./config.yml
COPY --chown=node:node ./dev/build/sideload ./data/sideload
COPY --chown=node:node ./package.json ./package.json
COPY --chown=node:node ./LICENSE ./LICENSE

USER node

VOLUME ["/wiki/data/content"]

EXPOSE 3000
EXPOSE 3443

HEALTHCHECK --interval=30s --timeout=30s --start-period=30s --retries=3 CMD curl -f http://localhost:3000/healthz

CMD ["node", "server"]
