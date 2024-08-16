# Build static files
# Node Bullseye has npm
FROM node:20.16.0-bullseye-slim AS buildWithDevDeps

# Install packages, build and keep only prod packages
WORKDIR /app
COPY *.json ./
COPY ./src ./src
COPY ./prisma ./prisma
RUN npm ci --ignore-scripts --no-update-notifier
RUN npm run prisma-generate
FROM node:20.16.0-bullseye-slim AS build

# Image defaults to /project; copy controlled by .dockerignore
COPY --chown=quarkus:quarkus . ./
RUN chmod +x ./mvnw && ./mvnw package -Pnative -DskipTests

# "A base image to run Quarkus native application using UBI Micro"
FROM quay.io/quarkus/quarkus-micro-image:2.0

# Port and health check
EXPOSE 3000
HEALTHCHECK --interval=300s --timeout=3s CMD curl -f http://localhost:3000/ || exit 1

# Startup
COPY --chown=1001:root --from=build /project/target/*-runner /app
USER 1001
CMD ["/app", "-Dquarkus.http.host=0.0.0.0"]
