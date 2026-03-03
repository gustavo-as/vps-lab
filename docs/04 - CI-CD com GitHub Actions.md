# 04 — CI/CD com GitHub Actions

## Visão geral

A cada `git push` na branch `main`, o GitHub Actions automaticamente:
1. Faz build da imagem Docker do projeto
2. Faz push da imagem para o GitHub Container Registry (ghcr.io)
3. Conecta na VPS via SSH e faz o deploy do novo container

---

## 1. Gerar chave SSH exclusiva para o GitHub Actions

Na VPS, executar:

```bash
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/github_actions -N ""
```

Adicionar a chave pública às autorizadas:

```bash
cat ~/.ssh/github_actions.pub >> ~/.ssh/authorized_keys
```

> ⚠️ Nunca commitar a chave privada no repositório. Ela deve ser guardada apenas como secret no GitHub.

---

## 2. Configurar secrets no GitHub

Acesse: **Repositório → Settings → Secrets and variables → Actions → New repository secret**

| Secret | Valor |
|---|---|
| `VPS_SSH_KEY` | Conteúdo completo da chave privada `~/.ssh/github_actions` |
| `VPS_HOST` | IP público da VPS |
| `VPS_USER` | `username` |

---

## 3. Estrutura do workflow

Cada projeto terá um arquivo `.github/workflows/deploy.yml` com a seguinte estrutura:

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd ~/projects/NOME_DO_PROJETO
            docker compose pull
            docker compose up -d --force-recreate
```

---

## 4. Fluxo completo por projeto

```
git push origin main
        │
        ▼
GitHub Actions triggered
        │
        ▼
SSH na VPS
        │
        ▼
docker compose up -d --force-recreate
        │
        ▼
Traefik detecta o novo container automaticamente
        │
        ▼
Projeto disponível em subdominio.gustavohub.com
```