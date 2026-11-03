# UD03: Solicitudes de red con `fetch` y `FormData`

## 1. Peticiones de red desde JavaScript

Una aplicación web puede solicitar datos al servidor y actualizar la interfaz sin recargar la página. Este tipo de comunicación se suele llamar **AJAX**, aunque los datos actuales suelen ser JSON y no XML.

La API `fetch()` envía una petición HTTP y devuelve una promesa. Para aprender promesas y `async`/`await`, consulta [UD02: Promesas y `async`/`await`](../u02-javascript/r06-promesas-async-await.md).

Una petición `fetch(url)` sin opciones realiza normalmente una petición `GET`:

```js
const response = await fetch("/api/users");
```

La respuesta se recibe en dos pasos:

1. `fetch` devuelve un objeto `Response` cuando se reciben las cabeceras.
2. Un método como `response.json()` lee y procesa el cuerpo; también devuelve una promesa.

---

## 2. Comprobar la respuesta y leer el cuerpo

Una respuesta HTTP como `404` o `500` **no rechaza automáticamente** la promesa de `fetch`. La promesa rechaza ante errores de red, solicitudes bloqueadas u otros problemas que impiden obtener una respuesta.

Comprueba `response.ok`, que es `true` para estados HTTP entre 200 y 299:

```js
async function loadJson(url) {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`Error HTTP ${response.status}`);
  }

  return response.json();
}

try {
  const users = await loadJson("/api/users");
  console.log(users);
} catch (error) {
  console.error("No se pudieron cargar los usuarios:", error);
}
```

Propiedades útiles de `Response`:

| Propiedad | Uso |
| --- | --- |
| `status` | Código de estado HTTP, como `200` o `404` |
| `ok` | Indica si el estado está en el rango `200`-`299` |
| `headers` | Encabezados de la respuesta |
| `url` | URL final de la respuesta |

Métodos habituales para leer el cuerpo:

| Método | Resultado |
| --- | --- |
| `response.json()` | Convierte JSON a valores JavaScript |
| `response.text()` | Lee el cuerpo como texto |
| `response.blob()` | Lee datos binarios como `Blob` |
| `response.formData()` | Analiza un cuerpo `multipart/form-data` |
| `response.arrayBuffer()` | Lee datos binarios como `ArrayBuffer` |

El cuerpo de la respuesta se consume una sola vez. No se puede llamar a `json()` y después a `text()` sobre la misma respuesta sin clonarla antes.

### Con promesas encadenadas

El mismo flujo se puede escribir sin `await`:

```js
fetch("/api/users")
  .then(response => {
    if (!response.ok) {
      throw new Error(`Error HTTP ${response.status}`);
    }

    return response.json();
  })
  .then(users => console.log(users))
  .catch(error => console.error("Error en la solicitud:", error));
```

### Leer encabezados

`response.headers` proporciona `get()` y permite recorrer los encabezados:

```js
const response = await fetch("/api/users");

console.log(response.headers.get("content-type"));

for (const [name, value] of response.headers) {
  console.log(`${name}: ${value}`);
}
```

---

## 3. Enviar datos con `fetch`

Las opciones más utilizadas son `method`, `headers` y `body`. Para enviar JSON, convierte el objeto con `JSON.stringify()` y especifica `Content-Type`:

```js
const user = {
  firstName: "Juan",
  lastName: "Pérez",
};

const response = await fetch("/api/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify(user),
});

if (!response.ok) {
  throw new Error(`Error HTTP ${response.status}`);
}

const result = await response.json();
console.log(result.message);
```

Un `body` puede ser una cadena, un objeto `FormData`, `URLSearchParams`, `Blob` o datos binarios. No todos los encabezados HTTP se pueden configurar desde JavaScript; algunos están controlados por el navegador.

Para datos codificados como formulario URL se puede usar `URLSearchParams`:

```js
const body = new URLSearchParams({
  search: "JavaScript",
  page: "1",
});

const response = await fetch("/api/search", {
  method: "POST",
  body,
});
```

---

## 4. Enviar formularios con `FormData`

`FormData` representa campos de formulario. Al construirlo a partir de un formulario, recoge sus controles con atributo `name`:

```html
<form id="profile-form">
  <label>
    Nombre
    <input name="firstName" value="Juan">
  </label>
  <label>
    Apellidos
    <input name="lastName" value="Pérez">
  </label>
  <button>Guardar</button>
</form>
```

```js
const form = document.querySelector("#profile-form");
const data = new FormData(form);

console.log(data.get("firstName")); // "Juan"
```

Se puede enviar directamente en una petición. No establezcas manualmente el encabezado `Content-Type` para `FormData`: el navegador debe añadir el límite (*boundary*) correcto para `multipart/form-data`.

```js
form.addEventListener("submit", async event => {
  event.preventDefault();

  try {
    const response = await fetch("/api/profile", {
      method: "POST",
      body: new FormData(form),
    });

    if (!response.ok) {
      throw new Error(`Error HTTP ${response.status}`);
    }

    const result = await response.json();
    console.log(result.message);
  } catch (error) {
    console.error("No se pudo guardar el perfil:", error);
  }
});
```

Métodos comunes:

| Método | Acción |
| --- | --- |
| `append(name, value)` | Añade un campo; puede haber varios con el mismo nombre |
| `set(name, value)` | Sustituye el valor o los valores existentes con ese nombre |
| `get(name)` | Obtiene el primer valor |
| `getAll(name)` | Obtiene todos los valores |
| `has(name)` | Comprueba si existe el campo |
| `delete(name)` | Elimina el campo o campos |

```js
const data = new FormData();

data.append("tag", "javascript");
data.append("tag", "web");
data.set("name", "Ana");

console.log(data.getAll("tag")); // ["javascript", "web"]

for (const [name, value] of data) {
  console.log(name, value);
}
```

### Enviar un archivo

Un formulario con `<input type="file">` incluye el archivo automáticamente:

```html
<form id="avatar-form">
  <input name="displayName" value="Ana">
  <input name="avatar" type="file" accept="image/*">
  <button>Subir</button>
</form>
```

```js
const avatarForm = document.querySelector("#avatar-form");

avatarForm.addEventListener("submit", async event => {
  event.preventDefault();

  try {
    const response = await fetch("/api/avatar", {
      method: "POST",
      body: new FormData(avatarForm),
    });

    if (!response.ok) {
      throw new Error(`Error HTTP ${response.status}`);
    }

    console.log(await response.json());
  } catch (error) {
    console.error("No se pudo subir el avatar:", error);
  }
});
```

También se puede añadir un `Blob` creado dinámicamente, como una imagen de `canvas`:

```js
const canvas = document.querySelector("#drawing");
const imageBlob = await new Promise(resolve => {
  canvas.toBlob(resolve, "image/png");
});

if (!imageBlob) {
  throw new Error("No se pudo crear la imagen");
}

const data = new FormData();
data.append("firstName", "Ana");
data.append("image", imageBlob, "drawing.png");

const response = await fetch("/api/drawings", {
  method: "POST",
  body: data,
});
```

---

## 5. Recibir y mostrar datos binarios

Para mostrar una imagen recibida como `Blob`, se puede crear una URL temporal. Cuando ya no se necesita, hay que liberarla:

```js
async function showImage(url) {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`Error HTTP ${response.status}`);
  }

  const imageBlob = await response.blob();
  const objectUrl = URL.createObjectURL(imageBlob);
  const image = document.createElement("img");

  image.src = objectUrl;
  image.alt = "Imagen recibida del servidor";
  document.body.append(image);

  image.addEventListener("load", () => {
    URL.revokeObjectURL(objectUrl);
  }, { once: true });
}
```

---

## 6. Solicitudes en paralelo

Si varias solicitudes son independientes, se pueden iniciar a la vez con `Promise.all`. El orden de los resultados coincide con el de las entradas:

```js
const urls = ["/api/users/1", "/api/users/2", "/api/users/3"];
const users = await Promise.all(urls.map(url => loadJson(url)));
```

### Tarea: usuarios de GitHub

Implementa `getUsers(names)`, que haga una petición en paralelo por cada nombre y devuelva un array en el mismo orden. Si un usuario no existe o su solicitud falla, la posición correspondiente debe contener `null`:

```js
async function getUsers(names) {
  return Promise.all(
    names.map(async name => {
      try {
        const response = await fetch(
          `https://api.github.com/users/${encodeURIComponent(name)}`
        );

        if (!response.ok) {
          return null;
        }

        return await response.json();
      } catch (error) {
        console.error(`No se pudo cargar el usuario ${name}:`, error);
        return null;
      }
    })
  );
}
```

Cada `map` inicia una petición sin esperar a las demás, y `Promise.all` conserva el orden original. Para más información sobre combinadores de promesas, consulta [UD02: Promesas y `async`/`await`](../u02-javascript/r06-promesas-async-await.md).

---

## 7. Resumen y buenas prácticas

- Comprueba `response.ok`; un estado HTTP fallido no rechaza automáticamente `fetch`.
- Comprueba el tipo de la respuesta antes de procesar su cuerpo y consume el cuerpo una sola vez.
- Usa `JSON.stringify()` y `Content-Type: application/json` al enviar JSON.
- Para `FormData`, deja que el navegador configure `Content-Type` y el *boundary*.
- Usa `try...catch` para manejar errores de red y errores del procesamiento del cuerpo.
- Valida la respuesta y los datos en el servidor; el código del navegador no es una frontera de seguridad.
- Evita incluir credenciales o secretos en el código cliente.
