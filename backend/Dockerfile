FROM node:20-slim AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

# 1. Install Dependencies with "Cheats"
FROM base AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml ./

# CRITICAL FIX 1: Create a temporary .npmrc that forces flat modules
# This makes transitive deps like @medusajs/utils visible to your config
RUN echo "node-linker=hoisted" > .npmrc && \
    echo "shamefully-hoist=true" >> .npmrc

RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile

# 2. Build
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
COPY --from=deps /app/.npmrc ./

# CRITICAL FIX 2: Help Node resolve ESM imports
# This fixes "Did you mean .../index.js?"
ENV NODE_OPTIONS="--experimental-specifier-resolution=node"

RUN pnpm build

# 3. Production Runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

# Copy necessary files for runtime
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/.medusa ./.medusa
COPY --from=builder /app/medusa-config.ts ./
COPY --from=builder /app/package.json ./
COPY --from=builder /app/src ./src

# Runtime Env Vars
ENV PORT=9000
ENV NODE_OPTIONS="--experimental-specifier-resolution=node"

EXPOSE 9000
CMD ["pnpm", "start"]
