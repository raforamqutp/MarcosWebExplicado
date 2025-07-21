### Plagio JavaScript Avanzado

#### **1. Manejo de LocalStorage**
```javascript
// GUARDAR DATOS
localStorage.setItem("clave", JSON.stringify(datos));

// LEER DATOS
const datosGuardados = JSON.parse(localStorage.getItem("clave"));

// ELIMINAR
localStorage.removeItem("clave");
```

#### **2. CRUD con Clases (Patrón Administrador)**
```javascript
class AdminRecursos {
  constructor() {
    this.recursos = JSON.parse(localStorage.getItem("recursos")) || [];
  }

  agregar(recurso) {
    this.recursos = [...this.recursos, recurso];
    this.guardar();
  }

  editar(id, nuevosDatos) {
    this.recursos = this.recursos.map(r => 
      r.id === id ? {...r, ...nuevosDatos} : r
    );
    this.guardar();
  }

  eliminar(id) {
    this.recursos = this.recursos.filter(r => r.id !== id);
    this.guardar();
  }

  guardar() {
    localStorage.setItem("recursos", JSON.stringify(this.recursos));
  }
}
```

#### **3. Consumo de APIs (Async/Await)**
```javascript
const fetchData = async () => {
  try {
    const res = await fetch('https://api.com/data');
    const data = await res.json();
    
    if (!res.ok) throw new Error(data.message);
    return data;
    
  } catch (error) {
    console.error("Error API:", error);
    return null;
  }
};
```

#### **4. Juegos de Memoria (Parejas)**
```javascript
// Generar cartas
const generarCartas = (items) => {
  return [...items, ...items]
    .sort(() => Math.random() - 0.5)
    .map((item, i) => ({...item, id: i, flipped: false, matched: false}));
};

// Manejar selección
const handleSelect = (carta) => {
  const nuevasCartas = cartas.map(c => 
    c.id === carta.id ? {...c, flipped: true} : c
  );
  
  setCartas(nuevasCartas);
  
  // Lógica de comparación
  if (cartasVolteadas.length === 1) {
    if (cartasVolteadas[0].name === carta.name) {
      setCartas(cartas.map(c => 
        c.name === carta.name ? {...c, matched: true} : c
      ));
    } else {
      setTimeout(() => {
        setCartas(cartas.map(c => 
          c.id === carta.id || c.id === cartasVolteadas[0].id 
            ? {...c, flipped: false} 
            : c
        ));
      }, 1000);
    }
  }
};
```

#### **5. Formularios React (Controlados)**
```javascript
function Formulario() {
  const [form, setForm] = useState({
    campo1: "",
    campo2: ""
  });

  const handleChange = (e) => {
    setForm({
      ...form,
      [e.target.name]: e.target.value
    });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    // Procesar datos
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        name="campo1"
        value={form.campo1}
        onChange={handleChange}
      />
      <button type="submit">Enviar</button>
    </form>
  );
}
```

#### **6. Renderizado Condicional (JSX)**
```jsx
{/* Mostrar cuando existe dato */}
{datos && <Componente datos={datos} />}

{/* Mostrar según condición */}
{isLoading ? <Spinner /> : <Contenido />}

{/* Lista de elementos */}
{items.map(item => (
  <li key={item.id}>{item.nombre}</li>
))}
```

#### **7. Hooks Esenciales (useState, useEffect)**
```javascript
// Estado simple
const [valor, setValor] = useState(inicial);

// Estado complejo
const [estado, setEstado] = useState({
  campo1: "",
  campo2: 0,
  loading: false
});

// Efecto con limpieza
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
  
  return () => clearInterval(timer); // Limpieza
}, [dependencias]);
```

#### **8. Manipulación de Arrays (Inmutabilidad)**
```javascript
// Agregar
const nuevoArray = [...arrayOriginal, nuevoElemento];

// Eliminar
const filtrado = arrayOriginal.filter(item => item.id !== id);

// Actualizar
const actualizado = arrayOriginal.map(item => 
  item.id === id ? {...item, propiedad: nuevoValor} : item
);

// Ordenar
const ordenado = [...arrayOriginal].sort((a, b) => a.valor - b.valor);
```

### Reglas de Oro para el Examen
1. **LocalStorage**: 
   - Siempre `JSON.stringify()` al guardar
   - Siempre `JSON.parse()` al leer
   - Verificar `null` con `|| []`

2. **Actualización de Estado**:
   - Objetos: `{...objeto, propiedad: valor}`
   - Arrays: `map`/`filter`/spread operator
   - React: Nunca mutar estado directamente

3. **Async/Await**:
   - Siempre `try/catch`
   - `await` solo en funciones `async`
   - Manejar estados de carga/error

4. **Eventos**:
   - Formularios: `e.preventDefault()`
   - Inputs: `e.target.value`

5. **Claves Únicas**:
   - Siempre `key` en listas (`key={item.id}`)
   - Usar IDs en lugar de índices

6. **Renderizado**:
   - Operador ternario para condicionales
   - `&&` para mostrar/ocultar

¡Esta hoja contiene el 95% de los patrones necesarios para resolver cualquier ejercicio de JavaScript avanzado! Solo debes rellenar los detalles específicos de cada problema.
