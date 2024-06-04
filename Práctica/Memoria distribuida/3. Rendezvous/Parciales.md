<h1 align="center">Rendezvous</h1>

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## Ejercicio 2) - 18 de diciembre, 2023 ✅

La oficina central de una empresa de venta de indumentaria debe calcular cuántas veces fue vendido cada uno de los artículos de su catálogo. La empresa se compone de 100 sucursales y cada una de ellas maneja su propia base de datos de ventas. La oficina central cuenta con una herramienta que funciona de la siguiente manera: ante la consulta realizada para un artículo determinado, la herramienta envía el identificador del artículo a cada una de las sucursales, para que cada uno de éstas calcule cuántas veces fue vendido en ella. Al final del procesamiento, la herramienta debe conocer cuántas veces fue vendido en total, considerando todas las sucursales. Cuando ha terminado de procesar un artículo comienza con el siguiente (suponga que la herramienta tiene una función generarArtículo que retorna el siguiente ID a consultar).
**Nota: maximizar la concurrencia. Supongo que existe una función ObtenerVentas(ID) que retorna la cantidad de veces que fue vendido el artículo con identificador ID en la base de datos de la sucursal que la llama.**

```ada
procedure parcial is
    task type Sucursal
    sucursales: array(1..100) of Sucursal

    task body Sucursal is
        idA: string
        cantidad: int
    begin
        loop
            Herramienta.SiguienteArticulo(idA)
            cantidad:= ObtenerVentas(idA)
            Herramienta.RecibirResultado(cantidad)
        end loop
    end Sucursal

    task Herramienta is
        Entry SiguienteArticulo(idArt: OUT string)
        Entry RecibirResultado(cuantasVeces: IN int)
    end Herramienta

    task body Herramienta is
        total: int
        idArticulo: string
    begin
        loop
            idArticulo:= GenerarArticulo()
            total:= 0
            for i=1 to 100 loop
                Accept SiguienteArticulo(idArt: OUT string) is
                    idArt:= idArticulo
                end SiguienteArticulo
            end loop
            for i=1 to 100 loop
                Accept RecibirResultado(cuantasVeces: IN int) is
                    total:= total + cuantasVeces
                end RecibirResultado
            end loop
            print(total)
        end loop
    end Herramienta
begin
    null
end parcial
```

## Ejercicio 2) - 4 de diciembre, 2023 ✅

En un negocio de cobros digitales hay P personas que deben pasar por la única caja de cobros para realizar el pago de sus boletas. Las personas son atendidas de acuerdo con el orden de llegada, teniendo prioridad aquellos que deben pagar menos de 5 boletas de los que pagan más. Adicionalmente, las personas embarazadas y los ancianos tienen prioridad sobre los dos casos anteriores. Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les devuelve el vuelto y los recibos de pago.

```ada
procedure parcial is
    task type Persona

    personas: array(1..P) of Persona

    task Caja is
        Entry PedidosAncianoEmbarazada(b: IN string: p: IN int; v: OUT int; r: OUT int)
        Entry PedidosMenosDe5(b: IN string: p: IN int; v: OUT int; r: OUT int)
        Entry PedidosNormales(b: IN string: p: IN int; v: OUT int; r: OUT int)
    end Caja

    task body Persona is
        cantBoletas: int
        soyAnciano: boolean
        soyEmbarazada: boolean
        boletas, recibo: string
        pago, vuelto: int
    begin
        if soyAnciano or soyEmbarazada then
            Caja.PedidosAncianoEmbarazada(boletas, pago, vuelto, recibo)
        else if cantBoletas < 5
            Caja.PedidosMenosDe5(boletas, pago, vuelto, recibo)
        else
            Caja.PedidosNormales(boletas, pago, vuelto, recibo)
        end if
    end Persona

    task body Caja is

    begin
        loop
            select
                when (PedidosAncianoEmbarazada'count = 0) and (PedidosMenosDe5'count = 0) ->
                    Accept PedidosNormales(b: IN string: p: IN int; v: OUT int; r: OUT int) is
                        v, r:= Procesar(b, p)
                    end PedidosNormales
            or
                when (PedidosAncianoEmbarazada'count = 0) ->
                    Accept PedidosMenosDe5(b: IN string: p: IN int; v: OUT int; r: OUT int) is
                        v, r:= Procesar(b, p)
                    end PedidosMenosDe5
            or
                Accept PedidosAncianoEmbarazada(b: IN string: p: IN int; v: OUT int; r: OUT int) is
                    v, r:= Procesar(b, p)
                end PedidosAncianoEmbarazada
            end select
        end loop
    end Caja
begin
    null
end parcial
```

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

## - ✅

Simular la venta de entradas a un evento musical por medio de un portal web. Hay N clientes que intentan comprar una entrada para el evento; los clientes pueden ser regulares o especiales (clientes que están asociados al sponsor del evento). Cada cliente especial hace un pedido al portal y espera hasta ser atendido; cada cliente regular hace un pedido y si no es atendido antes de los 5 minutos, vuelve a hacer el pedido siguiendo el mismo patrón (espera a lo sumo 5 minutos y si no lo vuelve a intentar) hasta ser atendido. Después de ser atendido, si consiguió comprar la entrada, debe imprimir el comprobante de la compra. El portal tiene E entradas para vender y atiende los pedidos de acuerdo al orden de llegada pero dando prioridad a los Clientes Especiales. Cuando atiende un pedido, si aún quedan entradas disponibles le vende una al cliente que hizo el pedido y le entrega el comprobante.
**Nota: no debe modelarse la parte de la impresión del comprobante, sólo llamar a una función Imprimir (comprobante) en el cliente que simulará esa parte; la cantidad E de entradas es mucho menor que la cantidad de clientes (T << C); todas las tareas deben terminar.**

```ada
procedure parcial is
    task type ClienteRegular
    task type ClienteEspecial

    task Portal is
        Entry PedidosRegular(e: OUT string; c: OUT string)
        Entry PedidosEspecial(e: OUT string; c: OUT string)
    end Portal

    clientesRegulares: array(1..N/2) of ClienteRegular
    clientesEspeciales: array(1..N/2) of ClienteEspecial

    task body ClienteRegular is
        entrada, comprobante: string
        meVoy: bool := false
    begin
        while not meVoy loop
            select
                Portal.PedidosRegular(entrada, comprobante)
                meVoy := true
                if entrada != ""
                    Imprimir(comprobante)
            or delay 60 * 5
                null
            end select
        end loop
    end Cliente

    task body ClienteEspecial is
        entrada, comprobante: string
    begin
        Portal.PedidosEspecial(entrada, comprobante)
        if entrada != ""
            Imprimir(comprobante)
    end Cliente

    task body Portal is
        entradas: cola(string)(E)
    begin
        for i=1 to N loop
            select
                when PedidosEspecial'count = 0 ->
                    Accept PedidosRegular(e: OUT string; c: OUT string) is
                        if entradas.empty() then
                            e := ""
                            c := ""
                        else
                            e := entradas.pop()
                            c := Generar(e)
                    end PedidosRegular
            or
                Accept PedidosEspecial(e: OUT string; c: OUT string) is
                        if entradas.empty() then
                            e := ""
                            c := ""
                        else
                            e := entradas.pop()
                            c := Generar(e)
                end PedidosEspecial
            end select
        end loop
    end Portal
begin
    null
end parcial
```

## - ✅

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

## - ✅

Una empresa de venta de calzado cuenta con S sedes. En la oficina central de la empresa se utiliza un sistema que permite controlar el stock de los diferentes modelos, ya que cada sede tiene una base de datos propia. El sistema de control de stock funciona de la siguiente manera: dado un modelo determinado, lo envía a las sedes para que cada una le devuelva la cantidad disponible en ellas; al final del procesamiento, el sistema informa el total de calzados disponibles de dicho modelo. Una vez que se completó el procesamiento de un modelo, se procede a realizar lo mismo con el siguiente modelo.
**Nota: suponga que existe una función DevolverStock(modelo,cantidad) que utiliza cada sede donde recibe como parámetro de entrada el modelo de calzado y retorna como parámetro de salida la cantidad de pares disponibles. Maximizar la concurrencia y no generar demora innecesaria.**

```ada
procedure parcial is
    task type Sede
    sedes: array(1..S) of Sede

    task oficinaCentral is
        Entry EnviarModelo(m: OUT string)
        Entry RecibirCantidad(c: IN int)
    end oficinaCentral

    task body Sede is
        m: string
        cantidad: int
    begin
        loop
            oficinaCentral.EnviarModelo(m)
            DevolverStock(m, cantidad)
            oficinaCentral.RecibirCantidad(cantidad)
        end loop
    end Sede

    task body oficinaCentral is
        modelo: string
        cantidadTotal: int
    begin
        loop
            cantidadTotal := 0
            modelo := SiguienteModelo()
            for i=1 to S * 2 loop
                select
                    Accept EnviarModelo(m: OUT) is
                        m:= modelo
                    end EnviarModelo
                or
                    Accept RecibirCantidad(c: IN int) is
                        cantidadTotal:= cantidadTotal + c
                    end RecibirCantidad
                end select
            end loop
            print(cantidadTotal)
        end loop
    end oficinaCentral
begin
    null
end parcial
```

## - ❓

Se debe controlar el acceso a una base de datos. Existen L procesos Lectores y E procesos Escritores que trabajan indefinidamente de la siguiente manera:
• Escritor: intenta acceder para escribir, si no lo logra inmediatamente, espera 1 minuto y vuelve a intentarlo de la misma manera.
• Lector: intenta acceder para leer, si no lo logro en 2 minutos, espera 5 minutos y vuelve a intentarlo de la misma manera.
Un proceso Escritor podrá acceder si no hay ningún otro proceso usando la base de datos; al acceder escribe y sale de la BD. Un proceso Lector podrá acceder si no hay procesos Escritores usando la base de datos; al acceder lee y sale de la BD. Siempre se le debe dar prioridad al pedido de acceso para escribir sobre el pedido de acceso para leer.

```ada
procedure parcial is
    task type Escritor
    task type Lector
    escritores: array(1..E) of Escritor
    lectores: array(1..L) of Lector

    task AdministradorBD is
        Entry PedirLector()
        Entry PedirEscritor()
        Entry SalirLector()
        Entry SalirEscritor()
    end AdministradorBD

    task body Escritor is
    begin
        loop
            select
                AdministradorBD.PedirEscritor()
                -- Escribe en la DB
                AdministradorBD.SalirEscritor()
            else
                delay 60
            end select
        end loop
    end Escritor

    task body Lector is
    begin
        loop
            select
                AdministradorBD.PedirLector()
                -- Lee en la DB
                AdministradorBD.SalirLector()
            or delay 60 * 2
                delay 60 * 5
            end select
        end loop
    end Lector

    task body AdministradorBD is
        cantUsando: int := 0
    begin
        loop
            select
                when (cantUsando == 0) -> {
                    Accept PedirEscritor()
                    Accept SalirEscritor()
                }
            or
                when (PedirEscritor'count == 0) -> {
                    Accept PedirLector()
                    cantUsando++
                }
            or
                Accept SalirLector()
                cantUsando--
            end select
        end loop
    end AdministradorBD
begin
    null
end parcial
```

## - ❓

Hay un sitio web para identificación genética que resuelve pedidos de N clientes. Cada cliente trabaja continuamente de la siguiente manera: genera la secuencia de ADN, la envía al sitio web para evaluar, y espera el resultado; después de esto puede comenzar a generar la siguiente secuencia de ADN. Para resolver estos pedidos el sitio web cuenta con 5 servidores idénticos que atienden los pedidos de acuerdo al orden de llegada (cada pedido es atendido por un único servidor).
**Nota: maximizar la concurrencia. Suponga que los servidores tienen una función ResolverAnálisis que recibe la secuencia de ADN y devuelve un entero con el resultado.**

```ada
procedure parcial is
    task type Cliente is
        Entry MiID(i: IN int)
        Entry MiResultado(r: IN int)
    end Cliente

    task type Servidor

    task SitioWeb is
        Entry Pedidos(idC: IN int; s: IN ADN)
        Entry SiguientePedido(idC: OUT int; s: OUT ADN)
    end SitioWeb

    clientes: array(1..N) of Cliente
    servidores: array(1..5) of Servidor

    task body Cliente is
        secuencia: ADN; resultado, id: int;
    begin
        Accept MiID(i: IN int) is
            id:= i
        end MiID
        loop
            secuencia:= GenerarSecuencia()
            SitioWeb.Pedidos(id, secuencia)
            Accept MiResultado(r: IN int) is
                resultado:= r
            end MiResultado
        end loop
    end Cliente

    task body Servidor is
        idCliente, resultado: int; secuencia: ADN;
    begin
        loop
            SitioWeb.SiguientePedido(idCliente, secuencia)
            resultado:= ResolverAnalisis(secuencia)
            clientes(idCliente).MiResultado(resultado)
        end loop
    end Servidor

    task body SitioWeb is
        idCliente: int; secuencia: ADN
    begin
        loop
            Accept Pedidos(idC: IN int; s: IN ADN) is
                idCliente:= idC
                secuencia:= s
            end SiguienteServidor
            Accept SiguientePedido(idC: OUT int; s: OUT ADN) is
                idC:= idCliente
                s:= secuencia
            end ServidorDisponible
        end loop
    end SitioWeb

begin
    for i=1 to N loop
        clientes(i).MiID(i)
    end loop
end parcial
```
