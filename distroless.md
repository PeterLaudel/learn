# Use distroless Images

Build with distroless. First step is the builder step. The second step is copy the build artifacts into
the distroless image.

```docker
FROM node:20 AS builder

# Disable telemetry
ENV NEXT_TELEMETRY_DISABLED=1

WORKDIR /app

# Building app
COPY --chown=node:node package*.json ./

# Install node modules
# Note: We also install dev deps as TypeScript may be needed
RUN npm install

# Copy files. Use dockerignore to avoid copying node_modules
COPY --chown=node:node . .

# Build
RUN npm run build

# Running the app
FROM gcr.io/distroless/nodejs20 AS runner
WORKDIR /app

# Mark as prod, disable telemetry, set port
ENV NODE_ENV=production
ENV PORT=3000
ENV NEXT_TELEMETRY_DISABLED=1

# Copy from build
COPY --from=builder /app/next.config.mjs ./
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nonroot:nonroot /app/.next ./.next
COPY --from=builder --chown=nonroot:nonroot /app/node_modules ./node_modules


USER nonroot

# Run app command
CMD ["./node_modules/next/dist/bin/next", "start"]
```

## How to test

Testing is necessary. Testing inside the distroless image is not really
doable because of missing package managers. That makes commands like 'npm run ci'
not doable. Run test while building the docker container is also not really useful, because
external dependencies like a database are not reachable from there. The solution is quit easy.

Run build step only for the builder stage:


```sh
docker build -t my-app:builder --target builder .
```

With given docker compose file.

```yml
services:
  app:
    build: .
    image: "${IMAGE:-<image-name>:dev}"
    command: ["npm", "run", "ci"]
```

You can add every dependency you need here. Like postgres for example. The image
environment variable speeds up build time in a CI/CD environment.

Now run docker compose:

```sh
IMAGE=my-app:builder docker compose up --abort-on-container-exit --exit-code-from app
```
