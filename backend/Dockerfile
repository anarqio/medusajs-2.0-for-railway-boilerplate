FROM node:20-slim AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

# 1. Install Dependencies
FROM base AS deps
WORKDIR /app
# Since Base Dir is /backend, we copy directly from current context
COPY package.json pnpm-lock.yaml ./
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile

# 2. Build
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
# Copy all backend source files
COPY . .
# Builds the application to .medusa/server
RUN pnpm build

# 3. Production Runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

# Copy production dependencies
COPY --from=builder /app/node_modules ./node_modules

# Copy the compiled application (.medusa folder is crucial for Medusa 2.0)
COPY --from=builder /app/.medusa ./.medusa

# Copy configuration and source files
# We copy src and tsconfig to ensure runtime path aliases (like 'lib/') resolve correctly
COPY --from=builder /app/medusa-config.ts ./
COPY --from=builder /app/tsconfig.json ./
COPY --from=builder /app/package.json ./
COPY --from=builder /app/src ./src

EXPOSE 9000

CMD ["pnpm", "start"]
