<center>

# Variables compartidas

</center>

## Sintaxis

<table>
  <thead>
    <tr>
      <th>Instrucción</th>
      <th>Semántica</th>
    </tr>
  </thead>
<tr><td>

```cs
< Sentencia/s >
```

</td><td>

El proceso ejecuta la/s sentencia/s en forma **atómica**. Nos brinda sincronización por **exclusión mutua**.

<tr><td>

```cs
< await (condición) Sentencia/s >
```

</td><td>

El proceso se demora (busy waiting) hasta que la condición sea verdadera y en ese momento ejecuta la/s sentencia/s en forma **atómica**. Nos brinda sincronización por **condición y exclusión mutua**.

<tr><td>

```cs
< await (condición) >
```

</td><td>

El proceso se demora hasta que la condición sea verdadera. Nos brinda sincronización por **condición**.

</table>

## Notas importantes

#### Acceso a la sección crítica

-   Una vez que la sección crítica se libera, **cualquiera** de los demás procesos que estaban esperando a acceder a esa sección crítica tomará control de la misma vía exclusión mutua. No hay ningún tipo de orden. No se puede saber cuál proceso tomará la sección crítica.

```cs
process Proceso [id: 1..N] {
    < Tarea() > // Cualquier proceso accede primero a la sección crítica. Una vez se libera, cualquier otro pasa.
}
```

#### Eficiencia

-   Siempre tratar de usar el < > lo menos posible, ya que reduce la concurrencia. No deberíamos incluir nada que no sea totalmente necesario dentro de los < >.
-   Las asignaciones o modificaciones a variables locales de cada proceso no se protegen, ni tampoco las posiciones de un arreglo compartido si cada posición pertenece a cada proceso.
-   Por ejemplo, en vez de incrementar una variable compartida en forma atómica dentro de un for podemos hacerlo al final de las iteraciones, utilizando, en los procesos, variables locales contadoras:

<table>
  <thead>
    <tr>
      <th>❌ Código ineficiente</th>
      <th>✅ Código eficiente</th>
    </tr>
  </thead>

<tr><td>

```cs
int contador;
process Proceso [id: 1..N] {
    for i = 0 to 10_000 {
        < contador++ >
    }
}
```

</td><td>

```cs
int contador;
process Proceso [id: 1..N] {
    int contadorLocal
    for i = 0 to 10_000 {
        contadorLocal++
    }
    < contador += contadorLocal >
}
```

</td></tr>
</table>

#### Orden de llegada

-   Si queremos que los procesos accedan al recurso compartido según el orden de llegada, necesitamos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento:

```cs
Cola cola; int siguiente = -1;
process Persona [id: 1..N] {
    while true {
        < if siguiente == -1
            siguiente = id
          else
            cola.push(id) >
        < await (siguiente == id) >
        // Usa el recurso
        < if cola.empty()
            siguiente = -1
          else
            siguiente = cola.pop() >
    }
}
```

#### Orden de algun atributo (prioridad)

-   Si queremos que los procesos accedan al recurso compartido según el orden de uno de sus atributos, necesitamos una ColaOrdenada compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento:

```cs
ColaOrdenada cola; int siguiente = -1;
process Persona [id: 1..N] {
    int edad = GetEdad()
    while true {
        < if siguiente == -1
            siguiente = id
          else
            cola.push(id, edad) > // Inserta ordenado por edad descendientemente
        < await (siguiente == id) >
        // Usa el recurso compartido
        < if cola.empty()
            siguiente = -1
          else
            siguiente = cola.pop() >
    }
}
```

#### Orden de identificador (ID)

-   Si queremos que los procesos accedan al recurso compartido según el orden de sus IDs, necesitamos una variable compartida inicializada en 0 que se irá incrementando de a 1 simbolizando los turnos de los procesos. Los demás procesos esperan a que siguiente sea su ID:

```cs
int siguiente = 1
process Proceso [id: 1..N] {
    < await (siguiente == id) >
    // Usa el recurso compartido
    siguiente++
}
```

#### Cuándo está libre el recurso compartido?

-   Que la cola de procesos esperando para usar un recurso compartido este vacía no nos asegura que el recurso compartido no esté siendo utilizado actualmente. Podría haber un último proceso usando el recurso.

---

<center>

# Semáforos

</center>

## Sintaxis

<table>
  <thead>
    <tr>
      <th>Instrucción</th>
      <th>Semántica</th>
    </tr>
  </thead>
<tr><td>

```cs
sem semaforo = 0
```

</td><td>

Un semáforo inicializado en 0 significa que todos los procesos se quedaran esperando ahí hasta que otro proceso haga un V().

<tr><td>

```cs
sem semaforo = N
```

</td><td>

Un semáforo inicializado en N significa que N proceso/s podrá/n estar en la sección crítica a la vez.

<tr><td>

```cs
sem semaforo[N] = 0
```

</td><td>

Arreglo de semáforos inicializados en 0, para que cada proceso de un arreglo de procesos se quede esperando. También llamados semáforos privados.

<tr><td>

```cs
P(semaforo)
```

</td><td>

Si semáforo == 0 -> se queda esperando. Si no -> semáforo--

<tr><td>

```cs
V(semaforo)
```

</td><td>

semáforo++

</table>

## Notas importantes

#### Acceso a la sección crítica

-   Una vez que la sección crítica se libera, **cualquiera** de los demás procesos que estaban esperando a acceder a esa sección crítica tomará control de la misma vía exclusión mutua. No hay ningún tipo de orden. No se puede saber cuál proceso tomará la sección crítica.

```cs
sem mutex = 1
process Proceso [id: 1..N] {
    P(mutex)
    // Cualquier proceso accede primero a la sección crítica. Una vez se libera, cualquier otro pasa.
    V(mutex)
}
```

#### Eficiencia

-   No deberíamos incluir nada que no sea totalmente necesario dentro de los semáforos mutex ya que reduce la concurrencia.
-   Las asignaciones o modificaciones a variables locales de cada proceso no se protegen, ni tampoco las posiciones de un arreglo compartido si cada posición pertenece a cada proceso.
-   Por ejemplo, en vez de incrementar una variable compartida en forma atómica dentro de un for podemos hacerlo al final de las iteraciones, utilizando, en los procesos, variables locales contadoras:

<table>
  <thead>
    <tr>
      <th>❌ Código ineficiente</th>
      <th>✅ Código eficiente</th>
    </tr>
  </thead>
<tr><td>

```cs
int contador;
sem mutex = 1;
process Proceso [id: 1..N] {
    for i = 0 to 10_000 {
        P(mutex)
        contador++
        V(mutex)
    }
}
```

</td><td>

```cs
int contador;
sem mutex 1;
process Proceso [id: 1..N] {
    int contadorLocal
    for i = 0 to 10_000 {
        contadorLocal++
    }
    P(mutex)
    contador += contadorLocal
    V(mutex)
}
```

</td></tr>
</table>

#### Usar un mutex para cosas relacionadas

-   Cosas que estén relacionadas entre sí deben usar un mismo mutex, no 2 mutex distintos. Análogamente, no tiene sentido usar el mismo semáforo para 2 secciones críticas que manipulan variables no relacionadas.

<table>
  <thead>
    <tr>
      <th>❌</th>
      <th>✅</th>
    </tr>
  </thead>
<tr><td>

```cs
Cola cola(50);
sem mutex1 = 1;
sem mutex2 = 1;
sem tamañoCola = 50;
process Proceso [id: 1..N] {
    P(tamañoCola)
    P(mutex1)
    int elemento = cola.pop()
    V(mutex1)
    // hace algo con la variable elemento
    P(mutex2)
    cola.push(elemento)
    V(mutex2)
    V(tamañoCola)
}
```

</td><td>

```cs
Cola cola(50);
sem mutex = 1;
sem tamañoCola = 50;
process Proceso [id: 1..N] {
    P(tamañoCola)
    P(mutex)
    int elemento = cola.pop()
    V(mutex)
    // hace algo con la variable elemento
    P(mutex)
    cola.push(elemento)
    V(mutex)
    V(tamañoCola)
}
```

</td></tr>
</table>

#### Semáforos inicializados en 0

-   Hay que tener mucho cuidado al inicializar semáforos en 0. Se puede producir deadlock fácilmente si no lo hacemos bien.

```cs
??
```

#### Barrera de un uso

-   Se van durmiendo los procesos y una vez llegaron todos el último los despierta a todos "a la vez".

```cs
sem mutex = 1;
sem barrera = 0;

process Proceso [id: 0..N-1] {
    P(mutex)
    contador++;
    if contador == N
        for i to N
            V(barrera)
    V(mutex)
    P(barrera)
}
```

#### Barrera de un uso vía Coordinador

-   Se van durmiendo los procesos y una vez llegaron todos el Coordinador los despierta a todos "a la vez".

```cs
sem espera = 0;
sem barrera = 0;
// Procesos
process Proceso [id: 0..N-1] {
    V(espera)
    P(barrera)
}

process Coordinador {
    for i = 0 to N-1 {
        P(espera)
    }
    for i = 0 to N-1 {
        V(barrera)
    }
}
```

#### Límite con 2 tipos de procesos

-   Cuando tenemos 2 tipos de procesos distintos y queremos que solo X cantidad del primer tipo de proceso esté en la sección crítica y solo Y del segundo tipo, y solo Z en total, tenemos que declarar los 3 semáforos y con los P y V ir de lo particular a lo general.

```cs
sem tipo1 = X
sem tipo2 = Y
sem general = Z
process Tipo1 [id: 0..N-1] {
    P(tipo1)
    P(general)
    // SC
    V(general)
    V(tipo1)
}
process Tipo2 [id: 0..N-1] {
    P(tipo2)
    P(general)
    // SC
    V(general)
    V(tipo2)
}
```

#### Semáforos como contadores de recursos disponibles

-   Cuando se tienen N recursos compartidos (un arreglo o cola de N posiciones por ejemplo) se utiliza:

    -   Un semáforo que se inicializa en el mismo valor del tamaño del recurso compartido (por ejemplo, si tenemos una cola compartida que inicialmente tiene 0 elementos, el semáforo se inicializa en 0)
    -   P() para simbolizar que se va a usar uno de esos recursos y por ende ahora hay 1 menos, además de el proceso dormirse cuando hay 0 recursos disponibles.
    -   V() para simbolizar que se libera uno de los recursos compartidos y ahora hay 1 más disponible.

```cs
int elementos[N]
sem preparado = 0
sem cantidadDisponibles = N
process Productor {
    int posicion = 0
    while true {
        P(cantidadDisponibles)
        elementos[posicion] = Producir()
        posicion = (posicion++) MOD N
        V(preparado)
    }
}
process Consumidor {
    int posicion = 0
    while true {
        P(preparado)
        int elemento = elementos[posicion]
        posicion = (posicion++) MOD N
        V(cantidadDisponibles)
    }
}
```

#### Orden de llegada

-   Si queremos que los procesos accedan al recurso compartido según el orden de llegada, necesitamos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento:

```cs

Cola cola
bool libre = true
sem mutex = 1
sem dormidos[N] = ([N] 0)
process Proceso [id: 0..N-1] {
    P(mutex)
    if libre {
        libre = false
        V(mutex)
    }
    else {
        cola.push(id)
        V(mutex)
        P(dormidos[id])
    }
    // Usa el recurso compartido
    P(mutex)
    if cola.empty()
        libre = true
    else
        V(dormidos[cola.pop()])
    V(mutex)
}
```

#### Orden de algun atributo (prioridad)

-   Si queremos que los procesos accedan al recurso compartido según el orden de uno de sus atributos, necesitamos una ColaOrdenada compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento:

```cs

ColaOrdenada cola
bool libre = true
sem mutex = 1
sem dormidos[N] = ([N] 0)
process Proceso [id: 0..N-1] {
    int edad = GetEdad()
    P(mutex)
    if libre {
        libre = false
        V(mutex)
    }
    else {
        cola.push(id, edad) // Inserta ordenado ascendientemente por edad
        V(mutex)
        P(dormidos[id])
    }
    // Usa el recurso compartido
    P(mutex)
    if cola.empty()
        libre = true
    else
        V(dormidos[cola.pop().id])
    V(mutex)
}
```

#### Orden de identificador (ID)

-   Si queremos que los procesos accedan al recurso compartido según el orden de sus IDs, necesitamos semáforos privados:

```cs
sem esperando[N] = ([N] 0)
process Proceso [id: 0..N-1] {
    if id != 0  // El primero pasa, el resto se duerme y cada uno va despertando al siguiente.
        P(esperando[id])
    // Usa el recurso compartido
    if id < N - 1
        V(esperando[id + 1])
}
```

#### Orden de llegada con Coordinador

-   Si queremos que los procesos accedan al recurso compartido según el orden de llegada mediante un Coordinador, necesitamos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento y luego el Coordinador los va despertando:

```cs
sem esperando[N] = ([N] 0)
sem mutex = 1
sem pedidos = 0
sem termine = 0
Cola cola
process Proceso [id: 0..N-1] {
    P(mutex)
    cola.push(id)
    V(mutex)
    V(pedidos)
    P(esperando[id])
    // Usa el recurso compartido
    V(termine)

}
process Coordinador {
    int siguiente
    for i = 0 to N-1 {
        P(pedidos)
        P(mutex)
        siguiente = cola.pop()
        V(mutex)
        V(esperando[siguiente])
        P(termine)
    }
}
```

#### Orden de llegada con Coordinador y más de 1 recurso compartido

-   Si queremos que los procesos accedan a los recursos compartidos según el orden de llegada mediante un Coordinador, necesitamos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento y luego el Coordinador los va despertando:

```cs
sem esperando[N] = ([N] 0)
sem cantidadRecursos = M
sem mutexProcesos = 1
sem mutexRecursos = 1
sem pedidos = 0
sem termine = 0
Cola cola
Cola recursos(M)
int miRecurso[M]
process Proceso [id: 0..N-1] {
    P(mutexProcesos)
    cola.push(id)
    V(mutexProcesos)
    V(pedidos)
    P(esperando[id])
    recurso = miRecurso[id]
    // Sección crítica
    P(mutexRecursos)
    recursos.push(recurso)
    V(mutexRecursos)
    V(termine)
    V(cantidadRecursos)
}
process Coordinador {
    int siguiente, recurso
    P(pedidos)
    P(mutexProcesos)
    siguiente = cola.pop()
    V(mutexProcesos)
    P(cantidadRecursos)
    P(mutexRecursos)
    recurso = recursos.pop()
    V(mutexRecursos)
    miRecurso[siguiente] = recurso
    V(esperando[siguiente])
}
```

#### N procesos consumen de una cola de recursos compartidos

-   Tenemos N procesos que tienen que consumir una cola de recursos lo más rápido posible.

```cs

sem mutex = 1
cola recursos
process Proceso [id: 0..N-1] {
    Recurso recurso
    P(mutex)
    while cant < M {
        cant++
        recurso = recursos.pop()
        V(mutex)
        // Usa el recurso compartido
        P(mutex)
    }
    V(mutex)
}
```

#### Passing The Condition vs Passing The Baton

-   En el método Passing The Baton, el control del semáforo es transferido del proceso que lo tiene actualmente, al siguiente. Suele usarse con semáforos binarios.
-   En el método Passing The Condition, los procesos se señalizan utilizando variables booleanas compartidas. Suele usarse con semáforos contadores.

#### Cuándo está libre el recurso compartido?

Para que el recurso compartido esté **libre** se deben cumplir dos condiciones simultáneamente:

1. Que la cola esté vacía.
2. Que nadie esté usando el recurso en este momento.

---

<center>

# Monitores

</center>

## Estructura y comunicación

-   No hay más variables globales/compartidas.
-   Las variables del monitor solo son visibles al monitor mismo.
-   Si el monitor tiene código de inicialización, no atiende ningún pedido hasta que ese código termine.
-   Los procesos interactúan entre ellos y con el/los recursos compartidos por medio de los procedures del monitor.
-   En un procedure de un monitor A se puede llamar a un procedure de otro monitor B. El monitor A no podrá continuar su ejecución ni atender otros pedidos hasta que el procedure de monitor B termine.
-   Cuando el monitor se libera, **todos los procesos** que están haciendo llamados a los procedures compiten por acceder. No se respeta el orden de llegada.

```cs
Monitor Nombre {
    Variables permanentes del monitor;

    {
        Inicialización de variables;
    }

    Procedure uno() {
        Variables locales del procedimiento;
    }
}

process P[id: 0..N-1] {
    nombre.uno();
}
```

## Sincronización

-   La exclusión mutua es implícita dentro de un monitor ya que no puede ejecutar más de un llamado a un procedimiento a la vez. Hasta que no termine el procedure o se duerma al proceso en una VC no se libera el monitor para atender otro llamado.
-   La operación **wait** duerme al proceso al final de la cola de la VC.
-   La operación **signal** despierta al primer proceso en la cola de la VC y éste pasa a competir nuevamente por acceder al monitor (no necesariamente pasa primero) y cuando lo logra continúa ejecutando en la instrucción siguiente a donde se durmió (wait).
-   La operación **signal_all** despierta a todos los procesos dormidos en la cola de la VC y todos ellos pasan a competir por el acceso.

```cs
cond vc;

wait(vc);

signal(vc);

signal_all(vc);
```

## Notas generales

-   En caso de no necesitar ningun orden, el monitor puede representar 100% al recurso compartido en vez de ser un "manejador" o "administrador" del recurso compartido.
-   Hacer signal en una **variable condición vacía no causa ningun error**.
-   En caso de necesitar respetar el orden de llegada, necesitamos un monitor que sea administrador del RC y que tenga 2 procedimientos, Pasar y Salir. El procedimiento Pasar va a encolar al proceso (dormirlo con wait()) si el recurso no está libre. El procedimiento Salir() le indica al próximo proceso en la cola que es su turno y pone el recurso en libre si la cola está vacía.
-   Si el orden no es por llegada, se necesita un arreglo de variables condición, donde cada proceso se duerme en su VC privada. Además se necesita una colaOrdenada para saber a cuál de esas VC privadas despertar.
-   En los ejercicios de tipo buffer 1 Consumidor y N Productores necesitamos 1) N procesos 2) un monitor Administrador 3) 1 proceso Servidor.
-   En los ejercicios de tipo buffer M Consumidores y N Productores necesitamos 1) N procesos 2) un monitor Administrador 3) M procesos Servidor. Lo único que cambia es while en vez de if empty(cola) en el procedure Siguiente y variables condición privadas para dormirse luego de hacer el push + el signal.
