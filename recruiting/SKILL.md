---
name: recruiting
description: >
  Skill de reclutamiento end-to-end para procesos de selección de talento.
  Gestiona candidatos desde múltiples fuentes (Gmail, Google Drive, PDF directo,
  LinkedIn/HireWatchers pegado manualmente), evalúa con scoring estructurado 1–5
  en 5 dimensiones, genera guiones personalizados de entrevista, analiza
  transcripciones post-entrevista (TAM-OS vía Zoom/ElevenLabs, Granola u otras),
  genera y evalúa pruebas/business cases, y produce un score final ponderado
  por etapa. Resultados centralizados en Notion.

  ACTIVAR SIEMPRE cuando el usuario mencione: 'proceso de selección', 'candidatos',
  'entrevistar a', 'screening de CVs', 'analizar candidatura', 'preparar entrevista',
  'valorar candidato', 'ranking de candidatos', 'business case candidato',
  '$screen', '$prep', '$debrief', '$case', '$case-eval', '$score-final',
  '$report', '$onboarding recruiting'.
  También activar cuando describa una búsqueda de talento, headhunting, o selección
  de personal para cualquier rol o empresa, aunque no use estos comandos exactos.

version: 1.3.0
author: AI FinLabs & The Agile Monkeys
repo: https://github.com/sergiocarbajal-cpu/claude-skills/tree/main/recruiting
changelog:
  1.2.0:
    - Nuevo comando $case: genera prueba/business case personalizado por candidato
    - Nuevo comando $case-eval: analiza entrega y asigna score con rúbrica dinámica
    - Nuevo comando $score-final: score compuesto ponderado por etapa del proceso
    - $prep y $debrief: detectan ronda automáticamente (1ª o 2ª entrevista)
    - Pesos por defecto del score final: screen 0% / debrief-1 30% / case 40% / debrief-2 30%
    - $onboarding Bloque 7: configuración de pesos del score final por etapa
    - notion-schema.md: campos Score Case, Fecha Case, Score Entrevista 2, Score Final Compuesto
  1.1.0:
    - $debrief: TAM-OS (Zoom→ElevenLabs) como fuente primaria de transcripciones
    - $debrief y $prep: contenido guardado en cuerpo de la página del candidato
    - $onboarding: derivación dinámica de red flags y señales positivas desde la JD
    - $screen, $prep, $debrief: usan señales del proceso cargadas desde Config
---

# Recruiting Skill v1.3

Skill de reclutamiento estructurado. Cubre el ciclo completo:

```
$screen → $prep → [entrevista 1] → $debrief
                                       ↓
                                   $case          ← genera prueba personalizada
                                       ↓
                             [candidato entrega]
                                       ↓
                                 $case-eval       ← analiza y puntúa
                                       ↓
                       $prep [ronda 2 auto]        ← guión 2ª entrevista
                                       ↓
                             [entrevista 2]
                                       ↓
                                 $debrief          ← detecta ronda 2 auto
                                       ↓
                               $score-final        ← score compuesto ponderado
```

Portable entre proyectos y empresas. Se configura en un único comando de onboarding por proceso.

---

## Instalación rápida

Pega esto en cualquier conversación de Claude con acceso a herramientas:

```
Instala la Recruiting Skill desde GitHub. Descarga el archivo en
https://raw.githubusercontent.com/sergiocarbajal-cpu/claude-skills/main/recruiting/SKILL.md
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
| `$screen [nombre] \| [fuente] \| [via]` | Analiza un candidato. `fuente`: Drive/Gmail/Manual. `via`: texto libre (LinkedIn, HireWatchers…) |
| `$screen --all` | Procesa todas las candidaturas pendientes de las fuentes configuradas |
| `$prep [nombre]` | Genera guión personalizado de entrevista (detecta ronda automáticamente) |
| `$debrief [nombre]` | Analiza transcripción post-entrevista y actualiza scoring (detecta ronda automáticamente) |
| `$case [nombre]` | Genera prueba / business case personalizado para el candidato |
| `$case-eval [nombre]` | Analiza la entrega del candidato y asigna score con rúbrica |
| `$score-final [nombre]` | Calcula el score final ponderado por etapa del proceso |
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

### Bloque 4: Fuente de transcripciones post-entrevista

9. "¿Cómo grabáis las entrevistas? Opciones:
   - **TAM-OS** (Zoom → ElevenLabs → TAM-OS, recomendado si tenéis TAM-OS conectado)
   - **Granola** (si usáis Granola como notetaker)
   - **Manual** (el usuario pegará las notas/transcripción directamente)"
   → `transcript_source`: `tam-os` | `granola` | `manual`

### Bloque 5: Scoring personalizado

10. Mostrar los pesos por defecto y preguntar si se ajustan:

```
Dimensión                Peso
─────────────────────────────
Experiencia relevante    20%
Skills funcionales       25%
Fit cultural             25%
Calidad de empresas      15%
Huella digital           15%
```

"¿Quieres ajustar algún peso para este proceso? (deben sumar 100%)"

### Bloque 6: Señales del proceso y Módulo A del case — derivados de la JD

Este bloque es **automático**: Claude analiza la JD y el contexto del proyecto
para derivar dos elementos que se guardan en la Config y se reutilizan para todos los candidatos.

**Paso 6a — Análisis de la JD**

A partir de la JD y de lo que el usuario ha explicado sobre el proyecto, inferir:

1. **Red flags bloqueantes** (3-5): características que hacen que un candidato NO sea viable
   para ESTE rol específico, independientemente de sus otras cualidades.

2. **Red flags de atención** (3-5): características que no bloquean pero bajan el score
   y deben confirmarse en entrevista.

3. **Señales positivas diferenciadores** (4-6): características que hacen que un candidato
   DESTAQUE por encima de la media para ESTE rol.

4. **Módulo A del case** (estándar): un ejercicio idéntico que TODOS los candidatos del
   proceso responderán, permitiendo comparación directa entre ellos.
   - Se deriva del contexto del proyecto: etapa de la empresa, recursos disponibles,
     reto principal del rol, horizonte temporal realista
   - Debe ser relevante para cualquier candidato del proceso, independientemente de su perfil
   - Formato: 1 pregunta estratégica + 1 ejercicio práctico acotado
   - Tiempo estimado: 2 horas (el Módulo B añadirá 1 hora más)
   - **No se personaliza por candidato** — su valor está en la comparabilidad

**Paso 6b — Presentar al usuario para validación**

Mostrar señales y Módulo A en este formato y pedir confirmación/ajuste:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚦 SEÑALES DEL PROCESO — [Rol] | [Empresa]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 RED FLAGS BLOQUEANTES
  · [flag 1] — Motivo: [...]
  ...

🟡 RED FLAGS DE ATENCIÓN
  · [flag 1] — Qué testear: [...]
  ...

✅ SEÑALES POSITIVAS DIFERENCIADORES
  · [señal 1] — Por qué importa: [...]
  ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 MÓDULO A DEL CASE (igual para todos los candidatos)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Enunciado completo del Módulo A tal como se enviará a los candidatos]

Tiempo estimado: ~2 horas
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
¿Ajustas algo antes de guardar?
```

**Paso 6c — Guardar en Config de Notion**:
- `Señales del proceso` → red flags y señales positivas validadas
- `Case Módulo A` → enunciado del módulo estándar validado
- `Case Módulo A Rúbrica` → rúbrica interna del Módulo A (no visible al candidato)

Estos campos se cargarán en contexto al inicio de cada sesión y se reutilizarán en cada `$case`.

### Bloque 7: Pesos del score final

Después de todos los parámetros anteriores, configurar cómo se pondera el score final:

```
Etapa                    Peso por defecto
────────────────────────────────────────
$screen (criba CV)             0%   ← solo sirve para decidir a quién llamar
$debrief ronda 1              30%
$case-eval                    40%
$debrief ronda 2              30%
```

"¿Quieres ajustar estos pesos? (las etapas activas deben sumar 100%)
Si el proceso no incluye case o no hay 2ª entrevista, los pesos
se redistribuirán automáticamente entre las etapas que existan."

→ `score_weights`: objeto con claves `screen`, `debrief1`, `case`, `debrief2`

**Redistribución automática** cuando faltan etapas:
- Sin case: debrief1 50% / debrief2 50%
- Sin 2ª entrevista: debrief1 60% / case 40%
- Solo 1 entrevista sin case: debrief1 100%

### Bloque 8: Generar configuración

Una vez recopilado y validado todo:

1. **Crear estructura Notion** → ver sección "Estructura Notion"
2. **Guardar config** → crear página Notion `⚙️ Config: [Rol] — [Empresa]`
   con todos los parámetros en formato tabla, incluyendo las señales del proceso
3. **Crear carpetas Google Drive** si no existen (`drive_cvs_folder`, `drive_reports_folder`)
4. **Confirmar al usuario**:
   - Proceso configurado ✓
   - Fuentes activas (Gmail / Drive / Manual)
   - Fuente de transcripciones: [transcript_source]
   - Link directo a la DB Notion de candidatos
   - Pesos de scoring por dimensión activos
   - Pesos del score final por etapa: screen [X]% / debrief1 [X]% / case [X]% / debrief2 [X]%
   - N red flags bloqueantes + N de atención + N señales positivas cargadas
5. "Listo. Empieza con `$screen [nombre del primer candidato] ([fuente])`."

---

## $screen — Analizar candidato

**Trigger**: `$screen [nombre] | [fuente] | [via]` o `$screen --all`

**Parámetros**:
- `fuente` (operacional, no se guarda en Notion): de dónde obtener el CV/perfil
  - `Drive` → buscar en `drive_source_folder`
  - `Gmail` → buscar en inbox con `gmail_query` + nombre
  - `Manual` → el candidato ha sido pegado o subido en la conversación (default si se omite)
- `via` (se guarda en Notion como "Fuente"): canal por el que llegó el candidato. Texto libre.
  - Ejemplos: `LinkedIn`, `HireWatchers`, `LinkedIn Jobs`, `referido Chechu`, `email directo`

**Ejemplos**:
```
$screen Santiago Molinari | Drive | LinkedIn
$screen Ana García | Manual | HireWatchers
$screen Pedro Ruiz | Gmail | referido Chechu
$screen Laura Niño | Drive | LinkedIn Jobs
```

### Detección automática de fase

Antes de procesar, Claude determina qué información hay disponible:
- **CV presente** (PDF/doc subido, Drive, Gmail con adjunto) → **FASE B: Screen completo**
- **Solo perfil público** (texto o URL de LinkedIn/HireWatchers, sin CV) → **FASE A: Pre-screen**

---

### FASE A — Pre-screen (sin CV)

Para candidatos de LinkedIn o HireWatchers donde no hay CV todavía.

**Paso A1: Recopilar perfil público**
- Texto/URL pegado en la conversación → analizar el perfil
- Web search siempre: `"[nombre]" site:linkedin.com`, `"[nombre]" [empresa] [rol keywords]`, `"[nombre]" (podcast OR articulo OR newsletter)`

**Paso A2: Scoring estimado** — mismo framework 5 dimensiones pero marcando score con `~` (estimado).
Ser conservador: con dudas, puntuar por debajo. Indicar qué dimensiones quedan sin confirmar.

**Paso A3: Output pre-screen**
```
⚡ PRE-SCREEN — [Nombre] (sin CV · score estimado)
Experiencia relevante  ███░░  ~3.0/5
Skills funcionales     ███░░  ~X.X/5  ← sin confirmar
...
SCORE ESTIMADO ········ ~X.X / 5
Señales clave: [2-3 bullets]
Sin confirmar sin CV: [gaps clave]
━━━━━━━━━━━━━━━━━━━━━
DECISIÓN: PEDIR CV ✓  /  ARCHIVAR ✗
Razón: [1 línea]
━━━━━━━━━━━━━━━━━━━━━
```

**Paso A4: Guardar en Notion**
- PEDIR CV → Status: `Pendiente CV`, score estimado, nota "Pre-screen — CV solicitado [fecha]"
- ARCHIVAR → Status: `Archivado`, score estimado, nota de motivo

---

### FASE B — Screen completo (con CV)

Para candidatos con CV disponible (Drive, Gmail, subido en conversación, o recibido tras Fase A).
Si el candidato ya tiene entrada en Notion de una Fase A previa, actualizar esa fila en lugar de crear una nueva.

**Paso B1: Recopilar información**

**a) PDF/doc subido en la conversación** → procesar directamente si existe
**b) Gmail** (si `gmail_query` está configurada):
   - Usar Gmail MCP: `search_threads` con `[gmail_query] [nombre]`
   - Extraer CV adjunto o contenido del email
**c) Google Drive** (si `drive_source_folder` está configurada):
   - Buscar archivos con nombre del candidato en la carpeta fuente
**d) Texto pegado en conversación** → perfil ya disponible de Fase A
**e) Web search** (siempre): mismo set de queries que Fase A

### Paso 2: Scoring

Evaluar cada dimensión de 1 a 5. Consultar `references/scoring-rubric.md`
para los criterios exactos de cada nota por dimensión.

Calcular score final = suma(nota_i × peso_i). Redondear a 1 decimal.

**Uso de las señales del proceso (cargadas desde Config de Notion)**

Al puntuar y redactar el output, aplicar siempre:

- **Red flags bloqueantes** → si el candidato presenta alguno, indicarlo explícitamente
  en el output y bajar el score de la dimensión correspondiente.
  Un red flag bloqueante puede justificar directamente "No Avanzar" sin importar el resto.

- **Red flags de atención** → documentar si aparece alguno; marcarlo para
  confirmar en la entrevista. No bajan automáticamente el score, pero se anotan en Notion.

- **Señales positivas diferenciadores** → si el candidato presenta alguna, destacarlo
  en el output y reflejarlo en el score de la dimensión correspondiente.
  Dos o más señales positivas pueden elevar la recomendación de "En Hold" a "Avanzar".

**Red flags universales** (aplican a cualquier proceso independientemente de la JD):
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
Experiencia relevante  ████░  3.5/5  (20%)  → X.XX
Skills funcionales     ████░  4.0/5  (25%)  → X.XX
Fit cultural           ███░░  3.0/5  (25%)  → X.XX
Calidad de empresas    ███░░  3.0/5  (15%)  → X.XX
Huella digital         ██░░░  2.0/5  (15%)  → X.XX

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

1. Copiar CV a `drive_cvs_folder` con nombre: `YYYYMMDD_apellido_nombre_rol.pdf`
   → Guardar el `viewUrl` del archivo copiado
2. Crear la fila del candidato en la Notion DB con todos los campos, incluyendo:
   - Campo "Fuente" → valor del parámetro `via` (texto libre; "Otro" si se omite)
   - Campo "CV en Drive" → `viewUrl` del archivo copiado en el paso anterior
3. Confirmar con link directo a la fila en Notion

---

## $prep — Preparar entrevista

**Trigger**: `$prep [nombre]`

**Detección automática de ronda**: Claude consulta Notion para saber en qué punto del proceso está el candidato:
- Si NO hay `Score Post-Entrevista` → **Ronda 1** (primera entrevista)
- Si hay `Score Post-Entrevista` pero NO hay `Score Entrevista 2` → **Ronda 2** (segunda entrevista)

El guión de ronda 2 se enfoca en lo que NO quedó claro en la ronda 1 y en validar la entrega del case.

### Paso 1: Leer datos del candidato

- Recuperar ficha de Notion: score card, puntos fuertes, red flags, notas
- Leer el CV en Drive si hace falta profundizar en detalles

### Paso 2: Generar guión personalizado

El guión incluye las siguientes secciones. Consultar `references/interview-guide-template.md`
para estructura detallada y tipos de pregunta.

**Antes de construir el guión**, cargar desde la Config de Notion:
- Las señales del proceso (red flags bloqueantes, de atención y positivas)
- Usarlas para diseñar las preguntas del bloque 3 (Skills profundos) y 4 (Fit cultural/situacional)

**Sección 0 — Contexto antes de llamar** (no es parte de la entrevista)
- "Lo que ya sabes — no preguntar": hechos del CV que no hay que confirmar
- "Lo que NO sabes y debes descubrir": los 4-5 puntos clave que la call debe responder
  (incluir los red flags de atención que apliquen a este candidato)

| Sección | Duración | Objetivo |
|---|---|---|
| 1. Apertura | 5 min | Contexto del proyecto en 90 segundos → reacción del candidato |
| 2. Trayectoria y categoría | 20 min | Sustancia detrás de los logros del CV, pensamiento en términos de behavior change |
| 3. Skills técnicos profundos | 20 min | Unit economics, AEO/GEO, AI stack, o la dimensión técnica diferencial del rol |
| 4. Fit cultural y situacional | 15 min | FTE motivation (misional vs pragmática), relocation si aplica, conflictos potenciales |
| 5. Cierre | 10 min | Sus preguntas + red flags a vigilar + señales positivas a confirmar |

Cada sección incluye:
- Pregunta(s) exacta(s) con el wording recomendado
- Qué se busca en la respuesta (señal 5/5)
- Red flags si la respuesta va por mal camino

Al final del guión:
- **Red flags a vigilar** durante la call (bullets concisos)
- **Señales positivas que confirmarían avanzar** (bullets concisos)

### Paso 3: Guardar en Notion

1. Guardar el guión completo como contenido dentro de la página del candidato en Notion
   usando `notion-update-page` con comando `insert_content` (no como sub-página separada)
2. Mostrar el guión completo en la conversación
3. "¿Quieres ajustar alguna sección antes de la entrevista?"

---

## $debrief — Análisis post-entrevista

**Trigger**: `$debrief [nombre]`

**Detección automática de ronda**: Claude consulta Notion para determinar qué debrief hacer:
- Si NO hay `Score Post-Entrevista` → **Debrief ronda 1** → actualiza `Score Post-Entrevista` y `Fecha Entrevista`
- Si hay `Score Post-Entrevista` → **Debrief ronda 2** → actualiza `Score Entrevista 2` y `Fecha Entrevista 2`

### Paso 1: Obtener la transcripción

En orden de preferencia según `transcript_source` configurado:

**TAM-OS** (si `transcript_source = tam-os`):
```
tam_os_search_meetings(
  mode="specific_meeting",
  participantName="[nombre candidato]",
  query="entrevista [nombre] [rol]",
  since="[fecha aproximada de la call]"
)
```
→ Obtener `meeting.id` del resultado
→ `mojo_get_meeting_transcript(by="id", meeting_id="[id]", segment_offset=0)`
→ Leer todos los segmentos del transcript

**Granola** (si `transcript_source = granola`):
→ `query_granola_meetings("[nombre] entrevista"`

**Manual** (si `transcript_source = manual` o ninguna fuente devuelve resultados):
→ Pedir al usuario que pegue la transcripción o sus notas

### Paso 2: Cruzar con el guión de $prep y las señales del proceso

Recuperar de Notion:
- El guión guardado en la página del candidato
- Las señales del proceso (red flags y positivas) de la Config

Para cada sección del guión, evaluar:
- ¿Se preguntó? ¿Se respondió? ¿Con qué profundidad?
- ¿Se resolvieron los red flags de atención identificados en el screening?
- ¿Se confirmaron o desmintieron las señales positivas esperadas?
- ¿Apareció algún red flag bloqueante que no estaba en el screening?

### Paso 3: Re-scoring post-entrevista

Re-evaluar las 5 dimensiones con evidencia concreta del transcript.
Anotar qué cambió vs el screening y por qué.

### Paso 4: Output en conversación

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 DEBRIEF — [Nombre] | [Fecha]
Screening: [X.X] · Post-entrevista: [Y.Y]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

VEREDICTO: [párrafo de 2-3 líneas con el diagnóstico clave]

ANÁLISIS POR BLOQUE:
[Para cada sección del guión de prep: ✅/⚠️/🔴 + qué dijo + qué significa]

SCORING REVISADO:
[Tabla con score screening vs post-entrevista y delta]

PUNTO(S) CRÍTICO(S): [Si hay algo que bloquea o cambia la decisión]

RECOMENDACIÓN: AVANZAR / NO AVANZAR / PENDIENTE
SIGUIENTE PASO: [acción concreta]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Paso 5: Actualizar Notion

**Propiedades** (via `notion-update-page update_properties`):
- `Score Post-Entrevista` → nuevo score
- `Status` → `Entrevistado`
- `Fecha Entrevista` → fecha de la call
- `Notas` → síntesis ejecutiva del debrief (máx. 3-4 líneas)
- `Recomendacion` → actualizar si cambió

**Contenido de la página** (via `notion-update-page insert_content`):
Añadir al cuerpo de la página del candidato una sección completa `# 📊 Debrief` con:
- Veredicto
- Análisis por bloque (texto narrativo, no bullets cortos)
- Tabla de scoring screening vs post-entrevista
- Punto(s) crítico(s)
- Próximo paso

---

## $case — Generar prueba / business case

**Trigger**: `$case [nombre]`
**Cuándo usar**: después del primer `$debrief`, para candidatos que avanzan a la siguiente fase.

La prueba tiene **dos módulos**:

| Módulo | Tipo | Generación | Propósito |
|---|---|---|---|
| **A** | Estándar | Una vez en onboarding, igual para todos | Comparabilidad entre candidatos |
| **B** | Personalizado | Por candidato según screen + debrief | Profundizar en gaps y fortalezas específicas |

Tiempo estimado total para el candidato: **3-4 horas** (A ~2h + B ~1-2h).

### Paso 1: Leer contexto

Cargar desde Notion:
- **Config del proceso**: `Case Módulo A` (enunciado) + `Case Módulo A Rúbrica` (interna)
- **Ficha del candidato**: score card, debrief ronda 1, red flags pendientes, señales a confirmar

### Paso 2: Generar Módulo B (personalizado)

A partir del debrief del candidato, diseñar un ejercicio que:
1. Confirme la fortaleza diferencial del candidato (lo que más prometía en la entrevista)
2. Teste el gap principal detectado en el debrief (lo que quedó sin confirmar)
3. Se base en el contexto real del proyecto — no genérico

Formato: 1 ejercicio práctico concreto (produce algo: plan, análisis, propuesta, copy, modelo…)

**Generar también la rúbrica interna del Módulo B** (no se muestra al candidato):
- Qué entrega sería un 5/5 / 3/5 / 1/5

### Paso 3: Ensamblar el case completo

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 BUSINESS CASE — [Nombre]
Tiempo estimado: 3-4 horas · Entrega: [fecha sugerida]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PARTE 1 — Reto estratégico (todos los candidatos)
[Enunciado del Módulo A tal como está en Config]

PARTE 2 — Ejercicio específico
[Enunciado del Módulo B generado para este candidato]

Entrega: PDF o documento enviado a [email/Drive]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Mostrar el case completo y preguntar: "¿Quieres ajustar algo antes de enviarlo?"

### Paso 4: Guardar en Notion

- Sección `# 📝 Business Case — Módulo A` → enunciado del módulo estándar
- Sección `# 📝 Business Case — Módulo B` → enunciado del módulo personalizado
- Sección `# 📊 Rúbrica Módulo B (interna)` → rúbrica del módulo personalizado
  (la rúbrica del Módulo A ya está en la Config, no se duplica)
- Actualizar Status → `Case Enviado`

---

## $case-eval — Evaluar entrega del business case

**Trigger**: `$case-eval [nombre]`

**Input** (en orden de preferencia):
1. PDF/doc subido en la conversación
2. Archivo en Google Drive (pegar URL o buscar en `drive_cvs_folder`)
3. Texto pegado directamente

### Paso 1: Leer rúbricas y enunciados

Recuperar de Notion:
- **Config del proceso**: `Case Módulo A Rúbrica` (rúbrica del módulo estándar)
- **Página del candidato**: `# 📝 Business Case — Módulo B` + `# 📊 Rúbrica Módulo B`

### Paso 2: Evaluar por módulo

**Módulo A** (estándar):
- Evaluar la respuesta contra la rúbrica del Módulo A (de la Config)
- Score A: 1-5
- Nota: el Módulo A de TODOS los candidatos se evalúa con la misma rúbrica → comparabilidad garantizada

**Módulo B** (personalizado):
- Evaluar contra la rúbrica específica del candidato
- Score B: 1-5

**Score Case final** = media ponderada configurable (por defecto 50% A + 50% B)

### Paso 3: Output en conversación

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 CASE EVAL — [Nombre] | [Fecha entrega]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

MÓDULO A (estándar — comparable con otros candidatos)
Score: [X.X]/5
→ [qué hizo bien, qué faltó, qué sorprendió]

MÓDULO B (personalizado)
Score: [X.X]/5
→ [ídem]

SCORE CASE FINAL ············· [X.X]/5

SEÑALES POSITIVAS CONFIRMADAS: [lista]
RED FLAGS CONFIRMADOS O NUEVOS: [lista]

RECOMENDACIÓN: AVANZAR A 2ª ENTREVISTA / NO AVANZAR
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Paso 4: Guardar en Notion

- Insertar análisis completo como sección `# 📊 Evaluación del Case` en la página del candidato
- Actualizar propiedades: `Score Case`, `Fecha Case`, Status → `Case Evaluado`

---

## $score-final — Score compuesto final

**Trigger**: `$score-final [nombre]`
**Cuándo usar**: después del segundo `$debrief` (o cuando el proceso de ese candidato está cerrado).

### Paso 1: Leer todos los scores del candidato

Desde Notion:
- `Score Post-Entrevista` (debrief 1)
- `Score Case`
- `Score Entrevista 2` (debrief 2)
- Pesos configurados en la Config del proceso

### Paso 2: Calcular score final ponderado

```
Score Final = Σ (score_etapa × peso_etapa) / Σ pesos_etapas_con_dato
```

Si falta alguna etapa (no se hizo el case, o no hubo 2ª entrevista), redistribuir
los pesos proporcionalmente entre las etapas disponibles.

### Paso 3: Output en conversación

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏁 SCORE FINAL — [Nombre]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Etapa             Score   Peso   Ponderado
──────────────────────────────────────────
Entrevista 1      X.X/5    30%     X.XX
Business Case     X.X/5    40%     X.XX
Entrevista 2      X.X/5    30%     X.XX
──────────────────────────────────────────
SCORE FINAL  ···············  X.X / 5

RECOMENDACIÓN FINAL: CONTRATAR / SEGUNDA OPCIÓN / NO CONTRATAR
[2-3 líneas de justificación con los puntos determinantes]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Paso 4: Actualizar Notion

- `Score Final Compuesto` → valor calculado
- `Recomendacion` → actualizar si ha cambiado
- Status → `Decisión Pendiente` o `Oferta` según la recomendación

---

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
- Distinguir entre "conoce el concepto" y "lo ha operado en producción" — especialmente en skills técnicos (unit economics, AEO/GEO, AI automation)

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
│   └── [Candidato X]          ← página del candidato
│       │                         (contenido en el cuerpo de la página)
│       ├── [Guión de entrevista]  ← sección insertada por $prep
│       └── [Debrief]             ← sección insertada por $debrief
└── 📊 Informe Final           ← link al PDF en Google Drive
```

**Nota importante sobre el contenido en Notion:**
El guión de entrevista (`$prep`) y el análisis post-entrevista (`$debrief`) se guardan
como secciones de contenido markdown **dentro del cuerpo de la página del candidato**,
no como sub-páginas separadas. Usar `notion-update-page` con comando `insert_content`.
Esto permite ver toda la información del candidato en una sola página al hacer click en su fila.

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
