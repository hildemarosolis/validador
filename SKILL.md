---
name: validador
description: Pasa cualquier idea, propuesta o intención de construcción que el usuario tenga en una sesión activa de Claude Code por un filtro estructurado de 6 chequeos antes de tocar una línea de código. Analiza qué entendió Claude, da opinión franca, propone mejoras concretas, evalúa viabilidad técnica con el stack actual del proyecto, detecta qué se rompe en lo ya construido, y termina con la pregunta clave — ¿es prudente hacerlo AHORA? — considerando deuda técnica, prioridades pendientes y energía disponible. Úsala SIEMPRE que el usuario diga "vamos a hacer", "se me ocurrió", "podríamos añadir", "quiero implementar", "voy a construir", "qué te parece si", "y si hacemos", o invoque /validador directamente. Está diseñada para builders con múltiples proyectos en producción que necesitan un freno técnico entre la idea y el commit. No genera código, no planea sprints — solo decide si la idea pasa al siguiente paso o se mata aquí.
---

# validador

Skill para que Claude Code pare antes de construir lo siguiente que se le ocurra al usuario, lo pase por 6 chequeos estructurados, y devuelva un veredicto de **GO / NO-GO / DEPENDE** con el razonamiento detrás. Pensada para operadores con varios sistemas en producción donde una idea mal-temporizada cuesta más que la idea mal-pensada.

## Por qué existe

Las sesiones de construcción con Claude Code tienen un sesgo brutal hacia decir que sí. Cuando el usuario dice "vamos a hacer X", el modelo por defecto entra en modo ejecutivo: planea, sugiere arquitectura, propone diff. Eso se siente productivo y arruina proyectos.

El verdadero costo de una idea no es construirla — es que esa idea bloquee, retrase o rompa todo lo que ya está en producción y sin terminar. El validador existe para cobrar ese costo **antes** del primer commit.

## Cuándo se activa

Slash command explícito: `/validador`.

También dispara cuando el usuario en una sesión activa de construcción dice frases tipo:
- "vamos a hacer / hagamos / construyamos..."
- "se me ocurrió que podríamos..."
- "y si añadimos / metemos / implementamos..."
- "qué te parece si..."
- "voy a..."
- "quiero meter..."

Si la idea ya está confirmada y el usuario solo pide implementación ("ahora codéalo", "ejecuta el plan"), **no** se activa. El validador es pre-decisión, no post-decisión.

## Cómo opera

Antes de responder, leer el contexto disponible del proyecto activo:
- Si hay `CLAUDE.md`, `README.md` o `BITACORA.md` en el cwd, léelos para entender estado actual.
- Si hay `package.json`, `pyproject.toml` o equivalente, mira el stack.
- Si hay carpetas que sugieren producción (`migrations/`, `prod/`, `deploy/`), nótalo.

No empezar el análisis a ciegas. La pregunta 5 ("¿qué se rompe?") es imposible de responder sin saber qué hay construido.

## Los 6 chequeos (en este orden)

### 1. Qué entendí

Reformular la idea del usuario en tus propias palabras, en 2-3 frases. No literal. Tradúcela a lo que estás escuchando realmente, incluyendo lo que el usuario no dijo pero está implícito ("entiendo que esto implica también X").

Si tu reformulación se desvía mucho de la idea original, **es señal de que la idea está mal especificada**. Dilo.

### 2. Opinión franca

Tu lectura honesta de la idea como concepto. ¿Es buena, mediocre, mala? ¿Resuelve un problema real o es interesante en abstracto? ¿Está alineada con la dirección del proyecto o es un desvío?

Prohibido el "depende". Prohibido el "tiene potencial pero...". Una postura clara. Si después de pensarlo de verdad la respuesta es matizada, escribe el matiz — pero como conclusión, no como evasión.

### 3. Mejoras posibles

Si hay una versión mejor de la idea, ponla aquí. Más simple, más barata, más enfocada, con menos riesgo. Una o dos alternativas concretas — no una lluvia de 8 opciones.

Si la idea ya está en su mejor forma, dilo: "esto es lo más limpio que se puede plantear de este problema". No inventar mejoras por cuota.

### 4. Viabilidad técnica

Con el stack que ya tiene el proyecto (lo que leíste del contexto), ¿qué tan factible es esto? Específico:
- ¿Las herramientas necesarias ya están instaladas / configuradas?
- ¿Hay dependencias nuevas? ¿Cuáles?
- ¿Qué piezas del sistema toca (DB, auth, API externa, frontend, infra)?
- ¿Hay un patrón existente para hacerlo, o hay que inventar?

Si necesitas algo que el usuario no mencionó (una API key nueva, un proveedor externo, una migración de DB), señálalo aquí. Es la diferencia entre "factible en 2 horas" y "factible en 2 semanas".

### 5. Qué se rompe

El chequeo más técnico. ¿Esta idea, implementada, afecta a algo de lo que ya está construido?

Áreas a considerar:
- **Contratos públicos** (APIs, schemas, tipos exportados): ¿algún consumidor existente rompe?
- **Base de datos**: ¿migraciones destructivas? ¿índices que se pierden? ¿datos a backfilear?
- **Auth / permisos**: ¿cambia quién puede hacer qué?
- **Performance**: ¿algún flujo crítico se vuelve más lento?
- **Tests existentes**: ¿hay tests que van a romper y por qué?
- **Despliegue**: ¿requiere downtime, env vars nuevas, rebuilds?

Si no rompe nada, dilo claro. Es el caso ideal, raro.

### 6. ¿Es prudente AHORA?

La pregunta más importante. Una idea técnicamente buena en mal momento es deuda futura.

Considerar:
- **Lo que el usuario ya tiene abierto**: ¿hay pendientes pesados que se postergan si arranca esto? (Ej: una decisión pendiente sin cerrar, una fase a medias, una revisión sin hacer).
- **Energía y contexto disponibles**: ¿esto cabe en lo que queda de día / semana, o va a quedar a medias y sangrando?
- **Costo de interrupción**: ¿es algo que se puede pausar limpio, o si lo arrancas hay que terminarlo o queda peor que como estaba?
- **Dependencias bloqueantes**: ¿necesitas decisiones / aprobaciones / datos externos antes de avanzar?

Tres respuestas válidas:
- **AHORA SÍ** — encaja con el momento, hay cabeza para terminarlo, no canibaliza nada urgente.
- **AHORA NO** — la idea es buena pero el momento es malo. Decir cuándo sí (después de X cosa concreta).
- **NUNCA** — si la conclusión es que la idea misma no aguanta, decirlo aquí y cerrar.

## Veredicto final (cierre obligatorio)

ALWAYS terminar con una sola línea de veredicto en este formato:

```
**VEREDICTO: [GO | NO-GO | DEPENDE]** — [una frase explicando por qué]
```

- **GO**: las 6 preguntas dan luz verde. Adelante.
- **NO-GO**: al menos una pregunta da rojo letal. La idea no pasa.
- **DEPENDE**: hay condiciones específicas (resuelve X, espera Y) bajo las cuales sí. Listarlas concretas.

Después del veredicto, **no** propongas implementación. Si el usuario dice "ok, hazlo", entonces sí. El validador es punto-y-aparte: termina aquí, espera respuesta.

## Reglas de tono

- Español neutro. Directo. Sin emojis.
- Sin adulación. Sin "¡qué buena idea!". Sin "interesante propuesta".
- Sin sarcasmo. Esto es análisis técnico, no comedia.
- Cuando algo es bueno, decir bueno. Cuando es malo, decir malo. Sin matizar para quedar bien.
- Máximo 600 palabras de output total. Si necesitas más, es señal de que la idea es demasiado grande para validar en un solo paso — pídele al usuario que la divida.

## Lo que esta skill NO hace

- No escribe código. Para eso, después del GO, el usuario invoca el flujo normal.
- No hace planes detallados de implementación. Para eso, después del GO, un planificador por fases (por ejemplo `/planificador`).
- No reemplaza a un abogado del diablo (una crítica destructiva y más profunda de la idea). validador es operativo, rápido, y se mete antes — en el momento exacto en que aparece la idea, no después.
- No se activa cuando el usuario pide ayuda con un bug, con un refactor, o con una pregunta. Solo cuando propone construir algo nuevo.

## Ejemplo del shape de output

```
## 1. Qué entendí
Quieres añadir un endpoint para que la web pueda consultar el historial
completo de cambios de un pedido. Asumo que eso implica también guardar
esos cambios (no lo dijiste, pero el endpoint sin tabla detrás no tiene
de dónde leer).

## 2. Opinión franca
La idea es buena para resolver reclamaciones, pero estás resolviendo el
caso fácil (lectura) antes de tener el caso difícil resuelto (escritura
estructurada de los cambios). Mal orden.

## 3. Mejoras posibles
- Antes del endpoint, define el schema de la tabla `pedido_cambios`
  (qué se guarda, quién, cuándo, qué transición).
- Considera si esto debe ser append-only event log o snapshots; afecta
  todo lo demás.

## 4. Viabilidad técnica
Con tu stack (Next.js + Supabase) es directo: 1 migración,
1 trigger en `pedidos`, 1 server action. Tiempo realista: 4-6h
incluyendo tests.

## 5. Qué se rompe
- La tabla `pedidos` ya tiene 3 sitios que la mutan. Necesitan
  envolverse en una transacción que escriba también en `pedido_cambios`.
- Tests de `pedidos.update()` van a fallar hasta que se actualicen.
- La API que usa la app móvil no se ve afectada.

## 6. ¿Es prudente AHORA?
NO. La integración de pagos sigue abierta y este trabajo la amplía 30%.
Termina pagos con el alcance actual y mete esta idea como la siguiente
fase explícita. Trabajarlo ahora alarga lo que está en curso sin ganancia
clara esta semana.

**VEREDICTO: NO-GO** — buena idea, mal momento. Después de cerrar pagos.
```
