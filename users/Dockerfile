# Stage 1: Install dependencies
FROM node:16-alpine as deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Stage 2: Build the source code
FROM deps as build
WORKDIR /app
COPY . .
RUN npm run build

# Stage 3: Run the app
FROM node:16-alpine as run
WORKDIR /app
ENV NODE_ENV=production

# Install curl to check if the server is up
RUN apk add --no-cache curl

# Create a new user and group to run the app
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nodeuser --ingroup nodejs

# Copy the built files from the previous stage
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/package.json ./
COPY --from=build /app/package-lock.json ./
RUN npm install --omit=dev

# Change the ownership of the app directory to the new user
RUN chown -R nodeuser:nodejs /app
USER nodeuser

EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 CMD curl --fail http://localhost:3000 || exit 1

ENV PORT=3000
CMD ["npm", "start"]