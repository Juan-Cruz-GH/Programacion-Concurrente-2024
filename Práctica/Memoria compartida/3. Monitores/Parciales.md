# Monitores

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## Ejercicio 2) - 18 de diciembre, 2023 ✅

En una montaña hay 30 escaladores que en una parte de la subida deben utilizar un único paso de a uno a la vez y de acuerdo al orden de llegada al mismo.
**Nota: sólo se pueden utilizar procesos que representen a los escaladores; cada escalador usa sólo una vez el paso**

```cs
Monitor Paso {
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

process Escalador[id: 0..29] {
    Paso.pasar()
    // Usa el paso
    Paso.salir()
}
```

## Ejercicio 2) - 4 de diciembre, 2023 ✅

En una empresa trabajan 20 vendedores ambulantes que forman 5 equipos de 4 personas cada uno (cada vendedor conoce previamente a que equipo pertenece). Cada equipo se encarga de vender un producto diferente. Las personas de un equipo se deben juntar antes de comenzar a trabajar. Luego cada integrante del equipo trabaja independientemente del resto vendiendo ejemplares del producto correspondiente. Al terminar cada integrante del grupo debe conocer la cantidad de ejemplares vendidos por el grupo.
**Nota: maximizar la concurrencia.**

```cs
Monitor Coordinadores[id: 1..5] {
    cond vendedores
    int cantLlegar, cantSalir, totalVendido = 0
    procedure inicio() {
        cantLlegar++
        if cantLlegar == 4
            signal_all(vendedores)
        else
            wait(vendedores)
    }
    procedure fin(IN int cantVendio, OUT int total) {
        cantSalir++
        totalVendido += cantVendio
        if cantSalir == 4
            signal_all(vendedores)
        else
            wait(vendedores)
        total = totalVendido
    }
}

process Vendedor[id: 1..20] {
    int total, cantVendida = 0
    int miEquipo = Equipo()
    Coordinadores[miEquipo].inicio()
    cantVendida = Vender()
    Coordinadores[miEquipo].fin(cantVendida, total)
}
```

## Ejercicio 2) - 10 de octubre, 2023 ✅

En una elección estudiantil, se utiliza una máquina para voto electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina de acuerdo con el orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. La máquina de voto sólo puede ser usada por una persona a la vez.
**Nota: la función Votar() permite usar la máquina**

```cs
Monitor Mesa {
    cond miTurno[N]
    cond autoridad, salio
    ColaOrdenada personas(int, int)

    procedure llegada(IN int id, IN int prioridad) {
        personas.push(id, prioridad) // Inserta ordenado por prioridad de forma ascendiente
        signal(autoridad)   // Avisarle a la autoridad que la persona quiere pasar a votar
        wait(miTurno[id])
    }
    procedure siguiente() {
        if personas.empty()
            wait(autoridad)
        int id = personas.pop().id
        signal(miTurno[id])
        wait(salio)
    }
    procedure salida() {
        signal(salio)
    }
}

process Persona[id: 0..N-1] {
    int prioridad = obtenerPrioridad()  // 1 = Normal | 0 = Anciano/Embarazada
    Mesa.llegada(id, prioridad) // solicitar acceso
    Votar()
    Mesa.salida() // liberar
}

process AutoridadDeMesa {
    while true
        Mesa.siguiente() // dar acceso para votar a la siguiente persona
}
```

## Ejercicio 2) - Segundo recuperatorio 2022 ✅

En un sistema operativo se ejecutan 20 procesos que periódicamente realizan cierto cómputo mediante la función Procesar(). Los resultados de dicha función son persistidos en un archivo, para lo que se requiere de acceso al subsistema de E/S. Sólo un proceso a la vez puede hacer uso del subsistema de E/S, y el acceso al mismo se define por la prioridad del proceso (menor valor indica mayor prioridad).

```cs
Monitor SubsistemaES {
    int esperando = 0
    bool libre = true
    cond colas[N]
    ColaOrdenada procesos(int, int)

    procedure pedir(IN int id, IN int prioridad) {
        if libre
            libre = false
        else {
            esperando++
            procesos.push(id, prioridad) // Inserta ordenado por prioridad
            wait(colas[id])
        }
    }
    procedure liberar() {
        if esperando == 0
            libre = true
        else {
            esperando--
            int id = procesos.pop().id
            signal(colas[id])
        }
    }
}

process Proceso[id: 1..20] {
    int prioridad = obtenerPrioridad()
    while true {
        resultados = Procesar()
        SubsistemaES.pedir(id, prioridad)
        Persistir(resultados)
        SubsistemaES.liberar()
    }
}
```

## Ejercicio 2) - Primer recuperatorio 2022 ✅

En una planta verificadora de vehículos existen 5 estaciones de verificación. Hay 75 vehículos que van para ser verificados, cada uno conoce el número de estación a la cual debe ir. Cada vehículo se dirige a la estación correspondiente y espera a que lo atiendan. Una vez que le entregan el comprobante de verificación, el vehículo se retira. Considere que en cada estación se atienden a los vehículos de acuerdo con el orden de llegada.
**Nota: maximizar la concurrencia.**

```cs
Monitor Admin[id: 0..4] {
    cola vehiculos
    string comprobante
    cond esperaVehiculos, esperaEstacion, fin

    procedure esperarAtencion(IN int id, OUT string comp) {
        vehiculos.push(id)
        signal(esperaEstacion)
        wait(esperaVehiculos)   // Espera a que esté su comprobante
        comp = comprobante
        signal(fin) // Avisa que tomó el comprobante y que se va
    }
    procedure siguiente(OUT int idVe) {
        if empty(vehiculos)
            wait(esperaEstacion)
        idVehiculo = vehiculos.pop()
    }
    procedure entregarComprobante(IN string comp) {
        comprobante = comp
        signal(esperaVehiculos) // Despierta al vehículo para que tome el comprobante
        wait(fin)   // Espera a que haya tomado el comprobante para atender al siguiente vehículo
    }
}

process Estacion[id: 0..4] {
    string comprobante
    int idVehiculo
    while true {
        Admin[id].siguiente(idVehiculo)
        comprobante = generarComprobante(idVehiculo)
        Admin[id].entregarComprobante(comprobante)
    }
}

process Vehiculo[id: 0..74] {
	string comprobante
    int numeroEstacion = GetNumero()
    Admin[numeroEstacion].esperarAtencion(id, comprobante)
}
```

## - ✅

Una boletería vende E entradas para un partido, y hay P personas (P>E) que quieren comprar. Se las atiende por orden de llegada y la función vender() simula la venta. La boletería debe informarle a la persona que no hay más entradas disponibles o devolverle el número de entrada si pudo hacer la compra.

```cs
Monitor Boleteria {
	int cantidadEntradas = E
	procedure comprar(OUT int numero) {
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
    int personasEsperando = 0
    bool libre = true
    cond cola
    procedure hacerFila() {
        if libre
			libre = false
		else {
		    personasEsperando++
			wait(cola)
		}
    }
    procedure salir() {
        if (personasEsperando == 0)
            libre = true
        else {
            personasEsperando--
			signal(cola)
        }
    }
}

process Persona[id: 0..P-1]{
    Fila.hacerFila()    // Tener 2 Monitores permite que se puedan encolar mientras una persona esta siendo atendida
    Boleteria.comprar(numero)
	Fila.salir()
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

process Auto[id: 0..N-1] {
	Puente.Entrar()
	Pasar()
	Puente.Salir()
}
```

## - ✅

En la guardia de traumatología de un hospital trabajan 5 médicos y una enfermera. A la guardia acuden P Pacientes que al llegar se dirigen a la enfermera para que le indique a que médico se debe dirigir y cuál es su gravedad (entero entre 1 y 10). Cuando tiene estos datos se dirige al médico correspondiente y espera hasta que lo termine de atender para retirarse. Cada médico atiende a sus pacientes en orden de acuerdo a la gravedad de cada uno.
**Nota: maximizar la concurrencia.**

```cs
Monitor Medicos[id: 0..4] {
    cond espera[N]
    ColaOrdenada pacientes(int, int) // idPaciente y gravedd
    bool libre = true
    int esperando = 0

    procedure pasar(IN int idPaciente, IN int gravedad) {
        if libre
            libre = false
        else {
            esperando++
            pacientes.push(idPaciente, gravedad) // inserta ordenado descendientemente por gravedad
            wait(espera[idPaciente])
        }
    }
    procedure salir() {
        if esperando == 0
            libre = true
        else {
            esperando--
            int siguiente = pacientes.pop().idPaciente
            signal(espera[siguiente])
        }
    }
}

Monitor Enfermera {
    int medicos[5] = (0 1 2 3 4)
    int i = 0

    procedure miMedico(OUT int medico, OUT int gravedad) {
        medico = medicos[i]
        i++
        if i == 5
            i = 0
        gravedad = EvaluarGravedad()
    }
}

process Paciente[id: 0..P-1] {
    int medico, gravedad
    Enfermera.miMedico(medico, gravedad)
    Medicos[medico].pasar(id, gravedad)
    // el médico lo atiende
    Medicos[medico].salir()
}
```

## - ✅

Un equipo de videoconferencia puede ser usado por una persona a la vez. Hay P personas que utilizan este equipo (una unica vez cada uno) para su trabajo, de acuerdo a su prioridad. La prioridad de cada persona está dada por un numero entero positivo. Ademas existe un administrador que cada 3hs incrementa en 1 la prioridad de todas las personas que están esperando usar el equipo.
**Nota: maximizar la concurrencia.**

```cs
Monitor EquipoVideo {
    bool libre = true
    int esperando = 0
    cond espera[N]
    ColaOrdenada personas(int, int) // id y prioridad

    procedure pedir(IN int id, IN int prioridad) {
        if libre
            libre = false
        else {
            esperando++
            personas.push(id, prioridad) // inserta ordenado ascendientemente por prioridad
            wait(espera[id])
        }
    }
    procedure salir() {
        if esperando == 0
            libre = true
        else {
            esperando--
            int siguiente = personas.pop().id
            signal(espera[siguiente])
        }
    }
    procedure incrementar() {
        if personas.NotEmpty()
            for persona in personas
                persona.prioridad++
    }
}

process Persona[id: 0..P-1] {
    int prioridad = GetPrioridad()
    EquipoVideo.pedir(id, prioridad)
    // Usa el equipo de video
    EquipoVideo.salir()
}

process Administrador {
    while true {
        delay(3 * 60 * 60)
        EquipoVideo.incrementar()
    }
}
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

process Estudiante[id: 1..E] {
    Horno.pedir()
    // Usa el horno
    Horno.liberar()
}
```
