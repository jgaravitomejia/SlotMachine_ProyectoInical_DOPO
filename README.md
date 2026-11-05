# Prueba de aceptación — Proyecto SlotMachine

**Integrantes:** Juan Sebastián Garavito Mejía · Laila Zareth Romano Guerrero
**Entrega:** GaravitoM-RomanoG

Simulador de una máquina tragamonedas inspirado en el *Problem I* de la maratón de programación internacional 2025 (**Slot Machine**). El proyecto tiene dos partes: **construir el simulador** y **resolver el problema de la maratón a ciegas**, sin poder ver los símbolos y sabiendo únicamente cuántos colores distintos se muestran.

A continuación se recorre el proyecto **ciclo por ciclo**. En cada ciclo se explica qué se pidió, qué implementamos, el paso a paso de la prueba (con el método que se usa, el parámetro que se envía y lo que se espera) y por qué se comporta así según los requisitos y decisiones de ese ciclo.

---

## Ciclo 1 — El simulador básico

### Qué se pidió
Construir el simulador: crear la máquina, **agregar/eliminar ruedas y símbolos**, **girar**, **consultar los símbolos** y **si la configuración es ganadora**, hacerla **visible/invisible** (debe funcionar también en modo invisible) y **terminar**. Los símbolos deben ser de colores distintos, la máquina debe **lucir diferente al ganar**, y si una acción no se puede realizar se le avisa al usuario.

### Qué implementamos
La clase **`SlotMachine`** (el controlador), con una lista de ruedas y una de símbolos; **`Wheel`** y **`Symbol`**, donde **cada rueda tiene sus propios símbolos** (composición) para que un mismo color pueda verse en varias ruedas a la vez. Reutilizamos el paquete `shapes` para el dibujo. Hay jackpot cuando todas las ruedas muestran el mismo símbolo; en ese caso el cuerpo de la máquina se pone **dorado**.

### Paso a paso
Crear: clic derecho en `SlotMachine` → `new SlotMachine()` → objeto **`maquina`**. Luego `maquina` → `makeVisible()`, que muestra el cuerpo gris todavía sin ruedas.

| Paso | Acción (método y parámetro) | Resultado esperado |
|---|---|---|
| 1 | `spin(1)` | Todavía no hay ruedas, así que no gira nada y el sistema lanza la excepción **IllegalStateException** con el mensaje *"No hay ruedas para girar."* |
| 2 | `delWheel(1)` | No existe ninguna rueda que eliminar, así que lanza **IllegalStateException** con *"No hay ruedas para eliminar."* |
| 3 | `addSymbol(1, "red")`, `addSymbol(2, "blue")`, `addSymbol(3, "green")` | Quedan registrados en la máquina los tres colores (red, blue y green) en ese orden |
| 4 | `addWheel(1)`, `addWheel(2)`, `addWheel(3)` | Aparecen tres ruedas y, como todas arrancan mostrando el primer símbolo (red), quedan iguales: hay jackpot y la máquina se pone **dorada** |
| 5 | `symbols()` | Devuelve la lista de colores de la máquina en orden: red, blue, green |
| 6 | `spin(2)` y luego `spin(3, 2)` | La rueda 2 pasa a blue y la rueda 3 a green; ahora las tres ruedas muestran colores distintos y la máquina vuelve a gris |
| 7 | `distinctSymbols()`, `isJackpot()`, `configuration()` | Hay 3 colores distintos, no hay jackpot (devuelve `false`) y la configuración visible es red, blue, green |
| 8 | `delSymbol("cyan")` | El color cyan no existe en la máquina, así que no borra nada y lanza **IllegalArgumentException** con *"No existe el simbolo de color cyan."* |
| 9 | `spin(new String[]{"green","green","green"})` | Las tres ruedas quedan en green: hay jackpot (la máquina se pone dorada), `isJackpot()` devuelve `true` y queda un solo color distinto |

### Por qué se comporta así
Cada rueda tiene **sus propios** símbolos, por eso al agregarlas todas muestran el primer símbolo (red) y de inmediato hay jackpot. Que haya jackpot es simplemente que todas las ruedas muestren el mismo color (un solo color distinto), y el color dorado cumple el requisito de "lucir distinto al ganar". Las operaciones imposibles (girar o eliminar sin ruedas, borrar un color que no existe) se rechazan; en el Ciclo 1 esto se avisaba con `ok()` y `JOptionPane`, y en el cierre (Ciclo 5) se cambió por **excepciones**.

---

## Ciclo 2 — Fijar, intercambiar y configurar

### Qué se pidió
Extensión del simulador: **intercambiar** dos ruedas, **fijar y soltar** una rueda, **rotar una rueda un número de pasos** y **dejar la máquina en una configuración dada**. Si la máquina es visible, `spin(wheel, steps)` debe verse paso a paso.

### Qué implementamos
`swap`, `lock`/`unlock` (con el estado `locked` en `Wheel`), `spin(wheel, steps)` y `spin(setSymbols)`. Una rueda fija no gira, y una configuración dada solo se aplica si es válida.

### Paso a paso
Se continúa sobre el objeto **`maquina`** del Ciclo 1.

| Paso | Acción (método y parámetro) | Resultado esperado |
|---|---|---|
| 1 | `spin(new String[]{"red","blue","green"})` | La máquina vuelve a quedar con los tres colores distintos |
| 2 | `lock(2)` | La rueda 2 queda fija (bloqueada) |
| 3 | `spin()` | Giran al azar las ruedas 1 y 3, pero la rueda 2 no se mueve porque está fija |
| 4 | `spin(2)` | Como la rueda 2 está fija, no gira y el sistema lanza **IllegalStateException** con *"La rueda 2 esta fija."* |
| 5 | `unlock(2)` | La rueda 2 queda libre y puede volver a girar |
| 6 | `swap(1, 3)` y luego `configuration()` | Las ruedas 1 y 3 intercambian su posición; al consultar la configuración se nota el cambio |
| 7 | `spin(1, 2)` | La rueda 1 avanza dos pasos de una sola vez |
| 8 | `spin(new String[]{"red","cyan","green"})` | La configuración incluye cyan, que no es un color de la máquina, así que no se aplica y lanza **IllegalArgumentException** con *"El color cyan no existe."* |
| 9 | `spin(new String[]{"red","blue"})` | La configuración trae dos colores pero hay tres ruedas; como no coincide, no se aplica y lanza **IllegalArgumentException** con *"La configuracion no coincide con el numero de ruedas."* |

### Por qué se comporta así
`lock` marca la rueda como fija y `SlotMachine` respeta ese estado antes de girarla; `swap` intercambia las posiciones de dos ruedas; y `spin(setSymbols)` valida la configuración (que no sea nula, que el largo coincida con el número de ruedas, que los colores existan y que ninguna rueda esté fija) **antes** de aplicarla. Si algo no cumple, no se cambia nada y se lanza la excepción correspondiente.

---

## Ciclo 3 — El solucionador de la maratón

### Qué se pidió
Crear una máquina con **igual número de ruedas y símbolos** inicializada al azar (`SlotMachine(n)`), **resolver** el problema de la maratón (`solve`) y **simular** la solución (`simulate`). La máquina es la *herramienta de prueba*: el solucionador solo puede usar `SlotMachine(n)`, `spin(wheel, steps)` y `distinctSymbols()`. Debe quedar **invisible en `solve`** y **visible en `simulate`**.

### Qué implementamos
El constructor `SlotMachine(n)`, que crea `n` símbolos distintos y `n` ruedas en posiciones al azar. La clase **`SlotMachineContest`** con `solve(n)` (resuelve a ciegas y devuelve la lista de acciones) y `simulate(n)` (muestra la solución en pantalla). El algoritmo trabaja en **tres fases**: **separar** (girar hasta que las `n` ruedas muestren símbolos distintos), **aprender** (usando la rueda 1 como referencia, averiguar cuántos pasos necesita cada rueda) y **alinear** (aplicar esos pasos de una sola vez).

### Paso a paso
Crear: clic derecho en `SlotMachineContest` → `new SlotMachineContest()` → objeto **`contest`**.

| Paso | Acción (método y parámetro) | Resultado esperado |
|---|---|---|
| 1 | *(la profesora elige un número `n` entre 3 y 50)* `contest.simulate(n)` | La máquina se arma sola con ese tamaño y, paso a paso, llega sola al jackpot (se pone dorada), sin que nosotros le indiquemos los colores |
| 2 | `contest.solve(n)` | Devuelve la lista de movimientos que hizo para ganar; se puede mostrar cuántos fueron, que siempre es menos de 10.000 |
| 3 | `new SlotMachine(51)` (o `new SlotMachine(2)`) | El problema solo admite de 3 a 50 ruedas; al pedir un tamaño fuera de ese rango la máquina no se crea y lanza **IllegalArgumentException** con *"El numero de ruedas debe estar entre 3 y 50."* |

### Por qué se comporta así
El reto es resolver **sin ver los símbolos**, con un solo dato: cuántos colores distintos hay. Una estrategia simple (mover la rueda que más suba los distintos) cae en un **mínimo local**, porque ese número no dice *a cuál* rueda te estás uniendo. Por eso usamos la **rueda 1 como referencia**: primero la dejamos en un símbolo único y luego, moviéndola un paso, identificamos qué rueda reacciona para igualarla.

> `solve` y `simulate` **ignoran** los tamaños fuera de rango (devuelven una lista vacía / no hacen nada) para que las pruebas de unidad funcionen. La **validación con excepción** del rango 3–50 está en el **constructor de la máquina** (`SlotMachine(n)`), que es el que se muestra en el paso 3.

---

## Ciclo 4 — Tipos de rueda y de símbolo (requisito no funcional)

### Qué se pidió
Manejar **distintos tipos** de rueda y de símbolo. Tres tipos de rueda: **normal**, **lefty** (si hay una rueda a su izquierda, al girar copia su estado) y **rebel** (no se deja bloquear, intercambiar ni eliminar). Tres tipos de símbolo: **normal**, **ephemeral** (en cada giro decrece su tamaño hasta quedar como un punto) y **shy** (alterna visible/invisible al ser seleccionado). Además, proponer un **tipo nuevo** propio. Requisito no funcional: los elementos de distinto tipo **deben distinguirse visualmente**.

### Qué implementamos
**Herencia**: `Wheel` es la rueda base (normal) y de ella heredan **`Lefty`** y **`Rebel`**; `Symbol` es el símbolo base (normal) y de él heredan **`Ephemeral`**, **`Shy`** y **`Growing`** (nuestro tipo propio: el opuesto de ephemeral, crece en cada selección). La creación se centraliza en las fábricas `createWheel` y `Symbol.create`. Para el requisito no funcional, **diferenciamos visualmente**: las ventanas de las ruedas por color (**normal blanca, lefty cian, rebel roja**) y los símbolos por silueta (**normal cuadrado, ephemeral cuadro pequeño, shy barra plana, growing barra alta**).

### Paso a paso
Crear una máquina limpia: `new SlotMachine()` → **`maquinaTipos`** → `makeVisible()`.

| Paso | Acción (método y parámetro) | Resultado esperado |
|---|---|---|
| 1 | `addSymbol("normal",1,"red")`, `addSymbol("ephemeral",2,"blue")`, `addSymbol("shy",3,"green")`, `addSymbol("growing",4,"gold")` | Quedan registrados cuatro símbolos, uno de cada tipo |
| 2 | `addWheel("normal",1)`, `addWheel("lefty",2)`, `addWheel("rebel",3)` | Aparecen tres ruedas y cada tipo tiene un color de ventana distinto: la normal blanca, la lefty cian y la rebel roja |
| 3 | *(observar las ventanas de las ruedas)* | Se evidencia el requisito no funcional: con solo mirar se distingue de qué tipo es cada rueda |
| 4 | `placeSymbol(1,"blue")`, `placeSymbol(2,"green")`, `placeSymbol(3,"gold")` | Cada rueda muestra un símbolo con una forma distinta: el ephemeral un cuadro pequeño, el shy una barra plana y el growing una barra alta |
| 5 | `placeSymbol(1,"red")` y luego `spin(2)` | Al girar la rueda lefty, esta copia el símbolo de la rueda que tiene a su izquierda (la rueda 1) y queda igual que ella |
| 6 | `lock(3)` | La rueda rebel no se deja bloquear: el sistema lanza **IllegalStateException** con *"La rueda 3 es rebelde: no se puede bloquear."* |
| 7 | `delWheel(3)` y luego `swap(1,3)` | La rueda rebel tampoco se deja eliminar ni intercambiar: en ambos casos lanza **IllegalStateException** explicando que es rebelde |
| 8 | poner una rueda en el símbolo ephemeral (o growing) y `spin(rueda, 4)` varias veces | Cada vez que el giro vuelve a caer en ese símbolo, el ephemeral se ve más pequeño y el growing más grande |

### Por qué se comporta así
Gracias al **polimorfismo**, `SlotMachine` trata a cualquier rueda o símbolo sin preguntar de qué tipo es. La `Rebel` redefine los permisos de bloquear, intercambiar y eliminar para negarse, y la `Lefty` redefine su giro para copiar a su vecina. Los símbolos especiales reaccionan mediante un aviso (`onSelected`) que cada subtipo implementa a su manera. La diferenciación por color de ventana y por silueta cumple el **requisito no funcional** de que los tipos se distingan a la vista.

---

## Ciclo 5 — Cierre: robustez y lecciones aprendidas

En este ciclo robustecimos el simulador y lo cerramos. Reemplazamos el manejo de errores por **excepciones de Java**: `IllegalArgumentException` para datos inválidos (número de ruedas fuera de 3–50, color inexistente, configuración incorrecta) e `IllegalStateException` para operaciones imposibles (máquina sin ruedas, rueda fija, rueda rebelde, tope de 50 ruedas). Son las excepciones que aparecen en todos los escenarios fallidos de arriba. A continuación, las dificultades que tuvimos y cómo las resolvimos.

### Resolver la maratón a ciegas
El mayor reto fue el solucionador. Nuestra primera estrategia —mover la rueda que más subiera el número de símbolos distintos— **fallaba**: con solo ese dato no se sabe a cuál rueda te estás uniendo, así que el algoritmo caía en un **mínimo local** (por ejemplo, con la mitad de las ruedas de un color y la mitad de otro, ningún movimiento de una sola rueda mejoraba). La medimos, entendimos por qué fallaba y, en vez de entregar algo que no servía, buscamos una solución correcta: usar la **rueda 1 como referencia**, llevarla a un símbolo único y, moviéndola, identificar qué rueda reacciona para igualarla. Anotamos todas las rotaciones y las aplicamos al final.

### Por qué con muchas ruedas tardaba tanto
Al probar con el máximo de 50 ruedas, `simulate` tardaba **más de una hora** y parecía que nunca llegaba al jackpot. Al medirlo descubrimos algo clave: el **algoritmo en sí corría en milisegundos** (resolver una máquina de 50 ruedas toma alrededor de 26 ms y unas 2.300 acciones). El cuello de botella **no era el cálculo, sino el dibujo**: `simulate` animaba alrededor de **25.000 pasos**, y cada dibujo en pantalla tiene una pausa interna de 10 milisegundos, lo que sumaba casi media hora solo en esperas, más el repintado. Lo resolvimos **reescribiendo `simulate`**: ahora **resuelve en modo invisible** (instantáneo) y luego **anima solo el movimiento neto de cada rueda** hasta el jackpot (alrededor de 1.100 pasos en vez de 25.000). Con eso pasó de más de una hora a unos pocos segundos, mostrando igual cómo la máquina llega al jackpot.

### Diferenciar las ruedas y los símbolos (requisito no funcional)
Cumplir el requisito no funcional también nos costó. Al principio **todas las ruedas se veían iguales** (una ventana blanca) y **los símbolos también**, así que no había forma de saber a simple vista cuál era de cada tipo. El reto era diferenciarlos **sin cambiar su comportamiento** y usando solo lo que ofrece el paquete `shapes`. Terminamos decidiendo:
- **Ruedas** → por el **color de la ventana**: normal blanca, lefty cian, rebel roja.
- **Símbolos** → por la **silueta**: normal cuadrado, ephemeral cuadro pequeño, shy barra plana y growing barra alta.

Además nos costó que las figuras de **distinto tamaño quedaran centradas** dentro de la ventana y no pegadas a una esquina, algo que tuvimos que ajustar aparte para que la diferencia se viera bien.

### Otros arreglos del cierre
- Corregimos un error en el solucionador por el cual una máquina de **3 ruedas** (el mínimo válido) no se resolvía.
- Ajustamos el solucionador para que usara únicamente los métodos permitidos como herramienta de prueba (`SlotMachine(n)`, `spin(wheel, steps)` y `distinctSymbols()`), respetando la restricción del Ciclo 3.

### Qué aprendimos como equipo
No nos conformamos con la primera solución: varias veces lo fácil habría sido entregar lo primero que funcionara, pero preferimos devolvernos, entender el problema de fondo y hacerlo bien. También aprendimos que medir antes de concluir ahorra tiempo —el caso de `simulate` lo demuestra— y que programar y revisar en conjunto, en vez de dividirnos el proyecto por partes, nos habría evitado varias de las correcciones de esta etapa.
