# Recruiting Skill — Claude Skills

Skill de reclutamiento end-to-end para Claude. Gestiona el ciclo completo de selección
de talento: sourcing, screening con scoring estructurado, preparación de entrevistas,
análisis post-entrevista, e informe final en PDF.

Desarrollada por **AI FinLabs & The Agile Monkeys**.

---

## ¿Qué hace?

| Comando | Qué hace |
|---|---|
| `$onboarding` | Configura el proceso: empresa, rol, JD, fuentes, rutas, pesos de scoring |
| `$screen [nombre]` | Analiza un candidato desde Gmail, Drive, PDF, LinkedIn paste o web |
| `$screen --all` | Procesa todas las candidaturas pendientes de las fuentes configuradas |
| `$prep [nombre]` | Genera guión personalizado de entrevista adaptado al perfil del candidato |
| `$debrief [nombre]` | Analiza la transcripción post-entrevista (Granola, Zoom o texto) |
| `$report` | Genera ranking de candidatos e informe final en PDF exportado a Google Drive |

---

## Integraciones

| Herramienta | Uso |
|---|---|
| **Notion MCP** | Base de datos de candidatos, guiones, debriefs, config del proceso |
| **Gmail MCP** | Lectura de candidaturas recibidas por email |
| **Google Drive MCP** | Lectura de CVs, almacenamiento de CVs procesados e informes PDF |
| **Granola MCP** | Lectura de transcripciones de entrevistas |
| **Web search** | Búsqueda de huella digital de candidatos |

La skill funciona aunque no todas las integraciones estén disponibles —
se adapta a las fuentes activas configuradas durante el `$onboarding`.

---

## Instalación

### Opción A — Un solo prompt (recomendada)

Abre una conversación de Claude con acceso a herramientas (bash, file tools) y pega:

```
Instala la Recruiting Skill desde GitHub.
Descarga el archivo en https://raw.githubusercontent.com/theam/claude-skills/main/recruiting/SKILL.md
y guárdalo en /mnt/skills/user/recruiting/SKILL.md
También descarga las referencias:
- https://raw.githubusercontent.com/theam/claude-skills/main/recruiting/references/scoring-rubric.md → /mnt/skills/user/recruiting/references/scoring-rubric.md
- https://raw.githubusercontent.com/theam/claude-skills/main/recruiting/references/interview-guide-template.md → /mnt/skills/user/recruiting/references/interview-guide-template.md
- https://raw.githubusercontent.com/theam/claude-skills/main/recruiting/references/notion-schema.md → /mnt/skills/user/recruiting/references/notion-schema.md
Confirma cuando esté listo y recuérdame que ejecute $onboarding.
```

### Opción B — Manual

1. Descarga este repositorio o los archivos individuales
2. Copia la carpeta `recruiting/` a `/mnt/skills/user/recruiting/` en tu proyecto de Claude
3. Reinicia la conversación para que Claude detecte la skill

---

## Uso rápido

Una vez instalada, escribe `$onboarding` para configurar tu primer proceso.
Claude te guiará por el flujo en bloques de preguntas.

```
Tú:    $onboarding

Claude: ¿Para qué empresa y proyecto es este proceso?
        ...

        [configura fuentes, rutas, scoring]

        Listo. Empieza con $screen [nombre del primer candidato].

Tú:    $screen Ana García

Claude: [analiza CV + web search + email]

        📋 SCORING — Ana García | Head of Growth
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        Experiencia relevante  ████░  3.5/5
        Skills funcionales     ████░  4.0/5
        ...
        SCORE TOTAL · 3.8 / 5
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
        Guardado en Notion ↗
```

---

## Estructura del repositorio

```
recruiting/
├── SKILL.md                            ← Skill principal (Claude la lee al activarse)
├── README.md                           ← Este archivo
└── references/
    ├── scoring-rubric.md               ← Criterios de puntuación 1-5 por dimensión
    ├── interview-guide-template.md     ← Estructura detallada del guión de entrevista
    └── notion-schema.md                ← Schema de la DB Notion (campos y tipos)
```

---

## Versiones y changelog

| Versión | Fecha | Cambios |
|---|---|---|
| 1.0.0 | 2026-06 | Primera versión. 5 comandos, Notion + Drive + Gmail + Granola. |

---

## Licencia

Uso interno — AI FinLabs & The Agile Monkeys.
Adaptación libre para otros proyectos dentro de la organización.

---

## Mantenimiento

Skill mantenida por el equipo de AI FinLabs.
Issues y sugerencias → abrir un issue en este repositorio o escribir a Sergio (sergiocarbajal@theagilemonkeys.com).
