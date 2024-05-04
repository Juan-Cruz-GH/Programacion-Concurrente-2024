<center>

# Pasaje de Mensajes Asincrónicos

</center>

## Notas generales

-   No hay variables compartidas, solo tenemos procesos y canales globales. Debido a esto, no se necesita ningún método de mutex.
-   Los canales son "colas", cuando se les envia un mensaje lo encolan, cuando se recibe un mensaje lo desencolan.
-   Los canales son de tipo **mailbox**, cualquier proceso puede enviar y/o recibir por cualquier canal.

## Sintaxis

-   Declaración de canales:

```cs
chan nombre(tipoDato) // Canal único.

chan arreglo[N](tipoDato) // Arreglo de canales.
```

-   Send: es no bloqueante.

```cs
send canal(dato)
send arreglo[i](dato)
```

-   Receive: es bloqueante.

```cs
receive canal(variable)
receive arreglo[i](variable)
```

-   Empty: si el canal está o no vacío.

```cs
empty(canal)
empty(arreglo[i])
```

## Notas importantes

-   Cuando un proceso único (un servidor por ej) debe comunicarse con procesos especificos de un arreglo de procesos, es necesario usar **canales privados**.
-   La operación empty debe usarse con cuidado, si se utiliza dentro de un arreglo de procesos se pueden producir errores. Hay que usarla dentro de un único proceso **Coordinador**.
