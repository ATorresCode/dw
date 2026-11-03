# UD03: Almacenamiento en el navegador

## 1. `localStorage` y `sessionStorage`

Web Storage ofrece dos almacenes de pares clave/valor accesibles desde JavaScript:

- `localStorage` conserva los datos entre sesiones y reinicios del navegador.
- `sessionStorage` conserva los datos durante la sesión de una pestaña; sobrevive a recargas, pero normalmente se elimina al cerrar esa pestaña.

Ambos están separados por **origen** (*scheme*, host y puerto). Por ejemplo, `https://example.com` y `http://example.com` tienen almacenes distintos. Los subdominios y puertos distintos también se aíslan.

| Característica | `localStorage` | `sessionStorage` |
| --- | --- | --- |
| Persistencia | Hasta que se borre | Mientras dure la sesión de la pestaña |
| Compartido entre pestañas del mismo origen | Sí | No, normalmente cada pestaña tiene el suyo |
| Enviado automáticamente al servidor | No | No |

Los navegadores imponen límites de espacio y políticas que pueden variar. El acceso al almacenamiento también puede fallar, por ejemplo, si está bloqueado por la configuración de privacidad o se ha alcanzado la cuota.

---

## 2. Guardar y recuperar valores

Las claves y los valores se guardan como cadenas. Usa la API de métodos, no el acceso a propiedades del objeto:

```js
localStorage.setItem("theme", "dark");

const theme = localStorage.getItem("theme");
console.log(theme); // "dark"
```

Métodos y propiedades principales:

| API | Acción |
| --- | --- |
| `setItem(key, value)` | Guarda una clave y un valor |
| `getItem(key)` | Devuelve el valor o `null` si no existe |
| `removeItem(key)` | Elimina una clave |
| `clear()` | Elimina todas las claves del almacén de ese origen |
| `key(index)` | Obtiene la clave de una posición |
| `length` | Número de claves almacenadas |

```js
localStorage.setItem("draft", "Texto sin terminar");
console.log(localStorage.length);
console.log(localStorage.key(0));

localStorage.removeItem("draft");
```

No uses `localStorage.theme` o `localStorage["theme"]` como sustituto de `setItem`/`getItem`. Las propiedades pueden entrar en conflicto con nombres propios de la API y su uso no genera el evento `storage` como se espera.

El almacenamiento no es un `Map` y no se recorre directamente con `for...of`. Se puede recorrer por índice:

```js
for (let index = 0; index < localStorage.length; index++) {
  const key = localStorage.key(index);

  if (key !== null) {
    console.log(key, localStorage.getItem(key));
  }
}
```

---

## 3. Guardar objetos con JSON

Al pasar un objeto directamente a `setItem`, se convierte en una cadena poco útil como `"[object Object]"`. Para conservar su estructura, utiliza JSON:

```js
const preferences = {
  theme: "dark",
  fontSize: 18,
};

localStorage.setItem("preferences", JSON.stringify(preferences));

const saved = localStorage.getItem("preferences");
const restored = saved === null ? null : JSON.parse(saved);

console.log(restored?.theme); // "dark"
```

El contenido puede estar corrupto o tener un formato distinto al esperado. Gestiona el error de análisis y valida los datos:

```js
function readPreferences() {
  const saved = localStorage.getItem("preferences");

  if (saved === null) {
    return { theme: "light", fontSize: 16 };
  }

  try {
    const value = JSON.parse(saved);

    if (
      typeof value !== "object" ||
      value === null ||
      typeof value.theme !== "string" ||
      typeof value.fontSize !== "number"
    ) {
      throw new Error("Formato de preferencias no válido");
    }

    return value;
  } catch (error) {
    console.error("No se pudieron leer las preferencias:", error);
    return { theme: "light", fontSize: 16 };
  }
}
```

---

## 4. Guardar automáticamente un borrador

El siguiente ejemplo restaura el texto guardado y lo actualiza al cambiar el `textarea`:

```html
<label for="draft">Borrador</label>
<textarea id="draft" rows="8"></textarea>
<button id="clear-draft" type="button">Borrar borrador</button>
```

```js
const draftKey = "my-app:draft";
const draft = document.querySelector("#draft");
const clearButton = document.querySelector("#clear-draft");

draft.value = localStorage.getItem(draftKey) ?? "";

draft.addEventListener("input", () => {
  try {
    localStorage.setItem(draftKey, draft.value);
  } catch (error) {
    console.error("No se pudo guardar el borrador:", error);
  }
});

clearButton.addEventListener("click", () => {
  localStorage.removeItem(draftKey);
  draft.value = "";
});
```

El código usa `input`, por lo que también detecta cambios hechos al pegar texto. Para datos que solo deban durar mientras se usa una pestaña, se puede sustituir `localStorage` por `sessionStorage`.

---

## 5. El evento `storage`

Cuando un documento modifica `localStorage`, los otros documentos que comparten ese almacén pueden recibir el evento `storage`. El evento **no** se dispara en la ventana que realizó el cambio:

```js
window.addEventListener("storage", event => {
  console.log(event.key);
  console.log(event.oldValue);
  console.log(event.newValue);
  console.log(event.url);
  console.log(event.storageArea === localStorage);
});
```

En `clear()`, `event.key`, `event.oldValue` y `event.newValue` son `null`. Al eliminar una clave, `newValue` es `null`; al crearla, `oldValue` es `null`.

Este evento permite sincronizar interfaces entre pestañas del mismo origen:

```js
window.addEventListener("storage", event => {
  if (event.storageArea !== localStorage || event.key !== "my-app:theme") {
    return;
  }

  document.documentElement.dataset.theme = event.newValue ?? "light";
});
```

El evento de `sessionStorage` está limitado a documentos que comparten esa sesión de almacenamiento; no sirve para sincronizar pestañas independientes.

---

## 6. Seguridad y buenas prácticas

- Web Storage es accesible desde cualquier JavaScript que se ejecute en el mismo origen. No guardes contraseñas, tokens sensibles ni datos que una persona usuaria no deba poder leer.
- Un ataque de *cross-site scripting* (XSS) puede leer y modificar el almacenamiento. Evita inyectar contenido no confiable como HTML.
- Los datos guardados pueden ser modificados desde las herramientas del navegador. Trátalos como entrada no confiable y valídalos antes de usarlos.
- Usa nombres de clave específicos de la aplicación para reducir colisiones, por ejemplo `my-app:preferences`.
- Maneja errores de lectura, escritura y análisis JSON.
- `localStorage` es síncrono; no lo uses para guardar grandes cantidades de datos o datos binarios voluminosos.

---

## 7. Resumen y práctica

| Necesidad | Solución |
| --- | --- |
| Persistir datos entre sesiones | `localStorage` |
| Mantener datos solo durante la sesión de una pestaña | `sessionStorage` |
| Guardar una cadena | `setItem` / `getItem` |
| Guardar un objeto sencillo | `JSON.stringify` / `JSON.parse` |
| Eliminar una clave | `removeItem` |
| Reaccionar a cambios desde otra ventana | Evento `storage` |

### Tarea: guardar automáticamente un campo

1. Crea un `textarea` cuyo contenido se restaure al abrir la página.
2. Guarda el contenido cada vez que se edite.
3. Añade un botón para borrar el borrador guardado.
4. Comprueba el comportamiento tras recargar la página y en otra pestaña.

### Tarea: preferencia de tema

Guarda la preferencia claro/oscuro en `localStorage`. Al abrir la página, aplica la preferencia guardada; cuando cambie desde otra pestaña, actualiza la interfaz mediante el evento `storage`.
