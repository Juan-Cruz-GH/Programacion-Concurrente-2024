<h1 align="center">Monitores</h1>

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
cond cola
```

</td><td>
Variable condición. Actúa como una cola de procesos dormidos. 
<tr><td>

```cs
cond cola[N]
```

</td><td>
Arreglo de variables condición. Permiten despertar o dormir a procesos particulares por medio de su ID. 
<tr><td>

```cs
wait(cola)
```

</td><td>
Duerme al final de la cola de la variable condición al proceso que está ejecutando el monitor en este momento.
<tr><td>

```cs
signal(cola)
```

</td><td>
Despierta al primer proceso dormido en la cola de la variable condición. Una vez despertado pasa a competir por acceder al monitor junto a todos los otros procesos que compitan. Cuando logra acceder continúa ejecutando en la instrucción siguiente a donde se durmió.
<tr><td>

```cs
signal_all(cola)
```

</td><td>
Despierta a todos los procesos dormidos en la cola de la variable condición y todos ellos pasan a competir por acceder al monitor.
</table>

## Notas importantes

#### Acceso al monitor

-   Una vez que el monitor se libera, **cualquiera** de los demás procesos que estaban esperando a acceder pasará a tomar control del monitor, incluyendo los que estaban esperando desde el programa principal y los que se despertaron. No hay ningún tipo de orden. No se puede saber cuál proceso pasará al monitor.

```cs
Monitor A {
    procedure abc() {
        delay(90)
    }
}
process Proceso [id: 0..N-1] {
    A.abc() // Cualquiera de ellos pasa
}
```

#### Eficiencia

-   Como un monitor solo puede estar ejecutando un procedure a la vez, cuando necesitamos que 2 o más tareas se realicen al mismo tiempo necesitamos más de un monitor.

<table>
  <thead>
    <tr>
      <th>❌ Código ineficiente</th>
      <th>✅ Código eficiente</th>
    </tr>
  </thead>
<tr><td>

```cs

```

</td><td>

```cs

```

</td></tr>
</table>

-   La exclusión mutua es implícita dentro de un monitor **ya que no puede ejecutar más de un llamado a un procedimiento a la vez**. Hasta que no termine el procedure o se duerma al proceso en una variable condición no se libera el monitor para atender otro llamado.
-   En un procedure de un monitor A se puede llamar a un procedure de otro monitor B. El monitor A no podrá continuar su ejecución ni atender otros pedidos hasta que el procedure de monitor B termine.
-   Cuando el monitor se libera, **todos los procesos** que están haciendo llamados a los procedures compiten por acceder. No se respeta el orden de llegada.
-   Hacer signal en una **variable condición vacía no causa ningun error**.
-   En caso de no necesitar ningun orden, el monitor puede representar 100% al recurso compartido en vez de ser un "manejador" o "administrador" del recurso compartido.
-   En los ejercicios de tipo buffer 1 Consumidor y N Productores necesitamos 1) N procesos 2) un monitor Administrador 3) 1 proceso Servidor.
-   En los ejercicios de tipo buffer M Consumidores y N Productores necesitamos 1) N procesos 2) un monitor Administrador 3) M procesos Servidor. Lo único que cambia es while en vez de if empty(cola) en el procedure Siguiente y variables condición privadas para dormirse luego de hacer el push + el signal.

## Tipos de ejercicios

#### Orden de llegada

<table>
  <thead>
    <tr>
      <th>Con 1 recurso</th>
      <th>Con M recursos</th>
      <th>Con límite basado en atributo</th>
    </tr>
  </thead>
<tr><td>

-   Los procesos acceden al recurso compartido según el orden de llegada.
-   Tenemos una variable condición en la cual los procesos se duermen mientras haya otro proceso utilizando la SC en ese momento.
-   Cuando el proceso actual termina de usar el recurso despierta al siguiente en la variable condición o pone libre en true si no hay nadie más.

```cs
Monitor A {
    cond cola
    bool libre = true
    int cantEsperando = 0

    procedure pasar() {
        if libre
            libre = false
        else {
            cantEsperando++
            wait(cola)
        }
    }
    procedure salir() {
        if cantEsperando == 0
            libre = true
        else {
            cantEsperando--
            signal(cola)
        }
    }
}
process Proceso [id: 0..N-1] {
    A.pasar()
    // Realiza acción con exclusión mutua
    A.salir()
}
```

</td><td>

??

```cs
Monitor A {
    cond cola
    int cantEsperando = 0

    procedure pasar() {
        while cantEsperando == M
            wait(cola)
        cantEsperando++
    }
    procedure salir() {
        cantEsperando--
        signal(cola)
    }
}
process Proceso [id: 0..N-1] {
    A.pasar()
    // Realiza acción con exclusión mutua
    A.salir()
}
```

</td><td>

-   Los procesos acceden al recurso compartido según el orden de llegada pero existe un límite basado en uno de sus atributos. Si se llega al límite, no pueden pasar más procesos hasta que se reduzca ese valor.
-   Tenemos variables condición privadas en las cuales los procesos se duermen si hay otro proceso utilizando la SC en ese momento o si no hay más espacio.
-   Cuando el proceso actual termina de usar el recurso despierta al siguiente en las variables condición privadas si queda espacio.

```cs
Monitor A {
    cond cola[N]
    int pesoTotal
    Cola procesos(int, int)

    procedure pasar(IN int peso, IN int id) {
        if (procesos.notEmpty()) or (pesoTotal + peso > límite) {
            procesos.push(peso, id)
            wait(cola[id])
            procesos.pop()
        }
        pesoTotal += peso
    }
    procedure salir(IN int peso) {
        pesoTotal -= peso
        if procesos.notEmpty() {
            int peso, id = procesos.pop().peso
            if pesoTotal + peso <= límite
                signal(cola[id])
        }
    }
}
process Proceso [id: 0..N-1] {
    A.pasar(peso, id)
    // Realiza acción con exclusión mutua
    A.salir(peso)
}
```

</td></tr>

</table>

#### Orden de llegada con Coordinador

<table>
  <thead>
    <tr>
      <th>Con 1 recurso</th>
      <th>Con M recursos</th>
    </tr>
  </thead>
<tr><td>

-   Los procesos acceden al recurso compartido según el orden de llegada mediante un Coordinador.
-   Tenemos 3 variables condición, una para simbolizar que llegó un nuevo proceso, otra para esperar que el Coordinador le avise que es su turno, y otra para que el proceso le avise al Coordinador que terminó.
-   Cuando el proceso actual termina de usar el recurso le avisa al Coordinador el cual despierta al siguiente en la cola.

```cs
Monitor A {
    cond pedidos
    cond termino
    cond turno
    int pedidos = 0

    procedure pasar() {
        pedidos++
        signal(pedidos)
        wait(turno)
    }
    procedure siguiente() {
        if pedidos == 0
            wait(pedidos)
        pedidos--
        signal(turno)
        wait(termino)
    }
    procedure salir() {
        signal(termino)
    }
}
process Proceso [id: 0..N-1] {
    A.pasar()
    // Realiza acción con exclusión mutua
    A.salir()
}
process Coordinador {
    for i=0 to N-1
        A.siguiente()
}
```

</td><td>

-   Los procesos acceden a M recursos compartidos según el orden de llegada mediante un Coordinador.
-   Tenemos 3 variables condición, una para simbolizar que llegó un nuevo proceso, otra para esperar que el Coordinador le avise que es su turno, y otra para que el proceso le avise al Coordinador que liberó un recurso.

```cs
Monitor A {
    cond pedidos
    cond liberar
    cond turno
    Cola procesos
    Cola recursos(M)
    int miRecurso[N]

    procedure pasar(IN int id, OUT int idRecurso) {
        procesos.push(id)
        signal(pedidos)
        wait(turno)
        idRecurso = miRecurso[id]
    }
    procedure siguiente() {
        if procesos.empty()
            wait(pedidos)
        if recursos.empty()
            wait(liberar)
        int procesoSig = procesos.pop()
        int recursoSig = recursos.pop()
        miRecurso[procesoSig] = recursoSig
        signal(turno)
    }
    procedure salir(IN int idRecurso) {
        recursos.push(idRecurso)
        signal(liberar)
    }
}
process Proceso [id: 0..N-1] {
    A.pasar(id, idRecurso)
    // Realiza acción de ese recurso con exclusión mutua
    A.salir(idRecurso)
}
process Coordinador {
    for i=0 to N-1
        A.siguiente()
}
```

</td></tr>
</table>

#### Orden de algun atributo (prioridad)

-   Los procesos acceden al recurso compartido según el orden de uno de sus atributos.
-   Tenemos una ColaOrdenada a la cual los procesos se encolarán mientras haya otro proceso utilizando la SC en ese momento además de dormirse en su semáforo privado.
-   Cuando el proceso actual termina de usar el recurso despierta al siguiente en la cola o pone libre en true si no hay nadie más.

```cs
Monitor A {
    cond colas[N]
    ColaOrdenada procesos(int, int)
    bool libre = true
    int cantEsperando = 0

    procedure pasar(IN int id, IN int edad) {
        if libre
            libre = false
        else {
            cantEsperando++
            procesos.push(id, edad) // Inserta ordenado por edad
            wait(colas[id])
        }
    }
    procedure salir() {
        if cantEsperando == 0
            libre = true
        else {
            cantEsperando--
            int id = procesos.pop().id
            signal(colas[id])
        }
    }
}
process Proceso [id: 0..N-1] {
    int edad = GetEdad()
    A.pasar(id, edad)
    // Realiza acción con exclusión mutua
    A.salir()
}
```

#### Orden de identificador (ID)

-   Los procesos acceden al recurso compartido según el orden de sus IDs.
-   Tenemos variables condición privadas en las cuales los procesos se duermen si no es su turno.
-   El proceso actual, cuando termina, despierta al siguiente.

```cs
Monitor A {
    cond colas[N]
    int sig = 0

    procedure pasar(IN int id) {
        if id != sig
            wait(colas[id])
    }
    procedure salir() {
        sig++
        if sig < N
            signal(colas[sig])
    }
}
process Proceso [id: 0..N-1] {
    A.pasar(id)
    // Realiza acción con exclusión mutua
    A.salir()
}
```
