<p align="center">
  <h1 align="center">IAAF</h1>
  <p align="center">
    <strong>Intelligence Artificial Architect Foreman — Un meta-agente de Claude Code que diseña blueprints completos de software.</strong>
  </p>
  <p align="center">
    Describe lo que quieres construir. Obtén un blueprint completo. Deja que Claude Code lo construya por ti.
  </p>
  <p align="center">
    <a href="README.md">English</a>
  </p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-Agent-blueviolet?style=for-the-badge" alt="Claude Code Agent" />
  <img src="https://img.shields.io/badge/Blueprints-Markdown-blue?style=for-the-badge" alt="Markdown Blueprints" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="MIT License" />
  <img src="https://img.shields.io/badge/by-SinapSYS-black?style=for-the-badge" alt="by SinapSYS Ecosistemas" />
</p>

---

## Que es IAAF?

Imagina que quieres construir una casa. Antes de que alguien agarre un martillo, necesitas un **plano** — un plan detallado que muestre cada cuarto, cada pared, cada tuberia y cada cable. Sin el, los constructores no sabrian que hacer.

**IAAF hace esto para software.**

Abres este proyecto en [Claude Code](https://claude.com/claude-code), le dices que quieres construir (un sitio web, una app, un SaaS, una API — lo que sea), y IAAF:

1. **Te hace preguntas inteligentes** sobre tu idea
2. **Diseña todo el sistema** — base de datos, API, frontend, autenticacion, pagos, todo
3. **Genera un archivo blueprint** (`.md`) tan detallado que otra instancia de Claude Code puede construir todo — paso a paso, sin preguntarte nada

```
Tu: "Quiero construir una app de gestion de proyectos como Trello"
         ↓
IAAF: hace preguntas, diseña todo
         ↓
The Foreman: inyecta reglas de eficiencia en el blueprint
         ↓
Blueprint.md: plan completo con tech stack, esquema de base de datos,
              rutas API, jerarquia de componentes, orden de construccion...
         ↓
Claude Code: lee el blueprint → construye toda la app (menos tokens)
```

**Tu describes la idea. IAAF crea el plano. The Foreman optimiza la ejecucion. Claude Code lo construye.**

### The Foreman

Cada blueprint incluye **The Foreman** — reglas de comportamiento integradas en el CLAUDE.md generado que hacen que Claude Code trabaje de forma mas eficiente:

- **Sin output desperdiciado** — elimina aperturas aduladoras, cierres vacios y sugerencias no solicitadas
- **Leer antes de escribir** — lee todos los archivos relevantes antes de tocar codigo
- **Testear y avanzar** — cuando los tests pasan, avanza sin refactorizar codigo que ya funciona
- **Consolidacion inteligente** — agrupa operaciones simples, mantiene tareas complejas de forma incremental

Resultado: **reduccion significativa en costos de API** durante la construccion, sin sacrificar calidad. El ahorro real depende de la complejidad del proyecto y la longitud de la conversacion.

---

## Como Funciona

### Fase 1: Descubrimiento
> "Que estas construyendo? Para quien es? Que tan grande debe ser?"

IAAF hace 2-3 preguntas para entender tu idea. Clasifica tu proyecto en uno de **10 arquetipos**.

### Fase 2: Profundizacion
> "Necesitas cuentas de usuario? Pagos? Funciones en tiempo real?"

Preguntas especificas para TU tipo de proyecto. Investiga mejores practicas usando `/deep-research`.

### Fase 3: Arquitectura
> "Esto es lo que yo construiria: Next.js + Supabase + Clerk + Stripe en Vercel..."

Presenta el tech stack completo con razones para cada decision. Tu confirmas o ajustas.

### Fase 4: Generar
Produce el blueprint final — un solo archivo `.md` con **16 secciones** cubriendo todo lo que Claude Code necesita.

---

## Tipos de Proyecto Soportados

| Tipo | Ejemplos | Stack por Defecto |
|------|----------|------------------|
| **SaaS / Web App** | Trello, Notion, Slack | Next.js + Supabase + Clerk + Stripe + Vercel |
| **Sitio de Marketing** | Landing pages, portafolios | Astro + Tailwind + Sanity + Vercel |
| **App Movil** | Apps iOS/Android | Expo (React Native) + Supabase + NativeWind |
| **API / Backend** | REST APIs, microservicios | Hono + Drizzle + PostgreSQL + Railway |
| **Herramienta Interna** | Paneles admin, dashboards | Next.js + shadcn/ui + Prisma + Recharts |
| **Plataforma de Contenido** | Blogs, docs, CMS | Next.js + Sanity + Algolia + Vercel |
| **E-Commerce** | Tiendas online | Next.js + Medusa.js + Stripe + Cloudinary |
| **App de IA / ML** | Chat AI, RAG, generacion | Next.js + Vercel AI SDK + Pinecone |
| **Herramienta CLI** | Terminal utilities | Node.js + Commander.js + Inquirer |
| **App Colaborativa** | Edicion en tiempo real | Next.js + Liveblocks + Yjs + Tiptap |

---

## Inicio Rapido

### Prerequisitos

- [Claude Code](https://claude.com/claude-code) instalado
- Una suscripcion a la API de Claude

### Instalacion

```bash
git clone https://github.com/rickpadro/iaaf.git
cd iaaf
claude
```

### Uso

**Paso 1:** Dile a IAAF que quieres construir.
**Paso 2:** Responde las preguntas (2-3 a la vez).
**Paso 3:** Revisa la arquitectura propuesta. Confirma o ajusta.
**Paso 4:** IAAF genera tu blueprint en `output/<nombre-proyecto>-blueprint.md`.
**Paso 5:** Copia el blueprint como CLAUDE.md en tu nuevo proyecto y abrelo en Claude Code.

### Modo Rapido

```
Tu: Hazme un SaaS para reservaciones de restaurante. Solo construyelo.
```

Solo 3 preguntas esenciales + valores por defecto inteligentes.

---

## Ejemplos

Revisa la carpeta `examples/` para ver blueprints completos de ejemplo:
- [SaaS Task Manager](examples/saas-task-manager-blueprint.md)
- [Marketing Landing Page](examples/marketing-landing-blueprint.md)

---

## Contribuir

- **Agregar nuevos arquetipos** — en `knowledge/archetypes/`
- **Mejorar building blocks** — mejores matrices de decision
- **Mejorar el template del blueprint** — mas secciones, mejor estructura
- **Traducir** — mas idiomas

---

## Licencia

MIT. Ver [LICENSE](LICENSE).

---

## Construido por SinapSYS Ecosistemas
