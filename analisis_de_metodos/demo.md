```mermaid
flowchart TB
    subgraph NIVEL_0["NIVEL 0: Manual Consciente"]
        USER[👤 Entrenador]
        SHEET[📊 Google Sheet<br/>Carga Diaria]
        USER -->|Carga desde celular| SHEET
    end

    subgraph NIVEL_1["NIVEL 1: Gatekeeper - Control de Calidad"]
        TRIGGER[⚡ Trigger: Row Added/Updated]
        VALIDATE{Validación<br/>Campos + Rangos}
        CLEAN[🧹 Normalización<br/>Trim, Type Cast]
        
        SHEET -->|Nueva fila| TRIGGER
        TRIGGER --> CLEAN
        CLEAN --> VALIDATE
    end

    subgraph REJECT["Camino RECHAZO"]
        ERROR_SHEET[❌ Update Sheet<br/>Status = Error]
        ERROR_MAIL[📧 Email Entrenador<br/>Corrección requerida]
        
        VALIDATE -->|❌ Falla validación| ERROR_SHEET
        ERROR_SHEET --> ERROR_MAIL
    end

    subgraph APPROVE["Camino APROBACIÓN"]
        BQ_INSERT[✅ BigQuery Insert<br/>raw_daily_load]
        SUCCESS_SHEET[✅ Update Sheet<br/>Status = Procesado]
        
        VALIDATE -->|✅ Pasa validación| BQ_INSERT
        BQ_INSERT --> SUCCESS_SHEET
    end

    subgraph STORAGE["Data Warehouse"]
        BQ[(BigQuery)]
        RAW[raw_daily_load]
        SNAPSHOTS[weekly_snapshots]
        ATHLETES[athletes]
        
        BQ_INSERT --> RAW
        RAW --> BQ
    end

    subgraph NIVEL_2["NIVEL 2: Operativo - Snapshots + Informes"]
        CRON[⏰ Cron Semanal<br/>Lunes 6AM]
        AGGREGATE[📊 Agregación SQL<br/>Métricas por atleta]
        SNAPSHOT_OUT[📄 Sheet Snapshot<br/>+ PDF Informe]
        DRIVE[💾 Google Drive<br/>Informes/]
        
        CRON -->|Trigger semanal| AGGREGATE
        BQ -->|Lee raw_daily_load| AGGREGATE
        AGGREGATE -->|Escribe| SNAPSHOTS
        SNAPSHOTS --> SNAPSHOT_OUT
        SNAPSHOT_OUT --> DRIVE
    end

    subgraph NIVEL_3["NIVEL 3: Producto - Alertas + Onboarding"]
        ALERT_QUERY[🔍 Query Diaria<br/>Cambios >25%]
        ALERT_NOTIFY[🔔 Notificación<br/>WhatsApp/Email]
        
        ONBOARD_TRIGGER[➕ Nuevo Atleta<br/>en Roster]
        CREATE_SHEET[📋 Crear Sheet<br/>Individual]
        WELCOME_MAIL[👋 Email Bienvenida]
        
        BQ -->|Lee snapshots| ALERT_QUERY
        ALERT_QUERY -->|Si cambio significativo| ALERT_NOTIFY
        
        ONBOARD_TRIGGER --> CREATE_SHEET
        CREATE_SHEET --> WELCOME_MAIL
        CREATE_SHEET -->|Registra| ATHLETES
    end

    subgraph NIVEL_4["NIVEL 4: Asistencia (Controlado)"]
        MANUAL_TRIGGER[🖱️ Ejecución Manual<br/>Entrenador solicita]
        LLM[🤖 Claude API<br/>Sugerencia narrativa]
        DRAFT[📝 Google Doc<br/>BORRADOR]
        
        MANUAL_TRIGGER --> LLM
        BQ -->|Datos atleta| LLM
        LLM -->|Texto sugerido| DRAFT
        DRAFT -.->|Revisión humana| USER
    end

    style NIVEL_0 fill:#e3f2fd
    style NIVEL_1 fill:#fff3e0
    style NIVEL_2 fill:#f3e5f5
    style NIVEL_3 fill:#e8f5e9
    style NIVEL_4 fill:#fce4ec
    
    style REJECT fill:#ffebee
    style APPROVE fill:#e8f5e9
    style STORAGE fill:#f5f5f5
```

## Leyenda del Flujo

### Niveles de Madurez

| Nivel | Color | Objetivo | Estado en Demo |
|-------|-------|----------|----------------|
| **0** | Azul claro | Entrada manual, sin automatización | ✅ Visible explícitamente |
| **1** | Naranja claro | Control de calidad automático | ✅ Core de la demo |
| **2** | Púrpura claro | Snapshots y reportes periódicos | ✅ Muestra valor operativo |
| **3** | Verde claro | Alertas y onboarding | ✅ Diferenciación producto |
| **4** | Rosa claro | Asistencia IA controlada | ⚠️ Solo ejemplo limitado |

### Tipos de Nodos

- 🔵 **Círculo**: Trigger o evento
- 🔷 **Diamante**: Decisión/validación
- 🟦 **Rectángulo**: Proceso/acción
- 🗄️ **Cilindro**: Base de datos
- ☁️ **Nube**: Servicio externo

### Puntos Críticos de la Demo

1. **Feedback Loop** (NIVEL 1): 
   - El sistema escribe de vuelta en el Sheet del usuario
   - No es una caja negra

2. **Idempotencia** (NIVEL 1):
   - `Status_Sistema` previene reprocesamiento
   - Hash MD5 en BigQuery evita duplicados

3. **Operación sin fricción** (NIVEL 2):
   - Cron semanal: 0 intervención manual
   - Informes listos los lunes antes que llegue el entrenador

4. **Producto vs Proyecto** (NIVEL 3):
   - Onboarding automático
   - Alertas contextuales (no prescriptivas)

5. **IA como asistente** (NIVEL 4):
   - Siempre manual
   - Siempre marcado como borrador
   - Nunca reemplaza criterio
