# Cross-Site Scripting (XSS)

El **Cross-Site Scripting (XSS)** es una vulnerabilidad de seguridad web que permite a un atacante inyectar scripts maliciosos (normalmente JavaScript) en las páginas que otros usuarios visualizan.

---

## 🚀 ¿Qué puede hacer un atacante con XSS?
Al ejecutar código en el navegador de la víctima, un atacante puede:
- **Suplantar** al usuario víctima.
- **Realizar acciones** en nombre del usuario.
- **Acceder a datos** privados (cookies, tokens de sesión).
- **Capturar credenciales** de acceso.
- **Inyectar funcionalidades** troyanas en el sitio web.

---

## 📂 Tipos de Ataques XSS

| Tipo | Origen del Script | Funcionamiento |
| :--- | :--- | :--- |
| **Reflejado** | Solicitud HTTP actual | El script se incluye en una URL o parámetro y se "refleja" en la respuesta inmediata. |
| **Almacenado** | Base de datos del sitio | El script se guarda permanentemente en el servidor (ej. comentarios, perfiles). |
| **Basado en DOM** | Código del lado del cliente | La vulnerabilidad está en el JavaScript de la página, no en el servidor. |

---

## 🧪 Prueba de Concepto (PoC)
Tradicionalmente se usa `alert()` para confirmar la vulnerabilidad, pero en navegadores modernos como Chrome (v92+) se recomienda usar `print()` en ciertos contextos de iframes:

```javascript
// Carga útil clásica
<script>alert('XSS')</script>

// Carga útil recomendada para Chrome moderno
<script>print() </script>
```
## 💥 Impacto de las Vulnerabilidades
El impacto real de un ataque XSS depende de la naturaleza de la aplicación y del nivel de acceso del usuario afectado:

- **Datos Sensibles:** En aplicaciones que manejan transacciones bancarias, correos electrónicos o registros médicos, el impacto suele ser **crítico**.
- **Privilegios Elevados:** Si el usuario comprometido tiene acceso de administrador, el atacante puede obtener el **control total** sobre toda la funcionalidad y los datos de la aplicación.
- **Acciones No Autorizadas:** El atacante puede realizar cualquier acción que el usuario víctima esté habilitado a hacer (cambiar contraseñas, realizar compras o borrar información).

---

## 🔍 Cómo encontrar y probar XSS
La gran mayoría de las vulnerabilidades se pueden encontrar de forma fiable con herramientas o de forma manual:

1. **Escaneo Automático:** El escáner de vulnerabilidades de **Burp Suite** es el estándar industrial para detectar XSS de forma rápida.
2. **Pruebas Manuales:** - Enviar una entrada única (como una cadena alfanumérica corta) en cada punto de entrada de la aplicación.
   - Identificar cada ubicación donde esa entrada se devuelve en la respuesta HTTP.
   - Probar cargas útiles (payloads) específicas para determinar si el código JavaScript se ejecuta.
   - Para **DOM XSS**, inspeccionar el código JavaScript del cliente para encontrar fuentes no confiables (ej. `document.cookie`) que fluyan hacia receptores (ej. `setTimeout` o `innerHTML`).

---

## 🛡️ Cómo prevenir los ataques XSS
La prevención eficaz requiere una combinación de las siguientes medidas:

- **Filtrar la entrada a la llegada:** Validar estrictamente la entrada del usuario basándose en lo que se espera (listas blancas).
- **Codificar los datos en la salida:** Transformar los datos controlables por el usuario antes de mostrarlos en el navegador (usando entidades HTML o escapes Unicode) para que no se interpreten como contenido activo.
- **Utilizar encabezados de respuesta apropiados:** Configurar `Content-Type` y `X-Content-Type-Options: nosniff` para asegurar que los navegadores interpreten las respuestas correctamente.
- **Política de Seguridad de Contenido (CSP):** Implementar una CSP como última línea de defensa para reducir la gravedad de cualquier vulnerabilidad residual.

---

## ❓ Preguntas Comunes (FAQs)

| Pregunta | Respuesta |
| :--- | :--- |
| **¿XSS vs CSRF?** | XSS inyecta scripts maliciosos; CSRF induce a la víctima a realizar acciones involuntarias. |
| **¿XSS vs Inyección SQL?** | XSS es del lado del cliente (ataca usuarios); SQLi es del lado del servidor (ataca la base de datos). |
| **¿Cómo prevenir en PHP?** | Usar listas blancas de entrada y `htmlentities()` con `ENT_QUOTES` para la salida HTML. |
| **¿Cómo prevenir en Java?** | Utilizar librerías como **Google Guava** para codificar la salida y validar tipos de datos. |

---

## 🔗 Conceptos Avanzados
- **Inyección de marcado colgante (Dangling markup):** Técnica para capturar datos cuando un XSS completo no es posible debido a filtros.
- **Bypass de CSP:** A veces, las políticas mal configuradas pueden ser eludidas para permitir la explotación.

---
_Resumen técnico generado para documentación de seguridad web._
