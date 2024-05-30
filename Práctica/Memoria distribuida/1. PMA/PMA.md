<h1 align="center">Pasaje de Mensajes Asincrónicos</h1>

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
chan nombre(tipoDato)
```

</td><td>

Canal único.

<tr><td>

```cs
chan arreglo[N](tipoDato)
```

</td><td>

Canales privados.

<tr><td>

```cs
send canal(dato)
```

</td><td>

Deja un dato en la cola implícita del canal. Es **no bloqueante**.

<tr><td>

```cs
send arreglo[i](dato)
```

</td><td>

Deja un dato en la cola implícita de un canal privado. Es **no bloqueante**.

<tr><td>

```cs
receive canal(variable)
```

</td><td>

Quita un elemento de la cola implícita del canal y lo pone en la variable. Es **bloqueante**.

<tr><td>

```cs
receive arreglo[i](variable)
```

</td><td>

Quita un elemento de la cola implícita del canal privado y lo pone en la variable. Es **bloqueante**.

<tr><td>

```cs
empty(canal)
```

</td><td>

Retorna V si la cola implícita del canal está vacía, F caso contrario.

<tr><td>

```cs
empty(arreglo[i])
```

</td><td>

Retorna V si la cola implícita del canal privado está vacía, F caso contrario.

</table>

## Notas importantes

-   No hay variables compartidas, solo se declaran canales o canales privados.

#### Canales privados

-   Cuando un proceso único (un servidor por ej) debe comunicarse con procesos especificos de un arreglo de procesos, es necesario usar **canales privados**.

```cs
chan privados[N](int)

process Persona[id: 0..N-1] {
  int num
  receive privados[id](num)
}

process Coordinador {
  for i=0 to N-1
    send privados[i](i)
}
```

#### Operación empty

-   La operación empty debe usarse con cuidado, si se utiliza dentro de un arreglo de procesos se pueden producir errores. Hay que usarla dentro de un único proceso **Coordinador**.

<table>
  <thead>
    <tr>
      <th>❌</th>
      <th>✅</th>
    </tr>
  </thead>
<tr><td>

```cs
chan pedidos(int)
process Persona[id: 0..N-1] {
  send pedidos(int.random())
}

process Empleado[id: 0..4] {
  int num
  if empty(pedidos)
    delay(60)
  else
    receive pedidos(num)
}
```

</td><td>

```cs
chan pedidos(int)
chan empleados(int)
chan empleadoSig[5](int)

process Persona[id: 0..N-1] {
  send pedidos(int.random())
}

process Coordinador {
  int idE, num
  receive empleados(idE)
  if empty(pedidos)
    num = -1
  else
    receive pedidos(num)
  send empleadoSig[idE](num)
}

process Empleado[id: 0..4] {
  int num
  send empleados(id)
  receive empleadoSig[id](num)
  if num == -1
    delay(60)
  else
    ProcesarPedido(num)
}
```

</td></tr>
</table>

#### If no determinístico

-   Para evitar busy waiting en un while true, podemos usar un canal vacío aviso() y hacer un receive aviso() antes del if.

<table>
  <thead>
    <tr>
      <th>❌</th>
      <th>✅</th>
    </tr>
  </thead>
<tr><td>

```cs
chan pedidos(int)

process Persona[id: 0..N-1] {
  send pedidos(int.random())
}

process Coordinador {
  int num
  while true {
    if not empty(pedidos)
      receive pedidos(num)
  }
}
```

Si no llegan pedidos por un rato,
se chequea la condición una y otra vez
generando busy waiting.

</td><td>

```cs
chan pedidos(int)
chan aviso

process Persona[id: 0..N-1] {
  send pedidos(int.random())
  send aviso()
}

process Coordinador {
  int num
  while true {
    receive aviso()
    if not empty(pedidos)
      receive pedidos(num)
  }
}

```

</td></tr>
</table>

#### Manejar prioridad con if no determinístico

-   Podemos hacer que se reciba un dato por uno u otro canal con prioridad utilizando if no determinísticos.

```cs
chan aviso()
chan pedidosPrioridad(int)
chan pedidosNormal(int)

process Coordinador {
  while true {
    receive aviso()
    if empty(pedidosPrioridad) and not empty(pedidosNormal)
      receive pedidosNormal()
    [] not empty(pedidosPrioridad)
      receive pedidosPrioridad()
  }
}
```

## Tipos de ejercicios

#### N clientes y 1 o más servidores

-   Hay N clientes que realizan pedidos y esperan **su** resultado.
-   Hay 1 o más servidores que aceptan los pedidos, los procesan y devuelven el resultado a ese cliente específico.
-   Hacemos uso de los canales privados.

```cs
chan pedidos(string)
chan miRespuesta[N](string)

process Cliente[id: 0..N-1] {
  string pedido, respuesta
  while true {
    send pedidos(id, pedido)
    receive miRespuesta[id](respuesta)
  }
}

process Servidor[id: 0..3] {
  int idC
  string pedido, respuesta
  while true {
    receive pedidos(idC, pedido)
    respuesta = Procesar(pedido)
    send miRespuesta[idC](respuesta)
  }
}
```

#### Más de 1 servidor que deben hacer delay cuando no hay pedidos

-   Si tenemos más de un servidor y éstos deben usar la operación delay() cuando no hay pedidos pendientes, se necesita un proceso intermedio Coordinador que es el que efectivamente chequea esa condición.

```cs
chan pedidos(string)
chan miRespuesta[N](string)
chan servidoresLibres(int)
chan servidorSiguiente[4](int, string)

process Cliente[id: 0..N-1] {
  string pedido, respuesta
  while true {
    send pedidos(id, pedido)
    receive miRespuesta[id](respuesta)
  }
}

process Coordinador {
  int idC, idS, pedido
  while true {
    receive servidoresLibres(idS)
    if empty(pedidos)
      idC = -1
    else
      receive pedidos(idC, pedido)
    send servidorSiguiente[idS](idC, pedido)
  }
}

process Servidor[id: 0..3] {
  int idC
  string pedido, respuesta
  while true {
    send servidoresLibres(id)
    receive servidorSiguiente[id](idC, pedido)
    if idC == -1
      delay(60)
    else {
      respuesta = Procesar(pedido)
      send miRespuesta[idC](respuesta)
    }
  }
}
```

#### Manejo de pedidos con prioridad

-   Queremos que los pedidos de ClientePrioridad tengan prioridad sobre los de los ClientesNormales, es decir que solo se aceptarán pedidos de clientes normales cuando no haya **ninguno** de clientes prioritarios.

```cs
chan pedidos(string)
chan miRespuesta[N](string)
chan servidoresLibres(int)
chan servidorSiguiente[4](int, string)

process Cliente[id: 0..N-1] {
  string pedido, respuesta
  bool soyPrioritario
  while true {
    if soyPrioritario {
      send pedidosPrioritarios(id, pedido)
      send aviso()
    }
    else {
      send pedidosNormales(id, pedido)
      send aviso()
    }
    receive miRespuesta[id](respuesta)
  }
}

process Coordinador {
  int idC, idS, pedido
  while true {
    receive aviso()
    if empty(pedidosPrioritarios) and not empty(pedidosNormales)
      receive pedidosNormales(idC, pedido)
    [] not empty(pedidosPrioritarios)
      receive pedidosPrioritarios(idC, pedido)
    send servidores(idC, pedido)
  }
}

process Servidor[id: 0..3] {
  int idC
  string pedido, respuesta
  while true {
    receive servidores(idC, pedido)
    respuesta = Procesar(pedido)
    send miRespuesta[idC](respuesta)
  }
}
```
