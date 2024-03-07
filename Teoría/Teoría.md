# Recursos

-   [Playlist con todas las teorías grabadas](https://www.youtube.com/playlist?list=PLDJU8kNAPOn-nY4Q5YesYIYuGw1AqIUm7)
- Test
<center>

# Clase 1

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

Tenemos un multicore. Todos los cores trabajan al mismo tiempo haciendo cada uno una tarea.

#### Concurrencia

Por ende, la concurrencia es un concepto de **software** que se puede adaptar a **cualquier arquitectura**!!. Un programa concurrente puede ser ejecutado en un monocore o multicore.

## Programa Concurrente

-   Que un programa se divida en 2 o más **procesos** que actúan al mismo tiempo para resolver un mismo problema.
-   Los procesos que forman parte de un programa concurrente suelen **interactuar** entre sí y comunicarse o sincronizarse.
-   No son determinísticos, lo cual dificulta el debugging.

#### Comportamiento de los procesos

1. **Independencia:** Pueden ser independientes entre sí y no interactuar nunca.
2. **Cooperación:** Por el contrario, pueden cooperar entre sí y comunicarse y sincronizarse para resolver una tarea común.
3. **Competencia:** Pueden competir por uno o más recursos compartidos.

Estas formas de actuar pueden combinarse de cualquier forma, en un programa concurrente.

## Procesos vs Hilos

1. **Proceso:** Pedazos de una cosa más grande. Cada proceso tiene su propio espacio de direcciones y recursos. Las librerias de memoria distribuida trabajan con procesos.
2. **Proceso liviano/thread/hilo:** Sub-pedazos de un mismo proceso, que tienen su propio contador de programa y su pila de ejecución, pero comparten el mismo espacio de direcciones y recursos que su proceso padre. Las librerias de memoria compartida trabajan con hilos.

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

Un proceso hace algo que invalida las suposiciones que hizo otro proceso. Casi siempre ocurre por no respetar las secciones críticas utilizando exclusión mutua o sinc. por condición. Debe evitarse siempre.

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

## Instrucciones

-   **_skip_**: termina inmediatamente y no tiene ningún efecto sobre ninguna variable del programa.
-   **_Sentencia de alternativa múltiple:_** Se chequean toads las condiciones booleanas al msimo tiempo y de las que son verdaderas se elige **una sola** de ellas en forma no determinística y se ejecuta la serie de instrucciones asociadas a esa condición.

```
if B1 -> S1
   B2 -> S2
   Bn -> Sn
fi
```

-   **_Sentencia de alternativa ITERATIVA múltiple:_** Igual que el anterior pero sigue repetiendo la ejecución una y otra vez hasta que TODAS las condiciones sean **falsas**.

```
do B1 -> S1
   B2 -> S2
   Bn -> Sn
od
```

-   **_co_**: Ejecuta Sn tareas concurrentemente. Su ejecución termina cuando todas las tareas concurrentes terminaron.

-   **_Process_**: Forma de definir un proceso concurrente. Podemos usar [i=1 to N] para definir N de estos.
