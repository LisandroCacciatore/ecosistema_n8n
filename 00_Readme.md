1️⃣ Estructura de carpetas del proyecto
n8n-sports-agents/
├── workflows/                  # Flujos de agentes
│   ├── 01_powerlifting_analysis.json
│   ├── 02_fatigue_tracking.json
│   └── 03_metric_alerts.json
├── triggers/                   # Triggers dedicados (opcional si se quiere separar)
│   ├── webhook_triggers.json
│   └── cron_triggers.json
├── credentials/                # Templates para credenciales
│   ├── google_sheets_cred.json
│   ├── slack_cred.json
│   └── postgres_cred.json
├── scripts/                    # Scripts auxiliares (NodeJS o Python)
│   └── data_cleaning.js
├── docs/                       # Documentación interna
│   ├── 01_powerlifting_analysis.md
│   ├── 02_fatigue_tracking.md
│   └── 03_metric_alerts.md
├── protocols/                  # Troubleshooting, checklist, buenas prácticas
│   └── troubleshooting.md
└── README.md                   # Guía de instalación y uso

2️⃣ Workflows de ejemplo
Agente 1 – Análisis de sesión de Powerlifting

Objetivo: Analizar datos de entrenamiento y generar reportes de rendimiento.

Trigger: Webhook (/powerlifting-session) o Google Sheets (nuevo registro de sesión)

Transformación: Validación de campos (peso, reps, sets)

Checks: Valores nulos, rango de peso (>0 y <500kg), repeticiones >0

Salida: Google Sheets / PostgreSQL + Slack (resumen de sesión)

Archivo: workflows/01_powerlifting_analysis.json

Flujo resumido de nodos:

Webhook Trigger → recibe JSON con {athlete_id, exercise, weight, reps, sets, date}

Function Node → limpieza y normalización

IF Node → validación de rango y nulos

Google Sheets Node → guardar registro

Slack Node → notificar resumen

Error Handling → enviar alerta por Slack en caso de error

Agente 2 – Seguimiento de Fatiga

Objetivo: Calcular indicadores de fatiga según métricas de entrenamiento y HRV.

Trigger: Cron diario a las 8AM

Transformación: Calcula fatigue_score usando fórmula (ejemplo: carga x intensidad / HRV)

Checks: HRV fuera de rango (<40 o >120 ms)

Salida: Google Sheets + Notificación por email

Archivo: workflows/02_fatigue_tracking.json

Flujo resumido de nodos:

Cron Node → ejecución diaria

Google Sheets Node → leer últimas sesiones

Function Node → calcular fatigue_score

IF Node → detectar fatiga alta

Slack / Email Node → enviar alertas

PostgreSQL Node → guardar histórico

Agente 3 – Alertas de métricas fuera de rango

Objetivo: Detectar métricas clave fuera de rango y notificar inmediatamente.

Trigger: Webhook (/metrics-update) → recibe datos en tiempo real

Transformación: Mapear métricas a límites normales

Checks: IF Node → compara con límites

Salida: Slack / Email / Webhook de integración

Archivo: workflows/03_metric_alerts.json

Flujo resumido de nodos:

Webhook Node

Function Node → parsear métricas

IF Node → cada métrica fuera de rango

Slack Node → enviar alerta

Webhook Node → opcional para otro sistema

Error Handling → log en Google Sheets

3️⃣ Documentación interna (docs/)

Cada flujo tendrá su propio markdown:

Ejemplo: docs/01_powerlifting_analysis.md

# Powerlifting Session Analysis

## Objetivo
Analizar sesiones de powerlifting y generar reportes de rendimiento.

## Inputs
- athlete_id: string
- exercise: string
- weight: number (kg)
- reps: number
- sets: number
- date: YYYY-MM-DD

## Outputs
- Registro en Google Sheets / PostgreSQL
- Notificación en Slack

## Casos de uso
- Registrar sesión manual
- Integrar con sensores de gimnasio o app de tracking

## Troubleshooting
- Error 400 → revisar payload JSON
- Valores fuera de rango → log en nodo IF

## Checklist
- [ ] Credenciales Google Sheets cargadas
- [ ] Slack Webhook configurado
- [ ] Probar webhook con sample JSON

4️⃣ Triggers

Cron: para agentes de seguimiento diario

Webhook: para recepción de datos en tiempo real

Integraciones: Google Sheets, APIs deportivas externas (ej. Strava, Garmin, Wodify)

5️⃣ Credenciales y endpoints

Se deben colocar en credentials/ y cargar en n8n:

// google_sheets_cred.json
{
  "name": "Google Sheets Sports",
  "type": "serviceAccount",
  "credentials": "<RUTA/JSON_SERVICE_ACCOUNT>"
}

// slack_cred.json
{
  "name": "Slack Alerts",
  "type": "webhook",
  "url": "<SLACK_WEBHOOK_URL>"
}

// postgres_cred.json
{
  "name": "PostgreSQL Sports DB",
  "host": "<DB_HOST>",
  "port": 5432,
  "database": "<DB_NAME>",
  "user": "<DB_USER>",
  "password": "<DB_PASSWORD>"
}


⚠️ Recordatorio: reemplazar <...> con tus credenciales y endpoints reales.

6️⃣ README.md base para GitHub
# n8n Sports Agents

Automatizaciones de n8n para gestión y análisis de datos deportivos.

## Estructura del proyecto



workflows/ → Agentes deportivos (JSON)
triggers/ → Triggers dedicados (webhook, cron)
credentials/ → Templates para credenciales
scripts/ → Scripts auxiliares
docs/ → Documentación de cada workflow
protocols/ → Troubleshooting y checklist
README.md → Esta documentación


## Instalación

1. Clonar el repositorio
```bash
git clone <REPO_URL>
cd n8n-sports-agents


Importar workflows en n8n

Menú → Import → File → Seleccionar JSON desde workflows/

Configurar credenciales

Menú → Credentials → Crear credenciales según credentials/

Ejecutar workflows por primera vez

Activar cron/webhook según el flujo

Probar con datos de ejemplo (sample_payload.json si existe)

Uso

01_powerlifting_analysis: registrar sesiones y alertas de rendimiento

02_fatigue_tracking: calcular fatiga diaria y enviar alertas

03_metric_alerts: monitorear métricas en tiempo real y notificar

Buenas prácticas

Nombrar nodos y variables descriptivamente

Separar entornos dev/prod

Revisar logs y nodos IF para troubleshooting

Documentar cada cambio en docs/
