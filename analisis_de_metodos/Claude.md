# 🔥 SÍ a todo. Y te lo explico técnicamente.

---

## 📊 Múltiples planes + IDs + Videos = Sistema escalable

Tu idea es **perfecta** porque:
1. No todos hacen 5/3/1 → necesitás identificar qué plan sigue cada uno
2. Los videos son oro para coaching remoto o corrección técnica
3. Todo centralizado en Drive = accesible desde cualquier lado

---

## 🏗️ Arquitectura del sistema actualizada

### **HOJA 1: `atletas`** (ahora con plan asignado)

```
id | nombre         | email               | plan_id | squat_tm | bench_tm | dead_tm | press_tm | activo | fecha_inicio
1  | Juan Pérez     | juan@mail.com       | 531     | 150      | 100      | 180     | 70       | TRUE   | 2026-01-15
2  | María García   | maria@mail.com      | BBB     | 100      | 60       | 120     | 45       | TRUE   | 2026-01-20
3  | Carlos López   | carlos@mail.com     | 531     | 120      | 80       | 150     | 60       | TRUE   | 2026-01-15
4  | Lucía Díaz     | lucia@mail.com      | GZCL    | 90       | 55       | 110     | 40       | TRUE   | 2026-02-01
5  | Roberto Sánchez| roberto@mail.com    | 531     | 140      | 95       | 170     | 65       | TRUE   | 2026-01-15
```

**Columna nueva: `plan_id`**
- Identifica qué programa sigue
- Permite tener varios planes corriendo en paralelo

---

### **HOJA 2: `planes_disponibles`** (catálogo de programas)

```
plan_id | nombre_plan        | descripcion                          | ciclo_duracion_semanas | activo
531     | 5/3/1 Wendler      | Progresión lineal 4 semanas         | 4                      | TRUE
BBB     | Boring But Big     | 5/3/1 + 5x10 accesorios             | 4                      | TRUE
GZCL    | GZCL Method        | Tier system con autoregulación      | 3                      | TRUE
TEXAS   | Texas Method       | Volumen/Recovery/Intensidad         | 1                      | TRUE
```

**Por qué esto es útil:**
- En el Form, el coach selecciona "Juan Pérez" → el sistema ya sabe que Juan hace 5/3/1
- Si cambias a alguien de plan, solo modificás el `plan_id` en la hoja `atletas`

---

### **HOJA 3: `programa_531`** (template específico de 5/3/1)

```
plan_id | ciclo | semana | dia | lift   | set_num | intensidad | reps_objetivo | tipo_set
531     | 1     | 1      | 1   | Squat  | 1       | 0.65       | 5             | warmup
531     | 1     | 1      | 1   | Squat  | 2       | 0.75       | 5             | work
531     | 1     | 1      | 1   | Squat  | 3       | 0.85       | 5+            | amrap
...
```

### **HOJA 4: `programa_BBB`** (template de Boring But Big)

```
plan_id | ciclo | semana | dia | lift   | set_num | intensidad | reps_objetivo | tipo_set
BBB     | 1     | 1      | 1   | Squat  | 1       | 0.65       | 5             | warmup
BBB     | 1     | 1      | 1   | Squat  | 2       | 0.75       | 5             | work
BBB     | 1     | 1      | 1   | Squat  | 3       | 0.85       | 5+            | amrap
BBB     | 1     | 1      | 1   | Squat  | 4       | 0.50       | 10            | bbb_vol
BBB     | 1     | 1      | 1   | Squat  | 5       | 0.50       | 10            | bbb_vol
...
```

👉 Cada plan tiene su propia hoja con su estructura

---

### **HOJA 5: `sesiones_programadas`** (SE GENERA AUTOMÁTICA)

Ahora esta hoja une:
- `atletas` (quién)
- `planes_disponibles` (qué plan sigue)
- `programa_[plan_id]` (qué toca hoy)

```
alumno_id | nombre       | plan_id | ciclo | semana | dia | lift   | set_num | peso_programado | reps_objetivo
1         | Juan Pérez   | 531     | 1     | 1      | 1   | Squat  | 3       | 127.5          | 5+
2         | María García | BBB     | 1     | 1      | 1   | Squat  | 4       | 50             | 10
4         | Lucía Díaz   | GZCL    | 1     | 1      | 1   | Squat  | 1       | 81             | 10
```

**Apps Script para generar esto:**

```javascript
function generarSesionesProgramadas() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var atletas = ss.getSheetByName("atletas").getDataRange().getValues();
  var programadas = ss.getSheetByName("sesiones_programadas");
  
  programadas.clear();
  programadas.appendRow(["alumno_id", "nombre", "plan_id", "ciclo", "semana", "dia", "lift", "set_num", "peso_programado", "reps_objetivo"]);
  
  for (var i = 1; i < atletas.length; i++) {
    if (!atletas[i][8]) continue; // si no está activo, skip
    
    var alumno_id = atletas[i][0];
    var nombre = atletas[i][1];
    var plan_id = atletas[i][3];
    
    // Obtener template del plan
    var templateSheet = ss.getSheetByName("programa_" + plan_id);
    if (!templateSheet) continue;
    
    var template = templateSheet.getDataRange().getValues();
    
    for (var j = 1; j < template.length; j++) {
      var ciclo = template[j][1];
      var semana = template[j][2];
      var dia = template[j][3];
      var lift = template[j][4];
      var set_num = template[j][5];
      var intensidad = template[j][6];
      var reps_objetivo = template[j][7];
      
      // Obtener TM del atleta para ese lift
      var tm = obtenerTM(atletas[i], lift);
      var peso_programado = tm * intensidad;
      
      programadas.appendRow([
        alumno_id,
        nombre,
        plan_id,
        ciclo,
        semana,
        dia,
        lift,
        set_num,
        peso_programado,
        reps_objetivo
      ]);
    }
  }
}

function obtenerTM(atletaRow, lift) {
  // atletaRow[4] = squat_tm, [5] = bench_tm, [6] = dead_tm, [7] = press_tm
  switch(lift.toLowerCase()) {
    case "squat": return atletaRow[4];
    case "bench": return atletaRow[5];
    case "deadlift": return atletaRow[6];
    case "press": return atletaRow[7];
    default: return 0;
  }
}
```

---

## 📱 Google Form actualizado (con selección de plan)

### **Campos del Form:**

1. **Atleta** (dropdown dinámico con los 15 nombres)
2. **Plan** (se autocompleta según el atleta seleccionado - campo oculto)
3. **Lift** (dropdown: Squat, Bench, Deadlift, Press)
4. **Peso levantado** (número)
5. **Reps completadas** (número)
6. **RPE** (escala 1-10)
7. **¿Subir video de ejecución?** (Sí / No)
8. **Video** (campo de archivo - SOLO aparece si dice "Sí")
9. **Observaciones** (texto libre)

---

## 📂 Estructura de Google Drive

```
/CoachingSystem
  /Atletas
    /Juan_Perez (id_1)
      /Videos
        /2026-02
          - squat_2026-02-10_10-30.mp4
          - deadlift_2026-02-12_11-15.mp4
      /Docs
        - historial_mediciones.pdf
        - plan_actual.pdf
    /Maria_Garcia (id_2)
      /Videos
        /2026-02
          - squat_2026-02-10_10-35.mp4
      /Docs
  /Programas
    - template_531.xlsx
    - template_BBB.xlsx
  /Reportes
    - resumen_semanal_2026-02-10.pdf
```

**Lógica de nomenclatura:**
- Carpeta por atleta (usando `id` para evitar problemas con nombres repetidos)
- Subcarpeta `/Videos` con estructura por año-mes
- Nombre de archivo: `[lift]_[fecha]_[hora].mp4`

---

## 🎥 Cómo funciona la subida de videos

### **Opción 1: Desde Google Form (más simple)**

**Configuración del Form:**

1. Campo "¿Subir video?" → Si/No
2. Campo "Video" (tipo: Upload de archivo)
   - Configurar que solo acepta: `.mp4`, `.mov`, `.avi`
   - Tamaño máximo: 100 MB
   - Destino: carpeta específica en Drive

**Apps Script para organizar el video:**

```javascript
function onFormSubmit(e) {
  var alumno_id = e.values[1];
  var lift = e.values[3];
  var subirVideo = e.values[7]; // "Sí" o "No"
  
  if (subirVideo == "Sí") {
    var videoFileId = e.values[8]; // ID del archivo subido
    
    if (videoFileId) {
      organizarVideo(alumno_id, lift, videoFileId);
    }
  }
  
  // ... resto del código (guardar datos, calcular comparación, etc.)
}

function organizarVideo(alumno_id, lift, videoFileId) {
  // Obtener archivo del Form
  var file = DriveApp.getFileById(videoFileId);
  
  // Obtener nombre del atleta
  var nombre = obtenerNombreAtleta(alumno_id);
  var nombreCarpeta = nombre.replace(/ /g, "_") + "_id_" + alumno_id;
  
  // Buscar o crear carpeta del atleta
  var carpetaRaiz = DriveApp.getFoldersByName("CoachingSystem").next();
  var carpetaAtletas = carpetaRaiz.getFoldersByName("Atletas").next();
  
  var carpetaAtleta = buscarOCrearCarpeta(carpetaAtletas, nombreCarpeta);
  var carpetaVideos = buscarOCrearCarpeta(carpetaAtleta, "Videos");
  
  // Crear carpeta del mes actual
  var hoy = new Date();
  var mesAnio = Utilities.formatDate(hoy, "GMT-3", "yyyy-MM");
  var carpetaMes = buscarOCrearCarpeta(carpetaVideos, mesAnio);
  
  // Renombrar archivo
  var timestamp = Utilities.formatDate(hoy, "GMT-3", "yyyy-MM-dd_HH-mm");
  var nuevoNombre = lift.toLowerCase() + "_" + timestamp + ".mp4";
  
  // Mover y renombrar
  file.setName(nuevoNombre);
  file.moveTo(carpetaMes);
  
  // Guardar URL en Sheet
  guardarURLVideo(alumno_id, lift, hoy, file.getUrl());
}

function buscarOCrearCarpeta(carpetaPadre, nombre) {
  var carpetas = carpetaPadre.getFoldersByName(nombre);
  
  if (carpetas.hasNext()) {
    return carpetas.next();
  } else {
    return carpetaPadre.createFolder(nombre);
  }
}

function guardarURLVideo(alumno_id, lift, fecha, url) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var videos = ss.getSheetByName("videos");
  
  if (!videos) {
    videos = ss.insertSheet("videos");
    videos.appendRow(["alumno_id", "lift", "fecha", "url"]);
  }
  
  videos.appendRow([alumno_id, lift, fecha, url]);
}
```

---

### **Opción 2: El atleta sube desde su teléfono (avanzado)**

**Flujo:**
1. El atleta recibe un link único a una carpeta compartida en Drive
2. Graba video en su teléfono
3. Lo sube directo a Drive desde la app móvil
4. El sistema detecta el nuevo archivo y lo organiza automáticamente

**Apps Script (trigger cuando se sube archivo):**

```javascript
function onFileUpload(e) {
  // Detectar nuevo archivo en carpeta compartida
  var carpetaShared = DriveApp.getFolderById("ID_CARPETA_SHARED");
  var archivos = carpetaShared.getFiles();
  
  while (archivos.hasNext()) {
    var file = archivos.next();
    var nombre = file.getName();
    
    // Parsear nombre (debe ser: alumno_id_lift_fecha.mp4)
    var partes = nombre.split("_");
    
    if (partes.length >= 3) {
      var alumno_id = partes[0];
      var lift = partes[1];
      
      // Mover a carpeta del atleta
      organizarVideo(alumno_id, lift, file.getId());
      
      // Eliminar de carpeta compartida
      carpetaShared.removeFile(file);
    }
  }
}
```

---

## 📊 Nueva hoja: `videos`

```
alumno_id | nombre       | lift    | fecha               | url                                    | revisado | feedback
1         | Juan Pérez   | Squat   | 2026-02-10 10:30:00 | https://drive.google.com/file/d/...   | FALSE    |
1         | Juan Pérez   | Deadlift| 2026-02-12 11:15:00 | https://drive.google.com/file/d/...   | TRUE     | Mejorar setup
2         | María García | Squat   | 2026-02-10 10:35:00 | https://drive.google.com/file/d/...   | FALSE    |
```

**Para qué sirve:**
- Trackear qué videos tenés sin revisar
- Dejar feedback escrito
- Cruzar con datos de rendimiento (¿el video malo coincide con RPE alto?)

---

## 📈 Looker Studio: Panel de videos

**Vista nueva en el dashboard:**

**Tabla interactiva:**
```
Atleta      | Lift    | Fecha      | RPE | Reps | Video               | Revisado
Juan Pérez  | Squat   | 2026-02-10 | 8   | 7    | [Ver video 🎥]      | ❌
María García| Squat   | 2026-02-10 | 7.5 | 6    | [Ver video 🎥]      | ❌
Carlos López| Deadlift| 2026-02-12 | 9   | 3    | [Ver video 🎥]      | ✅
```

**Filtros:**
- Solo mostrar videos sin revisar
- Filtrar por atleta
- Filtrar por RPE >8 (priorizar correcciones en lifts difíciles)

---

## 🎯 Flujo completo con videos

### **Lunes 10:30 AM - Juan hace su AMRAP de Squat**

1. Vos abrís el Form en la tablet
2. Seleccionás: "Juan Pérez"
3. El sistema autocompleta: Plan = "5/3/1"
4. Ingresás:
   - Lift: Squat
   - Peso: 127.5 kg
   - Reps: 7
   - RPE: 8
5. Juan te dice: "¿Podés revisar mi técnica?"
6. Marcas: **"¿Subir video? Sí"**
7. Juan graba con su teléfono, te manda el video por WhatsApp
8. Vos lo subís desde la tablet al Form
9. Submit

**Lo que pasa automáticamente:**
- Datos van a `registro_real`
- Video se sube a Drive → `/Atletas/Juan_Perez_id_1/Videos/2026-02/squat_2026-02-10_10-30.mp4`
- URL se guarda en hoja `videos`
- En Looker, aparece una nueva fila en "Videos sin revisar"

---

### **Martes por la noche - Revisás videos en tu casa**

1. Abrís Looker Studio
2. Filtras: "Videos sin revisar"
3. Ves:
   - Juan - Squat - RPE 8 - [Ver video 🎥]
   - Carlos - Deadlift - RPE 9 - [Ver video 🎥]
4. Clickeás en "Ver video", se abre en Drive
5. Ves que Carlos está redondeando la espalda
6. Abrís el Sheet, hoja `videos`, en la fila de Carlos escribís:
   - `revisado = TRUE`
   - `feedback = "Espalda redonda en setup. Revisar posición inicial"`
7. Al día siguiente le mandás el feedback por WhatsApp con timestamp del video

---

## ✅ Resumen técnico

| **Componente** | **Función** | **Dónde vive** |
|---|---|---|
| `atletas` | Quién entrena + qué plan sigue | Google Sheets |
| `planes_disponibles` | Catálogo de programas | Google Sheets |
| `programa_[plan_id]` | Template de cada plan | Google Sheets (1 hoja por plan) |
| `sesiones_programadas` | Qué debería pasar hoy | Sheets (generada por script) |
| `registro_real` | Qué pasó realmente | Sheets (llenada por Form) |
| `videos` | URLs de videos + feedback | Google Sheets |
| Google Form | Interfaz de carga | Google Forms |
| Apps Script | Automatización | Vinculado al Sheet |
| Drive | Almacenamiento de videos | `/CoachingSystem/Atletas/...` |
| Looker Studio | Visualización + acceso a videos | Conectado a Sheets |

---

## 💡 Bonus: Notificaciones automáticas

**Apps Script para avisarte cuando hay video nuevo:**

```javascript
function enviarNotificacionVideoNuevo(alumno_nombre, lift) {
  var mensaje = "🎥 Nuevo video: " + alumno_nombre + " - " + lift + "
Revisar en Looker Studio";
  
  // Opción 1: Email
  MailApp.sendEmail("tu_email@gmail.com", "Video nuevo para revisar", mensaje);
  
  // Opción 2: Telegram (necesita bot configurado)
  enviarTelegram(mensaje);
  
  // Opción 3: WhatsApp Business API (más complejo)
}
```

---

**¿Querés que ahora armemos el Sheet completo con esta estructura, o te explico primero cómo configurar el Google Form para que acepte videos?**


Roadmap: De MVP gratis a SaaS de $350/mes

🎯 Estrategia de Monetización
Modelo de 3 Tiers
TierPrecioTargetLímitesStarter$49/mesCoaches independientes1-15 atletas, 1 coachPro$149/mesBoxes pequeños / Studios16-50 atletas, hasta 3 coachesElite$349/mesGimnasios / Centros deportivos51-200 atletas, coaches ilimitados
O modelo anual con descuento:

Starter: $490/año ($40/mes)
Pro: $1,490/año ($124/mes)
Elite: $3,490/año ($291/mes)


📊 Feature Matrix (qué incluye cada tier)
FeatureStarterProEliteCore FeaturesTracking de sesiones✅✅✅Múltiples planes (5/3/1, BBB, GZCL, etc.)✅✅✅Comparación real vs programado✅✅✅Detección automática de alertas✅✅✅Dashboard en Looker StudioBásicoAvanzadoPremiumLímitesAtletas activos1550200Coaches13IlimitadosPlanes personalizados25IlimitadosAlmacenamiento de videos5 GB50 GB500 GBAnalytics & IAAnálisis de tendencias básico✅✅✅Predicción de 1RM con IA❌✅✅Recomendación automática de TM❌✅✅Análisis de video con IA (form check)❌❌✅Detección de riesgo de lesión❌❌✅AutomatizaciónEnvío de reportes semanalesEmailEmail + WhatsAppEmail + WhatsApp + SMSRecordatorios a atletas❌✅✅Ajuste automático de TM❌SugerenciasAuto-implementadoGeneración de ciclos futurosManualSemi-autoCompletamente autoColaboraciónNotas compartidas entre coaches❌✅✅Sistema de tareas/follow-ups❌✅✅Chat integrado con atletas❌❌✅Branding & White LabelLogo del gimnasio en reportes❌✅✅Dominio personalizado❌❌✅App móvil con tu marca❌❌✅IntegracionesGoogle Drive✅✅✅WhatsApp API❌✅✅Stripe (cobro a atletas)❌❌✅Calendly (agendamiento)❌✅✅MyFitnessPal / Cronometer❌❌✅SoporteDocumentación + videos✅✅✅Soporte por email48hs24hs12hsOnboarding personalizado❌1 sesión3 sesiones + implementaciónSoporte telefónico❌❌✅

🚀 Lista de Mejoras por Prioridad
Fase 1: MVP → Starter ($49/mes)
Lo que ya tenemos + mejoras básicas
1.1 Core Product (ya lo tenemos 90%)

✅ Sistema de tracking multi-atleta
✅ Múltiples planes (5/3/1, BBB, GZCL)
✅ Google Form + Sheets + Apps Script
✅ Comparación real vs programado
✅ Dashboards en Looker Studio
✅ Almacenamiento de videos en Drive

1.2 Mejoras críticas para venderlo

 Onboarding automatizado:

Wizard que copia template del Sheet
Script que crea carpetas en Drive
Email de bienvenida con tutorial en video
Checklist de setup (≈ 2 semanas dev)


 Interfaz mejorada del Form:

Form responsive (mobile-first)
Campos inteligentes (autocompleta plan según atleta)
Preview del peso programado antes de submit
Validación en tiempo real (≈ 1 semana dev)


 Dashboard básico mejorado:

Template de Looker Studio branded
Filtros por atleta/fecha/plan
Gráficos de progresión claros
KPIs principales (compliance rate, progresión promedio)
(≈ 1 semana design + 1 semana dev)


 Sistema de alertas por email:

Email diario con resumen de alertas
HTML template profesional
Botón para ver detalles en dashboard
(≈ 3 días dev)


 Documentación básica:

Manual de usuario en Notion/Google Sites
5 videos tutoriales (setup, carga de datos, lectura de dashboards, ajuste de TM, troubleshooting)
FAQ
(≈ 1 semana contenido)



Esfuerzo total Fase 1: ~6 semanas
ROI: Producto vendible a coaches independientes

Fase 2: Starter → Pro ($149/mes)
Features que justifican 3x el precio
2.1 Multi-coach & Roles

 Sistema de permisos:

Admin (dueño): acceso total
Coach: solo sus atletas asignados
Asistente: solo lectura
(≈ 2 semanas dev)


 Asignación de atletas a coaches:

Un atleta puede tener coach principal + asistentes
Notificaciones solo al coach asignado
Dashboard filtrado por coach
(≈ 1 semana dev)



2.2 IA para predicción de rendimiento

 Predicción de 1RM:

Modelo ML entrenado con datos de lifters
Input: últimos 5 AMRAPs + RPE
Output: 1RM estimado ± margen de error
(≈ 3 semanas dev: 1 semana data, 1 semana modelo, 1 semana integración)


 Recomendación inteligente de TM:

Analiza últimos 4 ciclos
Cruza reps en AMRAP + RPE + progresión histórica
Sugiere: mantener / +2.5kg / +5kg / -5% / deload
Explicación del por qué
(≈ 2 semanas dev)


 Detección de fatiga acumulada:

Algoritmo que detecta patrones:

RPE subiendo sin aumento de volumen
Ratio volumen/RPE cayendo 3+ semanas
Fallas en completar reps 2+ veces/semana


Alerta temprana + sugerencia de deload
(≈ 1 semana dev)



2.3 Automatización avanzada

 Auto-generación de próximo ciclo:

Al finalizar ciclo, el sistema genera el siguiente
Aplica ajuste de TM recomendado
Notifica al coach para aprobar/modificar
(≈ 2 semanas dev)


 Recordatorios a atletas (WhatsApp):

Integración con WhatsApp Business API
"Hoy toca Squat - Semana 2 - 3x3 @ 90%"
Recordatorio si no registró datos en 2 días
(≈ 2 semanas dev + $50/mes costo WhatsApp API)


 Templates de planes pre-cargados:

Librería de 10+ programas populares:

5/3/1, BBB, FSL, SSL
Texas Method
GZCL (General, Jacked & Tan 2.0)
Candito 6-week
nSuns
Madcow 5x5


1-click para asignar a un atleta
(≈ 2 semanas creación de templates)



2.4 Analytics Pro

 Dashboard avanzado:

Heatmap de compliance (quién entrena vs quién falta)
Análisis de volumen por grupo muscular
Comparación inter-atletas (benchmarking)
Proyección de 1RM futuro
(≈ 2 semanas dev)


 Exportación de reportes:

PDF con análisis mensual de cada atleta
Gráficos de progresión
Recomendaciones personalizadas
Branding del gimnasio
(≈ 1 semana dev)



Esfuerzo total Fase 2: ~17 semanas
ROI: Producto para boxes/estudios con múltiples coaches
