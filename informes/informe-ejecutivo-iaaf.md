# IAAF — Informe Ejecutivo y Plan de Mejora

> Fecha de revision: 2026-04-02
> Revisado por: Claude Code (Opus 4.6)
> Proyecto: [IAAF](https://github.com/rickpadro/iaaf) por SinapSYS Ecosistemas
> Licencia: MIT

---

# PARTE I: INFORME EJECUTIVO

---

## 1. Que es IAAF

IAAF es un **meta-agente para Claude Code** que actua como consultor senior de arquitectura de software. El flujo es simple:

```
Usuario describe una idea
       |
IAAF entrevista al usuario (3 fases)
       |
Genera un blueprint .md de 16 secciones
       |
Otra instancia de Claude Code construye el proyecto entero desde el blueprint
```

**Propuesta de valor:** Eliminar la brecha entre "tengo una idea" y "tengo un plan tecnico completo". El usuario no necesita saber arquitectura de software — IAAF toma las decisiones y las justifica.

---

## 2. Inventario de Componentes

### 2.1 Nucleo del Sistema

| Componente | Archivo | Funcion |
|---|---|---|
| System Prompt | `CLAUDE.md` (100 lineas) | Convierte a Claude en IAAF. Define las 4 fases del workflow, 10 reglas no negociables, integracion de skills y estilo conversacional |

**Observaciones sobre el CLAUDE.md:**
- Define 4 fases claras: Discovery, Deep Dive, Architecture, Generate
- Limita a 3 preguntas por mensaje (evita interrogatorios)
- Obliga a presentar la arquitectura para confirmacion antes de generar
- Incluye fast-track mode para usuarios que quieren velocidad
- El tono es "arquitecto confiado" no "asistente sumiso" — decision acertada
- Detecta idioma del usuario automaticamente

---

### 2.2 Base de Conocimiento: Arquetipos (6)

Cada arquetipo es una plantilla completa que incluye: stack por defecto con alternativas, estructura de directorios, patrones comunes, build order, pitfalls y skills recomendadas.

| Arquetipo | Archivo | Stack Default | Lineas |
|---|---|---|---|
| SaaS / Web App | `saas-webapp.md` | Next.js 15 + Supabase + Clerk + Stripe + Vercel | 125 |
| Marketing Site | `marketing-site.md` | Astro 5 + Tailwind + Sanity + Vercel | 117 |
| Mobile App | `mobile-app.md` | Expo (React Native) + Supabase + NativeWind | 130 |
| API / Backend | `api-backend.md` | Hono + Drizzle + PostgreSQL + Railway | 134 |
| Internal Tool | `internal-tool.md` | Next.js 15 + shadcn/ui + Prisma + Recharts | 117 |
| Content Platform | `content-platform.md` | Next.js 15 + Sanity + Algolia + Vercel | 136 |

**Analisis de calidad por arquetipo:**

**SaaS Web App** — El mas completo. Cubre auth flow (Clerk), billing (Stripe webhooks), data layer (Server Components + Server Actions), y un build order de 12 pasos. Incluye advertencias criticas como "no construir auth desde cero" y "agregar rate limiting desde el dia uno".

**Marketing Site** — Buena eleccion de Astro como default (zero JS). Enfasis correcto en performance y SEO. El build order prioriza design system antes que contenido. Incluye patron de React islands para formularios interactivos.

**Mobile App** — Solida cobertura de Expo/React Native. Incluye flujos de OTA updates (EAS Update), submission a app stores, y offline support. El build order de 11 pasos es practico. Debilidad: no cubre Flutter ni desarrollo nativo.

**API / Backend** — Buena eleccion de Hono como default (ligero, edge-compatible). Incluye patron de route structure, error handling consistente, y response shape estandar. Cubre background jobs (BullMQ) y caching (Redis). El build order de 12 pasos va de scaffolding a deploy.

**Internal Tool** — Enfocado en pragmatismo ("function over form"). Patron de DataTable generico con TanStack Table es acertado. Incluye exports (CSV/PDF) que muchas guias olvidan. Consejo clave: "no builds real-time unless needed, polling every 30s is fine".

**Content Platform** — Cubre tanto editorial (CMS-driven) como UGC (user-generated content). Incluye search (Algolia/Meilisearch), SEO critico para contenido (JSON-LD, RSS, sitemap), y patron de content rendering (Portable Text / MDX).

---

### 2.3 Base de Conocimiento: Building Blocks (8)

Guias de decision transversales que aplican a multiples arquetipos.

| Building Block | Archivo | Contenido Principal |
|---|---|---|
| Auth Patterns | `auth-patterns.md` | 7 soluciones comparadas (Clerk, NextAuth, Supabase Auth, Firebase, Lucia, API Keys, JWT custom). Patrones de implementacion, roles, session management |
| Database Patterns | `database-patterns.md` | 6 bases de datos + 4 ORMs comparados. Convenciones de naming, campos requeridos, soft deletes, migration strategy, performance patterns |
| Deployment Patterns | `deployment-patterns.md` | 6 plataformas comparadas. CI/CD en 3 niveles (minimal, standard, enterprise). Gestion de variables de entorno con ejemplos concretos |
| API Design Patterns | `api-design-patterns.md` | 4 estilos (REST, tRPC, GraphQL, Server Actions) con recomendacion por tipo de proyecto. Convenciones REST detalladas, response shapes, pagination, validation |
| Frontend Stacks | `frontend-stacks.md` | 6 frameworks comparados. Estrategias de rendering (SSR, SSG, ISR, Client). Component libraries. Patrones de data fetching y forms en Next.js 15 |
| Testing Patterns | `testing-patterns.md` | Recomendacion por tipo de proyecto (MVP = skip tests, Production = unit + E2E). Setup de Vitest y Playwright. Principios pragmaticos |
| Styling Systems | `styling-systems.md` | 5 sistemas comparados, Tailwind v4 como recomendacion universal. Arquitectura de design tokens en 3 capas. Integracion shadcn/ui. Typography scale. Responsive design |
| State Management | `state-management.md` | 9 soluciones comparadas. Stack recomendado: Server Components + Server Actions + TanStack Query + Zustand. Patrones real-time con Supabase Realtime |

**Hallazgos sobre los building blocks:**
- Todos siguen el mismo formato: decision matrix → recommendation → implementation patterns
- Las matrices de decision incluyen pros/cons/cuando usar — facilitan decisiones rapidas
- Los ejemplos de codigo son concretos y copiables
- El de testing tiene un enfoque pragmatico valioso: "MVP = skip tests, ship fast"
- El de auth es el mas completo con 7 opciones bien diferenciadas

---

### 2.4 Stack Compatibility Matrix

Archivo: `knowledge/stack-compatibility.md`

**Combinaciones probadas (6):**
- Modern SaaS Stack (Next.js + Supabase + Clerk + Stripe + Vercel)
- Lightweight SaaS Stack (Next.js + Supabase Auth + Lemonsqueezy + Vercel)
- API-First Stack (Hono + Drizzle + PostgreSQL/Neon + Railway)
- Content Stack (Astro + Sanity + Vercel)
- Mobile Stack (Expo + NativeWind + Supabase + EAS Build)
- Internal Tool Stack (Next.js + shadcn/ui + Prisma + Vercel)

**Combinaciones prohibidas (10):**
Incluye anti-patrones concretos como Prisma + Cloudflare Workers, Socket.io + Vercel, MongoDB + datos relacionales, Next.js + Express (redundante), Tailwind v3 patterns en v4.

**Tablas de compatibilidad:** Auth + Database, Hosting + Framework, ORM + Database — con justificacion para cada pareja.

Este archivo es uno de los diferenciadores mas valiosos del proyecto. Previene errores que consumirian horas de debugging.

---

### 2.5 Skills Registry

Archivo: `knowledge/skills-registry.md`

Mapea 11 skills de Claude Code a secciones del blueprint y fases del workflow:

**Skills usadas durante el diseno (por IAAF):**
- `/deep-research` — Comparar tecnologias en Phase 2
- `/ui-ux-pro-max` — Disenar sistema visual en Phase 3
- `/find-skills` — Descubrir skills para recomendar
- `/playwright-cli` y `/chrome-bridge-automation` — Analizar sitios de referencia

**Skills recomendadas para la construccion (por el builder):**
- `/frontend-design`, `/shadcn-ui`, `/seo-audit`, `/humanizer`, `/pdf-design`, `/playwright-cli`, `/web-reader`

---

### 2.6 Sistema de Preguntas (3 fases)

**Phase 1 — Discovery** (`phase-1-discovery.md`):
- 5 preguntas universales: Vision, Audiencia, Stage, Tech Preferences, Timeline
- Logica de clasificacion por keywords (ej: "subscription" → saas-webapp)
- Manejo de proyectos hibridos: usar arquetipo primario, notar secundario

**Phase 2 — Deep Dive** (`phase-2-branches.md`):
- 5 preguntas especificas por cada uno de los 6 arquetipos (30 preguntas total)
- Cada seccion indica que building blocks cargar segun las respuestas
- Preguntas adaptativas: "skip questions they've already answered"

**Phase 3 — Confirmation** (`phase-3-confirmation.md`):
- Presenta arquitectura propuesta en formato compacto (< 40 lineas)
- 3 preguntas: Architecture Fit, Design Direction (solo si hay UI), Hard Constraints
- Regla de no loopear mas de 2 veces — si no hay alineacion, preguntar directamente

---

### 2.7 Templates de Output

**Blueprint Template** (`blueprint-template.md`) — 16 secciones:

| # | Seccion | Que contiene |
|---|---------|---|
| 1 | Project Overview | Vision, goals, success metrics |
| 2 | Tech Stack | Tabla de cada tecnologia con razon |
| 3 | Directory Structure | Arbol completo con comentarios |
| 4 | Data Model | Entidades, campos, relaciones, SQL schema |
| 5 | API Design | Routes, endpoints, request/response, validacion |
| 6 | Frontend Architecture | Pages, component hierarchy, state management |
| 7 | Design System | Colors, typography, spacing, component style |
| 8 | Auth & Authorization | Flow, protected routes, roles, sessions |
| 9 | **Build Order** | Pasos numerados de cero a deployed (seccion mas critica) |
| 10 | Environment Setup | Prerequisites, env vars, comandos iniciales |
| 11 | Dependencies | Paquetes core y dev con proposito |
| 12 | Deployment Strategy | Hosting, CI/CD, domains, environments |
| 13 | Testing Strategy | Unit, integration, E2E — que testear y cuando |
| 14 | Skills to Use | Skills de Claude Code para cada paso del build |
| 15 | CLAUDE.md | Instrucciones completas para el builder (< 120 lineas) |
| 16 | Reglas No Negociables | Restricciones que no se pueden violar |

**CLAUDE.md Template** (`claude-md-template.md`):
- Template para la seccion 15 del blueprint
- Secciones: Commands, Tech Stack, Architecture, Code Organization Rules, Design System, Environment Variables, Reglas
- Guias de generacion: < 120 lineas, dense, valores concretos (hex codes, px sizes)

---

### 2.8 Documentacion

**README.md** (496 lineas):
- Bilingue completo (ingles + espanol)
- Analogia de construccion de casa para explicar el concepto
- Quick start en 5 pasos
- Tabla de 6 tipos de proyecto soportados
- Estructura del proyecto documentada
- Integracion de skills documentada
- Guia de contribucion

**LICENSE:** MIT standard

**.gitignore:** Cubre output/*.md, OS files, editor files, .claude/

---

## 3. Evaluacion por Criterio

### 3.1 Arquitectura del Sistema — 8.0 / 10

| Fortaleza | Detalle |
|---|---|
| Modularidad | Cada pieza (arquetipos, building blocks, questions, templates) es independiente y extensible |
| Flujo claro | 4 fases secuenciales bien definidas con inputs/outputs explicitos |
| CLAUDE.md efectivo | 100 lineas, conciso, con reglas accionables y estilo conversacional definido |
| Principio de single responsibility | Cada archivo tiene un proposito unico |

| Debilidad | Detalle |
|---|---|
| Sin versionado | No hay forma de trackear versiones de blueprints o del sistema |
| Sin validacion | No hay checklist para verificar que un blueprint generado este completo |
| Sin feedback loop | No hay mecanismo para aprender de blueprints que funcionaron o fallaron |

---

### 3.2 Cobertura de Arquetipos — 7.0 / 10

| Fortaleza | Detalle |
|---|---|
| Los 6 arquetipos cubren ~75% de proyectos web/mobile comunes | |
| Cada uno tiene un build order paso a paso | El build order es la pieza mas valiosa |
| Alternativas documentadas con criterio de cuando cambiar | No es "solo usa X", es "usa X, o Y si..." |

| Debilidad | Detalle |
|---|---|
| Falta E-commerce | Shopify headless, Medusa, Saleor — un vertical enorme |
| Falta AI/ML App | LLM integrations, vector DBs, RAG — el tipo de proyecto de mayor crecimiento |
| Falta CLI Tool | Herramientas de terminal son un caso de uso comun |
| Falta Real-time/Collaboration | Apps tipo Figma, Google Docs, chat — requieren arquitectura distinta |
| Falta Extension/Plugin | Browser extensions, VS Code extensions, Slack bots |
| Falta Monorepo | Proyectos con turborepo/nx que combinan frontend + backend + packages |
| Mobile limitado | Solo React Native/Expo. Sin Flutter ni nativo |

---

### 3.3 Building Blocks — 8.5 / 10

| Fortaleza | Detalle |
|---|---|
| Matrices de decision consistentes | Todas con pros/cons/cuando usar |
| Pragmatismo | "MVP = skip tests" es un consejo valiente y correcto |
| Codigo concreto | Patrones con snippets copiables, no solo teoria |
| Stack compatibility | Diferenciador unico — previene errores costosos |

| Debilidad | Detalle |
|---|---|
| Falta Payments/Billing | Stripe webhooks, subscription logic, billing portal — se menciona en arquetipos pero no tiene su propio building block |
| Falta Email/Notifications | Resend, push notifications, in-app notifications |
| Falta File Storage/Media | S3, Uploadthing, Cloudinary, procesamiento de imagenes |
| Falta Security | CORS, CSP, rate limiting avanzado, OWASP basics |
| Falta Observability | Sentry, logging, monitoring, alertas |
| Falta i18n | next-intl, locale routing, pluralizacion |

---

### 3.4 Flujo de Preguntas — 7.5 / 10

| Fortaleza | Detalle |
|---|---|
| Max 3 preguntas por mensaje | Evita abrumar al usuario |
| Clasificacion por keywords | Rapida y precisa en la mayoria de casos |
| Preguntas adaptativas | "Skip questions they've already answered" |
| Fast-track mode | 3 preguntas esenciales + smart defaults |

| Debilidad | Detalle |
|---|---|
| No pregunta presupuesto | Clerk ($) vs NextAuth (gratis) depende del budget |
| No pregunta experiencia del equipo | Stack para senior != stack para junior |
| No pregunta compliance | GDPR, HIPAA, SOC2 cambian radicalmente la arquitectura |
| No genera wireframes textuales | En Phase 3 podria presentar un layout ASCII de las pantallas principales |
| Manejo debil de hibridos | Solo "nota el secundario" — no hay logica para combinar build orders |

---

### 3.5 Templates de Output — 8.0 / 10

| Fortaleza | Detalle |
|---|---|
| 16 secciones cubren todo | Desde vision hasta reglas no negociables |
| Build order como pieza central | Seccion 9 es lo que hace al blueprint ejecutable |
| CLAUDE.md embebido | Seccion 15 permite que el builder trabaje autonomamente |
| Guias de generacion claras | "< 120 lineas", "valores concretos", "no fluff" |

| Debilidad | Detalle |
|---|---|
| Inconsistencia de idioma | Seccion 16 en espanol ("Reglas No Negociables") dentro de un template en ingles |
| Sin diagramas | No hay seccion para diagramas ER, flujo de datos, o arquitectura (Mermaid) |
| Sin estimacion de scope | No hay indicacion de complejidad o tiempo por paso del build order |
| Sin seccion de riesgos | No documenta que podria fallar o donde hay deuda tecnica aceptada |

---

### 3.6 Documentacion — 9.0 / 10

| Fortaleza | Detalle |
|---|---|
| README excelente | Claro, con analogia accesible, quick start en 5 pasos |
| Bilingue | Ingles + Espanol completo |
| Estructura documentada | Arbol con explicacion de cada carpeta |

| Debilidad | Detalle |
|---|---|
| README demasiado largo | 496 lineas por duplicar en 2 idiomas |
| Sin FAQ | Preguntas frecuentes y troubleshooting |
| Sin ejemplos de blueprints | El usuario no ve que esperar hasta que ejecuta |

---

### 3.7 Diversidad Tecnologica — 6.0 / 10

| Aspecto | Estado |
|---|---|
| JavaScript/TypeScript | Excelente cobertura |
| React / Next.js | Excelente cobertura |
| Astro | Buena cobertura |
| React Native / Expo | Buena cobertura |
| Python (Django/FastAPI) | No soportado |
| Go | Mencionado como alternativa, sin guia |
| Rust | No soportado |
| Java/Spring | No soportado |
| Ruby on Rails | No soportado |
| PHP/Laravel | No soportado |
| Vue/Nuxt | Mencionado como alternativa, sin guia |
| Svelte/SvelteKit | Mencionado, sin guia |
| AWS/GCP/Azure | No soportado como hosting principal |
| Docker/Kubernetes | No soportado |
| Terraform/IaC | No soportado |

El proyecto sirve excepcionalmente bien al ecosistema JS/TS moderno. Fuera de ese ecosistema, su utilidad es limitada.

---

## 4. Calificacion Global

| Criterio | Peso | Nota | Ponderado |
|---|---|---|---|
| Arquitectura del sistema | 15% | 8.0 | 1.20 |
| Cobertura de arquetipos | 20% | 7.0 | 1.40 |
| Building blocks | 20% | 8.5 | 1.70 |
| Flujo de preguntas | 15% | 7.5 | 1.13 |
| Templates de output | 15% | 8.0 | 1.20 |
| Documentacion | 5% | 9.0 | 0.45 |
| Diversidad tecnologica | 10% | 6.0 | 0.60 |
| **Total** | **100%** | | **7.68 / 10** |

**Veredicto:** Proyecto bien diseñado y funcional dentro de su nicho (ecosistema JS/TS moderno). La estructura es solida y extensible. Las principales oportunidades de mejora estan en ampliar cobertura (arquetipos, building blocks, tecnologias) y cerrar el feedback loop.

---

---

# PARTE II: PLAN DE MEJORA

---

## Prioridad 1 — Alto Impacto, Bajo Esfuerzo

Mejoras que se pueden implementar rapidamente y que resuelven gaps evidentes.

---

### 1.1 Agregar 4 Building Blocks Faltantes (Criticos)

**Archivo:** `knowledge/building-blocks/payments-patterns.md`
**Contenido propuesto:**
- Decision matrix: Stripe vs Lemonsqueezy vs Paddle vs PayPal
- Patron de suscripciones: pricing page → checkout → webhook → subscription tracking
- Webhook handling: eventos criticos, verificacion de firma, idempotencia
- Billing portal: gestion de suscripcion por el usuario
- Metricas: MRR, churn, LTV
- Compliance: SCA (Strong Customer Authentication), invoicing

**Archivo:** `knowledge/building-blocks/file-storage-patterns.md`
**Contenido propuesto:**
- Decision matrix: Uploadthing vs Supabase Storage vs S3 vs Cloudinary vs Vercel Blob
- Patron de upload: presigned URLs, progress tracking, validacion de tipo/tamano
- Procesamiento de imagenes: resize, optimize, formatos modernos (WebP/AVIF)
- CDN y caching de assets
- Politicas de acceso: publico vs privado vs signed URLs

**Archivo:** `knowledge/building-blocks/email-notifications-patterns.md`
**Contenido propuesto:**
- Decision matrix: Resend vs SendGrid vs AWS SES vs Postmark
- Emails transaccionales: templates, variables, preview
- Push notifications: Expo Notifications vs OneSignal vs Firebase FCM
- In-app notifications: patron de bell icon, read/unread, real-time
- Webhooks salientes: retry logic, signing, dead letter queue

**Archivo:** `knowledge/building-blocks/security-patterns.md`
**Contenido propuesto:**
- OWASP Top 10 checklist para cada arquetipo
- CORS configuration por tipo de proyecto
- CSP (Content Security Policy) headers
- Rate limiting: por IP, por usuario, por endpoint
- Input sanitization: XSS, SQL injection, path traversal
- Secrets management: env vars, vaults, rotation
- HTTPS, HSTS, secure cookies

---

### 1.2 Corregir Inconsistencias de Idioma

**Problema:** El template del blueprint (`templates/blueprint-template.md`) tiene la seccion 16 en espanol ("Reglas No Negociables") pero el resto esta en ingles. El `claude-md-template.md` tambien mezcla espanol.

**Solucion propuesta:**
- Mantener todos los templates en ingles como idioma base
- La regla 8 del CLAUDE.md ya dice "Detect the user's language from their first message. Use that language for all interaction and the blueprint itself" — esto debe aplicar tambien al titulo de la seccion 16
- Cambiar "Reglas No Negociables" a "Non-Negotiable Rules" en los templates base
- Agregar una nota en el CLAUDE.md: "Translate all section titles to the user's language when generating the blueprint"

---

### 1.3 Agregar Preguntas de Contexto a Phase 1

**Archivo a modificar:** `questions/phase-1-discovery.md`

**Preguntas nuevas a agregar:**

```markdown
### Q6: Budget & Services
**Ask:** "Are you open to paid services (Clerk for auth, Vercel Pro for hosting),
or do you prefer free/open-source alternatives?"

**Why:** Determines whether to recommend Clerk ($) vs NextAuth (free),
Supabase (free tier) vs self-hosted Postgres, Vercel Pro vs Railway.

### Q7: Team Experience
**Ask:** "Who's building this after the blueprint is generated?
Are they comfortable with the recommended stack?"

**Why:** A junior developer needs simpler tools and more guardrails.
A senior team can handle more complex architectures.
Affects: framework choice, abstraction level, testing expectations.

### Q8: Compliance (ask only if relevant)
**Ask:** "Any regulatory or compliance requirements?
GDPR, HIPAA, data residency, SOC2?"

**Why:** Compliance requirements fundamentally change architecture decisions:
data hosting region, encryption, audit logging, auth provider, data deletion.
```

---

## Prioridad 2 — Alto Impacto, Esfuerzo Medio

Mejoras que requieren mas trabajo pero amplian significativamente la cobertura.

---

### 2.1 Nuevos Arquetipos (4)

**Archivo:** `knowledge/archetypes/e-commerce.md`
**Contenido:**
- Stack: Next.js 15 + Medusa.js (o Shopify Storefront API) + Stripe + Algolia + Vercel
- Alternativas: Saleor, Commerce.js, Shopify Hydrogen
- Patrones: product catalog, cart, checkout flow, order management, inventory
- SEO critico: structured data para productos, sitemap dinamico
- Build order: scaffolding → product model → catalog pages → cart → checkout → payments → order tracking → admin → deploy

**Archivo:** `knowledge/archetypes/ai-ml-app.md`
**Contenido:**
- Stack: Next.js 15 + Vercel AI SDK + OpenAI/Anthropic API + Pinecone (vector DB) + Supabase + Vercel
- Alternativas: LangChain, LlamaIndex, Weaviate, Chroma
- Patrones: RAG (Retrieval Augmented Generation), streaming responses, prompt management, embeddings pipeline, token usage tracking, rate limiting, cost management
- Build order: scaffolding → API integration → chat UI → RAG pipeline → vector store → history/sessions → usage tracking → deploy

**Archivo:** `knowledge/archetypes/cli-tool.md`
**Contenido:**
- Stack: Node.js + TypeScript + Commander.js + Inquirer.js + chalk
- Alternativas: Go (cobra), Rust (clap), Python (click/typer)
- Patrones: command structure, interactive prompts, config files, output formatting, error handling, progress indicators
- Build order: scaffolding → command parser → core logic → interactive prompts → output formatting → config management → testing → npm publish

**Archivo:** `knowledge/archetypes/realtime-collab.md`
**Contenido:**
- Stack: Next.js 15 + Liveblocks (o PartyKit/Yjs) + Supabase + Clerk + Vercel
- Alternativas: Socket.io + custom CRDT, Ably, Pusher
- Patrones: presence (quién esta online), cursors, collaborative editing (CRDT/OT), conflict resolution, undo/redo, offline sync
- Build order: scaffolding → auth → data model → real-time connection → presence → collaborative editing → conflict resolution → offline → deploy

---

### 2.2 Ampliar Diversidad Tecnologica

**Archivo a modificar:** `knowledge/archetypes/api-backend.md`

Agregar secciones de stacks alternativos completos:

```markdown
## Alternative Stack: Python (FastAPI)
| Layer | Technology |
|-------|-----------|
| Framework | FastAPI |
| Language | Python 3.12+ |
| ORM | SQLAlchemy 2.0 + Alembic |
| Validation | Pydantic v2 |
| Database | PostgreSQL |
| Testing | pytest + httpx |
| Hosting | Railway / Fly.io / AWS Lambda (Mangum) |

## Alternative Stack: Go
| Layer | Technology |
|-------|-----------|
| Framework | Chi / Gin |
| ORM | GORM / sqlc |
| Validation | go-playground/validator |
| Database | PostgreSQL |
| Testing | go test + testify |
| Hosting | Fly.io / Railway / AWS ECS |
```

**Archivo a modificar:** `knowledge/building-blocks/deployment-patterns.md`

Agregar secciones para:
- AWS (Amplify para frontend, ECS/Fargate para backend, Lambda para serverless)
- GCP (Cloud Run para containers, Firebase Hosting para frontend)
- Docker Compose para desarrollo local y self-hosted

---

### 2.3 Agregar Seccion de Diagramas al Blueprint

**Archivo a modificar:** `templates/blueprint-template.md`

Agregar seccion entre 4 (Data Model) y 5 (API Design):

```markdown
## 4.5 Architecture Diagrams

### System Architecture
{Mermaid diagram showing how frontend, API, database, auth,
and external services connect}

### Data Model (ER Diagram)
{Mermaid ER diagram showing entities and relationships}

### Auth Flow
{Mermaid sequence diagram showing signup/login/protected route flow}

### Core Feature Flow
{Mermaid sequence diagram showing the main user workflow}
```

**Justificacion:** Los diagramas Mermaid son renderizables en markdown (GitHub, VS Code, Notion) y Claude Code puede generarlos nativamente. Un diagrama ER evita ambiguedades que las tablas de texto pueden dejar.

---

### 2.4 Crear Blueprints de Ejemplo

**Archivo:** `examples/saas-task-manager-blueprint.md`
- Ejemplo completo de un SaaS de gestion de tareas (tipo Trello simplificado)
- Las 16 secciones completas con datos reales
- Sirve como referencia de calidad para el Architect y para el usuario

**Archivo:** `examples/marketing-landing-blueprint.md`
- Ejemplo de landing page para un producto SaaS
- Muestra como se ve un blueprint para un proyecto mas simple

**Agregar al README:**
```markdown
## Examples
Check the `examples/` directory for complete blueprint examples:
- [SaaS Task Manager](examples/saas-task-manager-blueprint.md)
- [Marketing Landing Page](examples/marketing-landing-blueprint.md)
```

---

## Prioridad 3 — Impacto Medio, Esfuerzo Alto

Mejoras estructurales que requieren mas diseno y trabajo.

---

### 3.1 Sistema de Validacion de Blueprint

**Archivo nuevo:** `knowledge/blueprint-checklist.md`

Checklist que IAAF verifica antes de entregar el blueprint:

```markdown
## Blueprint Completeness Checklist

### Required for ALL blueprints:
- [ ] All 16 sections present and filled
- [ ] Build order has numbered steps with specific deliverables
- [ ] Every technology in Tech Stack section has a rationale
- [ ] Directory structure matches the chosen tech stack
- [ ] Environment variables listed for every external service
- [ ] CLAUDE.md section is < 120 lines and has concrete values
- [ ] Rules section has at least 5 non-negotiable rules

### Required for projects WITH a frontend:
- [ ] Design system has hex color values (not "blue" or "primary")
- [ ] Typography has specific font families and sizes
- [ ] Component hierarchy shown for at least 2 key pages
- [ ] Responsive breakpoints defined

### Required for projects WITH a database:
- [ ] All entities documented with field types
- [ ] Relationships explicitly stated
- [ ] SQL schema or ORM schema included
- [ ] Migration strategy defined

### Required for projects WITH auth:
- [ ] Auth flow described step by step
- [ ] Protected vs public routes listed
- [ ] Roles and permissions defined (if applicable)
- [ ] Session management approach stated
```

**Modificacion a CLAUDE.md:**
Agregar en Phase 4: "Before saving the blueprint, read `knowledge/blueprint-checklist.md` and verify every applicable item is covered."

---

### 3.2 Soporte Mejorado para Proyectos Hibridos

**Archivo nuevo:** `knowledge/hybrid-patterns.md`

```markdown
# Hybrid Project Patterns

## Common Combinations
| Primary | Secondary | Example |
|---------|-----------|---------|
| SaaS Web App | Marketing Site | SaaS with public landing page |
| API Backend | Internal Tool | API with admin dashboard |
| Content Platform | Marketing Site | Blog with product landing page |
| Mobile App | API Backend | Mobile app with custom backend |

## How to Handle Hybrids

### In the Blueprint:
1. Identify primary and secondary archetypes
2. Use primary archetype for the base architecture
3. Merge directory structures (use route groups in Next.js)
4. Combine build orders: build shared infra first,
   then primary feature, then secondary

### Build Order Merge Strategy:
1. Scaffolding (shared)
2. Database & auth (shared)
3. Primary archetype core features (steps 4-8)
4. Secondary archetype features (steps 9-11)
5. Polish & deploy (shared)
```

---

### 3.3 Separar README por Idioma

**Estado actual:** Un README de 496 lineas con contenido duplicado en ingles y espanol.

**Propuesta:**
- `README.md` — Ingles (lineas 1-257 del actual, ~260 lineas)
- `README.es.md` — Espanol (lineas 261-496 del actual, ~240 lineas)
- Agregar al inicio de cada uno: link al otro idioma
- Mas facil de mantener, mas facil de leer, reduce riesgo de desincronizacion

---

### 3.4 Building Blocks Avanzados (4)

**Archivo:** `knowledge/building-blocks/observability-patterns.md`
- Error tracking: Sentry vs LogRocket vs Bugsnag
- Logging: structured logging, log levels, Pino/Winston
- Monitoring: Vercel Analytics, Grafana, Datadog
- Alerting: PagerDuty, OpsGenie, Slack webhooks
- Health checks y status pages

**Archivo:** `knowledge/building-blocks/i18n-patterns.md`
- Decision matrix: next-intl vs react-i18next vs Paraglide
- Locale routing: subpath (/en, /es) vs subdomain vs domain
- Content translation: CMS-based vs file-based vs service (Crowdin)
- Date/number/currency formatting
- RTL support considerations

**Archivo:** `knowledge/building-blocks/performance-patterns.md`
- Core Web Vitals: LCP, FID, CLS — que son y como optimizar
- Image optimization: next/image, formats, lazy loading, CDN
- Bundle optimization: code splitting, dynamic imports, tree shaking
- Caching strategy: browser cache, CDN cache, ISR, SWR
- Database query optimization: indexes, connection pooling, caching

**Archivo:** `knowledge/building-blocks/ci-cd-patterns.md`
- GitHub Actions workflows para cada tipo de proyecto
- Branch strategies: trunk-based vs Git Flow vs GitHub Flow
- Preview deploys: Vercel, Netlify, Railway
- Automated testing in CI: lint → type check → unit → E2E
- Release management: semantic versioning, changelogs, tags

---

## Resumen del Plan de Mejora

| # | Mejora | Prioridad | Esfuerzo | Impacto | Archivos |
|---|--------|-----------|----------|---------|----------|
| 1.1 | 4 building blocks criticos | P1 | Bajo | Alto | 4 nuevos |
| 1.2 | Fix inconsistencias idioma | P1 | Bajo | Medio | 2 ediciones |
| 1.3 | Preguntas de contexto | P1 | Bajo | Alto | 1 edicion |
| 2.1 | 4 nuevos arquetipos | P2 | Medio | Alto | 4 nuevos |
| 2.2 | Stacks alternativos (Python, Go, AWS) | P2 | Medio | Alto | 2 ediciones |
| 2.3 | Seccion de diagramas | P2 | Medio | Medio | 1 edicion |
| 2.4 | Blueprints de ejemplo | P2 | Medio | Alto | 2 nuevos + 1 edicion |
| 3.1 | Validacion de blueprint | P3 | Alto | Medio | 1 nuevo + 1 edicion |
| 3.2 | Soporte hibridos | P3 | Alto | Medio | 1 nuevo + 2 ediciones |
| 3.3 | Separar README | P3 | Bajo | Bajo | 2 ediciones |
| 3.4 | 4 building blocks avanzados | P3 | Alto | Medio | 4 nuevos |

**Total de cambios propuestos:** 16 archivos nuevos + 11 ediciones a archivos existentes

---

## Roadmap Sugerido

**Fase A (Semana 1):** Prioridad 1 completa
- Crear 4 building blocks criticos
- Corregir inconsistencias de idioma
- Agregar 3 preguntas a Phase 1

**Fase B (Semanas 2-3):** Prioridad 2 completa
- Crear 4 nuevos arquetipos
- Agregar stacks alternativos
- Agregar diagramas al blueprint template
- Crear 2 blueprints de ejemplo

**Fase C (Semanas 4-6):** Prioridad 3 selectiva
- Crear checklist de validacion
- Mejorar soporte de hibridos
- Crear building blocks avanzados segun demanda

---

> Este informe cubre la totalidad de los 25 archivos del proyecto.
> Cada evaluacion se basa en la lectura completa de cada archivo, no en muestreo.
