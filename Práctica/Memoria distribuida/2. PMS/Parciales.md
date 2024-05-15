# Pasaje de Mensajes Sincrónico

-   ✅ significa que el ejercicio está chequeado y es correcto.
-   ❓ significa que el ejercicio falta ser chequeado.

## - ❓

En un torneo de programación hay 1 organizador, N competidores y S supervisores. El organizador comunica el desafío a resolver a cada competidor. Cuando un competidor cuenta con el desafío a resolver, lo hace y lo entrega para ser evaluado. A continuación, espera a que alguno de los supervisores lo corrija y le indique si está bien. En caso de tener errores, el competidor debe corregirlo y volver a entregar, repitiendo la misma metodología hasta que llegue a la solución esperada. Los supervisores corrigen las entregas respetando el orden en que los competidores van entregando.
**Nota: maximizar la concurrencia y no generar demora innecesaria.**

```cs
process Organizador {
    int id
    for i = 0 to N-1 {
        string desafio = Generar()
        Competidor[*]?recibirID(id)
        Competidor[id]!enviarDesafio(desafio)
    }
}
process Competidor [id: 0..N-1] {
    bool ok = false
    string desafio, desafioResuelto
    Organizador!recibirID(id)
    Organizador?enviarDesafio(desafio)
    while not ok {
        desafioResuelto = Resolver(desafio)
        Coordinador!enviarResuelto(desafioResuelto, id)
        Supervisor[*]?estaBien(ok)
    }
}
process Supervisor [id: 0..S-1] {
    string desafio
    bool correccion = false
    int idCompetidor
    while true {
        Coordinador!estoyLibre(id)
        Coordinador?recibirTrabajo(desafio, idCompetidor)
        correccion = Corregir(desafio)
        Competidor[idCompetidor]!estaBien(correccion)
    }
}
process Coordinador {
    int idSupervisor, idCompetidor
    string desafio
    cola pedidos
    while true {
        if(Competidor[*]?enviarResuelto(desafio, idCompetidor)) ->
            pedidos.push(desafio, idCompetidor)
        [] not empty(pedidos); Supervisor[*]?estoyLibre(idSupervisor) -> {
            (desafio, idCompetidor) = pedidos.pop()
            Supervisor[idSupervisor]!recibirTrabajo(desafio, idCompetidor)
            }
        end if
    }
}
```

## - ❓

En un comedor estudiantil hay un horno microondas que debe ser usado por E estudiantes de acuerdo con el orden de llegada. Cuando el estudiante accede al horno, lo usa y luego se retira para dejar al siguiente.
**Nota: cada Estudiante una sólo una vez el horno.**

```cs
process Estudiante[id: 0..E-1] {
    Coordinador!pedir(id)
    Coordinador?miTurno()
    Horno.usar()
    Coordinador!liberar()
}
process Coordinador {
    cola pedidos;
    bool libre = true
    int idEstudiante
    while true {
        if libre; Estudiante[*]?pedir(id) -> {
            libre = false
            Estudiante[id]!miTurno()
            }
        [] not libre; Estudiante[*]?pedir(id) -> {
            pedidos.push(id)
            }
        [] Estudiante[*]?liberar() {
            if pedidos.isEmpty()
                libre = true
            else
                Estudiante[pedidos.pop()]!miTurno()
            }
        }
}
```
