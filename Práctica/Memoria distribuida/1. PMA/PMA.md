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
-   Cuando un proceso único (un servidor por ej) debe comunicarse con procesos especificos de un arreglo de procesos, es necesario usar **canales privados**.
-   La operación empty debe usarse con cuidado, si se utiliza dentro de un arreglo de procesos se pueden producir errores. Hay que usarla dentro de un único proceso **Coordinador**.

## Tipos de ejercicios

#### ?

#### ?

#### ?

#### ?
