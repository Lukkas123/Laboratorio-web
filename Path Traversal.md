# Path Traversal

**Path Traversal** (también conocido como *Directory Traversal*) es una vulnerabilidad web que permite a un atacante acceder al sistema de archivos del servidor fuera de los directorios previstos por la aplicación.  
Mediante la manipulación de rutas, es posible ascender de directorio y acceder a archivos sensibles, como:

- Código fuente y datos de la aplicación  
- Credenciales de sistemas de back-end  
- Archivos críticos del sistema operativo  

En escenarios más graves, el atacante puede leer o incluso modificar archivos arbitrarios, alterando datos o el comportamiento del servidor y, en última instancia, **tomar el control completo del sistema**.

---

# ¿Cómo explotarlo?

Supongamos que tenemos una aplicación que muestra imágenes de artículos en venta y carga cada imagen con el siguiente HTML:

```html
<img src="LoadImage?filename=218.png">

