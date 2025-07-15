# Explicación JWT

## ¿Qué es JWT (JSON Web Token)?

### ¿Para qué sirve un JWT?

Un **JWT (JSON Web Token)** es un mecanismo seguro para identificar a un usuario sin necesidad de mantener una sesión activa en el servidor. Se emplea principalmente en APIs para autenticación **sin estado** (*stateless*).

Cuando un usuario inicia sesión, el servidor:

1. Verifica las credenciales (usuario y contraseña)
2. Crea un token JWT que contiene su identidad (por ejemplo, el DNI)
3. Envía el token al cliente (navegador, herramienta como Insomnia, app móvil, etc.)
4. En peticiones posteriores, el cliente envía ese token como comprobante de autenticación

> El servidor no necesita recordar la sesión; solo debe verificar si el token recibido es válido.

---

### Estructura de un JWT

Un JWT es una cadena compuesta por tres partes codificadas en Base64, separadas por puntos:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTYifQ.abc123signature
```

Cada segmento tiene una función específica:

1. **Header**: indica el tipo de token y el algoritmo de firma (por ejemplo, HS256)
2. **Payload**: contiene los datos del usuario (como el DNI)
3. **Signature (Firma)**: garantiza la integridad del token

---

### Ventajas del uso de JWT

* No se requiere mantener sesiones en memoria del servidor
* Facilita la escalabilidad (por ejemplo, en entornos con múltiples servidores)
* Validación rápida y eficiente
* Compatible con APIs REST, aplicaciones móviles y SPAs

---

## Uso de JWT en el Proyecto

A continuación, se detalla el flujo completo de seguridad en el proyecto, clase por clase y en el orden en que intervienen.

---

## Paso a Paso: Seguridad con Spring Security y JWT

### 1. `WebSecurityConfig.java`

**Propósito:**
Configura la seguridad general de la aplicación. Define:

* Las rutas que requieren autenticación
* Los roles necesarios para acceder a distintas rutas
* Los filtros de seguridad aplicados
* El mecanismo de validación de contraseñas

**Métodos clave:**

* `securityFilterChain()`: define reglas de acceso y registra filtros
* `authenticationManager()`: usa una clase personalizada para cargar usuarios (`EmpleadoUserDetailsService`)
* `passwordEncoder()`: aplica BCrypt para la validación de contraseñas

---

### 2. `EmpleadoUserDetailsService.java`

**Propósito:**
Carga los usuarios desde la base de datos al momento de autenticarse. Utiliza el DNI como identificador y retorna un objeto `UserDetails`.

**Método clave:**

* `loadUserByUsername(String dni)`: busca un empleado por su DNI y lo adapta al formato que Spring Security puede interpretar.

---

### 3. `EmpleadoUserDetails.java`

**Propósito:**
Adapta la entidad `Empleado` a una implementación de `UserDetails` que Spring Security puede utilizar.

**Métodos clave:**

* `getAuthorities()`: convierte el rol del empleado (por ejemplo, ADMINISTRATIVO) al formato `ROLE_ADMINISTRATIVO`
* `isAccountNonLocked()`: bloquea la cuenta si el estado del usuario no es ACTIVO
* `isEnabled()`: desactiva la cuenta si el estado es INACTIVO

---

### 4. `AuthController.java`

**Propósito:**
Punto de entrada para autenticación vía JWT. Expone el endpoint `/auth` que recibe credenciales.

**Flujo:**

* Recibe un JSON con `username` (DNI) y `password`
* Valida las credenciales mediante `AuthenticationManager`
* Si son válidas, llama a `JwtUtil.generateToken()` y retorna el token generado

---

### 5. `JwtUtil.java`

**Propósito:**
Gestiona la creación y validación de tokens JWT.

**Métodos clave:**

* `generateToken(username)`: genera un token válido por un período determinado (por ejemplo, 1 hora)
* `extractUsername(token)`: extrae el DNI desde el token
* `validateToken(token, userDetails)`: valida el token con respecto a los datos del usuario autenticado

---

### 6. `JwtFilter.java`

**Propósito:**
Filtro que intercepta cada petición HTTP. Si encuentra un encabezado `Authorization: Bearer <token>`, realiza:

* Extracción del token
* Verificación de la firma
* Extracción del DNI
* Carga del usuario correspondiente con `EmpleadoUserDetailsService`
* Creación de un objeto de autenticación y su inclusión en el `SecurityContext`

**Resultado:**
Spring trata la petición como autenticada si el token es válido.

---

### 7. Seguridad Aplicada

En `WebSecurityConfig` se definen reglas de autorización, por ejemplo:

```java
.requestMatchers("/admin/**").hasRole("ADMINISTRATIVO")
```

Esto significa que, para acceder a rutas como `/admin/usuarios`, el usuario autenticado debe tener el rol adecuado. En caso contrario, se devuelve un error 403 o se redirige a la página de login.

---

## Ejemplo Práctico con Insomnia

1. Se envía una petición `POST /auth` con un JSON como este:

```json
{
  "username": "12345678",
  "password": "123tamarindo"
}
```

2. Si las credenciales son correctas, el servidor responde con un token JWT.

3. Para acceder a rutas protegidas (por ejemplo, `/admin/reportes`), se envía una petición `GET` con el encabezado:

```
Authorization: Bearer <el-token>
```

4. El `JwtFilter` intercepta la petición, valida el token y, si es correcto, la autoriza.

---

## Clases Clave en el Proyecto

| Clase                        | Descripción                                   |
| ---------------------------- | --------------------------------------------- |
| `WebSecurityConfig`          | Configura reglas y filtros de seguridad       |
| `EmpleadoUserDetailsService` | Carga usuarios desde la base de datos por DNI |
| `EmpleadoUserDetails`        | Adapta la entidad `Empleado` a `UserDetails`  |
| `AuthController`             | Endpoint de autenticación con JWT             |
| `JwtUtil`                    | Generación y validación de tokens JWT         |
| `JwtFilter`                  | Filtro que valida tokens en cada petición     |
| `PasswordGenerator`          | Utilidad para generar contraseñas encriptadas |
