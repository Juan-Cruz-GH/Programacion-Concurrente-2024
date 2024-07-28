<h1 align="center">Cuestionario teorías 1 y 2</h1>

### 1. Mencione al menos 3 ejemplos donde pueda encontrarse concurrencia

Puede encontrarse concurrencia en el cuerpo humano, en los aviones, en los autos, en los servidores web, etc.

### 2. Escriba una definición de concurrencia. Diferencie procesamiento secuencial, concurrente y paralelo

La concurrencia significa realizar varias tareas al mismo tiempo, simultáneamente.

El procesamiento secuencial realiza una tarea a la vez. Hasta que no termina la tarea N no se realiza la tarea N + 1.

El procesamiento concurrente involucra compartir tiempo de CPU de un solo core entre N tareas.

El procesamiento paralelo procesa N tareas de forma 100% simultánea, cada una en un core distinto.

### 3. Describa el concepto de deadlock y qué condiciones deben darse para que ocurra.

El deadlock ocurre cuando hay un bloqueo donde 2 o más procesos esperan que el opuesto libere un recurso compartido, quedandose en un loop.

Puede ocurrir cuando:

1. Hay secciones críticas.
2. Los procesos mantienen los recursos que adquirieron mientras esperan adquirir más recursos.
3. Nadie puede sacarle un recurso a un proceso excepto él mismo.
4. Cada proceso tiene un recurso que su sucesor en el cilo está esperando adquirir.

### 4. Defina inanición. Ejemplifique.

La inanición ocurre cuando un proceso no llega nunca a ejecutarse ya que todos los demás procesos le ganan en la competencia por CPU.

Puede ocurrir por ejemplo cuando existen prioridades entre los procesos. Podría pasar que lleguen constantemente procesos de prioridad alta que pidan CPU haciendo que los de prioridad baja no puedan obtener CPU.

### 5. ¿Qué entiende por no determinismo? ¿Cómo se aplica este concepto a la ejecución concurrente?

El no determinismo significa que dada una entrada, pueden haber múltiples salidas, y no se puede predecir cuál será exactamente la salida en una ejecución determinada.

Este concepto se aplica a la ejecución concurrente ya que el orden en que se ejecutan los procesos concurrentes es no determinístico.

### 6. Defina comunicación. Explique los mecanismos de comunicación que conozca.

La comunicación entre procesos indica el modo en que se los procesos concurrentes organizan y transmiten datos entre ellos.

Existen 2 mecanismos, por Memoria Compartida y por Memoria Distribuida.

En la Compartida, todos los procesos comparten un único lugar de memoria. Para evitar interferencia, bloquean las posiciones de memoria y luego las liberan cuando terminaron de usarlas. Este mecanismo se utiliza en procesadores MonoCore.

En la Distribuida, cada core puede ejecutar varios procesos de forma concurrente (no paralela) y cada core se ejecuta de forma paralela, por lo tanto cierta parte (a veces por completo) de los procesos se ejecuta de forma paralela. Se establece un canal lógico o físico para que los cores se comuniquen entre sí.

### 7.

##### a) Defina sincronización. Explique los mecanismos de sincronización que conozca.

La sincronización ocurre cuando 2 o más procesos coordinan actividades entre ellos. Existen 2 mecanismos de sincronización:

1. Exclusión mutua: Un proceso bloquea el acceso a un recurso, lo utiliza, y luego lo libera.
2. Por condición: Un proceso se duerme hasta que una condición se torna verdadera.

##### b) ¿En un programa concurrente pueden estar presentes más de un mecanismo de sincronización? En caso afirmativo, ejemplifique

Un programa concurrente puede hacer uso de ambos mecanismos de sincronización sin problema. Por ejemplo, sincronización por mutex para proteger una variable compartida, y luego sincronización por condición para dormirse hasta que otro proceso haya hecho determinada tarea.

### 8. ¿Qué significa el problema de “interferencia” en programación concurrente? ¿Cómo puede evitarse?

La interferencia ocurre cuando un proceso hace algo que invalida las suposiciones que hace otro proceso. Puede evitarse vía sincronización por mutex o por condición.

### 9. ¿En qué consiste la propiedad de “A lo sumo una vez” y qué efecto tiene sobre las sentencias de un programa concurrente? De ejemplos de sentencias que cumplan y de sentencias que no cumplan con ASV.

Una sentencia a = b cumple la propiedad A lo suma una vez si:

-   b contiene como máximo una referencia crítica y a no es referenciada.
-   b no contiene ninguna referencia crítica.

El efecto que tiene esta propiedad es que si una sentencia a = b cumple la propiedad entonces su ejecución simula ser atómica y si no la cumple entonces DEBE ser ejecutada atómicamente.

Ejemplo de sentencias que cumplen la propiedad:

```cs
int a = 5;
int b = 2;
process A {
    a = 7;  // ✅ Cumple la propiedad ya que la parte izquierda de la asignación no es referenciada.
}
process B {
    b = 9;  // ✅ Cumple la propiedad ya que la parte izquierda de la asignación no es referenciada.
}
```

Ejemplo de sentencias que no cumplen la propiedad:

```cs
int a = 5;
int b = 2;
process A {
    a = b + 1;  // ❌ No cumple la propiedad ya que la parte izquierda de la asignación es referenciada.
}
process B {
    b = a + 1;  // ❌ No cumple la propiedad ya que la parte izquierda de la asignación es referenciada.
}
```

### 10. Dado el siguiente programa concurrente:

```cs
x = 2;
y = 4;
z = 3;
process P1 {
    x = y - z;
}
process P2 {
    z = x * 2;
}
process P3 {
    y = y - 1;
}

```

##### a) ¿Cuáles de las asignaciones cumplen con ASV? Justifique claramente.

-   La asignación en P1 no cumple la propiedad ya que X es referenciada en P2.
-   La asignación en P2 no cumple ya que Z es referenciada en P1.
-   La asignación en P3 no cumple ya que Y es referenciada en P1.

##### b) Indique los resultados posibles de la ejecución. Nota 1: las instrucciones NO SON atómicas. Nota 2: no es necesario que liste TODOS los resultados, pero si los que sean representativos de las diferentes situaciones que pueden darse.

1. Si se ejecutan en orden (P1, P2, P3):
    - x = 1
    - z = 2
    - y = 3
2. Si se ejecutan en el orden opuesto (P3, P2, P1):
    - y = 3
    - z = 4
    - x = -1
3. Si se ejecutan en el orden (P2, P1, P3):
    - z = 4
    - x = 0
    - y = 3

### 11. Defina acciones atómicas condicionales e incondicionales. Ejemplifique.

Las acciones atómicas son condicionales cuando solo se ejecutan una vez se cumple una condición.

Las acciones atómicas son incondicionales cuando no poseen ninguna condición, se ejecutan atómicamente siempre.

### 12. Defina propiedad de seguridad y propiedad de vida.

Una propiedad de seguridad asegura que no haya errores ni resultados extraños.

Una propiedad de vida asegura la ausencia de deadlock y que el programa eventualmente terminará.

### 13. ¿Qué es una política de scheduling? Relacione con fairness. ¿Qué tipos de fairness conoce?

Una acción atómica es elegible si es la próxima acción atómica en el proceso que será ejecutada. Si tenemos varios procesos tenemos varias acciones atómicas elegibles, y la política de scheduling decidirá cuál de todas ellas será la próxima en ejecutarse.

La política de scheduling se refiere al criterio que utiliza el sistema operativo para decidir a qué proceso darle CPU y cuándo.

Existen 3 tipos de fairness:

-   Incondicional: toda acción atómica incondicional elegible eventualmente se ejecuta.
-   Débil: Es incondicional y además toda acción atómica condicional elegible eventualmente se ejecuta, asumiendo que su condición sea true y permanece true hasta que es vista por el proceso que ejecuta la acción atómica condicional.
-   Fuerte: Es incondicional y además toda acción atómica condicional eventualmente se ejecuta, ya que su guarda se convierte en true con frecuencia infinita. No se puede implementar.

### 14. ¿Por qué las propiedades de vida dependen de la política de scheduling? ¿Cómo aplicaría el concepto de fairness al acceso a una base de datos compartida por N procesos concurrentes?

Las propiedades de vida dependen de la política de scheduling porque ésta afecta qué procesos se ejecutan y cuándo.

En el caso del acceso a una BD compartida por N procesos concurrentes aplicaría el concepto de fairness para asegurar que todos los procesos accedan eventualmente a la BD y no haya inanición.

### 15. Dado el siguiente programa concurrente, indique cuál es la respuesta correcta (justifique claramente)

```cs
int a = 1;
int b = 0;
process P1 {
    await (b == 1); a = 0;
}
process P2 {
    while (a == 1) {
        b = 1;
        b = 0;
    }
}
```

##### a) Siempre termina

##### b) Nunca termina

##### c) Puede terminar o no

Puede terminar o no, si se da el caso de que siempre se ejecuta el bloque entero dentro del while podría nunca terminarse el await (b == 1), ya que el proceso 2 pondría a la variable b en 0 antes de que el proceso 1 pueda llegar a chequear la condición de su await a tiempo, y quedarse esperando para siempre. Si en algún momento se ejecuta b = 1 y P1 llega a chequear la condición de su await antes que P2 ponga b = 0, se termina el programa.

---

<h1 align="center">Cuestionario teorías 3 y 4</h1>

### 1. ¿Qué propiedades deben garantizarse en la administración de una sección crítica en procesos concurrentes? ¿Cuáles de ellas son propiedades de seguridad y cuáles de vida? En el caso de las propiedades de seguridad, ¿cuál es en cada caso el estado “malo” que se debe evitar?

Se deben garantizar:

1. A lo sumo un proceso está en su sección crítica.
2. Si 2 o más procesos intentan entrar a la sección crítica y ésta está libre, si o si uno de ellos entra.
3. Si un proceso quiere entrar a la sección crítica y ésta está libre, debe entrar inmediatamente.
4. Todos los procesos eventualmente acceden a la sección crítica.

La propiedad 1 es de **seguridad**, ya que que si no se cumple implica que probablemente el resultado sea erróneo.
La propiedad 2 es de **vida**, ya que si no se cumple se produce deadlock y el programa podría no terminar.
La propiedad 3 es de **seguridad**, ya que si no se cumple se estarían generando demoras innecesarias.
La propiedad 4 es de **vida**, ya que si no se cumple habría inanición.

### 2. Resuelva el problema de acceso a sección crítica para N procesos usando un proceso coordinador. En este caso, cuando un proceso SC[i] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le otorgue permiso. Al terminar de ejecutar su sección crítica, el proceso SC[i] le avisa al coordinador. Desarrolle una solución de grano fino usando únicamente variables compartidas (ni semáforos ni monitores).

```cs
int siguiente = 0
bool llegue = false
bool fin = false
process SC [id: 0..N-1] {
    while (id != siguiente) skip
    llegue = true
    // Sección crítica
    fin = true
}
process Coordinador {
    for i=0 to N-1 {
        siguiente = i
        while (not llegue) skip
        llegue = false
        while (not fin) skip
        fin = false
    }
}
```

### 3. ¿Qué mejoras introducen los algoritmos Tie-breaker, Ticket o Bakery en relación a las soluciones de tipo spin-locks?

Los algoritmos Tie-breaker, Ticket y Bakery, a diferencia de los spin-locks, ofrecen **fairness** es decir una forma de evitar la inanición de los procesos que quieren acceder a la sección crítica.

### 4. Analice las soluciones para las barreras de sincronización desde el punto de vista de la complejidad de la programación y de la performance.

La barreras tipo contador compartido y flags con coordinador tienen complejidad O(N).
La barreras tipo árbol y simétrica tienen complejidad O(log(N)).

### 5. Explique gráficamente cómo funciona una butterfly barrier para 8 procesos usando variables compartidas.

![Butterfly Barrier](https://i.imgur.com/gtnHr4A.jpeg)

2 procesos se sincronizan entre sí, luego estos 2 con otros 2, luego esos 4 con otros 4, luego esos 8 con otros 8, etc.

### 6.

##### a) Explique la semántica de un semáforo

Un semáforo es una estructura abstracta que posee como único atributo interno un entero mayor o igual a 0, y que solo acepta 2 operaciones, P y V.

P duerme al proceso si el valor entero es igual a 0, y si no, lo decrementa en 1.
V incrementa el valor entero en 1.

##### b) Indique los posibles valores finales de x en el siguiente programa (justifique claramente su respuesta):

```cs
int x = 4
sem s1 = 1
sem s2 = 0
process A {
    P(s1)
    x = x * x
    V(s1)
}
process B {
    P(s2)
    P(s1)
    x = x * 3
    V(s1)
}
process C {
    P(s1)
    x = x - 2
    V(s2)
    V(s1)
}
```

1. Se ejecuta A primero. Entra a la sección crítica, x = 16. Se ejecuta C. x = 14. Se despierta B. x = 42.
2. Se ejecuta C primero. Entra a la sección crítica, x = 2. Despierta a B, B le gana a A, x = 6. Se ejecuta A, x = 36.
3. Se ejecuta C primero. Entra a la sección crítica, x = 2. Despierta a B, A le gana a B, x = 4. Se ejecuta B, x = 12.

### 7. Desarrolle utilizando semáforos una solución centralizada al problema de los filósofos, con un administrador único de los tenedores, y posiciones libres para los filósofos (es decir, cada filósofo puede comer en cualquier posición siempre que tenga los dos tenedores correspondientes).

...

### 8. Describa la técnica de Passing the Baton. ¿Cuál es su utilidad en la resolución de problemas mediante semáforos?

La técnica Passing the Baton se utiliza para respetar cierto orden al despertar a procesos dormidos.
Cuando un proceso está dentro de la sección crítica mantiene el "baton".
Cuando ese proceso sale de la SC le pasa el baton al siguiente proceso según un cierto orden (de llegada, de id, etc). Si no hay ningun proceso en la cola, entonces el baton lo toma el primer nuevo proceso que llegue.

### 9. Modifique las soluciones de Lectores-Escritores con semáforos de modo de no permitir más de 10 lectores simultáneos en la BD y además que no se admita el ingreso a más lectores cuando hay escritores esperando.

```cs
monitor ControladorRW {
   int cantLectores = 0, cantEscritores = 0, lectoresDormidos = 0, escritoresDormidos = 0;
   cond ok_leer, ok_escribir;

   procedure pedido_leer() {
      if (cantEscritores > 0) or (cantLectores == 10) or (escritoresDormidos > 0){
         lectoresDormidos++;
         wait(ok_leer);
      }
      else
         cantLectores++;
   }

   procedure libera_leer() {
      cantLectores--;
      if (cantLectores == 0) and (escritoresDormidos > 0) {
         escritoresDormidos--;
         signal(ok_escribir);
         cantEscritores++;
      }
   }

   procedure pedido_escribir() {
      if (cantLectores > 0) OR (cantEscritores > 0) {
         escritoresDormidos++;
         wait(ok_escribir);
      }
      else
         cantEscritores++;
   }

   procedure libera_escribir() {
      if (escritoresDormidos > 0) {
         escritoresDormidos--;
         signal(ok_escribir);
      }
      else {
         cantEscritores--;
         if (lectoresDormidos > 0) {
            cantLectores = lectoresDormidos;
            lectoresDormidos = 0;
            signal_all(ok_leer);
         }
      }
   }
}
```

---

<h1 align="center">Cuestionario teorías 5 y 6</h1>

### 1. Describa el funcionamiento de los monitores como herramienta de sincronización.

Los monitores son estructuras que actúan como administradores de recursos compartidos. Proveen exclusión mutua de forma implícita mediante sus procedures y pueden hacer uso de las variables condición para sincronización por condición.

### 2. ¿Qué diferencias existen entre las disciplinas de señalización “Signal and wait” y “Signal and continue”?

En Signal and Wait el proceso que hace el signal pasa a competir por acceder nuevamente al monitor, mientras que el proceso despertado pasa a ejecutar dentro del monitor a partir del wait.

En Signal and Continue el proceso que hace el signal continúa usando el monitor hasta terminar el procedure, y el proceso despertado pasa a competir por acceder nuevamente al monitor para continuar en la instrucción siguiente a donde se durmió.

### 3. ¿En qué consiste la técnica de Passing the Condition y cuál es su utilidad en la resolución de problemas con monitores? ¿Qué relación encuentra entre passing the condition y passing the baton?

La técnica Passing the Condition consiste en que los procesos se duermen o pasan a ejecutar una parte del monitor dependiendo de alguna condición interna del monitor.

Luego, similar a Passing the Baton, el proceso que está ejecutando el monitor actualmente, cuando sale, despierta al próximo proceso, el primero que se había dormido.

La relación entre estas dos técnicas es que ambas permiten respetar cierto orden de ejecución.

### 4. Desarrolle utilizando monitores una solución centralizada al problema de los filósofos, con un administrador único de los tenedores, y posiciones libres para los filósofos (es decir, cada filósofo puede comer en cualquier posición siempre que tenga los dos tenedores correspondientes).

...

### 5. Sea la siguiente solución propuesta al problema de alocación SJN:

```c
monitor SJN {
    bool libre = true;
    cond turno;
    procedure request (int tiempo) {
        if not libre
            wait(turno, tiempo);
        libre = false
    }
    procedure release() {
        libre = true
        signal(turno);
    }
}
```

##### a) Funciona correctamente con disciplina de señalización Signal and Continue?

...

##### b) Funciona correctamente con disciplina de señalización Signal and Wait?

...

### 6. Modifique la solución anterior para el caso de no contar con una instrucción wait con prioridad.

...

### 7. Modifique utilizando monitores las soluciones de Lectores-Escritores de modo de no permitir más de 10 lectores simultáneos en la BD, y además que no se admita el ingreso a más lectores cuando hay escritores esperando.

...

### 8. Resuelva con monitores el siguiente problema. Tres clases de procesos comparten el acceso a una lista enlazada: searchers, inserters y deleters. Los searchers sólo examinan la lista, y por lo tanto pueden ejecutar concurrentemente unos con otros. Los inserters agregan nuevos ítems al final de la lista; las inserciones deben ser mutuamente exclusivas para evitar insertar dos ítems casi al mismo tiempo. Sin embargo, un insert puede hacerse en paralelo con uno o más searches. Por último, los deleters remueven ítems de cualquier lugar de la lista. A lo sumo un deleter puede acceder la lista a la vez, y el borrado también debe ser mutuamente exclusivo con searches e inserciones.

...

### 9. El problema del “Puente de una sola vía” (One-Lane Bridge): autos que provienen del Norte y del Sur llegan a un puente con una sola vía. Los autos en la misma dirección pueden atravesar el puente al mismo tiempo, pero no puede haber autos en distintas direcciones sobre el puente.

...

##### a) Desarrolle una solución al problema, modelizando los autos como procesos y sincronizando con un monitor (no es necesario que la solución sea fair ni dar preferencia a ningún tipo de auto).

...

##### b) Modifique la solución para asegurar fairness (Pista: los autos podrían obtener turnos)

...

### 10. Indicar las características generales de Pthreads.

...

### 11. Explicar cómo se maneja la sincronización por exclusión mutua y por condición en Pthreads. Indicar la relación entre ambas. Explicar cómo se pueden simular los monitores en Pthreads.

...

---

<h1 align="center">Cuestionario teorías 7, 8 y 9</h1>

### 1. Defina y diferencie programa concurrente, programa distribuido y programa paralelo.

La forma más simple de diferenciar estos 3 conceptos es:

1. Un programa concurrente se ejecuta en una misma máquina y parece presentar ejecución simultánea, pero no la tiene, si no que se **simula** vía context switching aprovechando tiempos muertos.
2. Un programa distribuido ejecuta procesos en **diferentes máquinas** al mismo tiempo.
3. Un programa paralelo ejecuta procesos de forma simultánea en una **misma máquina**.

### 2. Marque al menos 2 similitudes y 2 diferencias entre los pasajes de mensajes sincrónicos y asincrónicos.

Similitudes:

1. Ambos poseen recepción de mensajes bloqueante.
2. Ambos permiten comunicación con un proceso determinado vía su ID.

Diferencias:

1. PMA declara los canales, PMS no.
2. PMA posee envío de mensajes no bloqueante mientras que en PMS esto es bloqueante.

### 3. Analice qué tipo de mecanismos de pasaje de mensajes son más adecuados para resolver problemas de tipo Cliente/Servidor, Pares que interactúan, Filtros, y Productores y Consumidores. Justifique claramente su respuesta.

-   En problemas de tipo Cliente/Servidor, es más adecuado utilizar PMA, para que los clientes puedan enviarle sus resultados al Servidor e inmediatamente puedan seguir trabajando.
-   En problemas de tipo Pares que interactúan, es más adecuado utilizar PMA, porque ??
-   En problemas de tipo Filtros, es más adecuado utilizar PMA, porque ??
-   En problemas de tipo Productores y Consumidores, es más adecuado utilizar PMS, porque ??

### 4. Indique por qué puede considerarse que existe una dualidad entre los mecanismos de monitores y pasaje de mensajes. Ejemplifique.

Con PMA se puede simular el mecanismo de un Monitor, ya que:

1. Tanto en PMA como en monitores no hay variables compartidas.
2. Tanto en PMA como en monitores los procesos realizan pedidos y esperan su resultado.
3. Tanto en PMA como en monitores, cuando se está atendiendo un pedido no se pueden atender los demás, pero si se pueden ir encolando.
4. Tanto en PMA como en monitores se puede respetar el orden de llegada de los pedidos, aunque de forma distinta.

### 5. ¿En qué consiste la comunicación guardada (introducida por CSP) y cuál es su utilidad? Describa cómo es la ejecución de sentencias de alternativa e iteración que contienen comunicaciones guardadas.

La comunicación guardada consiste en que las operaciones de envío "!" y recepción "?" pueden ser guardadas, es decir hacer un await hasta que una condición sea verdadera.

Consiste en una serie de "guardas" con formato `Condición; Comunicación -> Sentencias` donde la condición puede no estar en cuyo caso siempre es verdadera.

Cada guarda tiene 3 estados:

-   La guarda tiene éxito si la condición es verdadera y ejecutar la comunicación no causa demora.
-   La guarda falla si la condición es falsa.
-   La guarda se bloquea si la condición es verdadera pero la comunicación no puede ejecutarse en ese instante.

Tenemos dos estructuras que utilizan este mecanismo: el if y el do no determinísticos.

En estas estructuras se evalúan **todas las guardas al mismo tiempo** y si al menos una tiene condición verdadera, se elige una de ellas aleatoriamente. Si todas tienen condición falsa, se termina la ejecución. Si las guardas están bloqueadas, se espera hasta que alguna tenga éxito.

### 6. Marque similitudes y diferencias entre los mecanismos RPC y Rendezvous. Ejemplifique para la resolución de un problema a su elección.

Similitudes:

1. Ambos soportan comunicación bidireccional.

Diferencias:

1. RPC crea un nuevo proceso implícito que maneja cada llamado a una de sus funciones, mientras que Rendezvous usa el proceso existente.

### 7. Considere el problema de lectores/escritores. Desarrolle un proceso servidor para implementar el acceso a la base de datos, y muestre las interfaces de los lectores y escritores con el servidor. Los procesos deben interactuar:

##### a) con mensajes asincrónicos

##### b) con mensajes sincrónicos

##### c) con RPC

##### d) con Rendezvous

???

### 8. Modifique la solución con mensajes sincrónicos de la Criba de Eratóstenes para encontrar los números primos detallada en teoría de modo que los procesos no terminen en deadlock.

???

### 9. Suponga que N procesos poseen inicialmente cada uno un valor. Se debe calcular la suma de todos los valores y al finalizar la computación todos deben conocer dicha suma.

##### a) Analice (desde el punto de vista del número de mensajes y la performance global) las soluciones posibles con memoria distribuida para arquitecturas en estrella (centralizada), anillo circular, totalmente conectada, árbol y grilla bidimensional.

???

##### b) Escriba las soluciones para las arquitecturas mencionadas.

???

### 10. Describa sintéticamente las características de sincronización y comunicación de ADA.

En ADA tanto la sincronización como la comunicación se logra mediante Rendezvous: ejecutar la instrucción Entry (Proceso.NombreDeSuEntry) demora al llamador hasta que el receptor acepte la llamada, la procese, y devuelva el resultado.

---

<h1 align="center">Cuestionario teorías 10 y 11</h1>

### 1. Explique brevemente los 7 paradigmas de interacción entre procesos en programación distribuida vistos en teoría. En cada caso ejemplifique, indique qué tipo de comunicación por mensajes es más conveniente y qué arquitectura de hardware se ajusta mejor. Justifique sus respuestas.

1. **Master / Workers**:
    - Es la implementación distribuida del modelo Bag of Tasks. Se distribuye un trabajo entre N workers, donde el Master actúa como la bolsa de tareas.
    - Ejemplo: multiplicación de matrices dispersas.
    - Tipo de PM conveniente: PMA.
    - Arquitectura mejor: distribuida.
2. **Heartbeat**:
    - Los procesos deben, cada tanto, intercambiar información entre ellos. Esto produce una forma de barrera de sincronización entre ellos, impidiendo que se produzcan errores.
    - Ejemplo: grid computations (imágenes).
    - Tipo de PM conveniente: PMA.
    - Arquitectura mejor: ?
3. **Pipeline**:
    - El pipeline es un arreglo de procesos que actúan de filtros. La información fluye entre ellos transformándola de a poco.
    - Ejemplo: multiplicación de matrices en bloques.
    - Tipo de PM conveniente: PMA.
    - Arquitectura mejor: ?
4. **Probes and Echoes**:
    - La interacción entre procesos permite recorrer estructuras dinámicas, diseminando y recolectando información. Una primitiva Probe es un mensaje enviado por un nodo a todos sus hijos, mientras que un Echo es la respuesta a ese Probe.
    - Ejemplo: recorrer redes donde no hay un número constante de nodos activos.
    - Tipo de PM conveniente: ?
    - Arquitectura mejor: ?
5. **Broadcast**:
    - Un proceso envía un mensaje a todos los demás mediante una primitiva.
    - Ejemplo: toma de decisiones descentralizadas.
    - Tipo de PM conveniente: PMA.
    - Arquitectura mejor: distribuida
6. **Token Passing**:
    - Existe un Token que los procesos se van pasando entre sí. El proceso que tiene el Token en un momento determinado es el que tiene acceso con exclusión mutua al recurso compartido.
    - Ejemplo: se quiere hacer uso de un recurso compartido entre N procesos.
    - Tipo de PM conveniente: PMS.
    - Arquitectura mejor:
7. **Servidores Replicados**:
    - Los procesos Servidores gestionan el acceso a una instancia del recurso compartido cada uno.
    - Ejemplo: se quiere hacer uso de N recursos compartidos entre N procesos.
    - Tipo de PM conveniente: ?
    - Arquitectura mejor: ?

### 2. Describa el paradigma “bag of tasks”. ¿Cuáles son las principales ventajas del mismo?

El paradigma bag of tasks consiste en que se tiene una bolsa de tareas, y los procesos van quitando tareas de la bolsa y resolviendolas. Posiblemente agregan tareas nuevas a la bolsa.

Las principales ventajas son la escalabilidad y facilidad de equilibrar la carga.

### 3. Suponga n2 procesos organizados en forma de grilla cuadrada. Cada proceso puede comunicarse solo con los vecinos izquierdo, derecho, de arriba y de abajo (los procesos de las esquinas tienen solo 2 vecinos, y los otros en los bordes de la grilla tienen 3 vecinos). Cada proceso tiene inicialmente un valor local v.

##### a) Escriba un algoritmo heartbeat que calcule el máximo y el mínimo de los n2 valores. Al terminar el programa, cada proceso debe conocer ambos valores. (Nota: no es necesario que el algoritmo esté optimizado).

```cs
?
```

##### b) Analice la solución desde el punto de vista del número de mensajes.

?

##### c) Puede realizar alguna mejora para reducir el número de mensajes?

?

##### d) Modifique la solución de a) para el caso en que los procesos pueden comunicarse también con sus vecinos en las diagonales.

?

### 4. Explicar la clasifican de las comunicaciones punto a punto en las librerías de Pasaje de Mensajes en general: bloqueante y no bloqueante, con y sin buffering.

-   **Bloqueante con buffering**: Es el PMA. Copia el valor de la variable en el buffer y luego sigue su ejecución.
-   **Bloqueante sin buffering**: Es el PMS. Se bloquea hasta que el receptor haga la recepción.
-   **No bloqueante con buffering**: El dato está inseguro desde que se ejecuta el send y se copia el valor en el buffer (poco tiempo).
-   **No bloqueante sin buffering**: El dato está inseguro hasta que el receptor haga el receive, es decir, como el emisor sigue ejecutando, puede cambiar y manipular la variable que envió en el send una o varias veces antes que el receptor haga el receive.

### 5. Describa sintéticamente las características de sincronización y comunicación de MPI. Explicar por qué son tan eficientes las comunicaciones colectivas en MPI.

MPI soporta las barreras como método de sincronización.

MPI soporta las 4 alternativas de comunicación explicadas en el punto anterior.

Las comunicaciones colectivas de MPI son muy eficientes porque explotan el paralelismo.

### 6. ¿Qué relación encuentra entre el paralelismo recursivo y la estrategia de “dividir y conquistar”? ¿Cómo aplicaría este concepto a un problema de ordenación de un arreglo?

El paralelismo recursivo es un paradigma muy apto para la estrategia de dividir y conquistar, ya que consiste en asignar un proceso nuevo a cada llamada recursiva, dividiendo el problema naturalmente en los procesos. Esto se podría adaptar a la ordenación de un arreglo de la siguiente manera:

1. Fase de dividir:
    - Se divide el arreglo en dos mitades y se asigna una a cada uno de 2 procesos nuevos.
    - Esto se repitehasta que solo queda 1 numero en cada proceso.
2. Fase de conquistar:
    - Los procesos devuelven a su padre el array ordenado, y este a su vez ordena los resultados de sus hijos hasta llegar al proceso original.

### 7.

##### a) Cómo puede influir la topología de conexión de los procesadores en el diseño de aplicaciones concurrentes/paralelas/distribuidas? Ejemplifique.

?

##### b) Qué relación existe entre la granularidad de la arquitectura y la de las aplicaciones?

En una arquitectura de grano fino existen muchos procesadores pero con poca capacidad de procesamiento por lo que son mejores para programas que requieran mucha comunicación entre tareas o procesos que requieren poco procesamiento (programa de grano fino).

Por otro lado, en una arquitectura de grano grueso se tienen pocos procesadores con mucha capacidad de procesamiento por lo que son más adecuados para programas de grano grueso en los cuales se tienen pocos procesos que realizan mucho procesamiento y requieren de menos comunicación.

Es decir, la granularidad de las aplicaciones debería estar en sintonía con la granularidad de la arquitectura con la que se está trabajando.

### 8.

##### a) ¿Cuál es el objetivo de la programación paralela?

El principal objetivo de la programación paralela es la mejora de la performance en términos de tiempo de ejecución, al realizar varias tareas al mismo tiempo en vez de una por una.

##### b) ¿Cuál es el significado de las métricas de speedup y eficiencia? ¿Cuáles son los rangos de valores en cada caso?

El Speedup representa cuántas veces más rápida es la solución paralela comparada a la secuencial. Posee un rango de valores posibles entre 0 y el Speedup óptimo.

-   Si es < 1, la solución paralela es más lenta a la secuencial.
-   Si es = 1, la solución paralela es igual que la secuencial (en performance).
-   Si es > 1, la solución paralela es mejor que la secuencial.

El Speedup óptimo es la mayor aceleración que se podría encontrar en la arquitectura que tenemos. Significa tener en cuenta la potencia del mejor procesador de la arquitectura para así saber la potencia relativa de los demás procesadores. Este valor estará entre 0 y 1. El Speedup óptimo es la suma de estas potencias relativas.

La eficiencia indica que tan cerca estamos del Speedup óptimo y por ende representa qué tan bien se está aprovechando la arquitectura.

##### c) ¿En qué consiste la “ley de Amdahl”?

La Ley de Amdahl postula que para todo algoritmo existe un speedup máximo alcanzable (límite), independiente del número de procesadores. Ese valor dependerá de la cantidad de código paralelizable. Si el 80% de un programa es paralelizable:

Limite = 1/(1 − Paralelizable) = 1/(1 − 0.8) = 1/0.2 = 5

##### d) Suponga que la solución a un problema es paralelizada sobre p procesadores de dos maneras diferentes. En un caso, el speedup (S) está regido por la función S=p-1 y en el otro por la función S=p/2. ¿Cuál de las dos soluciones se comportará más eficientemente al crecer la cantidad de procesadores? Justifique claramente.

1. Caso 1:
    - Speedup = P - 1
    - P = 2 → Speedup = 1
    - P = 4 → Speedup = 3
    - P = 8 → Speedup = 7
    - P = 16 → Speedup = 15
2. Caso 2:
    - Speedup = P / 2 → 15.
    - P = 2 → Speedup = 1
    - P = 4 → Speedup = 2
    - P = 8 → Speedup = 4
    - P = 16 → Speedup = 8

Podemos ver que el Speedup del caso 2 va empeorando más y más comparado al del caso 1 cuando va aumentando la cantidad de procesadores. Esto nos dice que la solución 1 será más eficiente cuando ocurre eso.

##### e) Suponga que el tiempo de ejecución de un algoritmo secuencial es de 10000 unidades de tiempo, de las cuales sólo el 90% corresponde a código paralelizable. ¿Cuál es el límite en la mejora que puede obtenerse paralelizando el algoritmo? Justifique.

Limite = 1/(1 − Paralelizable) = 1/(1 − 0.9) = 1/0.1 = 10

El límite es 10. Paralelizando, el algoritmo puede ser como máximo 10 veces más rápido.

### 9.

```cs
process worker [ w = 1 to P] {
    int primera = (w-1)*(n/P) + 1;
    int ultima = primera + (n/P) - 1;
    for [i = primera to ultima] {
        for [j = 1 to n] {
            c[i,j] = 0;
            for [k = 1 to n]
                c[i,j] = c[i,j] + (a[i,k]*b[k,j]);
        }
    }
}
```

##### a) Analizando el código de multiplicación de matrices en paralelo planteado en la teoría, y suponiendo que N=256 y P=8, indique cuántas asignaciones, cuántas sumas y cuántos productos realiza cada proceso. ¿Cuál sería la cantidad para cada operación en la solución secuencial realizada por un único proceso?

En la solución secuencial (P = 1):

-   256/1 = 256. El único proceso calcula 256 filas.
-   El primer bucle se ejecuta 256 veces.
-   El segundo bucle se ejecuta 256 veces.
-   El tercer bucle se ejecuta 256 veces.
-   **Asignaciones**:
    -   (256 \* 256 \* 256) → Sale del código de asignación c[i,j] = c[i,j] + (a[i,k]\*b[k,j]);
    -   (256 \* 256) → Sale del código de asignación c[i,j] = 0;
    -   (256 \* 256 \* 256) + (256 \* 256) = 16_842_752
-   **Sumas**: (256 \* 256 \* 256) = 16_777_216
-   **Productos**: (256 \* 256 \* 256) = 16_777_216

En la solución paralela:

-   256/8 = 32. Cada proceso calcula 32 filas.
-   El primer bucle se ejecuta 32 veces para cada proceso.
-   El segundo bucle se ejecuta 256 veces para cada proceso.
-   El tercer bucle se ejecuta 256 veces para cada proceso.
-   **Asignaciones**: (32 \* 256 \* 256) + (32 \* 256) = 2_105_344
-   **Sumas**: (32 \* 256 \* 256) = 2_097_152
-   **Productos**: (32 \* 256 \* 256) = 2_097_152

##### b) Si los procesadores P1 a P7 son iguales, y sus tiempos de asignación son 1, de suma 2 y de producto 3, y si el procesador P8 es 3 veces más lento, ¿cuánto tarda el proceso total concurrente? ¿Cuál es el valor del speedup? ¿Cómo podría modificar el código para mejorar el speedup?

P1 a P7 tienen tiempo de asignación de 1, de suma 2 y de producto 3.
P8 es el triple de lento.

**El tiempo para el tipo de procesador P1 a P7 sería**:

Asignaciones: 2_105_344 \* 1 = 2_105_344
Sumas: 2_097_152 \* 2 = 4_194_304
Productos: 2_097_152 \* 3 = 6_291_456

(2_105_344 + 4_194_304 + 6_291_456) = 12_591_104 unidades de tiempo

**El tiempo para P8 sería**:

(2_105_344 + 4_194_304 + 6_291_456) \* 3 = 37_773_312

El tiempo total está determinado por el procesador más lento = **37_773_312**

El valor del speedup sería el tiempo si el programa fuera secuencial dividido el tiempo paralelo. El tiempo paralelo ya lo tenemos.

Tiempo secuencial: (16_842_752 \* 1) + (16_777_216 \* 2) + (16_777_216 \* 3) = 100_728_832

Speedup = 100_728_832 / 37_773_312 = **2.666**

El Speedup se podría mejorar ...

##### c) Calcule la Eficiencia que se obtiene en el ejemplo original y en el modificado.

...
