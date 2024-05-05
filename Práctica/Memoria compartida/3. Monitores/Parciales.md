# Monitores

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## Ejercicio 2) - 18 de diciembre, 2023 ✅

En una montaña hay 30 escaladores que en una parte de la subida deben utilizar un único paso de a uno a la vez y de acuerdo al orden de llegada al mismo.
**Nota: sólo se pueden utilizar procesos que representen a los escaladores; cada escalador usa sólo una vez el paso**

```cs
process Escalador [id: 0..29] {
    Paso.pasar()
    // Usa el paso
    Paso.salir()
}
monitor Paso {
    cond espera
    bool libre = true
    int cantEsperando = 0

    procedure pasar() {
        if libre
            libre = false
        else {
            cantEsperando++
            wait(espera)
        }
    }
    procedure salir() {
        if cantEsperando == 0
            libre = true
        else {
            cantEsperando--
            signal(espera)
        }
    }
}
```

## Ejercicio 2) - 4 de diciembre, 2023 ❓

En una empresa trabajan 20 vendedores ambulantes que forman 5 equipos de 4 personas cada uno (cada vendedor conoce previamente a que equipo pertenece). Cada equipo se encarga de vender un producto diferente. Las personas de un equipo se deben juntar antes de comenzar a trabajar. Luego cada integrante del equipo trabaja independientemente del resto vendiendo ejemplares del producto correspondiente. Al terminar cada integrante del grupo debe conocer la cantidad de ejemplares vendidos por el grupo.
**Nota: maximizar la concurrencia.**

```cs
process Vendedor [id: 1..20] {
    int miEquipo = Equipo()
    int total, cantVendida = 0
    Coordinadores[miEquipo].inicio()
    cantVendida = Vender()
    Coordinadores[miEquipo].fin(cantVendida, total)
}
monitor Coordinadores[id: 1..5] {
    cond vendedores
    int cantLlegar, cantSalir, totalVendido = 0
    procedure inicio() {
        cantLlegar++
        if cantLlegar == 4
            signal_all(vendedores)
        else
            wait(vendedores)
    }
    procedure fin(int cantVendio: IN, int totalVendido: OUT) {
        cantSalir++
        totalVendido += cantVendio  // No deberíamos tener 5 variables total, una por cada grupo?
        if cantSalir == 4
            signal_all(vendedores)
        else
            wait(vendedores)
    }
}
```

## Ejercicio 2) - 10 de octubre, 2023 ✅

En una elección estudiantil, se utiliza una máquina para voto electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina de acuerdo con el orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. La máquina de voto sólo puede ser usada por una persona a la vez.
**Nota: la función Votar() permite usar la máquina**

```cs
Monitor Mesa {
    cond miTurno[N]
    cond autoridad, salio
    colaOrdenada(int, int) personas

    procedure llegada(int id: IN, int prioridad: IN) {
        personas.push(prioridad, id) // Inserta ordenado por prioridad de forma ascendiente
        signal(autoridad)            // avisarle a la autoridad que la persona quiere pasar a votar
        wait(miTurno[id])
    }

    procedure siguiente() {
        if empty(personas)
            wait(autoridad)
        int id = personas.pop().id
        signal(miTurno[id])
        wait(salio)
    }

    procedure salida() {
        signal(salio)
    }
}

Process Persona [id: 0..N-1] {
    int prioridad = obtenerPrioridad()  // 1 = Normal | 0 = Anciano/Embarazada
    Mesa.llegada(id, prioridad) // solicitar acceso
    Votar()
    Mesa.salida() // liberar
}

Process AutoridadDeMesa {
    while true
        Mesa.siguiente() // dar acceso para votar a la siguiente persona
}
```

## Ejercicio 2) - Segundo recuperatorio 2022 ❓

En un sistema operativo se ejecutan 20 procesos que periódicamente realizan cierto cómputo mediante la función Procesar(). Los resultados de dicha función son persistidos en un archivo, para lo que se requiere de acceso al subsistema de E/S. Sólo un proceso a la vez puede hacer uso del subsistema de E/S, y el acceso al mismo se define por la prioridad del proceso (menor valor indica mayor prioridad).

```cs
Monitor SubsistemaES {
    int esperando = 0
    int usando = 0
    cond colas [N]
    Cola en_espera(int, int)

    procedure pedir (int id, int prioridad) {
        if (usando > 0) { // está ocupado

        esperando++ // incrementar contador para indicar que hay uno más en espera
        en_espera.push(prioridad, id) // inserta ordenado por prioridad
        wait (colas [id]) // dormir en cola condition individual
        }
        else // está libre
            usando++ // marcar como ocupado
    }
    procedure liberar() {
        if (esperando > 0) { // si hay procesos esperando
            int id = en_espera.pop() // seleccionar el de mayor prioridad
            signal (colas [id])  // despertar al de mayor prioridad
            esperando-- // decrementar para indicar que hay uno menos en espera
        }
        else
            usando-- //marcar como libre
    }
}

Process Proceso [i: 1..20] {
    int prioridad = obtenerPrioridad()
    While (true) {
        resultados = Procesar () // computar
        SubsistemaES.pedir (i,edad) // solicitar acceso
        Persistir (resultados) // usar subsistema E/S
        SubsistemaES.liberar() // liberar
    }
}
```

## Ejercicio 2) - Primer recuperatorio 2022 ❓

En una planta verificadora de vehículos existen 5 estaciones de verificación. Hay 75 vehículos que van para ser verificados, cada uno conoce el número de estación a la cual debe ir. Cada vehículo se dirige a la estación correspondiente y espera a que lo atiendan. Una vez que le entregan el comprobante de verificación, el vehículo se retira. Considere que en cada estación se atienden a los vehículos de acuerdo con el orden de llegada.
**Nota: maximizar la concurrencia.**

```cs
Monitor Admin [ id: 0..4] {
    cola vehiculos
    string comprobante
    cond esperaVehiculos, esperaEstacion, fin

    procedure esperarAtencion(id: IN int, comp: OUT string) {
        vehiculos.push(id)
        signal(esperaEstacion)
        wait(esperaVehiculos)   // Espera a que esté su comprobante
        comp = comprobante
        signal(fin) // Avisa que tomó el comprobante y que se va
    }
    procedure siguiente(idVehiculo: OUT int) {
        if empty(vehiculos)
            wait(esperaEstacion)
        idVehiculo = vehiculos.pop()
    }
    procedure entregarComprobante(comp: IN string) {
        comprobante = comp
        signal(esperaVehiculos) // Despierta al vehículo para que tome el comprobante
        wait(fin)   // Espera a que haya tomado el comprobante para atender al siguiente vehículo
    }
}

Monitor Estacion [ id: 0..4] {
    string comprobante
    int idVehiculo
    while true {
        Admin[id].siguiente(idVehiculo)
        comprobante = generarComprobante(idVehiculo)
        Admin[id].entregarComprobante(comprobante)
    }
}

process Vehiculo [ id: 0..74] {
	string comprobante
    int numeroEstacion = GetNumero()
    Admin[numeroEstacion].esperarAtencion(id, comprobante)
}
```

## - ❓

Una boletería vende E entradas para un partido, y hay P personas (P>E) que quieren comprar. Se las atiende por orden de llegada y la función vender() simula la venta. La boletería debe informarle a la persona que no hay más entradas disponibles o devolverle el número de entrada si pudo hacer la compra.

```cs
Monitor Boleteria {
	cantidadEntradas = E
	Procedure Comprar(out numero) {
		if (cantidadEntradas == 0) {
			numero = -1
			print("No hay mas entradas")
		}
		else {
			cantidadEntradas--
			numero = vender()
		}
	}
}
Monitor Fila {
    personasEsperando = 0
    libre = true
    cond cola
    Procedure HacerFila() {
        if libre
			libre = false
		else {
		    personasEsperando++
			wait(cola)
		}
    }
    Procedure Salir() {
        if (personasEsperando == 0)
            libre = true
        else {
            personasEsperando--
			signal(cola)
        }
    }
}
Process persona[id:1..P]{
    Fila.HacerFila()    // Tener 2 monitores permite que se puedan encolar mientras una persona esta siendo atendida
    numero = Boleteria.Comprar()
	Fila.Salir()
}
```

## - ✅

Por un puente turístico puede pasar sólo un auto a la vez. Hay N autos que quieren pasar (función pasar()) y lo hacen por orden de llegada.

```cs
Monitor Puente {
	bool libre = true
	int cantidadAutosEsperando = 0
	cond colaAutos

	procedure Entrar() {
		if libre
			libre = false
		else {
			cantidadAutosEsperando++
			wait(colaAutos)
		}
	}
	procedure Salir() {
		if cantidadAutosEsperando == 0
			libre = true
		else {
			cantidadAutosEsperando--
			signal(colaAutos)
		}
	}
}

process Auto [ id: 0..N-1] {
	Puente.Entrar()
	Pasar()
	Puente.Salir()
}
```

## - ❓

En la guardia de traumatología de un hospital trabajan 5 médicos y una enfermera. A la guardia acuden P Pacientes que al llegar se dirigen a la enfermera para que le indique a que médico se debe dirigir y cuál es su gravedad (entero entre 1 y 10). Cuando tiene estos datos se dirige al médico correspondiente y espera hasta que lo termine de atender para retirarse. Cada médico atiende a sus pacientes en orden de acuerdo a la gravedad de cada uno.
**Nota: maximizar la concurrencia.**

```cs
??
```

## - ❓

Un equipo de videoconferencia puede ser usado por una persona a la vez. Hay P personas que utilizan este equipo (una unica vez cada uno) para su trabajo, de acuerdo a su prioridad. La prioridad de cada persona está dada por un numero entero positivo. Ademas existe un administrador que cada 3hs incrementa en 1 la prioridad de todas las personas que están esperando usar el equipo.
**Nota: maximizar la concurrencia.**

```cs
??
```

## - ✅

En un comedor estudiantil hay un horno microondas que debe ser usado por E estudiantes de acuerdo con el orden de llegada. Cuando el estudiante accede al horno, lo usa y luego se retira para dejar al siguiente.
**Nota: cada Estudiante una sólo una vez el horno; los únicos procesos que se pueden usar son los “estudiantes”.**

```cs
Monitor Horno {
    int usando = 0
    int esperando = 0
    cond cola

    procedure pedir() {
        if usando > 0 {
            esperando++
            wait(cola)
        }
        else
            usando++
    }
    procedure liberar() {
        if esperando > 0 {
            signal(cola)
            esperando--
        }
        else
            usando--
    }
}

Process Estudiante [id: 1..E] {
    Horno.pedir()
    sleep(rand()) // Calentar
    Horno.liberar()
}
```
