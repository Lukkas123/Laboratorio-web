## Inyección SQL (SQLi)

La **inyección SQL** es una vulnerabilidad web que permite a un atacante manipular las consultas que una aplicación envía a su base de datos.

### Impacto
- **Acceso no autorizado a datos**: puede exponer información de otros usuarios o datos sensibles.
- **Modificación o eliminación de datos**: altera el contenido o comportamiento de la aplicación.
- **Compromiso del sistema**: en casos avanzados, puede afectar el servidor o la infraestructura backend.
- **Denegación de servicio (DoS)**: puede usarse para interrumpir el funcionamiento del sistema.
## Impacto de un ataque de Inyección SQL (SQLi)

Un ataque de **inyección SQL exitoso** puede comprometer gravemente la seguridad de una aplicación y su infraestructura.

### Consecuencias principales
- **Acceso a datos confidenciales**:
  - Contraseñas
  - Datos de tarjetas de crédito
  - Información personal de usuarios

- **Violaciones de datos (Data Breaches)**:
  - Exposición masiva de información sensible
  - Daño a la reputación de la organización
  - Sanciones y multas regulatorias

- **Persistencia del atacante**:
  - Creación de puertas traseras (*backdoors*)
  - Acceso prolongado y no detectado al sistema
## Detección de vulnerabilidades de Inyección SQL (SQLi)

La detección de **SQLi** puede realizarse de forma manual o automatizada, evaluando cómo responde la aplicación ante entradas manipuladas.

### Métodos manuales

Consiste en probar sistemáticamente cada punto de entrada (inputs, parámetros, formularios, headers, etc.):

- **Comillas simples (`'`)**  
  - Se envían para detectar errores de sintaxis o comportamientos anómalos.

- **Alteración de lógica SQL**  
  - Inyectar expresiones que cambien el resultado esperado y observar diferencias en la respuesta.

- **Condiciones booleanas**  
  - Ejemplos:
    - `OR 1=1` (verdadero)
    - `OR 1=2` (falso)  
  - Se comparan las respuestas para detectar inconsistencias.

- **Pruebas basadas en tiempo (Time-based)**  
  - Payloads que generan retrasos en la base de datos.
  - Se analiza si la respuesta tarda más de lo normal.

- **Pruebas OAST (Out-of-band)**  
  - Payloads que provocan interacciones externas (DNS/HTTP).
  - Se monitorean estas interacciones para confirmar la vulnerabilidad.

### Métodos automatizados

- **Escáneres de seguridad**:
  - Herramientas como **Burp Scanner** permiten detectar SQLi de forma rápida y confiable mediante análisis automatizados.
  - 
# Cómo detectar vulnerabilidades de inyección SQL

Se puede detectar la inyección SQL manualmente utilizando un conjunto sistemático de pruebas contra cada punto de entrada en la aplicación. Normalmente se envía:

* **El carácter de una sola cita `'`**: Para buscar errores u otras anomalías.
* **Sintaxis específica de SQL**: Que evalúa el valor original frente a un valor diferente para buscar diferencias sistemáticas.
* **Condiciones booleanas**: Como `OR 1=1` y `OR 1=2` para buscar diferencias en las respuestas.
* **Retrasos de tiempo**: Cargas útiles diseñadas para desencadenar pausas en la ejecución y medir la diferencia de tiempo en responder.
* **Cargas útiles de OAST**: Diseñadas para desencadenar una interacción de red fuera de banda (Out-of-band) y monitorear la interacción resultante.


## Inyección SQL en diferentes partes de la consulta

Aunque la mayoría de las vulnerabilidades ocurren en la cláusula `WHERE` de una consulta `SELECT`, también pueden surgir en:

* **Sentencias `UPDATE`**: Dentro de los valores actualizados o la cláusula `WHERE`.
* **Sentencias `INSERT`**: Dentro de los valores insertados.
* **Sentencias `SELECT`**: Dentro del nombre de la tabla o de la columna.
* **Sentencias `SELECT`**: Dentro de la cláusula `ORDER BY`.

---

## Ejemplos de inyección SQL

### 1. Recuperar datos ocultos
Permite modificar una consulta para devolver resultados adicionales.
* **URL:** `https://insecure-website.com/products?category=Gifts'--`
* **Consulta:** `SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`
* **Resultado:** El indicador `--` comenta el resto de la consulta, eliminando la restricción `released = 1`.

### 2. Subvertir la lógica de aplicación
Permite, por ejemplo, iniciar sesión sin una contraseña válida.
* **Usuario:** `administrator'--`
* **Consulta:** `SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`
* **Resultado:** Se accede como administrador al anular la comprobación de la contraseña.

### 3. Recuperación de datos de otras tablas
Uso de la palabra clave `UNION` para ejecutar una consulta adicional.
* **Entrada:** `' UNION SELECT username, password FROM users--`

---

## Vulnerabilidades de inyección SQL ciegas (Blind SQLi)
Ocurren cuando la aplicación no devuelve los resultados de la consulta ni detalles del error. Se explotan mediante:
* **Diferencias detectables**: Cambiar la lógica para notar cambios en la respuesta según una condición booleana.
* **Retrasos de tiempo**: Inferir información basada en el tiempo que tarda la aplicación en responder.
* **Interacción OAST**: Utilizar interacciones de red fuera de banda para exfiltrar datos (ej. búsquedas DNS).

---

## Inyección SQL de segundo orden
Ocurre cuando la aplicación almacena una entrada de usuario maliciosa en la base de datos y, posteriormente, la utiliza en una consulta SQL diferente de manera insegura. El desarrollador confía erróneamente en los datos porque ya estaban almacenados.

---

> **Advertencia:** Tenga cuidado al inyectar la condición `OR 1=1`. Si llega a una declaración `UPDATE` o `DELETE`, puede resultar en una pérdida accidental y masiva de datos.
## Ejemplos de inyección SQL

Existen muchas vulnerabilidades y técnicas que ocurren en diferentes situaciones. Algunos ejemplos comunes incluyen:

* **Recuperar datos ocultos**: Modificar una consulta para devolver resultados adicionales.
* **Subvertir la lógica de la aplicación**: Cambiar una consulta para interferir con el funcionamiento previsto.
* **Ataques de UNIÓN**: Recuperar datos de diferentes tablas de la base de datos.
* **Inyección SQL ciega**: Cuando los resultados de la consulta no se devuelven en las respuestas de la aplicación.

---

### Recuperación de datos ocultos

Imagina una aplicación de compras que muestra productos por categorías. Al hacer clic en "Regalos" (`Gifts`), el navegador solicita:
`https://insecure-website.com/products?category=Gifts`

La aplicación realiza la siguiente consulta SQL:
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Si la aplicación no tiene defensas, un atacante puede enviar:
https://insecure-website.com/products?category=Gifts'--

Esto resulta en la consulta:
SQL
```
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```
[!IMPORTANT]
El indicador -- es un comentario en SQL. Esto elimina la restricción AND released = 1, mostrando todos los productos, incluso los no lanzados.

Mostrar todos los productos

Para mostrar todos los productos de cualquier categoría (incluso las que no existen):
https://insecure-website.com/products?category=Gifts'+OR+1=1--

Consulta resultante:
SQL
```
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```
Como 1=1 siempre es cierto, la base de datos devuelve todos los elementos de la tabla.

 [!CAUTION]
Advertencia: Tenga cuidado al inyectar la condición OR 1=1. Es común que las aplicaciones usen datos de una sola solicitud en múltiples consultas. Si su condición llega a una sentencia UPDATE o DELETE, puede resultar en una pérdida accidental de datos.
### Subvertir la lógica de aplicación

Imagine una aplicación que permite a los usuarios iniciar sesión con un nombre de usuario y contraseña. Si un usuario envía el nombre de usuario `wiener` y la contraseña `bluecheese`, la aplicación comprueba las credenciales realizando la siguiente consulta SQL:

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```
Si la consulta devuelve los detalles de un usuario, entonces el inicio de sesión es exitoso. De lo contrario, se rechaza.
El ataque

En este caso, un atacante puede iniciar sesión como cualquier usuario sin necesidad de una contraseña. Puede hacerlo usando la secuencia de comentarios SQL -- para eliminar la comprobación de contraseña de la cláusula WHERE.

Por ejemplo, al enviar el nombre de usuario administrator'-- y una contraseña en blanco, se genera la siguiente consulta:
SQL
```
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```
Esta consulta devuelve al usuario cuyo username es administrator y registra correctamente al atacante como ese usuario.

  [!NOTE]
  Este método es una forma común de eludir sistemas de autenticación (Login Bypass) cuando las entradas no están debidamente saneadas.
### Recuperación de datos de otras tablas de base de datos

En los casos en que la aplicación responde con los resultados de una consulta SQL, un atacante puede usar una vulnerabilidad de inyección SQL para recuperar datos de otras tablas dentro de la base de datos. Puede utilizar la palabra clave **`UNION`** para ejecutar una consulta `SELECT` adicional y anexar los resultados a la consulta original.

#### El ataque
Por ejemplo, si una aplicación ejecuta la siguiente consulta que contiene la entrada del usuario `Gifts`:

```sql
SELECT name, description FROM products WHERE category = 'Gifts'
```
Un atacante puede enviar la entrada:
' UNION SELECT username, password FROM users--

Esto hace que la aplicación devuelva todos los nombres de usuario y contraseñas junto con los nombres y descripciones de los productos en la respuesta.

  [!TIP]
  Para que un ataque de unión tenga éxito, las consultas deben devolver el mismo número de columnas y los tipos de datos deben ser compatibles entre ambas sentencias.
### Vulnerabilidades de inyección SQL ciegas (Blind SQLi)

Muchas de las instancias de inyección SQL son vulnerabilidades ciegas. Esto significa que la aplicación **no devuelve los resultados** de la consulta SQL o los detalles de cualquier error de la base de datos dentro de sus respuestas. 

Las vulnerabilidades ciegas aún se pueden explotar para acceder a datos no autorizados, pero las técnicas involucradas son generalmente más complicadas y difíciles de realizar. Se pueden utilizar las siguientes técnicas:

* **Diferencias detectables**: Puede cambiar la lógica de la consulta para desencadenar una diferencia en la respuesta de la aplicación dependiendo de la verdad de una sola condición. Esto podría implicar inyectar una nueva condición booleana o desencadenar condicionalmente un error (como una división por cero).
* **Retrasos de tiempo**: Puede activar condicionalmente un retardo de tiempo en el procesamiento de la consulta. Esto permite inferir la verdad de la condición en función del tiempo que tarda la aplicación en responder.
* **Interacción fuera de banda (OAST)**: Puede activar una interacción de red utilizando técnicas OAST. Esta técnica es extremadamente poderosa y funciona donde otras no lo hacen. A menudo permite exfiltrar datos directamente a través del canal fuera de banda, por ejemplo, colocando los datos en una búsqueda de DNS para un dominio bajo su control.

> [!IMPORTANT]
> Aunque no haya una salida directa de datos en la pantalla, la base de datos sigue procesando la lógica inyectada, lo que permite la extracción de información bit a bit.
### Inyección SQL de segundo orden

* **Inyección SQL de primer orden**: Se produce cuando la aplicación procesa la entrada del usuario a partir de una solicitud HTTP e incorpora dicha entrada en una consulta SQL de manera insegura.

* **Inyección SQL de segundo orden**: Ocurre cuando la aplicación toma la entrada del usuario de una solicitud HTTP y la **almacena para su uso futuro**. Esto generalmente se hace colocando la entrada en una base de datos, pero no se produce ninguna vulnerabilidad en el punto donde se almacenan los datos. 

#### El mecanismo
Más tarde, al manejar una solicitud HTTP diferente, la aplicación recupera los datos almacenados y los incorpora en una consulta SQL de una manera insegura. Por esta razón, la inyección SQL de segundo orden también se conoce como **inyección SQL almacenada**.

> [!NOTE]
> Esto ocurre a menudo cuando los desarrolladores manejan de forma segura la ubicación inicial de los datos (confiando en que ya están "limpios" en la BD), pero los procesan posteriormente de manera insegura porque consideran erróneamente que se trata de datos de confianza.
## Examinar la base de datos

Algunas características centrales del lenguaje SQL se implementan de la misma manera a través de plataformas populares, y muchas formas de detectar y explotar vulnerabilidades funcionan de forma idéntica.

Sin embargo, existen muchas diferencias entre las bases de datos comunes, lo que significa que algunas técnicas funcionan de manera diferente según la plataforma. Por ejemplo:

* Sintaxis para la concatenación de cadenas.
* Comentarios.
* Consultas por lotes (o apiladas).
* API específicas de la plataforma.
* Mensajes de error.

### Obtener información de la base de datos

Después de identificar una vulnerabilidad, es útil obtener información sobre la base de datos para facilitar su explotación.

#### 1. Consultar la versión
Diferentes métodos funcionan para diferentes tipos de bases de datos. Si un método particular funciona, puedes inferir el tipo de base de datos. 
* **Ejemplo en Oracle:**
  ```sql
  SELECT * FROM v$version
  ```
2. Enumerar tablas y columnas

Puedes identificar qué tablas existen y qué columnas contienen. En la mayoría de las bases de datos, puedes ejecutar la siguiente consulta para enumerar las tablas:
SQL
```
SELECT * FROM information_schema.tables
```
    [!TIP]
    Consultar la documentación específica o una "Hoja de trucos de inyección SQL" es fundamental para adaptar los ataques a cada motor de base de datos (MySQL, PostgreSQL, MSSQL, etc.).
## Inyección SQL en diferentes contextos

No solo se puede inyectar a través de la cadena de consulta (URL). Es posible realizar ataques utilizando cualquier entrada controlable que la aplicación procese como una consulta SQL, como entradas en formato **JSON** o **XML**.

Estos formatos ofrecen formas de ofuscar ataques que podrían estar bloqueados por WAFs (Web Application Firewalls) u otros mecanismos de defensa.

### Evasión de filtros
Las implementaciones débiles suelen buscar palabras clave comunes de SQL. Es posible eludir estos filtros codificando o escapando caracteres en las palabras clave prohibidas. Por ejemplo, una inyección basada en XML puede usar una secuencia de escape para la letra `S` en `SELECT`:

```xml
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```
    [!IMPORTANT]
    El servidor decodificará esta secuencia (transformando &#x53; en S) antes de pasar la consulta al intérprete SQL, permitiendo que el ataque se ejecute a pesar de los filtros iniciales.
Aquí tienes la sección final sobre Prevención, diseñada para cerrar tu documento de GitHub con ejemplos de código claros y alertas de seguridad:
Markdown

## Cómo prevenir la inyección SQL

La forma más efectiva de evitar la mayoría de las instancias de inyección SQL es utilizando **consultas parametrizadas** (también conocidas como "declaraciones preparadas") en lugar de la concatenación de cadenas.

### Ejemplo de código vulnerable
El siguiente código permite la inyección porque concatena la entrada directamente:
```java
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```
### Ejemplo de código seguro

Se puede reescribir para impedir que la entrada del usuario interfiera con la estructura de la consulta:
Java
```
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```
### Limitaciones y Casos Especiales

Las consultas parametrizadas sirven para datos en las cláusulas WHERE, INSERT o UPDATE. Sin embargo, no se pueden usar para nombres de tablas, columnas o la cláusula ORDER BY. En esos casos, se debe:

    Utilizar Listas Blancas: Validar la entrada contra una lista de valores permitidos.

    Lógica Alternativa: Implementar una lógica diferente para entregar el comportamiento requerido sin inyectar la entrada directamente.

    [!CAUTION]
    Regla de Oro: Para que una consulta parametrizada sea efectiva, la cadena de la consulta debe ser siempre una constante codificada. Nunca debe contener datos variables. No confíes en el origen de los datos ni decidas caso por caso; usa siempre parametrización para evitar errores humanos.
