FROM node:22-slim AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

# 1. Install Dependencies
FROM base AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml ./

# Force pnpm to hoist dependencies so transitive deps are visible
RUN echo "node-linker=hoisted" > .npmrc && \
    echo "shamefully-hoist=true" >> .npmrc

RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile

# 2. Build
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
COPY --from=deps /app/.npmrc ./

# FIX: Inline the env var to ensure it applies to the build process
RUN NODE_OPTIONS="--experimental-specifier-resolution=node" pnpm build

# 3. Production Runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

# Copy necessary artifacts
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/.medusa ./.medusa
# Wildcard copy to handle .js or .ts config
COPY --from=builder /app/medusa-config.* ./
COPY --from=builder /app/tsconfig.json ./
COPY --from=builder /app/package.json ./

# Runtime Env Vars
ENV PORT=9000
ENV NODE_OPTIONS="--experimental-specifier-resolution=node"

EXPOSE 9000
CMD ["pnpm", "start"]
