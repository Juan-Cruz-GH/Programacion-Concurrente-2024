<h1 align="center">Pasaje de Mensajes Sincrónicos</h1>

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
procesoDestino!etiqueta(dato)
```

</td><td>

Un proceso le envía un dato a otro. Es **bloqueante**.

<tr><td>

```cs
procesoDestino[i]!etiqueta(dato)
```

</td><td>

Un proceso le envía un dato a otro específico dentro de un arreglo de procesos. Es **bloqueante**.

<tr><td>

```cs
procesoOrigen?etiqueta(variable)
```

</td><td>

Un proceso se queda esperando a que el procesoOrigen le envíe un dato, el cual almacena en variable. Es **bloqueante**.

<tr><td>

```cs
procesoOrigen[i]?etiqueta(variable)
```

</td><td>

Un proceso se queda esperando a que el procesoOrigen específico dentro del arreglo de procesos le envíe un dato, el cual almacena en variable. Es **bloqueante**.

<tr><td>

```cs
(Condición); Recepción -> Sentencias
```

</td><td>

-   La condición indica si el proceso está o no en condiciones de procesar el mensaje recibido en la Recepción. Puede omitirse en cuyo caso siempre es True.
-   La Recepción es justamente una sentencia de recepción (con el símbolo ?) que siempre debe estar.
-   Las Sentencias se ejecutarán luego de realizar la Recepción.
-   Cada guarda puede estar en 1 de 3 posibles estados:
    -   Éxito: La condición es V o está omitida y la recepción puede realizarse en ese mismo instante sin demora alguna (el emisor ya estaba esperando hacer el envío).
    -   Bloqueo: La condición es V o está omitida pero la recepción no puede realizarse de inmediato (el emisor aún no hizo el envío).
    -   Fallo: La condición es F.

<tr><td>

```cs
if
    []
    []
fi
do
    []
    []
od
```

</td><td>

-   En el If No Determinístico se evalúan **todas** las guardas al mismo tiempo y en base a eso:

    -   Si **una o más son exitosas** se selecciona una de ellas en forma aleatoria, se realiza la recepción, y se ejecutan las sentencias.
    -   Si ninguna guarda es exitosa pero hay **una o más con estado de bloqueo** entonces el proceso se demora en el If hasta que haya una exitosa, realiza la recepción, y ejecuta las sentencias.

-   El Do No Determinístico es igual que el If con la única diferencia de que en vez de realizar lo mencionado una sola vez, repite el mecanismo una y otra vez **hasta que todas las guardas estén en estado de fallo**.

</table>

## Notas importantes

-   No hay variables compartidas y los canales son implícitos, solo tenemos procesos.
-   Hay que tener cuidado al usar la recepción con [*], ya que si más de un proceso hizo un envío en ese canal, se recibirá el pedido de cualquiera de ellos, sin respetar el orden de llegada. Para que se respete el orden de llegada necesitamos un proceso intermedio, no un buffer si no un Administrador o Coordinador.

#### Barrera

```cs
process Cliente[id: 0..N-1] {
  Barrera!llegar()
  Barrera?seguir()
}

process Barrera {
  for i=0 to N-1
    Cliente[*]?llegar()
  for i=0 to N-1
    Cliente[i]!seguir()
}
```

## Tipos de ejercicios

#### Encolar pedidos cuando el servidor está ocupado, 1 Servidor

-   Siempre que tengamos N procesos y 1 Servidor, donde los procesos realizan tareas y no necesitan recibir el resultado de las mismas, necesitamos un proceso Buffer intermedio para que se vayan encolando los trabajos y los procesos no se retrasen.

```cs
process Cliente[id: 0..N-1] {
  string pedido
  while true {
    pedido = Pedido()
    Buffer!nuevoPedido(pedido)
  }
}

process Buffer {
  Cola pedidos(string)
  string pedido
  do
    [] Cliente[*]?nuevoPedido(pedido) -> pedidos.push(pedido)
    [] (pedidos.notEmpty); Servidor?libre() -> Servidor!siguiente(pedidos.pop())
  od
}

process Servidor {
  string pedido, respuesta
  Cola respuestas(string)
  while true {
    Buffer!libre()
    Buffer?siguiente(pedido)
    respuesta = Procesar(pedido)
    respuestas.push(respuesta)
  }
}
```

#### Encolar pedidos cuando el servidor está ocupado, S Servidores

-   Siempre que tengamos N procesos y 1 Servidor, donde los procesos realizan tareas y no necesitan recibir el resultado de las mismas, necesitamos un proceso Buffer intermedio para que se vayan encolando los trabajos y los procesos no se retrasen.

```cs
process Cliente[id: 0..N-1] {
  string pedido
  while true {
    pedido = Pedido()
    Buffer!nuevoPedido(pedido)
  }
}

process Buffer {
  Cola pedidos(string)
  string pedido
  int idS
  do
    [] Cliente[*]?nuevoPedido(pedido) -> pedidos.push(pedido)
    [] (pedidos.notEmpty); Servidor[*]?libre(idS) -> Servidor[idS]!siguiente(pedidos.pop())
  od
}

process Servidor[id: 0..S-1] {
  string pedido, respuesta
  Cola respuestas(string)
  while true {
    Buffer!libre(id)
    Buffer?siguiente[id](pedido)
    respuesta = Procesar(pedido)
    respuestas.push(respuesta)
  }
}
```

#### Exclusión mutua vía un Empleado

-   Queremos que los procesos accedan a un recurso con E.M por orden de llegada, donde un Empleado le va avisando al siguiente.

```cs
process Cliente[id: 0..N-1] {
  Admin!pedido(id)
  Empleado?empezar()
  UsarMaquina() // Usa la máquina con exclusión mutua
  Empleado!liberar()
}

process Admin {
  Cola pedidos(int)
  int idC
  do
    [] Cliente[*]?pedido(idC) -> pedidos.push(idC)
    [] (pedidos.notEmpty); Empleado?libre(idS) -> Empleado!siguiente(pedidos.pop())
  od
}

process Empleado {
  int idC
  while true {
    Admin!libre()
    Admin?siguiente(idC)
    Cliente[idC]!empezar()
    Cliente[idC]?liberar()
  }
}
```

#### Exclusión mutua vía el Coordinador mismo

-   Queremos que los procesos accedan a un recurso con E.M por orden de llegada, donde no hay un empleado si no que el Administrador mismo maneja el aviso al siguiente.

```cs
process Cliente[id: 0..N-1] {
  Admin!pedido(id)
  Admin?empezar()
  UsarMaquina() // Usa la máquina con exclusión mutua
  Admin!liberar()
}

process Admin {
  Cola pedidos(int)
  int idC
  bool libre = true
  do
    [] Cliente[*]?pedido(idC) ->
      if libre
        libre = false
        Cliente[idC]!empezar()
      else
        pedidos.push(idC)
    [] Cliente[*]?liberar() ->
      if pedidos.empty()
        libre = true
      else {
        idC = pedidos.pop()
        Cliente[idC]!empezar()
      }
  od
}
```
