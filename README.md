Despliegue del servidor N8N con Nginx + Let's Encrypt
Este repositorio contiene los archivos necesarios para desplegar el servicio N8N con soporte de base de datos PostgreSQL (pgvector), Redis y túnel Tailscale, utilizando Nginx como reverse proxy con HTTPS automatizado mediante Let's Encrypt (Certbot).

1️⃣ Configuración de Nginx
Instalar Nginx en el servidor
bash
Copiar
Editar
sudo apt update
sudo apt install nginx
Copiar la plantilla de configuración de Nginx
bash
Copiar
Editar
sudo cp nginx/site_template.conf /etc/nginx/sites-available/yourdomain.com
Editar el archivo y sustituir:

nginx
Copiar
Editar
server_name YOUR_DOMAIN_HERE;
por el dominio que se vaya a utilizar, por ejemplo:

nginx
Copiar
Editar
server_name example.com;
Activar el sitio
bash
Copiar
Editar
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
Verificar la configuración de Nginx
bash
Copiar
Editar
sudo nginx -t
Recargar Nginx
bash
Copiar
Editar
sudo systemctl reload nginx
2️⃣ Configuración de HTTPS (Let's Encrypt)
Instalar Certbot
bash
Copiar
Editar
sudo apt install certbot python3-certbot-nginx
Obtener certificado SSL para el dominio
bash
Copiar
Editar
sudo certbot --nginx -d yourdomain.com
Certbot configurará automáticamente HTTPS y programará la renovación automática del certificado.

3️⃣ Lanzar el stack (N8N + BBDD + Redis + Tailscale)
Desde la carpeta n8n/:

bash
Copiar
Editar
cd n8n/
docker-compose up -d
Estructura del stack
Nginx (reverse proxy, instalado en el sistema)

Let's Encrypt (certbot)

N8N (contenedor)

PostgreSQL + pgvector (contenedor)

Redis (contenedor)

Tailscale (contenedor VPN)

Variables de entorno
El archivo de variables se encuentra en:

bash
Copiar
Editar
n8n/example.env
Copiarlo como .env y configurar los valores necesarios.

Notas finales
Este repositorio permite a cualquier usuario desplegar su propio servidor N8N personalizado.

Solo es necesario sustituir YOUR_DOMAIN_HERE en la configuración de Nginx por el dominio que se vaya a utilizar.

Los certificados SSL se gestionan de forma automática gracias a Let's Encrypt.
