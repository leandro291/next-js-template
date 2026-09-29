# SETUP

Reglas de estructura, buenas prácticas y metodología de trabajo del proyecto. Aplican a todo el código nuevo, sea humano o generado por un agente.

Stack: Next.js (App Router), React, TypeScript, Tailwind, shadcn/ui, TanStack Query/Table, Zustand, Zod, Axios.

---

## 1. Estructura de carpetas

### Reglas

1. **Todo se organiza por módulo de dominio**, no por tipo técnico. Un módulo agrupa todo lo que pertenece a un dominio (`users`, `orders`, `billing`).
2. **Nombres en inglés**, siempre (carpetas, archivos, variables, tipos).
3. **Naming de TypeScript:**

   | Elemento                         | Convención                       | Ejemplo                    |
   | -------------------------------- | -------------------------------- | -------------------------- |
   | Archivos y carpetas              | `kebab-case`                     | `user-card.tsx`            |
   | Componentes, tipos, interfaces   | `PascalCase`                     | `UserCard`, `UserDto`      |
   | Funciones, variables, hooks      | `camelCase` (hooks con `use`)    | `getUsers`, `useUserList`  |
   | Constantes globales              | `UPPER_SNAKE_CASE`               | `API_BASE_URL`             |
   | Schemas Zod                      | `camelCase` + sufijo `Schema`    | `createUserSchema`         |
   | Stores Zustand                   | `use` + nombre + `Store`         | `useUserFilterStore`       |
   | Tests                            | mismo nombre + `.test.ts(x)`     | `user-service.test.ts`     |

4. **App Router:** `app/` contiene **solo routing** (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `route.ts`). Las páginas son delgadas: importan y componen desde `modules/`. Nada de lógica de negocio ni componentes de dominio dentro de `app/`.
5. **Server Components por defecto.** `"use client"` solo en el archivo que lo necesite (interactividad, hooks de estado/efecto, TanStack Query, Zustand), lo más abajo posible en el árbol.
6. **Un módulo no importa los internos de otro.** Lo que se comparte entre módulos se expone desde su `index.ts` o se sube a `components/shared`, `hooks` o `lib`.
7. **Solo se crean las subcarpetas que el módulo necesita** (YAGNI). Un módulo puede tener únicamente `components/` y `types/`.

### Estructura base

```
app/                          # routing only (App Router)
  (dashboard)/                # route group, no afecta la URL
    users/
      page.tsx
      [id]/page.tsx
  layout.tsx
  globals.css

modules/                      # dominio: una carpeta por módulo
  users/
    components/               # UI del dominio
      user-card.tsx
      user-table.tsx
    hooks/                    # hooks del dominio (queries, mutations)
      use-user-list.ts
    services/                 # acceso a datos (Axios), sin React
      user-service.ts
    schemas/                  # validación con Zod + tipos inferidos
      user-schema.ts
    store/                    # estado cliente (Zustand), solo si hace falta
      use-user-filter-store.ts
    types/
      user.ts
    index.ts                  # API pública del módulo

components/
  ui/                         # shadcn/ui (generado, no editar a lo loco)
  shared/                     # componentes reutilizables entre módulos
hooks/                        # hooks reutilizables entre módulos
lib/                          # utilidades transversales (cn, axios instance)
docs/                         # documentación y specs
```

### Ejemplo de uso

```tsx
// modules/users/services/user-service.ts
import { api } from "@/lib/api";
import type { User } from "../types/user";

export const getUsers = async (): Promise<User[]> => {
  const { data } = await api.get<User[]>("/users");
  return data;
};
```

```ts
// modules/users/hooks/use-user-list.ts
import { useQuery } from "@tanstack/react-query";
import { getUsers } from "../services/user-service";

export const useUserList = () =>
  useQuery({ queryKey: ["users"], queryFn: getUsers });
```

```tsx
// modules/users/components/user-table.tsx
"use client";

import { useUserList } from "../hooks/use-user-list";

export function UserTable() {
  const { data } = useUserList();
  // ...
}
```

```ts
// modules/users/index.ts  (lo único que ven los demás módulos y app/)
export { UserTable } from "./components/user-table";
export type { User } from "./types/user";
```

```tsx
// app/(dashboard)/users/page.tsx  (delgada: solo compone)
import { UserTable } from "@/modules/users";

export default function UsersPage() {
  return <UserTable />;
}
```

---

## 2. Buenas prácticas

**SOLID, DRY, KISS y YAGNI se aplican siempre**: componentes shadcn, componentes propios, funciones, hooks, services, stores, schemas.

| Principio | Qué significa acá                                                                                                  |
| --------- | ------------------------------------------------------------------------------------------------------------------ |
| **S**RP   | Una unidad, una responsabilidad: el service accede a datos, el hook orquesta, el componente renderiza.             |
| **O**CP   | Extender por props/composición (`children`, variantes) en vez de modificar el componente con más `if`.             |
| **L**SP   | Los componentes derivados o wrappers deben poder usarse donde se usa el base sin romper su contrato.               |
| **I**SP   | Props mínimas y específicas; no pasar objetos gigantes si solo se usan dos campos.                                 |
| **D**IP   | Los componentes dependen de hooks/abstracciones, no de Axios directo. Los hooks dependen de services.              |
| DRY       | Lógica repetida en 2+ lugares → se extrae. No antes (ver YAGNI).                                                   |
| KISS      | La solución más simple que funcione. Sin capas, patrones ni genéricos que el problema no pide.                     |
| YAGNI     | No construir para "algún día". Sin props, opciones, carpetas ni abstracciones sin un uso real actual.              |

### Componentes: shadcn primero

Al empezar cualquier componente de UI:

1. **Verificar si existe en shadcn** (`npx shadcn@latest add <name>`). Si existe, se usa.
2. Si no existe, **lo creamos nosotros pensando en reutilización**: props genéricas, estilos con Tailwind + `cn()`, sin lógica de dominio. Va en `components/shared/` si lo usan varios módulos, o en `modules/<domain>/components/` si es específico de uno.
3. Nota: el preset de shadcn es `base-nova` (basado en `@base-ui/react`, no Radix).

### Verificar antes de crear

**Siempre buscar antes de escribir.** Antes de crear un componente, función, hook, schema o util, confirmar que no exista ya (en el módulo, en `components/shared`, `hooks`, `lib` o en shadcn). Si existe algo parecido, se reutiliza o se extiende; no se duplica.

---

## 3. Metodología: SDD (Spec Driven Development)

En SDD ningún cambio se implementa sin una **spec aprobada por un humano** antes. La spec es la fuente de verdad: el código y los tests salen de ella.

> **Bloqueante:** el agente `developer` no arranca si la spec no tiene `Status: approved` y `Approved by: <nombre> (<fecha>)`. Esos campos los completa el orquestador solo después de que un humano apruebe explícitamente. Si la spec cambia criterios, contratos o plan, vuelve a `draft` y necesita otra aprobación.

### SDD o BUILD

El orquestador clasifica cada tarea (criterios completos en `.claude/agents/orchestrator.md`):

- **BUILD** (directo, sin spec): 1–3 archivos, sin módulo nuevo ni contratos nuevos. Por ejemplo fixes puntuales, estilos, config o dependencias.
- **SDD**: features, módulos nuevos, cambios en varios módulos, contratos nuevos o lógica que pide tests.

Las reglas de las secciones 1 y 2 aplican en los dos modos.

Uso recomendado: `claude --agent orchestrator`, así el orquestador habla directo con vos para aprobar specs. También se puede invocar como subagente desde una sesión normal: delega igual, pero devuelve el control al hilo principal para cada aprobación.

### Flujo

```
Requerimiento → Spec → Desarrollo (+ tests) → Review → Cierre
```

1. Se recibe el requerimiento.
2. **Spec** lo convierte en una spec escrita y se valida.
3. **Developer** implementa según la spec.
4. **Reviewer** verifica código contra la spec y las reglas de este documento.
5. Si el reviewer rechaza, vuelve a Developer, con un máximo de 3 vueltas antes de escalar al usuario. Si aprueba, se cierra.

**Planes alcanzables:** la spec se divide en fases que se verifican solas. Cada fase tiene como máximo ~5 tareas y ~10 archivos. Por sesión se hacen 1–2 fases.

**Paralelo sin conflictos:** cada tarea declara los archivos que le pertenecen (`Owns`). Las tareas de una misma fase no comparten archivos. Los archivos compartidos y las instalaciones van en una tarea de setup previa.

Las specs se guardan en `docs/specs/<module>/<feature>.md`.

### Agentes

| Agente           | Responsabilidad                                                                                                                                 | No hace                          |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| **Orquestador**  | Recibe el requerimiento, coordina el flujo, delega a los otros agentes en orden y consolida el resultado. Decide cuándo algo está terminado.   | Escribir specs ni código.        |
| **Spec**         | Convierte el requerimiento en spec: objetivo, alcance, criterios de aceptación, módulo afectado, contratos (tipos/schemas), casos borde, tests esperados. Revisa el código existente para no duplicar. | Implementar.                     |
| **Developer**    | Implementa exactamente lo que dice la spec siguiendo la sección 1 y 2. Escribe los unit tests que la spec pida.                                 | Ampliar el alcance de la spec.   |
| **Reviewer**     | Valida código y tests contra la spec y este documento (estructura, naming, SOLID/DRY/KISS/YAGNI, reutilización). Devuelve aprobado o lista de cambios. | Reescribir la implementación.    |

### Plantilla de spec

```md
# <Feature name>

- Module: <domain>
- Status: draft | approved | done
- Approved by: - | <name> (<YYYY-MM-DD>)

## Goal
## Scope (in / out)
## Acceptance criteria
## Contracts (types, Zod schemas, endpoints)
## Edge cases
## Tests (what must be covered)
## Plan (phases → tasks with Owns / Depends on / Group)
```

El formato completo está en `.claude/agents/spec.md`.

### Unit testing

- Se testea **donde aporta valor**: lógica de negocio, services, hooks con lógica, schemas Zod, utilidades y stores. No se testea markup trivial ni componentes que solo componen.
- Qué se testea lo define la spec (sección *Tests*).
- Los tests van junto al archivo que prueban: `user-service.ts` → `user-service.test.ts`.
- Un test que cubre un criterio de aceptación de la spec debe poder trazarse a ese criterio.
- El test runner todavía **no está instalado**. Se elige y configura al implementar el primer test; hasta entonces no se agrega.
