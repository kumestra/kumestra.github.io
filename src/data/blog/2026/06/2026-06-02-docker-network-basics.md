---
title: Docker Network Basics: Bridges, Names, Ports, and Cleanup
description: A practical guide to how Docker connects containers, how default and user-defined bridge networks differ, and what happens to networks when containers are removed.
pubDatetime: 2026-06-02T10:28:30Z
tags: [docker, networking, containers, devops]
---

Docker networking is easiest to understand if you separate three ideas that often get mixed together:

- how containers talk to each other
- how your host machine talks to containers
- how containers reach the outside internet

Those are related, but they are not the same thing. Most Docker networking confusion comes from expecting one of those paths to automatically imply another.

## Table of contents

## The Short Version

When you start a container, Docker usually gives it its own private network space. Inside that space, the container has its own network interface, IP address, routing table, and ports. Docker then connects that private space to a Docker-managed network on the host.

The default behavior looks roughly like this:

```text
container A
  eth0
    |
    | virtual ethernet pair
    |
Docker bridge on host
    |
host network
    |
internet or local LAN
```

That bridge is why a container can often reach the internet even though it is isolated from the host's normal process space. Docker creates the plumbing, routes traffic through the host, and usually applies NAT so outbound packets can leave through the host's network connection.

For many everyday apps, the most important rule is simple:

Use a user-defined bridge network when containers need to talk to each other by name.

## The Default Bridge

If you run a container without specifying a network, Docker connects it to the default network named `bridge`.

```bash
docker run -d --name db postgres
```

That command attaches the container named `db` to Docker's default `bridge` network. A real Postgres container also needs authentication configuration, such as `POSTGRES_PASSWORD`, to stay running; the network placement is the same either way.

You can see that network with:

```bash
docker network ls
```

And inspect it with:

```bash
docker network inspect bridge
```

Containers on the default bridge can generally reach each other by container IP address, assuming the target service is listening and firewall rules allow the traffic. The catch is name lookup. Another container on the default bridge should not be expected to resolve `db` as a hostname automatically.

That means this kind of mental model is incomplete:

```text
"The container is named db, so other containers can call http://db."
```

Container names are not automatically useful everywhere. Name-based discovery is one of the major reasons to create your own network.

## User-Defined Bridge Networks

A user-defined bridge network is still a bridge network, but it has nicer behavior for multi-container apps.

Create one:

```bash
docker network create app-net
```

Start two containers on it:

```bash
docker run -d --name web --network app-net nginx
docker run --rm -it --name client --network app-net alpine
```

Inside the `client` container, the name `web` can resolve through Docker's embedded DNS:

```bash
wget -qO- http://web
```

That is the big practical difference:

| Setup | Same network? | Can use container name? | Typical use |
| --- | --- | --- | --- |
| Default `bridge` | Yes, if both containers use the default | Usually no automatic name DNS | quick one-off containers |
| User-defined bridge | Yes, if both containers join it | Yes | local apps with multiple services |
| Different networks | No, unless a container joins both | No across isolated networks | separation between app groups |

So these two commands are not equivalent:

```bash
docker run -d --name db postgres
docker run -d --name db --network app-net postgres
```

The first puts `db` on the default bridge. The second puts `db` on `app-net`. If your app container is on `app-net`, it can talk to the second `db` by name, but not the first one.

## Container-to-Container Is Not Host-to-Container

Docker networking has an internal side and an external side.

If two containers are on the same user-defined network, they can usually talk to each other directly:

```text
app container -> db:5432
```

But that does not automatically expose the database to your host laptop or server. For host access, publish a port:

```bash
docker run -d --name web --network app-net -p 8080:80 nginx
```

This maps a host port to a container port:

```text
host port 8080 -> container port 80
```

Now a browser on the host can open:

```text
http://localhost:8080
```

Without `-p 8080:80`, the `web` container can still be reachable by other containers on `app-net`, but not necessarily from the host through `localhost:8080`.

This distinction matters a lot for databases. In development, you might publish Postgres:

```bash
docker run -d \
  --name db \
  --network app-net \
  -e POSTGRES_PASSWORD=example \
  -p 5432:5432 \
  postgres
```

That makes Postgres available to both:

- other containers on `app-net` through `db:5432`
- host tools through `localhost:5432`

In production-like setups, you might skip the published port and allow only internal containers to reach the database.

## Joining a Network Later

If you started a container on the wrong network, you usually do not need to recreate it immediately. You can connect it to another network:

```bash
docker network connect app-net db
```

A container can be attached to more than one Docker network. This is useful for gateway or proxy containers. For example, a reverse proxy might sit on a public-facing network and also join a private app network.

You can disconnect it too:

```bash
docker network disconnect app-net db
```

To see which networks a container uses:

```bash
docker inspect db
```

Or inspect the network:

```bash
docker network inspect app-net
```

## What Happens When Containers Stop or Get Removed

Stopping a container does not delete its network.

```bash
docker stop db
```

The container is stopped, but the network still exists.

Removing the container also does not delete a user-created network:

```bash
docker rm db
```

The container is removed and detached from the network, but the network itself remains.

Remove the network explicitly:

```bash
docker network rm app-net
```

Docker will refuse to remove a network while containers are still attached to it. If you want to clean up unused custom networks:

```bash
docker network prune
```

That removes networks that are not currently used by any containers.

Docker Compose has slightly different lifecycle behavior. If Compose creates a project network, then:

```bash
docker compose down
```

usually removes the Compose-created network along with the containers. If the network is marked as external, Compose will leave it alone.

## Common Network Drivers

Docker has several network drivers. Most local development uses bridge networks, but it helps to know the names:

| Driver | Meaning | When you might use it |
| --- | --- | --- |
| `bridge` | Private network on one Docker host | normal single-machine container apps |
| `host` | Container shares the host network namespace | performance-sensitive or low-level networking cases |
| `none` | Container has no network | locked-down jobs or manual network setup |
| `overlay` | Multi-host container networking | Docker Swarm or clustered setups |
| `macvlan` | Container appears directly on the physical network | advanced LAN integration |

For ordinary development, start with a user-defined bridge network. It gives you isolation, container-name DNS, and simple commands.

## A Practical Mental Model

Here is a small app layout:

```text
              published port
browser -> localhost:8080
              |
              v
        web container
              |
              | private Docker DNS name
              v
        db container

both containers are on app-net
```

The host uses the published port. The containers use Docker's internal network and DNS names.

That gives you two different addresses for two different audiences:

| Audience | Address |
| --- | --- |
| Host machine | `localhost:8080` |
| Another container on `app-net` | `http://web` |

Do not use `localhost` inside one container when you mean another container. Inside a container, `localhost` means that same container, not the host and not a sibling container.

## Commands Worth Remembering

Create a network:

```bash
docker network create app-net
```

Run a container on that network:

```bash
docker run -d --name web --network app-net nginx
```

Publish a port to the host:

```bash
docker run -d --name web --network app-net -p 8080:80 nginx
```

Connect an existing container to a network:

```bash
docker network connect app-net db
```

List networks:

```bash
docker network ls
```

Inspect a network:

```bash
docker network inspect app-net
```

Remove a network:

```bash
docker network rm app-net
```

Remove unused networks:

```bash
docker network prune
```

## The Rule of Thumb

If you are running one throwaway container, the default bridge is fine.

If you are running more than one container for the same app, create a user-defined bridge network and put the containers on it.

That one habit avoids a surprising amount of Docker networking confusion. Your containers get a shared private network, predictable DNS names, and a clean lifecycle you can inspect and manage explicitly.
