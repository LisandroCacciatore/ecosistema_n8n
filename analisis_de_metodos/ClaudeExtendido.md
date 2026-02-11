# 📋 PROMPT: Diseño Completo del Starter Pack

```markdown
# Sistema de Tracking de Entrenamiento - Starter Pack

## Contexto del Producto

Sos un arquitecto de software especializado en productos SaaS para coaching deportivo. Necesito que diseñes la especificación técnica completa del **Starter Pack** de un sistema de tracking de entrenamiento de fuerza.

---

## 🎯 Objetivo del Producto

Crear un sistema que permita a coaches independientes (1-15 atletas) trackear programas de fuerza estructurados (5/3/1, BBB, GZCL) con:
- Registro simple de sesiones
- Comparación automática real vs programado
- Detección de alertas básicas
- Visualización clara de progreso
- Almacenamiento de videos de técnica

---

## 👥 Usuarios del Sistema

### Usuario Principal: Coach Independiente
- **Perfil:** Entrenador con 5-15 atletas en gimnasio pequeño/garage gym
- **Conocimientos técnicos:** Básicos (usa Google Sheets, WhatsApp, Instagram)
- **Dolor principal:** Trackea en papel o Excel, pierde tiempo, no ve patrones
- **Dispositivo:** Tablet/smartphone en el gimnasio, laptop en casa
- **Budget:** $49/mes máximo
- **Time to value:** Debe estar usando el sistema en <30 minutos

### Usuario Secundario: Atleta
- **Perfil:** Persona que entrena fuerza 3-4x/semana
- **Conocimientos técnicos:** Básicos (usa apps móviles comunes)
- **Dolor principal:** No sabe si está progresando, no tiene feedback de técnica
- **Dispositivo:** Smartphone
- **Expectativa:** Ver su progreso fácilmente, recibir feedback del coach

---

## 🏗️ Stack Tecnológico Definido

- **Database:** Google Sheets (familiaridad del usuario, costo $0)
- **Backend Logic:** Google Apps Script (integración nativa)
- **Input Interface:** Google Forms (mobile-friendly, sin app necesaria)
- **Visualization:** Looker Studio (gratis, potente)
- **Storage:** Google Drive (videos, backups)
- **Notifications:** Gmail (SMTP incluido en Google Workspace)

**Restricciones:**
- NO usar servicios pagos adicionales en Starter
- Debe funcionar 100% en ecosistema Google
- Sin código fuera de Apps Script
- Sin bases de datos externas

---

## 📦 Features del Starter Pack

Diseñá la arquitectura técnica, estructura de datos y especificaciones Gherkin para cada una de estas features:

---

### Feature 1: Onboarding Automatizado

**User Story:** Como coach nuevo, quiero poder empezar a usar el sistema en menos de 30 minutos sin ayuda técnica.

**Acceptance Criteria:**

```gherkin
Feature: Onboarding Automatizado

  Background:
    Given soy un coach que acaba de suscribirse al Starter Pack
    And recibí un email de bienvenida con link de setup

  Scenario: Setup inicial exitoso
    Given hago click en el link de setup del email
    When se abre el wizard de configuración
    Then veo una pantalla de bienvenida con video de 2 minutos
    And veo un checklist de 5 pasos claramente numerados
    
  Scenario: Creación automática de estructura
    Given estoy en el paso 1 del wizard
    When hago click en "Crear mi sistema"
    Then el sistema crea automáticamente:
      | Recurso                    | Ubicación                          |
      | Google Sheet "MiGym-Data"  | Mi Google Drive raíz               |
      | Carpeta "MiGym-Videos"     | Mi Google Drive raíz               |
      | Google Form "Registro"     | Asociado al Sheet                  |
      | Dashboard Looker (copia)   | Mi cuenta de Looker Studio         |
    And recibo confirmación visual de cada recurso creado
    And cada recurso tiene el nombre de mi gimnasio
    
  Scenario: Carga inicial de datos maestros
    Given el sistema fue creado exitosamente
    When llego al paso 2 "Configurar Gimnasio"
    Then veo un form simple con campos:
      | Campo              | Tipo      | Requerido | Default        |
      | Nombre del gimnasio| Text      | Sí        | -              |
      | Nombre del coach   | Text      | Sí        | -              |
      | Email del coach    | Email     | Sí        | [pre-llenado]  |
      | Timezone           | Dropdown  | Sí        | [auto-detect]  |
    And al completar, los datos se guardan en Sheet "config_gimnasio"
    
  Scenario: Carga de primer atleta
    Given completé la configuración del gimnasio
    When llego al paso 3 "Agregar tu primer atleta"
    Then veo un form con campos:
      | Campo              | Tipo      | Requerido | Ayuda contextual           |
      | Nombre completo    | Text      | Sí        | "Ej: Juan Pérez"           |
      | Email              | Email     | No        | "Para enviarle reportes"   |
      | Plan asignado      | Dropdown  | Sí        | "5/3/1, BBB, GZCL"         |
      | Squat TM (kg)      | Number    | Sí        | "Training Max, no 1RM"     |
      | Bench TM (kg)      | Number    | Sí        | -                          |
      | Deadlift TM (kg)   | Number    | Sí        | -                          |
      | Press TM (kg)      | Number    | Sí        | -                          |
      | Fecha inicio       | Date      | Sí        | [hoy]                      |
    And al guardar, se crea fila en Sheet "atletas"
    And se generan automáticamente todas las sesiones programadas para 4 semanas
    
  Scenario: Configuración de notificaciones
    Given agregué al menos 1 atleta
    When llego al paso 4 "Notificaciones"
    Then veo opciones de configuración:
      | Opción                          | Tipo     | Default |
      | Email diario con resumen        | Toggle   | ON      |
      | Email semanal con análisis      | Toggle   | ON      |
      | Alertas inmediatas (RPE >9)     | Toggle   | ON      |
      | Hora de envío diario            | Time     | 20:00   |
    And al configurar, se crean triggers en Apps Script
    
  Scenario: Tour del sistema
    Given completé todos los pasos de configuración
    When llego al paso 5 "Tour del Sistema"
    Then veo un tutorial interactivo que muestra:
      | Pantalla           | Duración | Contenido                                    |
      | Google Form        | 30 seg   | Cómo cargar una sesión                       |
      | Google Sheet       | 30 seg   | Dónde están los datos                        |
      | Dashboard Looker   | 45 seg   | Cómo leer las visualizaciones                |
      | Carpeta Videos     | 15 seg   | Dónde se guardan los videos                  |
    And cada pantalla tiene botón "Siguiente" y "Saltar tour"
    And al finalizar veo un botón "Cargar mi primera sesión"
    
  Scenario: Validación de setup completo
    Given terminé el onboarding
    When accedo al dashboard principal
    Then veo un mensaje de éxito
    And veo el botón "Abrir Form de Registro" destacado
    And veo un link "Ver video tutorial completo"
    And NO veo ningún error en consola
    And todos los recursos están correctamente vinculados
```

**Requisitos Técnicos:**

```markdown
### Arquitectura del Wizard

**Componente:** Google Apps Script Web App

**Endpoint:** 
- `doGet()` → Renderiza el wizard
- `doPost()` → Procesa cada paso

**Flujo de ejecución:**
1. Usuario hace click en link único con token (generado al pago)
2. Script valida token contra Sheet "subscripciones"
3. Script ejecuta función `setupCompleto(email, gymName)`
4. Función crea recursos en paralelo
5. Función retorna URLs de cada recurso
6. Frontend del wizard muestra checkmarks

**Estructura de datos generada:**

Sheet: `config_gimnasio`
| gym_id | gym_name | coach_name | coach_email | timezone | created_at          | onboarding_completed |
|--------|----------|------------|-------------|----------|---------------------|---------------------|
| UUID   | String   | String     | String      | String   | Timestamp           | Boolean             |

Sheet: `config_notificaciones`
| gym_id | daily_email | weekly_email | instant_alerts | daily_time | created_at |
|--------|-------------|--------------|----------------|------------|------------|
| UUID   | Boolean     | Boolean      | Boolean        | Time       | Timestamp  |

**Scripts a generar:**
- `onboarding/createGymStructure.js`
- `onboarding/copyDashboardTemplate.js`
- `onboarding/setupTriggers.js`
- `onboarding/sendWelcomeEmail.js`
```

---

### Feature 2: Sistema de Registro de Sesiones

**User Story:** Como coach, quiero registrar rápidamente los lifts de mis atletas durante la sesión de entrenamiento desde mi tablet.

**Acceptance Criteria:**

```gherkin
Feature: Registro de Sesiones

  Background:
    Given soy un coach con atletas activos en el sistema
    And estoy en el gimnasio con mi tablet
    And tengo el Google Form abierto

  Scenario: Interfaz optimizada para carga rápida
    Given abro el form de registro
    Then veo una interfaz mobile-first con:
      | Elemento               | Comportamiento                                    |
      | Header                 | Logo del gym + "Registro de Sesión"               |
      | Dropdown Atleta        | Lista de atletas activos, ordenados alfabéticamente |
      | Dropdown Lift          | Squat, Bench, Deadlift, Press                     |
      | Input Peso             | Teclado numérico, placeholder "kg"                |
      | Input Reps             | Teclado numérico, placeholder "reps"              |
      | Selector RPE           | Slider visual 1-10 con colores (verde→rojo)       |
      | Toggle Video           | "¿Subir video?" OFF por default                   |
      | Textarea Observaciones | Opcional, placeholder "Notas sobre la serie"      |
      | Botón Submit           | Grande, verde, texto "Registrar"                  |
    And todos los campos críticos están en viewport sin scroll
    
  Scenario: Autocompletado inteligente
    Given seleccioné un atleta en el dropdown
    When el sistema detecta mi selección
    Then automáticamente:
      | Campo                | Valor autocompletado                                    |
      | Plan                 | [Plan asignado al atleta, campo oculto]                 |
      | Ciclo actual         | [Calculado desde fecha_inicio]                          |
      | Semana actual        | [Calculado desde fecha_inicio]                          |
      | Día actual           | [Basado en calendario del plan]                         |
      | Peso programado      | [Mostrado como referencia bajo input Peso]              |
      | Reps programadas     | [Mostrado como referencia bajo input Reps]              |
    And veo un mensaje contextual: "Hoy toca: Squat - Semana 2 - 3x3 @ 90%"
    
  Scenario: Validación en tiempo real
    Given estoy completando el form
    When ingreso datos en cada campo
    Then veo validación instantánea:
      | Campo | Valor ingresado | Validación                                           | Mensaje             |
      | Peso  | 0               | Error                                                | "Debe ser > 0"      |
      | Peso  | 500             | Warning                                              | "¿Seguro? Es muy alto" |
      | Reps  | 0               | Error                                                | "Debe ser > 0"      |
      | Reps  | 50              | Warning                                              | "¿Seguro? Son muchas reps" |
      | RPE   | <5 con peso alto| Warning                                              | "RPE bajo para ese peso" |
    And los mensajes se muestran inline bajo cada campo
    And el botón Submit está deshabilitado si hay errores
    
  Scenario: Comparación visual con lo programado
    Given ingresé Peso y Reps
    When los valores difieren de lo programado
    Then veo indicadores visuales:
      | Diferencia              | Indicador                                    |
      | Peso 5-10% menor        | 🟡 Amarillo "Levantaste un poco menos"      |
      | Peso >10% menor         | 🔴 Rojo "Peso muy por debajo de lo esperado"|
      | Peso >5% mayor          | 🟢 Verde "¡Superaste lo programado!"        |
      | Reps en AMRAP <5        | 🟡 Amarillo "Pocas reps en AMRAP"           |
      | Reps en AMRAP ≥8        | 🟢 Verde "¡Muy bien! Considerar subir TM"   |
    And cada indicador tiene un tooltip explicativo
    
  Scenario: Subida opcional de video
    Given marqué "¿Subir video?" como ON
    When llego al campo de video
    Then veo dos opciones:
      | Opción                  | Comportamiento                                          |
      | "Grabar ahora"          | Abre cámara del dispositivo, graba, sube automáticamente |
      | "Subir desde galería"   | Abre selector de archivos                                |
    And veo restricciones: "Máx 100MB, formatos: MP4, MOV"
    And mientras sube veo progress bar
    And al completar veo checkmark "✓ Video subido"
    
  Scenario: Submit exitoso con feedback inmediato
    Given completé todos los campos requeridos
    When presiono "Registrar"
    Then veo un loading spinner
    And en <3 segundos veo pantalla de confirmación:
      | Elemento           | Contenido                                             |
      | Icono              | ✓ grande y verde                                      |
      | Título             | "¡Sesión registrada!"                                 |
      | Resumen            | "Juan Pérez - Squat - 120kg x 5 reps - RPE 8"        |
      | Comparación        | "Programado: 115kg x 5+ → Hiciste +4% peso, +0 reps" |
      | Botón primario     | "Registrar siguiente atleta" (resetea form)           |
      | Botón secundario   | "Ver dashboard" (abre Looker)                         |
    And el form se limpia automáticamente
    And el dropdown de Atleta queda en el último seleccionado +1 (facilita carga masiva)
    
  Scenario: Manejo de errores de conexión
    Given estoy en zona con internet inestable
    When presiono Submit y la conexión falla
    Then el sistema:
      | Paso | Acción                                                           |
      | 1    | Guarda datos en localStorage del navegador                       |
      | 2    | Muestra mensaje "Sin conexión. Reintentando..."                  |
      | 3    | Intenta reenviar cada 10 segundos                                |
      | 4    | Muestra contador "Reintento 1/10"                                |
      | 5    | Si logra conexión → submit exitoso + "✓ Datos sincronizados"    |
      | 6    | Si 10 intentos fallan → "Guardado localmente. Se enviará al reconectar" |
    And al recuperar conexión, envía automáticamente
    And notifica al coach "3 sesiones pendientes sincronizadas"
    
  Scenario: Registro de sesión sin lo programado (off-program)
    Given un atleta hizo un lift fuera de programa
    When selecciono atleta y lift
    And veo mensaje "Este lift no está programado para hoy"
    Then tengo opción de:
      | Opción              | Resultado                                           |
      | "Cancelar"          | Vuelvo al form vacío                                 |
      | "Registrar igual"   | Permite completar form, marca sesión como "off-program" |
    And si registro igual, en el dashboard aparece con tag "Fuera de programa"
```

**Requisitos Técnicos:**

```markdown
### Arquitectura del Form

**Tecnología:** Google Form + Custom HTML (Apps Script Web App)

**Por qué NO usar Google Form nativo:**
- No permite validación en tiempo real
- No permite mostrar peso/reps programados
- No permite lógica condicional compleja
- UX limitada para mobile

**Decisión:** Custom HTML Form + Apps Script backend

**Stack del Form:**
- Frontend: HTML5 + TailwindCSS + Alpine.js (lightweight reactivity)
- Backend: Apps Script endpoints
- Storage: Google Sheets

**Endpoints necesarios:**

```javascript
// GET /form
function doGet(e) {
  // Renderiza el HTML form
  return HtmlService.createHtmlOutputFromFile('form')
    .setTitle('Registro de Sesión');
}

// POST /submit
function submitSession(data) {
  // Valida datos
  // Busca sesión programada
  // Guarda en Sheet "registro_real"
  // Si hay video, llama a organizarVideo()
  // Retorna confirmación
}

// GET /athlete-data/{athleteId}
function getAthleteData(athleteId) {
  // Retorna plan, ciclo, semana, sesión de hoy
  // Usado para autocompletar form
}

// POST /upload-video
function uploadVideo(athleteId, lift, videoBlob) {
  // Guarda video en Drive
  // Retorna URL del video
}
```

**Estructura de datos generada:**

Sheet: `registro_real`
| timestamp           | gym_id | alumno_id | plan_id | ciclo | semana | dia | lift     | set_num | peso_real | reps_real | rpe | video_url | observaciones | off_program |
|---------------------|--------|-----------|---------|-------|--------|-----|----------|---------|-----------|-----------|-----|-----------|---------------|-------------|
| 2026-02-10 10:30:00 | UUID   | 1         | 531     | 1     | 2      | 1   | Squat    | 3       | 120       | 5         | 8   | url       | Forma OK      | FALSE       |

**Scripts a generar:**
- `form/renderForm.html` (Frontend)
- `form/submitSession.js` (Backend)
- `form/getAthleteData.js` (Data fetching)
- `form/uploadVideo.js` (Video handling)
- `form/validations.js` (Business logic)
```

---

### Feature 3: Comparación Automática Real vs Programado

**User Story:** Como coach, quiero ver automáticamente si mis atletas están cumpliendo con el programa o están por debajo/encima de lo esperado.

**Acceptance Criteria:**

```gherkin
Feature: Comparación Real vs Programado

  Background:
    Given tengo atletas con sesiones registradas
    And cada atleta tiene un plan asignado (5/3/1, BBB, GZCL)

  Scenario: Generación automática de sesiones programadas
    Given un atleta fue creado con:
      | Campo         | Valor      |
      | Nombre        | Juan Pérez |
      | Plan          | 5/3/1      |
      | Squat TM      | 150kg      |
      | Fecha inicio  | 2026-01-15 |
    When el sistema ejecuta el trigger "generarSesionesProgramadas"
    Then se crean filas en Sheet "sesiones_programadas":
      | alumno_id | ciclo | semana | dia | lift  | set_num | peso_programado | reps_objetivo | tipo_set |
      | 1         | 1     | 1      | 1   | Squat | 1       | 97.5            | 5             | warmup   |
      | 1         | 1     | 1      | 1   | Squat | 2       | 112.5           | 5             | work     |
      | 1         | 1     | 1      | 1   | Squat | 3       | 127.5           | 5+            | amrap    |
    And se generan sesiones para 4 semanas completas (1 ciclo)
    And el cálculo de peso es: TM × intensidad (según template del plan)
    
  Scenario: Comparación automática post-submit
    Given Juan registró una sesión:
      | lift  | peso_real | reps_real | rpe |
      | Squat | 127.5     | 7         | 8   |
    When el sistema procesa el submit
    Then automáticamente:
      | Paso | Acción                                                              |
      | 1    | Busca sesión programada (atleta + ciclo + semana + lift + set_num)  |
      | 2    | Calcula delta_peso = peso_real - peso_programado                    |
      | 3    | Calcula delta_reps = reps_real - reps_objetivo (solo si AMRAP)      |
      | 4    | Calcula delta_rpe = comparación con RPE histórico promedio          |
      | 5    | Asigna flag según matriz de decisión                                |
      | 6    | Guarda fila en Sheet "comparacion"                                  |
    
  Scenario: Matriz de decisión para flags
    Given se calcularon los deltas
    When el sistema evalúa los valores
    Then asigna flags según:
      | Condición                                    | Flag                      | Color  |
      | delta_reps >= 3 AND rpe < 7.5                | "✅ Subir TM"            | Verde  |
      | delta_reps >= 0 AND rpe <= 8                 | "✓ Cumplió"              | Verde  |
      | delta_reps < 0 AND rpe <= 7                  | "⚠️ Subcarga"            | Amarillo|
      | delta_reps < 0 AND rpe > 8                   | "🔴 TM muy alto"         | Rojo   |
      | rpe > 9                                      | "🔴 RPE crítico"         | Rojo   |
      | delta_peso < -10%                            | "⚠️ Peso muy bajo"       | Amarillo|
      | off_program = TRUE                           | "ℹ️ Fuera de programa"   | Azul   |
    And un registro puede tener múltiples flags
    
  Scenario: Visualización en dashboard de comparación
    Given hay múltiples sesiones registradas
    When abro el dashboard de Looker Studio
    Then veo una tabla:
      | Atleta       | Fecha      | Lift     | Prog    | Real    | Δ Peso | Δ Reps | RPE | Flag           |
      | Juan Pérez   | 2026-02-10 | Squat    | 127.5kg | 127.5kg | 0%     | +2     | 8   | ✅ Subir TM   |
      | María García | 2026-02-10 | Bench    | 60kg    | 60kg    | 0%     | +1     | 7.5 | ✓ Cumplió     |
      | Carlos López | 2026-02-10 | Deadlift | 150kg   | 135kg   | -10%   | -2     | 9   | 🔴 TM muy alto|
    And cada fila tiene código de color según flag
    And puedo filtrar por atleta, fecha, flag
    And puedo ordenar por cualquier columna
    
  Scenario: Detección de tendencias (3+ sesiones)
    Given un atleta tiene 3+ sesiones registradas del mismo lift
    When el sistema analiza el histórico
    Then calcula métricas adicionales:
      | Métrica                    | Cálculo                                              |
      | Tendencia de RPE           | Promedio móvil últimas 3 sesiones                    |
      | Tendencia de delta_reps    | Si está subiendo, bajando o estable                  |
      | Variabilidad de RPE        | Desviación estándar últimas 5 sesiones               |
      | Consistencia de ejecución  | % de sesiones que cumplieron reps programadas        |
    And estas métricas se muestran en un panel separado "Tendencias"
    And se generan alertas si:
      | Condición                                  | Alerta                                      |
      | RPE sube 3 sesiones consecutivas           | "⚠️ Posible fatiga acumulada"              |
      | delta_reps negativo 2+ veces/semana        | "🔴 TM probablemente muy alto"             |
      | Variabilidad RPE > 2 puntos                | "ℹ️ RPE inconsistente - revisar criterio"  |
    
  Scenario: Histórico de comparaciones
    Given quiero ver la evolución de un atleta
    When filtro por atleta y lift en el dashboard
    Then veo un gráfico de líneas:
      | Eje X        | Eje Y (izq)    | Eje Y (der) |
      | Semanas      | Peso (kg)      | RPE         |
    And hay dos líneas:
      | Línea              | Color | Estilo     |
      | Peso programado    | Gris  | Punteada   |
      | Peso real          | Azul  | Sólida     |
    And los puntos donde delta_reps < 0 están marcados con ⚠️
    And puedo hacer hover en cada punto para ver detalles
```

**Requisitos Técnicos:**

```markdown
### Arquitectura de Comparación

**Trigger:** `onFormSubmit` de Sheet "registro_real"

**Flujo de ejecución:**

```javascript
function onFormSubmit(e) {
  var row = e.values; // Fila recién agregada
  
  // 1. Extraer datos
  var alumno_id = row[2];
  var ciclo = row[4];
  var semana = row[5];
  var lift = row[7];
  var set_num = row[8];
  var peso_real = row[9];
  var reps_real = row[10];
  var rpe = row[11];
  
  // 2. Buscar sesión programada
  var programado = buscarSesionProgramada(alumno_id, ciclo, semana, lift, set_num);
  
  if (!programado) {
    // Sesión off-program, marcar y skip comparación
    marcarOffProgram(row);
    return;
  }
  
  // 3. Calcular deltas
  var delta_peso = peso_real - programado.peso_programado;
  var delta_reps = (programado.tipo_set === "amrap") 
    ? reps_real - parseInt(programado.reps_objetivo)
    : null;
  
  // 4. Obtener RPE histórico
  var rpe_historico = calcularRPEHistorico(alumno_id, lift);
  var delta_rpe = rpe - rpe_historico;
  
  // 5. Aplicar matriz de decisión
  var flags = aplicarMatrizDecision({
    delta_peso: delta_peso,
    delta_reps: delta_reps,
    rpe: rpe,
    delta_rpe: delta_rpe,
    off_program: false
  });
  
  // 6. Guardar en Sheet comparacion
  guardarComparacion({
    alumno_id: alumno_id,
    timestamp: row[0],
    lift: lift,
    peso_programado: programado.peso_programado,
    peso_real: peso_real,
    reps_programadas: programado.reps_objetivo,
    reps_real: reps_real,
    rpe: rpe,
    delta_peso: delta_peso,
    delta_reps: delta_reps,
    flags: flags.join(", "),
    color: determinarColor(flags)
  });
  
  // 7. Si hay flags críticos, disparar alerta
  if (flags.includes("🔴 TM muy alto") || flags.includes("🔴 RPE crítico")) {
    enviarAlertaInmediata(alumno_id, lift, flags);
  }
}
```

**Funciones auxiliares requeridas:**

```javascript
function buscarSesionProgramada(alumno_id, ciclo, semana, lift, set_num) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("sesiones_programadas");
  var data = sheet.getDataRange().getValues();
  
  for (var i = 1; i < data.length; i++) {
    if (data[i][0] == alumno_id &&
        data[i][2] == ciclo &&
        data[i][3] == semana &&
        data[i][5] == lift &&
        data[i][6] == set_num) {
      return {
        peso_programado: data[i][7],
        reps_objetivo: data[i][8],
        tipo_set: data[i][9]
      };
    }
  }
  return null;
}

function calcularRPEHistorico(alumno_id, lift) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("registro_real");
  var data = sheet.getDataRange().getValues();
  
  var rpes = [];
  for (var i = 1; i < data.length; i++) {
    if (data[i][2] == alumno_id && data[i][7] == lift) {
      rpes.push(data[i][11]);
    }
  }
  
  if (rpes.length === 0) return 7.5; // Default
  
  var suma = rpes.reduce((a, b) => a + b, 0);
  return suma / rpes.length;
}

function aplicarMatrizDecision(params) {
  var flags = [];
  
  // Decisiones basadas en AMRAP
  if (params.delta_reps !== null) {
    if (params.delta_reps >= 3 && params.rpe < 7.5) {
      flags.push("✅ Subir TM");
    } else if (params.delta_reps >= 0 && params.rpe <= 8) {
      flags.push("✓ Cumplió");
    } else if (params.delta_reps < 0 && params.rpe <= 7) {
      flags.push("⚠️ Subcarga");
    } else if (params.delta_reps < 0 && params.rpe > 8) {
      flags.push("🔴 TM muy alto");
    }
  }
  
  // Decisiones basadas en RPE
  if (params.rpe > 9) {
    flags.push("🔴 RPE crítico");
  }
  
  // Decisiones basadas en peso
  var pct_diferencia = (params.delta_peso / params.peso_programado) * 100;
  if (pct_diferencia < -10) {
    flags.push("⚠️ Peso muy bajo");
  }
  
  if (params.off_program) {
    flags.push("ℹ️ Fuera de programa");
  }
  
  return flags;
}
```

**Estructura de datos generada:**

Sheet: `comparacion`
| alumno_id | nombre       | timestamp           | lift     | peso_prog | peso_real | delta_peso | reps_prog | reps_real | delta_reps | rpe | flags           | color   |
|-----------|--------------|---------------------|----------|-----------|-----------|------------|-----------|-----------|------------|-----|-----------------|---------|
| 1         | Juan Pérez   | 2026-02-10 10:30:00 | Squat    | 127.5     | 127.5     | 0          | 5+        | 7         | +2         | 8   | ✅ Subir TM    | #00FF00 |
| 2         | María García | 2026-02-10 10:35:00 | Bench    | 60        | 60        | 0          | 5+        | 6         | +1         | 7.5 | ✓ Cumplió      | #00AA00 |

**Scripts a generar:**
- `comparison/onFormSubmit.js`
- `comparison/buscarSesionProgramada.js`
- `comparison/aplicarMatrizDecision.js`
- `comparison/calcularTendencias.js`
- `comparison/guardarComparacion.js`
```

---

### Feature 4: Dashboard de Visualización en Looker Studio

**User Story:** Como coach, quiero ver de un vistazo el progreso de todos mis atletas sin tener que abrir el Sheet.

**Acceptance Criteria:**

```gherkin
Feature: Dashboard en Looker Studio

  Background:
    Given soy un coach con atletas activos
    And hay sesiones registradas en el sistema
    And el dashboard de Looker está conectado al Sheet

  Scenario: Estructura del dashboard
    Given abro el dashboard en Looker Studio
    Then veo 4 páginas claramente etiquetadas:
      | Página          | Propósito                                         |
      | 1. Overview     | Resumen general de todos los atletas              |
      | 2. Individual   | Análisis profundo de un atleta específico         |
      | 3. Alertas      | Tabla de flags críticos que requieren atención    |
      | 4. Tendencias   | Métricas de progreso a largo plazo                |
    And cada página tiene navegación visible (tabs en header)
    And el logo de mi gimnasio está en top-left
    
  Scenario: Página 1 - Overview
    Given estoy en la página "Overview"
    Then veo los siguientes paneles:
      
      Panel A - KPIs Principales (cards horizontales):
      | Métrica                    | Cálculo                                     | Color   |
      | Atletas activos            | COUNT(atletas WHERE activo=TRUE)            | Azul    |
      | Sesiones esta semana       | COUNT(registro_real WHERE fecha >= lunes)   | Verde   |
      | Compliance rate            | % de sesiones programadas vs realizadas     | Amarillo|
      | Alertas pendientes         | COUNT(flags WHERE color=rojo)               | Rojo    |
      
      Panel B - Tabla de atletas:
      | Columna           | Contenido                                                |
      | Nombre            | Clickeable, filtra página Individual                      |
      | Plan              | 5/3/1, BBB, GZCL                                          |
      | Última sesión     | Fecha + hace cuántos días                                 |
      | Compliance 30d    | % de sesiones realizadas vs programadas                   |
      | Progreso          | Indicador visual ↑↗→↘↓                                   |
      | Alertas           | Count de flags rojos/amarillos                            |
      
      Panel C - Gráfico de barras "Compliance por atleta":
      | Eje X        | Eje Y         | Color por barra                              |
      | Nombre       | % Compliance  | Verde >90%, Amarillo 70-90%, Rojo <70%       |
      
      Panel D - Timeline de sesiones (últimos 7 días):
      | Eje X        | Eje Y                | Contenido                        |
      | Día          | Hora (6am - 10pm)    | Dot por cada sesión registrada   |
      | Color        | Por atleta           | Facilita ver patterns de horario |
    
    And todos los paneles son responsive
    And todos los gráficos tienen tooltips informativos
    
  Scenario: Página 2 - Individual
    Given estoy en la página "Individual"
    And seleccioné un atleta del dropdown
    Then veo:
      
      Panel A - Info del atleta (header):
      | Campo              | Contenido                                           |
      | Nombre + foto      | Si tiene foto en Drive, sino inicial                |
      | Plan actual        | 5/3/1, BBB, etc.                                    |
      | Ciclo / Semana     | "Ciclo 2 - Semana 3"                                |
      | Fecha inicio       | "Inició: 15/01/2026 (hace 4 semanas)"               |
      | TMs actuales       | Squat: 150kg, Bench: 100kg, Dead: 180kg, Press: 70kg|
      
      Panel B - Progresión por lift (4 gráficos de líneas):
      | Gráfico     | Eje X    | Eje Y (izq)          | Eje Y (der) |
      | Squat       | Semanas  | Peso (kg)            | RPE         |
      | Bench       | Semanas  | Peso (kg)            | RPE         |
      | Deadlift    | Semanas  | Peso (kg)            | RPE         |
      | Press       | Semanas  | Peso (kg)            | RPE         |
      
      Cada gráfico tiene:
      - Línea gris punteada: Peso programado
      - Línea azul sólida: Peso real
      - Puntos naranjas: RPE (eje derecho)
      - Marcadores rojos en sesiones con flags críticos
      
      Panel C - AMRAPs Tracker:
      | Columna     | Contenido                                           |
      | Semana      | Semana del ciclo (1, 2, 3, 4)                       |
      | Squat       | Reps logradas (color: verde ≥5, amarillo 3-4, rojo <3)|
      | Bench       | ídem                                                |
      | Deadlift    | ídem                                                |
      | Press       | ídem                                                |
      
      Panel D - Tabla de últimas 10 sesiones:
      | Columna        | Contenido                                        |
      | Fecha          | DD/MM/YYYY                                       |
      | Lift           | Squat, Bench, Dead, Press                        |
      | Programado     | Peso x Reps                                      |
      | Realizado      | Peso x Reps                                      |
      | RPE            | Número con color (verde <7, amarillo 7-8, rojo >8)|
      | Flag           | Emoji del flag                                   |
      | Video          | 🎥 si existe, clickeable                        |
      
    And puedo exportar esta vista como PDF
    
  Scenario: Página 3 - Alertas
    Given estoy en la página "Alertas"
    Then veo:
      
      Panel A - Resumen de alertas (cards):
      | Métrica                 | Valor                                      |
      | Alertas críticas (🔴)   | COUNT(flags WHERE incluye 🔴)              |
      | Advertencias (⚠️)        | COUNT(flags WHERE incluye ⚠️)              |
      | Pendientes de revisión  | Alertas sin campo "revisado" marcado       |
      
      Panel B - Tabla de alertas ordenada por prioridad:
      | Columna        | Contenido                                        | Ordenamiento   |
      | Prioridad      | 🔴 Crítico, ⚠️ Advertencia, ℹ️ Info            | DESC           |
      | Atleta         | Nombre                                           | -              |
      | Fecha          | Cuándo ocurrió                                   | DESC (reciente)|
      | Lift           | Squat, Bench, etc.                               | -              |
      | Flag           | Texto del flag completo                          | -              |
      | Acción sugerida| Auto-generada según flag                         | -              |
      | Revisado       | Checkbox (actualiza Sheet al marcar)             | -              |
      
      Y veo filtros en header:
      - Por prioridad (🔴, ⚠️, ℹ️, todas)
      - Por atleta (dropdown)
      - Por lift (dropdown)
      - Solo no revisadas (toggle ON por default)
      
    And las filas críticas tienen fondo rojo claro
    And al hacer click en una fila se expande mostrando:
      - Contexto: últimas 3 sesiones del mismo lift
      - Recomendación específica
      - Botón "Marcar como revisado"
      
  Scenario: Página 4 - Tendencias
    Given estoy en la página "Tendencias"
    And tengo al menos 4 semanas de datos
    Then veo:
      
      Panel A - Progresión de TM (gráfico de barras agrupadas):
      | Eje X        | Series             | Colores                          |
      | Atletas      | Squat, Bench,      | Squat: azul, Bench: verde,       |
      |              | Dead, Press        | Dead: rojo, Press: amarillo      |
      | Eje Y        | % de incremento    | Desde inicio hasta ahora         |
      
      Panel B - Distribución de RPE por lift (box plots):
      | Gráfico      | Contenido                                          |
      | Squat        | Box plot de todos los RPEs registrados             |
      | Bench        | Muestra mediana, cuartiles, outliers               |
      | Deadlift     | Útil para ver si RPE es consistente                |
      | Press        | -                                                  |
      
      Panel C - Heatmap de compliance (calendario):
      | Eje X        | Eje Y         | Color de celda                         |
      | Días del mes | Atletas       | Verde: entrenó, Rojo: faltó, Gris: no programado |
      
      Panel D - Tabla de proyecciones:
      | Atleta       | Lift     | TM actual | Proyección +4 sem | Base                    |
      | Juan Pérez   | Squat    | 150kg     | 157.5kg (+5%)     | Promedio últimos 3 ciclos|
      | María García | Bench    | 60kg      | 62.5kg (+4%)      | ídem                    |
      
    And todos los gráficos permiten drill-down (click para filtrar)
    
  Scenario: Interactividad y filtros globales
    Given estoy en cualquier página del dashboard
    When uso los filtros del header
    Then puedo filtrar por:
      | Filtro          | Opciones                                          | Scope         |
      | Rango de fechas | Última semana, Último mes, Último ciclo, Custom   | Todas páginas |
      | Atleta          | Dropdown con todos los atletas activos            | Todas páginas |
      | Plan            | 5/3/1, BBB, GZCL, Todos                           | Todas páginas |
      | Lift            | Squat, Bench, Deadlift, Press, Todos              | Páginas 2-4   |
    And al aplicar un filtro, todas las páginas se actualizan
    And los filtros se mantienen al navegar entre páginas
    And hay un botón "Limpiar filtros" visible
    
  Scenario: Responsividad mobile
    Given abro el dashboard desde un smartphone
    Then veo:
      - Gráficos apilados verticalmente (no lado a lado)
      - Tablas con scroll horizontal si necesario
      - Cards de KPIs en 2 columnas (no 4)
      - Filtros colapsables en menú hamburger
      - Fuente de mínimo 14px legible
    And todos los elementos son clickeables con dedos (min 44x44px)
    
  Scenario: Performance y tiempo de carga
    Given tengo 15 atletas con 200+ sesiones totales
    When cargo el dashboard
    Then el tiempo de carga es:
      | Elemento              | Tiempo máximo |
      | Header + navegación   | <1 seg        |
      | KPIs principales      | <2 seg        |
      | Gráficos de línea     | <3 seg        |
      | Tablas completas      | <4 seg        |
    And veo loading spinners mientras carga cada panel
    And NO veo errores de timeout
```

**Requisitos Técnicos:**

```markdown
### Arquitectura del Dashboard

**Plataforma:** Looker Studio (Google Data Studio)

**Fuentes de datos:**

1. **Conexión 1: registro_real**
   - Tipo: Google Sheets
   - Hoja: `registro_real`
   - Refresh: Automático cada hora
   - Campos calculados necesarios:
     ```
     compliance_flag = IF(reps_real >= reps_programadas, "✓", "✗")
     dias_desde_sesion = DATE_DIFF(TODAY(), fecha, DAY)
     semana_del_año = WEEK(fecha)
     ```

2. **Conexión 2: comparacion**
   - Tipo: Google Sheets
   - Hoja: `comparacion`
   - Refresh: Automático cada hora
   - Campos calculados:
     ```
     prioridad_numerica = CASE
       WHEN CONTAINS_TEXT(flags, "🔴") THEN 1
       WHEN CONTAINS_TEXT(flags, "⚠️") THEN 2
       ELSE 3
     END
     ```

3. **Conexión 3: atletas**
   - Tipo: Google Sheets
   - Hoja: `atletas`
   - Refresh: Manual (cambia poco)

**Blended Data (joins necesarios):**

```
JOIN comparacion + atletas ON alumno_id
JOIN registro_real + sesiones_programadas ON (alumno_id, ciclo, semana, lift)
```

**Configuración de Looker:**

```javascript
// Theme personalizado
{
  "colors": {
    "primary": "#2563EB",    // Azul
    "success": "#10B981",     // Verde
    "warning": "#F59E0B",     // Amarillo
    "danger": "#EF4444",      // Rojo
    "neutral": "#6B7280"      // Gris
  },
  "fonts": {
    "heading": "Inter",
    "body": "Inter"
  }
}
```

**Scripts de soporte (Apps Script):**

```javascript
// Actualizar campo "revisado" desde Looker
function marcarAlertaRevisada(alertaId) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("comparacion");
  var data = sheet.getDataRange().getValues();
  
  for (var i = 1; i < data.length; i++) {
    if (data[i][0] == alertaId) { // Asumiendo columna A = ID
      sheet.getRange(i + 1, 15).setValue(true); // Columna O = revisado
      break;
    }
  }
}

// Generar proyecciones para tabla de tendencias
function calcularProyecciones() {
  var comparacion = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("comparacion").getDataRange().getValues();
  var atletas = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("atletas").getDataRange().getValues();
  
  // Lógica para calcular proyección de TM basada en:
  // - Promedio de incrementos en últimos 3 ciclos
  // - Tendencia de AMRAPs
  // - Consistencia de RPE
  
  // Guardar en nueva hoja "proyecciones"
}
```

**Template de Looker a duplicar:**

- URL del template master: `[LINK A TEMPLATE PÚBLICO]`
- Cada coach obtiene una copia al hacer onboarding
- Script que reemplaza data source con su Sheet

**Entregables:**

- `dashboards/looker_template.json` (configuración exportada)
- `dashboards/copyTemplate.js` (script para duplicar)
- `dashboards/README.md` (guía de configuración manual)
```

---

### Feature 5: Sistema de Alertas y Notificaciones

**User Story:** Como coach, quiero recibir alertas automáticas cuando un atleta necesita atención, sin tener que revisar el dashboard manualmente todos los días.

**Acceptance Criteria:**

```gherkin
Feature: Sistema de Alertas

  Background:
    Given soy un coach con el sistema configurado
    And tengo notificaciones habilitadas

  Scenario: Alerta inmediata por RPE crítico
    Given un atleta registra una sesión con RPE > 9
    When el sistema procesa el submit
    Then inmediatamente:
      | Paso | Acción                                                    |
      | 1    | Detecta condición crítica                                 |
      | 2    | Crea entrada en Sheet "alertas_pendientes"                |
      | 3    | Envía email al coach en <5 minutos                        |
    
    And el email contiene:
      | Elemento         | Contenido                                              |
      | Subject          | "🔴 ALERTA: RPE crítico - [Atleta] - [Lift]"          |
      | Header           | Logo del gym + "Atención requerida"                    |
      | Cuerpo principal | "[Atleta] registró RPE 9.5 en Squat con 120kg x 3 reps"|
      | Contexto         | "Últimas 3 sesiones de Squat: RPE 8, 8.5, 9.5"        |
      | Recomendación    | "Considerar: reducir TM 5-10%, insertar deload, revisar técnica" |
      | Botón CTA        | "Ver detalles en dashboard" → Link directo             |
      | Footer           | Link para ajustar preferencias de notificaciones       |
    
    And el email tiene formato HTML responsive
    And NO se envían más de 3 alertas inmediatas por día (evitar spam)
    
  Scenario: Resumen diario de sesiones
    Given configuré "Email diario" = ON a las 20:00
    And hay sesiones registradas hoy
    When llegan las 20:00
    Then recibo un email con:
      
      Subject: "📊 Resumen del día - [X] sesiones registradas"
      
      Sección 1 - KPIs del día:
      | Métrica                    | Valor                        |
      | Sesiones registradas       | 12                           |
      | Compliance del día         | 85% (programadas: 14)        |
      | RPE promedio               | 7.8                          |
      | Alertas generadas          | 2 (1 crítica, 1 advertencia) |
      
      Sección 2 - Alertas del día (solo si existen):
      | Atleta       | Lift     | Flag               | Acción sugerida      |
      | Carlos López | Deadlift | 🔴 TM muy alto    | Reducir TM 10%       |
      | Ana Ruiz     | Squat    | ⚠️ RPE subiendo   | Monitorear próxima sesión |
      
      Sección 3 - Destacados (top 3):
      - ✅ Juan Pérez - Squat AMRAP: 8 reps (programadas: 5+) → Subir TM
      - ✅ María García - Nueva PR: Deadlift 125kg
      - ⚠️ Luis Torres - 3er día sin entrenar (última sesión: hace 5 días)
      
      Sección 4 - Acción requerida:
      - [ ] Revisar 2 alertas críticas
      - [ ] Ajustar TM de Carlos López
      - [ ] Contactar a Luis Torres (ausente 5 días)
      
      Footer:
      - Ver dashboard completo [LINK]
      - Cambiar preferencias de email [LINK]
    
    And si NO hay sesiones hoy, NO se envía email
    
  Scenario: Resumen semanal los domingos
    Given configuré "Email semanal" = ON
    And es domingo a las 20:00
    When el sistema genera el resumen
    Then recibo un email con:
      
      Subject: "📈 Resumen Semanal - Semana del [fecha inicio] al [fecha fin]"
      
      Sección 1 - Overview de la semana:
      | Métrica                    | Valor             | Cambio vs semana anterior |
      | Total de sesiones          | 48                | +5 (+12%)                 |
      | Compliance promedio        | 87%               | +2%                       |
      | Atletas que entrenaron     | 14 de 15          | +1                        |
      | Volumen total (toneladas)  | 12,450 kg         | +850 kg (+7%)             |
      
      Sección 2 - Atletas destacados:
      1. Juan Pérez: Incrementó todos los TMs, compliance 100%, RPE consistente
      2. María García: Nueva PR en Bench, progresión sostenida
      3. Ana Ruiz: 4 semanas consecutivas sin faltar
      
      Sección 3 - Requieren atención:
      1. Carlos López: RPE alto 3 sesiones consecutivas → TM probablemente muy alto
      2. Luis Torres: Solo entrenó 1 vez → Contactar
      3. Roberto Díaz: Faltó a 4 sesiones programadas → Revisar compromiso
      
      Sección 4 - Proyecciones para próxima semana:
      - 3 atletas terminan ciclo → Revisar si subir TM
      - Semana de deload programada para 2 atletas
      - 5 atletas deberían lograr nuevas PRs
      
      Sección 5 - Acción requerida:
      - [ ] Ajustar TM de 3 atletas que terminan ciclo
      - [ ] Contactar a 2 atletas con baja asistencia
      - [ ] Revisar técnica de Carlos López (videos disponibles)
      
      Footer + firma
    
    And incluye gráficos embebidos (imágenes):
      - Compliance por atleta (barras)
      - RPE promedio por lift (box plot simplificado)
      - Volumen total últimas 4 semanas (línea)
    
  Scenario: Notificación de patrón de fatiga detectado
    Given un atleta tiene RPE subiendo 3 sesiones consecutivas
    And el ratio volumen/RPE está cayendo
    When el sistema ejecuta el análisis de tendencias (trigger diario)
    Then:
      | Paso | Acción                                                      |
      | 1    | Marca al atleta con flag "fatiga_acumulada"                 |
      | 2    | Crea entrada en "alertas_pendientes" con prioridad ALTA     |
      | 3    | Envía email específico al coach                             |
      
    Y el email contiene:
      Subject: "⚠️ Patrón de Fatiga Detectado - [Atleta]"
      
      Cuerpo:
      "El sistema detectó signos de fatiga acumulada en [Atleta]:
      
      Indicadores:
      - RPE incrementando: 7.5 → 8 → 8.5 → 9 (últimas 4 sesiones de Squat)
      - Ratio Volumen/RPE cayendo: 1,250 → 1,180 → 1,100 (-12% en 3 semanas)
      - Reps en AMRAP bajando: 7 → 6 → 5
      
      Recomendación:
      1. Insertar semana de deload inmediato
      2. Reducir volumen accesorio 30-50%
      3. Revisar recuperación (sueño, nutrición, stress)
      4. Considerar sesión de técnica ligera
      
      Próximos pasos sugeridos:
      - Programar semana de deload (sistema puede auto-generarla)
      - Agendar charla con el atleta
      - Revisar videos de ejecución si hay disponibles
      
      [Ver detalles completos en dashboard]"
      
  Scenario: Configuración de preferencias de notificaciones
    Given quiero personalizar mis notificaciones
    When accedo a "Configuración > Notificaciones"
    Then veo opciones para:
      
      | Categoría                  | Opciones                                        | Default |
      | Email diario               | ON/OFF, hora de envío (dropdown)                | ON, 20:00|
      | Email semanal              | ON/OFF, día (dropdown), hora                    | ON, Dom 20:00|
      | Alertas inmediatas         | ON/OFF                                          | ON      |
      | Umbral RPE crítico         | Slider 8-10                                     | 9       |
      | Umbral compliance bajo     | Slider 50-90%                                   | 70%     |
      | Incluir gráficos en emails | ON/OFF                                          | ON      |
      | Formato de email           | HTML/Plain text                                 | HTML    |
      
    And al cambiar cualquier opción, se actualiza Sheet "config_notificaciones"
    And veo preview del email con la configuración actual
    And puedo enviarme un "Email de prueba" para verificar
    
  Scenario: Gestión de alertas en dashboard
    Given tengo alertas pendientes
    When accedo a la página "Alertas" del dashboard
    Then puedo:
      | Acción                  | Resultado                                          |
      | Marcar como "Revisada"  | Desaparece de la vista "Pendientes"                |
      | Marcar como "Resuelta"  | Se archiva, desaparece del dashboard               |
      | Agregar nota            | Campo de texto libre, se guarda en Sheet           |
      | Posponer                | Selecciono fecha, reaparece ese día                |
      | Descartar               | Se elimina (con confirmación)                      |
    
    And cada acción actualiza el Sheet "alertas_pendientes"
    And hay un historial de alertas resueltas (ver últimas 30 días)
```

**Requisitos Técnicos:**

```markdown
### Arquitectura de Notificaciones

**Componentes:**

1. **Detección de alertas:** Apps Script triggers
2. **Almacenamiento:** Sheet "alertas_pendientes"
3. **Motor de envío:** Apps Script + Gmail API
4. **Templates:** HTML en archivos separados
5. **Configuración:** Sheet "config_notificaciones"

**Triggers necesarios:**

```javascript
// Trigger 1: Inmediato (al submit del form)
function onFormSubmit(e) {
  // ... código de comparación ...
  
  if (rpe > 9) {
    crearAlerta({
      tipo: "inmediata",
      prioridad: "critica",
      atleta_id: alumno_id,
      mensaje: "RPE crítico detectado",
      data: {lift, peso, reps, rpe}
    });
    
    enviarEmailInmediato(alumno_id, "rpe_critico");
  }
}

// Trigger 2: Diario a las 20:00
function enviarResumenDiario() {
  var config = obtenerConfigNotificaciones();
  
  if (!config.daily_email) return;
  
  var datos = recopilarDatosDiarios();
  var html = generarEmailDiario(datos);
  
  MailApp.sendEmail({
    to: config.coach_email,
    subject: "📊 Resumen del día - " + datos.total_sesiones + " sesiones",
    htmlBody: html
  });
}

// Trigger 3: Semanal (domingos 20:00)
function enviarResumenSemanal() {
  var config = obtenerConfigNotificaciones();
  
  if (!config.weekly_email) return;
  
  var datos = recopilarDatosSemanal();
  var graficos = generarGraficos(datos);
  var html = generarEmailSemanal(datos, graficos);
  
  MailApp.sendEmail({
    to: config.coach_email,
    subject: "📈 Resumen Semanal",
    htmlBody: html,
    inlineImages: graficos // Adjuntar gráficos
  });
}

// Trigger 4: Análisis de tendencias (diario 6am)
function analizarTendencias() {
  var atletas = obtenerAtletasActivos();
  
  atletas.forEach(function(atleta) {
    var tendencias = calcularTendenciasAtleta(atleta.id);
    
    if (tendencias.fatiga_detectada) {
      crearAlerta({
        tipo: "tendencia",
        prioridad: "alta",
        atleta_id: atleta.id,
        mensaje: "Patrón de fatiga acumulada",
        data: tendencias
      });
      
      enviarEmailInmediato(atleta.id, "fatiga_acumulada");
    }
  });
}
```

**Funciones auxiliares:**

```javascript
function crearAlerta(params) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("alertas_pendientes");
  
  sheet.appendRow([
    new Date(),                    // timestamp
    params.atleta_id,
    params.tipo,
    params.prioridad,
    params.mensaje,
    JSON.stringify(params.data),   // data serializada
    false,                         // revisada
    false,                         // resuelta
    null                           // fecha_resolucion
  ]);
}

function generarEmailDiario(datos) {
  var template = HtmlService.createTemplateFromFile('emails/resumen_diario');
  
  template.total_sesiones = datos.total_sesiones;
  template.compliance = datos.compliance;
  template.rpe_promedio = datos.rpe_promedio;
  template.alertas = datos.alertas;
  template.destacados = datos.destacados;
  template.acciones = datos.acciones_requeridas;
  template.dashboard_url = obtenerDashboardURL();
  
  return template.evaluate().getContent();
}

function generarGraficos(datos) {
  // Usa Google Charts API para generar imágenes
  var chartUrl = "https://image-charts.com/chart?";
  
  // Compliance por atleta
  var complianceChart = chartUrl + 
    "chs=600x300" +
    "&cht=bvg" +
    "&chd=t:" + datos.compliance_values.join(',') +
    "&chl=" + datos.atleta_nombres.join('|');
  
  var complianceBlob = UrlFetchApp.fetch(complianceChart).getBlob()
    .setName("compliance_chart");
  
  // Similar para otros gráficos...
  
  return {
    compliance: complianceBlob,
    // ... otros gráficos
  };
}

function recopilarDatosDiarios() {
  var hoy = new Date();
  var inicioDelDia = new Date(hoy.getFullYear(), hoy.getMonth(), hoy.getDate());
  
  var sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("registro_real");
  var data = sheet.getDataRange().getValues();
  
  var sesionesHoy = data.filter(function(row) {
    return row[0] >= inicioDelDia; // columna timestamp
  });
  
  return {
    total_sesiones: sesionesHoy.length,
    compliance: calcularCompliance(sesionesHoy),
    rpe_promedio: calcularRPEPromedio(sesionesHoy),
    alertas: obtenerAlertasDelDia(),
    destacados: obtenerDestacados(sesionesHoy),
    acciones_requeridas: generarAcciones()
  };
}
```

**Templates de email (HTML):**

```html
<!-- emails/resumen_diario.html -->
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: 'Inter', Arial, sans-serif; }
    .header { background: #2563EB; color: white; padding: 20px; }
    .kpi-card { 
      display: inline-block; 
      background: #F3F4F6; 
      padding: 15px; 
      margin: 10px;
      border-radius: 8px;
    }
    .alert-critica { background: #FEE2E2; border-left: 4px solid #EF4444; }
    .alert-warning { background: #FEF3C7; border-left: 4px solid #F59E0B; }
  </style>
</head>
<body>
  <div class="header">
    <h1>📊 Resumen del Día</h1>
    <p><?= new Date().toLocaleDateString() ?></p>
  </div>
  
  <div class="kpis">
    <div class="kpi-card">
      <h3><?= total_sesiones ?></h3>
      <p>Sesiones registradas</p>
    </div>
    <div class="kpi-card">
      <h3><?= compliance ?>%</h3>
      <p>Compliance</p>
    </div>
    <div class="kpi-card">
      <h3><?= rpe_promedio ?></h3>
      <p>RPE promedio</p>
    </div>
  </div>
  
  <? if (alertas.length > 0) { ?>
    <h2>🚨 Alertas del Día</h2>
    <? alertas.forEach(function(alerta) { ?>
      <div class="alert-<?= alerta.tipo ?>">
        <strong><?= alerta.atleta ?></strong> - <?= alerta.lift ?>
        <p><?= alerta.mensaje ?></p>
        <em>Acción: <?= alerta.accion ?></em>
      </div>
    <? }); ?>
  <? } ?>
  
  <!-- ... resto del template ... -->
  
  <div class="footer">
    <a href="<?= dashboard_url ?>">Ver Dashboard Completo</a>
  </div>
</body>
</html>
```

**Estructura de datos:**

Sheet: `alertas_pendientes`
| timestamp           | atleta_id | tipo        | prioridad | mensaje                 | data_json              | revisada | resuelta | fecha_resolucion | notas |
|---------------------|-----------|-------------|-----------|-------------------------|------------------------|----------|----------|------------------|-------|
| 2026-02-10 15:30:00 | 3         | inmediata   | critica   | RPE crítico detectado   | {"lift":"Squat","rpe":9.5} | FALSE    | FALSE    | null             | -     |

Sheet: `config_notificaciones`
| gym_id | coach_email      | daily_email | daily_time | weekly_email | weekly_day | instant_alerts | rpe_threshold |
|--------|------------------|-------------|------------|--------------|------------|----------------|---------------|
| UUID   | coach@email.com  | TRUE        | 20:00      | TRUE         | Sunday     | TRUE           | 9             |

**Scripts a generar:**
- `notifications/onFormSubmit.js`
- `notifications/enviarResumenDiario.js`
- `notifications/enviarResumenSemanal.js`
- `notifications/analizarTendencias.js`
- `notifications/crearAlerta.js`
- `notifications/templates/resumen_diario.html`
- `notifications/templates/resumen_semanal.html`
- `notifications/templates/alerta_inmediata.html`
```

---

### Feature 6: Almacenamiento y Gestión de Videos

**User Story:** Como coach, quiero que los videos de ejecución de mis atletas se organicen automáticamente y pueda accederlos fácilmente desde el dashboard.

**Acceptance Criteria:**

```gherkin
Feature: Gestión de Videos

  Background:
    Given soy un coach con el sistema configurado
    And tengo una estructura de carpetas en Google Drive

  Scenario: Subida de video desde el form
    Given estoy registrando una sesión en el form
    And marqué "¿Subir video?" = Sí
    When selecciono "Grabar ahora"
    Then se abre la cámara de mi dispositivo
    And veo un botón "Grabar" rojo
    And veo un timer 00:00
    
    When presiono "Grabar"
    Then el timer empieza a contar
    And el botón cambia a "Detener"
    And veo indicación visual que está grabando
    
    When presiono "Detener"
    Then veo preview del video
    And veo dos opciones:
      | Opción      | Acción                                    |
      | "Usar este" | Procede con la subida                     |
      | "Regrabar"  | Descarta y vuelve a abrir cámara          |
    
    When presiono "Usar este"
    Then veo progress bar de subida
    And veo porcentaje: "Subiendo... 45%"
    And el video se sube a Google Drive
    And al completar veo "✓ Video subido exitosamente"
    
  Scenario: Organización automática del video
    Given se subió un video para Juan Pérez, lift Squat, el 10/02/2026 a las 10:30
    When el sistema procesa la subida
    Then el video se guarda en:
      Ruta: `/CoachingSystem/Atletas/Juan_Perez_id_1/Videos/2026-02/`
      Nombre: `squat_2026-02-10_10-30.mp4`
    
    And se crea una fila en Sheet "videos":
      | alumno_id | nombre       | lift  | fecha               | url                        | revisado | feedback |
      | 1         | Juan Pérez   | Squat | 2026-02-10 10:30:00 | https://drive.google.com/... | FALSE    | -        |
    
    And la URL del video se vincula en Sheet "registro_real" (misma fila de la sesión)
    
  Scenario: Visualización de videos en dashboard
    Given hay videos subidos para varios atletas
    When abro la página "Individual" y selecciono un atleta
    Then en la tabla de "Últimas sesiones" veo:
      | Fecha      | Lift     | Video      |
      | 2026-02-10 | Squat    | 🎥 [click] |
      | 2026-02-08 | Deadlift | 🎥 [click] |
      | 2026-02-06 | Bench    | -          |
    
    When hago click en 🎥
    Then se abre el video en un modal overlay dentro del dashboard
    And veo el video con controles: play, pause, velocidad (0.5x, 1x, 2x)
    And debajo del video veo:
      | Campo       | Contenido                                    |
      | Fecha       | 10/02/2026 - 10:30 AM                        |
      | Lift        | Squat                                        |
      | Peso/Reps   | 120kg x 5 reps                               |
      | RPE         | 8                                            |
      | Observaciones| Forma OK                                    |
    
    And veo un textarea "Feedback del coach" (editable)
    And veo botón "Marcar como revisado"
    
  Scenario: Agregar feedback a un video
    Given abrí un video sin revisar
    When escribo en "Feedback del coach":
      "Buena profundidad. Mejorar estabilidad del core en la subida. Practicar pausa en el hoyo."
    And presiono "Guardar feedback"
    Then el texto se guarda en Sheet "videos", columna "feedback"
    And el campo "revisado" cambia a TRUE
    And se cierra el modal
    And el icono cambia de 🎥 a ✅
    
  Scenario: Panel de videos sin revisar
    Given tengo videos pendientes de revisión
    When abro la página "Alertas" del dashboard
    Then veo un panel adicional "Videos sin revisar":
      | Atleta       | Lift     | Fecha      | RPE | Acción            |
      | Juan Pérez   | Squat    | 2026-02-10 | 8   | 🎥 Revisar video  |
      | Carlos López | Deadlift | 2026-02-09 | 9   | 🎥 Revisar video  |
    
    And las filas con RPE > 8 están destacadas (priorizadas)
    And puedo ordenar por fecha (más reciente primero)
    And puedo filtrar por atleta o lift
    
  Scenario: Límites de almacenamiento
    Given tengo el plan Starter (5 GB de almacenamiento)
    When subo videos y me acerco al límite
    Then veo una advertencia al 80% de uso:
      "⚠️ Has usado 4 GB de 5 GB. Considera eliminar videos antiguos o mejorar a plan Pro."
    
    When intento subir un video que excedería el límite
    Then veo un error:
      "❌ No hay espacio suficiente. Este video necesita 150 MB pero solo hay 100 MB disponibles."
    
    And veo opciones:
      | Opción                          | Acción                                    |
      | "Liberar espacio"               | Abre panel de gestión de almacenamiento   |
      | "Mejorar a Pro (50 GB)"         | Redirige a página de upgrade              |
      | "Cancelar"                      | Vuelve al form sin subir                  |
    
  Scenario: Gestión de almacenamiento
    Given accedo a "Configuración > Almacenamiento"
    Then veo:
      Panel A - Uso actual:
      | Métrica                | Valor                  |
      | Total usado            | 3.2 GB de 5 GB (64%)   |
      | Videos totales         | 127                    |
      | Peso promedio/video    | 25 MB                  |
      | Atleta con más videos  | Juan Pérez (23 videos) |
      
      Panel B - Videos más antiguos (candidatos a eliminar):
      | Fecha      | Atleta       | Lift     | Tamaño | Revisado | Acción   |
      | 2025-11-10 | Pedro Gómez  | Squat    | 45 MB  | ✅       | 🗑️ Eliminar |
      | 2025-11-12 | Ana Torres   | Bench    | 32 MB  | ✅       | 🗑️ Eliminar |
      
      Panel C - Eliminación masiva:
      - Checkbox: "Eliminar videos revisados con más de X meses" (slider: 1-12 meses)
      - Botón: "Vista previa" (muestra qué se eliminaría)
      - Botón: "Eliminar seleccionados" (con confirmación)
    
    When selecciono videos y presiono "Eliminar seleccionados"
    Then veo confirmación: "¿Eliminar 15 videos (total: 380 MB)? Esta acción no se puede deshacer."
    And al confirmar, los videos se eliminan de Drive
    And las filas correspondientes en Sheet "videos" se marcan como "eliminado" = TRUE
    And el espacio se libera inmediatamente
    
  Scenario: Subida desde galería (video pre-grabado)
    Given estoy en el form y seleccioné "Subir desde galería"
    When selecciono un archivo .mp4 de mi galería
    Then el sistema valida:
      | Validación            | Límite   | Mensaje de error                         |
      | Tamaño del archivo    | 100 MB   | "Video muy grande. Máximo 100 MB."       |
      | Formato               | MP4, MOV | "Formato no soportado. Usa MP4 o MOV."   |
      | Duración              | 3 min    | "Video muy largo. Máximo 3 minutos."     |
    
    And si pasa validación, procede con subida (mismo flujo que grabar)
    
  Scenario: Acceso a videos desde link externo
    Given quiero compartir un video con un asistente
    When hago click derecho en el icono 🎥 en el dashboard
    Then veo opción "Copiar link del video"
    And al hacer click, se copia a clipboard un link compartible:
      `https://drive.google.com/file/d/[ID]/view?usp=sharing`
    
    And el link tiene permisos de "Anyone with the link can view"
    And expira automáticamente en 7 días (configurable)
```

**Requisitos Técnicos:**

```markdown
### Arquitectura de Videos

**Almacenamiento:** Google Drive

**Estructura de carpetas (auto-generada):**

```
/CoachingSystem
  /Atletas
    /Juan_Perez_id_1
      /Videos
        /2026-01
          - squat_2026-01-15_10-00.mp4
          - bench_2026-01-17_10-30.mp4
        /2026-02
          - squat_2026-02-10_10-30.mp4
          - deadlift_2026-02-12_11-00.mp4
      /Docs
        - plan_actual.pdf
    /Maria_Garcia_id_2
      /Videos
        /2026-02
          ...
```

**Scripts necesarios:**

```javascript
// 1. Función para organizar video subido desde form
function organizarVideo(fileId, alumno_id, lift, timestamp) {
  try {
    // Obtener archivo del form upload
    var file = DriveApp.getFileById(fileId);
    
    // Obtener datos del atleta
    var atleta = obtenerDatosAtleta(alumno_id);
    var nombreCarpeta = atleta.nombre.replace(/ /g, "_") + "_id_" + alumno_id;
    
    // Navegar/crear estructura de carpetas
    var raiz = obtenerOCrearCarpeta(DriveApp.getRootFolder(), "CoachingSystem");
    var atletasFolder = obtenerOCrearCarpeta(raiz, "Atletas");
    var atletaFolder = obtenerOCrearCarpeta(atletasFolder, nombreCarpeta);
    var videosFolder = obtenerOCrearCarpeta(atletaFolder, "Videos");
    
    // Carpeta del mes actual
    var fechaObj = new Date(timestamp);
    var mesAnio = Utilities.formatDate(fechaObj, Session.getScriptTimeZone(), "yyyy-MM");
    var mesFolder = obtenerOCrearCarpeta(videosFolder, mesAnio);
    
    // Renombrar archivo
    var nuevoNombre = lift.toLowerCase() + "_" + 
      Utilities.formatDate(fechaObj, Session.getScriptTimeZone(), "yyyy-MM-dd_HH-mm") + 
      ".mp4";
    
    file.setName(nuevoNombre);
    
    // Mover a carpeta correcta
    mesFolder.addFile(file);
    DriveApp.getRootFolder().removeFile(file); // Remover de ubicación original
    
    // Configurar permisos
    file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
    
    // Guardar metadata en Sheet
    guardarVideoMetadata({
      alumno_id: alumno_id,
      nombre: atleta.nombre,
      lift: lift,
      fecha: timestamp,
      url: file.getUrl(),
      file_id: file.getId(),
      size_mb: file.getSize() / (1024 * 1024),
      revisado: false,
      feedback: "",
      eliminado: false
    });
    
    return {
      success: true,
      url: file.getUrl(),
      file_id: file.getId()
    };
    
  } catch (error) {
    Logger.log("Error organizando video: " + error);
    return {
      success: false,
      error: error.toString()
    };
  }
}

// 2. Función para obtener o crear carpeta
function obtenerOCrearCarpeta(parent, nombre) {
  var folders = parent.getFoldersByName(nombre);
  
  if (folders.hasNext()) {
    return folders.next();
  } else {
    return parent.createFolder(nombre);
  }
}

// 3. Función para guardar metadata de video
function guardarVideoMetadata(data) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("videos");
  
  if (!sheet) {
    sheet = SpreadsheetApp.getActiveSpreadsheet().insertSheet("videos");
    sheet.appendRow([
      "video_id", "alumno_id", "nombre", "lift", "fecha", "url", 
      "file_id", "size_mb", "revisado", "feedback", "eliminado", "created_at"
    ]);
  }
  
  sheet.appendRow([
    Utilities.getUuid(),
    data.alumno_id,
    data.nombre,
    data.lift,
    data.fecha,
    data.url,
    data.file_id,
    data.size_mb,
    data.revisado,
    data.feedback,
    data.eliminado,
    new Date()
  ]);
}

// 4. Función para calcular uso de almacenamiento
function calcularUsoAlmacenamiento(gym_id) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("videos");
  var data = sheet.getDataRange().getValues();
  
  var totalSize = 0;
  var totalVideos = 0;
  var porAtleta = {};
  
  for (var i = 1; i < data.length; i++) {
    if (data[i][10] === true) continue; // Skip eliminados
    
    var size = data[i][7]; // size_mb
    var atleta = data[i][2]; // nombre
    
    totalSize += size;
    totalVideos++;
    
    if (!porAtleta[atleta]) {
      porAtleta[atleta] = {count: 0, size: 0};
    }
    porAtleta[atleta].count++;
    porAtleta[atleta].size += size;
  }
  
  return {
    total_mb: totalSize,
    total_gb: totalSize / 1024,
    total_videos: totalVideos,
    promedio_mb: totalSize / totalVideos,
    por_atleta: porAtleta,
    limite_gb: 5 // Starter tier
  };
}

// 5. Función para eliminar videos antiguos
function eliminarVideosAntiguos(meses) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet()
    .getSheetByName("videos");
  var data = sheet.getDataRange().getValues();
  
  var fechaLimite = new Date();
  fechaLimite.setMonth(fechaLimite.getMonth() - meses);
  
  var eliminados = 0;
  var espacioLiberado = 0;
  
  for (var i = 1; i < data.length; i++) {
    var fecha = new Date(data[i][4]);
    var revisado = data[i][8];
    var eliminado = data[i][10];
    var fileId = data[i][6];
    
    if (fecha < fechaLimite && revisado && !eliminado) {
      try {
        // Eliminar de Drive
        var file = DriveApp.getFileById(fileId);
        var size = file.getSize() / (1024 * 1024); // MB
        file.setTrashed(true);
        
        // Marcar como eliminado en Sheet
        sheet.getRange(i + 1, 11).setValue(true); // columna "eliminado"
        
        eliminados++;
        espacioLiberado += size;
      } catch (e) {
        Logger.log("Error eliminando file " + fileId + ": " + e);
      }
    }
  }
  
  return {
    videos_eliminados: eliminados,
    espacio_liberado_mb: espacioLiberado
  };
}

// 6. Validación de video antes de subir (client-side JS)
function validarVideo(file) {
  // Tamaño
  var maxSize = 100 * 1024 * 1024; // 100 MB
  if (file.size > maxSize) {
    return {
      valid: false,
      error: "Video muy grande. Máximo 100 MB."
    };
  }
  
  // Formato
  var allowedFormats = ['video/mp4', 'video/quicktime'];
  if (!allowedFormats.includes(file.type)) {
    return {
      valid: false,
      error: "Formato no soportado. Usa MP4 o MOV."
    };
  }
  
  return {valid: true};
}
```

**Frontend (HTML form) - Subida de video:**

```html
<!-- En form/renderForm.html -->

<div id="video-section" style="display: none;">
  <label>
    <input type="checkbox" id="upload-video-toggle" onchange="toggleVideoUpload()">
    ¿Subir video de ejecución?
  </label>
  
  <div id="video-upload-controls" style="display: none;">
    <button type="button" onclick="grabarVideo()">
      📹 Grabar ahora
    </button>
    <button type="button" onclick="document.getElementById('file-input').click()">
      📁 Subir desde galería
    </button>
    
    <input type="file" id="file-input" accept="video/mp4,video/quicktime" 
           style="display: none;" onchange="handleFileSelect(event)">
    
    <div id="upload-progress" style="display: none;">
      <progress id="progress-bar" value="0" max="100"></progress>
      <span id="progress-text">0%</span>
    </div>
    
    <div id="upload-success" style="display: none;">
      ✓ Video subido exitosamente
    </div>
  </div>
</div>

<script>
function toggleVideoUpload() {
  var controls = document.getElementById('video-upload-controls');
  var checked = document.getElementById('upload-video-toggle').checked;
  controls.style.display = checked ? 'block' : 'none';
}

function grabarVideo() {
  // Abrir cámara (usando MediaRecorder API)
  navigator.mediaDevices.getUserMedia({video: true, audio: false})
    .then(function(stream) {
      // Mostrar preview y controles de grabación
      // ... implementación completa en archivo separado
    })
    .catch(function(err) {
      alert("No se pudo acceder a la cámara: " + err);
    });
}

function handleFileSelect(event) {
  var file = event.target.files[0];
  
  // Validar
  var validation = validarVideo(file);
  if (!validation.valid) {
    alert(validation.error);
    return;
  }
  
  // Subir a Drive
  subirVideoADrive(file);
}

function subirVideoADrive(file) {
  var reader = new FileReader();
  
  reader.onload = function(e) {
    var content = e.target.result;
    
    // Llamar a Apps Script endpoint
    google.script.run
      .withSuccessHandler(onUploadSuccess)
      .withFailureHandler(onUploadError)
      .uploadVideo(content, file.name);
    
    // Mostrar progress
    document.getElementById('upload-progress').style.display = 'block';
    
    // Simular progreso (real progress tracking requiere API más compleja)
    simulateProgress();
  };
  
  reader.readAsArrayBuffer(file);
}

function simulateProgress() {
  var progress = 0;
  var interval = setInterval(function() {
    progress += 10;
    document.getElementById('progress-bar').value = progress;
    document.getElementById('progress-text').textContent = progress + '%';
    
    if (progress >= 90) {
      clearInterval(interval);
    }
  }, 500);
}

function onUploadSuccess(response) {
  document.getElementById('upload-progress').style.display = 'none';
  document.getElementById('upload-success').style.display = 'block';
  
  // Guardar URL del video en campo oculto del form
  document.getElementById('video-url-hidden').value = response.url;
  document.getElementById('video-file-id-hidden').value = response.file_id;
}

function onUploadError(error) {
  alert("Error subiendo video: " + error);
  document.getElementById('upload-progress').style.display = 'none';
}
</script>
```

**Estructura de datos:**

Sheet: `videos`
| video_id | alumno_id | nombre       | lift     | fecha               | url                          | file_id       | size_mb | revisado | feedback                | eliminado | created_at          |
|----------|-----------|--------------|----------|---------------------|------------------------------|---------------|---------|----------|-------------------------|-----------|---------------------|
| UUID     | 1         | Juan Pérez   | Squat    | 2026-02-10 10:30:00 | https://drive.google.com/... | abc123        | 28.5    | TRUE     | Buena profundidad...    | FALSE     | 2026-02-10 10:32:00 |

**Scripts a generar:**
- `videos/organizarVideo.js`
- `videos/uploadVideo.js` (backend)
- `videos/validarVideo.js` (client-side)
- `videos/calcularUsoAlmacenamiento.js`
- `videos/eliminarVideosAntiguos.js`
- `videos/grabarVideoRecorder.js` (MediaRecorder wrapper)
```

---

## 🎯 Entregables Finales del Starter Pack

Basándote en todas las especificaciones anteriores, genera:

### 1. **Documento de Arquitectura**
- Diagrama de flujo completo del sistema
- Diagrama entidad-relación de los Sheets
- Diagrama de secuencia para cada flujo principal (onboarding, registro sesión, comparación, alertas)

### 2. **Código Completo**
Todos los archivos Apps Script organizados en carpetas:
```
/starter-pack
  /onboarding
    - createGymStructure.js
    - copyDashboardTemplate.js
    - setupTriggers.js
    - sendWelcomeEmail.js
  /form
    - renderForm.html
    - submitSession.js
    - getAthleteData.js
    - validations.js
  /comparison
    - onFormSubmit.js
    - buscarSesionProgramada.js
    - aplicarMatrizDecision.js
    - guardarComparacion.js
  /notifications
    - enviarResumenDiario.js
    - enviarResumenSemanal.js
    - analizarTendencias.js
    - crearAlerta.js
    - templates/
      - resumen_diario.html
      - resumen_semanal.html
      - alerta_inmediata.html
  /videos
    - organizarVideo.js
    - uploadVideo.js
    - calcularUsoAlmacenamiento.js
    - eliminarVideosAntiguos.js
  /utils
    - helpers.js
    - constants.js
```

### 3. **Templates de Google Sheets**
Archivo .xlsx con todas las hojas pre-configuradas:
- config_gimnasio
- config_notificaciones
- atletas
- planes_disponibles
- programa_531
- programa_BBB
- programa_GZCL
- sesiones_programadas
- registro_real
- comparacion
- videos
- alertas_pendientes

Con:
- Headers en negrita
- Validaciones de datos
- Formato condicional básico
- Fórmulas donde corresponda

### 4. **Template de Looker Studio**
Link a dashboard duplicable con:
- Todas las páginas configuradas (Overview, Individual, Alertas, Tendencias)
- Data sources pre-conectados
- Filtros configurados
- Estilo visual coherente

### 5. **Documentación de Usuario**
Markdown files:
- `INSTALLATION.md` - Guía de instalación paso a paso
- `USER_GUIDE.md` - Manual de uso para coaches
- `FAQ.md` - Preguntas frecuentes
- `TROUBLESHOOTING.md` - Solución de problemas comunes

### 6. **Videos Tutoriales** (guión escrito)
Scripts para grabar:
1. Onboarding completo (5 min)
2. Registrar primera sesión (2 min)
3. Leer el dashboard (3 min)
4. Gestionar alertas (2 min)
5. Subir y revisar videos (2 min)

---

## 📐 Principios de Diseño a Respetar

1. **Mobile-first:** Todo debe funcionar perfecto en tablet/smartphone
2. **Simplicidad:** Un coach sin conocimientos técnicos debe poder usarlo solo
3. **Feedback inmediato:** Cada acción debe tener respuesta visual clara
4. **Tolerancia a errores:** Validaciones que previenen, no que castigan
5. **Consistencia:** Mismo lenguaje, mismos colores, mismos patrones en todo el sistema
6. **Performance:** <3 segundos para cualquier operación
7. **Escalabilidad:** Debe funcionar igual con 1 atleta que con 15

---

## ✅ Checklist de Validación

Antes de entregar el Starter Pack, verificar que:

- [ ] Onboarding completo en <30 minutos sin ayuda
- [ ] Form funciona offline (con sync posterior)
- [ ] Comparación genera flags correctos en 100% de casos
- [ ] Dashboards cargan en <5 segundos
- [ ] Emails se envían en horario configurado
- [ ] Videos se organizan correctamente en Drive
- [ ] No hay errores en consola de Apps Script
- [ ] Todos los triggers están configurados
- [ ] Documentación cubre todos los flujos
- [ ] Sistema funciona en Chrome, Safari, Firefox
- [ ] Responsive funciona en iPhone, iPad, Android

---

**Ahora generá el sistema completo.**
```

---

¿Querés que lo ejecute y empiece a generar el código, o preferís primero revisar/ajustar algo del prompt?
