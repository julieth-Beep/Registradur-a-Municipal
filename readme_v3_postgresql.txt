SISTEMA DE REGISTRO CIVIL - REGISTRADURÍA MUNICIPAL DE NOBSA

Aprendiz     : Julieth Dayana Peña Barrera
Ficha        : 3171062
Programa     : Tecnólogo en Análisis y Desarrollo de Software – ADSO
Centro       : CIMM – Regional Boyacá

1. MOTOR DE BASE DE DATOS ELEGIDO: POSTGRESQL 15 (pgAdmin 4)

Herramienta de administración utilizada: pgAdmin 4

JUSTIFICACIÓN DE LA ELECCIÓN:
PostgreSQL 15 fue seleccionado como motor para el Sistema de Registro Civil de la
Registraduría Municipal de Nobsa por las siguientes razones técnicas y contextuales:

En primer lugar, PostgreSQL es un motor de código abierto con soporte completo para
transacciones ACID, lo que garantiza la integridad de datos civiles críticos como
registros de nacimiento, cédulas y documentos electorales. Una transacción
interrumpida (por corte de energía o fallo de red) nunca dejará la base de datos en
estado inconsistente.

En segundo lugar, el costo cero de licenciamiento es determinante para una entidad
pública con presupuesto limitado. PostgreSQL no tiene restricciones de RAM, CPU ni
número de bases de datos como SQL Server Express.

En tercer lugar, PostgreSQL es el motor en el que se basa Supabase, por lo que una
futura migración a la nube requeriría cambios mínimos. pgAdmin 4 proporciona una
interfaz web administrativa accesible desde cualquier navegador, sin necesidad de
instalar software adicional en cada puesto de trabajo del personal de la
registraduría.

Finalmente, para una concurrencia baja (< 100 usuarios simultáneos típicos en una
registraduría municipal), PostgreSQL con su configuración predeterminada es más que
suficiente y ofrece rendimiento superior a SQLite o H2 en escenarios multiusuario.

2. PASOS PARA EJECUTAR EL PROYECTO

PRE-REQUISITOS:
  - JDK 17 o superior instalado y configurado en PATH
  - Apache Tomcat 10.x descomprimido y configurado
  - PostgreSQL 15 instalado y servicio activo
  - pgAdmin 4 instalado (incluido en el instalador de PostgreSQL para Windows)
  - Maven 3.8+ disponible en PATH

PASO 1 – Crear la base de datos en pgAdmin 4:
  a. Abrir pgAdmin 4 en el navegador (por defecto: http://localhost/pgadmin4)
  b. Conectarse al servidor local (Servers > PostgreSQL 15 > Connect)
  c. Clic derecho en "Databases" → Create → Database
     Nombre: registraduria_nobsa, Owner: postgres
  d. Seleccionar la BD registraduria_nobsa → clic derecho → Query Tool
  e. Abrir y ejecutar el archivo sql/registraduria_nobsa.sql
  f. Verificar con: SELECT COUNT(*) FROM ciudadanos; -- debe retornar 5

PASO 2 – Configurar db.properties:
  Editar src/main/resources/db.properties:
    db.engine=postgresql
    postgresql.url=jdbc:postgresql://localhost:5432/registraduria_nobsa
    postgresql.user=postgres
    postgresql.password=[CONTRASEÑA_CONFIGURADA_EN_INSTALACIÓN]

PASO 3 – Compilar el proyecto:
  Abrir terminal en la raíz del proyecto:
    mvn clean package
  Se generará: target/registraduria-nobsa.war

PASO 4 – Desplegar en Tomcat:
  Copiar registraduria-nobsa.war a [TOMCAT_HOME]/webapps/
  Iniciar Tomcat:
    Windows: [TOMCAT_HOME]/bin/startup.bat
    Linux:   [TOMCAT_HOME]/bin/startup.sh
  Acceder a: http://localhost:8080/registraduria-nobsa/

SOLUCIÓN DE PROBLEMAS COMUNES:
  - "Connection refused localhost:5432": verificar que el servicio PostgreSQL
    esté activo (Windows: Servicios → postgresql-x64-15 → Iniciado)
  - "password authentication failed": confirmar la contraseña del usuario
    postgres abriendo pgAdmin 4 con esas mismas credenciales.


3. RESPUESTAS A LAS PREGUNTAS DE REFLEXIÓN Y ANÁLISIS

────────────────────────────────────────────────────────────────────────────────
SECCIÓN 12.1 – PREGUNTAS CONCEPTUALES
────────────────────────────────────────────────────────────────────────────────

PREGUNTA 1:
¿Qué ventaja concreta ofrece el patrón DAO respecto a escribir SQL directamente
en el Servlet? ¿Cambiarías de motor sin el patrón DAO? ¿Cuántas líneas modificarías?

RESPUESTA:
El patrón DAO (Data Access Object) actúa como una frontera bien definida entre dos
responsabilidades que no deberían mezclarse: el control del flujo HTTP (Servlet) y
el acceso a la base de datos (DAO). Esta separación se traduce en una ventaja
concreta inmediata: cambiar el motor de base de datos o incluso reemplazar la
tecnología de persistencia completa (de SQL a una API REST, por ejemplo) no afecta
una sola línea de los Servlets ni de las vistas JSP.

Desde la perspectiva del mantenimiento, el DAO centraliza todo el SQL en un único
lugar: si se descubre un error en una consulta, se corrige en un solo archivo
sin rastrear en qué Servlet está el problema. Esto reduce el tiempo de depuración
considerablemente.

Sin el patrón DAO, una migración de PostgreSQL a SQL Server implicaría rastrear
manualmente cada consulta SQL embebida en los métodos doGet y doPost de los
Servlets. Para este proyecto con tres Servlets y 12 operaciones de base de datos
en total (CRUD ciudadanos, CRUD documentos, 3 consultas electorales con JOINs),
la estimación de líneas a modificar supera las 80, con riesgo alto de olvidar
alguna sentencia o introducir errores al reescribirlas.

────────────────────────────────────────────────────────────────────────────────

PREGUNTA 2:
¿Qué es JDBC y cuál es el rol del driver? ¿Por qué Supabase usa el mismo driver
que PostgreSQL? ¿Por qué H2 no necesita instalar un servidor?

RESPUESTA:
JDBC (Java Database Connectivity) es una API estándar incluida en el JDK que
establece un conjunto de interfaces (Connection, PreparedStatement, ResultSet,
DatabaseMetaData, entre otras) que toda implementación de acceso a datos relacionales
debe cumplir. Gracias a esta abstracción, el código de la aplicación es idéntico
independientemente del motor; solo el driver cambia.

El driver es el componente que implementa esas interfaces para un motor concreto.
Cuando se llama a DriverManager.getConnection("jdbc:postgresql://..."), el driver
de PostgreSQL establece una conexión TCP al puerto 5432, autentica al usuario según
el protocolo pg_wire, y retorna un objeto Connection que el código Java puede usar
sin conocer ningún detalle del protocolo subyacente.

Supabase utiliza el driver de PostgreSQL (org.postgresql.Driver) porque Supabase
no es un motor de base de datos original: es PostgreSQL desplegado en la nube con
servicios adicionales de autenticación, almacenamiento y API REST construidos
alrededor. El protocolo de red con el que acepta conexiones JDBC es exactamente el
mismo pg_wire, por eso el driver es intercambiable.

H2 no requiere instalación de servidor porque implementa el motor completo como
código Java. La clase org.h2.Driver, al recibir una URL de tipo jdbc:h2:mem:,
instancia el motor directamente dentro de la JVM de Tomcat. No hay proceso
separado, no hay socket de red, no hay instalador. La base de datos existe como
objetos en el heap de Java y desaparece cuando la JVM se detiene.

────────────────────────────────────────────────────────────────────────────────

PREGUNTA 3:
¿Qué es un PreparedStatement y por qué es superior a la concatenación de Strings?
¿Qué riesgo de seguridad mitiga específicamente?

RESPUESTA:
Un PreparedStatement es una sentencia SQL que se envía al motor de base de datos
en dos fases separadas: primero la estructura de la consulta con marcadores de
posición (?), y luego los valores de los parámetros de forma independiente. El
motor procesa la estructura en la primera fase y puede reutilizar el plan de
ejecución en llamadas posteriores con distintos valores, lo que mejora el
rendimiento.

La superioridad sobre la concatenación de Strings es principalmente de seguridad.
Cuando se construye una consulta concatenando directamente el input del usuario,
por ejemplo:

  String sql = "SELECT * FROM ciudadanos WHERE numero_documento = '" + doc + "'";

Un usuario malintencionado puede introducir como valor del campo el texto:
  ' OR '1'='1' --

Lo que produce la consulta:
  SELECT * FROM ciudadanos WHERE numero_documento = '' OR '1'='1' --'

Esta consulta retorna todos los registros de la tabla y puede comprometer datos
civiles completos de la registraduría. Esto se denomina SQL Injection y es la
vulnerabilidad más explotada en aplicaciones web según el ranking OWASP Top 10.

Con PreparedStatement, el valor introducido por el usuario se envía al servidor
PostgreSQL como un parámetro de tipo texto vinculado (bind parameter). El motor
nunca lo interpreta como código SQL; solo busca ese texto literal como valor en la
columna numero_documento, haciendo el ataque imposible.

────────────────────────────────────────────────────────────────────────────────

PREGUNTA 4:
¿Qué problema grave ocurriría si no se cierran Connection, PreparedStatement y
ResultSet? ¿Cómo se llama ese problema?

RESPUESTA:
En PostgreSQL, cada objeto Connection corresponde a un proceso backend (postgres)
en el servidor. Ese proceso consume entre 5 MB y 10 MB de memoria del servidor,
además de mantener bloqueos y buffers de transacción activos. Si una aplicación
abre conexiones y no las cierra, el servidor acumula procesos backend inactivos
indefinidamente.

PostgreSQL tiene un parámetro max_connections (por defecto 100 conexiones). Cuando
el número de conexiones abiertas alcanza ese límite, cualquier intento adicional
de conectarse falla con el error "FATAL: sorry, too many clients already". En ese
momento la aplicación deja de funcionar para todos los usuarios, aunque el servidor
físico esté con recursos disponibles.

Este problema se denomina fuga de recursos (resource leak), y cuando afecta
específicamente a las conexiones de base de datos se llama fuga de conexiones
(connection leak). Es uno de los errores más frecuentes y difíciles de diagnosticar
en aplicaciones web porque los síntomas (lentitud creciente, fallos esporádicos)
solo aparecen bajo carga real.

El bloque try-with-resources resuelve este problema de forma garantizada: al salir
del bloque, ya sea por flujo normal o por excepción, Java invoca automáticamente
el método close() de cada AutoCloseable declarado en el encabezado del try,
asegurando que ninguna conexión quede abierta accidentalmente.

────────────────────────────────────────────────────────────────────────────────
SECCIÓN 12.2 – PREGUNTAS DE ANÁLISIS
────────────────────────────────────────────────────────────────────────────────

PREGUNTA 5:
H2 modo memoria pierde datos al reiniciar. ¿Cuándo es ventaja y cuándo problema?
Dos proyectos con modo memoria y dos con modo archivo.

RESPUESTA:
La volatilidad del modo memoria se convierte en ventaja cuando el ciclo de vida
esperado de los datos coincide exactamente con el tiempo de vida del proceso JVM.
Si los datos solo tienen sentido durante una ejecución (como los resultados de una
prueba), borrarlos al reiniciar no es una pérdida sino una limpieza automática.

PROYECTOS DONDE H2 MODO MEMORIA ES LA OPCIÓN CORRECTA:

  1. Framework de pruebas automatizadas del sistema de registraduría (JUnit):
     Antes de cada clase de prueba se ejecuta el schema_h2.sql con los 5
     ciudadanos y documentos de prueba. Las pruebas insertan, modifican y
     eliminan registros sin afectar ninguna BD real. Al finalizar, los datos
     simplemente desaparecen. Esto garantiza pruebas deterministas y velocidad
     de ejecución alta.

  2. Pipeline de integración continua (CI/CD con GitHub Actions o Jenkins):
     En cada commit, el sistema compila el proyecto y ejecuta las pruebas
     de integración contra H2 en memoria. No se necesita un servidor PostgreSQL
     en el agente de CI, reduciendo costos y tiempo de configuración del entorno.

PROYECTOS DONDE H2 MODO ARCHIVO ES PREFERIBLE:

  1. Prototipo validado con el registrador municipal de Nobsa: durante la fase
     de validación de requisitos, el funcionario introduce datos reales de prueba
     en sesiones de trabajo distribuidas en varios días. Los datos deben
     sobrevivir entre sesiones; H2 en archivo (jdbc:h2:~/registraduria_nobsa)
     lo permite sin instalar PostgreSQL.

  2. Aplicación de escritorio para inspectores de campo: un inspector puede
     llevar una laptop sin conexión a internet, capturar datos en campo y luego
     sincronizar. H2 en archivo es ideal para ese escenario de conectividad
     intermitente con un único usuario a la vez.

────────────────────────────────────────────────────────────────────────────────

PREGUNTA 6:
Compara H2 y SQLite. ¿Diferencias clave? ¿Cuándo elegir una u otra?

RESPUESTA:
NATURALEZA TECNOLÓGICA:
  H2 es un motor escrito íntegramente en Java, sin dependencias nativas. Se integra
  como cualquier librería Maven y corre en cualquier JVM sin importar el sistema
  operativo. SQLite está escrito en C y requiere un binding nativo (xerial/sqlite-jdbc
  descarga el .so o .dll apropiado al iniciarse); en algunos entornos restringidos,
  las librerías nativas pueden representar un problema de seguridad o compatibilidad.

MODELO DE TIPOS:
  PostgreSQL y H2 comparten una filosofía de tipos estrictos: una columna DATE solo
  acepta fechas, una DECIMAL guarda precisión exacta. SQLite usa "type affinity":
  los tipos son sugerencias, no restricciones. Esto puede generar sorpresas al
  recuperar datos (por ejemplo, un precio almacenado como texto en lugar de número).

MODOS DE OPERACIÓN:
  H2 tiene tres modos: embebido, en servidor TCP y en memoria. El modo servidor
  permite múltiples clientes Java desde distintos procesos. SQLite solo soporta
  acceso embebido: un proceso a la vez puede escribir; lecturas concurrentes son
  posibles pero con restricciones.

INTEROPERABILIDAD:
  SQLite es el formato de archivo más portable del mundo: tiene drivers para Python,
  R, Swift, Go, Kotlin, C#, Ruby y prácticamente cualquier lenguaje de programación.
  H2 solo puede ser leído por aplicaciones Java o herramientas compatibles con JDBC.

RECOMENDACIÓN PRÁCTICA:
  Elegir H2 en proyectos Java puros, para pruebas automatizadas y cuando la
  coherencia tecnológica del stack es importante.
  Elegir SQLite cuando el archivo de datos debe ser consumido por herramientas
  en otros lenguajes (análisis de datos en Python, aplicaciones móviles), o cuando
  la portabilidad del archivo .db como artefacto es un requisito explícito.

────────────────────────────────────────────────────────────────────────────────

PREGUNTA 7:
Migración de H2 a PostgreSQL en producción. ¿Cuántas líneas modifica con DAO?
¿Y sin DAO?

RESPUESTA:
ESCENARIO CON PATRÓN DAO (implementación actual):

  Archivos Java a modificar:
    - ConexionDB.java: activar el caso "postgresql" en el switch y leer las
      credenciales correctas (5 a 8 líneas).
    - db.properties: cambiar db.engine=h2 por db.engine=postgresql y actualizar
      la URL y contraseña (3 líneas).
  
  Archivos SQL a ajustar (no son código Java):
    - registraduria_nobsa.sql: adaptar INT AUTO_INCREMENT a SERIAL, confirmar
      que CURRENT_TIMESTAMP y DATE son sintaxis válida en PostgreSQL (lo son).
  
  Total en código Java: 8 a 12 líneas modificadas.
  CiudadanoDAOImpl, DocumentoDAOImpl, ConsultaMesaDAOImpl, los tres Servlets
  y todas las vistas JSP permanecen sin ningún cambio.

ESCENARIO SIN PATRÓN DAO (SQL embebido en Servlets):

  Cada Servlet contiene sus propias consultas SQL. Habría que localizar y revisar:
    - CiudadanoServlet: insertar, listar, buscar, eliminar → 4 bloques SQL
    - DocumentoServlet: insertar, listar por ciudadano, listar global con JOIN,
      cambiar estado → 4 bloques SQL
    - ConsultaMesaServlet: consulta con JOIN 4 tablas, listar mesas, listar zonas
      → 3 bloques SQL
  
  Total: 11 bloques SQL distribuidos en 3 archivos Java. Cada bloque implica
  revisar tipos de datos (¿se usó DATETIME2 de H2 en lugar de TIMESTAMP?),
  funciones de fecha (NOW() vs CURRENT_TIMESTAMP), y cadenas de conexión.
  Estimación: entre 65 y 90 líneas de código Java a modificar, con alto riesgo de
  dejar algún bloque sin actualizar y descubrir el error solo en tiempo de ejecución.

────────────────────────────────────────────────────────────────────────────────

PREGUNTA 8:
Ferretería Los Andes de Tunja: 200 usuarios concurrentes. ¿Qué motor elegir?

RESPUESTA:
Con 200 usuarios concurrentes la elección se reduce a tres candidatos: PostgreSQL,
SQL Server y Supabase. Los motores embebidos (SQLite, H2) no son adecuados para
este nivel de concurrencia.

ANÁLISIS COMPARATIVO PARA ESTE ESCENARIO:

  PostgreSQL: motor open source con excelente manejo de concurrencia, sin costo de
  licencia, y soporte probado en miles de aplicaciones empresariales con cargas
  mucho mayores a 200 usuarios simultáneos. Con un pool HikariCP de 20 conexiones
  y un servidor con 4 GB de RAM, maneja esta carga sin dificultades.

  SQL Server Express: gratuito pero con límite de 1 GB de RAM para el buffer pool
  y 10 GB de tamaño de base de datos. Para una ferretería con catálogo extenso y
  200 usuarios, ese límite de RAM puede convertirse en un cuello de botella. Las
  ediciones con licencia eliminan ese límite pero implican costo significativo.

  Supabase: viable si la ferretería tiene conexión a internet estable. Elimina la
  carga de administración del servidor, pero introduce dependencia de terceros y
  latencia de red en cada consulta (aproximadamente 30-80 ms adicionales por
  operación comparado con servidor local).

RECOMENDACIÓN FINAL: PostgreSQL 15 en servidor local con HikariCP.
Es la opción con mejor relación costo/rendimiento/control para una empresa
mediana sin presupuesto de licenciamiento. Puede escalar a réplicas de lectura
si la carga aumenta en el futuro.

INFORMACIÓN ADICIONAL NECESARIA ANTES DE DECIDIR:
  - ¿Tienen servidor físico dedicado o solo equipos de escritorio compartidos?
  - ¿El personal técnico tiene experiencia administrando Linux o solo Windows?
  - ¿Hay operaciones de escritura intensivas (punto de venta en tiempo real) o
    principalmente lecturas de catálogo?
  - ¿Requieren acceso desde sucursales remotas (Duitama, Sogamoso)?

────────────────────────────────────────────────────────────────────────────────
SECCIÓN 12.3 – PREGUNTA DE INVESTIGACIÓN
────────────────────────────────────────────────────────────────────────────────

PREGUNTA 9:
¿Qué es un Connection Pool y por qué es esencial? Dos librerías. ¿H2 lo soporta?
¿Cómo modificar ConnectionFactory?

RESPUESTA:
QUÉ ES UN CONNECTION POOL:
Un Connection Pool es un componente de middleware que gestiona un conjunto de
conexiones JDBC abiertas y las reutiliza entre múltiples peticiones HTTP. Al iniciar
la aplicación, el pool abre un número configurable de conexiones al servidor
PostgreSQL (por ejemplo, 10 conexiones). Cada petición entrante toma una conexión
libre del pool, ejecuta sus operaciones y la devuelve; la conexión no se cierra
sino que queda disponible para la siguiente petición.

POR QUÉ ES ESENCIAL EN APLICACIONES MULTIUSUARIO:
Establecer una conexión con PostgreSQL implica: resolver la dirección del servidor,
completar el handshake TCP, enviar el mensaje de autenticación, recibir el
BackendKeyData y esperar el ReadyForQuery. Todo esto tarda entre 30 ms y 150 ms
en un servidor local y entre 100 ms y 500 ms en un servidor remoto. Sin pool,
200 usuarios concurrentes esperarían esos milisegundos adicionales en cada operación,
degradando la experiencia perceptiblemente. Con pool, el tiempo de obtención de
conexión se reduce a microsegundos.

LIBRERÍAS JAVA DE CONNECTION POOLING:

  1. HikariCP (recomendada):
     Considerada la implementación más eficiente disponible. Su tiempo de obtención
     de conexión del pool es inferior a 1 microsegundo en condiciones normales.
     Configuración mínima (solo URL, usuario, contraseña y tamaño del pool) y
     zero overhead comparado con un DataSource nativo. Es la opción predeterminada
     en Spring Boot y Quarkus.

  2. Apache DBCP2 (Database Connection Pooling):
     Desarrollada por la Apache Software Foundation. Más configurable que HikariCP,
     con soporte para validación de conexiones mediante queries de prueba, políticas
     de abandono de conexiones y estadísticas detalladas. Adecuada cuando se
     necesita control fino sobre el ciclo de vida de cada conexión.

H2 Y CONNECTION POOLING:
  H2 en modo embebido soporta connection pooling siempre que la URL incluya
  DB_CLOSE_DELAY=-1, que impide que H2 cierre y destruya la base de datos en
  memoria cuando el pool devuelve la última conexión. H2 en modo servidor TCP
  (jdbc:h2:tcp://localhost/~/nombre_bd) funciona con cualquier pool estándar
  sin configuración adicional.

MODIFICACIÓN DE ConexionDB.java CON HIKARICP PARA POSTGRESQL:

  // 1. Añadir en pom.xml:
  // <dependency>
  //   <groupId>com.zaxxer</groupId>
  //   <artifactId>HikariCP</artifactId>
  //   <version>5.1.0</version>
  // </dependency>

  import com.zaxxer.hikari.HikariConfig;
  import com.zaxxer.hikari.HikariDataSource;
  import java.sql.Connection;
  import java.sql.SQLException;

  public class ConexionDB {

      private static final HikariDataSource pool;

      static {
          HikariConfig config = new HikariConfig();
          config.setJdbcUrl(
              "jdbc:postgresql://localhost:5432/registraduria_nobsa");
          config.setUsername("postgres");
          config.setPassword("sena2025");
          config.setMaximumPoolSize(10);  // conexiones máximas simultáneas
          config.setMinimumIdle(2);       // conexiones siempre listas
          config.setIdleTimeout(30000);   // ms antes de cerrar conexión ociosa
          config.setConnectionTimeout(5000); // ms para obtener conexión del pool
          pool = new HikariDataSource(config);
      }

      public static Connection obtenerConexion() throws SQLException {
          // No abre nueva conexión: toma una del pool existente
          return pool.getConnection();
      }
  }

  Con esta implementación, los DAOs existentes (CiudadanoDAOImpl,
  DocumentoDAOImpl, ConsultaMesaDAOImpl) no requieren ninguna modificación:
  seguirán llamando a ConexionDB.obtenerConexion() y el try-with-resources
  devolverá la conexión al pool al finalizar, en lugar de cerrarla físicamente.

