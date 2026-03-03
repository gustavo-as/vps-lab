# 01 — Provisionamento da VPS na Hostinger

## Plano escolhido

| Item | Escolha |
|---|---|
| Provedor | Hostinger |
| Plano | KVM 2 |
| vCPU | 2 |
| RAM | 8 GB |
| Localização | França (menor latência para a Bélgica) |
| Sistema Operacional | Ubuntu 22.04 LTS (sem aplicativos pré-instalados) |
| Backup diário | Não contratado (usar snapshots manuais) |

### Por que KVM 2?

- Suporta 4 a 6 containers rodando simultaneamente
- Memória suficiente para builds Java (Maven/Gradle)
- Suporta Traefik + GitHub Actions sem travar
- Melhor custo-benefício para início

> ⚠️ Evitar o plano KVM 1 (4GB RAM) — trava durante builds de projetos Java.

---

## Configuração de DNS

Após provisionar a VPS, configurar os seguintes registros DNS no painel da Hostinger:

| Tipo | Nome/Host | Valor |
|---|---|---|
| A | `@` | `IP_DA_VPS` |
| A | `www` | `IP_DA_VPS` |
| A | `*` | `IP_DA_VPS` |

O registro wildcard (`*`) garante que qualquer subdomínio criado futuramente aponte automaticamente para a VPS, sem precisar criar novos registros DNS.

> ℹ️ A propagação de DNS pode levar de alguns minutos até 24 horas.