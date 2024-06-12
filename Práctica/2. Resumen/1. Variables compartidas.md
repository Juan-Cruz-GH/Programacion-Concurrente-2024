<h1 align="center">Variables compartidas</h1>

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
int contador
process Proceso [id: 1..N] {
    for i = 0 to 10_000 {
        < contador++ >
    }
}
```

</td><td>

```cs
int contador
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

-   Los procesos acceden al recurso compartido según el orden de llegada.
-   Tenemos una Cola compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento.

```cs
Cola cola
int siguiente = -1
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

-   Los procesos acceden al recurso compartido según el orden de uno de sus atributos.
-   Tenemos una ColaOrdenada compartida a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento.

```cs
ColaOrdenada cola
int siguiente = -1
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

-   Los procesos acceden al recurso compartido según el orden de sus IDs.
-   Tenemos una variable compartida inicializada en 0 que se irá incrementando de a 1 simbolizando los turnos de los procesos. Los demás procesos esperan a que siguiente sea su ID.

```cs
int siguiente = 0
process Proceso [id: 0..N-1] {
    < await (siguiente == id) >
    // Usa el recurso compartido
    siguiente++
}
```

#### Cuándo está libre el recurso compartido?

-   Que la cola de procesos esperando para usar un recurso compartido este vacía no nos asegura que el recurso compartido no esté siendo utilizado actualmente. Podría haber un último proceso usando el recurso.
