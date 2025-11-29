FROM node:20-slim AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

# 1. Install Deps
FROM base AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
COPY .npmrc ./
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile

# 2. Build
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
COPY --from=deps /app/.npmrc ./

# DEBUG: List files to verify context before build
RUN ls -la /app

ENV NODE_OPTIONS="--experimental-specifier-resolution=node"
RUN pnpm build

# DEBUG: List files AFTER build to see what artifacts exist
RUN ls -la /app && ls -la /app/.medusa

# 3. Production Runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

# Copy node_modules (Critical)
COPY --from=builder /app/node_modules ./node_modules

# Copy built artifacts (Critical)
COPY --from=builder /app/.medusa ./.medusa

# Copy config (Wildcard to be safe)
COPY --from=builder /app/medusa-config.* ./
COPY --from=builder /app/package.json ./

# Copy src ONLY if it exists (using a shell hack because COPY fails if missing)
# We'll skip explicitly copying src for a moment to see if the build passes without it
# Medusa 2.0 production builds usually run from .medusa/server, so src might not be strictly needed if compiled correctly
# If you absolutely need src, we can add it back later.

# Runtime Env
ENV PORT=9000
ENV NODE_OPTIONS="--experimental-specifier-resolution=node"

EXPOSE 9000
CMD ["pnpm", "start"]
