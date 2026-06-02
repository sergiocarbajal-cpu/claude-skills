# Schema de Notion — Recruiting Skill v1.0

Estructura de base de datos y páginas que crea `$onboarding` en Notion.
Usar este schema como referencia al crear o actualizar registros vía MCP.

---

## Jerarquía de páginas

```
📁 Recruiting: [Rol] — [Empresa]          ← Página padre del proceso
├── ⚙️ Config: [Rol]                       ← Config del proceso
├── 🗃️ Candidatos [DB]                     ← Base de datos principal
│   ├── [Candidato 1]                      ← Página del candidato
│   │   │   (propiedades en header)
│   │   ├── # 🎙️ Guión de entrevista       ← Sección en el cuerpo ($prep)
│   │   └── # 📊 Debrief                   ← Sección en el cuerpo ($debrief)
│   └── [Candidato N] ...
└── 📊 Informe Final                       ← Link al PDF en Google Drive
```

**Importante**: El guión de entrevista y el debrief se insertan como secciones markdown
en el **cuerpo de la página del candidato** (no como sub-páginas), usando
`notion-update-page` con comando `insert_content`. Esto permite ver todo el historial
del candidato en una sola vista al hacer click en su fila de la DB.

---

## Página: ⚙️ Config

Contiene los parámetros del proceso en formato tabla. No editar manualmente.

| Campo | Valor | Descripción |
|---|---|---|
| Empresa | [company] | Empresa que contrata |
| Proyecto | [project] | Proyecto o producto específico |
| Rol | [role] | Título exacto del rol |
| Fecha apertura | [YYYY-MM-DD] | Cuando se abrió el proceso |
| Gmail query | [query] | Query usada en Gmail MCP |
| Drive fuente | [url] | Carpeta de CVs en bruto |
| Drive CVs | [url] | Carpeta de CVs procesados |
| Drive informes | [url] | Carpeta de PDFs finales |
| Peso experiencia | [%] | |
| Peso skills | [%] | |
| Peso cultura | [%] | |
| Peso empresas | [%] | |
| Peso digital | [%] | |
| JD | [text/url] | Job description completa |
| Señales del proceso | [text] | JSON o markdown con red flags bloqueantes, red flags de atención y señales positivas derivadas de la JD. Se genera en $onboarding y se valida con el usuario. Se carga en contexto al inicio de cada sesión. |

---

## Base de datos: 🗃️ Candidatos

### Campos y tipos de propiedad Notion

| Nombre del campo | Tipo Notion | Valores / Notas |
|---|---|---|
| **Nombre** | Title | Nombre completo del candidato |
| Rol | Text | Rol al que aplica |
| Fuente | Select | LinkedIn · Gmail · HireWatchers · Drive · Referido · Otro |
| Status | Select | Nuevo · Screening · Pendiente CV · Entrevista Pendiente · Entrevistado · Case Enviado · Case Evaluado · Entrevista 2 Pendiente · Decisión Pendiente · Oferta · Rechazado · Archivado |
| Score Screening | Number | Decimal 1.0–5.0. Solo para criba — no entra en el score final por defecto |
| Score Post-Entrevista | Number | Decimal 1.0–5.0. Debrief ronda 1 |
| Score Case | Number | Decimal 1.0–5.0. Evaluación del business case |
| Score Entrevista 2 | Number | Decimal 1.0–5.0. Debrief ronda 2 |
| Score Final Compuesto | Number | Calculado por $score-final con pesos configurados. No es fórmula automática — lo escribe Claude |
| Fecha Entrevista | Date | Ronda 1 |
| Fecha Case | Date | Fecha de entrega del business case |
| Fecha Entrevista 2 | Date | Ronda 2 |
| Experiencia (dim) | Number | 1–5 |
| Skills (dim) | Number | 1–5 |
| Cultura (dim) | Number | 1–5 |
| Empresas (dim) | Number | 1–5 |
| Digital (dim) | Number | 1–5 |
| Red Flags | Text | Lista separada por · |
| Recomendación | Select | Avanzar · No Avanzar · Pendiente · En Hold |
| Fecha Entrevista | Date | |
| CV en Drive | URL | Link al PDF del CV en drive_cvs_folder |
| LinkedIn | URL | |
| Email | Email | |
| Notas breves | Text | Máx. 3 líneas de contexto rápido |
| Última Actualización | Last edited time | Automático |

### Views recomendadas

**Vista: Kanban por Status**
- Columnas: Nuevo / Screening / Entrevista Pendiente / Entrevistado / Decisión
- Agrupar por: Status
- Ordenar: Score Final desc dentro de cada columna

**Vista: Tabla de scoring** (para comparar candidatos)
- Columnas visibles: Nombre, Fuente, Score Screening, Score Post, Score Final, Recomendación
- Ordenar: Score Final desc
- Filtrar: ocultar Rechazados y Archivados

**Vista: Pipeline activo**
- Filtro: Status ≠ Rechazado AND Status ≠ Archivado
- Ordenar: Score Final desc

---

## Páginas de candidato

Cada candidato en la DB tiene una página hija con 3 sub-páginas.

### 📋 Score Card (creada por `$screen`)

Contenido en markdown:

```markdown
## Score Card — [Nombre] | [Fecha]

| Dimensión | Nota | Peso | Ponderado |
|---|---|---|---|
| Experiencia relevante | X.X | 25% | X.XX |
| Skills funcionales | X.X | 25% | X.XX |
| Fit cultural | X.X | 20% | X.XX |
| Calidad de empresas | X.X | 15% | X.XX |
| Huella digital | X.X | 15% | X.XX |
| **TOTAL** | | | **X.X** |

### Puntos fuertes
- ...

### Red flags detectados
- ...

### Fuentes consultadas
- CV: [link]
- LinkedIn: [url]
- Web search: [resumen de hallazgos]

### Recomendación del reclutador
[Texto]
```

### 📝 Guión de entrevista (creada por `$prep`)

Seguir la estructura de 6 secciones de `references/interview-guide-template.md`.
Incluir al final la rúbrica de evaluación vacía para rellenar post-entrevista.

### 🎙️ Debrief (creada por `$debrief`)

```markdown
## Debrief — [Nombre] | [Fecha entrevista]

**Score pre-entrevista**: X.X / 5
**Score post-entrevista**: X.X / 5
**Entrevistador/es**: [nombres]

### Momentos destacados
- ...

### Red flags resueltos
- ...

### Red flags confirmados o nuevos
- ...

### Análisis de sus preguntas
[Texto: qué preguntó, qué revela sobre su interés]

### Recomendación final
**AVANZAR / NO AVANZAR / PENDIENTE**

Justificación: [texto]
```

---

## Notas de implementación para Notion MCP

Al crear la DB vía Notion MCP (`notion-create-database`):

1. Crear primero la página padre con `notion-create-pages`
2. Crear la DB como hija de la página padre con `notion-create-database`
3. Los campos de tipo `Formula` deben definirse después de crear los campos
   de los que dependen (`Score Screening`, `Score Post-Entrevista`)
4. Las views se crean con `notion-create-view` después de la DB

Si el MCP no soporta todas las propiedades, crear la DB con los campos básicos
y documentar los campos pendientes para configuración manual.

**Campos mínimos para que la skill funcione**:
Nombre, Fuente, Status, Score Screening, Score Post-Entrevista, Recomendación, CV en Drive.
