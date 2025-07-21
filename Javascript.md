### Resumen del Código (Núcleo Funcional)
**Objetivo principal**:  
Sistema CRUD para administrar citas médicas con:
- Creación/Edición de citas
- Validación de datos
- Notificaciones
- Persistencia en memoria

### Estructuras Clave para Memorizar

#### 1. **Selectores DOM (Siempre igual)**
```javascript
const inputPaciente = document.querySelector('#paciente');
// Repetir para otros campos
const form = document.querySelector('#formulario');
```

#### 2. **Objeto de Datos (Modelo)**
```javascript
const citaObj = {
  id: generarId(),
  paciente: '',
  // ...otros campos
};
```

#### 3. **Clase Administradora (Core)**
```javascript
class AdminCitas {
  constructor() { this.citas = []; }
  
  agregar(cita) {
    this.citas = [...this.citas, cita];
    this.mostrar();
  }
  
  editar(citaActualizada) {
    this.citas = this.citas.map(c => 
      c.id === citaActualizada.id ? citaActualizada : c
    );
  }
  
  eliminar(id) {
    this.citas = this.citas.filter(c => c.id !== id);
  }
  
  mostrar() {
    // Limpiar contenedor
    contenedorCitas.innerHTML = '';
    
    // Generar HTML para cada cita
    this.citas.forEach(cita => {
      const div = document.createElement('div');
      div.innerHTML = `
        <p>Paciente: ${cita.paciente}</p>
        <!-- otros campos -->
        <button onclick="editar(${cita.id})">Editar</button>
        <button onclick="eliminar(${cita.id})">Eliminar</button>
      `;
      contenedorCitas.appendChild(div);
    });
  }
}
```

#### 4. **Manejadores de Eventos**
```javascript
// Actualizar objeto al escribir
inputPaciente.addEventListener('input', (e) => {
  citaObj.paciente = e.target.value;
});

// Enviar formulario
form.addEventListener('submit', (e) => {
  e.preventDefault();
  
  if (validarCampos()) {
    if (editando) {
      citas.editar({ ...citaObj });
    } else {
      citas.agregar({ ...citaObj });
    }
    resetForm();
  }
});
```

#### 5. **Funciones Esenciales**
```javascript
// Generar ID único
function generarId() {
  return Math.random().toString(36).slice(2) + Date.now();
}

// Validación básica
function validarCampos() {
  return Object.values(citaObj).every(val => val.trim() !== '');
}

// Resetear formulario
function resetForm() {
  form.reset();
  Object.keys(citaObj).forEach(key => {
    if (key !== 'id') citaObj[key] = '';
  });
}
```

#### 6. **Edición (Patrón clave)**
```javascript
function cargarEdicion(cita) {
  // Llenar formulario con datos existentes
  inputPaciente.value = cita.paciente;
  // ...otros campos
  
  // Copiar al objeto principal
  Object.assign(citaObj, cita);
  
  // Cambiar a modo edición
  editando = true;
}
```

---
---


### Flujo Completo para Memorizar
1. **Inicialización**:
   - Crear instancia de `AdminCitas`
   - Configurar listeners de inputs y formulario

2. **Captura de datos**:
   - Cada input actualiza `citaObj`

3. **Envío**:
   - Validar campos
   - Clonar objeto (`{...citaObj}`)
   - Agregar o Editar según bandera

4. **Renderizado**:
   - `mostrar()` genera HTML dinámico
   - Botones llaman a `editar()`/`eliminar()`

5. **Edición**:
   - Copiar cita existente al objeto principal
   - Cambiar texto del botón a "Guardar Cambios"

### Sintaxis Crítica (Reglas de Oro)
1. **Clonar objetos**: `{ ...objeto }`
2. **Actualizar arrays**:
   - Agregar: `[...array, nuevo]`
   - Editar: `.map(item => cond ? nuevo : item)`
   - Eliminar: `.filter(item => item.id !== id)`
3. **Eventos**:
   - `elemento.addEventListener('evento', callback)`
   - `e.preventDefault()` en submits
4. **Elementos dinámicos**:
   - `document.createElement('div')`
   - `elemento.innerHTML = ...`
   - `contenedor.appendChild(elemento)`



---
---


### Estructuras Clave para Memorizar

#### 1. **Estados con useState (Siempre igual)**
```javascript
const [ciudad, setCiudad] = useState('');
const [pais, setPais] = useState('');
const [clima, setClima] = useState(null);
const [cargando, setCargando] = useState(false);
const [error, setError] = useState('');
```

#### 2. **Función para consumir API (Patrón universal)**
```javascript
const obtenerClima = async () => {
  // Validación básica
  if (!ciudad) {
    setError('Ingresa ciudad');
    return;
  }

  setCargando(true);
  setError('');
  
  try {
    const url = `https://api.openweathermap.org/...?q=${ciudad},${pais}&appid=API_KEY`;
    const respuesta = await fetch(url);
    const datos = await respuesta.json();
    
    setClima({
      ciudad: datos.name,
      temperatura: datos.main.temp,
      // ...otros datos
    });
  } catch (err) {
    setError('Error al obtener datos');
  } finally {
    setCargando(false);
  }
};
```

#### 3. **JSX para Inputs y Botón (Estructura básica)**
```jsx
return (
  <div>
    <input 
      value={ciudad}
      onChange={(e) => setCiudad(e.target.value)}
      placeholder="Ciudad"
    />
    
    <input 
      value={pais}
      onChange={(e) => setPais(e.target.value)}
      placeholder="País (opcional)"
    />
    
    <button onClick={obtenerClima} disabled={cargando}>
      {cargando ? 'Cargando...' : 'Buscar Clima'}
    </button>
  </div>
);
```

#### 4. **Mostrar Resultados (Renderizado condicional)**
```jsx
{clima && (
  <div>
    <h2>Clima en {clima.ciudad}</h2>
    <p>Temperatura: {Math.round(clima.temperatura)}°C</p>
    <p>{clima.descripcion}</p>
    <img 
      src={`https://openweathermap.org/img/wn/${clima.icono}.png`} 
      alt="Icono clima"
    />
  </div>
)}

{error && <p className="error">{error}</p>}
```

#### 5. **Manejo de Eventos Clave**
```jsx
// Enter en inputs
onKeyPress={(e) => e.key === 'Enter' && obtenerClima()}
```

### Flujo Completo para Memorizar
1. **Captura de datos**:
   - Usuario ingresa ciudad (obligatorio) y país (opcional)
   - Inputs actualizan estados `ciudad` y `pais`

2. **Solicitud API**:
   - Botón llama a `obtenerClima()`
   - Validación básica de entrada
   - Estados de carga y error se actualizan

3. **Consumo API**:
   - Construir URL con parámetros dinámicos
   - `fetch()` para obtener datos
   - Transformar respuesta con `.json()`

4. **Actualización de estado**:
   - Éxito: `setClima()` con datos procesados
   - Error: `setError()` con mensaje

5. **Renderizado**:
   - Condicional para mostrar datos de clima o errores
   - Botón muestra estado de carga


### Sintaxis Crítica (Reglas de Oro)
1. **Hooks de estado**:
   ```javascript
   const [valor, setValor] = useState(inicial);
   ```

2. **Event handlers**:
   ```jsx
   onChange={(e) => setEstado(e.target.value)}
   onClick={funcion}
   ```

3. **Async/await**:
   ```javascript
   async function() {
     const res = await fetch(url);
     const data = await res.json();
   }
   ```

4. **Renderizado condicional**:
   ```jsx
   {estado && <Componente />}
   {condicion ? <Verdadero /> : <Falso />}
   ```

5. **Deshabilitar botón durante carga**:
   ```jsx
   <button disabled={cargando}>...</button>
   ```


### Esqueleto Mental para Examen
```javascript
import React, { useState } from 'react';

function Componente() {
  // 1. Estados
  const [input1, setInput1] = useState('');
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  // 2. Función API
  const fetchData = async () => {
    if (!input1) {
      setError('Campo requerido');
      return;
    }

    setLoading(true);
    
    try {
      const url = `https://api.com/${input1}`;
      const res = await fetch(url);
      const json = await res.json();
      setData(json);
    } catch (err) {
      setError('Error API');
    } finally {
      setLoading(false);
    }
  };

  // 3. JSX
  return (
    <div>
      <input 
        value={input1}
        onChange={(e) => setInput1(e.target.value)}
      />
      
      <button onClick={fetchData} disabled={loading}>
        {loading ? 'Cargando...' : 'Buscar'}
      </button>
      
      {data && <MostrarData data={data} />}
      {error && <p>{error}</p>}
    </div>
  );
}
```


---
---



### Puntos Clave para Memorizar

#### 1. **Carga de datos desde localStorage**
```javascript
useEffect(() => {
  // Obtener datos de localStorage
  const storedPokemons = localStorage.getItem("selectedPokemons") || 
                         localStorage.getItem("pokemonFavorites");
  
  if (storedPokemons) {
    const parsedPokemons = JSON.parse(storedPokemons);
    setFavoritePokemons(parsedPokemons);
    setCards(generateCards(parsedPokemons));
  } else {
    setError("No tienes Pokémon favoritos");
  }
}, []);
```

#### 2. **Generación de cartas**
```javascript
const generateCards = (pokemons) => {
  // Duplicar y mezclar cartas
  return [...pokemons, ...pokemons]
    .sort(() => Math.random() - 0.5)
    .map((pkmn, i) => ({
      ...pkmn,
      id: i,
      flipped: false,
      matched: false
    }));
};
```

#### 3. **Lógica del juego**
```javascript
const handleCardClick = (card) => {
  // Solo procesar si se pueden voltear cartas
  if (/* condiciones */) return;
  
  // Voltear carta
  const updatedCards = cards.map(c => 
    c.id === card.id ? {...c, flipped: true} : c
  );
  
  setCards(updatedCards);
  
  // Verificar si hay par
  if (flippedCards.length === 1) {
    const [firstCard] = flippedCards;
    
    if (firstCard.name === card.name) {
      // Par encontrado: marcar como matched
      setCards(cards.map(c => 
        c.name === card.name ? {...c, matched: true} : c
      ));
      setMatchedCards([...matchedCards, firstCard, card]);
      setScore(score + 2);
    } else {
      // No es par: voltear de nuevo después de 1s
      setTimeout(() => {
        setCards(cards.map(c => 
          (c.id === firstCard.id || c.id === card.id) ? 
          {...c, flipped: false} : c
        ));
      }, 1000);
    }
    setFlippedCards([]);
  } else {
    setFlippedCards([card]);
  }
};
```

#### 4. **Reinicio del juego**
```javascript
const resetGame = () => {
  // Regenerar cartas con los mismos favoritos
  setCards(generateCards(favoritePokemons));
  setFlippedCards([]);
  setMatchedCards([]);
  setScore(0);
  setTimeLeft(300);
};
```


### Flujo Completo para Memorizar
1. **Inicialización**:
   - Cargar Pokémon favoritos desde localStorage
   - Generar pares de cartas mezcladas

2. **Juego**:
   - Click en carta: voltear y guardar en estado
   - Segundo click: comparar nombres
     - Coincidencia: marcar como "matched"
     - Error: voltear después de 1s
   - Contar puntos y pares encontrados

3. **Finalización**:
   - Tiempo agotado o todos los pares encontrados
   - Opción de reiniciar con mismos favoritos


### Sintaxis Crítica (LocalStorage + React)
1. **Leer de localStorage**:
   ```javascript
   const data = localStorage.getItem("key");
   if (data) {
     const parsed = JSON.parse(data);
     setState(parsed);
   }
   ```

2. **Guardar en localStorage**:
   ```javascript
   // (Normalmente en otro componente)
   localStorage.setItem("key", JSON.stringify(data));
   ```

3. **Generar pares de cartas**:
   ```javascript
   [...items, ...items].sort(() => Math.random() - 0.5)
   ```

4. **Manejo de estado de cartas**:
   ```javascript
   // Voltear
   setCards(cards.map(c => c.id === id ? {...c, flipped: true} : c));
   
   // Emparejar
   setCards(cards.map(c => c.name === name ? {...c, matched: true} : c));
   ```

5. **Temporizador con useRef**:
   ```javascript
   const timerRef = useRef(null);
   
   // Iniciar
   timerRef.current = setInterval(() => {}, 1000);
   
   // Detener
   clearInterval(timerRef.current);
   ```


### Esqueleto Mental para Juegos con localStorage
```javascript
import { useState, useEffect } from 'react';

function MemoryGame() {
  const [items, setItems] = useState([]);
  const [gameData, setGameData] = useState([]);

  // 1. Cargar datos iniciales
  useEffect(() => {
    const savedItems = localStorage.getItem("userFavorites");
    if (savedItems) {
      setItems(JSON.parse(savedItems));
      setGameData(generateGameData(JSON.parse(savedItems)));
    }
  }, []);

  // 2. Generar datos del juego
  const generateGameData = (items) => {
    return [...items, ...items]
      .sort(() => Math.random() - 0.5)
      .map((item, i) => ({ ...item, id: i, flipped: false, matched: false }));
  };

  // 3. Manejar selección
  const handleSelect = (item) => {
    // Lógica de comparación
  };

  // 4. Reiniciar juego
  const resetGame = () => {
    setGameData(generateGameData(items));
  };

  return (
    <div>
      {gameData.map(item => (
        <Card 
          key={item.id} 
          item={item} 
          onClick={() => handleSelect(item)}
        />
      ))}
      <button onClick={resetGame}>Reiniciar</button>
    </div>
  );
}
```


---
---



### Explicación Ultra-Condensada: Formulario React

**Objetivo principal**:  
Formulario básico que captura y maneja datos usando estados de React.


### Núcleo Funcional (Para Memorizar)

#### 1. **Declaración de Estados** (Siempre igual)
```javascript
const [nombre, setNombre] = useState('');
const [email, setEmail] = useState('');
```

#### 2. **Manejador de Envío** (Patrón universal)
```javascript
const manejarEnvio = (e) => {
  e.preventDefault();  // Evita recarga de página
  // Acción con datos (aquí alerta)
  alert(`Nombre: ${nombre}, Email: ${email}`);
};
```

#### 3. **JSX del Formulario** (Estructura mínima)
```jsx
<form onSubmit={manejarEnvio}>
  <input
    value={nombre}
    onChange={(e) => setNombre(e.target.value)}
    required
  />
  
  <input
    value={email}
    onChange={(e) => setEmail(e.target.value)}
    required
  />
  
  <button type="submit">Enviar</button>
</form>
```


### Flujo Completo en 3 Pasos:
1. **Estado inicial**: Variables vacías (`nombre`, `email`)
2. **Captura de datos**: 
   - Cada tecla actualiza el estado (`onChange`)
   - `e.target.value` captura el valor actual
3. **Envío**:
   - `onSubmit` ejecuta `manejarEnvio`
   - `e.preventDefault()` bloquea recarga
   - Acción con datos capturados


### Sintaxis Crítica (Reglas de Oro)
1. **Por cada campo**:
   ```jsx
   <input
     value={estado}
     onChange={(e) => setEstado(e.target.value)}
   />
   ```

2. **Manejador de formulario**:
   ```javascript
   const handler = (e) => {
     e.preventDefault();
     // Usar datos aquí
   };
   ```

3. **Etiqueta formulario**:
   ```jsx
   <form onSubmit={handler}>...</form>
   ```


### Esqueleto Mental para Cualquier Formulario:
```javascript
import React, { useState } from 'react';

function MiFormulario() {
  // 1. Estados por campo
  const [campo1, setCampo1] = useState('');
  const [campo2, setCampo2] = useState('');

  // 2. Manejador de envío
  const handleSubmit = (e) => {
    e.preventDefault();
    // Acción con datos (ej: console.log(campo1, campo2))
  };

  // 3. JSX
  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={campo1} 
        onChange={(e) => setCampo1(e.target.value)}
      />
      <input 
        value={campo2} 
        onChange={(e) => setCampo2(e.target.value)}
      />
      <button type="submit">Enviar</button>
    </form>
  );
}
```

**¡Este patrón funciona para cualquier formulario en React!** Solo necesitas:
1. Un `useState` por campo
2. Un `onChange` que actualice el estado
3. Un `onSubmit` que prevenga el default y procese los datos
