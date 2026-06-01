# Ejemplo: NO-GO por mal timing

Caso típico del uso real de `validador`. La idea es técnicamente buena, pero el filtro la detiene por timing — hay trabajo abierto que se ampliaría sin necesidad.

---

## Input (en una sesión activa de Claude Code)

> "Vamos a añadir un endpoint en Exterior OS que devuelva el historial completo de cambios de un expediente. Sería útil para la auditoría y para que el frontend muestre una timeline."

Usuario invoca `/validador` antes de que Claude empiece a planear nada.

---

## Output (lo que devuelve la skill)

## 1. Qué entendí

Quieres exponer un endpoint en Exterior OS que devuelva el historial completo de cambios de un expediente. Asumo que esto implica también modelar el storage de los cambios — un endpoint sin tabla detrás no tiene de dónde leer. No mencionaste el storage; voy a tratarlo como parte de la propuesta porque no se puede separar.

## 2. Opinión franca

La idea es buena para auditoría regulatoria y para construir una timeline frontend. Pero estás resolviendo el caso fácil (lectura) antes de tener resuelto el difícil (escritura estructurada de los cambios). Mal orden. Si construyes el endpoint primero, vas a terminar volviendo a tocarlo cuando el storage cambie.

## 3. Mejoras posibles

- Antes del endpoint, define el schema de `expediente_cambios`: ¿qué se guarda exactamente, quién, cuándo, qué transición? Esto debería ser el primer paso, no el segundo.
- Decide si esto es **append-only event log** (todos los cambios, con causa) o **snapshots** (estado en momento X). Afecta todo lo demás. Si es event log, alinéalo con el patrón append-only que ya usas en `AMIGOS-GROUP-ECOSISTEMA/BITACORA.md`.

## 4. Viabilidad técnica

Con tu stack (Next.js 16 + Supabase + Drizzle) es directo:
- 1 migración para `expediente_cambios` con FK a `expediente`
- 1 trigger en `expediente` (o wrapper en server actions) para escribir los cambios
- 1 server action / API route para leer

Tiempo realista: 4-6 horas si lo haces solo, incluyendo tests. No requiere dependencias nuevas, no requiere API externa.

## 5. Qué se rompe

- La tabla `expediente` ya tiene **3 sitios** que la mutan (crear expediente vía puente, marcar evento administrativo, cerrar expediente). Los tres necesitan envolverse en una transacción que escriba también en `expediente_cambios`.
- Tests de `expediente.update()` van a fallar hasta que se actualicen. Estimar +2h adicionales solo en arreglar tests.
- El **contrato del puente** Amigos→Exterior no se ve afectado — `POST /api/bridge/expedientes` sigue igual.
- RLS: la nueva tabla necesita policies; si no las defines de entrada, expones cambios de un expediente a personas que no deberían verlo.

## 6. ¿Es prudente AHORA?

**NO.** Razones concretas:

- **Slice 15 sigue abierto** (Kanban). Este trabajo lo amplía ~30% en alcance.
- **Slice 17 (Portal Cliente) depende de Slice 15 cerrado.** Si extiendes 15, retrasa 17.
- Hoy en esta sesión ya construiste 2 cosas (`detector-humo`, este ejemplo). Tu energía técnica para una migración + tres refactors + tests está abajo.
- Este endpoint no desbloquea ningún flujo de cliente esta semana. Auditoría es importante pero no urgente.

Cuando sí: después de cerrar Slice 15, meterlo como **Slice 16 explícito** con scope claro. Probablemente en 7-10 días.

---

**VEREDICTO: NO-GO** — buena idea, mal momento. Reanudar después de cerrar Slice 15, no antes.

---

## Lo que pasaría sin `validador`

Sin el filtro, la respuesta default de Claude habría sido algo como: *"¡Perfecto! Vamos a empezar por crear la migración. Aquí está el archivo SQL..."* — y a las 3 horas el usuario tiene un branch a medias, Slice 15 sin cerrar, y dos refactors de tests pendientes sin que nadie haya pensado si era el momento.

Ese es el costo real que `validador` cobra antes del primer commit.
