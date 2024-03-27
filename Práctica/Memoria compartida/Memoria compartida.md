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


---

<center>

# Monitores

</center>
