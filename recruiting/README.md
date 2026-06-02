# Recruiting Skill — Claude Skills

Skill de reclutamiento end-to-end para Claude. Gestiona el ciclo completo de selección
de talento: sourcing, screening con scoring estructurado, preparación de entrevistas,
análisis post-entrevista, evaluación de business cases, score final ponderado por etapa,
e informe final en PDF.

Desarrollada por **AI FinLabs & The Agile Monkeys**.

---

## ¿Qué hace?

| Comando | Qué hace |
|---|---|
| `$onboarding` | Configura el proceso: empresa, rol, JD, fuentes, rutas, pesos, señales dinámicas del proceso |
| `$screen [nombre]` | Analiza un candidato desde Gmail, Drive, PDF, LinkedIn paste o web |
| `$screen --all` | Procesa todas las candidaturas pendientes de las fuentes configuradas |
| `$prep [nombre]` | Genera guión personalizado de entrevista (detecta ronda 1 o 2 automáticamente) |
| `$debrief [nombre]` | Analiza transcripción post-entrevista (detecta ronda automáticamente) |
| `$case [nombre]` | Genera prueba / business case personalizado con rúbrica interna |
| `$case-eval [nombre]` | Analiza la entrega del candidato y asigna score contra la rúbrica |
| `$score-final [nombre]` | Calcula el score compuesto ponderado por etapa del proceso |
| `$report` | Genera ranking de candidatos e informe final en PDF exportado a Google Drive |

---

## Flujo completo

```
$screen     → criba (el score no cuenta en el resultado final por defecto)
$prep       → guión ronda 1
$debrief    → análisis ronda 1          peso: 30%
$case       → genera prueba personalizada con rúbrica interna
$case-eval  → evalúa entrega            peso: 40%
$prep       → guión ronda 2 (auto)
$debrief    → análisis ronda 2          peso: 30%
$score-final → score compuesto final
$report     → PDF con ranking
```

Los pesos son configurables en `$onboarding`. Si falta alguna etapa, se redistribuyen automáticamente.

---

## Integraciones

| Herramienta | Uso |
|---|---|
| **Notion MCP** | Base de datos de candidatos, guiones, debriefs, config del proceso |
| **Gmail MCP** | Lectura de candidaturas recibidas por email |
| **Google Drive MCP** | Lectura de CVs, almacenamiento de CVs procesados e informes PDF |
| **TAM-OS MCP** | Transcripciones de entrevistas vía Zoom + ElevenLabs (fuente primaria) |
| **Granola MCP** | Transcripciones de entrevistas (fuente alternativa) |
| **Web search** | Búsqueda de huella digital de candidatos |

La skill funciona aunque no todas las integraciones estén disponibles —
se adapta a las fuentes activas configuradas durante el `$onboarding`.

---

## Instalación

### Opción A — Un solo prompt (recomendada)

Abre una conversación de Claude con acceso a herramientas y pega:

```
Instala la Recruiting Skill desde GitHub.
Descarga el archivo en https://raw.githubusercontent.com/sergiocarbajal-cpu/claude-skills/main/recruiting/SKILL.md
y guárdalo en /mnt/skills/user/recruiting/SKILL.md
También descarga las referencias:
- https://raw.githubusercontent.com/sergiocarbajal-cpu/claude-skills/main/recruiting/references/scoring-rubric.md → /mnt/skills/user/recruiting/references/scoring-rubric.md
- https://raw.githubusercontent.com/sergiocarbajal-cpu/claude-skills/main/recruiting/references/interview-guide-template.md → /mnt/skills/user/recruiting/references/interview-guide-template.md
- https://raw.githubusercontent.com/sergiocarbajal-cpu/claude-skills/main/recruiting/references/notion-schema.md → /mnt/skills/user/recruiting/references/notion-schema.md
Confirma cuando esté listo y recuérdame que ejecute $onboarding.
```

### Opción B — Manual

1. Descarga este repositorio o los archivos individuales
2. Copia la carpeta `recruiting/` a `/mnt/skills/user/recruiting/` en tu proyecto de Claude
3. Reinicia la conversación para que Claude detecte la skill

---

## Uso rápido

Una vez instalada, escribe `$onboarding` para configurar tu primer proceso.
Claude te guiará por el flujo en bloques de preguntas, incluyendo la derivación
automática de red flags y señales positivas a partir de la JD.

---

## Estructura del repositorio

```
recruiting/
├── SKILL.md                            ← Skill principal (Claude la lee al activarse)
├── README.md                           ← Este archivo
└── references/
    ├── scoring-rubric.md               ← Criterios de puntuación 1-5 por dimensión
    ├── interview-guide-template.md     ← Estructura detallada del guión de entrevista
    └── notion-schema.md                ← Schema de la DB Notion (campos, tipos, Status)
```

---

## Versiones y changelog

| Versión | Fecha | Cambios |
|---|---|---|
| 1.2.0 | 2026-06 | `$case`, `$case-eval`, `$score-final`. Pesos configurables por etapa (default: 0/30/40/30). `$prep` y `$debrief` detectan ronda automáticamente. |
| 1.1.0 | 2026-06 | TAM-OS como fuente de transcripciones. Contenido en página del candidato. Señales dinámicas del proceso desde la JD. |
| 1.0.0 | 2026-06 | Primera versión. 5 comandos, Notion + Drive + Gmail + Granola. |

---

## Licencia

Uso interno — AI FinLabs & The Agile Monkeys.
Adaptación libre para otros proyectos dentro de la organización.

---

## Mantenimiento

Skill mantenida por el equipo de AI FinLabs.
Issues y sugerencias → abrir un issue en este repositorio o escribir a Sergio (sergiocarbajal@theagilemonkeys.com).
