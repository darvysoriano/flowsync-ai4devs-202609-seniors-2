# Prompts del Módulo 3

Solo lo que se teclea (en la terminal o en Claude Code), en orden, y cuándo. El resto del contexto (qué decir, por qué, cómo reacciona el mentor) vive en `GUION.md`. **Todos van pre-escritos y listos para copiar-pegar**: no se ensambla ni se improvisa nada en vivo (ver "Setup previo del mentor"). Los números de `PROMPT N` coinciden con los del `GUION.md`.

> Hoy los outputs son **código y specs**, de punta a punta: migración, API y pantalla. El harness del Módulo 1 sigue activo (skill `commit`, hook de formateo, subagente revisor), y encima se instala OpenSpec.

> **Nota — cómo se contesta a `propose`, y por qué ya no hay fichas que pegar.** `/opsx:propose` **pregunta antes de escribir** cuando detecta ambigüedad que afecte al alcance o al comportamiento observable (está en sus propias instrucciones: *"Do NOT proceed without understanding what the user wants to build"*). Verificado en dos ejecuciones en directo: **no pregunta en abierto, pregunta con OPCIONES y marca una como recomendada.** Así que el mentor **no pega nada: elige la recomendada en cada pregunta y lo dice en voz alta.**
>
> Lo que **no** puede quedar a merced de esa elección va **dentro del propio PROMPT 6 y 12**, no en una ficha aparte. Es deliberado y es mejor: el prompt llega **antes** de que el agente genere sus opciones, así que las opciones ya salen respetando la restricción. Cada una de esas líneas protege algo concreto de una demo posterior o repite un criterio literal del backlog; están marcadas en su sitio.

---

## Bloque 4 — Demo 1: qué es una spec viva

### COMANDO/PROMPT 1 — Inicializar OpenSpec en el proyecto (Paso 1)
En la **terminal**, en la raíz del repo. El prework lo hizo en un sandbox desechable; esta es la primera vez que se hace sobre FlowSync.
```bash
openspec init
```
**Sin `--tools`, a propósito: se quiere ver el asistente.** Pregunta por el IDE y se elige **Claude Code**. Enseñar la lista completa es contenido: alguien va a preguntar por Copilot o VS Code (pasó en una edición anterior), y la respuesta es que **leen las mismas skills que instala la opción de Claude**, así que quedan cubiertos.

⚠️ **Después hay que reiniciar Claude Code por completo.** El propio comando lo dice al terminar (*"Restart your IDE for the new commands to take effect"*): los slash commands se cargan al arrancar, así que en una sesión ya abierta `/opsx:` no aparece. No es un fallo de instalación.

### COMANDO/PROMPT 1b — Enseñar que `specs/` nace vacía (Paso 2)
```bash
ls -R openspec/
```
```bash
openspec list --specs
```
`list --specs` responde `No specs found.` Es el punto de partida del que nace todo lo demás.

### PROMPT 2 — Ingeniería inversa: la spec de lo que ya existe (Paso 3)
Se **pega** en Claude Code, ya reiniciado. **Es una desviación consciente de lo que recomienda OpenSpec**, y el guion la anuncia como tal.
```
Vamos a documentar como spec de OpenSpec la capability que YA existe en este proyecto: la de cuentas y acceso (registro, inicio de sesión, sesión y perfil). Esto es ingeniería inversa: leer el código y escribir lo que hace HOY, no proponer nada nuevo ni cambiar una línea.
1. Explora el vertical de autenticación COMPLETO, en las dos capas. En `backend/`: sus controladores, el modelo de usuario, sus validadores, sus rutas y sus middlewares. En `frontend/`: las pantallas de acceso, el estado de sesión y la protección de rutas. Ignora todo lo que no sea autenticación.
2. Escribe `openspec/specs/auth/spec.md` con el formato exacto de una spec VIVA de OpenSpec: un `## Purpose` de una o dos frases, y debajo `## Requirements`, con `### Requirement:` en los que el sistema SHALL hacer algo, y bajo cada uno al menos un `#### Scenario:` de cuatro almohadillas con sus dos viñetas `- **WHEN**` y `- **THEN**`. En castellano, salvo las mayúsculas RFC-2119.
3. NO uses secciones ADDED, MODIFIED ni REMOVED: eso es de un delta, y esto no es un delta, es la verdad actual del sistema.
4. Describe solo comportamiento observable desde fuera. Nada de nombres de clase ni de rutas de archivo. Para el backend, observable = la petición y la respuesta HTTP. Para el frontend, observable = lo que una persona ve y puede hacer en pantalla.
5. Al terminar, dime qué comportamiento del código NO has podido expresar como requisito, y por qué.
```
> 💥 **El punto 5 es el que da el mejor momento de la demo.** En el ensayo devolvió ocho puntos, y dos eran **defectos reales** en código ya entregado (el cierre de sesión no envuelve su respuesta como el resto y su mensaje está en inglés; las credenciales inválidas devuelven `400` en vez de `401`). No lo recortes.

### COMANDO/PROMPT 3 — Validar y ver la spec en la CLI (Paso 4)
```bash
openspec validate auth --strict
```
```bash
openspec list --specs
```

### COMANDO/PROMPT 4 — Versionar (Paso 5)
Ejecutar `/commit`. Mensaje esperado: `docs(openspec): spec viva de la capability de cuentas y acceso`.

---

## Bloque 5 — Demo 2: la base de tareas, con OpenSpec

### COMANDO/PROMPT 5 — El perfil, antes de que muerda (Paso 1)
```bash
openspec config list
```
La salida incluye `workflows: propose, explore, apply, update, sync, archive (from core profile)`: **seis**, y `verify` no está entre ellos.

### PROMPT 6 — Proponer el change de la base, full-stack (Paso 2)
Nombra las historias por su ruta para que el agente lea los criterios reales del repo, no los invente.
```
/opsx:propose Dale de alta la capability de tareas del equipo, que todavía no existe. Llama al change exactamente `add-task-list` y a la capability exactamente `tasks` (el delta va en `specs/tasks/spec.md`, no inventes otro nombre). El alcance son cinco historias del backlog, con sus criterios de aceptación: @docs/backlog/E3-actividad-equipo/us-lista-compartida.md y @docs/backlog/E2-gestion-tareas/us-crear-tarea.md, @docs/backlog/E2-gestion-tareas/us-titulo-obligatorio.md, @docs/backlog/E2-gestion-tareas/us-responsable-y-estado-por-defecto.md y @docs/backlog/E2-gestion-tareas/us-cambiar-estado.md. Léelas enteras antes de proponer nada.
Este change es FULL-STACK: la API de AdonisJS en `backend/` Y la interfaz de React 19 en `frontend/`. La interfaz reutiliza los componentes que ya hay en `frontend/src/components/ui/` y el patrón de páginas y rutas que ya existe para el login. No montes design system nuevo ni añadas dependencias.
SIN TESTS: no montes base de pruebas ni escribas tests en este change. Ni tasks.md ni el delta deben incluirlos.
Los tres estados de una tarea viajan por la API como `pending`, `in_progress` y `done`, y se pintan en pantalla como Pendiente, En curso y Hecho. Es un conjunto cerrado: cualquier otro valor se rechaza con 422. NO uses los valores en castellano como identificadores.
Seis restricciones que NO son negociables, aunque no me preguntes por ellas:
1. La tarea NO tiene fecha de vencimiento en este change, y la lista no muestra fechas ni marcas de vencida. No la añadas ni la dejes preparada.
2. La API tiene exactamente tres operaciones: listar todas las tareas, crear una y actualizarla. NO hay lectura individual de una tarea, no hay borrado y no hay endpoints de equipo.
3. Al crear, el título es lo ÚNICO que se pide: el formulario no ofrece ni sugiere responsable, estado ni fecha. La tarea nace en pendiente y con quien la crea como responsable.
4. NO hay regla de orden decidida para la lista. No inventes un criterio de ordenación, no ordenes explícitamente, y anótalo como punto abierto en el proposal.
5. Una sola lista compartida, la misma para todos: no hay tareas privadas ni vista de "mis tareas", y cualquiera puede cambiar el estado y el responsable de cualquier tarea.
6. En la lista, el responsable se identifica por su NOMBRE. Si no tiene nombre puesto se pinta "Sin nombre": nunca su correo ni su id.
Traduce cada criterio de aceptación a scenarios de la spec delta, no lo copies tal cual. Y antes de escribir, pregúntame lo que de verdad cambie el alcance o el comportamiento observable.
```
> **Por qué esas seis y no otras.** No son gusto del autor: cada una protege algo que se rompe si el diálogo de opciones decide por su cuenta. **1 y 2 sostienen la Demo 3** — sin la 1 el agente adelanta la fecha y la Demo 3 se queda sin delta; sin la 2 no habrá `REMOVED` que enseñar, porque el `REMOVED` nace justo de que la spec viva diga *"no hay lectura individual"*. **3, 5 y 6 son criterios literales** de las historias, y la opción recomendada no tiene por qué respetarlos. **4 es un punto abierto del backlog** que el agente rellenaría solo. Todo lo demás (navegación, estado vacío, selector de personas) se deja al diálogo **a propósito**: ahí es donde el mentor enseña cómo se elige.

### ▸ Y ahora el diálogo de opciones — no se pega nada, se elige (Paso 2)
`propose` se para y abre **un cuadro de preguntas con opciones**, cada una con una marcada como **recomendada**. En directo salieron cuatro, en pestañas: título máximo, control de estado, orden de la lista y navegación.

**Qué hace el mentor: elegir la recomendada en todas, y decir en voz alta que lo está haciendo y por qué.** No hay nada que copiar ni que pegar.

⚠️ Si cancelas el cuadro sin querer, **se vuelve a lanzar el PROMPT 6 y ya está**. Pasó en el ensayo; no rompe nada, solo cuesta volver a proponer.

### COMANDO/PROMPT 7 — Ver el change montado antes de tocarlo (Paso 3)
En la **terminal**, en la pestaña de al lado. Es lo que se proyecta mientras se revisa el contrato.
```bash
openspec list
```
```bash
openspec show add-task-list
```
```bash
openspec validate add-task-list --strict
```
⚠️ Si `show` dice que no existe, es el **nombre del change**: el PROMPT 6 lo pide literal, pero si el agente derivó otro de la descripción, el primer `openspec list` te lo dice. Usa ese nombre en el `apply` y en el `archive`. Igual en la Demo 3 con `add-task-due-date`.

### PROMPT 8 — Corregir el contrato ANTES del código (Paso 3) · **OPCIONAL**
**Este prompt puede no ejecutarse, y no pasa nada.** Solo si al revisar el delta falta o sobra algo. **Va sobre el change, no sobre el código**, y esa es la lección.
```
/opsx:update add-task-list Antes de implementar, corrige el contrato: [decir aquí el ajuste concreto que se haya visto al revisar]. Toca el delta de specs/ y, si hace falta, tasks.md. No escribas código todavía.
```

### COMANDO/PROMPT 9 — Implementar, primera tanda: backend (Paso 4)
Se lanza **en dos tandas**. No es solo por atención: **partirlo también sale más rápido** (medido: 11 min 45 s partido frente a 13 min 36 s de una tirada), y de paso permite un `/compact` entre medias.
```
/opsx:apply add-task-list Haz solo las secciones de backend de tasks.md (modelo de datos y API). Para al terminar la última de backend y no toques el frontend todavía.
Seguimos en la misma rama: no crees una rama nueva, no abras PR y no lances el subagente adversarial-reviewer.
```

### COMANDO/PROMPT 9b — Implementar, segunda tanda: frontend (Paso 4)
Tras enseñar la API funcionando. El `apply` continúa donde lo dejó, leyendo la checklist del disco.
```
/opsx:apply add-task-list Continúa con las secciones de frontend de tasks.md: la capa de acceso a la API y la interfaz de la lista. La sección de verificación de punta a punta la hago yo a mano en el navegador, no la marques.
Seguimos en la misma rama: no crees una rama nueva, no abras PR y no lances el subagente adversarial-reviewer.
```

### COMANDO/PROMPT 10 — Archivar y fusionar en la spec viva (Paso 6)
En la **terminal**. Es determinista y no gasta tokens: lo hace la CLI, no el modelo.
```bash
openspec archive add-task-list
```
**Va solo, y se para aquí.** Su salida dice qué spec actualiza, con qué totales, y **avisa si quedan tareas sin marcar**. Eso se lee en pantalla antes de teclear nada más.

### COMANDO/PROMPT 10b — Y ahora sí, la lista de specs vivas (Paso 6)
```bash
openspec list --specs
```
Devuelve **dos**. Después, `ls openspec/changes/archive/`.

### COMANDO/PROMPT 11 — Versionar (Paso 7)
Ejecutar `/commit`. Mensaje esperado: `feat(tasks): capability de tareas con su spec viva`.

---

## Bloque 6 — Demo 3: la fecha de vencimiento, delta sobre la spec viva

### PROMPT 12 — Proponer el change de la fecha (Paso 2)
```
/opsx:propose Añade la fecha de vencimiento y las tareas vencidas, según @docs/backlog/E2-gestion-tareas/us-fechas-vencimiento.md. Léela entera. Llama al change exactamente `add-task-due-date`. Backend y frontend, SIN TESTS.
La fecha es de calendario, sin hora: una tarea cuya fecha es HOY todavía no está vencida, lo estará mañana.
Seis restricciones que NO son negociables, aunque no me preguntes por ellas:
1. Una tarea está vencida si se cumplen las TRES a la vez: tiene fecha, esa fecha es anterior a hoy, y su estado NO es `done`. Sin fecha no se vence nunca, y una tarea en `done` nunca está vencida por antigua que sea su fecha.
2. Ese veredicto lo da el BACKEND, y la API lo expone como un campo booleano `isOverdue` en la representación de una tarea. NO lo resuelva el frontend por su cuenta. Si para respetar el huso horario de cada persona necesitas su día, que el cliente lo mande y el servidor devuelva el veredicto; lo que no vale es que la API no diga si una tarea está vencida.
3. `isOverdue` NO se persiste en ninguna columna: se calcula en cada lectura. Nada de jobs ni de procesos que marquen tareas por la noche. Y el cliente no puede enviarlo: si lo envía, se ignora.
4. Se acepta poner una fecha anterior a hoy, al crear y al actualizar: registrar algo que ya llega tarde es legítimo, nace vencida y no se rechaza. La fecha se puede poner, cambiar y quitar, y quitarla se hace enviándola explícitamente vacía.
5. La historia dice "al abrir la tarea" y ese endpoint no existe todavía. Este change añade la lectura individual de una tarea, que es la superficie mínima que la historia necesita, y lo anota como decisión en el design con su punto abierto. La pantalla de detalle completa NO entra.
6. La LISTA no cambia lo que muestra: sigue con título, responsable y estado, sin fecha y sin marca de vencida en pantalla.
Fuera de alcance: notificaciones, recordatorios, recurrencia, ordenar o filtrar por fecha, y tests de cualquier tipo.
Y antes de escribir, pregúntame lo que de verdad cambie el alcance o el comportamiento observable.
```
> 🚨 **La restricción 2 es la que más caro se paga si se cae, y no es teórica: pasó.** En una ejecución el agente decidió, con buen argumento de husos horarios, que **la regla de vencimiento viviera solo en el frontend** — y lo dejó escrito y honesto en su `design.md`. La API se quedaba sin `isOverdue`. El problema es que **el Módulo 4 ya está escrito sobre ese campo**: su hallazgo estrella es precisamente un `isOverdue` que devuelve `true` aunque el estado sea `done`. Sin el campo, la siguiente sesión se queda sin material. Es la misma lección que los estados en inglés: **lo que consume otra sesión se fija en el prompt, no se deja a una elección en vivo.**
>
> ⚠️ **Ojo con la 6, que se lee mal:** la lista no *muestra* la marca, pero la API **sí la devuelve**. No son incompatibles, y conviene tenerlo claro por si un alumno lo pregunta.

### ▸ Y aquí también: se eligen las opciones recomendadas (Paso 2)
Igual que en la Demo 2 — `propose` vuelve a abrir su cuadro de preguntas con opciones. **Se elige la recomendada en todas y se dice en voz alta.** No hay ficha que pegar.

### COMANDO/PROMPT 13 — Leer el delta, que es lo que se proyecta (Paso 3)
```bash
openspec show add-task-due-date --json --deltas-only
```
Si la salida en JSON queda ilegible en pantalla, abrir directamente el archivo en el editor: `openspec/changes/add-task-due-date/specs/tasks/spec.md`. Es el archivo que se lee en voz alta.

### PROMPT 13b — El momento humano: que defienda su diseño (Paso 3) · **OPCIONAL**
Se **pega** en Claude Code **después** de abrir el `design.md` en pantalla. **Está escrito a propósito sin nombrar qué decidió**, porque eso cambia de una ejecución a otra: se le pide que **nombre él mismo la alternativa que descartó**, que además es más fuerte que adivinarla.
```
Abre el design.md de este change y dime, de las decisiones técnicas que has tomado, cuál es la que más te ha costado y qué alternativa descartaste para quedarte con ella. Defiende tu elección frente a esa alternativa con consecuencias concretas sobre esta spec: qué escenario del delta se rompería con cada una. Si sigues pensando que tu decisión es la buena, no cambies nada.
```
> ⚠️ **Es OPCIONAL y suele saltarse por tiempo.** Si vas justo, sáltalo sin remordimiento: la Slide "Se negocia en la spec, no en el PR" se puede dar sobre el `design.md` abierto en pantalla, señalando con el dedo la decisión y el punto abierto, sin teclear nada. Lo que **no** se recorta es abrir el `design.md`.

### COMANDO/PROMPT 14 — Instalar el séptimo comando, delante de la clase (Paso 4)
En la **terminal**, en la raíz del repo. Aquí se cobra la trampa que se anunció en la Demo 2: `verify` **no está** en el perfil `core` y hay que activarlo a mano.
```bash
openspec config profile
```
**Es un asistente interactivo, como el `openspec init` del principio, y por eso se hace en vivo.** El recorrido, verificado:
1. Enseña el estado actual: `Delivery: both` · `Workflows: 6 selected (core)`.
2. Pregunta *"What do you want to configure?"* → elegir **`Workflows only`**.
3. Sale una lista con los **doce** workflows que existen. Marcar **`verify`** (dejando los seis que ya estaban).
4. Enseña el diff: `profile: core -> custom`, y `workflows` con uno más.
5. Como está dentro de un proyecto OpenSpec, pregunta *"Apply changes to this project now?"* → **sí**. Corre `openspec update` solo y escribe `.claude/commands/opsx/verify.md`.
6. Termina diciendo: *"Run `openspec update` in your other projects to apply."* **Esa frase es la lección, y sale de la herramienta, no del guion.**

⚠️ **Después hay que reiniciar Claude Code**, por el mismo motivo que en el PROMPT 1: los slash commands se cargan al arrancar. **Reiniciar aquí no cuesta nada**: está verificado que el `apply` no necesita el contexto de la conversación, porque re-lee el change y el `tasks.md` del disco.

### COMANDO/PROMPT 15 — Implementar (Paso 5)
```
/opsx:apply add-task-due-date
Seguimos en la misma rama: no crees una rama nueva, no abras PR y no lances el subagente adversarial-reviewer.
```

### COMANDO/PROMPT 15b — `verify`: ¿de verdad el código cumple el contrato? (Paso 6)
Se teclea **después del `apply` y ANTES del `archive`**, que es el único punto donde tiene sentido: lee el código contra el change que todavía está abierto.
```
/opsx:verify add-task-due-date
```
> 💥 **Devuelve un informe con tres dimensiones** —*Completeness*, *Correctness*, *Coherence*— y clasifica los hallazgos en **CRITICAL / WARNING / SUGGESTION**. Y **ejecuta comprobaciones de verdad** por el camino (`typecheck`, `lint`, `build`), no solo lee. **Medido: 2 min 54 s**, aunque sobre el change grande de la Demo 2; el de la fecha es más pequeño, así que eso es techo y no presupuesto. En esa medición encontró **cuatro CRITICAL reales**, y el mejor fue código muerto: una pantalla implementada que **nadie había enrutado**, así que el `build` pasaba porque compilaba, no porque se usara.
>
> ⚠️ **Puede volver limpio, y eso también es contenido**, no un fallo de la demo: *"lo he pasado, no ha encontrado nada, y ahora sé que no ha encontrado nada — que es distinto de no haber mirado"*. No lo fuerces ni lo repitas buscando hallazgos.
>
> ⚠️ **Es la IA opinando, no una puerta**: el propio `verify.md` se autodeclara `experimental`, no bloquea el `archive`, y está sesgado a callarse. Decirlo en voz alta es parte del bloque.

### COMANDO/PROMPT 16 — El wow: archivar (Paso 7)
En la **terminal**. **Va solo**: su salida es una línea y es la prueba — dice qué spec actualiza y con qué totales de añadidos y modificados.
```bash
openspec archive add-task-due-date
```

### COMANDO/PROMPT 16b — Y la prueba en el diff (Paso 7)
```bash
git diff --stat openspec/specs/tasks/spec.md
```
Y después, **abrir `openspec/specs/tasks/spec.md` en el editor** y buscar `ADDED`. No hay ninguno: ese es el momento.

### COMANDO/PROMPT 17 — Cerrar la tarea de verdad, y NO esperarlo (Paso 8)
**Esta es la única vez de la sesión que se deja actuar al harness entero**, y se lanza **sin quedarse mirando**: en cuanto arranca, seguir con el Bloque 8. Vuelves al final, antes del cierre, a enseñar lo que encontró.
```
Da la tarea por cerrada siguiendo las reglas de proceso de CLAUDE.md: commitea, abre el PR y pasa el subagente adversarial-reviewer sobre todo el trabajo de esta rama.
```
> 💥 **Como todo ha vivido en la misma rama, revisa las TRES demos, no solo la última.** Medido: **11 min 16 s**, que es justo lo que dura el Bloque 8 más el Q&A. En el ensayo devolvió dos hallazgos reales, y uno de ellos **incumplía un escenario de nuestra propia spec** (*«Token inválido o caducado»*: un `401` a mitad de sesión dejaba al usuario atascado en la lista en vez de devolverlo al login). Ni el `apply` ni el `validate` lo vieron.
