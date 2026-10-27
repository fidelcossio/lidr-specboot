---
name: prd-add
description: Añade uno o varios requerimientos nuevos (bugs o mejoras que reporta el usuario) al último PRD en borrador de Cashflow, en docs/PRD_<fecha>.md, o abre un PRD borrador nuevo si no hay ninguno. Solo captura y clasifica; no investiga el código a fondo ni arma fases (eso es prd-propose).
version: 1.0.0-cashflow
---
# prd-add

Captura rápida de puntos para el próximo PRD. Úsala cuando el usuario diga "agrega al PRD…", "anota este requerimiento", "/prd-add …" o reporte un bug o una mejora que no se va a implementar ya.

Este skill **no** convierte el punto en un requerimiento detallado: deja el PRD en borrador, con el punto registrado y marcado como pendiente. La investigación de código, las fases y la priorización las hace `prd-propose`, que se corre cuando el usuario lo pida (normalmente tras acumular varios puntos).

El formato de referencia de un PRD terminado es el último PRD que **no** está en borrador (hoy `docs/PRD_2026-10-20.md`). Este skill solo escribe el esqueleto y la sección 2 de ese formato.

## Ciclo de vida de un PRD

El campo **Estado** de la tabla de cabecera manda. Valores, en orden:

| Estado | Quién lo pone | Significa |
|---|---|---|
| `Borrador · recogiendo puntos` | `prd-add` | Hay puntos sin convertir en requerimientos. Admite más puntos. |
| `Borrador para revisión` | `prd-propose` | Todos los puntos están convertidos, con fases y preguntas abiertas. Admite más puntos (vuelve al estado anterior). |
| `Aprobado · en ejecución` | el usuario (o el agente cuando el usuario lo apruebe) | Ya no admite puntos: lo nuevo va a un PRD nuevo. |
| `Cerrado` | el usuario | Todo entregado, aplazado o descartado. |

"PRD en borrador" = cualquier estado que empiece por `Borrador`.

## Pasos

1. **Localizar el borrador.** Lista `docs/PRD_*.md` y lee la tabla de cabecera de cada uno. Toma el más reciente (por la fecha del nombre) cuyo Estado empiece por `Borrador`.
   - Si no hay ninguno, crea `docs/PRD_<hoy>.md` (fecha local `YYYY-MM-DD`) con el [esqueleto](#esqueleto-de-un-prd-nuevo). Si ya existe un archivo con ese nombre y no es borrador, añade un sufijo `-2`.
   - Si hay más de un borrador, usa el más reciente y avisa al usuario de los otros.
2. **Separar los puntos.** Divide lo que pidió el usuario en puntos independientes: uno por problema o deseo, no por frase. No fusiones dos peticiones distintas ni partas una en requerimientos (eso es de `prd-propose`).
3. **Clasificar cada punto.**
   - `B<n>` si es un **bug** (algo que hoy funciona mal o contradice lo documentado), `M<n>` si es una **mejora**. Numera continuando los del mismo PRD (B1, B2…; M1, M2…), nunca reutilices un número.
   - Si dudas entre bug y mejora, elige uno y anótalo en la nota del punto; `prd-propose` lo puede reclasificar.
4. **Buscar solapes (rápido, sin investigar el código).** Busca palabras clave del punto en los demás `docs/PRD_*.md`, en `docs/historias/` y en `openspec/changes/` (incluido `archive/`). Si un requerimiento previo ya lo cubre o lo toca en parte, anótalo como "Relacionado" (por ejemplo "R07 del PRD 2026-10-20 ya ordenó la bandeja"). Si lo cubre **por completo**, pregunta al usuario antes de añadirlo.
5. **Escribir el punto** en dos sitios del borrador:
   - **Sección 2, tabla de puntos:** una fila `| **M1** | <resumen en una línea, en palabras del usuario> | pendiente | pendiente (prd-propose) |`. Las columnas "¿Uno o varios?" y "Requerimientos" quedan en `pendiente` hasta `prd-propose`.
   - **Anexo A, entrada literal:** un bloque por punto con la fecha, el texto del usuario **tal cual** (con sus faltas; es la fuente) y, si aplica, una línea `Relacionado:` y una línea `Nota:` con la duda de clasificación. No reescribas el texto del usuario.
6. **Actualizar la cabecera.**
   - Estado → `Borrador · recogiendo puntos` (también si estaba en `Borrador para revisión`: ahora hay puntos sin convertir).
   - Fuente → cuenta de puntos actualizada ("N bugs y M mejoras reportados por el usuario entre <fecha> y <fecha>").
7. **No toques** las secciones 1 y 3–7 si `prd-propose` ya las escribió: solo `prd-propose` las regenera.
8. **Informar** en dos o tres líneas: archivo, puntos añadidos con su id, solapes encontrados, cuántos puntos pendientes hay y que `prd-propose` los convierte cuando el usuario quiera.

No se commitea ni se despliega nada. Si el usuario lo pide, usa la skill `commit`.

## Esqueleto de un PRD nuevo

Copia la estructura numerada del PRD de referencia para que `prd-propose` solo tenga que rellenar. Los textos en `<…>` se sustituyen.

```markdown
# PRD — Cashflow · <YYYY-MM-DD>

| | |
|---|---|
| **Fecha** | <YYYY-MM-DD> |
| **Estado** | Borrador · recogiendo puntos |
| **Producto** | PWA "Cashflow" (`Personal cashflow/`), Cloud Function `syncSms`, parser de SMS |
| **Fuente** | <N> bugs y <M> mejoras reportados por el usuario el <YYYY-MM-DD> |
| **PRD anterior** | [PRD del <fecha>](PRD_<fecha>.md) (<su estado>) |

---

## 1. Contexto y objetivo

_Pendiente: lo escribe `prd-propose`._

---

## 2. De los puntos a los requerimientos

| Punto | Lo que pidió el usuario | ¿Uno o varios? | Requerimientos |
|---|---|---|---|

---

## 3. Orden propuesto

_Pendiente: lo escribe `prd-propose`._

---

## 4. Decisiones que más condicionan el plan

_Pendiente: lo escribe `prd-propose`._

---

## 5. Requerimientos

_Pendiente: lo escribe `prd-propose`._

---

## 6. Hallazgos transversales

_Pendiente: lo escribe `prd-propose`._

---

## 7. Notas comunes de implementación

_Pendiente: lo escribe `prd-propose`._

---

## Anexo A — Entrada literal

Lo que escribió el usuario, sin editar. Es la fuente de la sección 2.

### M1 · <YYYY-MM-DD>
> <texto literal>

Relacionado: <requerimiento previo, si hay>
```
