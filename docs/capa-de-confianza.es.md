# La capa de confianza

[English](capa-de-confianza.md)

Resumen propio de la dirección de Torbyn para equipos de agentes de código.
Es un diseño: casi todo lo que describe **no está construido**. Al final se
dice qué existe hoy.

## El problema

Varios agentes de código trabajando a la vez sobre un mismo proyecto lo hacen
sin garantías:

1. **Se pisan.** Dos agentes editan el mismo archivo y gana el último que
   escribe. Nadie se entera hasta que algo deja de funcionar.
2. **No se sabe quién hizo qué.** En una carpeta compartida, git ve cambios,
   no autores.
3. **Deshacer es trabajo a mano.** Si un agente rompe algo, volver atrás sin
   perder lo de los demás exige saber git, y lo que no estaba guardado en un
   commit se pierde.
4. **Juntar el trabajo es a ciegas.** Con una rama por agente, unir tres ramas
   sigue siendo la tarea manual de siempre.
5. **«Terminado» no significa «funciona».** El agente dice que acabó; nadie
   corrió las pruebas.
6. **Cuando una herramienta se agota, el trabajo queda a medias.** Seguir en
   otra exige explicarle todo otra vez.

## La idea

Torbyn no compite con los agentes ni los dirige. Pone **las garantías
alrededor**:

- quién puede tocar qué;
- cómo volver atrás;
- qué se junta;
- qué está verificado;
- cómo se sigue cuando una herramienta se cae.

La persona sigue mandando: cada garantía la activa, la configura o la ejecuta
ella.

## Las reglas que no cambian

- **Torbyn no lee lo que pasa en las terminales.** Sus señales salen de git,
  de si un proceso sigue abierto y de su código de salida, nunca del texto de
  una terminal.
- **Decide la persona.** Nada se ejecuta por iniciativa de Torbyn ni de un
  agente: deshacer, juntar, verificar o pasar el trabajo a otra herramienta son
  clics suyos.
- **Nada sale del equipo.** Todo es git local y archivos locales.

## Los cinco pilares

### 1. Quién puede tocar qué

Un formato abierto, en la carpeta `.torbyn/` del proyecto, para que la
persona asigne rutas a cada agente y todas las herramientas lo lean.

- Con las herramientas que permiten comprobar una edición antes de hacerla,
  Torbyn la impide si la ruta es de otro.
- Con las demás, la regla llega como instrucción, y Torbyn avisa después si
  alguien se salió. Para eso cada agente tiene que trabajar en su propia
  carpeta con git.

### 2. Puntos de restauración y deshacer

Antes de abrir cada agente, un punto de restauración de su carpeta. Incluye lo
que todavía no está en un commit y los archivos nuevos, y no toca nada de la
persona: ni sus ramas ni su índice. Lo ignorado, como `.env` o
`node_modules`, no entra.

- **Deshacer siempre deja otro punto antes**, así que deshacer también se
  puede deshacer.
- Con un agente en su propia carpeta, se deshace entero y de forma exacta.
- En una carpeta compartida, se deshace archivo por archivo, y la persona
  elige cuáles.

### 3. Revisión y fusión

Una vista por proyecto con lo que cambió cada agente. Avisa de los choques
**antes** de que pasen, mientras los agentes todavía trabajan («estos dos
agentes chocarían en este archivo»), sin tocar ninguna carpeta.

Solo se ofrece juntar el trabajo de un agente si:

- está en un commit;
- está verificado;
- no toca rutas de otro;
- no choca.

Siempre hay un punto de restauración antes, y Torbyn **no resuelve choques
por su cuenta**: dice qué archivos chocan y con quién.

### 4. Verificación

La persona fija los comandos que para su proyecto significan «terminado»: sus
pruebas, el comprobador de tipos, el linter. Torbyn los corre en la carpeta
del agente.

- **El veredicto es el código de salida**, nunca una lectura del texto.
- **Va atado al estado exacto de los archivos.** Si algo cambió después, vuelve
  a «sin verificar».
- **Los comandos los fija solo la persona**, nunca el repositorio.
- **Corren con un entorno mínimo** y con un tiempo máximo, sin más permisos
  que la terminal del propio agente.

### 5. Continuidad

«Continuar en…» pasa la tarea de un agente a otra herramienta, en la misma
carpeta y la misma rama. Lo pulsa la persona, nunca es automático.

El agente nuevo recibe **solo referencias**: qué rama, desde qué commit y qué
mirar en git. Nunca texto escrito por otra IA. Si el agente anterior dejó una
nota, se le dice dónde está y la lee él.

## Qué existe hoy

**Construido:**

- **Rutas asignadas.** La persona asigna partes del proyecto a un agente o a
  sí misma, en el lanzador o mientras trabajan.
  - Con Claude Code, Torbyn impide que un agente edite o cree archivos en una
    ruta de otro, también si la ruta se asignó después de abrirlo. Para eso
    hace falta Git para Windows.
  - Con las demás herramientas es una instrucción, y Torbyn avisa si se
    salieron cuando cada agente trabaja en su propia carpeta con git.
- **El contrato de trabajo.** Cada proyecto tiene tres listas: las rutas
  reservadas a un agente, las de solo lectura (las que la persona se asigna a
  sí misma) y, si se quiere, las rutas en las que un agente puede escribir.
  - Claude Code lo cumple con el mismo bloqueo de las rutas asignadas.
  - Las demás herramientas lo reciben como instrucción.
  - Torbyn lo comprueba con git contra el commit que tenía la carpeta del
    agente al lanzarlo, y cada celda dice «Contrato: cumple», «Contrato: 1
    incumplimiento» (o más) o «Contrato: sin comprobar» con el motivo. En una
    carpeta compartida dice siempre «Contrato: sin comprobar»: ahí no se sabe
    quién cambió qué.
- **Una carpeta por agente.** Cada agente elige dónde trabaja; con un
  repositorio con commits, cada uno en su propio worktree y su propia rama.
- **Cambios por agente.** Lo que cambió cada uno, archivo por archivo, leído
  con git sin ejecutar nada del repositorio.

**No cubre:** lo que un agente cambia con comandos de terminal (Bash).

**Diseñado, sin construir:** los puntos de restauración, la revisión y fusión,
la verificación y la continuidad.
