# Semáforos

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## Ejercicio 1) - 18 de diciembre, 2023 ✅

Se debe simular el uso de una máquina expendedora de gaseosas
con capacidad para 100 latas por parte de U usuarios. Además existe un repositor encargado de reponer las latas de la máquina. Los usuarios usan la máquina según el orden de llegada. Cuando les toca usarla, sacan una lata y luego se retiran. En el caso de que la máquina se quede sin latas, entonces le debe avisar al repositor para que cargue nuevamente la máquina en forma completa. Luego de la recarga, saca una lata y se retira.
**Nota: maximizar la concurrencia; mientras se reponen las latas se debe permitir que otros usuarios puedan agregarse a la fila.**

```cs
sem mutexMaquina = 1
bool libre = true
Cola usuarios(Usuario)
sem esperando[U] = ([U] 0)

Cola maquina(Gaseosa)(100)
sem seVacio = 0
sem maquinaRecargada = 0

process Usuario[id: 0..U-1] {
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
sem mutexCola = 1
Cola transacciones(10000)(Transaccion)

int resultados[10] = {[10] 0}
sem mutexRes[10] = 1

sem mutexFin = 1
int cantTerminaron = 0
sem barrera = 0

process Worker[id: 1..7] {
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
        for i=1 to 7
            V(barrera)   // despertar a los demás
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
bool libre = true
Cola personas
sem esperar[P] = {[P] 0}

process Persona[id: 0..P-1] {
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
sem mutex = 1
int libres = T
Cola personas
sem esperar[P] = {[P] 0}

sem mutexTerminales = 1
Cola terminales(Terminal)

process Persona[id: 0..P-1] {
    Terminal t
    P(mutex)
    if libres != 0 {
        libres--
        V(mutex)
    }
    else {
        personas.push(id)
        V(mutex)
        P(esperar[id])
    }
    P(mutexTerminales)
    t = terminales.pop()
    V(mutexTerminales)

    UsarTerminal(t)

    P(mutexTerminales)
    terminales.push(t)
    V(mutexTerminales)
    P(mutex)
    if personas.isEmpty()
        libres++
    else
        V(esperar[personas.pop()])
    V(mutex)
}
```

## Ejercicio 1) - Segundo recuperatorio 2022 ✅

En una planta verificadora de vehículos, existen 7 estaciones donde se dirigen 150 vehículos para ser verificados. Cuando un vehículo llega a la planta, el coordinador de la planta le indica a qué estación debe dirigirse. El coordinador selecciona la estación que tenga menos vehículos asignados en ese momento. Una vez que el vehículo sabe qué estación le fue asignada, se dirige a la misma y espera a que lo llamen para verificar. Luego de la revisión, la estación le entrega un comprobante que indica si pasó la revisión o no. Más allá del resultado, el vehículo se retira de la planta.
**Nota: maximizar la concurrencia.**

```cs
sem mutex = 1
Cola llegada(int)
sem llegue = 0

sem esperarEstacion[150] ([150] 0)
int estacionesAsignadas[150] = ([150] 0)

sem mutexAutos = 1
int autosPorEstacion[7] = ([7] 0)

sem mutexEstaciones[7] = ([7] 1)
Cola estaciones(int)[7]

sem esperarAuto[7] = ([7] 0)

sem esperarResultado[150] = ([150] 0)
string comprobantes[150]

Process Vehículo[id: 1..150] {
    string comprobante
    P(mutex)
    llegada.push(id)// encola su ID para que lo atienda el coordinador
    V(mutex)
    V(llegue) // avisa para que lo atienda el coordinador

    P(esperarEstacion[id])
    miEstacion = estacionesAsignadas[id]

    P(mutexEstaciones[miEstacion])
    estaciones[miEstacion].push(id) // encola su ID para que lo atiendan la estación
    V(mutexEstaciones[miEstacion])

    V(esperarAuto[miEstacion]) // avisa para que lo atiendan en la estación
    P(esperarResultado[id]) // espera resultado
    comprobante = comprobantes[id]

    P(mutexAutos)
    autosPorEstacion[miEstacion]-- // decrementa contador para mantener valor actualizado
    v(mutexAutos)
}

Process Entrada {
    int estacionMin, id
    while true {
        P(llegue) // espera pedido de atencion
        P(mutex)
        id = llegada.pop() // desencola pedido con exclusion mutua
        V(mutex)

        P(mutexAutos)
        estacionMin = min(autosPorEstacion)  // posicion de la cantidad minima
        autosPorEstacion[estacionMin]++  // incrementa para actualizar celda
        V(mutexAutos)

        estacionesAsignadas[id] = estacionMin // asigna estacion
        V(esperarEstacion[id])
    }
}

Process Estacion[id: 1..7] {
    int idAuto
    while true {
        P(esperarAuto[id]) // se bloquea a la espera de que haya vehiculos

        P(mutexEstaciones[id])
        idAuto = estaciones[id].pop() // desencola vehiculo
        V(mutexEstaciones[id])

        comprobantes[id] = verificar(idAuto)   // verificar

        V(esperarResultado[id]) // le avisa que ya esta el resultado disponible
    }
}
```

## Ejercicio 1) - Primer recuperatorio 2022 ✅

En un restorán trabajan C cocineros y M mozos. De forma repetida, los cocineros preparan un plato y lo dejan listo en la bandeja de platos terminados, mientras que los mozos toman los platos de esta bandeja para repartirlos entre los comensales. Tanto los cocineros como los mozos trabajan de a un plato por vez. Modele el funcionamiento del restorán considerando que la bandeja de platos listos puede almacenar hasta P platos. No es necesario modelar a los comensales ni que los procesos terminen.

```cs
Plato bandeja[P]
int posC, posM = 0
sem tamañoBandeja = P
sem preparado = 0
sem mutexC, mutexM = 1

process Cocinero[id: 0..C-1] {
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

process Mozo[id: 0..M-1] {
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

## - ✅

Existen 15 sensores de temperatura y 2 módulos centrales de procesamiento. Un sensor mide la temperatura cada cierto tiempo (función medir()), la envía al módulo central para que le indique qué acción debe hacer (un número del 1 al 10) (función determinar() para el módulo central) y la hace (función realizar()). Los módulos atienden las mediciones por orden de llegada.

```cs
sem mutex = 1
cola temps(int)
sem tempEnviada = 0
sem esperarAccion[15] = ([15] 0)
int miAccion[15]

process Sensor[id: 0..14] {
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

## - ✅

Simular un exámen técnico para concursos Nodocentes en la Facultad, en el mismo participan 100 personas distribuidas en 4 concursos con un coordinador en cada una de ellos. Cada persona sabe en qué concurso participa. El coordinador de cada concurso espera hasta que lleguen las 25 personas correspondientes al mismo, les entrega el exámen a resolver (el mismo para todos los de ese concurso) y luego corrige los exámenes de esas 25 personas de acuerdo al orden en que van entregando. Cada persona al llegar debe esperar a que su coordinador (el de su concurso) le dé el exámen, lo resuelve, lo entrega para que su coordinador lo evalúe y espera hasta que le deje la nota para luego retirarse.
**Nota: maximizar la concurrencia; sólo usar procesos que representen a las personas y coordinadores; todos los procesos deben terminar.**

```cs
sem esperarIntegrantes[25] = ([25] 0)
sem esperarExamen[4] = ([4] 0)

string examenes[4]

sem mutexExamen[4] = ([4] 1)
Cola colas[4](string)
sem examenTerminado[4] = ([4] 0)

string notas[100]
sem esperarNota[100] = ([100] 0)

process Persona[id: 0..99] {
    int miConcurso = GetConcurso() // Valor entre 0 y 3
    V(esperarIntegrantes[miConcurso])   // Avisa a su Coordinador que llegó
    P(esperarExamen[miConcurso])    // Espera que su Coordinador ponga el exámen
    string examen = examenes[miConcurso]    // Obtiene su examen
    string examenResuelto = Resolver(examen)    // Lo resuelve

    P(mutexExamen[miConcurso])
    colas[miConcurso].push(examenResuelto, id)  // Le envía a su Coordinador el exámen
    V(mutexExamen[miConcurso])
    V(examenTerminado[miConcurso])

    P(esperarNota[id])  // Espera que su Coordinador lo corriga
    string miNota = notas[id]   // Obtiene su nota
}

process Coordinador [id: 0..3] {
    string examen
    int persona
    for i=0 to 24
        P(esperarIntegrantes[id])   // Espera a los 25 integrantes de su grupo
    examenes[id] = EntregarExamen() // Pone el examen de su grupo
    for i=0 to 24
        V(esperarExamen[id])    // Avisa a su grupo que ya está el exámen

    for i=0 to 24 {
        P(examenTerminado[id])  // Espera que haya examenes terminados
        P(mutexExamen[id])
        examen, persona = colas[id].pop()   // Obtiene el exámen terminado por orden de llegada
        V(mutexExamen[id])
        notas[persona] = Corregir(examen)
        V(esperarNota[persona]) // Le avisa a la persona que ya está su exámen
    }
}
```

## - ✅

Un banco decide entregar promociones a sus clientes por medio de su agente de prensa, el cual lo hace de la siguiente manera: el agente debe entregar 50 premios entre los 1000 clientes, para esto, obtiene al azar un número de cliente y le entrega el premio, una vez que este lo toma continúa con la entrega.
**Notas: Cuando los 50 premios han sido entregados el agente y los clientes terminan su ejecución; No se puede utilizar una estructura de tipo arreglo para almacenar los premios de los clientes.**

```cs
sem esperarPremios[1000] = ([1000] 0)
Premio premioActual
sem tomoPremio = 0

process Cliente[id: 0..999] {
    P(esperarPremios[id])
    if premioActual != null {
        Premio p = premioActual
        V(tomoPremio)
    }
}

process Banco {
    Premio premios[50]
    for i=0 to 49 {
        int numeroClienteAzar = GetNumero() // Obtiene al azar un número cliente
        premioActual = premios[i]
        V(esperarPremios[numeroClienteAzar])
        P(tomoPremio)
    }
    premioActual = null
    for i=0 to 999
        V(esperarPremios[i])
}
```

## - ✅

En una empresa hay UN Coordinador y 30 Empleados que formarán 3 grupos de 10 empleados cada uno. Cada grupo trabaja en una sección diferente y debe realizar 345 unidades de un producto. Cada empleado al llegar se dirige al coordinador para que le indique el número de grupo al que pertenece y una vez que conoce este dato comienza a trabajar hasta que se han terminado de hacer las 345 unidades correspondientes al grupo (cada unidad es hecha por un único empleado). Al terminar de hacer las 345 unidades los 10 empleados del grupo se deben juntar para retirarse todos juntos. El coordinador debe atender a los empleados de acuerdo al orden de llegada para darle el número de grupo (a los 10 primeros que lleguen se le asigna el grupo 1, a los 10 del medio el 2, y a los 10 últimos el 3). Cuando todos los grupos terminaron de trabajar el coordinador debe informar (imprimir en pantalla) el empleado que más unidades ha realizado (si hubiese más de uno con la misma cantidad máxima debe informarlos a todos ellos).
**Nota: maximizar la concurrencia; suponga que existe una función Generar() que simula la elaboración de una unidad de un producto.**

```cs
Cola empleados(30)
sem llegue = 0
sem mutex = 1
sem miNumeroGrupo[30] = ([30] 0)
int miGrupo[30] = ([30] 0)

sem mutexGrupo[3] = ([3] 1)
int cantUnidadesGrupo[3] = ([3] 0)
int unidadesQueHice[30] = ([30] 0)

sem mutexBarreras[3] = ([3] 1)
int cantTerminaron[3] = ([3] 0)
sem barreras[3] = ([3] 0)

sem avisarCoordinador = 0

process Empleados[id: 0..29] {
    P(mutex)
    empleados.push(id)
    V(mutex)
    V(llegue)
    P(miNumeroGrupo[id])
    int grupo = miGrupo[id]
    P(mutexGrupo[grupo])
    while cantUnidadesGrupo[grupo] < 345 {
        cantUnidadesGrupo[grupo]++
        V(mutexGrupo[grupo])
        unidadesQueHice[id]++
        Generar()
        P(mutexGrupo[grupo])
    }
    V(mutexGrupo[grupo])

    P(mutexBarreras[grupo])
    cantTerminaron[grupo]++
    if cantTerminaron[grupo] == 10
        for i=0 to 9
            V(barreras[grupo])
    V(mutexBarreras[grupo])
    P(barreras[grupo])
    V(avisarCoordinador)
}

process Coordinador {
    int idActual
    int cantLlegaron = 0
    int grupoActual = 1
    for i=0 to 29 {
        P(llegue)
        P(mutex)
        idActual = empleados.pop()
        V(mutex)
        cantLlegaron++
        if cantLlegaron == 10
            cantLlegaron = 0
            grupoActual++
        miGrupo[idActual] = grupoActual
        V(miNumeroGrupo[idActual])
    }
    for i=0 to 29
        P(avisarCoordinador)
    print(unidadesQueHice.max()) // Asumo que la función devuelve más de un valor si hay más de un máximo
}
```

## - ✅

En una fábrica de muebles trabajan 50 empleados. A llegar, los empleados forman 10 grupos de 5 personas cada uno, de acuerdo al orden de llegada (los 5 primeros en llegar forman el primer grupo, los 5 siguientes el segundo grupo, y así sucesivamente). Cuando un grupo se ha terminado de formar, todos sus integrantes se ponen a trabajar. Cada grupo debe armar M muebles (cada mueble es armado por un solo empleado); mientras haya muebles por armar en el grupo los empleados los irán resolviendo (cada mueble es armado por un solo empleado).
**Nota: Cada empleado puede tardar distinto tiempo en armar un mueble. Sólo se pueden usar los procesos “Empleado”, y todos deben terminar su ejecución. Maximizar la concurrencia.**

```cs
sem mutex = 1
int empleados = 0
int grupos = 0
sem esperarGrupo[10] = ([10] 0)

sem mutexGrupo[10] = ([10] 1)
int mueblesPorGrupo[10] = ([10] 0)

process Empleado[id: 0..49] {
    P(mutex)
    empleados++
    grupo = grupos
    if empleados == 5 {
        for i=1 to 4
            V(esperarGrupo[grupo])
        empleados = 0
        grupo++
        V(mutex)
    }
    else {
        V(mutex)
        P(esperarGrupo[grupo])
    }
    P(mutexGrupo[grupo])
    while (mueblesPorGrupo[grupo] < M) {
        mueblesPorGrupo[grupo]++
        V(mutexGrupo[grupo])
        ArmarMueble()
        P(mutexGrupo[grupo])
    }
    V(mutexGrupo[grupo])
}
```
