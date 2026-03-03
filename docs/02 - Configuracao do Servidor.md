# 02 — Configuração Inicial do Servidor

## Acesso inicial

```bash
ssh root@IP_DA_VPS
```

---

## 1. Atualizar o sistema

```bash
apt update && apt upgrade -y
```

---

## 2. Criar usuário admin

Nunca trabalhar diretamente com o usuário `root`.

```bash
adduser username
usermod -aG sudo username
```

---

## 3. Configurar autenticação SSH por chave

No computador local, gerar o par de chaves:

```bash
ssh-keygen -t ed25519 -C "username@vps"
```

Copiar a chave pública para o servidor:

```bash
ssh-copy-id username@IP_DA_VPS
```

Testar o acesso com a nova chave:

```bash
ssh username@IP_DA_VPS
```

---

## 4. Desabilitar login root por SSH

```bash
sudo nano /etc/ssh/sshd_config
```

Alterar a linha:

```
PermitRootLogin yes
```

Para:

```
PermitRootLogin no
```

Reiniciar o serviço SSH:

```bash
sudo systemctl restart ssh
```

---

## 5. Configurar o Firewall (UFW)

Liberar apenas as portas necessárias:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

Verificar status:

```bash
sudo ufw status
```

Resultado esperado:

```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

---

## Checklist de segurança

- [x] Sistema atualizado
- [x] Usuário admin criado (sem uso do root)
- [x] SSH por chave configurado
- [x] Login root por SSH desabilitado
- [x] Firewall ativo (portas 22, 80, 443 apenas)