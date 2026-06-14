# Guild Ranking — Pipeline Runner

> Scheduled execution layer for the [guildranking.io](https://guildranking.io) ranking pipeline.

![Java](https://img.shields.io/badge/Java-25-007396)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4-6DB33F)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Storage-336791)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED)
![GitHub Actions](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF)

## Overview

This repository has a single responsibility: **run the Guild Ranking pipeline on a schedule.**

It contains no application source of its own. The pipeline ships as a self-contained, versioned Docker image, and everything in this repository exists to pull that image and execute it as a batch job inside a clean, ephemeral environment. Each run starts fresh, completes its work, and tears itself down — leaving no long-lived process to maintain.

The pipeline powers the live rankings served at **[guildranking.io](https://guildranking.io)**.

## What the pipeline does

On every run, the containerized job:

- **Ingests** World of Warcraft guild data from authoritative third-party sources, including Raider.IO and Warcraft Logs.
- **Consolidates** raid progression, roster, and performance signals into a single coherent dataset per guild.
- **Ranks** guilds deterministically at both language and world scope.
- **Persists** the finished ranked dataset to PostgreSQL, ready for the public site to serve.

It is a true batch job — it boots, does the work, and exits. There is no idle service and no exposed endpoint.

## How it runs

```
cron-job.org ──(repository_dispatch)──▶ GitHub Actions ──(pull & run image)──▶ Pipeline container ──▶ PostgreSQL
```

1. An external scheduler (**cron-job.org**) issues a `repository_dispatch` event to this repository through the GitHub API on a fixed cadence.
2. The dispatch triggers the GitHub Actions workflow.
3. The workflow authenticates to the private container registry, pulls the current pipeline image, and runs it.
4. Configuration and credentials are injected at runtime from GitHub Actions Secrets — nothing sensitive is committed to the repository.
5. The container performs a full ranking run and exits; the runner is disposed of automatically.

## Tech stack

- **Java 25 · Spring Boot** — the batch runner that drives ingestion, ranking, and persistence
- **PostgreSQL** — system of record for the published rankings
- **Docker** — packaging and runtime isolation
- **GitHub Actions** — scheduled orchestration and execution

## Operations

- **Cadence** is controlled entirely by the external scheduler; adjusting the schedule requires no change to this repository.
- **Manual runs** can be started from the **Actions** tab when on-demand execution is enabled.
- **Logs** for every run are available in the corresponding GitHub Actions run.

