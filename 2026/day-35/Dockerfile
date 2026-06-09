
# Stage 1: Building the app (install dependencies, compile)
## Its only job is to produce the .next/ folder. That's it. Everything it does is in service of that one output:

# -> Sets up the full Node.js environment
# -> Installs every dependency including dev deps
# -> Receives all environment variables
# -> Runs npm run build to compile the app
# -> Dies — never ships to production

# Think of it as a construction site. Cranes, scaffolding, cement mixers, workers — all present during construction, none of it is part of the finished building.



FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ARG NEXTAUTH_SECRET
ARG NEXTAUTH_URL
ARG NEXT_PUBLIC_ENCRYPTION_KEY
ARG ENCRYPTION_SECRET

ENV NEXTAUTH_SECRET=$NEXTAUTH_SECRET
ENV NEXTAUTH_URL=$NEXTAUTH_URL
ENV NEXT_PUBLIC_ENCRYPTION_KEY=$NEXT_PUBLIC_ENCRYPTION_KEY
ENV ENCRYPTION_SECRET=$ENCRYPTION_SECRET
ENV NODE_OPTIONS="--max-old-space-size=512"

RUN npm run build





# Stage 2: Copy only the built artifact into a minimal base image (`alpine`, `distroless`, or `scratch`)

## Its only job is to serve the already-built app:

# -> Starts completely fresh with a minimal Alpine image
# -> Copies only the built output (.next/) from stage 1
# -> Installs only production dependencies
# -> Runs npm start to serve the app

# No build tools, no source code, no dev dependencies, no compiler — just the finished product running.


FROM node:22-alpine AS runner

WORKDIR /app

COPY --from=builder /app/package*.json ./

RUN npm install

COPY --from=builder /app/.next ./.next

COPY --from=builder /app/public ./public

EXPOSE 3000

CMD ["npm" , "start"]
