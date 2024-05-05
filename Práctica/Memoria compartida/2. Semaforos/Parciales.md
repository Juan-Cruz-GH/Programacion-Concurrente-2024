# Semáforos

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## Ejercicio 1) - 18 de diciembre, 2023 ✅

Se debe simular el uso de una máquina expendedora de gaseosas
con capacidad para 100 latas por parte de U usuarios. Además existe un repositor encargado de reponer las latas de la máquina. Los usuarios usan la máquina según el orden de llegada. Cuando les toca usarla, sacan una lata y luego se retiran. En el caso de que la máquina se quede sin latas, entonces le debe avisar al repositor para que cargue nuevamente la máquina en forma completa. Luego de la recarga, saca una lata y se retira.
**Nota: maximizar la concurrencia; mientras se reponen las latas se debe permitir que otros usuarios puedan agregarse a la fila.**

```cs
Cola maquina(Gaseosa)(100)
Cola usuarios(Usuario)
bool libre = true
sem mutexMaquina = 1
sem seVacio = 0
sem maquinaRecargada = 0
sem esperando[U] = ([U] 0)

process Usuario [id: 0..U-1] {
    P(mutexMaquina)
    if libre {
        libre = false
        V(mutexMaquina)
    }
    else {
        usuarios.push(id)
        V(mutexMaquina)
        P(esperando[id])
    }
    if maquina.empty() {
        V(seVacio)
        P(maquinaRecargada)
    }
    Gaseosa g = maquina.pop()
    P(mutexMaquina)
    if usuarios.empty()
        libre = true
    else
        V(esperando[usuarios.pop])
    V(mutexMaquina)
}
process Repositor {
    Cola maquinaRepuesta(Gaseosa)(100)
    while true {
        P(seVacio)
        maquina = maquinaRepuesta
        V(maquinaRecargada)
    }
}
```

## Ejercicio 1) - 4 de diciembre, 2023 ✅

Un sistema debe validar un conjunto de 10000 transacciones que se encuentran disponibles en una estructura de datos. Para ello, el sistema dispone de 7 workers, los cuales trabajan colaborativamente validando de a 1 transacción por vez cada uno. Cada validación puede tomar un tiempo diferente y para realizarla los workers disponen de la función Validar(t), la cual retorna como resultado un número entero entre 0 al 9. Al finalizar el procesamiento, el último worker en terminar debe informar la cantidad de transacciones por cada resultado de la función de validación.
**Nota: maximizar la concurrencia.**

```cs
Cola transacciones(10000)(Transaccion)
int resultados[10] = {[10] 0}
sem mutexRes[10] = 1
sem mutexCola = 1
sem mutexFin = 1
sem barrera = 0
int cantTerminaron = 0
process Worker [id: 1..7] {
    Transaccion t
    int res
    P(mutexCola)
    while transacciones.notEmpty() {
        t = transacciones.pop()
        V(mutexCola)

        res = Validar(t)
        P(mutexRes[res])
        resultados[res]++
        V(mutexRes[res])

        P(mutexCola)
    }
    V(mutexCola)

    P(mutexFin)
    cantTerminaron++
    if cantTerminaron == 7 {    // el último proceso
        for i=0 to 9
            print(resultados[i])
        for i=1 to 6
            V(barrera)   // despertar a los otros 6
        V(mutexFin)
    }
    V(mutexFin)
    P(barrera)
}
```

## Ejercicio 1) a) - 10 de octubre, 2023 ✅

En una estación de trenes, asisten P personas que deben realizar una carga de su tarjeta SUBE en la terminal disponible. La terminal es utilizada en forma exclusiva por cada persona de acuerdo con el orden de llegada. Implemente una solución utilizando sólo emplee procesos Persona.
**Nota: la función UsarTerminal() le permite cargar la SUBE en la terminal disponible.**

```cs
sem mutex = 1
sem esperar[P] = {[P] 0}
Cola personas
bool libre = true

process Persona [id: 0..P-1] {
    P(mutex)
    if libre {
        libre = false
        V(mutex)
    }
    else {
        personas.push(id)
        V(mutex)
        P(esperar[id])
    }
    UsarTerminal()
    P(mutex)
    if personas.empty()
        libre = true
    else
        V(esperar[personas.pop()])
    V(mutex)
}
```

## Ejercicio 1) b) - 10 de octubre, 2023 ✅

Resuelva el mismo problema anterior pero ahora considerando que hay T terminales disponibles. Las personas realizan una única fila y la carga la realizan en la primera terminal que se libera. Recuerde que sólo debe emplear procesos Persona.
**Nota: la función UsarTerminal(t) le permite cargar la SUBE en la terminal t**

```cs
sem s = 1
sem s_cola = 1
sem esperar[P] = {[P] 0}
int libres = T
Cola personas
Cola terminales

process Persona [id: 0..P-1] {
    P(s)
    if libres == 0 {
        personas.push(id)
        V(s)
        P(esperar[id])
    }
    else {
        libres--
        V(s)
    }
    P(s_cola)
    t = terminales.pop()
    V(s_cola)

    UsarTerminal(t)

    P(s_cola)
    terminales.push(t)
    V(s_cola)
    P(s)
    if personas.isNotEmpty()
        V(esperar[personas.pop()])
    else
        libres++
    V(s)
}
```

## Ejercicio 1) - Segundo recuperatorio 2022 ✅

En una planta verificadora de vehículos, existen 7 estaciones donde se dirigen 150 vehículos para ser verificados. Cuando un vehículo llega a la planta, el coordinador de la planta le indica a qué estación debe dirigirse. El coordinador selecciona la estación que tenga menos vehículos asignados en ese momento. Una vez que el vehículo sabe qué estación le fue asignada, se dirige a la misma y espera a que lo llamen para verificar. Luego de la revisión, la estación le entrega un comprobante que indica si pasó la revisión o no. Más allá del resultado, el vehículo se retira de la planta.
**Nota: maximizar la concurrencia.**

```cs
int cant_estaciones [7] = {0} [7]
int estaciones_asignadas [150]
sem cont_estaciones = 1
sem entrada = 1
sem atencion_entrada = 0
sem espera_estacion[150] {0} [150]
sem atencion_verificacion[7] = {0} [7]
sem s_estacion[7] = {1} [7]
sem espera_resultado[150] = {0} [150]
string resultado [150]
Cola atencion_entrada, estacion [7]

Process Vehículo [id: 1..150] {
    P(entrada)
    push (atencion_entrada, i) // encola su ID para que lo atienda el coordinador
    V(entrada)

    V(sem atencion_entrada) // avisa para que lo atienda el coordinador
    P(sem espera_estacion[i]) // espera estacion
    mi_estacion = estaciones_asignadas [i] // copia

    P(s_estacion[mi_estacion])
    push (estacion [mi_estacion], i) // encola su ID para que lo atiendan la estación
    V(s_estacion[mi_estacion])

    V(atencion_verificacion [mi_estacion]) // avisa para que lo atiendan en la estación
    P(espera_resultado[i]) // espera resultado
    P(cont_estaciones)
    cant_estaciones[mi_estacion]-- // decrementa contador para mantener valor actualizado
    v(cont_estaciones)
}
Process Entrada {
    int id_min_estacion, id
    while true {
        P(atencion_entrada) // espera pedido de atencion
        P(entrada)
        id = atencion_entrada.pop() // desencola pedido con exclusion mutua
        V(entrada)

        P(cont_estaciones)
        id_min_estacion = min(cant_estaciones)  // posicion de la cantidad minima
        cant_estaciones[id_min_estacion]++  // incrementa para actualizar celda
        V(cont_estaciones)

        estaciones_asignadas[id] = id_min_estacion // asigna estacion
        V(espera_estacion[id])
    }
}
Process Estacion [id: 1..7] {
    int id
    while true {
        P(atencion_verificacion[id]) // se bloquea a la espera de que haya vehiculos

        P(s_estacion[id])
        id = estacion[id].pop() // desencola vehiculo
        V(s_estacion[id])

        resultado[id] = verificar(id)   // verificar

        V(espera_resultado[id]) // le avisa que ya esta el resultado disponible
    }
}
```

## Ejercicio 1) - Primer recuperatorio 2022 ❓

En un restorán trabajan C cocineros y M mozos. De forma repetida, los cocineros preparan un plato y lo dejan listo en la bandeja de platos terminados, mientras que los mozos toman los platos de esta bandeja para repartirlos entre los comensales. Tanto los cocineros como los mozos trabajan de a un plato por vez. Modele el funcionamiento del restorán considerando que la bandeja de platos listos puede almacenar hasta P platos. No es necesario modelar a los comensales ni que los procesos terminen.

```cs
Plato bandeja[P]
int posC, posM = 0
sem tamañoBandeja = P
sem preparado = 0
sem mutexC, mutexM = 1

process Cocinero [id: 0..C-1] {
    Plato plato
	while true {
		plato = Preparar()
		P(tamañoBandeja)    // Espera que haya espacio en la bandeja
		P(mutexC)
		bandeja[posC] = plato   // Pone el plato en la bandeja
		posC = (posC + 1) mod P // Pasa a la siguiente posición de la bandeja
		V(mutexC)
		V(preparado)    // Avisa que hay un plato nuevo en la bandeja
	}
}
process Mozo [id: 0..M-1] {
    Plato plato
    while true {
        P(preparado)    // Espera que haya plato en la bandeja
        P(mutexM)
        plato = bandeja[posM]   // Saca plato de la bandeja
        posM = (posM + 1) mod P // Pasa a la siguiente posición de la bandeja
        V(mutexM)
        V(tamañoBandeja)
        Entregar(plato)
    }
}
```

## - ❓

Existen 15 sensores de temperatura y 2 módulos centrales de procesamiento. Un sensor mide la temperatura cada cierto tiempo (función medir()), la envía al módulo central para que le indique qué acción debe hacer (un número del 1 al 10) (función determinar() para el módulo central) y la hace (función realizar()). Los módulos atienden las mediciones por orden de llegada.

```cs
sem mutex = 1
sem tempEnviada = 0
sem esperarAccion[15] = ([15] 0)
cola temps(int)
int miAccion[15]
process Sensor [id: 0..14] {
	while true {
		int temperatura = medir()
		P(mutex)
		temps.push(id, temperatura)
		V(mutex)
		V(tempEnviada)
        P(esperarAccion[id])
        realizar(miAccion[id])
	}
}

process Central [id: 0..1] {
    int accion, idSensor
	while true {
		P(tempEnviada)
		P(mutex)
		idSensor, temperatura = temps.pop()
        V(mutex)
		accion = determinar(temperatura)
        miAccion[idSensor] = accion
        V(esperarAccion[idSensor])
	}
}
```

## - ❓

Simular un exámen técnico para concursos Nodocentes en la Facultad, en el mismo participan 100 personas distribuidas en 4 concursos con un coordinador en cada una de ellos. Cada persona sabe en qué concurso participa. El coordinador de cada concurso espera hasta que lleguen las 25 personas correspondientes al mismo, les entrega el exámen a resolver (el mismo para todos los de ese concurso) y luego corrige los exámenes de esas 25 personas de acuerdo al orden en que van entregando. Cada persona al llegar debe esperar a que su coordinador (el de su concurso) le dé el exámen, lo resuelve, lo entrega para que su coordinador lo evalúe y espera hasta que le deje la nota para luego retirarse.
**Nota: maximizar la concurrencia; sólo usar procesos que representen a las personas y coordinadores; todos los procesos deben terminar.**

```cs
??
```

## - ❓

Un banco decide entregar promociones a sus clientes por medio de su agente de prensa, el cual lo hace de la siguiente manera:el agente debe entregar 50 premios entre los 1000 clientes,para esto, obtiene al azar un número de cliente y le entrega el premio, una vez que este lo toma continúa con la entrega.
**Notas: Cuando los 50 premios han sido entregados el agente y los clientes terminan su ejecución; No se puede utilizar una estructura de tipo arreglo para almacenar los premios de los clientes.**

```cs
??
```

## - ❓

En una empresa hay UN Coordinador y 30 Empleados que formarán 3 grupos de 10 empleados cada uno. Cada grupo trabaja en una sección diferente y debe realizar 345 unidades de un producto. Cada empleado al llegar se dirige al coordinador para que le indique el número de grupo al que pertenece y una vez que conoce este dato comienza a trabajar hasta que se han terminado de hacer las 345 unidades correspondientes al grupo (cada unidad es hecha por un único empleado). Al terminar de hacer las 345 unidades los 10 empleados del grupo se deben juntar para retirarse todos juntos. El coordinador debe atender a los empleados de acuerdo al orden de llegada para darle el número de grupo (a los 10 primeros que lleguen se le asigna el grupo 1, a los 10 del medio el 2, y a los 10 últimos el 3). Cuando todos los grupos terminaron de trabajar el coordinador debe informar (imprimir en pantalla) el empleado que más unidades ha realizado (si hubiese más de uno con la misma cantidad máxima debe informarlos a todos ellos).
**Nota: maximizar la concurrencia; suponga que existe una función Generar() que simula la elaboración de una unidad de un producto.**

```cs
??
```

## - ❓

En una fábrica de muebles trabajan 50 empleados. A llegar, los empleados forman 10 grupos de 5 personas cada uno, de acuerdo al orden de llegada (los 5 primeros en llegar forman el primer grupo, los 5 siguientes el segundo grupo, y así sucesivamente). Cuando un grupo se ha terminado de formar, todos sus integrantes se ponen a trabajar. Cada grupo debe armar M muebles (cada mueble es armado por un solo empleado); mientras haya muebles por armar en el grupo los empleados los irán resolviendo (cada mueble es armado por un solo empleado).
**Nota: Cada empleado puede tardar distinto tiempo en armar un mueble. Sólo se pueden usar los procesos “Empleado”, y todos deben terminar su ejecución. Maximizar la concurrencia.**

```cs
int empleados = 0
int grupos = 0
int muebles[10] = {[10] 0}
sem s_reunion[10] = {[10] 0}
sem s_muebles[10] = {[10] 1}
sem s_llegada = 1

process Empleado [id: 0..49] {
    int grupo
    P(s_llegada)
    grupo = grupos
    empleados++
    if empleados < 5 {
        V(s_llegada)
        P(s_reunion[grupo])
    }
    else {
        for i = 1 to 4 do
            V(s_reunion[grupo])
        empleados = 0
        grupo++
        V(s_llegada)
    }
    // Trabajar
    P(s_muebles[grupo])
    while (muebles[grupo] < M) {
        muebles[grupo]++
        V(s_muebles[grupo])
        // Armar mueble
        sleep(rand())
        P(s_muebles[grupo])
    }
    V(s_muebles[grupo])
}

```
