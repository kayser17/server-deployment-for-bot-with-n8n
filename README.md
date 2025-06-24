# Despliegue del servidor N8N con Nginx + Let's Encrypt + Tailscale

Este repositorio contiene los archivos necesarios para desplegar el servicio N8N con soporte de base de datos PostgreSQL (pgvector), Redis y túnel Tailscale, utilizando **Nginx como reverse proxy** con HTTPS automatizado mediante Let's Encrypt (Certbot).

---

## 1️⃣ Configuración de Nginx

### Instalar Nginx en el servidor

```bash
sudo apt update
sudo apt install nginx
```

---

### Copiar la plantilla de configuración de Nginx

```bash
sudo cp nginx/site_template.conf /etc/nginx/sites-available/yourdomain.com
```

Editar el archivo y sustituir:

```nginx
server_name YOUR_DOMAIN_HERE;
```

por el dominio que se vaya a utilizar, por ejemplo:

```nginx
server_name example.com;
```

---

### Activar el sitio

```bash
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
```

---

### Verificar la configuración de Nginx

```bash
sudo nginx -t
```

---

### Recargar Nginx

```bash
sudo systemctl reload nginx
```

---

## 2️⃣ Configuración de HTTPS (Let's Encrypt)

### Instalar Certbot

```bash
sudo apt install certbot python3-certbot-nginx
```

---

### Obtener certificado SSL para el dominio

```bash
sudo certbot --nginx -d yourdomain.com
```

Certbot configurará automáticamente HTTPS y programará la renovación automática del certificado.

---

## 3️⃣ Configuración de Tailscale

El primer paso es crear una cuenta en Tailscale y añadir la máquina del servidor a tu **Tailnet**. Puedes hacerlo descargando la aplicación Tailscale desde su página web: https://tailscale.com/download

Una vez instalada, necesitas una clave para añadir los contenedores de Docker a tu Tailnet.

Pasos:

1. Crea la carpeta `~/.config` si no existe:

```bash
mkdir -p ~/.config
```

2. Crea un archivo llamado `tsauthkey` en `~/.config`:

```bash
nano ~/.config/tsauthkey
```

3. Ve al panel de administración de Tailscale: https://login.tailscale.com/admin/settings/keys

4. Haz clic en **Generate auth key...**

- Activa la opción **Reusable**.
- Genera la clave.
- Copia la clave en el archivo `~/.config/tsauthkey`.

5. Asegura los permisos del archivo:

```bash
chmod 600 ~/.config/tsauthkey
```

---

## 4️⃣ Configuración de N8N

1. Navegar a la carpeta `n8n/`:

```bash
cd n8n/
```

2. Copiar el archivo de variables:

```bash
cp example.env .env
```

3. Editar el archivo `.env`:

- `N8N_HOST` debe ser el hostname de tu servidor.
- `WEBHOOK_URL` debe ser la URL de tu servidor.
- `GENERIC_TIMEZONE` debe ser tu zona horaria.

Ejemplo:

```env
N8N_HOST=yourdomain.com
WEBHOOK_URL=https://yourdomain.com/
GENERIC_TIMEZONE=Europe/Madrid
```

---

## 5️⃣ Lanzar el stack (N8N + BBDD + Redis + Tailscale)

Desde la carpeta `n8n/`:

```bash
docker-compose up -d
```

---
