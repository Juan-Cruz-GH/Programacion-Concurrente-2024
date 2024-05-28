<h1 align="center">Semáforos</h1>

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
process Proceso [id: 0..N-1] {
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
int contador = 0
sem mutex = 1
process Proceso [id: 0..N-1] {
    for i = 0 to 10_000 {
        P(mutex)
        contador++
        V(mutex)
    }
}
```

</td><td>

```cs
int contador = 0
sem mutex = 1
process Proceso [id: 0..N-1] {
    int contadorLocal = 0
    for i = 0 to 10_000
        contadorLocal++
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
Cola cola(50)(int)
sem mutex1 = 1
sem mutex2 = 1
sem tamañoCola = 50
process Proceso [id: 0..N-1] {
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
Cola cola(50)(int)
sem mutex = 1
sem tamañoCola = 50
process Proceso [id: 0..N-1] {
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
-   En el ejemplo ambos procesos se bloquean ya que se duermen en el P y ninguno de los 2 llega a hacer el V.

```cs
sem espera = 0
process A {
    P(espera)
    V(espera)
}
process B {
    P(espera)
    V(espera)
}
```

#### Barreras

<table>
  <thead>
    <tr>
      <th>Barrera de un uso</th>
      <th>Barrera de un uso vía Coordinador</th>
    </tr>
  </thead>
<tr><td>

Se van durmiendo los procesos y una vez llegaron todos el último los despierta al resto "a la vez".

```cs
int contador = 0
sem mutex = 1
sem barrera = 0
process Proceso [id: 0..N-1] {
    P(mutex)
    contador++
    if contador == N
        // El último proceso
        // puede realizar una tarea él solo si quiere
        // RealizarTarea()
        for i=0 to N-2
            V(barrera)
    V(mutex)
    P(barrera)
}
```

</td><td>

Se van durmiendo los procesos y una vez llegaron todos el Coordinador los despierta a todos "a la vez".

```cs
sem espera = 0
sem barrera = 0

process Proceso [id: 0..N-1] {
    V(espera)
    P(barrera)
}

process Coordinador {
    for i = 0 to N-1
        P(espera)
    for i = 0 to N-1
        V(barrera)
}
```

</td></tr>
</table>

#### Passing The Condition vs Passing The Baton

-   En el método Passing The Baton, el control del semáforo es transferido del proceso que lo tiene actualmente, al siguiente. Suele usarse con semáforos binarios.
-   En el método Passing The Condition, los procesos se señalizan utilizando variables booleanas compartidas. Suele usarse con semáforos contadores.

#### Cuándo está libre el recurso compartido?

Para que el recurso compartido esté **libre** se deben cumplir dos condiciones simultáneamente:

1. Que la cola esté vacía.
2. Que nadie esté usando el recurso en este momento.

## Tipos de ejercicios

#### Límite con 2 tipos de procesos

-   Hay 2 tipos de procesos distintos y solo X cantidad del primer tipo de proceso puede estar en la sección crítica, solo Y del segundo tipo, y solo Z en total.
-   Tenemos los 3 semáforos y con los P y V ir de lo particular a lo general.

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

<table>
  <thead>
    <tr>
      <th>Con 1 recurso</th>
      <th>Con M recursos</th>
    </tr>
  </thead>
<tr><td>

-   Los procesos acceden al recurso compartido según el orden de llegada.
-   Tenemos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento además de dormirse en su semáforo privado.
-   Cuando el proceso actual termina de usar el recurso despierta al siguiente en la cola o pone libre en true si no hay nadie más.

```cs
Cola cola(int)
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

</td><td>

-   Los procesos acceden a M recursos compartidos según el orden de llegada.
-   Tenemos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento además de dormirse en su semáforo privado, y un entero que guarda la cantidad de recursos que quedan.
-   Cuando el proceso actual termina de usar el recurso despierta al siguiente en la cola o aumenta la cantidad de recursos disponibles si no hay nadie más.

```cs
Cola procesos(int)
Cola recursos(M)(int)
int recursosDisponibles = M
sem mutexProcesos = 1
sem mutexRecursos = 1
sem dormidos[N] = ([N] 0)
process Proceso [id: 0..N-1] {
    P(mutexProcesos)
    if recursosDisponibles != 0 {
        recursosDisponibles--
        V(mutexProcesos)
    }
    else {
        cola.push(id)
        V(mutexProcesos)
        P(dormidos[id])
    }

    P(mutexRecursos)
    r = recursos.pop()
    V(mutexRecursos)
    // Usa el recurso compartido
    P(mutexRecursos)
    recursos.push(r)
    V(mutexRecursos)

    P(mutexProcesos)
    if procesos.empty()
        recursosDisponibles++
    else
        V(dormidos[procesos.pop()])
    V(mutexProcesos)
}
```

</td></tr>
</table>

#### Orden de llegada con Coordinador

<table>
  <thead>
    <tr>
      <th>Con 1 recurso</th>
      <th>Con M recursos</th>
    </tr>
  </thead>
<tr><td>

-   Los procesos acceden al recurso compartido según el orden de llegada mediante un Coordinador.
-   Tenemos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento además de dormirse en su semáforo privado.
-   Cuando el proceso actual termina de usar el recurso le avisa al Coordinador el cual despierta al siguiente en la cola.

```cs
Cola cola(int)
sem esperando[N] = ([N] 0)
sem mutex = 1
sem pedidos = 0
sem termine = 0
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

</td><td>

-   Los procesos acceden a M recursos compartidos según el orden de llegada mediante un Coordinador.
-   Tenemos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento además de dormirse en su semáforo privado.
-   El proceso recibe el recurso que le toca vía un arreglo compartido, en su posición.
-   Cuando el proceso actual termina de usar el recurso le avisa al Coordinador el cual despierta al siguiente en la cola.

```cs
Cola cola(int)
Cola recursos(M)(int)
int miRecurso[M]
sem esperando[N] = ([N] 0)
sem cantidadRecursos = M
sem mutexProcesos = 1
sem mutexRecursos = 1
sem pedidos = 0
sem termine = 0
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

</td></tr>
</table>

#### Orden de algun atributo (prioridad)

-   Los procesos acceden al recurso compartido según el orden de uno de sus atributos.
-   Tenemos una ColaOrdenada compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento además de dormirse en su semáforo privado.
-   Cuando el proceso actual termina de usar el recurso despierta al siguiente en la cola o pone libre en true si no hay nadie más.

```cs

ColaOrdenada cola(int, int)
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

-   Los procesos acceden al recurso compartido según el orden de sus IDs.
-   Tenemos semáforos privados en los cuales los procesos se duermen si no es su turno.
-   El proceso actual, cuando termina, despierta al siguiente.

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

#### N procesos consumen de una cola de recursos compartidos

-   Tenemos N procesos que tienen que consumir una cola de recursos lo más rápido posible. La estructura es:

```
P
while
    V
    P
V
```

```cs
Cola recursos(int)
sem mutex = 1
process Proceso [id: 0..N-1] {
    Recurso recurso
    P(mutex)
    while recursos.notEmpty() {
        recurso = recursos.pop()
        V(mutex)
        // Usa el recurso compartido
        P(mutex)
    }
    V(mutex)
}
```
