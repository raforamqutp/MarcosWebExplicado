## **LOCALSTORAGE EN JAVASCRIPT PURO**

```js
// GUARDAR datos
localStorage.setItem("clave", JSON.stringify(valor));

// OBTENER y PARSEAR datos
const datos = JSON.parse(localStorage.getItem("clave"));

// ACTUALIZAR datos (ejemplo con array)
let arr = JSON.parse(localStorage.getItem("clave")) || [];
arr.push(nuevoValor);
localStorage.setItem("clave", JSON.stringify(arr));

// ELIMINAR una clave
localStorage.removeItem("clave");

// LIMPIAR todo el localStorage
localStorage.clear();
```

---

## **LOCALSTORAGE EN REACT (con `useEffect`)**

```jsx
import { useEffect, useState } from "react";

// Estado base
const [datos, setDatos] = useState([]);

// CARGAR datos al montar el componente
useEffect(() => {
  const stored = localStorage.getItem("clave");
  if (stored) {
    setDatos(JSON.parse(stored));
  }
}, []);

// GUARDAR o ACTUALIZAR localStorage (cada vez que `datos` cambie)
useEffect(() => {
  localStorage.setItem("clave", JSON.stringify(datos));
}, [datos]);

// EJEMPLO: añadir un nuevo valor
const agregarDato = nuevo => {
  setDatos(prev => [...prev, nuevo]);
};

// BORRAR todos los datos
const borrarTodo = () => {
  localStorage.removeItem("clave");
  setDatos([]);
};
```

---

## ✅ **PATRÓN REUTILIZABLE DE LOCALSTORAGE EN REACT**

Puedes hacer un **custom hook** para reutilizar:

```jsx
// useLocalStorage.js
import { useEffect, useState } from "react";

export function useLocalStorage(key, defaultValue = []) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : defaultValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

```jsx
// En tu componente
const [favoritos, setFavoritos] = useLocalStorage("pokemonFavorites");

const añadirFavorito = (nuevo) => {
  setFavoritos(prev => [...prev, nuevo]);
};
```

---

## 🧾 CONCLUSIÓN

| Acción     | JS Puro                                 | React con `useEffect`                        |
| ---------- | --------------------------------------- | -------------------------------------------- |
| Guardar    | `setItem(clave, JSON.stringify(valor))` | `useEffect(() => { setItem(...) }, [value])` |
| Leer       | `JSON.parse(getItem(clave))`            | `useEffect(() => { setState(...) }, [])`     |
| Actualizar | leer → modificar → `setItem()`          | actualizar `state` → se guarda solo          |
| Eliminar   | `removeItem(clave)`                     | llamar `removeItem(clave)` en handler        |

¿Deseas que te genere esta cheat sheet como un PDF o imagen lista para imprimir o compartir?
