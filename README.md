# 🚀 gustavohub.com — VPS Lab & Portfolio

Repositório de documentação e configuração da minha infraestrutura pessoal hospedada em uma VPS na Hostinger.

## Objetivo

- Hospedar minha **landing page** como desenvolvedor full stack
- Disponibilizar **projetos de portfólio** (Java, Angular, React, Node.js) para testes
- Usar como **laboratório de estudos** de DevOps: CI/CD, containers, orquestração

## Stack de Infraestrutura

| Tecnologia | Função |
|---|---|
| Ubuntu 22.04 LTS | Sistema operacional do servidor |
| Docker + Docker Compose | Containerização dos serviços |
| Traefik v3 | Reverse proxy + SSL automático (Let's Encrypt) |
| GitHub Actions | CI/CD — deploy automático via `git push` |

## Arquitetura

```
Internet
    │
    ▼
Traefik (porta 80/443)
    │
    ├── gustavohub.com        → Landing Page
    ├── traefik.gustavohub.com → Dashboard Traefik
    ├── api.gustavohub.com    → Backend Node.js
    ├── app.gustavohub.com    → Frontend Angular/React
    └── *.gustavohub.com      → Outros projetos
```

Todos os serviços rodam em containers Docker conectados à `traefik-network`. O Traefik roteia o tráfego por subdomínio e gerencia os certificados SSL automaticamente.

## Documentação

1. [Provisionamento da VPS](docs/01-hostinger-vps.md)
2. [Configuração inicial do servidor](docs/02-server-setup.md)
3. [Docker + Traefik](docs/03-docker-traefik.md)
4. [CI/CD com GitHub Actions](docs/04-cicd.md)
5. [Landing Page](docs/05-landing-page.md)
6. [Projetos de portfólio](docs/06-projects.md) *(em breve)*

## Servidor

- **Provedor:** Hostinger
- **Plano:** KVM 2 (2 vCPU / 8GB RAM)
- **Localização:** França
- **OS:** Ubuntu 22.04 LTS
- **Domínio:** gustavohub.com