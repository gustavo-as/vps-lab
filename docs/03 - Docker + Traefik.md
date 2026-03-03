# 03 — Docker + Traefik

## Instalação do Docker

### 1. Instalar dependências

```bash
sudo apt install -y ca-certificates curl gnupg
```

### 2. Adicionar repositório oficial do Docker

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
```

### 3. Instalar Docker Engine

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 4. Adicionar usuário ao grupo docker

```bash
sudo usermod -aG docker username
newgrp docker
```

### 5. Verificar instalação

```bash
docker run hello-world
docker compose version
```

### 6. Habilitar inicialização automática

```bash
sudo systemctl enable docker
```

---

## Configuração do Traefik

O Traefik atua como reverse proxy, roteando o tráfego por subdomínio para o container correto e gerenciando certificados SSL via Let's Encrypt automaticamente.

### 1. Criar estrutura de pastas

```bash
mkdir -p ~/traefik && cd ~/traefik
touch acme.json && chmod 600 acme.json
```

> ⚠️ O `chmod 600` no `acme.json` é obrigatório — o Traefik recusa iniciar se as permissões estiverem incorretas.

### 2. Criar rede Docker compartilhada

```bash
docker network create traefik-network
```

Todos os containers de projetos devem se conectar a essa rede.

### 3. Gerar senha para o dashboard

```bash
sudo apt install -y apache2-utils
htpasswd -nbB usuario 'suasenha'
```

O hash gerado será usado no `docker-compose.yml`. Atenção: cada `$` no hash deve ser dobrado (`$$`) dentro do arquivo docker-compose.

### 4. Criar `~/traefik/docker-compose.yml`

```yaml
services:
  traefik:
    image: traefik:v3
    container_name: traefik
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    networks:
      - traefik-network
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./acme.json:/acme.json
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.web.http.redirections.entrypoint.to=websecure"
      - "--entrypoints.web.http.redirections.entrypoint.scheme=https"
      - "--certificatesresolvers.letsencrypt.acme.email=SEU_EMAIL@gmail.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/acme.json"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`traefik.gustavohub.com`)"
      - "traefik.http.routers.dashboard.service=api@internal"
      - "traefik.http.routers.dashboard.entrypoints=websecure"
      - "traefik.http.routers.dashboard.tls.certresolver=letsencrypt"
      - "traefik.http.routers.dashboard.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=username:$$HASH_GERADO_AQUI"

networks:
  traefik-network:
    external: true
```

### 5. Subir o Traefik

```bash
cd ~/traefik && docker compose up -d
```

> ⚠️ **Atenção:** Usar `traefik:v3.0` (versão inicial) causa erro de incompatibilidade com versões recentes do Docker:
> `client version 1.24 is too old. Minimum supported API version is 1.44`
> Sempre usar `traefik:v3` (latest v3) para garantir compatibilidade.

### 6. Verificar

```bash
docker ps
```

O container `traefik` deve aparecer com status `Up` e portas `0.0.0.0:80->80/tcp` e `0.0.0.0:443->443/tcp`.

---

## Como adicionar um novo projeto ao Traefik

Qualquer novo container precisa de:

1. Estar conectado à `traefik-network`
2. Ter as labels corretas no `docker-compose.yml`:

```yaml
networks:
  - traefik-network

labels:
  - "traefik.enable=true"
  - "traefik.http.routers.NOME.rule=Host(`subdominio.gustavohub.com`)"
  - "traefik.http.routers.NOME.entrypoints=websecure"
  - "traefik.http.routers.NOME.tls.certresolver=letsencrypt"
  - "traefik.http.services.NOME.loadbalancer.server.port=PORTA_INTERNA"
```

O Traefik detecta o container automaticamente e configura o roteamento e SSL sem necessidade de reiniciar.