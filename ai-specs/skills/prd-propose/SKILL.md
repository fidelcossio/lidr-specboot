---
name: prd-propose
description: Convierte los puntos pendientes del último PRD en borrador de Cashflow (los que fue dejando prd-add) en requerimientos completos con el formato y el nivel de detalle del PRD de referencia (docs/PRD_2026-10-20.md) — estado actual con archivo:línea, propuesta, criterios de aceptación, impacto técnico, preguntas abiertas, fases, orden priorizado, decisiones y hallazgos — tras una investigación breve del código en solo lectura.
version: 1.0.0-cashflow
---
# prd-propose

Úsala cuando el usuario pida "propón el PRD", "termina el PRD", "/prd-propose", o quiera revisar los puntos acumulados con `prd-add` antes de priorizarlos. Lee el código pero **no lo cambia**: el resultado es solo el documento.

Ciclo de vida y regla para encontrar el borrador: los de la skill `prd-add` (`ai-specs/skills/prd-add/SKILL.md`). Si no hay ningún PRD en borrador, o el borrador no tiene puntos pendientes, dilo y para.

## Antes de empezar

1. **Lee el PRD de referencia completo**: el más reciente cuyo Estado no empiece por `Borrador` (hoy `docs/PRD_2026-10-20.md`). Es la plantilla de tono, estructura y detalle. Imítalo; no inventes secciones.
2. **Lee el borrador**: sección 2 (puntos y cuáles están `pendiente`) y el Anexo A (texto literal; es la fuente, no la sección 2).
3. **Lee el contexto del proyecto** que condiciona las propuestas: el `CLAUDE.md` del workspace (modelo de datos, orden de scripts, `APP_VERSION`, motores duplicados), `docs/base-standards.md` y lo que esté en curso (`openspec/changes/` sin archivar, `docs/historias/` del PRD anterior). Un requerimiento nuevo no se planifica encima de un change abierto que toca los mismos archivos sin decirlo.
4. **Numeración.** Los requerimientos usan ids globales: continúa desde el `R` más alto de todos los `docs/PRD_*.md` (si el anterior acaba en R38, el primero nuevo es R39). Si `prd-propose` ya corrió sobre este borrador, conserva los ids existentes y añade a continuación.

## Investigación (solo lectura)

Por cada punto pendiente:

- Localiza **dónde vive hoy** el comportamiento: archivos, funciones, líneas. Cita `archivo:línea` del estado actual del repo.
- Distingue lo **verificado** (leído en el código y comprobado) de lo **según el análisis** (inferido) y de lo **no probado** en navegador o dispositivo. El PRD lo dice explícitamente donde importa.
- Busca lo que el punto **no dice pero arrastra**: el mismo patrón repetido en otras secciones, los dos motores de reglas (`page-sync-rules.jsx` y `functions/syncRules.js`), el parser duplicado, migraciones de esquema, `APP_VERSION`, `firestore.rules`, documentación que quedaría desactualizada.
- Si al investigar encuentras un bug que el usuario no reportó, va a **Hallazgos transversales** (sección 6) y, si merece trabajo propio, como requerimiento con Origen "hallazgo al analizar Mn".
- Con muchos puntos (más de cinco, aprox.) puedes repartir la lectura en subagentes de solo lectura, uno por punto; verifica tú mismo las afirmaciones delicadas antes de escribirlas y dilo en "Cómo se hizo".

## Qué escribir

Reescribe el borrador completo con esta estructura, la misma del PRD de referencia. Las secciones 1, 3, 4, 6 y 7 se **regeneran** con todos los requerimientos del documento, no solo los nuevos.

### Cabecera
Tabla `Fecha · Estado · Producto · Fuente` (más `PRD anterior` si el esqueleto la trae). Estado → `Borrador para revisión: tiene preguntas abiertas por requerimiento` (o `Borrador para revisión` si no quedan preguntas).

### 1. Contexto y objetivo
- Un párrafo: qué reportó el usuario y qué capas toca.
- "Este documento:" con los cuatro fines numerados (convertir en requerimientos verificables, registrar lo que existe y lo que falta, dejar las preguntas junto a cada requerimiento, proponer un orden).
- **Cómo se hizo**: cómo se investigó y la lista de afirmaciones delicadas verificadas en el código. Cierra con que las referencias `archivo:línea` son del repo a la fecha y pueden desplazarse.

### 2. De los puntos a los requerimientos
Tabla `Punto | Lo que pidió el usuario | ¿Uno o varios? | Requerimientos`. "¿Uno o varios?" explica el corte ("Dos: el bug y la causa que lo agrava"). Después, en viñetas, los puntos **ya resueltos parcialmente** en el código y cuánto trabajo real queda.

### 3. Orden propuesto
- **Criterios**, los tres del PRD de referencia: dependencias, afinidad de contexto (mismos archivos y conceptos), impacto y valor (primero lo que protege datos y se usa a diario; lo barato de alto valor antes que lo caro incierto).
- **Fases**: tabla `Fase | Por qué en este lugar`, con el rango de requerimientos en el nombre ("**1. Consistencia de listas** (R39–R41)"). Con un solo requerimiento, una sola fase.
- **Alternativas válidas**: qué bloques podrían moverse si el usuario prioriza distinto.
- **Lista ordenada**: `# | Requerimiento | Origen | Tamaño | Depende de`. Las dependencias pueden apuntar a requerimientos de PRD anteriores; escríbelas con su id (`R15 (PRD 2026-10-20)`).

### 4. Decisiones que más condicionan el plan
Lista numerada con solo las preguntas que cambian **alcance u orden**, cada una con el requerimiento entre paréntesis. El resto se queda en cada requerimiento.

### 5. Requerimientos
Agrupados por fase (`### Fase N — Nombre`), cada uno con este bloque exacto:

```markdown
#### Rnn — Título corto en forma de resultado
**Origen:** M1 · **Tipo:** bug | mejora | mejora técnica | decisión de datos · **Tamaño:** 0 | S | M | L (o rango S–M) · **Depende de:** — | Rxx (motivo breve)

**Problema.** / **Estado actual** (verificado | según el análisis)
- Hechos con `archivo:línea`. Para un bug: **Causa probable** con el razonamiento y, si ayuda, un ejemplo paso a paso.

**Propuesta**
- Qué cambia, con los nombres de funciones, campos y componentes concretos. Si hay modelo nuevo, sus campos. Si se descartó una alternativa evidente, por qué.

**Criterios de aceptación**
- Verificables y concretos, con cifras o casos ("Dado X, entonces Y"). Incluye qué NO debe cambiar.

**Impacto técnico**
- Archivos. Esquema (`_schemaVersion`) y migración, o "Sin migración". Si toca `functions/` o `firestore.rules` (despliegue aparte). `core.js` → verificar en un puerto nuevo. Documentación a actualizar. Fuera de alcance.

**Preguntas abiertas**
1. Solo lo que el repo no puede responder. Con opciones (a)/(b)/(c) cuando las hay y una recomendada.
```

Omite **Impacto técnico** o **Preguntas abiertas** solo si de verdad no aplican (como hace el PRD de referencia en algunos S). Separa las fases con `---`.

**Tamaños**: 0 = solo decisión o documentación; S = un archivo o un ajuste acotado, sin modelo nuevo; M = varios archivos o un campo nuevo sin migración; L = migración de esquema, cruza cliente y servidor, o rediseña un flujo.

### 6. Hallazgos transversales
Tabla `# | Hallazgo | Verificado (Sí / Según el análisis / No / Documentado en …) | Se atiende en`. Si no hay ninguno, dilo en una línea.

### 7. Notas comunes de implementación
Viñetas con lo que aplica a varios requerimientos: `APP_VERSION` actual en `sw.js` y una subida por cambio; puerto nuevo para `core.js`; los dos motores de reglas; parser duplicado; esquema; qué requerimientos merecen `/opsx:propose` y cuáles van directo; commits directo a `main`. Lee los valores actuales (no copies los del PRD anterior).

### Anexo A
Se conserva intacto: es la entrada literal. No borres ni edites lo que escribió el usuario.

## Al terminar

1. Repasa el documento contra el PRD de referencia: mismas secciones, mismo orden, mismo tipo de bloque por requerimiento, nada `pendiente` en la sección 2.
2. Comprueba cada `archivo:línea` citado (que la línea exista y diga lo que el PRD dice).
3. Informa: archivo, requerimientos creados (id y título), fases, y las decisiones de la sección 4 que el usuario debe responder. Ofrece el siguiente paso del flujo: responder preguntas → enriquecer con `enrich-us` → `/opsx:propose`.

No se commitea ni se despliega nada. Si el usuario aprueba el PRD, cambia su Estado a `Aprobado · en ejecución`; desde ahí, `prd-add` abre un PRD nuevo.
