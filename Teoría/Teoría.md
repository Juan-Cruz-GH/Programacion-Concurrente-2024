<center>

# Clase 1 - 6 de marzo, 2024

</center>

## Concurrencia

-   Hacer más de una cosa al mismo tiempo, simultáneamente.
-   Se encuentra tanto en la vida real como en las computadoras.

## No determinismo

-   El determinismo significa que dado una entrada, siempre obtenemos la misma salida.
-   El no determinismo, por el contrario, significa que dado una misma entrada, podemos obtener distintas salidas en cada ejecución.

## Por qué es necesaria la concurrencia?

-   Porque aprovechan mucho mejor a los procesadores multicore.
-   Porque es mucho más fácil pensar una solución concurrente para un problema concurrente. Las soluciones secuenciales para problemas concurrentes suelen ser tediosas y complejas.
-   Porque mejoran la performance incluso en procesadores monocore.
-   Porque ayuda a los casos de entradas asíncronas.

## Objetivos de los sistemas concurrentes

1. Ajustar la arquitectura al problema a resolver.
2. Aumentar la performance de los programas.

## Procesamiento Secuencial, Concurrente y Concurrente con Paralelismo

#### Secuencial

Hago una cosa atrás de otra. Hasta que no termino el paso N no puedo realizar el paso N + 1. Establece un **orden estricto temporal**.

#### Concurrente

Dedica una parte del tiempo a cada tarea, aprovechando tiempos muertos. Todo esto en **un solo core**. Básicamente compartir tiempo de CPU de un mismo core entre N tareas, por ejemplo vía **time slicing** y los **context switch**.

#### Concurrente con Paralelismo

Tenemos un multicore. Todos los cores trabajan al mismo tiempo de forma concurrente haciendo cada uno una tarea. El objetivo principal es reducir el tiempo de ejecución.

#### Concurrencia

Por ende, la concurrencia es un concepto de **software** que se puede adaptar a **CUALQUIER ARQUITECTURA**. Un programa concurrente puede ser ejecutado en un monocore o multicore.

## Programa Concurrente

-   Un programa concurrente es uno que se compone de 2 o más **procesos** que actúan al mismo tiempo para resolver un mismo problema.
-   Los procesos que forman parte de un programa concurrente suelen **interactuar** entre sí y comunicarse o sincronizarse.
-   No son determinísticos, lo cual dificulta el debugging.

#### Comportamiento de los procesos

1. **Independencia:** Pueden ser independientes entre sí y no interactuar nunca.
2. **Cooperación:** Por el contrario, pueden cooperar entre sí y comunicarse y sincronizarse para resolver una tarea común.
3. **Competencia:** Pueden competir por uno o más recursos compartidos.

Estas formas de actuar pueden combinarse de cualquier forma, en un programa concurrente.

## Procesos vs Hilos

1. **Proceso:** Pedazos de una cosa más grande. Cada proceso tiene su propio espacio de direcciones y recursos. Las bibliotecas de memoria distribuida trabajan con procesos.
2. **Proceso liviano/thread/hilo:** Sub-pedazos de un mismo proceso, que tienen su propio contador de programa y su pila de ejecución, pero comparten el mismo espacio de direcciones y recursos que su proceso padre. Las bibliotecas de memoria compartida trabajan con hilos.

## Memoria Compartida vs Distribuida

La comunicación entre procesos indica el modo en que se organizan y transmiten datos entre tareas concurrentes.

1. **Compartida:** Los procesos tienen acceso a un lugar de memoria que comparten. Para evitar pisarse, pueden bloquear o liberar posiciones de memoria de ese mismo espacio compartido. Solo se puede usar en un procesador monocore.
2. **Distribuida:** Los procesos están en procesadores separados. Se establece un **canal** lógico o físico para la transmisión de datos. Los procesos deben saber cuándo tienen mensajes para leer y cuándo deben enviar mensajes. Se puede usar tanto en un procesador monocore como en un multicore.

## Sincronización

Significa conocer información de un proceso para **coordinar** actividades. Existen 2 formas de sincronización:

1. **Por exclusión mutua:** Un proceso bloquea el acceso a un recurso hasta que él termina de utilizarlo. Se utiliza para resolver **secciones críticas**.
2. **Por condición:** Un proceso se duerme hasta que una condición es verdadera.

Un programa concurrente puede hacer uso de ambos métodos de sincronización sin problema.

## Interferencia

Un proceso hace algo que invalida las suposiciones que hizo otro proceso. Casi siempre ocurre por no respetar las secciones críticas utilizando exclusión mutua o sinc. por condición. **Debe evitarse siempre**.

## Prioridad de un proceso

Si un proceso tiene mayor prioridad que otro, puede causar la suspensión de otro proceso. También puede obligar a otro proceso a liberar un recurso para él poder usarlo.

## Granularidad de una aplicación

Relación entre el cómputo y la comunicación.

1. **Grano fino:** **Muchos** procesos que se comunican **mucho** pero realizan pocas actividades cada uno. Se adapta mucho mejor a una arquitectura con muchos cores.
2. **Grano grueso:** **Pocos** procesos que se comunican **poco** pero realizan actividades grandes cada uno. Se adapta mucho mejor a una arquitectura con pocos cores.

## Manejo de recursos compartidos

Un tema clave de la programación concurrente es la administración de recursos compartidos.

-   Incluye la asignación; métodos de acceso; bloqueo; liberación; seguridad; consistencia.
-   **Fairness:** Que todos los procesos logren acceder al R.C en forma equitativa, es decir aproximadamente la misma cantidad de tiempo cada uno.

## Deadlock

Ocurre cuando hay un bloqueo, donde 2 o más procesos se quedan esperando a que el otro libere el recurso compartido, quedandose en loop infinito.

Puede ocurrir cuando:

1. Existen secciones críticas en la solución.
2. Los procesos mantienen los recursos que ya adquirieron mientras esperan adquirir más recursos.
3. Nadie puede sacarle un recurso a un proceso excepto él mismo.
4. Existe un ciclo tal que cada proceso tiene un recurso que su sucesor está esperando acceder.

## Requerimientos de software de un lenguaje concurrente

Los lenguajes de programación concurrente deben proveer instrucciones adecuadas para especificar mecanismos de comunicación y sincronización.

## Problemas asociados a la programación concurrente

-   La necesidad de utilizar mecanismos de exclusión mutua y sincronización agregan complejidad a los programas.
-   El no determinismo dificulta el debugging.
-   El overhead de context switch puede reducir la performance, si tenemos una enorme cantidad de procesos.
-   Mayor tiempo de desarrollo y puesta a punto. Sobre todo si estamos tratando de paralelizar un algoritmo secuencial.
-   Dificultad al alinear la solución (grano fino o grueso) a la arquitectura que poseemos.

## Concurrencia a nivel de hardware

Debido a que la concurrencia utilizando un solo core está limitada a la velocidad propia de ese núcleo, surgieron los multicore, los cuales incrementan la performance drásticamente explotando la concurrencia al máximo.

#### Multicores de memoria compartida

-   Esquema UMA vs NUMA.
-   Poseen problemas de sincronización y consistencia.

#### Multicores de memoria distribuida

-   N procesadores conectados entre sí vía una red.
-   Cada uno posee su propia memoria, evitando problemas de consistencia.
-   Los procesadores interactúan entre sí solo vía pasaje de mensajes.

## Instrucciones

-   **_Declaración_**:

```
tipo variable = valor
int x = 8

tipo arreglo[tamaño] = ([tamaño] valor repetido en todas las posiciones)
int a[10] = ([10] 0) // Arreglo tamaño 10 con todas sus posiciones en 0.
```

-   **_Asignación_**:

```
a = b
a = a + 1
c = a + b
a[3] = 6

d :=: f // swappea los valores de d con los de f.

```

-   **_skip_**: termina inmediatamente y no tiene ningún efecto sobre ninguna variable del programa.
-   **_Sentencia de alternativa múltiple_**: Se chequean toads las condiciones booleanas al msimo tiempo y de las que son verdaderas se elige **una sola de ellas en forma no determinística** y se ejecuta la serie de instrucciones asociadas a esa condición.

```
if B1 -> S1
   B2 -> S2
   Bn -> Sn
fi
```

-   **_Sentencia de alternativa ITERATIVA múltiple_**: Igual que el anterior pero sigue repetiendo la ejecución una y otra vez hasta que TODAS las condiciones sean **falsas**.

```
do B1 -> S1
   B2 -> S2
   Bn -> Sn
od
```

-   **_co_**: Espera a que el proceso creado termine antes de seguir ejecutando la siguiente sentencia luego del co.

```
co S1 // .. // Sn oc
Ejecuta Sn tareas concurrentemente. Su ejecución termina cuando todas las tareas concurrentes terminaron.

co [i = 1 to n] { a[i] = 0; b[i] = 0 } oc
Crea N tareas concurrentes
```

-   **_process_**: Ejecuta en background, es decir que no espera a que el process termine.

```
process Estudiante {
   ...
}
Proceso único, independiente.

process Estudiantes [i = 1 to N] {
   ...
}
N procesos independientes.

```

---

<center>

# Clase 2 - 13 de marzo, 2024

</center>

## Acciones atómicas

#### Estado de un programa concurrente

El estado de un PC son los valores que tienen todas las variables (locales y compartidas) cuando detenemos la ejecución en un instante de tiempo determinado.

#### Acciones atómicas

Cada sentencia (línea) puede estar formada por 1 o más acciones atómicas.

Una acción atómica es una acción que no puede descomponerse en acciones más pequeñas, y por ende no se puede interrumpir o interferir por otros procesos.

-   De grano **fino**: las "reales", es decir las de hardware.
-   De grano **grueso**: simuladas vía mutex.

Por ejemplo, A = B (asignar una variable a otra) NO es atómica, pero A = 1 si lo es.

Load PosMemA, registro // es atómica de grano fino.

#### Historias de un programa concurrente

Secuencia de acciones atómicas que se van ejecutando en cada proceso. Algunas historias serán válidas y otras no, ya que romperían la lógica o semántica del problema a resolver. Hay que evitar, mediante restricciones, las historias no válidas.

#### Tiempos e intuición

Nunca asumir nada sobre los tiempos de ejecución y no confiar en la intuición cuando analizamos programas concurrentes.

#### Referencia crítica

Significa leer el valor de una variable que es modificada por otro proceso.

#### Propiedad "a lo sumo una vez"

Una asignación a = b satisface esta propiedad si cumple cualquiera de estas 2 condiciones:

1. **b** contiene A LO SUMO una referencia crítica y **a** no es referenciada por otro proceso.
2. **b** no contiene ninguna referencia crítica, en cuyo caso **a** puede ser leída por otro proceso.

-   En resumen, puede haber a lo sumo una variable compartida y puede ser referenciada a lo sumo una vez.
-   Si una asignación cumple la propriedad ASV entonces su ejecución simula ser atómica.
-   Si una asignación NO cumple la propiedad ASV entonces es necesario ejecutarla atómicamente.

## Sincronización

#### Sentencia <> y await

La sincronización por exclusión mutua construye una acción atómica de grano grueso.

```
< Sentencia/s >
```

-   **Await para exclusión mutua (incondicional).** Significa que todas las instrucciones que se encuentran dentro de estos dos símbolos se ejecuta de forma atómica.

```
< await (condición) Sentencia/s >
```

-   **Await general (condicional).** Significa que mientras la condición sea falsa el proceso espera/hace busy waiting (NO SE DUERME). Cuando es verdadera ejecuta en forma atómica las sentencias.

```
< await (condición) >
```

-   **Await para sincronización por condición (condicional).** Significa que mientras la condición sea falsa el proceso espera/hace busy waiting (NO SE DUERME). Cuando es verdadera sale del await.

## Propiedades y fairness

Una propiedad de un PC es un atributo que siempre es verdadero en todas las historias del PC. Será verdadero sin importar cuantas veces ejecute el programa.

#### Propiedades de seguridad vs de vida

-   **Seguridad:** Que no haya errores o cosas raras.

    -   Nada malo le ocurre a un proceso.
    -   Corrección parcial -> Siempre que el programa termina, termina "bien", es decir da un resultado correcto. No asegura que el programa termine en sí.

-   **Vida:** Que ningun proceso se atasque.
    -   Ausencia de deadlock -> ningun proceso se atasca para siempre.
    -   Terminación -> el programa siempre terminará. No asegura que el resultado final sea correcto o no.

Si se cumplen ambas propiedades, tenemos **total correctness**.

#### Acciones atómicas elegibles

Una acción atómica es elegible si es la próxima acción atómica en el proceso que será ejecutada. Si tenemos varios procesos tenemos varias acciones atómicas elegibles, y la política de scheduling decidirá cuál de todas ellas será la próxima en ejecutarse.

#### Fairness y políticas de scheduling

-   **Fairness:** Intenta garantizar que todos los procesos tengan chance de avanzar.
-   **Política de scheduling:** Según la configuración del sistema operativo, se elegirá como siguiente a ejecutar una acción atómica de uno u otro proceso.

#### Tipos de fairness

1. **Incondicional:** Toda acción atómica incondicional elegible eventualmente se ejecuta.
   Round Robin utiliza este fairness.
2. **Débil:** Es incondicional y además toda acción atómica condicional elegible eventualmente se ejecuta, asumiendo que su condición sea true y permanece true hasta que es vista por el proceso que ejecuta la acción atómica condicional.
3. **Fuerte:** Es incondicional y además toda acción atómica condicional eventualmente se ejecuta, ya que su guarda se convierte en true con frecuencia infinita.
   No se puede implementar.

---

<center>

# Clase 3 - 20 de marzo, 2024

</center>

-   [Link a la clase](https://drive.google.com/file/d/1rvUdra_2C5l11c2yCAn61pszhDuRUyqL/view?usp=sharing)

## Problema de la Sección Crítica

-   Se busca que un pedazo de código se ejecute de forma atómica de forma real, sin usar pseudocódigo como el await o < >.
-   Necesitamos un protocolo de entrada y uno de salida, es decir debemos implementar el "<" y el ">".
-   Es decir: bloqueo -> sección crítica -> liberación.

#### Propiedades que la Sección Crítica debe cumplir

1. A lo sumo un proceso está en su sección crítica.
2. Si 2 o más procesos intentan entrar a la sección crítica, si o si uno de ellos tendrá éxito (ausencia de deadlock).
3. Si un proceso intenta entrar a la sección crítica y todos los demás ya terminaron o están ejecutando la sección no crítica, entonces ese proceso debe poder entrar inmediatamente a su SC, sin demora (ausencia de demora innecesaria).
4. Si un proceso intenta entrar a la sección crítica eventualmente lo hará (ausencia de inanición).

#### Traducción de sentencias await a while loops

-   Las 3 sentencias await se pueden traducir a código real una vez que obtengamos los protocolos de entrada a la sección crítica:

```
< Sentencia/s >

se traduce a:

Protocolo de entrada;
Sentencia/s;
Protocolo de salida;
```

```
< await (condición) Sentencia/s >

se traduce a:

Protocolo de entrada;
while (not condición) {
   Protocolo de salida;
   Protocolo de entrada;
}
Sentencia/s;
Protocolo de salida;
```

```
< await (condición) >

se traduce a:

while (not condición) skip
```

#### Solución de "grano grueso"

-   Se trata de una solución (aún ficticia ya que usa await, pseudocódigo) que permite evitar usar los "< >" y usar solo el await, para lograr mutex en una sección crítica.
-   Para 2 procesos:

```cs
bool lock = false;

process SC1 {
   while true {
      < await (not lock) lock = true; >
      // sección crítica;
      lock = false;
      // sección no crítica
   }
}

process SC2 {
   while true {
      < await (not lock) lock = true; >
      // sección crítica;
      lock = false;
      // sección no crítica
   }
}
```

-   Para N procesos:

```cs
bool lock = false;

process SC [id: 1..N] {
   while true {
      < await (not lock) lock = true; >
      // sección crítica;
      lock = false;
      // sección no crítica
   }
}
```

#### Solución de grano fino 1: Spin Locks

-   Se trata de una solución **no fair** que busca reemplazar el await por instrucciones reales de máquina.
-   Utiliza la instrucción de máquina Test and Set (TS) que funciona de la siguiente manera:

```cs
func bool TS(bool ok) {
   < bool inicial = ok;
   ok = true;
   return inicial; >
}
```

-   Spin Locks para N procesos:

```cs
bool lock = false;

process SC [id: 1..N] {
   while true {
      while (TS(lock)) skip;
      // sección crítica;
      lock = false;
      // sección no crítica
   }
}
```

-   Se puede mejorar un poco la performance usando la instrucción Test and Test and Set, la cual no escribe en la variable lock si el valor no ha cambiado, pero aún así sigue sin ser ideal.

#### Solución de grano fino 2: Tie-Breaker

-   Se trata de una solución **fair**, que intenta balancear el acceso a la sección crítica entre muchos procesos, para evitar inanición.
-   Se vuelve extremadamente complejo en la versión de N procesos.
-   Tie-Breaker para 2 procesos:

```cs
bool in1 = false, in2 = false;
int ultimo = 1;

process SC1 {
   while true {
      in1 = true;
      ultimo = 1;
      while (in2 and ultimo == 1) skip;
      // sección crítica
      in1 = false;
      // sección no crítica
   }
}
process SC2 {
   while true {
      in2 = true;
      ultimo = 2;
      while (in1 and ultimo == 2) skip;
      // sección crítica
      in2 = false;
      // sección no crítica
   }
}
```

-   Tie-Breaker para N procesos. Demasiado complejo:

```cs
int in[1..N] = ([N] 0);
int ultimo[1..N] = ([N] 0);

process SC [i: 1..N] {
   while true {
      for j = 1 to N {
         in[i] = j;
         ultimo[j] = i;
         for k = 1 to N st i <> k {
            // espera si el proceso kj está en una etapa más alta y el proceso i fue el último en entrar a la etapa j
            while (in[k] >= in[i] and ultimo[j] == i) skip;
         }
      }
      // sección crítica;
      in[i] = 0;
      // sección no crítica;
   }
}
```

#### Solución de grano fino 3: Ticket

-   Otra solución **fair** pero más simple y eficiente.
-   Los procesos toman un número mayor que el del último proceso que sacó un ticket, y esperan a ser atendidos por orden de este ticket.
-   Podría sufrir de integer overflow eventualmente.
-   Usa el Fetch-and-Add (FA) que funciona de la siguiente manera:

```cs
func FA(int var, int incremento) {
   < temp = var;
   var = var + incremento;
   return temp; >
}
```

-   Ticket para N procesos:

```cs
int numero = 1;
int proximo = 1;
int turno[1..N] = ([N] 0);

process SC [i: 1..N] {
   while true {
      turno[i] = FA(numero, 1);
      while (turno[i] <> proximo) skip;
      // sección crítica
      proximo = proximo + 1;
      // sección no crítica
   }
}
```

#### Solución de grano fino 4: Bakery

-   Otra solución **fair** un poco más compleja.
-   Es teórica, no implementable.
-   Bakery para N procesos:

```cs
int turno[1..N] = ([N] 0);

process SC [i: 1..N] {
   while true {
      turno[i] = 1; // comienza el protocolo de entrada
      turno[i] = max(turno) + 1;
      for j = 1 to N st j != i
         while (turno[j] != 0) and ( (turno[i], i) > (turno[j], j)) skip;
      // sección crítica
      turno[i] = 0;
      // sección no crítica
   }
}
```

## Sincronización Barrier

Una barrera es un punto de **demora** al cual TODOS los procesos deben llegar y una vez que ésto ocurre se les permite pasar a todos a la vez.

#### Método 1: Contador compartido

-   Cada proceso incrementa una variable compartida al llegar.
-   Cuando esa VC == cantidad de procesos, entonces los procesos pueden pasar.
-   Poco práctico cuando necesitamos más de una barrera en un mismo programa concurrente.
-   Se puede implementar de forma grano fino con Fetch-and-Add:

```cs
int cantidad = 0;
process worker[id: 1..N] {
   while true {
      FA(cantidad, 1);
      while (cantidad <> N) skip;
   }
}
```

#### Método 2: Flags y coordinadores

-   Tenemos un arreglo de N posiciones donde cada posición tendrá 0 o 1 dependiendo si ese proceso ya llegó o no. Cuando la suma de todas las posiciones de ese arreglo == N, ya llegaron todos y podemos levantar la barrera.
-   Requiere un proceso extra Coordinador.
-   Permite utilizar tantas barreras como queramos en un mismo programa concurrente.
-   Reintroduce contención de memoria y es ineficiente.
-   Flags y coordinadores:

```cs
int arribo[1..N] = ([N] 0);
int continuar[1..N] = ([N] 0);

process worker[id: 1..N] {
   while true {
      arribo[i] = 1;
      while (continuar[i] == 0) skip;
      continuar[i] = 0;
   }
}
process Coordinador {
   while true {
      for i = 1 to N {
         while (arribo[i] == 0) skip;
         arribo[i] = 0;
      }
      for i = 1 to N {
         continuar[i] = 1;
      }
   }
}
```

#### Método 3: Árboles (combining tree barrier)

-   Requiere un proceso y procesador extra.
-   Se complejiza la solución ya que los procesos pueden jugar distintos roles (hoja, raíz, nodo interno), aunque es eficiente.
-   Los procesos van avisando que llegaron, desde las hojas hasta la raíz, y cuando el proceso raiz recibe las señales de ambos de sus hijos significa que ya llegaron todos, y entonces envía hacia abajo, desde la raíz hasta las hojas, que ya pueden continuar porque la barrera se completó.

#### Método 4: Barreras Simétricas

-   Se basa en que dos procesos se sincronizen entre si, y luego estos 2 sincronizen con otros 2, y luego los 4 con otros 4, y 8 con otros 8, etc etc hasta completar la barrera entera en O(log (N)) en vez de O(N).
-   Barrera Simétrica para N procesos:

```cs
int E = log(N);
int arribo[1..N] = ([N] 0)

process worker[id: 1..N] {
   int j;
   while true {
      // inicio de la barrera
      for etapa = 1 to E {
         j = (i-1) XOR (1 << (etapa-1));
         while (arribo[i] == 1) skip;
         arribo[i] = 1;
         while (arribo[j] == 0) skip;
         arribo[j] = 0;
      }
      // fin de la barrera
   }
}
```

## Defectos de la sincronización por busy waiting

-   Los protocolos de tipo busy waiting o spin locks (sinónimos) no son ideales:
    -   Son complejos.
    -   No separan claramente las variables de sincronización de las variables de cómputos.
    -   Son dificiles de verificar y corregir.
    -   Son ineficientes si se los utiliza en multiprogramación, ya que un procesador ejecutando un proceso **spinning** podría ser usado de forma más productiva por otro proceso.
-   Es por todo esto que necesitamos herramientas más potentes para diseñar protocolos de sincronización: **Semáforos y monitores**.

---

<center>

# Clase 4 - 27 de marzo, 2024

</center>

## Semáforos

Se trata de un **tipo de datos abstracto** (un objeto) que acepta solo 2 operaciones/métodos: P() y V() que se ejecutan de forma **atómica**.

-   Internamente posee un atributo integer no negativo.
-   La operación P() duerme al proceso si ese atributo es 0. Si no, lo decrementa en 1. Por eso esta operación es **bloqueante**.
-   La operación V() siempre incrementa el atributo en 1.
-   Los semáforos permiten proteger secciones críticas vía mutex y además permiten implementar sincronización por condición.

#### Operaciones en detalle

-   Si o si se inicializan en su declaración:

```cs
sem mutex = 1; // Correcto
sem mutex; // Incorrecto
sem mutex[N] = ( [N] 1 ) // Semáforos privados
```

-   Operaciones P y V:

```cs
P(sem) {
   < await (sem > 0); sem-- >
}

V(sem) {
   < sem++ >
}
```

#### Sección crítica usando semáforos

Es tan simple como hacer:

```cs
sem mutex = 1
...
P(mutex)
// sección crítica
V(mutex)
// sección no crítica
```

-   Si mutex fuese inicializado en 0 todos los procesos se atascarían en el P() y habría deadlock.
-   Si mutex fuese inicializado en 2 por ejemplo, eso significa que 2 procesos podrían estar en la sección crítica a la vez.

#### Barreras

-   Inicializando un semáforo en 0, podemos crear barreras:
    -   El P() simboliza que el proceso 1 está esperando (se duerme) a que un evento ocurra.
    -   El V() simboliza que el proceso 2 señala que un evento ocurrió y ahora el proceso 1 puede seguir.

#### Semáforos binarios divididos

Un semáforo binario es uno que solo almacena internamente un 1 o un 0, a diferencia de un semáforo general el cual puede almacenar 0, 1, 2, y siguiendo.

Un semáforo binario dividido ocurre cuando, dado N semáforos binarios, la suma de todos ellos da 0 o 1.

Esta técnica soluciona por ejemplo el problema de N productores y M consumidores en un buffer compartido.

#### Contadores de recursos

Cada semáforo cuenta la cantidad de unidades libres de un recurso compartido. Esto es útil cuando los procesos compiten por recursos de múltiples unidades.

Es decir, tenemos sem libres = N, y cuando hacemos P() simbolizamos que ahora hay una unidad libre **menos** en ese recurso compartido, y cuando hacemos V() simbolizaos que ahora hay una unidad libre **más** en ese recurso compartido.

Si tenemos N consumidores y M productores necesitamos un semáforo mutex para cada uno, ya que hay que proteger la próxima posición en el arreglo compartido para que no se pise el valor.

#### Exclusión mutua selectiva

Se da entre procesos que compiten por el acceso a **conjuntos superpuestos de variables compartidas**. Es decir, N procesos adquieren un recurso cada uno, pero necesitan también un segundo recurso que está siendo utilizado por otro proceso. Se produce deadlock.

Se puede solucionar diviendo los N procesos entre los N-1 primeros y luego el último separados.

#### Técnica Passing The Baton

Se trata de una técnica general para implementar el **await**.

Cuando un proceso está dentro de la sección crítica mantiene el baton que simboliza su permiso para ejecutar.

Cuando el proceso debe salir de la sección crítica pasa el baton (el control) a otro proceso. **Si ninguno está esperando por el baton (ningun proceso está esperando entrar a la sección crítica) entonces éste se libera para que lo tome el próximo proceso que trata de entrar**.

Esta técnica se utiliza mucho cuando necesitamos cierto orden entre los procesos para utilizar un recurso compartido.

#### Alocación de recursos y scheduling

Hay que decidir cuándo se le puede dar acceso a un recurso compartido a un proceso determinado.

Un recurso es cualquier objeto, elemento, componente, dato, sección crítica, por la que un proceso puede ser demorado esperando adquirirlo.

Tendríamos dos operaciones:

-   request(parámetros)
-   release(parámetros)

Puede usarse vía Passing The Baton:

```cs
request() {
   P(e);
   if request no puede ser satisfecho {
      delay
   }
   // tomar unidad
   SIGNAL;
}

release() {
   P(e);
   // retornar unidad
   SIGNAL;
}
```

---

<center>

# Clase 5 - 3 de abril, 2024

</center>

## Monitores

#### Conceptos básicos

-   Los monitores son módulos estructurados que contienen **variables** que almacenan el estado del recurso compartido y **procedimientos** que implementan operaciones sobre éste.
-   Representan recursos compartidos.
-   Los procedimientos se ejecutan siempre y de forma implícita con **exclusión mutua**.
-   La sincronización por condición se realizará de forma explícita vía **variables condition**.
-   Un proceso que invoca un procedimiento puede ignorar cómo está implementado (abstracción).
-   Los procedimientos pueden recibir parámetros.
-   No existen más las variables compartidas/globales.
-   Cualquier tipo de sincronización o comunicación entre procesos se debe hacer a través del monitor.

#### Sintaxis

```cs
monitor NombreMonitor {
   declaraciones de variables permanentes;
   código de inicialización; // Se ejecuta una sola vez al INICIO del programa.

   procedure op1(parámetros formales) {
      cuerpo del procedure
   }

   procedure op2(parámetros formales) {
      cuerpo del procedure
   }
}

process Proceso {
   NombreMonitor.op1(dato1);
   NombreMonitor.op2(dato2)
}
```

#### Sincronización por condición

Se programa explícitamente con **variables condition** -> cond vc.

-   Las variables condition son implícitamente una cola de procesos demorados.
-   Poseen varias operaciones:

1. wait(vc): el proceso se demora **al final** de la cola y deja el acceso exclusivo al monitor
2. signal(vc): despierta al proceso **al principio** de la cola (si hay alguno) y lo quita de ella. El proceso despertado podrá recién ejecutar cuando requiera el acceso exclusivo al monitor.
3. signal_all(vc): despierta a **todos** los procesos demorados en la vc, quedando la cola totalmente vacía.

##### Sincronización Signal and Continue

El proceso que hace el **_signal_** continúa usando el monitor, y el proceso despertado pasa a **competir por acceder nuevamente** al monitor para continuar con su ejecución (en la instrucción siguiente al wait).

##### Sincronización Signal and Wait

El proceso que hace el **_signal_** pasa a competir por acceder nuevamente al monitor, mientras que el proceso despertado pasa a ejecutar dentro del monitor a partir siguiente al wait.

#### Wait vs P y Signal vs V

| Wait                              | P                                                  |
| --------------------------------- | -------------------------------------------------- |
| El proceso **siempre** se duerme. | El proceso solo se duerme **si el semáforo es 0.** |

| Signal                                                                                                   | V                                                                                                                       |
| -------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Si hay procesos dormidos despierta al **primero** de ellos. En caso contrario no tiene efecto posterior. | Incrementa el semáforo para que un proceso dormido o que hará un P continue. No sigue **ningún orden** al despertarlos. |

#### Operaciones adicionales (puramente teóricas)

-   **_empty(cv)_**: retorna true si la controla controlada por cv está vacía.
-   **_wait(cv, rank)_**: el proceso se demora en la cola de cv en orden **ascendente** de acuerdo al parámetro rank y deja acceso exclusivo al monitor.
-   **_minrank(cv)_**: retorna el minimo ranking de demora.

## Técnicas con monitores

#### Simulando un semáforo

-   Hay que usar un **while** y no un **if** para evitar que el semáforo pueda tener valor menor a 0: con un if podría pasar que cuando el semáforo vale 1, un proceso decremente el semáforo y luego se despierta a uno que estaba dormido el semáforo pasaría a valer -1. **Esto se da porque recordemos que despertar a un proceso dormido con el signal no asegura que pase a ejecutar inmediatamente, si no que pasa a competir con todos los otros procesos.**
-   La diferencia es que en esta solución hay un orden parcial, la operación V despierta al primero que se durmió, en vez de liberar el semáforo para que **cualquiera** lo tome como ocurría en los semáforos. El orden "parcial" se debe a que un proceso le gane el acceso al proceso que acaba de ser despertado. El orden solo ocurre entre los procesos que se **habían dormido**, por eso es parcial.
-   Para que esta solución sea idéntica a un semáforo se debería hacer un **_signal_all(pos)_** en vez de **_signal(pos)_**, aunque sería menos eficiente.

```cs
monitor Semaforo {
   int s = 1;
   cond pos;

   procedure P() {
      while (s == 0)
         wait(pos);
      s--;
   }

   procedure V() {
      s++;
      signal(pos);
   }
}
```

#### Passing The Condition

-   Esta solución, por más que no sigue la definición exacta de semáforo, es fair y respeta por completo el orden de llegada.

```cs
monitor PassingTheCondition {
   int s = 1;
   int dormidos = 0; // Cuenta la cantidad de procesos dormidos en la variable condición.
   cond pos;

   procedure P() {
      if (s == 0) {
         dormidos++;
         wait(pos);
      }
      else
         s--;
   }

   procedure V() {
      if (dormidos == 0)
         s++;
      else {
         dormidos--;
         signal(pos);
      }
   }
}
```

#### Alocación SJN: Wait con Prioridad

-   Similar al anterior pero se despiertan los procesos según el orden de algún atributo, en este caso ese atributo es "tiempo" pero puede ser cualquier otro.
-   Como no podemos usar el wait con prioridad en una variable condition, usamos una colaOrdenada auxiliar.

```cs
monitor ShortestJobNext {
   bool libre = true;
   cond turno[N];
   colaOrdenada espera;

   procedure request(int id, int tiempo) {
      if (libre)
         libre = false;
      else {
         espera.push(id, tiempo)
         wait(turno[id])
      }
   }

   procedure release() {
      if (empty(espera))
         libre = true;
      else {
         id = espera.pop().id
         signal(turno[id]);
      }
   }
}
```

#### Buffer limitado: Sincronización por condición básica

-   Problema típico de productores y consumidores en un buffer de tamaño N.

```cs
monitor BufferLimitado {
   typeT buf[N];
   int ocupado, libre, cantidad = 0;
   cond not_lleno, not_vacio;

   procedure depositar(typeT datos) {
      while (cantidad == n)
         wait (not_lleno);
      buf[libre] = datos;
      libre = (libre + 1) MOD N;
      cantidad++;
      signal(not_vacio);
   }

   procedure retirar(typeT resultado) {
      while (cantidad == 0)
         wait(not_vacio);
      resultado = buf[ocupado];
      ocupado = (ocupado + 1) MOD N;
      cantidad--;
      signal(not_lleno);
   }
}
```

#### Lectores y Escritores: Broadcast Signal

-   Cuando se dan las condiciones todos los procesos compiten y uno de ellos entra sin ningún tipo de orden.

```cs
monitor ControladorRW {
   int cantLectores, cantEscritores = 0;
   cond ok_leer, ok_escribir;

   procedure pedido_leer( ) {
      while (cantEscritores > 0)
         wait (ok_leer);
      cantLectores++;
   }

   procedure libera_leer( ) {
      cantLectores--;
      if (cantLectores == 0)
         signal (ok_escribir);
   }

   procedure pedido_escribir( ) {
      while (cantLectores > 0) OR (cantEscritores > 0)
         wait (ok_escribir);
      cantEscritores++;
   }

   procedure libera_escribir( ) {
      cantEscritores--;
      signal (ok_escribir);
      signal_all (ok_leer);
   }
}
```

#### Lectores y Escritores: Passing The Condition

-   Mayor control de a quién dejamos pasar una vez la DB está libre.

```cs
monitor ControladorRW {
   int cantLectores = 0, cantEscritores = 0, lectoresDormidos = 0, escritoresDormidos = 0;
   cond ok_leer, ok_escribir;

   procedure pedido_leer() {
      if (cantEscritores > 0) {
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

#### Reloj lógico: Covering Condition

-   Timer que permite a los procesos dormirse por una cantidad determinada de tiempo.
-   Asumimos que **_peek_** retorna un número muy grande si la cola está vacía.

```cs
monitor Timer {
   int horaActual = 0;
   cond espera[N];
   colaOrdenada dormidos;

   procedure demorar(int intervalo, int id) {
      int horaDespertar = horaActual + intervalo;
      dormidos.push(id, horaDespertar)
      wait(espera[id])
   }

   procedure tick() {
      horaActual++;
      int aux = dormidos.peek().horaDespertar
      while (aux <= horaActual) {
         idAux = dormidos.pop().id
         signal(espera[idAux])
         aux = dormidos.peek().horaDespertar
      }
   }
}
```

#### Peluquero dormilón: Rendezvous

-   Serie de sincronizaciones mutuas -> Rendezvous.
-   corteDePelo -> Llamado por los clientes que retornan luego de recibir un corte de pelo.
-   proximoCliente -> Llamado por el peluquero para esperar que un cliente se siente en su silla y luego le corta el pelo.
-   corteTerminado -> Llamado por el peluquero para que el cliente abandone la peluquería.

```cs
monitor Peluqueria {
   int peluquero, silla, abierto = 0;
   cond peluqueroDisponible, sillaOcupada, puertaAbierta, salioCliente;

   procedure corteDePelo() {
      while (peluquero == 0)
         wait (peluqueroDisponible);
      peluquero--;
      signal (sillaOcupada);
      wait (puertaAbierta);
      signal (salioCliente);
   }

   procedure proximoCliente() {
      peluquero++;
      signal(peluqueroDisponible);
      wait(sillaOcupada);
   }

   procedure corteTerminado() {
      signal(puertaAbierta);
      wait(salioCliente);
   }
}
```

---

<center>

# Clase 6 - 10 de abril, 2024

</center>

## Pthreads

-   Se trata de una biblioteca del lenguaje C para programación paralela en **memoria compartida**. Sin embargo, hay muchas libraries muy similares en otros lenguajes.
-   Con esta biblioteca se pueden crear threads, asignarles atributos, darlos por terminados, identificarlos, etc.
-   Un thread es un proceso "liviano" que tiene su propio Program Counter y su propia pila de ejecución, pero no controla el "contexto pesado" por ejemplo las tablas de página.

#### Creación y terminación

-   Inicialmente se tiene un solo proceso, el programa principal, que va creando hilos paralelos.
-   1 hilo -> 1 función.
-   El programa principal debe esperar a que todos los threads terminen para poder terminar.
-   Pthreads provee 4 funciones básicas para especificar concurrencia.

```c
#include <pthread.h>

// Genera o crea un hilo nuevo. Cada hilo tiene un manejador, atributos, la función que va a ejecutar y por último todos los parámetros que necesita esa función.
int pthread_create(pthread_t *thread_handle, const pthread_attr_t *attribute, void * (*thread_function)(void *), void *arg);

// Un hilo indica que ya terminó y devuelve algún resultado. Se llama al final de la función del create.
int pthread_exit(void *res);

// Esperar a que el hilo que pasamos haya terminado su ejecución.
int pthread_join(pthread_t thread, void **ptr);

// Solicitar que se cancele un hilo.
int pthread_cancel(pthread_t thread);

```

#### Mutex

-   Las secciones críticas en Pthreads se implementan usando mutex locks vía variables mutex.
-   Estas variables mutex tienen 2 estados, locked y unlocked.
-   El único thread que puede desbloquear el mutex es **el mismo que lo bloqueó**.
-   Para entrar a la SC, el thread debe lograr bloquear el mutex, y cuando sale de la SC debe desbloquearlo.
-   Todos los mutex se inicializan como desbloqueados.

```c
// Bloquea la variable mutex pasada como parámetro.
int pthread_mutex_lock(pthread_mutex_t *mutex);

// Desbloquea la variable mutex pasada como parámetro.
int pthread_mutex_unlock(pthread_mutex_t *mutex);

// Inicializa la variable mutex para que se pueda empezar a usar. Se le pasan una serie de atributos (tipo de mutex por ej).
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *lock_attr);
```

#### Tipos de mutex

Uno de los atributos de los mutex que pueden settearse en la inicialización es su **tipo**, de los cuales hay 3:

1. **Mutex Normal**: Si un thread intenta bloquear el mismo mutex que ya había bloqueado antes se puede producir **deadlock**.
2. **Mutex Recursive**: Permite que un mutex sea bloqueado muchas veces seguidas sin problema, pero eventualmente se deberán realizar **la misma cantidad de unlocks** que la cantidad de veces que se bloqueó para evitar deadlock.
3. **Mutex Error Check**: Es como el mutex normal pero en vez de producir deadlock **responde con un reporte de error** cuando el mismo thread intenta bloquear por segunda vez el mutex.

#### Overhead de bloqueos

-   Como ya hemos visto antes, si dentro de los locks ponemos demasiadas operaciones y/o instrucciones que tomen mucho tiempo, se degrada significativamente la performance.
-   Para esto, se puede reducir esta pérdida usando una función:

```c
// Si la variable mutex está desbloqueada, la bloquea. Si está bloqueada, el proceso no se duerme y puede seguir trabajando en otras cosas.
int pthread_mutex_trylock(pthread_mutex_t *mutex_lock);
```

#### Variables condición

-   Se pueden usar variables condición para que un thread se auto-bloquee hasta que el programa alcance un estado determinado.
-   Cada variable condición estará asociada a un valor booleano. Cuando éste se vuelve **true**, los threads pueden avanzar.
-   La condición puede ser compleja (varias condiciones juntas) aunque dificulta el debugging.
-   Una VC siempre tiene un mutex asociada a ella.
-   Si la VC es falsa, el thread espera (no usa CPU).

```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex)

int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime)

int pthread_cond_signal(pthread_cond_t *cond)

int pthread_cond_broadcast(pthread_cond_t *cond)

int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr)

int pthread_cond_destroy(pthread_cond_t *cond)
```

#### Atributos y sincronización

-   Cada entidad (thread, mutex, variable condición, etc) puede tener atributos, los cuales se settean vía **objetos atributos**.
-   Los attribute objects son estructuras de datos que describen las propiedades de la entidad.
-   Una vez que las propiedades están establecidas, el AO es pasado al método init.
-   Esto facilita la modificación del código.

## Semáforos en Pthreads

-   Los threads pueden sincronizarse por semáforos importando la biblioteca **_semaphore.h_**.

```c
// Se declaran globales a los threads.
sem_t semaforo

// Se le pasa como parámetro el semáforo como tal; el alcance (compartido por hilos de más de un proceso o solo los hilos de un solo proceso); inicial es el valor inicial (0, 1, etc).
sem_init(&semaforo, alcance, inicial)

// Exactamente igual al P().
sem_wait(&semaforo)

// Exactamente igual al V().
sem_post(&semaforo)
```

## Monitores en Pthreads

-   Pthreads no posee monitores como tal, pero podemos simularlos combinando:

1. Los mutex -> para la exclusión mutua.
2. Las variables condición -> para la sincronización por condición.

-   El acceso exclusivo al monitor se simula usando una variable mutex la cual se bloquea antes de la llamada al procedure y se desbloquea al terminar la misma (una variable mutex por cada monitor).
-   Cada llamado de un proceso a un procedure debe ser reemplazado por el código de ese procedure.

---

<center>

# Clase 7 - 17 de abril, 2024

</center>

## Programación Concurrente en Memoria Distribuida

#### Conceptos generales

-   Las herramientas de Memoria Distribuida están pensadas para arquitecturas de MD.
-   Una arquitectura de memoria distribuida es una en donde se tienen clusters, es decir varios procesadores conectados vía cable donde cada uno de ellos puede ser un multicore con una memoria compartida, pero donde cada cluster en sí tiene SU propia memoria.
-   Un programa distribuido es un programa concurrente comunicado por mensajes que supone la ejecución sobre una arquitectura de MD.
-   Los procesos únicamente comparten **canales** que pueden ser físicos o lógicos. Los canales también se dividen en:
    -   **Mailbox** (cualquier proceso envía y cualquiera recibe) vs **input port** (cualquier proceso envía pero 1 solo recibe) vs **link** (1 solo proceso envía y 1 solo recibe).
    -   **Uni** (cada comunicación va en un solo sentido) vs **bidireccionales** (en una misma comunicación los datos van y vuelven).
    -   **Sincrónicos** (los 2 procesos que se quieren comunicar deben sincronizarse/esperarse mutuamente) vs **asincrónicos** (el canal actúa como una cola, no es bloqueante).

#### Características

-   Las únicas "variables" globales ahora son los canales.
-   Por esto, no se necesita ningun mecanismo de mutex.
-   Los procesos interactúan comunicándose vía canales.
-   Los canales se acceden vía primitivas de envío y recepción.
-   Mecanismos de Procesamiento Distribuido:
    -   PMA (?, ?)
    -   PMS (?, ?)
    -   RPC (bidireccional, sincrónica)
    -   Rendezvous (bidireccional, sincrónica)
-   La sincronización de la comunicación interproceso depende del patrón de interacción: Clientes/Servidores; Pares interactuantes; Productores/Consumidores.

#### Relación entre mecanismos de sincronización

-   Semáforos: Mejora respecto a Busy Waiting.
-   Monitores: Combinan mutex (implícito) con sincronización explícita.
-   Pasaje de Mensajes: Extiende semáforos con datos.
-   RPC y Rendezvous: Combina la interfaz procedural de monitores con Pasaje de Mensajes implícito.

## Pasaje de Mensajes Asincrónicos

#### Canales

-   Cada canal es una **cola** de mensajes enviados y no recibidos.
-   Los canales son asincrónicos, unidireccionales, y de tipo mailbox.

##### Declaración de canales

```cs
chan nombre(tipoDato);

// Por ejemplo
chan entrada(char);
chan canal(int a, int b, char c);
chan resultado[N](int);
```

##### Operación send

-   Deja una copia del dato con el valor que tenga en ese instante al final de la cola del canal.
-   No demora al emisor.
-   Es una operación atómica.

```cs
send canal(dato);

// Por ejemplo
send canal(1);
```

##### Operación receive

-   Toma el primer elemento de la cola y lo elimina de la misma.
-   Si el canal está vacío, el proceso se **demora** hasta que haya al menos un elemento, por ende es una operación bloqueante.
-   Las variables en el receive deben ser del mismo tipo que el tipo de dato del canal en su declaración.
-   Es una operación atómica.

```cs
receive canal(variable);

// Por ejemplo
int miVariable;
receive canal(miVariable);
```

##### Operación empty

-   Determina si la cola del canal está vacía o no.
-   Debe usarse con cuidado:
    -   Puede ocurrir que la operación de falso, es decir, que haya al menos un elemento en la cola, pero al momento de hacer el receive ya no está ese elemento, porque otro proceso le ganó e hizo el receive antes. Esto provocaría que si la cola tenía 1 solo elemento el proceso que perdió se quede bloqueado en el receive, cosa que queríamos evitar.
    -   Puede ocurrir que la operación de verdadero, es decir, que no hay ningún elemento en la cola, y sin embargo justo después se agregue un nuevo elemento a la cola sin que el proceso llegue a darse cuenta.

```cs
empty(canal) // V o F
```

#### Clientes y Servidores (Monitores Activos)

-   Podemos simular el funcionamiento de un monitor como manejador de un recurso compartido vía PMA.
-   Tenemos N procesos Cliente y 1 proceso Servidor.
-   Los clientes envian pedidos a un canal compartido y luego realizan un receive en un canal privado de cada uno de ellos.
-   El Servidor atiende los pedidos de los clientes por orden de llegada naturalmente, ya que las colas de los canales mantienen este orden.
-   Tambien se pueden simular las variables condición de forma similar.
-   Existe una correspondencia directa entre este método de PMA y los Monitores propiamente dichos:

| Monitores                     | Monitores simulados vía PMA                   |
| ----------------------------- | --------------------------------------------- |
| Variables permanentes         | Variables locales del Servidor                |
| Identificadores de procedures | Canal request y tipos de operación            |
| Llamado a procedure           | send request(), receive respuesta()           |
| Entry del monitor             | receive request()                             |
| Retorno del procedure         | send respuesta()                              |
| Sentencia wait                | Salvar pedido pendiente                       |
| Sentencia signal              | Recuperar/procesar pedido pendiente           |
| Cuerpos de los procedure      | Sentencias del case switch según la operación |

#### Pares (peers) interactuantes

-   Se trata de un problema donde N procesos poseen un valor cada uno y todos deben conocer el máximo y el mínimo de entre todos ellos.
-   Existen 3 tipos de soluciones a este problema:
    -   **Centralizada**: Los procesos le envian su información a un proceso Servidor que calculará el min y max global y luego almacenará los resultados en un arreglo de canales resultado, donde cada posición pertenecerá a cada proceso.
    -   **Simétrica**: Hay un canal entre cada par de procesos. Todos los procesos ejecutan el mismo código y no hay proceso Servidor. Cada proceso envía su dato local a los N-1 restantes y luego recibe y procesa los N-1 datos de los demás, de forma tal que en paralelo toda la arquitectura está calculando el mínimo y el máximo y toda la arquitectura tiene acceso a los N datos.
    -   **Anillo**: El proceso P[i] recibe mensajes de P[i-1] y envía mensajes a P[i+1]. Posee 2 etapas, en la primera cada proceso recibe 2 valores y los compara con su local, enviando un min y un max a su sucesor. En la segunda todos reciben la circulación del min y max global. El primer proceso de todos debe ser ligeramente distinto a todos los demás.

---

<center>

# Clase 8 - 24 de abril, 2024

</center>

## Pasaje de Mensajes Sincrónicos

#### Conceptos básicos

-   Los canales son de tipo **link** (1 emisor 1 receptor), **sincrónicos** e **implícitos**.
-   A diferencia de PMA, la primitiva de envío de PMS es **bloqueante**:

    -   El emisor queda esperando que el mensaje sea recibido por el receptor.
    -   La cola de mensajes asociada a un canal se reduce a 1 mensaje -> **menos uso de memoria**.
    -   Naturalmente **se reduce el grado de concurrencia** respecto a PMA: los emisores se bloquean por más tiempo.
    -   Es más fácil que ocurra **deadlock**.

#### Buffer

-   Una técnica para contrarrestar en cierto grado la reducción de concurrencia en PMS es utilizar un proceso buffer.
-   Se puede usar por ej en el problema de 1 Productor y 1 Consumidor; N Productores y M Consumidores; N Clientes y 1 Servidor.

## Lenguaje de PMS: CSP

#### Conceptos básicos

-   Introduce comunicación guardada.
-   Los canales son links entre dos procesos en vez de mailbox global. Son half-duplex y nominados.
-   Las sentencias de Entrada (?) y Salida (!) son el único medio por el cual los procesos se comunican.
-   Para que se produzca la comunicación, deben matchear, y luego se ejecutan en simultáneo.
-   Los canales no se declaran, son implícitos vía **ports**.

#### Sintaxis

-   Destino y Fuente son procesos.
-   Fuente[*] significaría que cualquier proceso dentro del arreglo de procesos Fuente realiza la recepción.
-   port es una etiqueta que se usa para distinguir entre distintas clases de mensajes que un proceso podría recibir.

```cs
Destino!port(e1, ..., eN);
Fuente?port(x1, ..., xN);
Fuentes[*]?port(x1, ..., xN);
```

-   Dos procesos se comunican cuando hacen matching, es decir:

```cs
A!canal(dato);

B?canal(resultado);
```

#### Comunicación guardada

-   Como ? y ! son bloqueantes, se generan problemas si un proceso quiere comunicarse con otros sin conocer el orden en que los otros quieren hacerlo con él.
-   Las operaciones ? y ! pueden ser guardadas, es decir, hacer un **"await"** hasta que una condición sea verdadera.

```cs
Condición; Comunicación -> Sentencias;
```

-   La condición puede omitirse en cuyo caso es True.
-   Condición + Comunicación forman la **guarda**:
    -   La guarda **tiene éxito** si la condición es True y ejecutar C no causa demora.
    -   La guarda **falla** si la condición es False.
    -   La guarda **se bloquea** si la condición es True pero C no puede ejecutarse en ese instante.

#### If y Do con comunicación guardada

```cs
if
   Condición1; Comunicación1 -> Sentencias1;
   Condición2; Comunicación2 -> Sentencias2;
   CondiciónN; ComunicaciónN -> SentenciasN;
fi
```

1. Se evalúan todas las guardas al mismo tiempo.
    - Si **todas son falsas**, termina el if.
    - Si **al menos una es V**, se elige una aleatoriamente.
    - Si **algunas guardas se bloquean**, se espera hasta que alguna de ellas tenga éxito.
2. Luego de elegir una guarda exitosa, se ejecuta la sentencia de comunicación de esa guarda.
3. Se ejecutan las sentencias de esa guarda.

El do es igual excepto que se sigue iterando hasta que **todas las condiciones sean falsas**.

<center>

# Clase 9 - 15 de mayo, 2024

</center>

## RPC y Rendezvous

#### Conceptos básicos

-   La comunicación es **bidireccional**.
-   RPC es parecido a monitores.
-   En Rendezvous tenemos un montón de procesos que cada tanto deben interactuar.

## RPC (Remote Procedure Call)

-   Los programas se componen de módulos que pueden estar distribuidos en distintas máquinas o no. Cada módulo puede tener muchos procesos y procedures.
-   Los procesos de un módulo pueden compartir variables y llamar a procedures de ese módulo.
-   Un proceso en un módulo puede comunicarse con procesos de otro módulo sólo invocando procedimientos exportados por éste.

#### Sincronización en módulos

...

## Rendezvous

#### Conceptos generales

-   Combina comunicación y sincronización.
-   Un proceso cliente invoca una operación mediante un call, pero esta operación es servida por un proceso existente en lugar de por uno nuevo.
-   Un proceso servidor usa una sentencia de entry para esperar por un call.
-   Las operaciones se atienden de a una por vez.
    ...

## ADA - Lenguaje con Rendezvous

#### Conceptos generales

-   Tenemos un único programa llamado Procedure.
-   Un programa ADA tiene tareas (tasks) que se pueden ejecutar independientemente y contienen primitivas de sincronización
-   Los puntos de entrada a una tarea se llaman Entrys.
-   Una tarea puede aceptar la comunicación con otro proceso vía la instrucción Accept.
-   Se puede declarar un Type Task y luego crear instancias de procesos (tareas) identificadas con dicho type.
    ...

---

<center>

# Clase 10 - 22 de mayo, 2024

</center>

---

<center>

# Clase 11 - 29 de mayo, 2024

</center>
