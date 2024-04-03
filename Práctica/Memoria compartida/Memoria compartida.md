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
sem s; // Incorrecto! Se deben inicializar en la declaración.
sem mutex = 1;
sem espera[5] = ( [5] 1 ) // Arreglo de 5 semáforos, todos inicializados en 1.
```

## Operaciones

- P(s): puede demorar al proceso.
```cs
{ await ( s > 0 ); s = s - 1; }
```

- V(s): nunca demora al proceso.
```cs
{ s = s + 1; }
```

## Notas generales

- Nuevamente hay que **achicar lo más posible las secciones críticas**: es decir, cuando usamos un semáforo mutex = 1, lo que esté dentro del P y el V tiene que ser lo más acotado posible. Es por esto que, al igual que en Variables Compartidas, podemos por ejemplo usar contadores locales dentro del bucle que ejecutan muchos procesos, y al finalizar ese bucle actualizar la variable global con esos contadores locales, minimizando drásticamente la cantidad de bloqueos y liberaciones de la sección crítica.

- Hay que tener mucho cuidado al inicializar semáforos en 0. Se puede producir deadlock fácilmente si no lo hacemos bien.

- Cosas que estén relacionadas entre sí deben ir en una misma sección crítica, no en 2 o más secciones críticas separadas. Análogamente, no tiene sentido usar el mismo semáforo para 2 secciones críticas separadas.


- Barrera de un uso, donde queremos despertar a todos a la vez:
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

- Barrera de un uso, donde queremos despertar a todos a la vez, vía un Coordinador:
```cs
sem espera = 0;
sem barrera = 0;
// Procesos
V(espera)
P(barrera)
---
// Coordinador
for i = 0 to N - 1 {
    P(espera)
}
for i = 0 to N - 1 {
    V(barrera)
}

```

- Cuando queremos despertar a proceso/s en particular, necesitamos **semáforos privados** (arreglo de semáforos).

- Si estamos en un bucle no hay que preguntar si la cola está vacía, ya que causamos busy waiting. Tenemos que tener un semáforo contador que se inicializa en 0 y que va a tener en todo momento la cantidad de elementos en la cola, y los procesos harán un V sobre ese semáforo cada vez que pusheen algo. El coordinador hará un P, para esperar que haya por lo menos 1 elemento.

- Para que el recurso compartido esté **libre** se deben cumplir dos condiciones simultáneamente: 
    1. Que la cola esté vacía.
    2. Que nadie esté usando el recurso en este momento.

- Passing The Condition vs Passing The Baton: ???
---

<center>

# Monitores

</center>
