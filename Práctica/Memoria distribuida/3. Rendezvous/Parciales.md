# Rendezvous

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## Ejercicio 3) - 13 de noviembre, 2023 ✅

La página web del Banco Central exhibe las diferentes cotizaciones del dólar oficial de 20 bancos del país, tanto para la compra como para la venta. Existe una tarea programada que se ocupa de actualizar la página en forma periódica y para ello consulta la cotización de cada uno de los 20 bancos. Cada banco dispone de una API, cuya única función es procesar las solicitudes de aplicaciones externas. La tarea programada consulta de a una API por vez, esperando a lo sumo 5 segundos por su respuesta. Si pasado ese tiempo no respondió, entonces se mostrará vacía la información de ese banco.

```ada
procedure BCRA is
    task type APIBanco is
        entry consultarDolarOficial(compra: OUT string; venta: OUT string)
    end APIBanco
    task TareaProgramada

    APIs: array(1..20) of APIBanco

    task body TareaProgramada is
        compra, venta: string
        cotizaciones: array(1..20) of (string, string)
    begin
        for i = 1 to 20 loop
            select
                APIs(i).consultarDolarOficial(compra, venta)
                cotizaciones(i) := (compra, venta)
            or delay 5
                cotizaciones(i) := (null, null)
            end select
        end loop
    end TareaProgramada

    task body APIBanco is
        cotizacionCompra: string
        cotizacionVenta: string
    begin
        loop
            Accept consultarDolarOficial(compra: OUT string; venta: OUT string) is
                compra:= cotizacionCompra
                venta:= cotizacionVenta
            end consultarDolarOficial
        end loop
    end APIBanco
begin
    null
end BCRA;
```

## - ❓

Simular la venta de entradas a un evento musical por medio de un portal web. Hay N clientes que intentan comprar una entrada para el evento; los clientes pueden ser regulares o especiales (clientes que están asociados al sponsor del evento). Cada cliente especial hace un pedido al portal y espera hasta ser atendido; cada cliente regular hace un pedido y si no es atendido antes de los 5 minutos, vuelve a hacer el pedido siguiendo el mismo patrón (espera a lo sumo 5 minutos y si no lo vuelve a intentar) hasta ser atendido. Después de ser atendido, si consiguió comprar la entrada, debe imprimir el comprobante de la compra. El portal tiene E entradas para vender y atiende los pedidos de acuerdo al orden de llegada pero dando prioridad a los Clientes Especiales. Cuando atiende un pedido, si aún quedan entradas disponibles le vende una al cliente que hizo el pedido y le entrega el comprobante.
**Nota: no debe modelarse la parte de la impresión del comprobante, sólo llamar a una función Imprimir (comprobante) en el cliente que simulará esa parte; la cantidad E de entradas es mucho menor que la cantidad de clientes (T << C); todas las tareas deben terminar.**

```ada
??
```

## - ❓

Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y llevarse su comprobante. Los clientes se dividen entre los regulares y los premium, habiendo R clientes regulares y P clientes premium. Existe un único empleado en el banco, el cual atiende de acuerdo al orden de llegada, pero dando prioridad a los premium sobre los regulares. Si a los 30 minutos de llegar un cliente regular no fue atendido, entonces se retira sin realizar el pago. Los clientes premium siempre esperan hasta ser atendidos.

```ada
procedure Parcial is
    Task Type ClienteRegular;
    Task Type ClientePremium;

    Task Empleado is
        Entry PedidoRegular(pago: IN int; comprobante: OUT string)
        Entry PedidoPremium(pago: IN int; comprobante: OUT string)
    end Empleado

    regulares: array(1..R) of ClienteRegular;
    premiums: array(1..P) of ClientePremium;

    Task Body ClienteRegular is
        pago: int; comprobante: string;
    Begin
        Select
            Empleado.PedidoRegular(pago, comprobante)
        Or delay (30 * 60)
            null
        End select;
    end ClienteRegular

    Task Body ClientePremium is
        pago: int; comprobante: string;
    Begin
        Empleado.PedidoPremium(pago, comprobante)
    end ClientePremium

    Task Body Empleado is
    Begin
        loop
            select
                when (PedidoPremium'count == 0) ->
                    Accept PedidoRegular(pago: IN int; comprobante: OUT string) do
                        comprobante = Pagar(pago)
                    end PedidoRegular
            or
                Accept PedidoPremium(pago: IN int; comprobante: OUT string) do
                    comprobante = Pagar(pago)
                end PedidoPremium
            end select;
        end loop
    end Empleado

Begin
    Null
End Parcial
```

## - ❓

Una empresa de venta de calzado cuenta con S sedes. En la oficina central de la empresa se utiliza un sistema que permite controlar el stock de los diferentes modelos, ya que cada sede tiene una base de datos propia. El sistema de control de stock funciona de la siguiente manera: dado un modelo determinado, lo envía a las sedes para que cada una le devuelva la cantidad disponible en ellas; al final del procesamiento, el sistema informa el total de calzados disponibles de dicho modelo. Una vez que se completó el procesamiento de un modelo, se procede a realizar lo mismo con el siguiente modelo.
**Nota: suponga que existe una función DevolverStock(modelo,cantidad) que utiliza cada sede donde recibe como parámetro de entrada el modelo de calzado y retorna como parámetro de salida la cantidad de pares disponibles. Maximizar la concurrencia y no generar demora innecesaria.**

```ada
Procedure parcial is
    Task type Sede is
    end Sede
    sedes: array(1..S) of Sede

    Task oficinaCentral is
        Entry EnviarModelo(m OUT: modelo)
        Entry RecibirCantidad(cant IN: int)

    end oficinaCentral

    Task Body Sede is
        db: DataBase; m: modelo; cantidad: int
    Begin
        loop
            oficinaCentral.EnviarModelo(m)
            DevolverStock(m, cantidad)
            oficinaCentral.RecibirCantidad(cantidad)
        end loop
    End Sede

    Task Body oficinaCentral is
        m: modelo; cantidadTotal: int
    Begin
        loop
            cantidadTotal := 0
            m := ModeloDeInteres()
            for i := 1 to S loop
                accept EnviarModelo(mod OUT: modelo) do
                    mod := m
                end EnviarModelo
            end loop

            for i := 1 to S loop
                accept RecibirCantidad(cant IN: int) do
                    cantidadTotal += cant
                end RecibirCantidad
            end loop
        end loop
    End oficinaCentral

Begin
    NULL
End parcial
```

## - ❓

Se debe controlar el acceso a una base de datos. Existen L procesos Lectores y E procesos Escritores que trabajan indefinidamente de la siguiente manera:
• Escritor: intenta acceder para escribir, si no lo logra inmediatamente, espera 1 minuto y vuelve a intentarlo de la misma manera.
• Lector: intenta acceder para leer, si no lo logro en 2 minutos, espera 5 minutos y vuelve a intentarlo de la misma manera.
Un proceso Escritor podrá acceder si no hay ningún otro proceso usando la base de datos; al acceder escribe y sale de la BD. Un proceso Lector podrá acceder si no hay procesos Escritores usando la base de datos; al acceder lee y sale de la BD. Siempre se le debe dar prioridad al pedido de acceso para escribir sobre el pedido de acceso para leer.

```ada
Procedure Parcial is
    Task Type Escritor
    escritores: array(1..E) of Escritor
    Task Body Escritor is
    Begin
        loop
            select
                AdministradorBD.PedirEscribir()
                // escribe
                AdministradorBD.SalirEscritor()
            else
                delay 60
            end select
        end loop
    End Escritor

    Task Type Lector
    lectores: array(1..L) of Lector
    Task Body Lector is
    Begin
        loop
            select
                AdministradorBD.PedirLeer()
                // lee
                AdministradorBD.SalirLector()
            or delay 120
                delay 300
            end select
        end loop
    End Lector

    Task AdministradorBD is
        Entry PedirLeer()
        Entry PedirEscribir()
        Entry SalirLector()
        Entry SalirEscritor()
    End AdministradorBD
    Task Body AdministradorBD is
        cantUsando: int
    Begin
        cantUsando := 0
        loop
            select
                when (cantUsando == 0) -> {
                    Accept PedirEscribir()
                    Accept SalirEscritor()
                }
            or
                when (PedirEscribir'count == 0) -> {
                    Accept PedirLeer()
                    cantUsando++
                }
            or
                Accept SalirLector()
                cantUsando--
            end select
        end loop
    End AdministradorBD
Begin

End Parcial
```
