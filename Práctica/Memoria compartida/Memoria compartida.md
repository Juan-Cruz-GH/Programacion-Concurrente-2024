<center>

# Variables compartidas

</center>

## <> y Await

```
< Sentencia/s >
```

-   El proceso ejecuta la/s sentencia/s en forma **atómica**. Nos brinda sincronización por **exclusión mutua**.

```
< await (condición) Sentencia/s >
```

-   El proceso se demora (busy waiting) hasta que la condición sea verdadera y en ese momento ejecuta la/s sentencia/s en forma **atómica**. Nos brinda sincronización por **condición y exclusión mutua**.

```
< await (condición) >
```

-   El proceso se demora (busy waiting) hasta que la condición sea verdadera. Nos brinda sincronización por **condición**.

## Notas generales

-   Siempre tratar de usar el < > lo menos posible, ya que reduce la concurrencia. Además, nunca deberíamos poner nada que no sea totalmente necesario dentro de los < >. Por ejemplo, en vez de incrementar una variable compartida en forma atómica dentro de un for podemos hacerlo al final de las iteraciones, utilizando, en los procesos, variables locales contadoras.
-   Que la cola de procesos esperando para usar un recurso compartido este vacía no nos asegura que el recurso compartido no esté siendo utilizado actualmente. Podría haber un último proceso usando el recurso.
-   Una vez que la sección crítica se libera, **cualquiera** de los demás procesos que estaban esperando a acceder a esa sección crítica tomará control de la misma via exclusión mutua. No hay ningun tipo de orden. No se puede saber cuál proceso tomará la sección crítica.
-   Si queremos que los procesos accedan a la sección crítica según el orden de sus IDs, necesitamos una variable compartida inicializada en 0 que se irá incrementando de a 1 simbolizando los turnos de los procesos.
-   Si queremos que los procesos accedan a la sección crítica según el orden de llegada, necesitamos una Cola compartida que almacenará el orden original de los procesos en la primer iteración, orden que se respetará tal cual en las proximas iteraciones.
-   Si queremos que los procesos accedan a la sección crítica según el orden de alguno de sus atributos, necesitamos una ColaOrdenada compartida que almacenará el orden original de los procesos en la primer iteración, orden que se respetará tal cual en las proximas iteraciones.

---

<center>

# Semáforos

</center>

## Declaraciones

```cs
sem s; // ❌ Se deben inicializar en la declaración.
sem mutex = 1; // ✅ Inicializar un sem en 1 significa que 1 solo proceso a la vez podrá estar en la sección crítica.
sem espera[5] = ( [5] 1 ) // ✅ Arreglo de 5 semáforos, todos inicializados en 1. También llamados semáforos privados.
```

## Operaciones

-   P(s): **puede demorar** al proceso.

```cs
{ await (s > 0); s--; }
```

-   V(s): **nunca demora** al proceso.

```cs
{ s++; }
```

## Notas generales

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
    ... Completar

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

... Completar

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
-   Hay 2 formas de hacer barreras: ... Completar
