---
name: peloteo
description: Flujo completo de una tarea con Codex como revisor, sin el humano en medio. Claude pregunta lo de producto/UX, hace el plan, lo discute con Codex hasta consenso, implementa, verifica, vuelve a discutir el código con Codex y sube una PR consensuada. Úsala cuando el usuario diga "quiero trabajar en X", "peloteo", "pelotéalo con codex", "hazlo con codex hasta la PR" o escriba /peloteo. No para preguntas rápidas ni cambios de una línea.
---

# Peloteo Claude ↔ Codex hasta la PR

El usuario decide **producto y UX**. Claude y Codex deciden **lo técnico** entre
ellos. El usuario solo vuelve a aparecer para revisar la *feature* (no el código)
en la PR y aprobarla.

Claude implementa. Codex piensa y revisa (siempre en solo lectura).

> Cómo invocar Codex, flags, modelos y el problema del stdin: [[codex-headless-usage]].
> Revisión de diffs: [[codex-review]]. Discusión por rondas: [[codex-consult]].

## 0. Preguntas (la única parte con el usuario)

Antes de nada, pregunta **solo** lo que no puedes decidir tú: comportamiento
esperado, casos de uso, textos, flujos, qué pasa en los bordes desde el punto de
vista del usuario. Nada técnico: eso lo resolvéis Codex y tú.

Cuando esté respondido, dilo en una línea ("Listo, me voy a pelotearlo con Codex;
te aviso con la PR") y **no vuelvas a molestar** salvo por lo del paso 7.

## 1. Preparar

```bash
REPO=$(git rev-parse --show-toplevel)
SLUG=<nombre-corto-de-la-tarea>
DIR="$REPO/.peloteo/$SLUG"
mkdir -p "$DIR"
grep -qx '.peloteo/' "$REPO/.git/info/exclude" || echo '.peloteo/' >> "$REPO/.git/info/exclude"
git switch -c "$SLUG"   # desde la rama principal actualizada
```

`.peloteo/` queda fuera de git (exclude local, no toca el `.gitignore`): ahí van
el plan, los prompts y las respuestas de Codex. Las variables de shell **no
persisten** entre llamadas a Bash: repite `DIR=...` o usa rutas absolutas, y
guarda el id de la sesión de Codex en un fichero.

## 2. Plan

Explora el repo y escribe `$DIR/plan.md`: objetivo, decisiones de producto del
paso 0 (tal cual las dijo el usuario), ficheros que vas a tocar, enfoque, riesgos,
y **cómo vas a verificar** que funciona (tests, comandos, qué mirar en la app).

## 3. Peloteo del plan

**Ronda 1** — sesión nueva, guarda el id:

```bash
cd "$REPO" && cat > "$DIR/plan-prompt-1.md" <<'PROMPT'
Eres revisor técnico de un plan de implementación. Lee .peloteo/<slug>/plan.md
y el código que necesites del repo (solo lectura).

Las decisiones de producto/UX están cerradas: no las discutas, solo avisa si
alguna es técnicamente imposible o muy cara.

Busca: enfoque equivocado, piezas que faltan, cosas que se rompen, una forma más
pequeña de hacerlo, y huecos en la verificación.

Responde con objeciones numeradas (concretas, con fichero/función si aplica) y
termina con UNA línea exactamente así:
VEREDICTO: APROBADO      (si no tienes objeciones que importen)
VEREDICTO: CAMBIOS       (si las tienes)
PROMPT
codex exec --cd "$REPO" --sandbox read-only --json \
  -o "$DIR/plan-review-1.md" - < "$DIR/plan-prompt-1.md" > "$DIR/plan-review-1.jsonl"
head -1 "$DIR/plan-review-1.jsonl" | sed -E 's/.*"thread_id":"([^"]+)".*/\1/' > "$DIR/codex-thread"
cat "$DIR/plan-review-1.md"
```

**Responde a cada objeción** — no las aceptes por defecto:
- De acuerdo → cambia `plan.md`.
- En desacuerdo → explica por qué, con evidencia del código.
- No lo ves claro → pregúntale.

**Ronda 2+** — misma sesión (Codex recuerda la ronda anterior):

```bash
cd "$REPO" && cat > "$DIR/plan-prompt-2.md" <<'PROMPT'
He actualizado .peloteo/<slug>/plan.md. Sobre tus objeciones:
1. <aceptada: qué cambié> / <rechazada: por qué>
2. ...
Revisa el plan actualizado y mis respuestas. Mismo formato, misma línea final.
PROMPT
codex exec resume "$(cat "$DIR/codex-thread")" -c sandbox_mode='"read-only"' \
  -o "$DIR/plan-review-2.md" - < "$DIR/plan-prompt-2.md"
```

**Parar cuando:** `VEREDICTO: APROBADO`, o tras **4 rondas**. Si después de 4
siguen quedando desacuerdos técnicos, no los resuelvas a la fuerza: anótalos para
la PR (paso 7) con las dos posturas y sigue con tu versión si es la más pequeña y
segura; si el desacuerdo bloquea de verdad, ese es el único caso en que paras y
preguntas al usuario, explicado en términos de producto.

## 4. Implementar

Implementa el plan consensuado en la rama. Commits pequeños con sentido.

## 5. Verificar (antes de enseñarle nada a Codex)

Ejecuta lo que pusiste en "cómo vas a verificar": tests, build, lint, y si hay
interfaz, arranca la app y comprueba la feature de verdad. Arregla lo que falle.
Codex revisa código; **que funcione lo compruebas tú**.

## 6. Peloteo del código

Dos pasadas por ronda, porque ven cosas distintas:

**a) Ojos frescos** — `codex review` nativo, sin contexto del plan:

```bash
BASE=$(git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null | sed 's@^origin/@@')
BASE=${BASE:-main}   # main/master/develop
cd "$REPO" && codex review --base "$BASE" > "$DIR/code-review-1.md" 2>&1 < /dev/null
tail -60 "$DIR/code-review-1.md"   # los hallazgos [P#] fichero:línea van al final
```

`codex review` **no acepta prompt** junto a `--base`/`--uncommitted`/`--commit`
(codex-cli 0.160 da error). Si quieres darle contexto, usa la pasada b.

**b) Contra lo acordado** — en la misma sesión del plan, que ya recuerda las
decisiones:

```bash
cd "$REPO" && cat > "$DIR/code-prompt-1.md" <<PROMPT
He implementado el plan. Revisa el diff (git diff $BASE...HEAD) contra lo que
acordamos: lo que falte, lo que se desvíe, bugs y casos borde. Además, el review
nativo dijo esto; dime si estás de acuerdo:
$(tail -60 "$DIR/code-review-1.md")
Mismo formato, misma línea VEREDICTO final.
PROMPT
codex exec resume "$(cat "$DIR/codex-thread")" -c sandbox_mode='"read-only"' \
  -o "$DIR/code-r1.md" - < "$DIR/code-prompt-1.md"
```

Igual que con el plan: arregla lo que tenga razón, rebate lo que no (en esa misma
sesión). Vuelve a verificar (paso 5) y repite hasta `VEREDICTO: APROBADO` sin
hallazgos P0/P1 que aceptes, **máximo 3 rondas**.

En repos grandes una review puede tardar varios minutos: lánzala con
`run_in_background: true` y no la mates a medias.

## 7. PR

Push de la rama y PR. **No la mergees nunca**: eso lo hace el usuario.

La descripción es para alguien que revisa la feature, no el código:

```markdown
## Qué cambia
<en términos de producto: qué ve / qué puede hacer el usuario ahora>

## Cómo probarlo
<pasos concretos: dónde ir, qué pulsar, qué debería pasar>

## Verificación
<qué tests/comandos corriste y qué comprobaste a mano>

## Peloteo con Codex
- Plan: N rondas. Cambios que salieron de ahí: <lista corta>
- Código: N rondas. Arreglado: <lista corta>
- Sin consenso (si hay): <punto> — Claude: <postura> / Codex: <postura> → qué se hizo y por qué
```

Al usuario, en el chat: el enlace a la PR y "cómo probarlo". Nada más.

## Reglas

- Codex **siempre en solo lectura** (`--sandbox read-only`, y en `resume`
  `-c sandbox_mode='"read-only"'`). Quien escribe código eres tú.
- **Cierra siempre el stdin** (`< fichero`, heredoc o `< /dev/null`) o Codex se
  queda colgado esperando entrada.
- Peloteo ≠ obedecer. Si Codex propone algo más grande que no hace falta, di que no
  y explica por qué. El objetivo es el cambio más pequeño que está bien.
- No subas la effort de Codex para "pensar más": no encuentra más cosas (ver
  [[codex-headless-usage]]). Si una ronda se queda corta, pregunta mejor.
