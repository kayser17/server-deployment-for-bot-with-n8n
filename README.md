# Despliegue del servidor para un usar n8n
Este repositorio contiene dos formas de despliegue del servidor N8N:

- **Modo local**: para pruebas y desarrollo, más sencillo de configurar. Con n8n y Postgres
- **Modo servidor (producción)**: para despliegue en un servidor con Nginx como reverse proxy, HTTPS automatizado (Let's Encrypt) y VPN (Tailscale).

---
# 1. Despliegue en local (para pruebas)

### Pasos:

1. Navegar a la carpeta `local/`:

```bash
cd local/
```

2. Lanzar los contenedores:

```bash
docker-compose up -d
```

3. Acceder a N8N desde:

```
http://localhost:5678
```
**Notas**:

- No es necesario configurar `.env`, las variables ya están definidas dentro del `docker-compose.yml`.
- Este modo es ideal para pruebas rápidas y desarrollo local.

---
# 2. Despliegue en servidor (producción)

## 2.1 Configuración de Nginx
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

## 2.2 Configuración de HTTPS (Let's Encrypt)

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

## 2.3 Configuración de Tailscale

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

## 2.4 Configuración de N8N y servicios

1. Navegar a la carpeta `servidor/`:

```bash
cd servidor/
```

2. Copiar el archivo de variables:

```bash
cp example.env .env
```

3. Editar el archivo `.env`:

- Variables de N8N:

```env
N8N_HOST=yourdomain.com
WEBHOOK_URL=https://yourdomain.com/
GENERIC_TIMEZONE=Europe/Madrid
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=yourpassword
```

- Variables de PostgreSQL:

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=yourpassword
POSTGRES_DB=n8n
```

- Variables de pgAdmin:

```env
PGADMIN_DEFAULT_EMAIL=your-email@example.com
PGADMIN_DEFAULT_PASSWORD=yourpassword
```

4. Editar el archivo `docker-compose.yml`:

- La contraseña de Redis se encuentra definida directamente en el `docker-compose.yml`:

```yaml
command: ["redis-server", "--requirepass", "your_redis_password"]
```

Si deseas cambiar la contraseña de Redis, modifícala en este campo.

---

**Servicios que se lanzan en modo servidor**:

✅ N8N  
✅ PostgreSQL + pgvector  
✅ Redis  
✅ pgAdmin (opcional: para administración de la base de datos)  
✅ Tailscale

---

5. Lanzar todo el stack:

```bash
docker-compose up -d
```

---
## 2.5 Lanzar los contenedores 

Desde la carpeta `n8n/`:

```bash
docker-compose up -d
```

---
# Variables de entorno

- En **modo local**, no es necesario `.env` (las variables están en el `docker-compose.yml`).

- En **modo servidor**, el archivo de variables es:

```
servidor/example.env
```

Copiarlo como `.env` y configurar los valores.

---
# Notas finales

- Este repositorio permite desplegar el servidor N8N en modo local o en modo servidor.
- En modo servidor, se utiliza Nginx como reverse proxy con HTTPS y conexión segura con Tailscale.
- Los certificados SSL se gestionan de forma automática gracias a Let's Encrypt.
