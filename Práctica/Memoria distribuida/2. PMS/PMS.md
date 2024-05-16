<center>

# Pasaje de Mensajes Sincrónicos

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
-   Siempre que tengamos N procesos y 1 Servidor, donde los procesos realizan tareas y no necesitan recibir el resultado de las mismas, necesitamos un proceso Buffer intermedio para que se vayan encolando los trabajos y los procesos no se retrasen.
-   Hay que tener cuidado al usar la recepción con [*], ya que esto implica que el Servidor recibe pedidos en cualquier orden. Para que se respete el orden de llegada necesitamos nuevamente un proceso intermedio, esta vez no un buffer si no un Administrador o Coordinador.

## Tipos de ejercicios

#### ?

#### ?

#### ?

#### ?
