<center>

# Variables compartidas

</center>

## Sintaxis

| Instrucción                         | Semántica                                                                                                                                                                                               |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `< Sentencia/s >`                   | El proceso ejecuta la/s sentencia/s en forma **atómica**. Nos brinda sincronización por **exclusión mutua**.                                                                                            |
| `< await (condición) Sentencia/s >` | El proceso se demora (busy waiting) hasta que la condición sea verdadera y en ese momento ejecuta la/s sentencia/s en forma **atómica**. Nos brinda sincronización por **condición y exclusión mutua**. |
| `< await (condición) >`             | El proceso se demora hasta que la condición sea verdadera. Nos brinda sincronización por **condición**.                                                                                                 |

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
-   Por ejemplo, en vez de incrementar una variable compartida en forma atómica dentro de un for podemos hacerlo al final de las iteraciones, utilizando, en los procesos, variables locales contadoras:

```cs
// ❌ Código ineficiente
int contador;
process Proceso [id: 1..N] {
    for i = 0 to 10_000 {
        < contador++ >
    }
}
```

```cs
// ✅ Código eficiente
int contador;
process Proceso [id: 1..N] {
    int contadorLocal
    for i = 0 to 10_000 {
        contadorLocal++
    }
    < contador += contadorLocal >
}
```

#### Orden de llegada

-   Si queremos que los procesos accedan a la sección crítica según el orden de llegada, necesitamos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento:

```cs
Cola cola; int siguiente = -1;
process Persona [id: 1..N] {
    while true {
        < if siguiente == -1
            siguiente = id
          else
            cola.push(id) >
        < await (siguiente == id) >
        // Sección crítica
        < if cola.empty()
            siguiente = -1
          else
            siguiente = cola.pop() >
    }
}
```

#### Orden de algun atributo (prioridad)

-   Si queremos que los procesos accedan a la sección crítica según el orden de uno de sus atributos, necesitamos una ColaOrdenada compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento:

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
        // Sección crítica
        < if cola.empty()
            siguiente = -1
          else
            siguiente = cola.pop() >
    }
}
```

#### Orden de identificador (ID)

-   Si queremos que los procesos accedan a la sección crítica según el orden de sus IDs, necesitamos una variable compartida inicializada en 0 que se irá incrementando de a 1 simbolizando los turnos de los procesos. Los demás procesos esperan a que siguiente sea su ID:

```cs
int siguiente = 1
process Proceso [id: 1..N] {
    < await (siguiente == id) >
    // Sección critica
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

| Instrucción           | Semántica                                                                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sem semaforo = 0`    | Un semáforo inicializado en 0 significa que todos los procesos se quedaran esperando ahí hasta que otro proceso haga un V().                      |
| `sem semaforo = N`    | Un semáforo inicializado en N significa que N procesos podrán estar en la sección crítica a la vez.                                               |
| `sem semaforo[N] = 0` | Arreglo de semáforos inicializados en 0, para que cada proceso de un arreglo de procesos se quede esperando. También llamados semáforos privados. |
| `P(semaforo)`         | Si semáforo == 0 -> se queda esperando. Si no -> semáforo--                                                                                       |
| `V(semaforo)`         | semáforo++                                                                                                                                        |

## Notas importantes

#### ...

-   ...

```cs

```

#### ...

-   ...

```cs

```

#### ...

-   ...

```cs

```

#### ...

-   ...

```cs

```

-   Nuevamente hay que **achicar lo más posible las secciones críticas**: es decir, cuando usamos un semáforo mutex = 1, lo que esté dentro del P y el V tiene que ser lo más acotado posible. Es por esto que, al igual que en Variables Compartidas, podemos por ejemplo usar contadores locales dentro del bucle que ejecutan muchos procesos, y al finalizar ese bucle actualizar la variable global con esos contadores locales, minimizando drásticamente la cantidad de bloqueos y liberaciones de la sección crítica.
-   Hay que tener mucho cuidado al inicializar semáforos en 0. Se puede producir deadlock fácilmente si no lo hacemos bien.
-   Cosas que estén relacionadas entre sí deben ir en una misma sección crítica, no en 2 o más secciones críticas separadas. Análogamente, no tiene sentido usar el mismo semáforo para 2 secciones críticas separadas.
-   Barrera de un uso, donde queremos despertar a todos a la vez:

```cs
sem mutex = 1;
sem barrera = 0;

P(mutex)
contador++;
if contador == N {
    for i to N {
        V(barrera)
    }
}
V(mutex)
P(barrera)
```

-   Barrera de un uso, donde queremos despertar a todos a la vez, vía un Coordinador:

```cs
sem espera = 0;
sem barrera = 0;
// Procesos
V(espera)
P(barrera)
---
// Coordinador
for i = 0 to N-1 {
    P(espera)
}
for i = 0 to N-1 {
    V(barrera)
}

```

-   Cuando queremos despertar a proceso/s en particular, o enviarle datos a proceso/s en particular, necesitamos **semáforos privados** (arreglo de semáforos).
-   Si estamos en un bucle no hay que preguntar si la cola está vacía, ya que causamos busy waiting. Tenemos que tener un semáforo contador que se inicializa en 0 y que va a tener en todo momento la cantidad de elementos en la cola, y los procesos harán un V sobre ese semáforo cada vez que pusheen algo. El coordinador hará un P, para esperar que haya por lo menos 1 elemento.
-   Para que el recurso compartido esté **libre** se deben cumplir dos condiciones simultáneamente:

    1. Que la cola esté vacía.
    2. Que nadie esté usando el recurso en este momento.

-   Passing The Condition vs Passing The Baton:
    -   En el método Passing The Baton, el control del semáforo es transferido del proceso que lo tiene actualmente, al siguiente. Suele usarse con semáforos binarios.
    -   En el método Passing The Condition, los procesos se señalizan utilizando variables booleanas compartidas. Suele usarse con semáforos contadores.

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
