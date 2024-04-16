<center>

# Cuestionario teorías 1 y 2

</center>

### 1. Mencione al menos 3 ejemplos donde pueda encontrarse concurrencia

### 2. Escriba una definición de concurrencia. Diferencie procesamiento secuencial, concurrente y paralelo

### 3. Describa el concepto de deadlock y qué condiciones deben darse para que ocurra.

### 4. Defina inanición. Ejemplifique.

### 5. ¿Qué entiende por no determinismo? ¿Cómo se aplica este concepto a la ejecución concurrente?

### 6. Defina comunicación. Explique los mecanismos de comunicación que conozca.

### 7.

##### a) Defina sincronización. Explique los mecanismos de sincronización que conozca.

##### b) ¿En un programa concurrente pueden estar presentes más de un mecanismo de sincronización? En caso afirmativo, ejemplifique

### 8. ¿Qué significa el problema de “interferencia” en programación concurrente? ¿Cómo puede evitarse?

### 9. ¿En qué consiste la propiedad de “A lo sumo una vez” y qué efecto tiene sobre las sentencias de un programa concurrente? De ejemplos de sentencias que cumplan y de sentencias que no cumplan con ASV.

### 10. Dado el siguiente programa concurrente:

```cs
x = 2; y = 4; z = 3;
co
   P1        || P2         || P3
   x = y - z || z = x * 2 || y = y - 1
oc
```

##### a) ¿Cuáles de las asignaciones dentro de la sentencia co cumplen con ASV? Justifique claramente.

##### b) Indique los resultados posibles de la ejecución. Nota 1: las instrucciones NO SON atómicas. Nota 2: no es necesario que liste TODOS los resultados, pero si los que sean representativos de las diferentes situaciones que pueden darse.

### 11. Defina acciones atómicas condicionales e incondicionales. Ejemplifique.

### 12. Defina propiedad de seguridad y propiedad de vida.

### 13. ¿Qué es una política de scheduling? Relacione con fairness. ¿Qué tipos de fairness conoce?

### 14. ¿Por qué las propiedades de vida dependen de la política de scheduling? ¿Cómo aplicaría el concepto de fairness al acceso a una base de datos compartida por N procesos concurrentes?

### 15. Dado el siguiente programa concurrente, indique cuál es la respuesta correcta (justifique claramente)

```cs
int a = 1, b = 0;
co
   await (b = 1) a = 0 || while (a = 1) { b = 1; b = 0; }
oc
```

##### a) Siempre termina

##### b) Nunca termina

##### c) Puede terminar o no

---

<center>

# Cuestionario teorías 3 y 4

</center>

### 1. ¿Qué propiedades deben garantizarse en la administración de una sección crítica en procesos concurrentes? ¿Cuáles de ellas son propiedades de seguridad y cuáles de vida? En el caso de las propiedades de seguridad, ¿cuál es en cada caso el estado “malo” que se debe evitar?

### 2. Resuelva el problema de acceso a sección crítica para N procesos usando un proceso coordinador. En este caso, cuando un proceso SC[i] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le otorgue permiso. Al terminar de ejecutar su sección crítica, el proceso SC[i] le avisa al coordinador. Desarrolle una solución de grano fino usando únicamente variables compartidas (ni semáforos ni monitores).

### 3. ¿Qué mejoras introducen los algoritmos Tie-breaker, Ticket o Bakery en relación a las soluciones de tipo spin-locks?

### 4. Analice las soluciones para las barreras de sincronización desde el punto de vista de la complejidad de la programación y de la performance.

### 5. Explique gráficamente cómo funciona una butterfly barrier para 8 procesos usando variables compartidas.

![Butterfly Barrier](https://i.imgur.com/gtnHr4A.jpeg)

### 6.

##### a) Explique la semántica de un semáforo

##### b) Indique los posibles valores finales de x en el siguiente programa (justifique claramente su respuesta):

```cs
int x = 4; sem s1 = 1, s2 = 0;
co
   P(s1); x = x * x ; V(s1);

   P(s2); P(s1); x = x * 3; V(s1);

   P(s1); x = x - 2; V(s2); V(s1);
oc
```

### 7. Desarrolle utilizando semáforos una solución centralizada al problema de los filósofos, con un administrador único de los tenedores, y posiciones libres para los filósofos (es decir, cada filósofo puede comer en cualquier posición siempre que tenga los dos tenedores correspondientes).

### 8. Describa la técnica de Passing the Baton. ¿Cuál es su utilidad en la resolución de problemas mediante semáforos?

### 9. Modifique las soluciones de Lectores-Escritores con semáforos de modo de no permitir más de 10 lectores simultáneos en la BD y además que no se admita el ingreso a más lectores cuando hay escritores esperando.

---

<center>

# Cuestionario teorías 5 y 6

</center>

### 1. Describa el funcionamiento de los monitores como herramienta de sincronización.

### 2. ¿Qué diferencias existen entre las disciplinas de señalización “Signal and wait” y “Signal and continue”?

### 3. ¿En qué consiste la técnica de Passing the Condition y cuál es su utilidad en la resolución de problemas con monitores? ¿Qué relación encuentra entre passing the condition y passing the baton?

### 4. Desarrolle utilizando monitores una solución centralizada al problema de los filósofos, con un administrador único de los tenedores, y posiciones libres para los filósofos (es decir, cada filósofo puede comer en cualquier posición siempre que tenga los dos tenedores correspondientes).

### 5. Sea la siguiente solución propuesta al problema de alocación SJN:

```c
monitor SJN {
    bool libre = true;
    cond turno;
    procedure request (int tiempo) {
        if not libre
            wait(turno, tiempo);
        libre = false
    }
    procedure release() {
        libre = true
        signal(turno);
    }
}
```

##### a) Funciona correctamente con disciplina de señalización Signal and Continue?

##### b) Funciona correctamente con disciplina de señalización Signal and Wait?

### 6. Modifique la solución anterior para el caso de no contar con una instrucción wait con prioridad.

### 7. Modifique utilizando monitores las soluciones de Lectores-Escritores de modo de no permitir más de 10 lectores simultáneos en la BD, y además que no se admita el ingreso a más lectores cuando hay escritores esperando.

### 8. Resuelva con monitores el siguiente problema. Tres clases de procesos comparten el acceso a una lista enlazada: searchers, inserters y deleters. Los searchers sólo examinan la lista, y por lo tanto pueden ejecutar concurrentemente unos con otros. Los inserters agregan nuevos ítems al final de la lista; las inserciones deben ser mutuamente exclusivas para evitar insertar dos ítems casi al mismo tiempo. Sin embargo, un insert puede hacerse en paralelo con uno o más searches. Por último, los deleters remueven ítems de cualquier lugar de la lista. A lo sumo un deleter puede acceder la lista a la vez, y el borrado también debe ser mutuamente exclusivo con searches e inserciones.

### 9. El problema del “Puente de una sola vía” (One-Lane Bridge): autos que provienen del Norte y del Sur llegan a un puente con una sola vía. Los autos en la misma dirección pueden atravesar el puente al mismo tiempo, pero no puede haber autos en distintas direcciones sobre el puente.

##### a) Desarrolle una solución al problema, modelizando los autos como procesos y sincronizando con un monitor (no es necesario que la solución sea fair ni dar preferencia a ningún tipo de auto).

##### b) Modifique la solución para asegurar fairness (Pista: los autos podrían obtener turnos)

### 10. Indicar las características generales de Pthreads.

### 11. Explicar cómo se maneja la sincronización por exclusión mutua y por condición en Pthreads. Indicar la relación entre ambas. Explicar cómo se pueden simular los monitores en Pthreads.
