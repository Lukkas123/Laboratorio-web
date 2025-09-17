## Path Traversal

**Path Traversal** (también conocido como *Directory Traversal*) es una vulnerabilidad web que permite a un atacante acceder al sistema de archivos del servidor fuera de los directorios previstos por la aplicación.  
Mediante la manipulación de rutas, es posible ascender de directorio y acceder a archivos sensibles, como:

- Código fuente y datos de la aplicación  
- Credenciales de sistemas de back-end  
- Archivos críticos del sistema operativo  

En escenarios más graves, el atacante puede leer o incluso modificar archivos arbitrarios, alterando datos o el comportamiento del servidor y, en última instancia, **tomar el control completo del sistema**.

---

## ¿Cómo explotarlo?

Supongamos que tenemos una aplicación que muestra imágenes de artículos en venta y carga cada imagen con el siguiente HTML:

```html
<img src="LoadImage?filename=218.png">
```
La URL LoadImage toma un parámetro filename y devuelve el contenido del archivo especificado.
La imagen se almacena en el sistema de archivos en /var/www/images/. Para devolver la imagen, la aplicación concatena el nombre de archivo (por ejemplo /var/www/images/218.png) y utiliza una API del sistema de archivos para leer su contenido.

Si no existe ninguna validación, un atacante podría solicitar el archivo /etc/passwd del sistema con una petición como:
/var/www/images/../../../etc/passwd
La secuencia ../ es válida y permite ascender de directorio hasta la raíz (/), desde donde es posible acceder a etc/passwd.
En sistemas Windows, el mismo ataque puede realizarse usando ..\ en lugar de ../.

Evasión de defensas
Algunas aplicaciones implementan filtros para impedir el Path Traversal, pero es posible eludirlos de varias maneras:

Ruta absoluta: Referenciar directamente un archivo, por ejemplo:
filename=/etc/passwd

Secuencias de recorrido anidadas: ....// o ....\/, que se reducen a ../ al normalizarse.

Codificación URL: Codificar los caracteres ../, por ejemplo %2e%2e%2f o incluso doble codificación %252e%252e%252f. Otras codificaciones no estándar como ..%c0%af o ..%ef%bc%8f también pueden funcionar.

Prefijo obligatorio: Si se requiere que el archivo comience con la carpeta base, se puede incluirla seguida de secuencias de recorrido:
filename=/var/www/images/../../../etc/passwd

Extensión obligatoria: Si se requiere una extensión, se puede usar un byte nulo para terminar la ruta antes de la extensión:
filename=../../../etc/passwd%00.png

Cómo prevenir el ataque
Valida siempre la ruta canónica del archivo en función de la entrada del usuario.
Ejemplo en Java:

```java
Copiar código
File file = new File(BASE_DIRECTORY, userInput);
if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {
}
```
Este enfoque garantiza que el usuario solo pueda acceder a archivos dentro de los directorios permitidos, evitando el ascenso en el sistema de archivos.

