<center>

# Pasaje de Mensajes Sincrónicos

</center>

## Notas generales

-   No hay variables compartidas, solo tenemos procesos. Debido a esto, no se necesita ningún método de mutex.
-   Los canales son implícitos, no se declaran.
-   Los canales son de tipo **link**, un único emisor y un único receptor.

## Sintaxis

-   Envío: es bloqueante y sincrónica.

```cs
destino!port(dato)
destino[i]!port(dato)
```

-   Recepción: es bloqueante y sincrónica.

```cs
origen?port(variable)
origen[i]?port(variable)
origen[*]?port(variable)
```

## Comunicación guardada

-   Permite, a través de las estructuras If y Do no determínisticos, seleccionar en forma aleatoria entre varias alternativas de **recepción** en base a las condiciones y los mensajes que están listos para ser recibidos.
-   Cada alternativa del If o Do ND es una guarda de la forma:

```cs
Condición; Recepción -> Sentencias
```

-   La condición indica si el proceso está o no en condiciones de procesar el mensaje recibido en la Recepción. Puede omitirse en cuyo caso siempre es True.
-   La Recepción es justamente una sentencia de recepción (con el símbolo ?) que siempre debe estar.
-   Las Sentencias se ejecutarán luego de realizar la Recepción.
-   Cada guarda puede estar en 1 de 3 posibles estados:
    -   Éxito: La condición es V o está omitida y la recepción puede realizarse en ese mismo instante sin demora alguna (el emisor ya estaba esperando hacer el envío).
    -   Bloqueo: La condición es V o está omitida pero la recepción no puede realizarse de inmediato (el emisor aún no hizo el envío).
    -   Fallo: La condición es F.

## If y Do no determinísticos con comunicación guardada

-   En el If ND se evalúan **todas** las guardas al mismo tiempo y en base a eso:

    -   Si **una o más son exitosas** se selecciona una de ellas en forma aleatoria, se realiza la recepción, y se ejecutan las sentencias.
    -   Si ninguna guarda es exitosa pero hay **una o más con estado de bloqueo** entonces el proceso se demora en el If hasta que haya una exitosa, realiza la recepción, y ejecuta las sentencias.

-   El Do ND es igual que el If con la única diferencia de que en vez de realizar lo mencionado una sola vez, repite el mecanismo una y otra vez **hasta que todas las guardas estén en estado de fallo**.

## Notas importantes

-   Siempre que tengamos N procesos y 1 Servidor, donde los procesos realizan tareas y no necesitan recibir el resultado de las mismas, necesitamos un proceso Buffer intermedio para que se vayan encolando los trabajos y los procesos no se retrasen.
-   Hay que tener cuidado al usar la recepción con [*], ya que esto implica que el Servidor recibe pedidos en cualquier orden. Para que se respete el orden de llegada necesitamos nuevamente un proceso intermedio, esta vez no un buffer si no un Administrador o Coordinador.

---

<center>

# Rendezvous

</center>
