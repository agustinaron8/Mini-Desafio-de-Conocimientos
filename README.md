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

Fragmento de código:

```python
def registrar(
        self,
        usuario_id,
        libro_isbn,
        fecha_prestamo,
        fecha_devolucion,
        estado="Pendiente de Devolución",
    ):
        """
        Registra un préstamo en la base de datos si no existe previamente.
        """
        try:
            gestor_libros = Gestor_Libros()

            # Comprobar si ya existe un préstamo pendiente para el mismo libro y usuario
            cursor = self.db.get_connection().cursor()
            cursor.execute(
                """
                SELECT COUNT(*) FROM prestamo
                WHERE usuario_id = ? AND libro_isbn = ? AND estado = 'Pendiente de Devolución'
                """,
                (usuario_id, libro_isbn),
            )
            prestamo_existente = cursor.fetchone()[0]

            if prestamo_existente > 0:
                print("Ya existe un préstamo pendiente para este libro y usuario.")
                return False

            # Obtener datos del libro
            cursor.execute("SELECT * FROM libro WHERE isbn = ?", (libro_isbn,))
            libro_data = cursor.fetchone()

            if not libro_data:
                print("El libro con ese ISBN no existe.")
                return False

            # Crear una instancia de Libro a partir de los datos obtenidos
            libro = Libro(*libro_data)
            nueva_cantidad = libro.cantidad - 1

            if libro.cantidad <= 0:
                print("No hay cantidad disponible.")
                return False

            # Insertar el préstamo
            cursor.execute(
                "INSERT INTO prestamo (usuario_id, libro_isbn, fecha_prestamo, fecha_devolucion, estado) VALUES (?, ?, ?, ?, ?)",
                (usuario_id, libro_isbn, fecha_prestamo, fecha_devolucion, estado),
            )

            # Actualizar cantidad del libro con el orden correcto de parámetros
            gestor_libros.modificar(
                libro.titulo,
                libro.genero,
                libro.anio_publicacion,
                libro.autor_id,
                nueva_cantidad,
                libro.isbn,
            )

            self.db.get_connection().commit()  # Confirmar transacción
            self.notificar()  # Notificar si aplica
            return True

        except Exception as e:
            print(f"Error al registrar el préstamo: {e}")
            self.db.get_connection().rollback()
            return False
```

## 🌐Patrones de Diseño:
¿Podrías mencionar y describir brevemente dos patrones de diseño de software que
hayas utilizado en tus proyectos? ¿Cómo contribuyeron estos patrones a la eficiencia o
mantenibilidad del código?

En el Trabajo Práctico recién mencionado, utilicé el patrón singleton para asegurar el acceso a una única instancia de base de datos y para las interfaces gráficas que mostraban cada módulo, este patrón ayudó a lograr clase cohesivas, con responsabilidades simples y claras.

También el utilicé el patrón Factory para la creación de los distintos tipos de usuario (alumno y profesor), que contenían características comunes, permitiendo la extensión futura de distintos tipos de usuarios escalando horizontalmente a la altura de las subclases actuales Profesor y Estudiante, reutilizando el código de la clase plantilla Usuario.

Además de estos patrones estructurales, conozco patrones de comportamiento como pueden ser Strategy o State, que favorecen la delegación de responsabilidades y el bajo acoplamiento de las clases del sistema.

## ️ Interfaces en Programación:
Explícanos qué es una interfaz en programación y proporciona un ejemplo de cómo y
cuándo usarías una en un proyecto. Mostrá una firma de interfaz (código breve) y un
caso de sustitución.

Una interfaz funciona a modo de contrato, establece (sin dar implementación) el conjunto de métodos que una clase debe implementar para realizarla. Con ella garantizamos que estas clases "realizadoras" de la interfaz compartan cierta funcionalidad(pudiendo implementarlas de distinto modo internamente), permitiendo el polimorfismo en función de las solicitudes concretas que tenga la interfaz en tiempo de ejecución.
Ayudan a desacoplar el código y que sea más simple de mantener.

Usaría una interfaz por ejemplo aplicando el patrón Strategy, en una situación en la que tenga que resolver de distintas formas una situación.
Me gusta mucho el fútbol y es uno de mis hobbies, así que voy con un ejemplo de ese tipo:

En el fútbol europeo, se otorga la bota de oro de una temporada al jugador que sume más puntos en función del siguiente cálculo:

```
Coeficiente de la liga * goles anotados

siendo el coeficiente distinto según el nivel de la liga:
Una liga de 1er orden tiene un multiplicador x2.
Una de 2do orden tiene un multiplicador x1,5.
Una de 3er orden tiene un multiplicador x1.
```
La interfaz del ejemplo podría verse así en JAVA:
``` Java
public interface CalculadorPuntos {
    double calcularPuntos(int goles);
}
```
Y sus implementaciones:
``` Java
public class LigaPrimerOrden implements CalculadorPuntos {
    @Override
    public double calcularPuntos(int goles) {
        return goles * 2.0;
    }
}

public class LigaSegundoOrden implements CalculadorPuntos {
    @Override
    public double calcularPuntos(int goles) {
        return goles * 1.5;
    }
}

public class LigaTercerOrden implements CalculadorPuntos {
    @Override
    public double calcularPuntos(int goles) {
        return goles * 1.0;
    }
}
```
Y un caso de sustitución:
``` Java
public class Jugador {
    private String nombre;
    private CalculadorPuntos calculador;

    public Jugador(String nombre, CalculadorPuntos calculador) {
        this.nombre = nombre;
        this.calculador = calculador;
    }

    public void mostrarPuntos(int goles) {
        System.out.println(nombre + " tiene " + calculador.calcularPuntos(goles) + " puntos.");
    }

    public static void main(String[] args) {
        Jugador messi = new Jugador("Messi", new LigaPrimerOrden());
        Jugador ronaldo = new Jugador("Ronaldo", new LigaSegundoOrden());

        messi.mostrarPuntos(30);  
        ronaldo.mostrarPuntos(30);
    }
}
```