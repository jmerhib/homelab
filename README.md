# bee01 homelab stack

[![ci](https://github.com/jmerhib/homelab/actions/workflows/ci.yml/badge.svg)](https://github.com/jmerhib/homelab/actions/workflows/ci.yml)

Self-hosted media and local-AI stack on a single headless mini PC (Ubuntu, Radeon 780M iGPU), defined as one Docker Compose file.

```
            LAN clients
   ┌───────────┼─────────────┐
 Plex :32400  Open WebUI :3000  Ollama API :11434
 (host net,   (chat UI) ──────▶ (ROCm, 780M GPU,
  VAAPI)                         qwen3:14b @ 32k ctx)
   │
 /srv/media (own NVMe)         volumes on /var/lib/docker (own partition)
```

| Service | Purpose | Notes |
|---|---|---|
| Plex | Media server | `/dev/dri` passthrough for HW transcoding; in-RAM transcode dir |
| Ollama (ROCm) | Local LLM API | iGPU needs `HSA_OVERRIDE_GFX_VERSION=11.0.2` + `OLLAMA_IGPU_ENABLE=1`; q8 KV cache fits a 14B model at 32k context in ~13 GB |
| Open WebUI | Chat front end | Talks to Ollama over the compose network |

## Run

```bash
cp .env.example .env   # fill in IDs
docker compose up -d
```

## CI

GitHub Actions on every push/PR: `yamllint`, `docker compose config` validation, a `gitleaks` secret scan, and a `trivy` scan of every image in the compose file (fails on fixable CRITICAL CVEs). Secrets stay in the untracked `.env`.

## Roadmap

Monitoring (Prometheus/Grafana) → Ansible provisioning → tested restic backups → k3s + GitOps.
