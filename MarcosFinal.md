Aquí tienes 6 procesos clave del proyecto con sus clases relevantes y ubicación:

---

### **1. CRUD de Clientes**  
- **Clase**: `ClienteController.java`  
- **Proceso**:  
  - Crear/editar/eliminar clientes con validación de DNI único  
  - **Endpoints**:  
    - `GET /clientes` → Listar  
    - `POST /registrarCliente` → Crear  
    - `POST /clientes/editar/{id}` → Actualizar  

---

### **2. Registro de Empleados con Roles**  
- **Clase**: `EmpleadoController.java`  
- **Proceso**:  
  - Registrar empleados con roles (`ADMINISTRATIVO`, `RECEPCION`, etc.) y estado (`ACTIVO`/`INACTIVO`)  
  - **Validación**:  
    - Fecha de nacimiento con `@DateTimeFormat`  
    - Contraseña hasheada con BCrypt (`PasswordEncoder`)  

---

### **3. Asignación de Membresías a Clientes**  
- **Clase**: `MembresiaController.java`  
- **Proceso**:  
  - Vincular membresías (Diario/Mensual/Anual) a clientes existentes  
  - **Relación**: `@ManyToOne` con `Cliente` en `Membresia.java`  
  - **Endpoint**:  
    - `POST /membresia/asignar`  

---

### **4. Venta de Entradas y Generación de Comprobantes**  
- **Clase**: `EntradaController.java`  
- **Proceso**:  
  - Registrar entrada con DNI de cliente y monto  
  - Generar comprobante TXT (`generarTxtComprobante()`)  
  - **Endpoint**:  
    - `POST /entradas/registrar`  

---

### **5. Autenticación con JWT**  
- **Clases**:  
  - `AuthController.java` (Endpoint `/auth`)  
  - `JwtFilter.java` (Validación de tokens)  
- **Proceso**:  
  - Generar token JWT al validar credenciales  
  - Filtro verifica token en cada petición protegida  
  - Solo pasa el DNI

---

### **6. Reporte Financiero (Sumatoria de Ingresos)**  
- **Clase**: `EntradaRepository.java`  
- **Proceso**:  
  - Consulta JPQL para sumar ingresos diarios:  
    ```java
    @Query("SELECT SUM(e.monto) FROM Entrada e WHERE DATE(e.fecha) = :fecha")
    Double getTotalIngresosPorFecha(LocalDate fecha);
    ```  
  - Usado en `DashboardController.java` para reportes  

---

### Relación entre Procesos  
```mermaid
graph LR
    A[CRUD Clientes] --> B[Asignar Membresías]
    B --> C[Vender Entradas]
    C --> D[Reporte Financiero]
    E[Registro Empleados] --> F[Autenticación JWT]
```


---
---
---




### **1. Acceso a Base de Datos con Entity y JPA Repository**
**Implementación**:  
- **Entidades JPA**:  
  - Clases como `Cliente.java`, `Empleado.java`, y `Membresia.java` están anotadas con `@Entity`, definiendo el mapeo objeto-relacional (ORM) a tablas MySQL.  
  - Ejemplo en `Cliente.java`:
    ```java
    @Entity
    @Table(name = "cliente")
    public class Cliente {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;  // Auto-incremental
        @Column(nullable = false, unique = true) // Restricción de unicidad
        private String dni;
    }
    ```
  - **Relaciones**:  
    - `@ManyToOne` en `Membresia.java` para vincular con `Cliente`.

- **Repositorios**:  
  - Interfaces como `ClienteRepository.java` extienden `JpaRepository`, heredando operaciones CRUD:
    ```java
    public interface ClienteRepository extends JpaRepository<Cliente, Long> {
        // Query method automático
        Cliente findByDni(String dni); 
    }
    ```
  - **Consultas personalizadas**:  
    - JPQL en `EntradaRepository.java`:
      ```java
      @Query("SELECT SUM(e.monto) FROM Entrada e WHERE DATE(e.fecha) = :fecha")
      Double getTotalIngresosPorFecha(@Param("fecha") LocalDate fecha);
      ```

**Dominio demostrado**:  
Uso de anotaciones JPA (`@Entity`, `@Column`) para mapeo preciso de modelos, junto a repositorios que aprovechan Spring Data JPA para reducir código boilerplate. Las queries personalizadas con `@Query` optimizan operaciones complejas.

---

### **2. Mecanismos de Seguridad**
**Implementación**:  
- **Spring Security**:  
  - Configuración central en `WebSecurityConfig.java`:
    ```java
    @Configuration
    @EnableWebSecurity
    public class WebSecurityConfig {
        @Bean
        public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
            http
                .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/admin/**").hasRole("ADMINISTRATIVO") // Autorización por roles
                    .anyRequest().authenticated()
                )
                .formLogin(form -> form.loginPage("/login").permitAll());
        }
    }
    ```
  - **Validación de credenciales**:  
    - `EmpleadoUserDetailsService.java` implementa `UserDetailsService` para cargar usuarios desde la BD.

- **Reglas de negocio**:  
  - En `EmpleadoUserDetails.java`, se controla el acceso con métodos como `isAccountNonLocked()` basado en el campo `estado` de `Empleado`.

**Dominio demostrado**:  
Integración de Spring Security para manejar autenticación/autorización, con personalización de `UserDetailsService` y uso de roles definidos en la entidad `Empleado`. Se aplican políticas de acceso granular (ej: rutas `/admin/**`).

---

### **3. Manejo de Tokens JWT**
**Implementación**:  
- **Generación/Validación**:  
  - `JwtUtil.java` genera tokens firmados con algoritmo HS256:
    ```java
    public String generateToken(String username) {
        return Jwts.builder()
            .setSubject(username) // DNI del empleado
            .signWith(SignatureAlgorithm.HS256, SECRET)
            .compact();
    }
    ```
  - **Filtro JWT**:  
    - `JwtFilter.java` intercepta peticiones para validar tokens:
      ```java
      if (jwtUtil.validateToken(jwt, userDetails)) {
          // Establece autenticación en el contexto
          SecurityContextHolder.getContext().setAuthentication(authToken);
      }
      ```

- **Integración con Spring Security**:  
  - Registro del filtro en `WebSecurityConfig`:
    ```java
    http.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
    ```

**Dominio demostrado**:  
Uso de la librería `jjwt` para tokens JWT, con validación de firma y extracción de claims. El filtro se integra en la cadena de seguridad de Spring para proteger endpoints REST. La ausencia de manejo de expiración (mejora sugerida) muestra comprensión de limitaciones actuales.

---

### **Diagrama de Flujo JWT + Spring Security**
```mermaid
sequenceDiagram
    participant Client
    participant AuthController
    participant JwtUtil
    participant JwtFilter
    participant SecurityContext

    Client->>AuthController: POST /auth (dni, password)
    AuthController->>JwtUtil: generateToken(dni)
    JwtUtil-->>AuthController: JWT
    AuthController-->>Client: 200 + Token

    Client->>App: GET /api/data (Header: Bearer JWT)
    App->>JwtFilter: doFilterInternal()
    JwtFilter->>JwtUtil: extractUsername(jwt)
    JwtFilter->>SecurityContext: setAuthentication()
    App-->>Client: 200 + Data
```


**Áreas de mejora**:  
- Validación de expiración de tokens en `JwtUtil`.  
- Uso de `@PreAuthorize` para control granular en métodos.  
