<h1 align="center">Rendezvous</h1>

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

#### Accepts acotados

-   Lo que pongamos dentro del cuerpo de un accept debe ser lo más acotado posible, ya que estamos demorando al proceso que hizo el call. Evitar poner código ineficiente o lento allí.

<table>
  <thead>
    <tr>
      <th>❌</th>
      <th>✅</th>
    </tr>
  </thead>
<tr><td>

```ada
procedure ejemplo is
  task type Servidor is
    entry Pedido(p: IN int; resul: OUT int)
  end Servidor
  servidores: array(1..N) of Servidor

  task Persona

  task body Persona is
    pedido, resultado, resultadoTotal: int
  begin
    for i=1 to N loop
      servidores(i).Pedido(pedido, resultado)
      resultadoTotal:= resultadoTotal + resultado
    end loop
  end Persona

  task body Servidor is
    valor: int
  begin
    loop
      Accept Pedido(p: IN int; resul: OUT int)
        resul:= Procesar(p)
      end Pedido
    end loop
  end Servidor
begin
  null
end
```

Está mal porque no realiza todos los pedidos al mismo tiempo: realiza uno, espera que termine, luego realiza el siguiente, etc. No hay concurrencia.

</td><td>

```ada
procedure ejemplo is
  task type Servidor is
    entry Pedido(p: IN int)
  end Servidor
  servidores: array(1..N) of Servidor

  task Persona is
    entry Resultado(r: IN int)
  end Persona

  task body Persona is
    pedido, resultadoTotal: int
  begin
    for i=1 to N loop
      servidores(i).Pedido(pedido)
    end loop
    for i=1 to N loop
      Accept Resultado(r: IN int) is
        resultadoTotal:= resultadoTotal + r
      end Resultado
    end loop
  end Persona

  task body Servidor is
    pedido, resultado: int
  begin
    loop
      Accept Pedido(p: IN int)
        pedido:= p
      end Pedido
      resultado := Procesar(pedido)
      Persona.Resultado(resultado)
    end loop
  end Servidor
begin
  null
end
```

</td></tr>
</table>

#### Asignación de IDs

-   A veces necesitamos que los procesos sepan cuál es su ID.
-   Como los procesos no conocen sus IDs por defecto, para lograr que los conozcan se los enviamos desde el programa principal mediante un Entry que tienen los procesos.

```ada
procedure ejemplo is
  task type Persona is
    entry MiID(i: IN int)
  end Persona

  personas: array(1..N) of Persona

  task body Persona is
    id: int
  begin
    Accept MiID(i: IN int) is
      id := i
    end MiID
  end Persona
begin
  for i=1 to N loop
    personas(i).MiID(i)
  end loop
end ejemplo
```

#### Manejar prioridad

-   Podemos manejar la prioridad utilizando select + when + accept.
-   Solo se aceptan pedidos normales cuando hay ninguno prioritario.

```ada
...
loop
  select
    Accept PedidoPrioritario()
  or
    when PedidoPrioritario'count = 0 ->
      Accept PedidoNormal()
end loop
...
```

#### Todos los procesos necesitan que el Coordinador les envie un valor

-   Cuando tenemos esta situación, es mejor que los procesos le pidan el valor al Coordinador, es decir que el Entry sea del Coordinador, y no al revés.
-   Esto es porque si el Entry lo tienen los procesos, el Coordinador debe primero conocer los IDs de los procesos, y además, si el 1er proceso por ejemplo pierde CPU, bloquea el resto del loop.

```ada
procedure ejemplo is
  task Coordinador is
    entry ConocerValor(v: OUT int)
  end Coordinador

  task type Persona

  personas: array(1..N) of Persona

  task body Persona is
    valor: int
  begin
    Coordinador.ConocerValor(valor)
  end Persona

  task body Coordinador is
    valor: int
  begin
    for i=1 to N loop
      Accept ConocerValor(v: OUT int)
        v:= valor
      end ConocerValor
    end loop
  end Coordinador
begin
  null
end
```

## Tipos de ejercicios

#### ?

#### ?

#### ?

#### ?
