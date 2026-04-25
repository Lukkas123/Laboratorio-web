# Cross-site Request Forgery (CSRF)

El **Cross-site Request Forgery** (falsificación de petición en sitios cruzados) es una vulnerabilidad de seguridad web que permite a un atacante inducir a los usuarios a realizar acciones que no tienen intención de efectuar. 

Este ataque permite eludir parcialmente la **Política de Mismo Origen (SOP)**, la cual está diseñada para evitar que diferentes sitios web interfieran entre sí.

---

##  Impacto de un ataque CSRF
En un ataque exitoso, el atacante causa que la víctima realice una acción involuntaria. Dependiendo de la función afectada, el impacto puede ser:

* **Modificación de cuenta:** Cambios de correo electrónico o contraseñas.
* **Acciones financieras:** Transferencias de fondos no autorizadas.
* **Control total:** Si el usuario tiene privilegios de administrador, el atacante puede comprometer la aplicación completa, accediendo a datos sensibles y funcionalidades administrativas.

---

##  Condiciones para que exista CSRF
Para que una función sea vulnerable a CSRF, deben cumplirse tres condiciones:

1.  **Una acción relevante:** Existe una acción dentro de la aplicación que el atacante tiene interés en forzar (ej. modificar permisos o datos de perfil).
2.  **Gestión de sesiones basada en cookies:** La aplicación se apoya únicamente en cookies de sesión para identificar al usuario. El navegador las envía automáticamente en cada petición.
3.  **Parámetros predecibles:** Las solicitudes no contienen parámetros cuyos valores el atacante no pueda adivinar. No hay tokens secretos o validaciones adicionales (como pedir la contraseña actual).

---

##  Funcionamiento del ataque

### 1. La solicitud vulnerable
Supongamos que un sitio web permite cambiar el correo del usuario mediante esta solicitud:

```http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Cookie: session=ID_DE_SESION_DEL_USUARIO

email=usuario@ejemplo.com
```
2. El exploit del atacante:

El atacante crea una página web maliciosa con un formulario oculto que se auto-ejecuta mediante JavaScript:
   ```html
<html>
    <body>
        <form action="[https://vulnerable-website.com/email/change](https://vulnerable-website.com/email/change)" method="POST">
            <input type="hidden" name="email" value="atracante@evil.com" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```
3. La ejecución:
Cuando la víctima (que tiene una sesión activa en el sitio vulnerable) visita la página del atacante
a) El navegador de la víctima carga el HTML y ejecuta el script.
b) Se envía la petición POST hacia el servidor vulnerable.
c) El navegador adjunta automáticamente la cookie de sesión.
d) El servidor procesa el cambio de correo creyendo que el usuario lo solicitó voluntariamente.
[!NOTE]Si la acción se puede realizar mediante el método GET, el ataque es aún más sencillo, ya que puede ocultarse en una etiqueta de imagen: <img src="https://vulnerable-website.com/email/change?email=atracante@evil.com">

###  Defensas eficaces contra CSRF

| Método | Descripción | Nivel de Seguridad |
| :--- | :--- | :--- |
| **Tokens CSRF** | El servidor genera un valor único, secreto e impredecible que el cliente debe incluir en cada petición de escritura. | **Alto** (Recomendado) |
| **Atributo SameSite** | Configura las cookies (`Lax` o `Strict`) para que el navegador no las envíe en peticiones originadas desde sitios externos. | **Medio-Alto** |
| **Validación de Referer** | El servidor verifica el encabezado HTTP `Referer` para asegurar que la petición se originó en su propio dominio. | **Bajo-Medio** |
| **Verificación adicional** | Exigir la contraseña actual o un código MFA para acciones críticas (como cambiar el correo o password). | **Muy Alto** |
