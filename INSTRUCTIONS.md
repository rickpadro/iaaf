# IAAF — Guia de Uso

> Intelligence Artificial Architect Foreman
> Version 1.1 | SinapSYS Ecosistemas

---

## Requisitos

- [Claude Code](https://claude.ai/claude-code) instalado
- Git instalado
- Suscripcion Claude (API o Claude Max)

---

## Instalacion

IAAF no se instala. Se clona como subcarpeta dentro de tu proyecto.

```bash
cd C:\ruta\a\tu\proyecto
git clone https://github.com/rickpadro/iaaf.git .iaaf
```

Eso crea `.iaaf/` con toda la base de conocimiento. Para actualizar en el futuro:

```bash
cd .iaaf
git pull
cd ..
```

---

## 4 Modos de Operacion

| Modo | Para que | Que genera |
|------|----------|-----------|
| **GREENFIELD** | Construir un proyecto nuevo desde cero | Blueprint completo de 16 secciones |
| **IMPROVE** | Mejorar un proyecto existente (seguridad, performance, deuda tecnica) | Plan de mejoras priorizado |
| **EXTEND** | Agregar una feature nueva a un proyecto existente | Blueprint de extension |
| **FIX** | Diagnosticar y corregir un bug puntual | Diagnostico + fix directo |

---

## Modo GREENFIELD — Proyecto nuevo

### Cuando usarlo

- No tienes codigo todavia
- Quieres que IAAF diseñe la arquitectura completa
- Puede leer un DDS/PRD/spec si ya tienes uno

### Pasos

```bash
# 1. Crear la carpeta del proyecto
mkdir C:\xampp\htdocs\mi-nuevo-proyecto
cd C:\xampp\htdocs\mi-nuevo-proyecto

# 2. Clonar IAAF
git clone https://github.com/rickpadro/iaaf.git .iaaf

# 3. Activar modo GREENFIELD
copy .iaaf\CLAUDE.init.md CLAUDE.md

# 4. Abrir Claude Code
claude
```

### Que pasa dentro

1. Claude se presenta como The Architect
2. Te pregunta si tienes un documento (DDS, PRD, spec)
   - Si tienes: le das el path o pegas el contenido. Salta la mayoria de preguntas.
   - Si no tienes: te hace 5-8 preguntas sobre tu proyecto.
3. Te presenta la arquitectura propuesta (stack, modulos, enfoque)
4. Tu confirmas o ajustas
5. Genera el blueprint en `.iaaf/output/<nombre>-blueprint.md`
6. Reemplaza CLAUDE.md con instrucciones de construccion (incluye The Foreman)
7. Audita el blueprint contra tu documento fuente (si lo proporcionaste)

### Despues de generar

```bash
# Cerrar Claude Code (exit)
# Reabrirlo — ahora lee el CLAUDE.md de build
claude
# Decirle "Paso 1" y empieza a construir
```

### El blueprint se guarda en

```
.iaaf/output/<nombre-proyecto>-blueprint.md
```

---

## Modo IMPROVE — Mejorar proyecto existente

### Cuando usarlo

- El proyecto funciona pero tiene problemas (lento, inseguro, desordenado)
- Quieres reducir deuda tecnica
- Quieres un diagnostico profesional de tu codigo

### Pasos

```bash
cd C:\ruta\a\mi-proyecto-existente

# 1. Clonar IAAF (solo la primera vez)
git clone https://github.com/rickpadro/iaaf.git .iaaf

# 2. Resguardar tu CLAUDE.md existente
rename CLAUDE.md CLAUDE.project.md

# 3. Activar modo IMPROVE
copy .iaaf\CLAUDE.improve.md CLAUDE.md

# 4. Abrir Claude Code
claude
```

### Que pasa dentro

1. Claude se presenta en modo IMPROVE
2. Te pregunta que quieres mejorar (o si quieres un analisis completo)
3. **SCAN**: Lee 15-20 archivos clave de tu proyecto (configs, modelos, rutas, estructura)
4. **DIAGNOSE**: Identifica issues de seguridad, performance, calidad de codigo, testing, dependencias
5. Te presenta una tabla priorizada de hallazgos
6. Tu eliges cuales abordar
7. **GENERATE**: Genera un plan con pasos concretos (que archivo cambiar, como verificar)
8. **AUDIT**: Verifica que el plan referencia archivos reales y es coherente
9. Reemplaza CLAUDE.md con instrucciones de build que incluyen tus instrucciones originales

### Despues de terminar

```bash
# Cerrar Claude Code
# Reabrir para ejecutar las mejoras
claude
# Decirle "Paso 1" y empieza a mejorar

# Cuando termine todo, restaurar tu CLAUDE.md original
rename CLAUDE.project.md CLAUDE.md
```

### El plan se guarda en

```
.iaaf/output/<nombre-proyecto>-improvements.md
```

---

## Modo EXTEND — Agregar feature

### Cuando usarlo

- El proyecto funciona bien
- Quieres agregar algo nuevo (modulo, integracion, pagina)
- Quieres que la feature respete la arquitectura existente

### Pasos

```bash
cd C:\ruta\a\mi-proyecto-existente

# 1. Clonar IAAF (solo la primera vez)
git clone https://github.com/rickpadro/iaaf.git .iaaf

# 2. Resguardar tu CLAUDE.md existente
rename CLAUDE.md CLAUDE.project.md

# 3. Activar modo EXTEND
copy .iaaf\CLAUDE.extend.md CLAUDE.md

# 4. Abrir Claude Code
claude
```

### Que pasa dentro

1. Claude se presenta en modo EXTEND
2. Te pregunta que feature quieres agregar (o si tienes un spec)
3. **SCAN**: Lee el proyecto para entender stack, patrones, convenciones
4. **DESIGN**: Diseña la feature siguiendo tus patrones existentes
5. Te presenta el diseño — que tablas, que rutas, que archivos
6. Tu confirmas o ajustas
7. **GENERATE**: Genera el plan de extension con pasos y verificaciones
8. **AUDIT**: Verifica que los archivos referenciados existen y el plan es coherente
9. Reemplaza CLAUDE.md con instrucciones de build

### Despues de terminar

```bash
# Cerrar Claude Code
# Reabrir para construir la feature
claude

# Cuando termine, restaurar tu CLAUDE.md original
rename CLAUDE.project.md CLAUDE.md
```

### El plan se guarda en

```
.iaaf/output/<nombre-proyecto>-extension-<feature>.md
```

---

## Modo FIX — Corregir bug

### Cuando usarlo

- Algo esta roto y necesitas arreglarlo
- Sabes que falla pero no sabes por que
- Quieres un diagnostico rapido

### Pasos

```bash
cd C:\ruta\a\mi-proyecto-existente

# 1. Clonar IAAF (solo la primera vez)
git clone https://github.com/rickpadro/iaaf.git .iaaf

# 2. Resguardar tu CLAUDE.md existente
rename CLAUDE.md CLAUDE.project.md

# 3. Activar modo FIX
copy .iaaf\CLAUDE.fix.md CLAUDE.md

# 4. Abrir Claude Code
claude
```

### Que pasa dentro

1. Claude se presenta en modo FIX
2. Le describes el bug (error, comportamiento, que esperabas)
3. Lee 3-5 archivos relevantes (no el proyecto entero)
4. Te presenta el diagnostico (causa raiz, archivos afectados, riesgo)
5. Te propone el fix concreto
6. Si confirmas, lo aplica y verifica

### Despues de terminar

```bash
# Restaurar tu CLAUDE.md original
rename CLAUDE.project.md CLAUDE.md
```

### Diferencia con los otros modos

FIX no genera blueprint ni plan. Es diagnostico + correccion directa. Si el problema es demasiado grande (afecta 10+ archivos), Claude te recomendara cambiar a modo IMPROVE.

---

## Referencia Rapida

### Proyecto nuevo (no hay CLAUDE.md)

```bash
git clone https://github.com/rickpadro/iaaf.git .iaaf
copy .iaaf\CLAUDE.init.md CLAUDE.md
claude
```

### Proyecto existente (ya hay CLAUDE.md)

```bash
git clone https://github.com/rickpadro/iaaf.git .iaaf
rename CLAUDE.md CLAUDE.project.md
copy .iaaf\CLAUDE.{improve|extend|fix}.md CLAUDE.md
claude
# Al terminar:
rename CLAUDE.project.md CLAUDE.md
```

### Actualizar IAAF

```bash
cd .iaaf
git pull
cd ..
```

---

## Preguntas Frecuentes

**IAAF lee todos mis archivos?**
No. En modo GREENFIELD no lee nada del proyecto (no existe). En IMPROVE y EXTEND lee 15-20 archivos clave. En FIX lee 3-5. Usa un indice (`knowledge/index.md`) para decidir que building blocks cargar, no carga todos.

**Puedo usar IAAF con cualquier lenguaje?**
Hoy tiene mejor soporte para JavaScript/TypeScript (Next.js, React, Node), PHP (Laravel), Python (FastAPI) y Go. Para otros stacks funciona pero con menos guias especificas.

**Que pasa si IAAF propone algo que no quiero?**
En Phase 3 (Architecture) siempre te presenta la propuesta para que confirmes. Puedes ajustar cualquier cosa — stack, estructura, patrones, orden de build.

**Se pierde mi CLAUDE.md original?**
No. Lo renombras a `CLAUDE.project.md` antes de activar IAAF. Claude lo lee automaticamente. Al terminar, lo restauras con `rename`.

**Cuanto consume en tokens?**
Depende del modo. GREENFIELD lee ~12,000 tokens de knowledge base. IMPROVE y EXTEND leen eso mas 15-20 archivos de tu proyecto. FIX lee minimo (~3,000 tokens de knowledge + 3-5 archivos).

**Puedo usar varios modos en el mismo proyecto?**
Si. Primero GREENFIELD para crear, despues EXTEND para agregar features, IMPROVE para optimizar, FIX para bugs. Solo cambia que CLAUDE.md copias cada vez.

---

## Estructura de IAAF

```
.iaaf/
├── CLAUDE.init.md          Modo GREENFIELD
├── CLAUDE.improve.md       Modo IMPROVE
├── CLAUDE.extend.md        Modo EXTEND
├── CLAUDE.fix.md           Modo FIX
├── knowledge/
│   ├── index.md            Indice (que leer y cuando)
│   ├── archetypes/         10 tipos de proyecto
│   ├── building-blocks/    12 guias de decision
│   ├── audit-protocol.md   Protocolo de auditoria
│   ├── blueprint-checklist.md  QA del blueprint
│   ├── hybrid-patterns.md  Proyectos hibridos
│   └── stack-compatibility.md  Combinaciones de stack
├── questions/              Preguntas por fase
├── templates/              Templates de output
├── examples/               Blueprints de ejemplo
└── output/                 Donde se guardan los blueprints generados
```
