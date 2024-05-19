<center>

# Rendezvous

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

```ada
Task tarea;

Task tarea is
  Entry entry1;
End tarea;

Task tarea is
  Entry entry1(A, B: IN integer; C: OUT char; D: IN OUT boolean);
End tarea;
```

</td><td>

Especificación de una tarea **sin entrys**, con entrys **sin parámetros** y con entrys **con parámetros**.

<tr><td>

```ada
Task type tipoTarea is
  Declaración de los ENTRYs del tipo tarea
End tipoTarea;
```

</td><td>

Especificación de un tipo tarea (para albergar **N tareas** iguales).

<tr><td>

```ada
Task body tarea is
  Variables
Begin
  Implementación de la tarea
End tarea;

Task body tipoTarea is
  Variables
Begin
  Implementación del tipo tarea
End tipoTarea;
```

</td><td>

Cuerpo de la tarea o tipo tarea.

<tr><td>

```ada
arreglo: Array(1..10) of tipoTarea,
```

</td><td>

Arreglo de tareas.

<tr><td>

```ada
Tarea1.nombreEntry;
Tarea2.nombreEntry(5, x, y, z);
```

</td><td>

Especificación de un **entry call** sin parámetros y luego con parámetros.

<tr><td>

```ada
Select
  Tarea2.nombreEntry(5, x, y, z);
  sentencias a realizar sólo si se completó el entry call (puede no haber ninguna)
Else
  sentencias a realizar solo si se canceló el entry call
End select;
```

</td><td>

Especificación de un **entry call condicional**. No espera a que le acepten el pedido, si la otra tarea no está lista para realizar el accept a su pedido **inmediatamente**, entonces lo cancela (lo quita de la cola implícita) y realiza otra cosa.

<tr><td>

```ada
Select
  Tarea2.nombreEntry(5, x, y, z);
  sentencias a realizar sólo si se completó el entry call (puede no haber ninguna)
Or delay tiempo
  sentencias a realizar solo si se canceló el entry call
End select;
```

</td><td>

Especificación de un **entry call temporal**. Espera a lo sumo un tiempo a que la otra tarea realice el accept a su pedido, **pasado el tiempo** cancela el entry call (lo quita de la cola implícita) y realiza otra cosa.

<tr><td>

```ada
Accept nombreEntry;

Accept nombreEntry is
  cuerpo del accept
End nombreEntry;

Accept nombreEntry(A, B: IN integer; C: OUT char; D: IN OUT boolean) is
  cuerpo del accept
End nombreEntry;
```

</td><td>

Especificación del accept sin cuerpo, con cuerpo, y con cuerpo y parámetros.

<tr><td>

```ada
Select
  Accept nombreEntry;
  sentencias a realizar sólo si se completó el accept
Or
  When (condición) -> Accept nombreEntry2 is
                        cuerpo del accept
                      End nombreEntry2;
                      sentencias a realizar sólo si se completó el accept
End select;
```

</td><td>

Select para los Accept. Permite tener varias alternativas de
ACCEPT que pueden o no tener asociada una condición booleana (no se puede utilizar los parámetros del entry como parte de la condición) utilizando la clausula when.

Si un accept no tiene "when" entonces su condición es siempre true.

Elige entre 2 o más **Accepts que estén en true y además tenga al menos 1 llamada pendientes** de forma no determinística.

Se le puede agregar al final un **else** o un **or delay** a esta estructura.

</table>

## Notas importantes

-   Lo que pongamos dentro del cuerpo de un accept debe ser lo más acotado posible, ya que estamos demorando al proceso que hizo el call. Evitar poner código ineficiente o lento allí.

## Tipos de ejercicios

#### ?

#### ?

#### ?

#### ?
