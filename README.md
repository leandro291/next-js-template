# next-js-template

Template base para proyectos Next.js con estructura por módulos de dominio, shadcn/ui y un flujo de trabajo **SDD (Spec Driven Development)** con agentes de Claude Code.

## Stack

- **Next.js 16** (App Router, Turbopack) + **React 19** + **TypeScript** strict
- **Tailwind CSS 4** (configurado en `app/globals.css`, sin `tailwind.config`)
- **shadcn/ui** preset `base-nova` (basado en `@base-ui/react`, no Radix) + `lucide-react`
- Instalados, listos para usar: `@tanstack/react-query`, `@tanstack/react-table`, `zustand`, `axios`, `zod`

## Empezar

```bash
npm install
npm run dev        # http://localhost:3000
```

| Comando | Qué hace |
| ------- | -------- |
| `npm run dev` / `build` / `start` | Desarrollo, build y producción |
| `npm run lint` | ESLint 9 (`core-web-vitals` + `typescript`) |
| `npx tsc --noEmit` | Typecheck |
| `npx shadcn@latest add <component>` | Agregar componentes de UI |

## Estructura

```
app/          # solo routing (páginas delgadas que componen desde modules/)
modules/      # un módulo por dominio: components, hooks, services, schemas, store, types
components/
  ui/         # shadcn/ui
  shared/     # reutilizables entre módulos
hooks/        # hooks compartidos
lib/          # utilidades transversales (cn, instancia de axios)
docs/         # SETUP.md y specs
```

Las reglas completas (naming, estructura, SOLID/DRY/KISS/YAGNI, shadcn primero) están en **[`docs/SETUP.md`](docs/SETUP.md)**, que es la fuente de verdad del proyecto.

## Flujo de trabajo: SDD con agentes

```
Requerimiento → Spec → Desarrollo (+ tests) → Review → Cierre
```

| Agente | Rol |
| ------ | --- |
| `orchestrator` | Punto de entrada. Clasifica la tarea en **BUILD** (directo, 1–3 archivos) o **SDD**, planifica fases y coordina a los demás. |
| `spec` | Escribe la spec en `docs/specs/<module>/<feature>.md` (criterios, contratos, plan por fases). |
| `developer` | Implementa una tarea aprobada, tocando solo sus archivos (`Owns`). |
| `reviewer` | Valida contra la spec y `SETUP.md`. Solo lectura. |

```bash
claude --agent orchestrator
```

> Ninguna implementación arranca sin una spec **aprobada por un humano** (`Status: approved` + `Approved by`).

Los agentes viven en `.claude/agents/`.

## Skills recomendadas

Para trabajar de forma óptima con Claude Code en este template:

| Skill | Para qué | Instalación |
| ----- | -------- | ----------- |
| [**ui-ux-pro-max**](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | Criterio de UI/UX: estilos, paletas, tipografía, accesibilidad y guías por stack. | `npx skills add nextlevelbuilder/ui-ux-pro-max-skill` |
| [**frontend-design**](https://github.com/anthropics/skills) | Diseño visual con dirección propia, evitando interfaces genéricas. | `npx skills add anthropics/skills --skill frontend-design` |
| [**vercel-labs**](https://github.com/vercel-labs/agent-skills) | Buenas prácticas de React/Next.js (`vercel-react-best-practices`) y review de UI (`web-design-guidelines`). | `npx skills add vercel-labs/agent-skills` |
| [**ponytail**](https://github.com/DietrichGebert/ponytail) | Fuerza la solución más simple que funcione (YAGNI, stdlib y dependencias existentes primero). Encaja con las reglas de `SETUP.md`. | `/plugin marketplace add DietrichGebert/ponytail`<br>`/plugin install ponytail@ponytail` |
| [**caveman**](https://github.com/JuliusBrussee/caveman) | Respuestas ultra comprimidas: menos tokens y sesiones más largas. | `npx skills add JuliusBrussee/caveman` |
| [**superpowers**](https://github.com/obra/superpowers) | Procesos de ingeniería: brainstorming, planes, TDD, debugging sistemático y verificación antes de cerrar. | `/plugin install superpowers@claude-plugins-official` |

Los comandos `/plugin ...` se ejecutan dentro de Claude Code.

## Deploy

Pensado para [Vercel](https://vercel.com/new): importá el repo y listo.
