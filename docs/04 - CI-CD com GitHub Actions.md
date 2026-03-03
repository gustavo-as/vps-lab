# 04 — CI/CD com GitHub Actions

## Visão geral

A cada `git push` na branch `main`, o GitHub Actions automaticamente:
1. Conecta na VPS via SSH
2. Faz `git pull` do repositório
3. Faz build da imagem Docker e recria o container

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
name: Deploy to VPS

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
            git pull origin main
            docker compose up -d --build --force-recreate
```

> ℹ️ O arquivo `.github/workflows/deploy.yml` deve estar dentro da pasta `.github/workflows/` na raiz do repositório. A pasta `.github` é oculta no Windows/Mac — ativar "Mostrar itens ocultos" no Explorer para visualizá-la.

> ⚠️ Os secrets `VPS_SSH_KEY`, `VPS_HOST` e `VPS_USER` precisam ser configurados em **cada repositório** separadamente em: **Settings → Secrets and variables → Actions → New repository secret**

---

## 4. Preparar a VPS para um novo projeto

Antes do primeiro deploy, criar a pasta do projeto na VPS e clonar o repositório:

```bash
mkdir -p ~/projects/NOME_DO_PROJETO
cd ~/projects/NOME_DO_PROJETO
git clone https://github.com/gustavo-as/NOME_DO_PROJETO.git .
```

A partir daí, todo `git push` na branch `main` faz o deploy automaticamente via GitHub Actions.

---

## 5. Fluxo completo por projeto

```
git push origin main
        │
        ▼
GitHub Actions triggered
        │
        ▼
SSH na VPS → git pull → docker compose up --build
        │
        ▼
Traefik detecta o novo container automaticamente
        │
        ▼
Projeto disponível em subdominio.gustavohub.com (com SSL)
```