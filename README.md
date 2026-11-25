# Skills Claude Code ↔ Codex

Para que Claude Code pelotee con Codex por su cuenta (plan y código) y tú solo
aparezcas al principio (producto/UX) y al final (revisar la feature en la PR).

| Skill | Qué hace |
|---|---|
| `peloteo` | **La que buscas.** "Quiero trabajar en X" → preguntas de producto → plan → Claude y Codex discuten el plan hasta `VEREDICTO: APROBADO` → implementa → verifica → discuten el código → PR con resumen del peloteo. |
| `codex-headless-usage` | La referencia: cómo llamar a `codex exec` sin que se cuelgue, qué modelo usar, flags. Las demás tiran de esta. |
| `codex-review` | Review suelta de un diff con Codex ("revísalo con codex"). |
| `codex-consult` | Discusión suelta por rondas sobre una decisión técnica. |

## Instalar

```bash
cp -R peloteo codex-headless-usage codex-review codex-consult ~/.claude/skills/
```

Requisitos:
- `codex` CLI ≥ 0.172 con sesión iniciada (`brew upgrade --cask codex`, luego `codex login`).
- `gh` con sesión iniciada si quieres que Claude abra la PR.
- Recomendado en `~/.codex/config.toml` (las skills asumen esto):

  ```toml
  model = "gpt-6.2-sol"
  model_reasoning_effort = "low"
  ```

  Sí, `low`: más effort no encontró más fallos en ninguna prueba, solo tarda más.
  `gpt-6.2-sol` es el que mejor salió en mis pruebas con regresiones reales
  (noviembre 2026). Si usas `gpt-5.6-sol`, se retira el 23-dic-2026.

## Usarlo

En Claude Code, dentro del repo:

> quiero trabajar en: <la tarea>

o `/peloteo <la tarea>`. Responde a sus preguntas de producto y vete. Vuelve
cuando te pase la PR.

Todo el rastro del peloteo (plan, prompts, respuestas de Codex) queda en
`.peloteo/<tarea>/` dentro del repo, excluido de git, por si quieres leer cómo
discutieron.

## Cómo funciona por dentro

- Codex corre **siempre en solo lectura**: piensa y revisa, Claude escribe.
- Cada tarea abre **una sesión de Codex** y la retoma con
  `codex exec resume <id>`, así Codex recuerda lo que se acordó en el plan cuando
  revisa el código.
- En el código hay dos pasadas: `codex review` nativo (ojos frescos, sin
  contexto) y la sesión del plan (¿cumple lo acordado?).
- Límites: 4 rondas en el plan, 3 en el código. Si no hay consenso, no se para:
  lo apunta en la PR con las dos posturas. Solo te pregunta si el desacuerdo
  bloquea de verdad.

## Adaptar

Lo más probable que quieras tocar, todo en `peloteo/SKILL.md`:
- **Paso 0**: qué tipo de cosas te tiene que preguntar a ti.
- **Paso 5**: cómo se verifica en tu stack (tests, levantar la app, etc.). Si
  tienes un `CLAUDE.md` en el repo con los comandos, mejor todavía.
- **Paso 7**: la plantilla de descripción de la PR.
- **Límites de rondas** si ves que se queda corto o se enrolla.
