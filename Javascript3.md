### Plagio JavaScript Avanzado (v2)

---

#### **1. CRUD en JavaScript Puro (DOM)**
```javascript
// SELECTORES
const inputs = document.querySelectorAll('input');
const form = document.querySelector('form');

// OBJETO MODELO
const modelo = {
  id: generarId(),
  campo1: '',
  campo2: ''
};

// CLASE ADMINISTRADORA
class CRUD {
  constructor() { this.datos = []; }
  
  agregar(item) {
    this.datos = [...this.datos, item];
    this.mostrar();
  }
  
  editar(item) {
    this.datos = this.datos.map(d => d.id === item.id ? item : d);
  }
  
  eliminar(id) {
    this.datos = this.datos.filter(d => d.id !== id);
  }
  
  mostrar() {
    contenedor.innerHTML = '';
    this.datos.forEach(item => {
      contenedor.innerHTML += `
        <div>${item.campo1}</div>
        <button onclick="editar(${item.id})">Editar</button>
        <button onclick="eliminar(${item.id})">Eliminar</button>
      `;
    });
  }
}

// EVENTOS
inputs.forEach(input => {
  input.addEventListener('input', (e) => {
    modelo[e.target.name] = e.target.value;
  });
});

form.addEventListener('submit', (e) => {
  e.preventDefault();
  if (validar()) {
    editando ? crud.editar({...modelo}) : crud.agregar({...modelo});
    resetForm();
  }
});

// FUNCIONES AUX
function generarId() {
  return Math.random().toString(36).slice(2) + Date.now();
}

function validar() {
  return Object.values(modelo).every(v => v.trim());
}
```

---

#### **2. Consumo de APIs en React**
```javascript
import React, { useState } from 'react';

function ComponenteAPI() {
  // ESTADOS
  const [input, setInput] = useState('');
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  // FETCH
  const fetchData = async () => {
    if (!input) {
      setError('Campo requerido');
      return;
    }
    
    setLoading(true);
    try {
      const res = await fetch(`https://api.com/${input}`);
      const json = await res.json();
      setData(json);
    } catch (err) {
      setError('Error en API');
    } finally {
      setLoading(false);
    }
  };

  // JSX
  return (
    <div>
      <input 
        value={input} 
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && fetchData()}
      />
      
      <button onClick={fetchData} disabled={loading}>
        {loading ? 'Cargando...' : 'Buscar'}
      </button>
      
      {data && <div>{/* Render data */}</div>}
      {error && <p>{error}</p>}
    </div>
  );
}
```

---

#### **3. localStorage + Juegos (React)**
```javascript
import { useState, useEffect } from 'react';

function JuegoMemoria() {
  // ESTADOS
  const [items, setItems] = useState([]);
  const [cartas, setCartas] = useState([]);
  const [volteadas, setVolteadas] = useState([]);
  const [pares, setPares] = useState([]);

  // CARGA INICIAL
  useEffect(() => {
    const saved = localStorage.getItem('key');
    if (saved) {
      const parsed = JSON.parse(saved);
      setItems(parsed);
      setCartas(generarCartas(parsed));
    }
  }, []);

  // GENERAR CARTAS
  const generarCartas = (items) => {
    return [...items, ...items]
      .sort(() => Math.random() - 0.5)
      .map((item, i) => ({ ...item, id: i, flipped: false, matched: false }));
  };

  // MANEJAR CLICK
  const handleClick = (carta) => {
    const nuevasCartas = cartas.map(c => 
      c.id === carta.id ? { ...c, flipped: true } : c
    );
    setCartas(nuevasCartas);
    
    if (volteadas.length === 1) {
      if (volteadas[0].nombre === carta.nombre) {
        setCartas(cartas.map(c => 
          c.nombre === carta.nombre ? { ...c, matched: true } : c
        ));
      } else {
        setTimeout(() => {
          setCartas(cartas.map(c => 
            [volteadas[0].id, carta.id].includes(c.id) ? 
            { ...c, flipped: false } : c
          ));
        }, 1000);
      }
      setVolteadas([]);
    } else {
      setVolteadas([carta]);
    }
  };
}
```

---

#### **4. Formularios React (Estructura Universal)**
```javascript
import React, { useState } from 'react';

function Formulario() {
  // 1 ESTADO POR CAMPO
  const [campo1, setCampo1] = useState('');
  const [campo2, setCampo2] = useState('');

  // MANEJADOR SUBMIT
  const handleSubmit = (e) => {
    e.preventDefault();
    // Procesar datos (campo1, campo2)
  };

  // JSX
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={campo1}
        onChange={(e) => setCampo1(e.target.value)}
        required
      />
      <input
        value={campo2}
        onChange={(e) => setCampo2(e.target.value)}
        required
      />
      <button type="submit">Enviar</button>
    </form>
  );
}
```

---

### Reglas de Oro (Memorizar)
1. **CRUD JavaScript**:
   - Clonar objetos: `{...objeto}`
   - Actualizar arrays: 
     - Agregar: `[...array, nuevo]`
     - Editar: `.map(item => cond ? nuevo : item)`
     - Eliminar: `.filter(item => item.id !== id)`

2. **React APIs**:
   - `async/await` + `try/catch` en fetch
   - 4 estados clave: `[dato, setDato]`, `[cargando, setCargando]`, `[error, setError]`, `[datos, setDatos]`
   - Render condicional: `{dato && <Componente/>}`

3. **React localStorage**:
   - `useEffect` vacío para carga inicial
   - `localStorage.getItem()` + `JSON.parse()`
   - Generar pares: `[...items, ...items].sort(() => Math.random() - 0.5)`

4. **Formularios**:
   - `e.preventDefault()` en submit
   - Por cada campo: `value={estado}` + `onChange={(e) => setEstado(e.target.value)}`
   - Validación básica: `Object.values(modelo).every(v => v.trim())`

---

### Flujos Universales
| Caso          | Pasos Clave                                                                 |
|---------------|-----------------------------------------------------------------------------|
| **CRUD**      | Selectores → Objeto modelo → Clase CRUD → Eventos → Validación → Render     |
| **APIs**      | Estados → Fetch (try/catch) → Actualizar estado → Render condicional       |
| **localStorage** | Cargar datos → Generar cartas → Manejar clicks → Comparar → Actualizar estado |
| **Formularios**  | Estados por campo → Manejador submit (e.preventDefault) → Procesar datos   |

¡Esta hoja cubre el 90% de los ejercicios típicos de JavaScript/React! Basta con reemplazar los nombres de variables y adaptar a cada caso específico.
