# IAAF — Documento Tecnico

> IA Architect Foreman
> Version: 1.0
> Fecha: 2026-04-03

---

## 1. Que es IAAF

IAAF es una estrategia de 3 capas para generar blueprints de software que otra instancia de Claude Code puede construir de forma autonoma y eficiente.

```
IAAF = IA + Architect + Foreman

  IA           →  Sabe (base de conocimiento)
  Architect    →  Diseña (entrevista + generacion)
  Foreman      →  Optimiza (reglas de ejecucion eficiente)
```

**Problema que resuelve:** Un humano tiene una idea de software pero no sabe (o no quiere) diseñar la arquitectura tecnica. Necesita un plan tan detallado que una IA pueda construirlo sin hacer preguntas.

**Como lo resuelve:** IAAF entrevista al humano, toma decisiones de arquitectura informadas por una base de conocimiento, genera un blueprint de 16 secciones, y embebe reglas de eficiencia para que la construccion consuma 30-40% menos tokens.

---

## 2. Actores

IAAF tiene 4 actores. 3 son capas del sistema y 1 es externo.

### Actor 1: El Humano (externo)

| Atributo | Descripcion |
|---|---|
| **Quien es** | La persona que tiene la idea del proyecto. Puede ser un fundador, un developer, un product manager, o alguien sin conocimiento tecnico. |
| **Que hace** | Describe lo que quiere construir. Responde preguntas. Confirma o ajusta la arquitectura propuesta. |
| **Que NO hace** | No toma decisiones tecnicas (a menos que quiera). No escribe codigo. No elige el stack (a menos que tenga preferencias). |
| **Cuando participa** | Phases 1, 2 y 3 (entrevista y confirmacion). NO participa en Phase 4 (generacion). |
| **Que recibe** | Un archivo `.md` (blueprint) que puede copiar como `CLAUDE.md` en un nuevo proyecto. |

**Perfiles tipicos del Humano:**

| Perfil | Como se comporta | Como IAAF se adapta |
|---|---|---|
| Fundador no-tecnico | "Quiero una app como Uber para mascotas" | The Architect toma todas las decisiones, explica en terminos simples |
| Developer junior | "Quiero un SaaS con Next.js pero no se como estructurarlo" | The Architect respeta la preferencia de Next.js, decide el resto |
| Developer senior | "Necesito un API con Hono, Drizzle, PostgreSQL en Railway. Solo dame el build order." | Fast-track mode: 3 preguntas, smart defaults, genera rapido |
| Product manager | "Necesito un dashboard interno para el equipo de ventas con estos KPIs" | The Architect traduce requerimientos de negocio a arquitectura |

---

### Actor 2: IA (Capa de Inteligencia)

| Atributo | Descripcion |
|---|---|
| **Que es** | La base de conocimiento estatica del sistema. 35 archivos Markdown con ~26,000 palabras. |
| **Rol** | Proveer informacion estructurada para que The Architect tome decisiones informadas. |
| **Cuando actua** | No "actua" — es consultada por The Architect durante las 4 fases. |
| **Analogia** | Es la biblioteca de referencia del arquitecto. No diseña nada, pero contiene todo lo que el arquitecto necesita saber. |

**Componentes de la capa IA:**

#### 2a. Arquetipos (10 archivos)

Un arquetipo es una **plantilla de referencia para un tipo de proyecto**. Cada uno contiene:

| Seccion | Que dice | Por que existe |
|---|---|---|
| Stack default | Que tecnologias usar y por que | Evita que The Architect "invente" un stack desde cero. Usa combinaciones probadas. |
| Alternativas | Cuando cambiar cada tecnologia | Permite adaptarse sin desviarse a combinaciones riesgosas. |
| Directory structure | Como organizar el codigo | Cada tipo de proyecto tiene una estructura diferente. Un API no se organiza como un SaaS. |
| Common patterns | Como implementar auth, data, billing, etc. | Patrones que el 80% de los proyectos de ese tipo necesitan. |
| Build order | En que orden construir, paso a paso | La pieza mas critica. Sin esto, Claude Code no sabe por donde empezar. |
| Common pitfalls | Errores frecuentes a evitar | Experiencia condensada. "No hagas X porque Y." |
| Skills | Que skills de Claude Code ayudan | Conecta el blueprint con herramientas especificas. |

**Los 10 arquetipos:**

| Arquetipo | Resuelve | Stack default |
|---|---|---|
| `saas-webapp` | Apps con cuentas, suscripciones, dashboards | Next.js + Supabase + Clerk + Stripe + Vercel |
| `marketing-site` | Landing pages, portfolios, sitios estaticos | Astro + Tailwind + Sanity + Vercel |
| `mobile-app` | Apps iOS/Android | Expo + NativeWind + Supabase |
| `api-backend` | APIs REST, microservicios | Hono + Drizzle + PostgreSQL + Railway |
| `internal-tool` | Paneles admin, dashboards internos | Next.js + shadcn + Prisma + Recharts |
| `content-platform` | Blogs, docs, CMS | Next.js + Sanity + Algolia + Vercel |
| `e-commerce` | Tiendas online | Next.js + Medusa + Stripe + Cloudinary |
| `ai-ml-app` | Chat AI, RAG, apps con LLM | Next.js + Vercel AI SDK + Pinecone |
| `cli-tool` | Herramientas de terminal | Node.js + Commander + Inquirer |
| `realtime-collab` | Edicion colaborativa en tiempo real | Next.js + Liveblocks + Yjs + Tiptap |

**Como se usa un arquetipo:**
1. The Architect clasifica el proyecto del Humano en 1 de los 10 arquetipos
2. Lee ESE archivo (no los otros 9)
3. Usa el stack default como punto de partida
4. Adapta segun las respuestas del Humano en Phase 2
5. El build order del arquetipo se convierte en la base de la Seccion 9 del blueprint

**Proyectos hibridos:** Cuando un proyecto es dos cosas (ej: SaaS + marketing site), se usa el arquetipo primario como base y se mergea el secundario siguiendo `knowledge/hybrid-patterns.md`.

#### 2b. Building Blocks (12 archivos)

Un building block es una **guia de decision transversal** que aplica a multiples arquetipos. Cada uno contiene:

| Seccion | Que dice |
|---|---|
| Decision matrix | Tabla comparativa con opciones, pros, cons, cuando usar |
| Recommendation by project type | "Para SaaS usa X, para API usa Y" |
| Implementation patterns | Codigo concreto de como implementar la opcion recomendada |
| Common pitfalls | Errores a evitar |

**Los 12 building blocks:**

| Building Block | Que resuelve | Cuando se lee |
|---|---|---|
| `auth-patterns` | Que auth usar (Clerk, NextAuth, Supabase Auth, JWT...) | Proyecto tiene cuentas de usuario |
| `database-patterns` | Que DB y ORM usar (Postgres, Prisma, Drizzle...) | Proyecto tiene base de datos |
| `api-design-patterns` | REST vs tRPC vs GraphQL vs Server Actions | Proyecto expone o consume una API |
| `frontend-patterns` | Framework, styling, state management, rendering | Proyecto tiene UI |
| `testing-patterns` | Que testear y con que (Vitest, Playwright) | Cualquier proyecto de produccion |
| `payments-patterns` | Stripe, suscripciones, webhooks, billing | Proyecto tiene pagos |
| `integrations-patterns` | Email, push, in-app notifications, file uploads | Proyecto envia emails o maneja archivos |
| `security-ops-patterns` | CORS, rate limiting, Sentry, logging, health checks | Cualquier proyecto de produccion |
| `deployment-patterns` | Hosting, CI/CD, GitHub Actions, preview deploys | Cualquier proyecto que se deploya |
| `i18n-patterns` | Internacionalizacion, locale routing, traducciones | Proyecto multilenguaje |
| `performance-patterns` | Core Web Vitals, imagenes, bundles, caching | Proyecto publico o performance-critical |
| `claude-behavior-patterns` | **The Foreman** — reglas de eficiencia para el builder | SIEMPRE en Phase 4 |

**Diferencia clave entre arquetipo y building block:**
- **Arquetipo** = vertical. Cubre TODO sobre un TIPO de proyecto.
- **Building block** = horizontal. Cubre TODO sobre una DECISION que cruza multiples tipos.

```
                   auth   db   api   frontend   payments   deploy
                   ────   ──   ───   ────────   ────────   ──────
  SaaS             ████   ██   ███   ████████   ████████   ██████
  Marketing        ░░░░   ░░   ░░░   ████████   ░░░░░░░░   ██████
  Mobile           ████   ██   ███   ████████   ░░░░░░░░   ██████
  API              ████   ██   ███   ░░░░░░░░   ░░░░░░░░   ██████
  
  ████ = building block relevante para ese arquetipo
  ░░░░ = no se carga
```

#### 2c. Archivos de Soporte (4 archivos)

| Archivo | Que hace | Cuando se lee |
|---|---|---|
| `index.md` | **Indice de toda la base de conocimiento.** 1 linea por archivo con condicion de carga. Evita que The Architect abra archivos innecesarios. | Phase 2 y Phase 4 — SIEMPRE antes de cargar building blocks |
| `stack-compatibility.md` | 10 combinaciones de tecnologia probadas + 10 anti-patrones. Ej: "Prisma no funciona en Cloudflare Workers, usa Drizzle." | Phase 3 — al validar el stack propuesto |
| `hybrid-patterns.md` | Como combinar 2 arquetipos (merge de stacks, directorios, build orders) | Phase 1 — cuando el proyecto es hibrido |
| `blueprint-checklist.md` | Checklist de QA con ~40 items agrupados por tipo (frontend, DB, auth, payments). | Phase 4 — antes de guardar el blueprint |

#### 2d. Skills Registry (1 archivo)

`skills-registry.md` mapea 11 skills de Claude Code a fases y secciones del blueprint. Le dice a The Architect:
- Que skills usar EL MISMO durante el diseno (ej: `/deep-research` en Phase 2)
- Que skills RECOMENDAR en el blueprint para el builder (ej: `/frontend-design` en Seccion 14)

---

### Actor 3: The Architect (Capa de Diseno)

| Atributo | Descripcion |
|---|---|
| **Que es** | Una instancia de Claude Code configurada mediante `CLAUDE.md` para actuar como consultor senior de arquitectura. |
| **Rol** | Entrevistar al Humano, tomar decisiones de arquitectura, generar el blueprint. |
| **Que NO hace** | No escribe codigo. No construye el proyecto. No implementa nada. Solo diseña. |
| **Personalidad** | Arquitecto confiado que recomienda, no asistente que lista opciones. "Yo usaria X porque Y" — no "Podrias usar X, Y o Z." |
| **Idioma** | Detecta el idioma del Humano en su primer mensaje y usa ese idioma para toda la interaccion y el blueprint. |

**El Architect opera en 4 fases secuenciales:**

#### Phase 1: DISCOVERY (Entender)

| Aspecto | Detalle |
|---|---|
| **Input** | El primer mensaje del Humano + `phase-1-discovery.md` |
| **Que hace** | Pregunta 2-3 preguntas abiertas: que es, para quien, que etapa, preferencias tech |
| **Decision clave** | Clasifica el proyecto en 1 de 10 arquetipos basandose en keywords |
| **Output** | Arquetipo identificado. Lee el archivo del arquetipo correspondiente. |
| **Reglas** | Max 3 preguntas por mensaje. Si el Humano dice "just build it" → fast-track mode (3 preguntas esenciales, smart defaults). |
| **Archivos leidos** | `CLAUDE.md`, `phase-1-discovery.md`, 1 arquetipo, opcionalmente `hybrid-patterns.md` |

**Preguntas disponibles (8):**

| # | Pregunta | Obligatoria | Que determina |
|---|---|---|---|
| Q1 | Que estas construyendo? | Si | El arquetipo |
| Q2 | Para quien es? | Si | Escala, complejidad de UX |
| Q3 | Que etapa? MVP o produccion? | Si | Scope del blueprint |
| Q4 | Preferencias de tech? | Si en fast-track | Overrides al stack default |
| Q5 | Timeline? | Opcional | Ambicion del build order |
| Q6 | Presupuesto para servicios? | Opcional | Clerk ($) vs NextAuth (gratis), etc. |
| Q7 | Quien construye esto? | Opcional | Complejidad del stack |
| Q8 | Compliance? GDPR, HIPAA? | Opcional | Hosting, encryption, audit logging |

#### Phase 2: DEEP DIVE (Profundizar)

| Aspecto | Detalle |
|---|---|
| **Input** | Arquetipo identificado + `phase-2-branches.md` + `knowledge/index.md` |
| **Que hace** | Pregunta 3-5 preguntas especificas al tipo de proyecto. Carga building blocks segun las respuestas. |
| **Decision clave** | Que building blocks son relevantes para ESTE proyecto especifico |
| **Output** | Conocimiento detallado del proyecto: features, auth, payments, real-time, integraciones. |
| **Reglas** | Skip preguntas ya respondidas. Usa `index.md` para decidir que cargar. NO carga todo. |
| **Skills usadas** | `/deep-research` si hay tech desconocida. `/find-skills` una vez para descubrir skills para el builder. |

**Ejemplo de carga condicional:**

```
Humano dice: "SaaS con pagos y login social"

The Architect:
  Lee: saas-webapp.md                    (arquetipo)
  Lee: index.md                          (indice — decide que mas cargar)
  Lee: auth-patterns.md                  (porque hay login)
  Lee: payments-patterns.md              (porque hay pagos)
  Lee: database-patterns.md              (SaaS siempre tiene DB)
  NO lee: i18n-patterns.md               (no menciono idiomas)
  NO lee: performance-patterns.md        (no es el foco)
  NO lee: integrations-patterns.md       (no menciono email/archivos)
```

#### Phase 3: ARCHITECTURE (Decidir)

| Aspecto | Detalle |
|---|---|
| **Input** | Todo lo recopilado en Phases 1-2 + `phase-3-confirmation.md` + `stack-compatibility.md` |
| **Que hace** | Presenta la arquitectura propuesta: tech stack con razones, features del V1, approach de build. Todo en 1 mensaje, < 40 lineas. |
| **Decision clave** | El Humano confirma o pide cambios |
| **Output** | Arquitectura aprobada. Si tiene frontend, invoca `/ui-ux-pro-max` para diseñar el sistema visual. |
| **Reglas** | Max 2 rondas de ajuste. Si no hay alineacion, preguntar directamente cual es el punto de friccion. |

**Formato de presentacion:**

```
"Here's what I'd build:

| Layer       | Technology              | Why                              |
| Framework   | Next.js 15 (App Router) | Full-stack, Vercel deploy        |
| Auth        | Clerk                   | Saves 2 days vs custom auth      |
| Database    | Supabase (Postgres)     | Auth + DB + Realtime in one      |
| Payments    | Stripe                  | Industry standard, Checkout UI   |
| Hosting     | Vercel                  | Zero-config for Next.js          |

V1 includes: auth, core CRUD, dashboard, Stripe billing, landing page.
V1 does NOT include: real-time, mobile, i18n.

Build approach: auth first, then data model, then core feature, then payments, then polish.

Does this match your vision?"
```

#### Phase 4: GENERATE (Producir)

| Aspecto | Detalle |
|---|---|
| **Input** | Arquitectura aprobada + templates + `claude-behavior-patterns.md` + `blueprint-checklist.md` |
| **Que hace** | Genera el blueprint completo de 16 secciones + diagramas Mermaid. Embebe The Foreman en la Seccion 15. |
| **Decision clave** | Ninguna — todas las decisiones ya fueron tomadas. Esto es pura generacion. |
| **Output** | Archivo `output/<nombre>-blueprint.md` |
| **QA** | Lee `blueprint-checklist.md` y verifica ~40 items antes de guardar |

**Archivos leidos en Phase 4:**

| Archivo | Para que |
|---|---|
| `blueprint-template.md` | Esqueleto de 16 secciones |
| `claude-md-template.md` | Template del CLAUDE.md embebido (con The Foreman) |
| `skills-registry.md` | Skills a recomendar en Seccion 14 |
| `claude-behavior-patterns.md` | Reglas universales + por arquetipo para The Foreman |
| `blueprint-checklist.md` | QA gate — verificar completitud |

**Las 16 secciones del blueprint:**

| # | Seccion | Generada por | Informada por |
|---|---|---|---|
| 1 | Project Overview | Phase 1 | Respuestas del Humano |
| 2 | Tech Stack | Phase 3 | Arquetipo + building blocks + stack-compatibility |
| 3 | Directory Structure | Phase 3 | Arquetipo (default structure) |
| 4 | Data Model | Phase 2 | database-patterns + respuestas del Humano |
| 4b | Architecture Diagrams | Phase 4 | Todo lo anterior, renderizado en Mermaid |
| 5 | API Design | Phase 2 | api-design-patterns + arquetipo |
| 6 | Frontend Architecture | Phase 3 | frontend-patterns + arquetipo |
| 7 | Design System | Phase 3 | `/ui-ux-pro-max` + respuestas del Humano |
| 8 | Auth & Authorization | Phase 2 | auth-patterns + arquetipo |
| 9 | Build Order | Phase 4 | Arquetipo (base) + adaptaciones + Efficiency Guidelines |
| 10 | Environment Setup | Phase 4 | Tech stack decidido en Phase 3 |
| 11 | Dependencies | Phase 4 | Tech stack |
| 12 | Deployment Strategy | Phase 4 | deployment-patterns |
| 13 | Testing Strategy | Phase 4 | testing-patterns |
| 14 | Skills to Use | Phase 4 | skills-registry |
| 15 | CLAUDE.md | Phase 4 | claude-md-template + The Foreman + todo lo anterior |
| 16 | Non-Negotiable Rules | Phase 4 | Decisiones criticas de Phases 1-3 |

---

### Actor 4: The Foreman (Capa de Optimizacion)

| Atributo | Descripcion |
|---|---|
| **Que es** | Un conjunto de reglas de comportamiento que se inyectan en el blueprint generado. No es una entidad que "actua" — es un set de instrucciones que otro actor (Claude Code builder) leera y seguira. |
| **Rol** | Hacer que Claude Code construya el proyecto de forma eficiente: menos output innecesario, menos turnos de API, menos operaciones repetidas. |
| **Cuando actua** | Nunca directamente. Sus reglas se activan cuando otra instancia de Claude Code lee el CLAUDE.md del proyecto destino y empieza a construir. |
| **Analogia** | Es un cartel de reglas colgado en la obra de construccion. No es una persona — es un conjunto de instrucciones que los constructores deben seguir. |

**The Foreman tiene 4 componentes:**

#### 4a. Reglas Universales (9 reglas)

Aplican a TODOS los proyectos, independientemente del tipo. Se embeben en `## The Foreman` del CLAUDE.md generado.

| # | Regla | Que ahorra |
|---|---|---|
| 1 | Sin aperturas sycofantes ni cierres vacios | Output tokens (35-45% del fluff) |
| 2 | Leer antes de escribir | Turnos (evita reescribir por no entender el contexto) |
| 3 | Preferir editar sobre reescribir | Tool calls (Edit es mas barato que Write) |
| 4 | No re-leer archivos ya en contexto | Turnos + input tokens |
| 5 | Testear antes de declarar terminado | Evita bugs que costarian turnos arreglar despues |
| 6 | No refactorizar codigo que ya pasa tests | Turnos + riesgo de bugs nuevos |
| 7 | Sin sugerencias no solicitadas | Output tokens |
| 8 | Soluciones simples (3 lineas > abstraccion prematura) | Output tokens + complejidad innecesaria |
| 9 | Instrucciones del usuario siempre ganan | Valvula de escape — el Humano tiene la ultima palabra |

#### 4b. Reglas por Arquetipo (5 sets)

Reglas adicionales especificas al tipo de proyecto. Se agregan junto a las universales.

| Arquetipo | Reglas adicionales |
|---|---|
| SaaS / Internal Tool | Code first, explain later. State bug, show fix, stop. Write CRUD in one pass. |
| API / Backend | Response shape consistente. Tests junto a routes. Validate at boundary only. |
| Marketing / Landing | No JS innecesario. Build sections as self-contained units. Inline critical CSS. |
| Mobile | Check iOS + Android. Verify React Native compatibility before install. |
| Content / CMS | Server-render all content. SEO metadata at creation time, not later. Use ISR by default. |

#### 4c. Build Efficiency Guidelines

Se embeben en la Seccion 9 (Build Order) como preambulo antes de los pasos numerados. Definen el patron de ejecucion por paso:

```
READ  →  Leer todos los archivos relevantes antes de escribir
PLAN  →  Planear la solucion completa antes de tocar codigo
WRITE →  Implementar. Simple en un solo paso, complejo incrementalmente.
TEST  →  Testear una vez. Si pasa, avanzar.
STOP  →  No refactorizar ni mejorar. Siguiente paso.
```

#### 4d. Tool Call Awareness

Se embebe como seccion en el CLAUDE.md generado. 5 guias para reducir el numero de tool calls:

| Guia | Efecto |
|---|---|
| Preferir operaciones grandes sobre muchas pequeñas | Menos turnos de API |
| Leer todos los archivos al inicio del paso, no uno por uno | Consolida lecturas en 1 turno |
| Escribir archivos completos en una operacion | Menos tool calls |
| Si un paso pasa de 15 tool calls, replantear el approach | Fuerza a pensar antes de actuar |
| Es una guia, no un limite duro. Correctness > speed. | Valvula de escape — no sacrificar calidad |

---

## 3. Flujo Completo de Datos

```
HUMANO                    THE ARCHITECT                    BLUEPRINT
  │                           │                               │
  │  "Quiero un SaaS de..."  │                               │
  │ ─────────────────────>   │                               │
  │                           │                               │
  │                    ┌──────┴──────┐                        │
  │                    │  PHASE 1    │                        │
  │                    │  Lee:       │                        │
  │                    │  - CLAUDE.md│                        │
  │                    │  - phase-1  │                        │
  │                    │  Clasifica: │                        │
  │                    │  saas-webapp│                        │
  │                    │  Lee:       │                        │
  │                    │  - arquetipo│                        │
  │                    └──────┬──────┘                        │
  │                           │                               │
  │  <── "Necesitas pagos?    │                               │
  │       Roles? Real-time?"  │                               │
  │                           │                               │
  │  "Si pagos, no real-time" │                               │
  │ ─────────────────────>   │                               │
  │                           │                               │
  │                    ┌──────┴──────┐                        │
  │                    │  PHASE 2    │                        │
  │                    │  Lee:       │                        │
  │                    │  - index.md │                        │
  │                    │  - phase-2  │                        │
  │                    │  Carga:     │                        │
  │                    │  - auth     │                        │
  │                    │  - payments │                        │
  │                    │  - database │                        │
  │                    └──────┬──────┘                        │
  │                           │                               │
  │                    ┌──────┴──────┐                        │
  │                    │  PHASE 3    │                        │
  │                    │  Lee:       │                        │
  │                    │  - phase-3  │                        │
  │                    │  - compat   │                        │
  │                    │  Usa:       │                        │
  │                    │  /ui-ux-pro │                        │
  │                    └──────┬──────┘                        │
  │                           │                               │
  │  <── "Propongo Next.js +  │                               │
  │       Supabase + Clerk +  │                               │
  │       Stripe en Vercel"   │                               │
  │                           │                               │
  │  "Confirmo."              │                               │
  │ ─────────────────────>   │                               │
  │                           │                               │
  │                    ┌──────┴──────┐                        │
  │                    │  PHASE 4    │                        │
  │                    │  Lee:       │                        │
  │                    │  - template │                        │
  │                    │  - claude-md│                        │
  │                    │  - skills   │                        │
  │                    │  - behavior │                        │
  │                    │  - checklist│                        │
  │                    │             │                        │
  │                    │  Genera 16  │                        │
  │                    │  secciones  │ ──────────────────>    │
  │                    │  + Foreman  │        blueprint.md    │
  │                    │  + QA check │                        │
  │                    └──────┬──────┘                        │
  │                           │                               │
  │  <── "Blueprint generado  │                               │
  │       en output/saas.md"  │                               │
  │                           │                               │
  │                           │                               │
  ▼                           ▼                               ▼

              *** SESION 2: CONSTRUCCION ***

HUMANO                    CLAUDE CODE (BUILDER)             PROYECTO
  │                           │                               │
  │  Copia blueprint como     │                               │
  │  CLAUDE.md en nuevo       │                               │
  │  proyecto. Abre Claude.   │                               │
  │ ─────────────────────>   │                               │
  │                           │                               │
  │                    ┌──────┴──────┐                        │
  │                    │  Lee CLAUDE │                        │
  │                    │  .md que    │                        │
  │                    │  contiene:  │                        │
  │                    │             │                        │
  │                    │  The Foreman│  ← reglas de eficiencia│
  │                    │  Build Order│  ← que construir       │
  │                    │  Tech Stack │  ← que tecnologias     │
  │                    │  Design     │  ← como se ve          │
  │                    │  Rules      │  ← que no violar       │
  │                    │             │                        │
  │                    │  Construye  │                        │
  │                    │  paso a paso│ ──────────────────>    │
  │                    │  siguiendo  │      codigo real       │
  │                    │  el build   │      archivos          │
  │                    │  order      │      configs           │
  │                    │             │      tests             │
  │                    │  Con 30-40% │                        │
  │                    │  menos      │                        │
  │                    │  tokens     │                        │
  │                    └─────────────┘                        │
```

---

## 4. Consumo de Tokens

### Por que importa

Cada turno de Claude Code agrega ~40,000 tokens de cache-read. El costo total de una sesion es:

```
Costo = (tokens input por turno) x (numero de turnos) + (tokens output por turno) x (numero de turnos)
```

IAAF optimiza ambos ejes:
- **La capa IA** reduce input innecesario con lazy loading via `index.md`
- **The Foreman** reduce output innecesario (fluff) y turnos (one-pass, no re-read, no refactor)

### Consumo de la sesion de diseno (The Architect)

| Fase | Archivos leidos | Tokens aprox |
|---|---|---|
| Phase 1 | CLAUDE.md + phase-1 + 1 arquetipo | ~2,300 |
| Phase 2 | index.md + phase-2 (1 seccion) + 2-4 building blocks | ~3,900 |
| Phase 3 | phase-3 + stack-compatibility | ~1,400 |
| Phase 4 | 2 templates + skills + behavior + checklist | ~4,900 |
| **Total input de knowledge** | | **~12,500** |

Para referencia: si leyera TODOS los 35 archivos serian ~35,000 tokens. El lazy loading via `index.md` ahorra ~64%.

### Consumo de la sesion de construccion (Builder con Foreman)

| Concepto | Sin Foreman | Con Foreman | Reduccion |
|---|---|---|---|
| Output tokens por turno | ~800 | ~500 | -35% |
| Turnos para un build completo (12 pasos) | 150-250 | 100-160 | -35% |
| Cache-read overhead total | ~8M tokens | ~5.2M tokens | -35% |

---

## 5. Que Contiene el Blueprint (el producto final)

El blueprint es un archivo `.md` de ~500-800 lineas con 16 secciones + diagramas. Es **100% autocontenido** — Claude Code no necesita nada mas para construir.

| Seccion | Lineas aprox | Fuente principal |
|---|---|---|
| 1. Project Overview | 15 | Humano (Phase 1) |
| 2. Tech Stack | 15 | Arquetipo + building blocks (Phase 3) |
| 3. Directory Structure | 30 | Arquetipo |
| 4. Data Model | 40 | Humano (Phase 2) + database-patterns |
| 4b. Architecture Diagrams | 30 | Generado en Phase 4 (Mermaid) |
| 5. API Design | 25 | api-design-patterns + Humano |
| 6. Frontend Architecture | 25 | frontend-patterns + Humano |
| 7. Design System | 30 | `/ui-ux-pro-max` (Phase 3) |
| 8. Auth & Authorization | 20 | auth-patterns |
| 9. Build Order | 60 | Arquetipo + Efficiency Guidelines |
| 10. Environment Setup | 25 | Tech stack |
| 11. Dependencies | 20 | Tech stack |
| 12. Deployment Strategy | 15 | deployment-patterns |
| 13. Testing Strategy | 15 | testing-patterns |
| 14. Skills | 10 | skills-registry |
| 15. CLAUDE.md (embebido) | 80 | claude-md-template + The Foreman |
| 16. Non-Negotiable Rules | 10 | Decisiones criticas |
| **Total** | **~465** | |

La Seccion 15 (CLAUDE.md embebido) es lo que Claude Code lee directamente. Contiene The Foreman, el stack, la arquitectura, las reglas — todo en < 120 lineas optimizadas para que el builder trabaje sin preguntar.

---

## 6. Restricciones y Limites

| Restriccion | Detalle |
|---|---|
| **Sesgo tecnologico** | IAAF favorece el ecosistema JavaScript/TypeScript (Next.js, React, Node.js). Soporte para Python y Go existe solo en el arquetipo api-backend. No hay soporte para Java, Ruby, PHP, Rust (excepto CLI). |
| **Dependencia de Claude Code** | El sistema esta diseñado para Claude Code especificamente. No funciona con otros asistentes de IA. |
| **Skills de terceros** | Varias skills referenciadas (`/ui-ux-pro-max`, `/deep-research`, `/frontend-design`) son de terceros y pueden no estar disponibles o cambiar. |
| **No genera codigo** | IAAF genera planos, no codigo. La calidad del codigo final depende de Claude Code (builder), no de IAAF. |
| **No valida post-build** | No hay mecanismo para verificar si el proyecto construido desde el blueprint funciona correctamente. |
| **Benchmark no probado a escala** | El 30-40% de ahorro de The Foreman es una estimacion basada en benchmarks pequenos, no en mediciones de produccion con proyectos completos. |
| **Base de conocimiento estatica** | Los stacks recomendados (Next.js 15, Tailwind v4, etc.) quedaran desactualizados con el tiempo. Requiere mantenimiento manual. |
