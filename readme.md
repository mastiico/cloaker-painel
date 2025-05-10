# 📄 COMO USAR O CLOAKER LINUX

Este documento explica como configurar e rodar o **Cloaker Linux** em sua VPS, com proxy reverso via NGINX e os arquivos HTML armazenados em `src/pages/black`.

---

## 🔧 PRÉ-REQUISITOS

- VPS com Linux x64 (Ubuntu/Debian/CentOS)
- Acesso root
- NGINX instalado
- Porta **3005** liberada no firewall
- Node.js **NÃO** é necessário (binário standalone)

---

## 🚀 PASSO 1 — Executar o binário

```bash
cd ~/cloaker-fix
chmod +x cloaker-linux
./cloaker-linux
```

> Isso iniciará o servidor local em `http://localhost:3005`

---

## 📂 PASSO 2 — Estrutura de pastas

Organize seus arquivos HTML da página "black" assim:

```
cloaker-fix/
├── cloaker-linux
└── src/
    └── pages/
        ├── black/
        │   ├── index.html
        │   ├── style.css
        │   └── script.js
        └── white/
            ├── index.html
            ├── style.css
            └── script.js
```

O Cloaker serve automaticamente essa pasta em `/`.

---

## 🌐 CONFIGURAÇÃO DO SERVIDOR WEB

### Nginx

```nginx
server {
    listen 80;
    server_name seusite.com www.seusite.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name seusite.com www.seusite.com;

    ssl_certificate /etc/letsencrypt/live/seusite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/seusite.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://localhost:3005;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location ~ /\. {
        deny all;
    }
}
```

---

### Apache

```apache
<VirtualHost *:80>
    ServerName seusite.com
    ServerAlias www.seusite.com

    # Redireciona para HTTPS
    Redirect permanent / https://seusite.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName seusite.com
    ServerAlias www.seusite.com

    # Certificados SSL (Let's Encrypt)
    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/seusite.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/seusite.com/privkey.pem

    # Cabeçalhos de segurança recomendados
    Header always set X-Frame-Options SAMEORIGIN
    Header always set X-Content-Type-Options nosniff
    Header always set X-XSS-Protection "1; mode=block"

    # Proxy reverso para o cloaker na porta 3005
    ProxyPreserveHost On
    ProxyPass / http://localhost:3005/
    ProxyPassReverse / http://localhost:3005/

    # Bloqueio de arquivos ocultos (.htaccess, .env, etc.)
    <Directory "/var/www/html">
        Require all granted
        <FilesMatch "^\.">
            Require all denied
        </FilesMatch>
    </Directory>
</VirtualHost>
```

---

## 🧪 HTML de exemplo (`src/pages/black/index.html`)

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <title>Página Black</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>Bem-vindo à Página Black</h1>
  <p>Esta é a versão pública servida pelo cloaker.</p>
  <script src="script.js"></script>
</body>
</html>
```

---

## 💡 DICAS FINAIS

- Use `tmux` ou `screen` para deixar o binário rodando em segundo plano
- Você pode transformar o binário em serviço com `systemd` se quiser rodar no boot
- Atualize a pasta `src/pages/black` para modificar a página pública

---

## 📦 Download do Executável

🔗 [Clique aqui para baixar o `cloaker-linux`](https://www.mediafire.com/file/qs2kp0h93m0tadg/cloaker-linux/file)
