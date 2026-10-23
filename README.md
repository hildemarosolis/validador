# validador

> Skill para Claude Code que pasa cada idea de construcción por un filtro de 6 chequeos antes del primer commit. Diseñada para operadores con múltiples sistemas en producción.

## El problema

Las sesiones de construcción con Claude Code tienen un sesgo brutal hacia decir que sí. Dices "vamos a hacer X" y antes de respirar ya hay arquitectura propuesta, diff sugerido, y 200 líneas en el chat. Se siente productivo y arruina proyectos.

El verdadero costo de una idea no es construirla. Es que esa idea **bloquee, retrase o rompa todo lo que ya está en producción y sin terminar**.

## Qué hace esta skill

Cuando dices `/validador` (o cuando aparezca una frase tipo "vamos a hacer", "se me ocurrió", "y si añadimos"), Claude pausa antes de cualquier construcción y devuelve 6 chequeos en orden:

1. **Qué entendí** — reformulación honesta de la idea
2. **Opinión franca** — postura clara, sin "depende"
3. **Mejoras posibles** — versión más simple, si la hay
4. **Viabilidad técnica** — con tu stack actual, qué tan factible es
5. **Qué se rompe** — impacto en lo ya construido
6. **¿Es prudente AHORA?** — la pregunta clave: timing > idea

Termina con un veredicto: **GO**, **NO-GO**, o **DEPENDE**.

Todo en menos de 600 palabras. Sin adulación, en español directo.

## Instalación

### Opción A — Skill global (recomendada)

```bash
git clone https://github.com/hildemarosolis/validador.git ~/.claude/skills/validador
```

Reinicia Claude Code. Disponible en todas tus sesiones.

### Opción B — Symlink desde repo local

```bash
git clone https://github.com/hildemarosolis/validador.git ~/REPOS/validador
ln -s ~/REPOS/validador ~/.claude/skills/validador
```

Ventaja: editas el SKILL.md en el repo y los cambios se reflejan al instante.

## Uso

### Disparo explícito

```
/validador

vamos a añadir un endpoint de auditoría que devuelva el historial
completo de cambios de un pedido
```

### Disparo automático (frases gatillo)

Sin tener que escribir el comando, Claude detecta frases tipo:
- "vamos a hacer", "hagamos", "construyamos"
- "se me ocurrió", "y si añadimos", "qué te parece si"
- "voy a meter", "quiero implementar"

Si la idea ya está confirmada y solo pides ejecución, el validador NO se activa.

## Filosofía

Esta skill es **operativa, rápida y pre-decisión**. Es distinta de:

| Skill | Cuándo |
|---|---|
| **validador** | En el momento exacto en que se te ocurre una idea, antes de planear nada |
| `/planificador` | Después del GO del validador, para descomponer el trabajo en fases |
| Un abogado del diablo | Para destruir críticamente una idea ya formada — más profundo, más largo |
| Una revisión de arquitectura | Para una decisión arquitectónica formal (ADR) |

El validador no reemplaza ninguna. Se mete antes que todas.

## Por qué confiar en el veredicto

- **Lee tu contexto antes de opinar**: revisa CLAUDE.md, README, package.json, BITACORA si los hay. No improvisa sobre un proyecto que no conoce.
- **La pregunta 6 es la importante**: "¿es prudente AHORA?" considera lo pendiente, no solo la idea en abstracto.
- **Veredicto obligatorio**: cierra con GO / NO-GO / DEPENDE, sin escape. No te deja con "podría ser".
- **No escribe código en la misma respuesta**: el validador termina aquí. Si dices "ok, hazlo", entonces sí.

## Ejemplo

Ver [`examples/`](examples/) — un caso típico procesado de principio a fin.

## Roadmap

- [x] v0.1 — versión inicial con 6 chequeos fijos
- [ ] v0.2 — recordar veredictos previos en la misma sesión para detectar patrones ("vienes proponiendo 3 cosas que dieron NO-GO seguidas")
- [ ] v0.3 — modo retrospectivo: pasar ideas YA construidas por el filtro para auditar deuda técnica
- [ ] v0.4 — integración con un log persistente de veredictos por proyecto

## Contribuir

Si pasas una idea por el validador y el veredicto te parece mal calibrado, abre un issue con la idea original + el veredicto que diste tú + el que dio la skill. Es el mejor feedback posible.

## Licencia

MIT — ver [LICENSE](LICENSE).

## Autor

Hildemaro Solís — segundo repo público. Parte de una serie de skills para builders que ya tienen demasiado en producción.

Pareja recomendada: [detector-humo](https://github.com/hildemarosolis/detector-humo) (para evaluar ideas externas / reels de redes) + validador (para evaluar tus propias ideas internas).
