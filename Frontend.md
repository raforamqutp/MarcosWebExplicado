### Frontend

#### **1. Estructura General del Proyecto**
El proyecto sigue una arquitectura MVC (Modelo-Vista-Controlador) con Spring Boot y Thymeleaf para el frontend. La estructura de paquetes se organiza en:
- **Entidades**: Clases POJO que representan tablas de la base de datos.
- **Repository**: Interfaces Spring Data JPA para operaciones CRUD.
- **Controllers**: Manejan peticiones HTTP y coordinan lógica entre vistas y modelos.
- **Recursos**: Archivos estáticos (CSS, JS, imágenes) y plantillas Thymeleaf.

---

### **2. Explicación del Código Frontend (HTML)**

#### **A. Patrones Comunes en Todas las Vistas**
1. **Navbar Superior**:
   - **Tecnologías**: Bootstrap 5 + Font Awesome.
   - **Funcionalidad**:
     - Menú responsive con `data-bs-toggle="collapse"`.
     - Buscador con formulario de búsqueda.
     - Dropdown de notificaciones dinámico con badges (`bg-danger` para contador).
     - Perfil de usuario con opciones de gestión.
   - **Internacionalización**: Usa `th:text="#{key}"` para textos dinámicos desde `messages.properties`.

2. **Sidebar**:
   - **Componentes Clave**:
     - Logo del sistema con imagen dinámica (`th:src`).
     - Menú lateral con íconos Font Awesome y estados activos (`active`).
     - Agrupación lógica de secciones con separadores (`<hr>`).
   - **Navegación**: Enlaces con `th:href="@{/ruta}"` para integración con controladores.

3. **Sistema de Internacionalización**:
   - Claves como `#{sidebar.dashboard}` cargan textos desde:
     ```properties
     # messages.properties
     sidebar.dashboard=Panel de Control
     # messages_en.properties
     sidebar.dashboard=Dashboard
     ```

---

#### **B. Vista Específica: `Inventario.html`**
**Sección de Agregar Producto**:
```html
<form th:action="@{/Inventario/agregar}" method="post">
  <input name="nombre" type="text" class="form-control" required>
  <input name="cantidad" type="number" min="0" required>
  <input name="precio" type="number" step="0.01" required>
</form>
```
- **Binding con Backend**: Los atributos `name` mapean directamente al objeto `Producto` en Spring MVC.
- **Validaciones**: 
  - `required`: Campos obligatorios.
  - `min="0"`: Evita valores negativos.
  - `step="0.01"`: Permite decimales en precios.

**Tabla de Productos**:
```html
<tr th:each="prod, iterStat : ${productos}">
  <td th:text="${iterStat.count}">1</td>
  <td th:text="${prod.nombre}">Proteína</td>
  <td th:text="${prod.cantidad}">10</td>
  <td th:text="${'S/ ' + #numbers.formatDecimal(prod.precio, 1, 'COMMA', 2, 'POINT')}">S/ 50.00</td>
  <td>
    <span th:if="${prod.cantidad <= 5}" class="badge bg-danger">Stock Crítico</span>
  </td>
</tr>
```
- **Thymeleaf Features**:
  - `th:each`: Itera sobre la lista `productos` enviada por el controlador.
  - `#numbers.formatDecimal`: Formatea precios con 2 decimales y separadores.
  - `th:if`: Muestra badges condicionales según niveles de stock.

**Modal de Edición**:
```javascript
detalleModal.addEventListener('show.bs.modal', (event) => {
  const button = event.relatedTarget;
  const id = button.getAttribute('data-id');
  // Mapea data-attributes a campos del modal
});
```
- **Funcionamiento**: 
  1. Al hacer clic en "Ver detalles", se capturan los `data-attributes` del producto.
  2. Se pobla el modal con los valores usando JavaScript.
  3. El formulario de edición envía los datos a `/Inventario/editar`.

---

#### **C. Vista: `login.html`**
**Formulario de Login**:
```html
<form th:action="@{/login}" method="post">
  <input type="text" name="username" placeholder="DNI">
  <input type="password" name="password">
</form>
<div th:if="${param.error}" class="alert alert-danger">Credenciales inválidas</div>
```
- **Spring Security Integration**:
  - Los campos `username` y `password` son requeridos por Spring Security.
  - `th:if="${param.error}"`: Muestra error si las credenciales fallan.

---

#### **D. Vista: `Asignar_membresia.html`**
**Dropdowns Dinámicos**:
```html
<select th:field="*{cliente}" required>
  <option th:each="cliente : ${clientes}" 
          th:value="${cliente.id}" 
          th:text="${cliente.nombreCompleto}">
  </option>
</select>
```
- **Binding con Objetos**:
  - `th:field="*{cliente}"`: Enlaza al atributo `cliente` del objeto `Membresia`.
  - Los options se generan dinámicamente desde la lista `clientes` del modelo.

**Mensajes de Confirmación**:
```html
<div th:if="${mensaje}">
  <p th:text="${mensaje}"></p>
  <p>Cliente: <span th:text="${clienteAsignado}"></span></p>
</div>
```
- **Feedback al Usuario**: Muestra resultados de la operación usando variables del controlador.

---

#### **E. Vista: `Registro_Cliente.html`**
```html
<form id="formCliente" class="needs-validation"
      th:action="@{/registrarCliente}" 
      method="post" 
      th:object="${cliente}" novalidate>
```
Aquí es donde se hace la vinculación del formulario HTML con el objeto Java cliente que ya viene con datos (por ejemplo, si es un formulario de edición o si estamos precargando algo).

### ¿Cómo se conectan los campos del formulario con los datos?

```html
<input type="text" class="form-control" id="nombre" th:field="*{nombreCompleto}" ...>
```

Este `th:field="*{nombreCompleto}"` le dice a Thymeleaf:

> “Llena este campo con el valor de `cliente.getNombreCompleto()` si existe”.

Y así con todos los campos:

| Campo en HTML | th\:field           | Valor cargado desde                        |
| ------------- | ------------------- | ------------------------------------------ |
| Nombre        | `*{nombreCompleto}` | `cliente.nombreCompleto` (getter del bean) |
| DNI           | `*{dni}`            | `cliente.dni`                              |
| Teléfono      | `*{telefono}`       | `cliente.telefono`                         |
| Sexo          | `*{sexo}`           | `cliente.sexo`                             |
| etc.          | ...                 | ...                                        |

Entonces, cuando el backend (por ejemplo, desde un controlador de Spring) te envía un modelo con:

```java
model.addAttribute("cliente", cliente);
```

Thymeleaf se encarga de:

* **Llenar los inputs** con los valores del objeto `cliente`.
* **Conservar la información** al hacer una edición (por ejemplo, si estás actualizando un cliente).
* **Validar los errores** y mostrarlos si los campos están vacíos o incorrectos.

---

Aunque en HTML no "ves" la conexión a la base de datos, **Thymeleaf interpreta el objeto Java que le manda el servidor**, lo recorre, y lo **vincula con los campos HTML** gracias a:

* `th:object="${cliente}"`
* `th:field="*{...}"`

