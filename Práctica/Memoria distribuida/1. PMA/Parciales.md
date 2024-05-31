# Pasaje de Mensajes Asincrónico

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## Ejercicio 1) - 13 de noviembre, 2023 ✅

En una oficina existen 100 empleados que envían documentos para imprimir en 5 impresoras compartidas. Los pedidos de impresión son procesados por orden de llegada y se asignan a la primera impresora que se encuentre libre.

```cs
chan solicitudImpresion(int, string)
chan miDocumentoImpreso[N](string)

process Empleado[id: 0..99] {
    string doc, docImpreso
    while true {
        doc = Generar()
        send solicitudImpresion(id, doc)
        receive miDocumentoImpreso[id](docImpreso)
    }
}

process Impresora[id: 0..4] {
    string doc, docImpreso
    int idE
    while true {
        receive solicitudImpresion(idE, doc)
        docImpreso = Imprimir(doc)
        send miDocumentoImpreso[idE](docImpreso)
    }
}
```

## - ❓

Se debe modelar el funcionamiento de una casa de venta de repuestos automotores, en la que trabajan V vendedores y que debe atender a C clientes. El modelado debe considerar que: (a) cada cliente realiza un pedido y luego espera a que se lo entreguen; y (b) los pedidos que hacen los clientes son tomados por cualquiera de los vendedores. Cuando no hay pedidos para atender, los vendedores aprovechan para controlar el stock de los repuestos (tardan entre 2 y 4 minutos para hacer esto).
**Nota: maximizar la concurrencia**

```cs
chan vendedoresLibres(int)
chan pedidosClientes(string, int) // el pedido y el id del cliente
chan esperarRepuesto[C](string)
chan pedidosVendedores[V](string, int)

process Cliente [id: 0..C-1] {
    string pedido, repuesto;
    send pedidosClientes(pedido, id)
    receive esperarRepuesto[id](repuesto)
}
process Vendedor [id: 0..V-1] {
    int idCliente, segundos;
    string pedido, repuesto;
    while true {
        send vendedoresLibres(id)
        receive pedidosVendedores[id](pedido, idCliente)
        if pedido == "" {
            segundos = rand(2,4) * 60
            delay(segundos)
        }
        else {
            repuesto = resolver(pedido)
            send esperarRepuesto[idCliente](repuesto)
        }
    }
}
process Coordinador {
    int idCliente, idVendedor;
    string pedido;
    while true {
        receive vendedoresLibres(idVendedor)
        if empty(pedidosClientes) {
            send pedidosVendedores[idVendedor]("", -1)
        }
        else {
            receive pedidosClientes(pedido, idCliente)
            send pedidosVendedores[idVendedor](pedido, idCliente)
        }
    }
}
```

## - ❓

Se debe simular la atención en un banco con 3 cajas para atender a N clientes que pueden ser especiales (son las embarazadas y los ancianos) o regulares. Cuando el cliente llega al banco se dirige a la caja con menos personas esperando y se queda ahí hasta que lo terminan de atender y le dan el comprobante de pago. Las cajas atienden a las personas que van a ella de acuerdo al orden de llegada pero dando prioridad a los clientes especiales; cuando terminan de atender a un cliente le debe entregar un comprobante de pago.
**Nota: maximizar la concurrencia.**

```cs
??
```
