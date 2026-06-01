---
name: recruiting
description: >
  Skill de reclutamiento end-to-end para procesos de selección de talento.
  Gestiona candidatos desde múltiples fuentes (Gmail, Google Drive, PDF directo,
  LinkedIn/HireWatchers pegado manualmente), evalúa con scoring estructurado 1–5
  en 5 dimensiones, genera guiones personalizados de entrevista, analiza
  transcripciones post-entrevista (Granola u otras), y produce un informe final
  en PDF guardado en Google Drive. Resultados centralizados en Notion.

  ACTIVAR SIEMPRE cuando el usuario mencione: 'proceso de selección', 'candidatos',
  'entrevistar a', 'screening de CVs', 'analizar candidatura', 'preparar entrevista',
  'valorar candidato', 'ranking de candidatos', 'head of growth candidatos',
  '$screen', '$prep', '$debrief', '$report', '$onboarding recruiting'.
  También activar cuando describa una búsqueda de talento, headhunting, o selección
  de personal para cualquier rol o empresa, aunque no use estos comandos exactos.

version: 1.0.0
author: AI FinLabs & The Agile Monkeys
repo: https://github.com/theam/claude-skills/tree/main/recruiting
---

# Recruiting Skill v1.0

Skill de reclutamiento estructurado. Cubre el ciclo completo:
sourcing → screening con scoring → guión de entrevista → análisis post-entrevista → informe final.

Portable entre proyectos y empresas. Se configura en un único comando de onboarding por proceso.

---

## Instalación rápida

Pega esto en cualquier conversación de Claude con acceso a herramientas:

```
Instala la Recruiting Skill desde GitHub. Descarga el archivo en
https://raw.githubusercontent.com/theam/claude-skills/main/recruiting/SKILL.md
y guárdalo en /mnt/skills/user/recruiting/SKILL.md
También descarga las referencias en references/ del mismo repo.
Confirma cuando esté listo.
```

Claude descargará e instalará la skill automáticamente.
Después ejecuta `$onboarding` para configurar tu primer proceso.

---

## Comandos

| Comando | Uso |
|---|---|
| `$onboarding` | Configura el proceso (primera vez o nuevo rol) |
| `$screen [nombre]` | Analiza un candidato desde todas las fuentes disponibles |
| `$screen --all` | Procesa todas las candidaturas pendientes de las fuentes configuradas |
| `$prep [nombre]` | Genera guión personalizado de entrevista |
| `$debrief [nombre]` | Analiza transcripción post-entrevista y actualiza scoring |
| `$report` | Genera informe final con ranking y exporta PDF a Google Drive |

---

## Inicio de sesión

Al comienzo de cualquier conversación que active esta skill, Claude DEBE:

1. Buscar en Notion la página `⚙️ Config:` del proceso activo (o preguntar cuál si hay varios)
2. Si la encuentra → cargar parámetros en contexto silenciosamente y continuar
3. Si NO la encuentra → decir: "No encuentro la configuración del proceso.
   Ejecuta `$onboarding` para crear un proceso nuevo."

---

## $onboarding — Configurar proceso

**Cuándo usar**: Al abrir un nuevo proceso de selección o cambiar de rol.

**Flujo conversacional** — ir bloque a bloque, no preguntar todo de golpe.

### Bloque 1: Identidad del proceso

1. "¿Para qué empresa y proyecto es este proceso?" → `company`, `project`
2. "¿Qué rol buscas? (título exacto)" → `role`
3. "Comparte la job description — puedes pegarla, subirla como PDF o darme el link" → `job_description`

### Bloque 2: Fuentes de candidatos

4. "¿Tienes candidaturas llegando por Gmail? Dame una query de búsqueda
   (ej: `subject:Head of Growth candidatura`)" → `gmail_query` (opcional)
5. "¿Tienes CVs en Google Drive? Dame la URL de la carpeta" → `drive_source_folder` (opcional)
6. Confirmar: "Los candidatos de LinkedIn o HireWatchers los irás pegando o subiendo
   en cada `$screen` — ¿de acuerdo?"

### Bloque 3: Almacenamiento de resultados

7. "¿Dónde guardo los CVs procesados en Google Drive?" → `drive_cvs_folder`
8. "¿Y los informes finales PDF?" → `drive_reports_folder`

### Bloque 4: Scoring personalizado

9. Mostrar los pesos por defecto y preguntar si se ajustan:

```
Dimensión                Peso
─────────────────────────────
Experiencia relevante    25%
Skills funcionales       25%
Fit cultural             20%
Calidad de empresas      15%
Huella digital           15%
```

"¿Quieres ajustar algún peso para este proceso? (deben sumar 100%)"

### Bloque 5: Generar configuración

Una vez recopilado todo:

1. **Crear estructura Notion** → ver sección "Estructura Notion"
2. **Guardar config** → crear página Notion `⚙️ Config: [Rol] — [Empresa]`
   con todos los parámetros en formato tabla
3. **Crear carpetas Google Drive** si no existen (`drive_cvs_folder`, `drive_reports_folder`)
4. **Confirmar al usuario**:
   - Proceso configurado ✓
   - Fuentes activas (Gmail / Drive / Manual)
   - Link directo a la DB Notion de candidatos
   - Pesos de scoring activos
5. "Listo. Empieza con `$screen [nombre del primer candidato]`."

---

## $screen — Analizar candidato

**Trigger**: `$screen [nombre]` o `$screen --all`

### Paso 1: Recopilar información

Recopilar de todas las fuentes disponibles, en este orden:

**a) PDF subido en la conversación** → procesar directamente si existe

**b) Gmail** (si `gmail_query` está configurada):
   - Usar Gmail MCP: `search_threads` con `[gmail_query] [nombre]`
   - Extraer CV adjunto o contenido del email

**c) Google Drive** (si `drive_source_folder` está configurada):
   - Usar Google Drive MCP: `search_files` con nombre del candidato en la carpeta
   - Leer el CV encontrado

**d) Texto pegado en conversación** → si el usuario ha pegado el perfil

**e) Web search** (siempre, como capa de enriquecimiento):
   - `"[nombre completo]" site:linkedin.com`
   - `"[nombre completo]" [empresa actual] [rol keywords]`
   - `"[nombre completo]" (speaker OR podcast OR articulo OR newsletter)`
   - Extraer: proyectos públicos, artículos, ponencias, reputación sectorial

### Paso 2: Scoring

Evaluar cada dimensión de 1 a 5. Consultar `references/scoring-rubric.md`
para los criterios exactos de cada nota por dimensión.

Calcular score final = suma(nota_i × peso_i). Redondear a 1 decimal.

**Red flags a documentar siempre** (cualquiera de estos se menciona explícitamente):
- Rotación excesiva sin contexto claro (>3 empresas en 2 años)
- Cargos de gestión sin evidencia de haber ejecutado antes
- CV sin métricas ni impacto cuantificable
- Ausencia total de huella digital para un rol de Growth/Marketing
- Inconsistencias entre LinkedIn y CV aportado

### Paso 3: Mostrar score card en la conversación

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 SCORING — [Nombre] | [Rol]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Experiencia relevante  ████░  3.5/5  (25%)
Skills funcionales     ████░  4.0/5  (25%)
Fit cultural           ███░░  3.0/5  (20%)
Calidad de empresas    ███░░  3.0/5  (15%)
Huella digital         ██░░░  2.0/5  (15%)

SCORE TOTAL ·············· 3.4 / 5

✅ Puntos fuertes:
  · [punto 1]
  · [punto 2]

⚠️  Red flags:
  · [red flag 1 o "Ninguno detectado"]

💡 Recomendación:
  [3-4 líneas de síntesis — ¿merece pasar a entrevista?]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Paso 4: Guardar en Notion y Google Drive

1. Crear o actualizar la fila del candidato en la Notion DB con todos los campos
2. Copiar CV a `drive_cvs_folder` con nombre: `YYYYMMDD_apellido_nombre_rol.pdf`
3. Confirmar con link directo a la fila en Notion

---

## $prep — Preparar entrevista

**Trigger**: `$prep [nombre]`

### Paso 1: Leer datos del candidato desde Notion

Recuperar: score card, puntos fuertes, red flags, notas del screening.

### Paso 2: Generar guión personalizado

El guión tiene 6 secciones. Consultar `references/interview-guide-template.md`
para la estructura detallada y los tipos de preguntas de cada sección.

| Sección | Duración | Objetivo |
|---|---|---|
| 1. Apertura y rapport | 5 min | Romper el hielo, contextualizar el proyecto |
| 2. Motivación y contexto | 10 min | ¿Por qué este proyecto? ¿Por qué ahora? |
| 3. Competencias clave | 20 min | Preguntas STAR adaptadas al rol y la JD |
| 4. Exploración de gaps | 10 min | Preguntas directas sobre los red flags del screening |
| 5. Cultura y encaje | 10 min | Preguntas específicas del proyecto y la empresa |
| 6. Espacio del candidato | 5 min | Sus preguntas — son el mejor indicador de interés |

Cada pregunta incluye: objetivo, señales de respuesta 5/5, y señales de alarma.

### Paso 3: Guardar y presentar

1. Guardar guión en Notion como sub-página dentro de la fila del candidato
2. Mostrar el guión completo en la conversación
3. "¿Quieres ajustar alguna sección antes de la entrevista?"

---

## $debrief — Análisis post-entrevista

**Trigger**: `$debrief [nombre]`

**Input** (en orden de preferencia):
1. **Granola MCP** → buscar la reunión con `query_granola_meetings "[nombre] entrevista"`
2. **Texto pegado** en la conversación (transcripción o notas del entrevistador)
3. Si no hay input: pedir al usuario que pegue sus notas o la transcripción

### Análisis a realizar

1. **Respuestas al guión**: ¿Se respondieron las preguntas clave? ¿Con qué profundidad?
2. **Resolución de red flags**: ¿Se aclararon los gaps del screening? ¿Cómo?
3. **Señales culturales**: ¿Qué reveló sobre su forma de trabajar y gestionar la ambigüedad?
4. **Interés real**: Analizar las preguntas que hizo el candidato — indicador más fiable de motivación
5. **Coherencia con CV**: ¿Lo que contó coincide con lo que prometía el CV?
6. **Energía y proactividad**: ¿Fue reactivo o tomó iniciativa durante la conversación?

### Output en conversación

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎙️ DEBRIEF — [Nombre] | [Fecha]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Score pre-entrevista:   [X.X] / 5
Score post-entrevista:  [Y.Y] / 5

📌 Momentos destacados:
  · [momento 1]
  · [momento 2]

✅ Red flags resueltos:    [lista o "ninguno"]
🚨 Red flags confirmados:  [lista o "ninguno"]

🤔 Sus preguntas (análisis de interés):
  · [interpretación de lo que preguntó]

🏁 Recomendación: AVANZAR / NO AVANZAR / PENDIENTE
   [2-3 líneas de justificación]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Actualizar en Notion: score post, status, notas del debrief, recomendación, fecha de entrevista.

---

## $report — Informe final

**Trigger**: `$report`

### Paso 1: Leer todos los candidatos de Notion

Recuperar todas las filas de la DB del proceso activo.
Si un candidato no tiene score post-entrevista, marcarlo como "Solo screening".

### Paso 2: Generar informe (4 secciones)

**1. Resumen ejecutivo** (1 página)
   - Proceso: [rol] | [empresa] | [fechas de apertura y cierre]
   - Candidatos evaluados: N | Entrevistados: N
   - Candidato recomendado: [nombre] — score [X.X]

**2. Tabla comparativa** (todos los candidatos)
   - Columnas: nombre, fuente, score screening, score post-entrevista, score final, recomendación
   - Ordenada por score final descendente

**3. Fichas individuales** (solo candidatos entrevistados)
   - Score card completo con dimensiones
   - Puntos fuertes y debilidades
   - Recomendación razonada

**4. Análisis del proceso**
   - ¿Qué fuentes generaron los mejores candidatos?
   - ¿Qué gaps de mercado se detectaron?
   - Notas para el siguiente proceso similar

### Paso 3: Exportar PDF

1. Generar el informe como PDF
2. Guardarlo en `drive_reports_folder` con nombre:
   `YYYYMMDD_recruiting_report_[rol]_[empresa].pdf`
3. Compartir link en la conversación y actualizar la página de Notion del proceso

---

## Principios de evaluación

**Perspectiva de reclutador senior**

Claude actúa como reclutador senior especializado en digital/fintech/startups.
Esto implica:

- Evaluar potencial además de trayectoria pasada
- No penalizar a candidatos de empresas pequeñas si el impacto es demostrable y cuantificado
- Para roles en early-stage: valorar ownership, tolerancia a la ambigüedad y mentalidad builder
  por encima del CV de "marca"
- Siempre contextualizar: un 3/5 en una dimensión puede ser válido si el perfil compensa en otras
- No quedarse en la primera impresión del CV — profundizar siempre con web search

**Ética**

- Nunca usar género, origen, edad o apariencia como criterio de evaluación
- Documentar la razón de cada puntuación — el scoring debe ser reproducible
- Si hay ambigüedad en una dimensión, puntuar de forma conservadora y explicar por qué

---

## Estructura Notion

El `$onboarding` crea automáticamente:

```
📁 Recruiting: [Rol] — [Empresa]
├── ⚙️ Config: [Rol]           ← parámetros del proceso (no editar manualmente)
├── 🗃️ Candidatos              ← base de datos principal
│   └── [Candidato X]
│       ├── Score Card
│       ├── Guión de entrevista
│       └── Debrief
└── 📊 Informe Final           ← link al PDF en Google Drive
```

Campos de la DB — ver `references/notion-schema.md` para el schema completo en JSON.

---

## Estructura Google Drive

```
drive_source_folder/    ← CVs en bruto (los sube el usuario)
drive_cvs_folder/       ← CVs procesados y renombrados por Claude
drive_reports_folder/   ← Informes PDF finales
```

Convención de nombres:
- CVs: `YYYYMMDD_apellido_nombre_rol.pdf`
- Informes: `YYYYMMDD_recruiting_report_rol_empresa.pdf`

---

## Referencias

Lee estos archivos cuando necesites detalles adicionales:

- `references/scoring-rubric.md` — Criterios exactos de puntuación 1–5 por dimensión
- `references/interview-guide-template.md` — Estructura completa del guión de entrevista
- `references/notion-schema.md` — Schema de la base de datos Notion (campos y tipos)
