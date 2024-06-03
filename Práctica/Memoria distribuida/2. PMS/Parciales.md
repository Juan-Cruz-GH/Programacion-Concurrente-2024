<h1 align="center">Pasaje de Mensajes Sincrónicos</h1>

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## Ejercicio 1) - 4 de diciembre, 2023 ✅

En la estación de trenes hay una terminal de SUBE que debe ser usada por P personas de acuerdo con el orden de llegada. Cuando la persona accede a la terminal, la usa y luego se retira para dejar al siguiente.
**Nota: cada Persona una sólo una vez la terminal.**

```cs
process Persona[id: 0..P-1] {
    Admin!pedido(id)
    Admin?usar()
    // usar la terminal
    Admin!liberar()
}

process Admin {
    bool libre = true
    Cola personas(int)
    int idP
    while true {
        if
        [] libre; Persona[*]?pedido(idP) -> {
            libre = false;
            Persona[idP]!usar();
        }
        [] not libre; Persona[*]?pedido(idP) -> {
            personas.push(idP);
        }
        [] ;Persona[*]?liberar(idP) -> {
            if (personas.isEmpty())
                libre = true;
            else
                Persona[personas.pop()]!usar();
        }
        fi
    }
}
```

## Ejercicio 2) - 13 de noviembre, 2023 ✅

En una oficina existen 100 empleados que envían documentos para imprimir en 5 impresoras compartidas. Los pedidos de impresión son procesados por orden de llegada y se asignan a la primera impresora que se encuentre libre.

```cs
process Empleado[id: 0..99] {
    string doc, docImpreso
    while true {
        doc = Generar()
        Coordinador!solicitudImpresion(id, doc)
        Impresora[*]?miDocumentoImpreso[id](docImpreso)
    }
}

process Coordinador {
    Cola c(int, string)
    string doc
    int idI, idE
    while true {
        if
        [] ;Empleado[*]?solicitudImpresion(idE, doc) -> {
            c.push(idE, doc)
        }
        [] cola.NotEmpty(); Impresora[*]?solicitudTrabajo(idI) -> {
            idE, doc = c.pop()
            Impresora[idI]!asignacionTrabajo(idE, doc)
        }
        fi
    }
}

process Impresora[id: 0..4] {
    string doc, docImpreso
    int idE
    while true {
        Coordinador!solicitudTrabajo(id)
        Coordinador?asignacionTrabajo(idE, doc)
        docImpreso = Imprimir(doc)
        Empleado[idE]!miDocumentoImpreso(docImpreso)
    }
}
```

## - ✅

En un torneo de programación hay 1 organizador, N competidores y S supervisores. El organizador comunica el desafío a resolver a cada competidor. Cuando un competidor cuenta con el desafío a resolver, lo hace y lo entrega para ser evaluado. A continuación, espera a que alguno de los supervisores lo corrija y le indique si está bien. En caso de tener errores, el competidor debe corregirlo y volver a entregar, repitiendo la misma metodología hasta que llegue a la solución esperada. Los supervisores corrigen las entregas respetando el orden en que los competidores van entregando.
**Nota: maximizar la concurrencia y no generar demora innecesaria.**

```cs
process Organizador {
    int id
    string desafio
    for i = 0 to N-1
        Competidor[i]!enviarDesafio(desafio)
}

process Competidor[id: 0..N-1] {
    bool estaBien = false
    string desafio, desafioResuelto

    Organizador?enviarDesafio(desafio)
    while not estaBien {
        desafioResuelto = Resolver(desafio)
        Coordinador!enviarResuelto(id, desafioResuelto)
        Supervisor[*]?correccion(estaBien)
    }
}

process Supervisor [id: 0..S-1] {
    string desafio
    bool correccion
    int idCompetidor
    while true {
        Coordinador!estoyLibre(id)
        Coordinador?recibirTrabajo(idCompetidor, desafio)
        correccion = Corregir(desafio)
        Competidor[idCompetidor]!correccion(correccion)
    }
}

process Coordinador {
    int idSupervisor, idCompetidor
    string desafio
    Cola pedidos(int, string)

    while true {
        if
        [] ;Competidor[*]?enviarResuelto(idCompetidor, desafio) -> {
            pedidos.push(idCompetidor, desafio)
        }
        [] not empty(pedidos); Supervisor[*]?estoyLibre(idSupervisor) -> {
            idCompetidor, desafio = pedidos.pop()
            Supervisor[idSupervisor]!recibirTrabajo(idCompetidor, desafio)
        }
        fi
    }
}
```

## - ✅

En un comedor estudiantil hay un horno microondas que debe ser usado por E estudiantes de acuerdo con el orden de llegada. Cuando el estudiante accede al horno, lo usa y luego se retira para dejar al siguiente.
**Nota: cada Estudiante usa sólo una vez el horno.**

```cs
process Estudiante[id: 0..E-1] {
    Coordinador!pedir(id)
    Coordinador?miTurno()
    HornoUsar()
    Coordinador!liberar()
}

process Coordinador {
    Cola pedidos(int);
    bool libre = true
    int idEstudiante
    while true {
        if
        [] ;Estudiante[*]?pedir(id) -> {
            if libre {
                libre = false
                Estudiante[id]!miTurno()
            }
            else
                pedidos.push(id)
        }
        [] ;Estudiante[*]?liberar() -> {
            if pedidos.isEmpty()
                libre = true
            else
                Estudiante[pedidos.pop()]!miTurno()
        }
    }
}
```
