# Ejemplo: NO-GO por mal timing

Caso típico. La idea es técnicamente buena, pero el filtro la detiene por timing: hay trabajo abierto que se ampliaría sin necesidad.

---

## Input (en una sesión activa de Claude Code)

> "Vamos a añadir un endpoint que devuelva el historial completo de cambios de un pedido. Sería útil para resolver reclamaciones y para que la web muestre una línea de tiempo."

El usuario invoca `/validador` antes de que Claude empiece a planear nada.

---

## Output (lo que devuelve la skill)

## 1. Qué entendí

Quieres exponer un endpoint que devuelva el historial completo de cambios de un pedido de tu tienda. Asumo que esto implica también guardar esos cambios: un endpoint sin tabla detrás no tiene de dónde leer. No mencionaste dónde se guardan; lo trato como parte de la propuesta porque no se puede separar.

## 2. Opinión franca

La idea es buena para resolver reclamaciones y para construir una línea de tiempo en la web. Pero estás resolviendo el caso fácil (lectura) antes de tener resuelto el difícil (escritura estructurada de los cambios). Mal orden. Si construyes el endpoint primero, vas a volver a tocarlo cuando cambie la forma de guardar.

## 3. Mejoras posibles

- Antes del endpoint, define el schema de `pedido_cambios`: ¿qué se guarda exactamente, quién, cuándo, qué transición? Esto debería ser el primer paso, no el segundo.
- Decide si esto es un **append-only event log** (todos los cambios, con su causa) o **snapshots** (el estado en un momento dado). Afecta a todo lo demás.

## 4. Viabilidad técnica

Con tu stack (Next.js + Supabase) es directo:
- 1 migración para `pedido_cambios` con FK a `pedidos`
- 1 trigger en `pedidos` (o un wrapper en las server actions) para escribir los cambios
- 1 server action / API route para leer

Tiempo realista: 4-6 horas si lo haces solo, incluidos los tests. No requiere dependencias nuevas ni API externas.

## 5. Qué se rompe

- La tabla `pedidos` ya tiene **3 sitios** que la mutan (crear pedido, cambiar estado, cancelar). Los tres necesitan envolverse en una transacción que escriba también en `pedido_cambios`.
- Los tests de `pedidos.update()` van a fallar hasta que se actualicen. Calcula unas 2 horas más solo para arreglarlos.
- La API que usa la app móvil no se ve afectada: sigue igual.
- Permisos: la tabla nueva necesita sus reglas de acceso desde el principio. Si no las defines de entrada, expones los cambios de un pedido a personas que no deberían verlos.

## 6. ¿Es prudente AHORA?

**NO.** Razones concretas:

- **La integración de pagos sigue abierta.** Este trabajo la amplía ~30 % en alcance.
- **Los envíos dependen de cerrar pagos.** Si alargas pagos, retrasas envíos.
- Hoy en esta sesión ya construiste dos cosas. La energía técnica para una migración, tres refactors y sus tests está baja.
- Este endpoint no desbloquea ningún flujo de cliente esta semana. Es importante, pero no urgente.

Cuándo sí: después de cerrar pagos, como una fase explícita con alcance claro. Probablemente en 7-10 días.

---

**VEREDICTO: NO-GO** — buena idea, mal momento. Retomar después de cerrar pagos, no antes.

---

## Lo que pasaría sin `validador`

Sin el filtro, la respuesta por defecto de Claude habría sido algo como: *"¡Perfecto! Vamos a empezar por crear la migración. Aquí está el archivo SQL..."* — y a las 3 horas el usuario tiene una rama a medias, pagos sin cerrar y dos refactors de tests pendientes, sin que nadie se haya preguntado si era el momento.

Ese es el costo real que `validador` cobra antes del primer commit.
