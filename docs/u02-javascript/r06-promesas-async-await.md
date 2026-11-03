# UD02: Promesas y `async`/`await`

## 1. Programación asíncrona

Una operación **asíncrona** empieza ahora y termina más adelante. Mientras se completa, JavaScript puede seguir atendiendo otras tareas, como eventos o interacciones de la página.

Algunos ejemplos son una petición de red, la carga de un recurso o un temporizador:

```js
console.log("Inicio");

setTimeout(() => {
  console.log("El temporizador ha terminado");
}, 1000);

console.log("Fin");
```

La consola muestra primero `Inicio` y `Fin`. El callback del temporizador se ejecuta más tarde.

### Callbacks

Un **callback** es una función que se pasa a otra función para que esta la ejecute cuando termine una tarea.

Por ejemplo, podemos cargar un script y avisar cuando termine:

```js
function loadScript(src, callback) {
  const script = document.createElement("script");
  script.src = src;
  script.onload = () => callback(null, script);
  script.onerror = () => {
    callback(new Error(`No se pudo cargar el script ${src}`));
  };
  document.head.append(script);
}

loadScript("/scripts/app.js", (error, script) => {
  if (error) {
    console.error(error);
    return;
  }

  console.log(`Se ha cargado ${script.src}`);
});
```

Este ejemplo sigue la convención **error first callback**: el primer argumento comunica el error y los siguientes contienen el resultado. Si todo va bien, se llama al callback con `callback(null, resultado)`.

Para realizar tareas una después de otra, los callbacks se pueden anidar:

```js
loadScript("/scripts/one.js", (error, script) => {
  if (error) {
    console.error(error);
    return;
  }

  loadScript("/scripts/two.js", (error, script) => {
    if (error) {
      console.error(error);
      return;
    }

    loadScript("/scripts/three.js", (error, script) => {
      if (error) {
        console.error(error);
        return;
      }

      console.log("Se han cargado los tres scripts");
    });
  });
});
```

Con muchas operaciones, el anidamiento crece hacia la derecha y el código se vuelve difícil de leer. Este patrón se conoce como **infierno de callbacks** o *callback hell*. Las promesas permiten expresar estas secuencias de forma más ordenada.

---

## 2. Qué es una promesa

Una **promesa** es un objeto que representa el resultado futuro de una operación asíncrona. El código que inicia la operación produce el resultado; el código que necesita ese resultado se registra para recibirlo.

Una promesa tiene uno de estos estados:

| Estado | Significado |
| --- | --- |
| `pending` | La operación todavía no ha terminado |
| `fulfilled` | La operación ha terminado correctamente y tiene un valor |
| `rejected` | La operación ha fallado y tiene un motivo o error |

Una promesa empieza pendiente y, cuando se establece, pasa a `fulfilled` o `rejected`. Su estado no vuelve a cambiar.

### Crear una promesa

El constructor recibe una función ejecutora con dos funciones: `resolve` y `reject`. La ejecutora se invoca inmediatamente al crear la promesa.

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Hecho");
  }, 1000);
});
```

Para indicar que la tarea ha terminado correctamente, se llama a `resolve(valor)`. Para indicar que ha fallado, se llama a `reject(error)`:

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error("La operación ha fallado"));
  }, 1000);
});
```

> En el código moderno normalmente consumimos promesas devueltas por APIs como `fetch`. Crear una promesa manualmente es útil cuando necesitamos adaptar una operación basada en callbacks o coordinar una operación propia.

---

## 3. Consumir promesas: `then`, `catch` y `finally`

### `then`

`then` registra funciones que se ejecutarán cuando la promesa se resuelva o se rechace:

```js
promise.then(
  result => {
    console.log("Resultado:", result);
  },
  error => {
    console.error("Error:", error);
  }
);
```

Si solo interesa el resultado satisfactorio, se proporciona un argumento:

```js
promise.then(result => {
  console.log(result);
});
```

### `catch`

`catch` gestiona un rechazo. Equivale a `then(null, manejadorDeError)`:

```js
promise.catch(error => {
  console.error("La operación ha fallado:", error);
});
```

Es habitual colocar un único `catch` al final de una cadena para gestionar los errores de cualquiera de sus pasos.

### `finally`

`finally` se ejecuta tanto si la promesa se cumple como si se rechaza. Se utiliza para tareas de limpieza que no dependen del resultado, como ocultar un indicador de carga:

```js
obtenerDatos()
  .then(result => mostrarDatos(result))
  .catch(error => mostrarError(error))
  .finally(() => ocultarIndicadorDeCarga());
```

El callback de `finally` no recibe el resultado ni el error. Normalmente estos valores pasan al siguiente manejador sin cambiar. Si el propio callback de `finally` lanza un error, la promesa resultante se rechaza con ese nuevo error.

---

## 4. Encadenar promesas

Cada llamada a `then`, `catch` o `finally` devuelve una promesa nueva. Si un manejador devuelve un valor, ese valor pasa al siguiente `then`:

```js
new Promise(resolve => {
  setTimeout(() => resolve(1), 1000);
})
  .then(result => {
    console.log(result); // 1
    return result * 2;
  })
  .then(result => {
    console.log(result); // 2
    return result * 2;
  })
  .then(result => {
    console.log(result); // 4
  });
```

La salida es `1`, `2`, `4`. Cada paso recibe el resultado del anterior.

Si un manejador devuelve otra promesa, la cadena espera a que se establezca y pasa su resultado al siguiente paso:

```js
obtenerUsuario()
  .then(user => cargarPerfil(user.id))
  .then(profile => mostrarPerfil(profile))
  .catch(error => console.error("No se pudo cargar el perfil:", error));
```

No es lo mismo encadenar varios `then` sobre la misma promesa que encadenar sus resultados:

```js
const promise = Promise.resolve(1);

// Los dos manejadores reciben independientemente el valor 1.
promise.then(value => console.log(value));
promise.then(value => console.log(value));
```

Para que el segundo paso reciba el resultado del primero, se encadenan las llamadas:

```js
Promise.resolve(1)
  .then(value => value * 2)
  .then(value => console.log(value)); // 2
```

### Ejecutar tareas secuencialmente

Usando la versión promisificada de la función de carga, cada paso comienza cuando termina el anterior:

```js
loadScriptPromise("/scripts/one.js")
  .then(() => loadScriptPromise("/scripts/two.js"))
  .then(() => loadScriptPromise("/scripts/three.js"))
  .then(() => {
    one();
    two();
    three();
  })
  .catch(error => console.error("No se pudieron cargar los scripts:", error));
```

La cadena se mantiene plana y los errores pueden gestionarse en un punto común.

---

## 5. Promesas y `fetch`

`fetch(url)` devuelve una promesa. Esta se cumple cuando se reciben las cabeceras de la respuesta; los métodos como `response.json()` y `response.text()` devuelven otras promesas para leer el cuerpo.

```js
fetch("/api/user")
  .then(response => response.json())
  .then(user => {
    console.log(user.name);
  })
  .catch(error => {
    console.error("No se pudo completar la solicitud:", error);
  });
```

**Importante:** una respuesta HTTP de error, como `404` o `500`, no rechaza la promesa de `fetch` por sí sola. Hay que comprobar `response.ok`:

```js
function loadJson(url) {
  return fetch(url).then(response => {
    if (!response.ok) {
      throw new Error(`Error HTTP: ${response.status}`);
    }

    return response.json();
  });
}

loadJson("/api/user")
  .then(user => console.log(user))
  .catch(error => console.error("No se pudieron cargar los datos:", error));
```

La función `loadJson` devuelve la promesa para que quien la llame pueda encadenar operaciones o gestionar los errores.

### Devolver una promesa en cada operación

Si un paso asíncrono inicia otra operación, debe devolver su promesa. Así, la cadena espera a que ese paso también termine:

```js
function showAvatar(user) {
  return new Promise((resolve, reject) => {
    const image = document.createElement("img");
    image.src = user.avatarUrl;

    image.onload = () => resolve(user);
    image.onerror = () => reject(new Error("No se pudo cargar el avatar"));

    document.body.append(image);
  });
}

loadJson("/api/user")
  .then(user => showAvatar(user))
  .then(user => console.log(`Avatar cargado para ${user.name}`))
  .catch(error => console.error(error));
```

---

## 6. Errores de promesas

Un rechazo salta al manejador de error más cercano. Un error lanzado dentro de un ejecutor o de un manejador se convierte automáticamente en el rechazo de la promesa correspondiente:

```js
Promise.resolve()
  .then(() => {
    throw new Error("Error en el manejador");
  })
  .catch(error => {
    console.error(error.message); // "Error en el manejador"
  });
```

Esto permite colocar un `catch` al final de una cadena:

```js
fetch("/api/user")
  .then(response => {
    if (!response.ok) {
      throw new Error(`Error HTTP: ${response.status}`);
    }

    return response.json();
  })
  .then(user => mostrarUsuario(user))
  .catch(error => console.error("No se pudo cargar el usuario:", error));
```

Si el `catch` gestiona el error y termina normalmente, la cadena puede continuar con un `then`. Si no sabe gestionarlo, puede volver a lanzarlo:

```js
obtenerDatos()
  .catch(error => {
    if (error instanceof URIError) {
      console.error("La dirección contiene un formato incorrecto");
      return null;
    }

    throw error;
  })
  .then(result => {
    if (result !== null) {
      mostrarDatos(result);
    }
  })
  .catch(error => {
    console.error("Otro error que no se pudo gestionar:", error);
  });
```

### Rechazos no gestionados

Si una promesa se rechaza y no hay un `catch` que lo gestione, el navegador emite el evento `unhandledrejection`:

```js
window.addEventListener("unhandledrejection", event => {
  console.error("Rechazo de promesa no gestionado:", event.reason);
});
```

Este evento sirve para registrar errores inesperados, no sustituye a gestionar los rechazos en el lugar adecuado. Si se puede recuperar de un error, es preferible hacerlo en el flujo normal de la aplicación.

Para una explicación general de `try...catch` y `finally`, consulta [r05-errores.md](./r05-errores.md).

---

## 7. Métodos estáticos de `Promise`

Los métodos siguientes permiten combinar varias promesas. Reciben un iterable, normalmente un array.

### `Promise.all`

Espera a que todas las promesas se cumplan y devuelve un array de resultados en el mismo orden que las promesas de entrada. Si alguna se rechaza, el resultado de `Promise.all` se rechaza con ese error:

```js
const urls = ["/api/user/1", "/api/user/2", "/api/user/3"];

async function loadUsers() {
  const users = await Promise.all(urls.map(url => loadJson(url)));
  console.log(users);
}

loadUsers().catch(error => console.error("No se pudieron cargar los usuarios:", error));
```

Utilízalo cuando necesites que todas las operaciones terminen correctamente antes de continuar.

### `Promise.allSettled`

Espera a que todas las promesas se establezcan, tanto si se cumplen como si se rechazan. Devuelve un resultado para cada una:

```js
async function inspectUsers() {
  const results = await Promise.allSettled([
    loadJson("/api/user/1"),
    loadJson("/api/user/2"),
    loadJson("/api/user/unknown"),
  ]);

  for (const result of results) {
    if (result.status === "fulfilled") {
      console.log("Resultado:", result.value);
    } else {
      console.error("Error:", result.reason);
    }
  }
}

inspectUsers().catch(error => console.error("No se pudieron revisar los usuarios:", error));
```

Los resultados tienen esta forma:

```js
[
  { status: "fulfilled", value: resultado },
  { status: "rejected", reason: error },
]
```

Es útil cuando cada operación puede tener éxito o fallar de manera independiente. En entornos actuales no se necesita instalar manualmente un *polyfill* para `Promise.allSettled`.

### `Promise.race`

Se establece con la primera promesa que se resuelva **o se rechace**:

```js
const firstResult = await Promise.race([
  new Promise(resolve => setTimeout(() => resolve("rápida"), 100)),
  new Promise(resolve => setTimeout(() => resolve("lenta"), 500)),
]);

console.log(firstResult); // "rápida"
```

`Promise.race` no cancela las operaciones que pierden la carrera; solo deja de esperar sus resultados.

### `Promise.any`

Se cumple con el primer resultado satisfactorio. Ignora los rechazos mientras haya alguna promesa pendiente; si todas se rechazan, produce un `AggregateError` con los errores en `error.errors`:

```js
async function useFirstAvailable() {
  const result = await Promise.any([
    Promise.reject(new Error("Primer servicio no disponible")),
    Promise.resolve("Respuesta del segundo servicio"),
  ]);

  console.log(result); // "Respuesta del segundo servicio"
}

useFirstAvailable();
```

Si todas fallan:

```js
async function inspectFailedServices() {
  try {
    await Promise.any([
      Promise.reject(new Error("Error A")),
      Promise.reject(new Error("Error B")),
    ]);
  } catch (error) {
    if (error instanceof AggregateError) {
      console.error(error.errors);
    }
  }
}

inspectFailedServices();
```

### `Promise.resolve` y `Promise.reject`

`Promise.resolve(value)` crea una promesa cumplida con un valor. Si el valor ya es una promesa, la devuelve adoptando su resultado:

```js
Promise.resolve("Listo").then(value => console.log(value));
```

`Promise.reject(error)` crea una promesa rechazada:

```js
Promise.reject(new Error("No se pudo completar la operación"))
  .catch(error => console.error(error.message));
```

En código moderno estos métodos aparecen menos al trabajar directamente con `async`/`await`, pero siguen siendo útiles para normalizar resultados o construir ejemplos.

### Resumen de los métodos

| Método | Resultado |
| --- | --- |
| `Promise.all(iterable)` | Todos los resultados; rechaza si una entrada se rechaza |
| `Promise.allSettled(iterable)` | Estado y resultado/error de cada entrada |
| `Promise.race(iterable)` | Resultado o error de la primera promesa que se establece |
| `Promise.any(iterable)` | Resultado de la primera promesa cumplida; si todas fallan, `AggregateError` |
| `Promise.resolve(value)` | Promesa cumplida con el valor |
| `Promise.reject(error)` | Promesa rechazada con el error |

---

## 8. Promisificar callbacks

**Promisificar** es convertir una función basada en callbacks en otra que devuelve una promesa. Esto permite integrarla en una cadena con `then` o usarla con `await`.

```js
function loadScript(src, callback) {
  const script = document.createElement("script");
  script.src = src;
  script.onload = () => callback(null, script);
  script.onerror = () => {
    callback(new Error(`No se pudo cargar ${src}`));
  };
  document.head.append(script);
}

function loadScriptPromise(src) {
  return new Promise((resolve, reject) => {
    loadScript(src, (error, script) => {
      if (error) {
        reject(error);
      } else {
        resolve(script);
      }
    });
  });
}

loadScriptPromise("/scripts/app.js")
  .then(script => console.log(`Se ha cargado ${script.src}`))
  .catch(error => console.error(error));
```

El siguiente ayudante ilustra la conversión para una función que utiliza la convención `callback(error, resultado)`:

```js
function promisify(callbackFunction) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      callbackFunction.call(this, ...args, (error, result) => {
        if (error) {
          reject(error);
        } else {
          resolve(result);
        }
      });
    });
  };
}
```

El ayudante supone que el callback devuelve un único resultado. Si una biblioteca tiene una convención distinta, como varios valores de resultado o ningún argumento de error, hay que adaptar la conversión a esa API. En muchos entornos modernos es preferible usar las funciones de conversión que ofrece la propia plataforma o biblioteca.

---

## 9. `async` y `await`

`async`/`await` es una sintaxis para trabajar con promesas de forma secuencial y legible. No bloquea el hilo mientras espera una operación asíncrona.

### Funciones `async`

Una función declarada con `async` siempre devuelve una promesa:

```js
async function getNumber() {
  return 1;
}

getNumber().then(value => console.log(value)); // 1
```

Es equivalente a devolver explícitamente una promesa cumplida:

```js
function getNumber() {
  return Promise.resolve(1);
}
```

Una función `async` también puede devolver una promesa que ya existe. Su resultado se adopta y se propaga al llamador.

### Esperar con `await`

`await` espera el resultado de una promesa dentro de una función `async` y devuelve el valor con el que se cumple:

```js
function delay(value, milliseconds) {
  return new Promise(resolve => {
    setTimeout(() => resolve(value), milliseconds);
  });
}

async function showMessage() {
  const result = await delay("¡Hecho!", 1000);
  console.log(result);
}

showMessage();
```

La función se pausa en `await`, pero el resto de JavaScript puede seguir ejecutándose. Fuera de una función `async`, `await` solo puede utilizarse en contextos que admiten *top-level await*, como ciertos módulos.

### Reescribir una cadena

Con `then`:

```js
function loadUser() {
  return fetch("/api/user")
    .then(response => response.json())
    .then(user => fetch(`/api/profiles/${user.id}`))
    .then(response => response.json());
}
```

Con `async`/`await`:

```js
async function loadUser() {
  const userResponse = await fetch("/api/user");
  const user = await userResponse.json();

  const profileResponse = await fetch(`/api/profiles/${user.id}`);
  return profileResponse.json();
}
```

Los métodos de una clase también pueden declararse `async`:

```js
class Waiter {
  async wait() {
    return await delay(1, 1000);
  }
}

new Waiter().wait().then(value => console.log(value));
```

---

## 10. Errores con `async`/`await`

Si una promesa rechaza, `await` lanza el motivo del rechazo como un error. Por tanto, se puede gestionar con `try...catch`:

```js
async function loadUser() {
  try {
    const response = await fetch("/api/user");

    if (!response.ok) {
      throw new Error(`Error HTTP: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error("No se pudo cargar el usuario:", error);
    throw error;
  }
}
```

Este `catch` puede registrar o transformar el error y volver a lanzarlo para que el llamador también pueda gestionarlo.

Si no hay un `try...catch` dentro de la función `async`, el error rechaza la promesa que devuelve la función:

```js
async function loadUser() {
  const response = await fetch("/api/user");

  if (!response.ok) {
    throw new Error(`Error HTTP: ${response.status}`);
  }

  return response.json();
}

loadUser().catch(error => {
  console.error("No se pudo cargar el usuario:", error);
});
```

Llamar a una función `async` sin esperar ni gestionar su promesa puede producir un rechazo no gestionado:

```js
loadUser(); // El llamador no espera ni captura el posible rechazo
```

Cuando el llamador necesita el resultado, debe usar `await` dentro de otra función `async` o encadenar un `catch`.

---

## 11. Resumen y práctica

### ¿Qué herramienta usar?

| Necesidad | Solución habitual |
| --- | --- |
| Avisar cuando termina una operación antigua basada en callbacks | Callback |
| Representar un resultado futuro o encadenar tareas | Promesa |
| Escribir un flujo de promesas de forma secuencial | `async`/`await` |
| Gestionar un rechazo en una cadena | `.catch()` |
| Gestionar un rechazo al esperar una promesa | `try...catch` |
| Esperar a que todas las operaciones tengan éxito | `Promise.all()` |
| Obtener todos los resultados, incluso los fallidos | `Promise.allSettled()` |
| Usar el primer resultado o error disponible | `Promise.race()` |
| Usar el primer resultado satisfactorio | `Promise.any()` |

### Buenas prácticas

- Devuelve las promesas de las funciones asíncronas para que el llamador pueda esperarlas o gestionar sus errores.
- Usa `Promise.all` cuando todas las operaciones sean necesarias; usa `Promise.allSettled` si los resultados independientes también son útiles cuando alguna falla.
- Comprueba `response.ok` al usar `fetch`: los errores HTTP no rechazan automáticamente la promesa.
- Gestiona los errores donde puedas recuperarte o aportar contexto útil; relanza los errores que no sabes manejar.
- Evita mezclar callbacks y promesas en una misma cadena si puedes adaptar la función una sola vez.

### Reto: cargar varios usuarios

Crea `loadUsers(ids)` para cargar en paralelo los usuarios de `/api/users/{id}`:

1. Utiliza `map` y `Promise.all` para devolver los datos de todos los usuarios.
2. Comprueba `response.ok` y genera un error si una solicitud falla.
3. Añade una variante con `Promise.allSettled` que muestre los usuarios cargados y registre los errores individuales.
4. Reescribe la función con `async`/`await`.
