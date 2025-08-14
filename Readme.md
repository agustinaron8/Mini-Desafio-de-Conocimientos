# Mini-Desafio-de-Conocimientos
## Agustin Aron version

## ️🛠 Arquitectura de Software:
### ¿Podes describir un tipo de arquitectura de software que hayas utilizado en algún proyecto? ¿Cómo contribuyó esta arquitectura a cumplir con los requisitos del sistema? Incluí diagrama simple (puede ser texto ASCII) y 1 trade-off de esta arquitectura.

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
### ¿Cómo gestionas el manejo de errores y excepciones en tus proyectos? Proporciona un ejemplo de cómo este enfoque mejoró la robustez de una aplicación.

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
### ¿Podrías mencionar y describir brevemente dos patrones de diseño de software que hayas utilizado en tus proyectos? ¿Cómo contribuyeron estos patrones a la eficiencia o mantenibilidad del código?

En el Trabajo Práctico recién mencionado, utilicé el patrón singleton para asegurar el acceso a una única instancia de base de datos y para las interfaces gráficas que mostraban cada módulo, este patrón ayudó a lograr clase cohesivas, con responsabilidades simples y claras.

También utilicé el patrón Factory para la creación de los distintos tipos de usuario (alumno y profesor), que contenían características comunes, permitiendo la extensión futura de distintos tipos de usuarios escalando horizontalmente a la altura de las subclases actuales Profesor y Estudiante, reutilizando el código de la clase plantilla Usuario.

Además de estos patrones estructurales, conozco patrones de comportamiento como pueden ser Strategy o State, que favorecen la delegación de responsabilidades y el bajo acoplamiento de las clases del sistema.

## ️ Interfaces en Programación:
### Explícanos qué es una interfaz en programación y proporciona un ejemplo de cómo y cuándo usarías una en un proyecto. Mostrá una firma de interfaz (código breve) y un caso de sustitución.

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
Sus implementaciones:
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

## 🌟 Buenas Prácticas de Programación:
### ¿Cuáles consideras que son tres buenas prácticas esenciales en el desarrollo de software y cómo las has aplicado en tus proyectos anteriores? Nombrá 3 y contá 1 ejemplo concreto por cada una.

Apoyándome un poco en lo que ya vengo contándoles, creo que tanto el uso de Patrones de Diseño como el Manejo de Errores y Excepciones con buenas prácticas, a las que voy a sumarles escribir código limpio y legible, el uso de algoritmos eficientes y hacer Gestión de Configuración de Software.

Ya que dí ejemplos de Patrones de Diseño y Manejo de Errores y Excepciones anteriormente, voy a enforcarme en las otras tres buenas prácticas que mencioné:

- Escribir código limpio y legible:
Consiste en mantener una estructura organizada de código, usar comentarios sólo en forma necesaria y mientras sirva para mejorar el entendimiento, y seguir nomenclaturas para variables y funciones.
Por ejemplo: cameCase para variables y snake_case para funciones.
He aplicado y practicado esto a lo largo de mi carrera en múltiples proyectos y trabajos prácticos donde se requería programar.

- Uso de algoritmos eficientes:
Consiste en elegir estructuras de datos y algoritmos que favorezcan disminuir el tiempo de ejecución y el consumo de memoria, volviendo el sistema más escalable a futuro y más resiliente a sobrecargas de solicitudes.
Por ejemplo, en la materia Algoritmos y Estructuras de Datos, usaba inserciones o búsquedas binarias en lugar de lineales.

- Uso de Gestión de Configuración de Software:
Implica mantener el proyecto organizado utilizando herramientas de gestión de cambios como Git, con nomenclaturas, estructura del repositorio y criterio de línea base establecidos previamente.
Esto permite garantizar la trazabilidad de un producto, identificar versiones estables y volver a ellas en caso de ser necesario.
Por ejemplo, en la asignatura Ingeniería y Calidad de Software, mantuve junto a mi grupo un repositorio que contenía Trabajos Prácticos y Conceptuales, material de estudio, notas de clase, etc.

## 🔍 Uso de Git y Branching
### Háblanos sobre una estrategia de branching que hayas utilizado en tus proyectos. ¿Por qué elegiste esa estrategia y cómo ayudó al flujo de trabajo del equipo?

En proyectos con compañeros, hemos utilizado Branching individual, en donde se creaba una rama main y una rama por persona, estableciendo un encargado de revisar los Pull Request y responsabilidades claras sobre lo que cada uno debía programar.
Elegimos esa estrategia debido a que teníamos distintos tiempos para brindarle al desarollo del proyecto y necesitábamos un espacio de desarrollo individual, esto ayudó a poder codear cada uno en sus tiempos libres y a no pisarnos ni generar merge conflicts (modificando varias personas el mismo fragmento de código).

## 📝Pruebas de Software:
### ¿Qué tipos de pruebas de software conocés y cuál ha sido tu experiencia implementándolas?

Los tipos de pruebas pueden dividirse:
- Según el nivel:
Unitarias: Prueban funciones o clases individuales.
De Integración: Se usan para probar más de un módulo a la vez y verificar que trabajen bien juntos.
De Sistema: Prueban el total del sistema, suelen darse en un entorno que simule al entorno real.
De Aceptación: Se realizan frente al cliente o usuario final para verificar que se cumplen sus requerimientos

- Según el objetivo:
Funcionales: Prueban que el software haga lo que tiene que hacer en términos de funcionalidad.
No funcionales: Evalúan aspectos como rendimiento, escalabilidad, seguridad y usabilidad.

- Según el modo de ejecución:
Manuales: Las ejecuta un humano, generalmente siguiendo un caso de prueba.
Automatizadas: Se ejecutan por scripts o alguna herramienta de testing, algunas veces forman parte de un pipeline como uno de sus pasos.

- Según el enfoque:
De caja negra: No se conoce su código o proceso interno, evalúa entradas y salidas.
De caja blanca: Se centra en la cobertura del código, se prueban las rutas y se revisan las condiciones y bucles.

En mi experiencia, realicé Casos de Prueba para la materia Ingenieria y Calidad de Software para poder documentar bugs y que sean reproducibles.
Por otro lado, jugué un poco hace tiempo con Jenkins para realizar Pipelines que corran pruebas automatizadas previo a un merge en GitHub.

## ✅ Principios SOLID:
### ¿Estás familiarizado con los principios SOLID de la programación orientada a objetos? Si es así, elegí uno y describe cómo lo aplicaste en un proyecto real. Elegí 1 principio y pegá 6–10 líneas de código que lo muestren.
Si! Conozco los principios SOLID de programación, voy a hablar sobre el principio Open/Closed, que indica que una clase tiene que estar abierta para la extensión, pero cerrada para la modificación.
Por ejemplo, en el proyecto de gestión de librería ya mencionado, cuando apliqué este patrón al aplicar el patrón Factory para los usuarios, existiendo inicialmente Profesor y Estudiante, pero pudiendo existir a futuro otro tipo de usuario.
Realizo un ejemplo breve y reducido a continuación:
``` Python
class Usuario:
    def __init__(self, id, nombre):
        self.id = id
        self.nombre = nombre
    
class NoDocente(Usuario):
    def __init__(self, id, nombre, tipoUsuario="NoDocente"):
        super()._init_(id, nombre)
        self.tipoUsuario = tipoUsuario
```

## 📊 Bases de Datos Relacionales vs. No Relacionales:
### ¿Cuáles son las principales diferencias entre las bases de datos relacionales y no relacionales? ¿Qué criterios tomarías para definir qué tipo de base de datos era la más adecuada y por qué? Dado este caso: “logs de eventos + reportes transaccionales”, ¿qué elegís y por qué?

La principal diferencia se encuentra en la estructura, SQL (estructuradas) se basa en tablas con filas y columnas mientras que las NoSQL permiten otras formas de guardar la información como clave-valor o grafos, esto deriva en no poder almacenar todo tipo de dato en una Base de Datos SQL, como por ejemplo una imagen, pero si en una NoSQL, brindando a esta última un esquema más flexible.
En las transacciones, SQL garantiza ACID(Atómica, Consistente, Íntegra y Durable), mientras que NoSQL no.
La forma de escalar la Base de Datos también es distinta, en SQL necesitamos escalar verticalmente mejorando el servidor, y en NoSQL se puede escalar horizontalmente agregando nodos.

Para elegir una base de datos, tendría en cuenta.
- La estructura de datos y relaciones (+ estructurado -> SQL)
- Escalabilidad: NoSQL puede ser más útiles si a futuro puedo tener grandes volúmenes de datos o datos cambiantes.
- Transacciones críticas: Cuando se requiere de integridad, SQL con transacciones ACID es mejor opción.
- Consultas que voy a necesitar: SQL es más útil para consultas complejas.
- Costo: SQL puede ser más costoso por su forma de escalar verticalmente.

Con logs de eventos + reportes transaccionales:

Los logs de eventos son datos que llegan en gran volumen y necesitan de una estructura flexible, por lo que para ellos sería mejor usar una Base de Datos No Relacional.

Para los reportes transaccionales, que necesitan consistencia e integridad, usaría una Base de Datos Relacional.

Entonces, de ser posible usaría un esquema híbrido:
- NoSQL para logs de eventos.
- SQL para reportes transaccionales.

## ➿Bugs
### Contanos sobre un bug difícil: cómo lo reprodujiste, qué hipótesis probaste y cómo confirmaste la causa raíz.
En la materia Ingeniería y Calidad de Software,luego de desarrollar una User Story, tuve que testear junto a mi grupo la User Story desarrollada por otros compañeros.
Ejecutando el primer caso de prueba, que consistiía en publicar un pedido con parámetros mínimos, me encontré un bug bloqueante, ya que no podía concluir un pedido tras confirmarlo y seguir usando la aplicación.
Mi hipótesis fue que, al confirmar el pedido, se actualizaba su estado internamente pero no se disparaba el redireccionamiento a la pantalla de inicio.
Para reproducirlo, volví a abrir la aplicación, a cargar todos los parámetros mínimos para publicar el pedido, realizando los mismos pasos documentados en el caso de prueba.
Confirmé la hipótesis con una funcionalidad de consulta, los pedidos se publicaban, pero quedaba bloqueado el uso de la aplicación y sólo se podía reiniciarla para volverla a usar.
