# Certificados SSL/TLS

Este directorio debe contener los certificados SSL para HTTPS.

## Generar certificados autofirmados (Desarrollo)

Para desarrollo local, puedes generar certificados autofirmados con OpenSSL:

```bash
openssl req -x509 -newkey rsa:4096 -keyout certs/private.key -out certs/certificate.crt -days 365 -nodes
```

O usar este comando con valores predefinidos:

```bash
openssl req -x509 -newkey rsa:4096 -keyout certs/private.key -out certs/certificate.crt -days 365 -nodes -subj "/C=ES/ST=Madrid/L=Madrid/O=Example/CN=localhost"
```

## Configurar HTTPS en nginx.conf

Una vez generados los certificados, actualiza `nginx.conf` agregando un segundo `server` block:

```nginx
server {
    listen 443 ssl http2;
    server_name _;

    ssl_certificate /etc/nginx/certs/certificate.crt;
    ssl_certificate_key /etc/nginx/certs/private.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://wildfly_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Redirigir HTTP a HTTPS (opcional)
server {
    listen 80;
    server_name _;
    return 301 https://$host$request_uri;
}
```

## Producción

Para producción, usa certificados de una Autoridad Certificadora (CA) como Let's Encrypt.

