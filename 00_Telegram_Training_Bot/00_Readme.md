# 🏋️ Training Bot - Guía de Setup Completa

## 📋 Checklist General

- [ ] Crear bot en Telegram
- [ ] Instalar nodo de Telegram en n8n
- [ ] Crear dataset en BigQuery
- [ ] Configurar credenciales en n8n
- [ ] Importar workflow base
- [ ] Hacer primera prueba

---

## 1️⃣ CREAR BOT EN TELEGRAM (5 minutos)

### Paso a paso:

1. **Abrir Telegram y buscar**: `@BotFather`

2. **Enviar**: `/newbot`

3. **Elegir nombre**:
   - Nombre display: `Training Assistant` (o el que quieras)
   - Username: `tu_training_bot` (debe terminar en `_bot`)

4. **Guardar el TOKEN**:
   ```
   Te va a dar algo como:
   123456789:ABCdefGHIjklMNOpqrsTUVwxyz
   ```
   ⚠️ **IMPORTANTE**: Guardalo en un lugar seguro, lo vas a necesitar en n8n.

5. **Configurar comandos** (opcional pero recomendado):
   - En BotFather: `/setcommands`
   - Elegir tu bot
   - Pegar esto:
   ```
   start - Iniciar bot
   rutina - Ver rutina actual
   log - Registrar ejercicio
   progreso - Ver progreso
   nueva - Crear rutina nueva
   help - Ayuda
   ```

6. **Probar que funciona**:
   - Buscar tu bot en Telegram
   - Enviar `/start`
   - Debe responder algo (aunque sea genérico)

---

## 2️⃣ INSTALAR NODO DE TELEGRAM EN N8N

### Opción A: Desde la UI de n8n

1. Ir a **Settings** → **Community Nodes**
2. Buscar: `n8n-nodes-telegram`
3. Click en **Install**
4. Esperar que instale
5. Reiniciar n8n si es necesario

### Opción B: Desde terminal (si tenés acceso SSH a la VM)

```bash
# Conectarte a tu VM
cd ~/.n8n  # o donde tengas n8n instalado

# Instalar el paquete
npm install n8n-nodes-telegram

# Reiniciar n8n
pm2 restart n8n
# O si usás docker:
docker restart n8n
```

### Verificar instalación:

- En n8n, crear nuevo workflow
- Buscar nodo "Telegram"
- Deberías ver: **Telegram Trigger** y **Telegram**

---

## 3️⃣ CREAR DATASET EN BIGQUERY

### Opción A: Desde la consola de Google Cloud

1. Ir a: https://console.cloud.google.com/bigquery
2. Seleccionar tu proyecto (el mismo donde está la VM)
3. Click en **CREATE DATASET**
4. Configuración:
   - **Dataset ID**: `training_bot`
   - **Location**: `us-central1` (o la región que uses)
   - **Default table expiration**: Never
5. Click **CREATE DATASET**

### Opción B: Desde bq CLI

```bash
bq mk --dataset \
  --location=us-central1 \
  tu-proyecto-id:training_bot
```

### Ejecutar el schema SQL:

1. Copiar el SQL del artifact anterior
2. Reemplazar `tu_proyecto.tu_dataset` por:
   - `tu-proyecto-id.training_bot`
3. En BigQuery console → **Compose New Query**
4. Pegar y ejecutar cada bloque

---

## 4️⃣ CONFIGURAR CREDENCIALES EN N8N

### Para Telegram:

1. En n8n → **Settings** → **Credentials**
2. **New Credential** → Buscar "Telegram"
3. Pegar tu **Bot Token** (el que te dio BotFather)
4. Guardar

### Para BigQuery (si no lo tenés):

1. En Google Cloud Console:
   - IAM & Admin → Service Accounts
   - Create Service Account
   - Darle rol: `BigQuery Data Editor` y `BigQuery Job User`
2. Crear Key (JSON)
3. Descargar el archivo JSON
4. En n8n:
   - New Credential → Google Cloud
   - Subir el JSON
   - Guardar

---

## 5️⃣ CONFIGURAR WEBHOOK DE TELEGRAM

### ⚠️ CRÍTICO: Necesitás HTTPS

Tu Caddy ya te da HTTPS, así que estás cubierto.

### URL del webhook:

Formato:
```
https://tu-dominio.com/webhook/telegram-bot
```

Ejemplo:
```
https://n8n.tudominio.com/webhook-test/telegram-bot
```

### En n8n:

1. Crear workflow nuevo
2. Agregar nodo **Telegram Trigger**
3. Configurar:
   - **Credentials**: Las que creaste
   - **Updates**: `message`
4. Click en **Listen for Test Event**
5. Copiar la URL que genera
6. Enviar mensaje a tu bot
7. Si funciona, verás el mensaje en n8n

---

## 6️⃣ ESTRUCTURA BÁSICA DEL WORKFLOW

```
Telegram Trigger
    ↓
Function (Parse Command)
    ↓
Switch (Router por comando)
    ↓
┌─────────┬─────────┬─────────┐
│ /rutina │  /log   │/progreso│
└─────────┴─────────┴─────────┘
    ↓         ↓         ↓
BigQuery  BigQuery  BigQuery
    ↓         ↓         ↓
 Format   Format    Format
    ↓         ↓         ↓
Telegram Send Message
```

---

## 🧪 TEST INICIAL (comandos básicos)

### Primer comando: `/start`

**Flujo**:
1. Usuario envía `/start`
2. Bot responde: "¡Hola! Soy tu asistente de entrenamiento"

### Segundo comando: `/rutina`

**Flujo**:
1. Usuario envía `/rutina`
2. n8n consulta BigQuery
3. Bot responde con la rutina actual

---

## 🔧 TROUBLESHOOTING

### Telegram no recibe mensajes:

- Verificar que el webhook esté activo
- Verificar HTTPS (Caddy debe estar corriendo)
- En BotFather: `/deleteWebhook` y reconectar

### BigQuery da error de permisos:

- Verificar Service Account
- Verificar roles en IAM

### n8n no ejecuta el workflow:

- Verificar que esté "Active" (toggle en verde)
- Ver logs: `pm2 logs n8n`

---

## 📊 PRÓXIMOS PASOS

Una vez que tengas funcionando:

1. `/start` básico
2. `/rutina` consultando BigQuery
3. Agregar `/log` para registrar
4. Implementar `/progreso`
5. Integrar con Looker para imágenes

---

## 🆘 NECESITÁS AYUDA

Si te trabás en algún paso específico, avisame y te doy la solución exacta para ese punto.

Siguiente paso recomendado: **Crear el bot en Telegram** (5 min) y avisame cuando lo tengas listo.
