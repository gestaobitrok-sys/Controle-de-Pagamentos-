# Deploy do Controle de Pagamentos no Lightsail

Mesmo servidor (IP `15.229.168.209`), agora um quarto app. Subdomínio `pagamentos.trokservice.com.br`, porta `3004`.

## 1. Clonar

```bash
git clone https://github.com/gestaobitrok-sys/Controle-de-Pagamentos-.git ~/controle-pagamentos
cd ~/controle-pagamentos
```

## 2. Senha e chave secreta

```bash
cp .env.example .env
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
sed -i "s/^APP_PASSWORD=.*/APP_PASSWORD=SUA_SENHA_AQUI/" .env
sed -i "s/^SESSION_SECRET=.*/SESSION_SECRET=SUA_CHAVE_AQUI/" .env
grep -c "troque-esta" .env   # deve dar 0
```

## 3. Instalar e testar

```bash
npm install
npm approve-scripts better-sqlite3
npm install
node server.js
```
Deve mostrar `Servidor de Controle de Pagamentos rodando na porta 3004`. `Ctrl+C` para parar.

## 4. PM2

```bash
pm2 start server.js --name controle-pagamentos
pm2 save
```

## 5. Nginx

```bash
sudo tee /etc/nginx/sites-available/pagamentos > /dev/null << 'EOF'
server {
    listen 80;
    server_name pagamentos.trokservice.com.br;
    client_max_body_size 20M;

    location / {
        proxy_pass http://127.0.0.1:3004;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOF
sudo ln -sf /etc/nginx/sites-available/pagamentos /etc/nginx/sites-enabled/pagamentos
sudo nginx -t
sudo systemctl restart nginx
```

## 6. DNS

A `pagamentos` → `15.229.168.209`
AAAA `pagamentos` → `2600:1f1e:b26:d00:a82b:e75a:2b70:920b`

## 7. HTTPS

```bash
sudo certbot --nginx -d pagamentos.trokservice.com.br
```
Depois `COOKIE_SECURE=true` no `.env` e `pm2 restart controle-pagamentos`.
