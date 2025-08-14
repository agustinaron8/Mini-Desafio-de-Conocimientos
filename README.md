# Mini-Desafio-de-Conocimientos
## Agustin Aron version

## ️🛠 Arquitectura de Software:
¿Podes describir un tipo de arquitectura de software que hayas utilizado en algún proyecto? ¿Cómo contribuyó esta arquitectura a cumplir con los requisitos del sistema? Incluí diagrama simple (puede ser texto ASCII) y 1 trade-off de esta arquitectura.

En un proyecto realizado para el Trabajo Práctico Integrador de la asignatura Backend de Aplicaciones (dictada en JAVA), implementé una arquitectura de microservicios con una API Gateway central que centralizaba, redireccionaba y autenticaba las peticiones, y dos servicios (Pruebas y Reportes).

Con esta arquitectura, logré cumplir con requisitos de seguridad (autenticación centralizada), escalabilidad (posibilidad de escalar un servicio si recibe más carga) y mantenibilidad (ya que cada módulo está desarrollado y desplegado de forma independiente, facilitando futuras correcciones o nuevas funcionalidades).

```
          [Cliente Web/App]
                  |
            [API Gateway]
            /           \
[Servicio Pruebas]   [Servicio Reportes]
```
Trade-off:
Si bien se ganó escalabilidad y mantenibilidad modular, se aumentó a cambio la complejidad del sistema, debido a tener que configurar la comunicación entre los servidos desplegados de forma independiente.


## ⚠ Manejo de Errores y Excepciones:
¿Cómo gestionas el manejo de errores y excepciones en tus proyectos? Proporciona un
ejemplo de cómo este enfoque mejoró la robustez de una aplicación.

Suelo utilizar en mis proyectos bloques de tipo try-catch, el primero contiene el código que al ejecutarse podría generar algún error, y el segundo contiene la forma de actúar ante ese error o excepción.

Por ejemplo, para la asignatura Diseño de Aplicaciones con Objetos (Python), el Trabajo Práctico Integrador constaba de desarrollar un sistema de gestión para una librería: En su módulo de gestión de préstamos, cada operación sobre la base de datos (registrar un préstamo o devolución, calcular un interes por demora) estaba inserta dentro de un bloque try-except en este caso, ya que Python provee esa sintaxis.
Este manejo de excepciones mejora la robustez ya que vuelve a la aplicación resiliente ante fallos, permitiendo que siga operando, por ejemplo, si al registrar un préstamo, por algún motivo no se encontraba el libro en la base de datos o había un error con la cantidad de ejemplares disponibles, el sistema revertía los cambios (rollback) y notificaba el problema sin realizar modificaciones indebidas en el stock. 