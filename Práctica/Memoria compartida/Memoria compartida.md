# Variables compartidas

- Siempre tratar de usar el < > lo menos posible, ya que reduce la concurrencia. Además, nunca deberíamos poner nada que no sea totalmente necesario dentro de los < >. Por ejemplo, en vez de incrementar una variable compartida en forma atómica dentro de un for podemos hacerlo al final de las iteraciones, utilizando, en los procesos, variables locales contadoras.
- Que la cola de procesos esperando para usar un recurso compartido este vacía no nos asegura que el recurso compartido no esté siendo utilizado actualmente. Podría haber un último proceso usando el recurso.

# Semáforos

# Monitores