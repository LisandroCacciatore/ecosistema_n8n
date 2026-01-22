¡Me alegra muchísimo que haya funcionado! 🎉 Ha sido una batalla épica contra puertos, redes y certificados, pero ahora tenés una infraestructura profesional, escalable y tuya.

Acá tenés la **Documentación Técnica Completa** en formato Markdown. Podés copiar este bloque, guardarlo en un archivo llamado `README.md` o `DOCUMENTACION.md` en tu computadora, y tenerlo como referencia sagrada por si alguna vez tenés que migrar o reinstalar.

---

# Guía de Despliegue: n8n + Evolution API + Caddy en Google Cloud

Esta guía documenta el proceso completo para desplegar un entorno de automatización profesional utilizando **n8n** (automatización), **Evolution API** (WhatsApp) y **Caddy** (Reverse Proxy con SSL Automático) en una máquina virtual de Ubuntu.

---

## 1. Infraestructura: Google Cloud Platform (GCP)

### 1.1 Creación de la VM

* **Servicio:** Compute Engine > VM Instances.
* **OS:** Ubuntu 20.04 o 22.04 LTS (x86/64).
* **Tipo de máquina:** e2-medium (recomendado para empezar) o e2-standard-2.
* **Disco:** Al menos 20GB.

### 1.2 Configuración de Red (IP Estática)

Para evitar que la IP cambie al reiniciar:

1. Ir a **VPC Network** > **IP Addresses**.
2. Reservar la dirección IP externa actual de la VM como **Estática**.

### 1.3 Firewall (Excepciones)

Es crítico abrir los puertos HTTP y HTTPS para que Caddy pueda certificar el dominio.

1. Ir a **VPC Network** > **Firewall**.
2. Crear regla **Ingress** (Entrada):
* **Targets:** `All instances in the network` (o usar tags como `http-server`).
* **Source filter:** IPv4 ranges.
* **Source IP ranges:** `0.0.0.0/0` (Todo internet).
* **Protocols and ports:** `tcp:80`, `tcp:443`.


3. **Prioridad:** Asegurar que no haya reglas de bloqueo con prioridad más alta.

---

## 2. Configuración de Dominio (DNS)

En tu registrador de dominio (ej. Namecheap):

* **Tipo:** `A Record`
* **Host:** `@` (o `n8n` si usas subdominio).
* **Value:** `[TU_IP_PUBLICA_GCP]` (Ej: 34.9.x.x).
* **TTL:** Automatic (o 5 min).

> **Nota:** Esperar unos minutos para la propagación DNS antes de configurar SSL.

---

## 3. Preparación del Servidor (Docker)

Conectarse por SSH a la VM e instalar Docker Engine y Docker Compose (V2).

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Docker
sudo apt install docker.io -y

# Iniciar y habilitar Docker
sudo systemctl start docker
sudo systemctl enable docker

# Instalar Plugin Docker Compose (V2)
sudo apt install docker-compose-plugin -y

# Verificar versiones
docker --version
docker compose version

```

---

## 4. Estructura de Archivos y Red

Creamos una red de Docker para que los contenedores se hablen entre sí sin exponer puertos innecesarios.

```bash
# Crear red externa
docker network create n8n_network

```

Estructura de directorios recomendada:

```text
/home/usuario/
├── caddy/
│   ├── docker-compose.yml
│   └── Caddyfile
├── n8n/
│   └── docker-compose.yml
└── evolution-api/
    └── docker-compose.yml

```

---

## 5. Configuración de Servicios

### 5.1 Caddy (Reverse Proxy & SSL)

Archivo: `~/caddy/docker-compose.yml`

```yaml
version: '3.8'
services:
  caddy:
    image: caddy:2-alpine
    container_name: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - n8n_network

volumes:
  caddy_data:
  caddy_config:

networks:
  n8n_network:
    external: true

```

Archivo: `~/caddy/Caddyfile`
**Nota:** Reemplazar `tudominio.com` por el real.

```caddy
tudominio.com {
    # Enrutamiento para Evolution API
    handle_path /evolution/* {
        reverse_proxy evolution-api:8080
    }

    # Enrutamiento por defecto para n8n
    handle {
        reverse_proxy n8n:5678
    }
}

```

### 5.2 n8n (Automatización)

Archivo: `~/n8n/docker-compose.yml`

```yaml
version: "3.8"
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    environment:
      - N8N_HOST=tudominio.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://tudominio.com/
      - GENERIC_TIMEZONE=America/Argentina/Buenos_Aires
      # CRÍTICO: Desactivar cookie segura si Caddy maneja el SSL
      - N8N_SECURE_COOKIE=false 
      # CRÍTICO: Escuchar en todas las interfaces internas
      - N8N_LISTEN_ADDRESS=0.0.0.0
      # Clave de encriptación (GUARDAR EN LUGAR SEGURO)
      - N8N_ENCRYPTION_KEY=tu_clave_generada_aqui
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - n8n_network

volumes:
  n8n_data:

networks:
  n8n_network:
    external: true

```

### 5.3 Evolution API (WhatsApp)

Archivo: `~/evolution-api/docker-compose.yml`

```yaml
version: '3.8'
services:
  evolution-api:
    image: atendai/evolution-api:latest
    container_name: evolution-api
    restart: unless-stopped
    environment:
      # TU CLAVE SECRETA PARA PROTEGER EL WHATSAPP
      - AUTHENTICATION_API_KEY=TuClaveSeguraEvolution2024
      - SERVER_URL=https://tudominio.com/evolution
      - WEBHOOK_GLOBAL_ENABLED=true
      - CONFIG_SESSION_PHONE_CLIENT=Evolution API
    volumes:
      - evolution_data:/evolution/instances
      - evolution_store:/evolution/store
    networks:
      - n8n_network

volumes:
  evolution_data:
  evolution_store:

networks:
  n8n_network:
    external: true

```

---

## 6. Despliegue

Ejecutar en orden:

```bash
# 1. Levantar Caddy
cd ~/caddy
docker compose up -d

# 2. Levantar n8n
cd ~/n8n
docker compose up -d

# 3. Levantar Evolution API
cd ~/evolution-api
docker compose up -d

```

---

## 7. Troubleshooting (Solución de Problemas)

### Error 502 Bad Gateway

* **Causa:** Caddy no puede conectar con n8n.
* **Solución:**
1. Verificar que n8n corre: `docker logs n8n`.
2. Verificar que ambos están en `n8n_network`.
3. Asegurar que n8n tiene `N8N_LISTEN_ADDRESS=0.0.0.0` (si escucha en 127.0.0.1, Caddy no puede entrar).



### Login de n8n falla o recarga constantemente

* **Causa:** Conflicto de cookies HTTPS/HTTP.
* **Solución:** Poner `N8N_SECURE_COOKIE=false` en el docker-compose de n8n y reiniciar.

### Error SSL / Pantalla Roja en el Navegador

* **Causa A (Certificado Inválido):** Caddy falló al validar con Let's Encrypt.
* *Fix:* Revisar puertos 80/443 en Firewall GCP. Borrar volumen `caddy_data` y reiniciar.


* **Causa B (Fortinet/Red Corporativa):** Un firewall de oficina está interceptando la conexión.
* *Fix:* Probar acceso desde datos móviles (4G).



### Docker: "Conflict. The container name is already in use"

* **Causa:** Quedó un contenedor viejo o mal borrado.
* **Solución:** `docker rm -f [NOMBRE_O_ID]` y volver a hacer `docker compose up -d`.

### Docker: "KeyError: ContainerConfig"

* **Causa:** Versión obsoleta de `docker-compose` (v1).
* **Solución:** Usar el comando moderno `docker compose` (sin guion, v2).
