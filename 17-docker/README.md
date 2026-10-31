# 17 — Docker

"It works on my machine" usually means the other machine has a different JDK, different config, or a missing dependency. Docker packages your app together with everything it needs to run, so the same thing runs on your laptop, in CI, and on a server. This section containerizes the Order service from [16 — Spring](../16-spring/README.md).

## Images and containers

- **Image** — a read-only package containing your app, a JDK/JRE, and a slimmed-down Linux filesystem. Built once, then shared through a **registry** (Docker Hub, or a private one).
- **Container** — a running instance of an image, with its own filesystem, processes, and network. You can start many containers from one image.

The relationship is a lot like class and object from [03](../03-oop-fundamentals/README.md): the image is the definition, and containers are the instances. A container is not a full virtual machine — it shares the host's kernel, which is why it starts in seconds rather than minutes.

Anything written inside a container disappears when the container is removed. Treat containers as disposable, and keep data you care about in a database or a volume (see below).

## Install

- **macOS / Windows**: [Docker Desktop](https://www.docker.com/products/docker-desktop/) (it has licensing terms for larger companies, so check with your team what TapRoot uses)
- **Linux**: [Docker Engine](https://docs.docker.com/engine/install/) from your distro's instructions

Verify the install:

```
docker --version
docker run --rm hello-world
```

## Writing a Dockerfile

A `Dockerfile` is the recipe for an image. Put this in the project root, next to `pom.xml`:

```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

- `FROM` — the base image to build on, here a Java 21 runtime matching [00 — Setup](../00-setup/README.md).
- `WORKDIR` — the directory the following instructions run in (created if missing).
- `COPY` — copies files from your machine into the image.
- `EXPOSE` — documents the port the app listens on. It doesn't publish the port; `-p` does that at run time.
- `ENTRYPOINT` — the command that runs when a container starts.

This Dockerfile copies a jar that already exists, so build it first (section [13](../13-build-tools/README.md)):

```
mvn package
docker build -t order-service .
```

`-t` names the image, and `.` is the build context: the folder Docker can copy files from.

Add a `.dockerignore` next to the Dockerfile so that local files stay out of the build context. It works like `.gitignore`:

```
.git
.idea
*.iml
```

### Multi-stage builds

The Dockerfile above depends on someone having run `mvn package` first. A multi-stage build compiles inside Docker instead, so the only thing a machine needs is Docker itself:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /src
COPY pom.xml .
RUN mvn -q dependency:go-offline
COPY src ./src
RUN mvn -q package -DskipTests

FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /src/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Only the last stage ends up in the final image, so Maven and your source code aren't shipped with it.

Each instruction creates a cached layer, and a layer is rebuilt only if it or something before it changed. That's why `pom.xml` is copied before `src`: changing your code doesn't re-download every dependency.

## Running a container

```
docker run --rm -p 8080:8080 -e SPRING_PROFILES_ACTIVE=dev order-service
```

- `-p 8080:8080` — publishes a port as `host:container`. Without it, nothing on your machine can reach the app. `-p 9090:8080` would serve it on your port 9090.
- `-e` — sets an environment variable. Spring Boot reads `SPRING_PROFILES_ACTIVE` as `spring.profiles.active`, so the profiles from section 16 work without rebuilding the image.
- `--rm` — removes the container when it exits.

Build one image and change its behavior through environment variables. The same image then moves through the environments in [14](../14-environment-setup/README.md), and what you tested in UAT is exactly what reaches PROD.

Never put passwords or API keys in a Dockerfile or `COPY` them into an image. Anyone who can pull the image can read them, even if a later instruction deletes the file.

## Commands you'll actually use

```
docker ps                      # running containers (-a includes stopped ones)
docker logs -f <container>     # follow a container's logs
docker exec -it <container> sh # open a shell inside a running container
docker stop <container>        # stop a container
docker rm <container>          # remove a stopped container
docker images                  # list local images
docker rmi <image>             # remove an image
docker system prune            # clean up stopped containers and unused data
```

`docker logs` and `docker exec` are the container versions of the `tail -f` and shell habits from section 14 — they're the first things to reach for when a container misbehaves.

## Networking

Inside a container, `localhost` means that container, not your machine. This is the most common beginner mistake with Docker: a `spring.datasource.url` pointing at `localhost:5432` works when you run the app in your IDE, then fails with a connection error once the app is in a container.

Containers on the same Docker network reach each other by name, which is what Compose sets up for you next.

## Docker Compose

Compose defines several containers in one `compose.yaml` and starts them together — useful for running your app with a real database locally:

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/orders
      SPRING_DATASOURCE_USERNAME: orders
      SPRING_DATASOURCE_PASSWORD: local-only
      SPRING_JPA_HIBERNATE_DDL_AUTO: update
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: orders
      POSTGRES_PASSWORD: local-only
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U orders -d orders"]
      interval: 5s
      timeout: 3s
      retries: 10
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

- The host in the JDBC URL is `db`, the service name, not `localhost`.
- `condition: service_healthy` waits for Postgres to accept connections before starting the app. A plain `depends_on` only waits for the container to start.
- `db-data` is a **volume**: storage that lives outside the container, so your rows survive the database container being recreated.
- The password here is fine because this file is for local development only. Real credentials come from the environment, never from a committed file.

```
docker compose up --build    # build and start everything (add -d to run in the background)
docker compose logs -f app   # follow one service's logs
docker compose down          # stop and remove the containers, keeping volumes
docker compose down -v       # also delete volumes, wiping the database
```

Which registry, base images, and deploy tooling TapRoot uses is worth confirming with your team — the concepts above carry over regardless.

## Exercise: containerize the Order service

Using the Order service from section 16:

1. Write a multi-stage `Dockerfile` and a `.dockerignore`, then build the image.
2. Run it with the `dev` profile and repeat the `POST` and `GET` round trip with `curl` against the container.
3. Run `docker logs` and `docker exec` against the running container, and confirm you can find the request in the logs or the jar in `/app`.
4. Add the `compose.yaml` above and the PostgreSQL driver as a runtime dependency (`org.postgresql:postgresql`). `ORDER` is a reserved word in SQL, so add `@Table(name = "orders")` to the entity.
5. Create an order, run `docker compose down`, then `docker compose up`, and confirm the order is still there. Then try `docker compose down -v` and explain why it's gone.

## Checklist

- [ ] Can explain the difference between an image and a container
- [ ] Can write a Dockerfile and explain what each instruction does
- [ ] Know why a multi-stage build needs nothing but Docker on the build machine
- [ ] Comfortable using `docker ps`, `docker logs`, and `docker exec` to inspect a running container
- [ ] Can explain why `localhost` inside a container doesn't reach your machine or another container
- [ ] Know why secrets belong in the environment rather than in an image
- [ ] Order service runs under Compose, and data survives `docker compose down`
