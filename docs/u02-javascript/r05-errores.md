# UD02: Manejo de errores en JavaScript

## 1. Errores y `try...catch`

Los errores pueden deberse a un fallo de programación, a datos inesperados o a un problema externo. Si un error de ejecución no se gestiona, se interrumpe el flujo normal del programa.

La construcción `try...catch` permite ejecutar código que podría fallar y decidir qué hacer si se produce un error:

```js
try {
  // Código que puede producir un error
} catch (error) {
  // Código para gestionar el error
}
```

El funcionamiento es el siguiente:

1. Se ejecuta el bloque `try`.
2. Si termina sin errores, se omite `catch` y continúa la ejecución.
3. Si ocurre un error, se interrumpe el resto de `try` y se ejecuta `catch`.
4. El parámetro de `catch` recibe el valor lanzado. Lo habitual es que sea un objeto `Error`.

### Cuando no se produce un error

```js
try {
  console.log("Inicio del bloque try"); // (1)
  console.log("Fin del bloque try");    // (2)
} catch (error) {
  console.log("Este mensaje no se muestra"); // (3)
}
```

Se muestran los mensajes (1) y (2). El bloque `catch` se omite.

### Cuando se produce un error

```js
try {
  console.log("Inicio del bloque try"); // (1)
  lalala; // ReferenceError: lalala no está definida
  console.log("Esta línea no se ejecuta"); // (2)
} catch (error) {
  console.log("Se ha producido un error"); // (3)
}
```

Se muestran los mensajes (1) y (3). El error detiene la ejecución del bloque `try`, pero el programa puede continuar después de `catch`.

> `try...catch` solo captura errores que se producen durante la ejecución. No puede capturar errores de sintaxis del propio archivo: JavaScript debe poder analizar el código antes de empezar a ejecutarlo.

También se puede omitir el parámetro si no necesitamos consultar el error:

```js
try {
  ejecutarOperacion();
} catch {
  console.log("No se pudo completar la operación");
}
```

Conviene capturar solo los errores que realmente podemos gestionar. Un `catch` vacío o que oculte cualquier problema dificulta la depuración.

---

## 2. El objeto `Error`

Los errores nativos de JavaScript suelen ser objetos con estas propiedades:

| Propiedad | Contenido |
| --- | --- |
| `name` | Tipo de error, por ejemplo `"ReferenceError"` o `"TypeError"` |
| `message` | Mensaje descriptivo |
| `stack` | Traza de llamadas que ayuda a encontrar el origen del error; está disponible en los entornos habituales, aunque no forma parte del estándar ECMAScript |

```js
try {
  lalala;
} catch (error) {
  console.log(error.name);    // "ReferenceError"
  console.log(error.message); // Mensaje que indica que lalala no está definida
  console.log(error.stack);   // Traza útil para depurar
  console.log(error.constructor.name); // "ReferenceError"
  console.log(error);         // El objeto de error completo
}
```

`stack` puede incluir rutas o detalles internos de la aplicación. En una aplicación real, no se suele mostrar directamente a la persona usuaria.
También se puede consultar el nombre del constructor con `error.constructor.name`, aunque `error.name` es más habitual.

---

## 3. Gestionar datos JSON

Un caso habitual es analizar datos recibidos de otra fuente. `JSON.parse()` lanza un `SyntaxError` si la cadena no contiene JSON válido:

```js
const json = '{"name":"John","age":30}';
const user = JSON.parse(json);

console.log(user.name); // "John"
console.log(user.age);  // 30
```

Podemos capturar el error cuando los datos estén mal formados:

```js
const json = "{ json malo }";

try {
  const user = JSON.parse(json);
  console.log(user.name);
} catch (error) {
  console.log("No se pudieron leer los datos");
  console.log(error.name);    // "SyntaxError"
  console.log(error.message); // Detalles del formato incorrecto
}
```

Que el JSON sea válido no garantiza que contenga todas las propiedades que necesita la aplicación. Podemos comprobar los datos y generar un error propio:

```js
const json = '{"age":30}';

try {
  const user = JSON.parse(json);

  if (!user.name) {
    throw new Error("Falta la propiedad name");
  }

  console.log(user.name);
} catch (error) {
  console.log(`No se pudieron procesar los datos: ${error.message}`);
}
```

En este ejemplo se utiliza `Error`, porque el JSON está bien formado: el problema es que faltan datos requeridos, no que la sintaxis JSON sea incorrecta.

---

## 4. Lanzar errores con `throw`

El operador `throw` genera un error y detiene la ejecución normal del bloque actual:

```js
throw new Error("Ha ocurrido un problema");
```

JavaScript permite lanzar cualquier valor, incluso una cadena o un número, pero es preferible lanzar objetos `Error`. Así se mantienen disponibles propiedades como `name`, `message` y, normalmente, `stack`.

```js
const error = new Error("No se pudo completar la operación");
```

Hay constructores para distintos tipos de error:

```js
const generalError = new Error("Error general");
const syntaxError = new SyntaxError("Sintaxis incorrecta");
const referenceError = new ReferenceError("Referencia no válida");
const typeError = new TypeError("Tipo de valor no válido");
```

El nombre del tipo se guarda en `name` y el texto que se pasa al constructor en `message`:

```js
const error = new Error("Estas cosas pasan...");

console.log(error.name);    // "Error"
console.log(error.message); // "Estas cosas pasan..."
```

Utiliza el tipo más adecuado cuando realmente describa el problema. No conviene usar `SyntaxError` para cualquier dato incorrecto: también se pueden crear errores de aplicación con `Error`.

---

## 5. Relanzar errores

Un bloque `catch` recibe cualquier error producido en `try`, no solo los errores que esperábamos. Si solo sabemos gestionar algunos tipos, debemos volver a lanzar los demás con `throw`.

`instanceof` permite comprobar si un objeto pertenece a una clase de error:

```js
try {
  JSON.parse("{ json malo }");
} catch (error) {
  if (error instanceof SyntaxError) {
    console.log("El JSON no tiene un formato válido");
  } else {
    throw error;
  }
}
```

En este caso, el `catch` gestiona `SyntaxError`. Cualquier otro error vuelve a propagarse y puede ser gestionado por un bloque exterior:

```js
try {
  try {
    throw new ReferenceError("Falta una variable");
  } catch (error) {
    if (error instanceof SyntaxError) {
      console.log("Error de sintaxis en los datos");
    } else {
      throw error; // Relanzamiento: este bloque no sabe gestionarlo
    }
  }
} catch (error) {
  console.log(`Otro bloque ha capturado ${error.name}: ${error.message}`);
}
```

La regla práctica es: **captura los errores que sabes resolver y relanza el resto**. Así no ocultamos errores de programación mostrándolos como si fueran errores de datos.

---

## 6. La cláusula `finally`

`finally` es opcional y se ejecuta tanto si `try` termina correctamente como si se produce un error:

```js
try {
  // Intentar ejecutar una operación
} catch (error) {
  // Gestionar el error, si se produce
} finally {
  // Ejecutar siempre al terminar try o catch
}
```

Por ejemplo:

```js
try {
  console.log("Se intenta realizar el trabajo");

  if (confirm("¿Cometer un error?")) {
    throw new Error("Error de ejemplo");
  }
} catch (error) {
  console.log("El bloque catch ha capturado el error");
} finally {
  console.log("El bloque finally se ejecuta siempre");
}
```

El orden es `try → catch → finally` si se produce un error, y `try → finally` si no se produce.

`finally` también puede usarse sin `catch`:

```js
try {
  ejecutarOperacion();
} finally {
  liberarRecurso();
}
```

En ese caso, el error no se gestiona y seguirá propagándose después de ejecutar `finally`.

### Usar `finally` para completar una tarea

Podemos usarlo, por ejemplo, para medir el tiempo de una operación aunque esta falle:

```js
function fib(n) {
  if (n < 0 || !Number.isInteger(n)) {
    throw new Error("Debe ser un número entero no negativo");
  }

  return n <= 1 ? n : fib(n - 1) + fib(n - 2);
}

const input = prompt("Introduce un número entero no negativo", "35");
const num = input === null || input.trim() === "" ? Number.NaN : Number(input);
let result;
let errorMessage;
const start = Date.now();

try {
  result = fib(num);
} catch (error) {
  errorMessage = error.message;
} finally {
  const duration = Date.now() - start;
  console.log(`La ejecución tardó ${duration} ms`);
}

if (errorMessage) {
  console.log(`No se pudo calcular Fibonacci: ${errorMessage}`);
} else {
  console.log(`fib(${num}) = ${result}`);
}
```

El bloque `finally` garantiza que la duración se calcule tanto si `fib` termina correctamente como si lanza un error. La implementación recursiva de este ejemplo es sencilla, pero resulta lenta para valores grandes.

> Evita usar `return` o lanzar otro error desde `finally`: esas instrucciones pueden reemplazar el valor devuelto o el error original y hacer más difícil entender el flujo del programa.

### ¿`finally` o código después de `try...catch`?

Si el `catch` gestiona todos los errores y el flujo continúa normalmente, el código posterior puede parecer equivalente:

```js
try {
  hacerElTrabajo();
} catch (error) {
  gestionarError(error);
}

limpiar();
```

Sin embargo, si `catch` relanza el error, el flujo no llega a `limpiar()`:

```js
try {
  hacerElTrabajo();
} catch (error) {
  gestionarErrorConocido(error);
  throw error; // El código posterior no se alcanza
}

limpiar();
```

Con `finally`, la limpieza se ejecuta incluso cuando el error se relanza:

```js
try {
  hacerElTrabajo();
} catch (error) {
  gestionarErrorConocido(error);
  throw error;
} finally {
  limpiar();
}
```

Por eso `finally` es especialmente útil para liberar recursos o restaurar el estado de la aplicación, independientemente de cómo termine la operación.

---

## 7. Errores en operaciones asíncronas

Para aprender cómo funcionan las promesas, su encadenamiento y `async`/`await`, consulta [UD02: Promesas y `async`/`await`](./r06-promesas-async-await.md).

`try...catch` puede capturar el rechazo de una promesa si se espera con `await` dentro del bloque:

```js
async function cargarDatos() {
  try {
    const data = await obtenerDatos();
    console.log(data);
  } catch (error) {
    console.error("No se pudieron cargar los datos", error);
  }
}
```

Si no esperamos la promesa, el error asíncrono no se captura con ese `try...catch`:

```js
try {
  obtenerDatos(); // Falta await
} catch (error) {
  // No captura un rechazo posterior de la promesa
}
```

En las solicitudes `fetch`, una respuesta HTTP de error, como `404`, no rechaza la promesa por sí sola. Hay que comprobar `response.ok` y lanzar un error si se quiere gestionar de la misma manera:

```js
async function cargarUsuario() {
  try {
    const response = await fetch("/api/user");

    if (!response.ok) {
      throw new Error(`Error HTTP: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error("No se pudo cargar el usuario", error);
    throw error;
  }
}
```

---

## 8. Captura global en el navegador

En el navegador se puede asignar una función a `window.onerror` para recibir información sobre errores no capturados. Los parámetros incluyen el mensaje, la URL del script, la línea, la columna y el objeto de error:

```js
window.onerror = function (message, url, line, col, error) {
  console.error("Error no capturado:", {
    message,
    url,
    line,
    col,
    error,
  });
};
```

Por ejemplo, una función que falla sin un `try...catch`:

```js
function readData() {
  badFunction(); // ReferenceError si no existe badFunction
}

readData();
```

El controlador global puede registrar el problema o enviarlo a un servicio de monitorización, pero normalmente **no permite recuperar la ejecución** tras un error de programación. No debe utilizarse como sustituto de la gestión local de errores.

`window.onerror` es específico del navegador; en Node.js se utilizan mecanismos propios de ese entorno. Evita mostrar detalles internos del error a las personas usuarias: registra la información necesaria y presenta un mensaje adecuado.

---

## 9. Resumen

| Necesidad | Solución |
| --- | --- |
| Ejecutar código que puede fallar | `try...catch` |
| Consultar el tipo y el mensaje | `error.name` y `error.message` |
| Generar un error propio | `throw new Error("mensaje")` |
| Comprobar un tipo de error | `error instanceof SyntaxError` |
| Propagar un error no gestionado | `throw error` |
| Ejecutar código al terminar, siempre | `finally` |
| Capturar un rechazo de promesa | `try...catch` con `await` |
| Registrar errores no capturados en el navegador | `window.onerror` |

### Buenas prácticas

- Captura errores cuando puedas gestionarlos o aportar contexto útil.
- No uses un `catch` vacío para ocultar fallos.
- Relanza los errores que no sabes manejar.
- Para errores propios, utiliza objetos `Error` y mensajes claros.
- Usa `finally` para limpiar o liberar recursos, no para ocultar errores.
- No confundas una respuesta HTTP fallida de `fetch` con un rechazo de promesa.

### Tarea: limpieza con `finally`

Compara estos dos fragmentos:

```js
try {
  hacerElTrabajo();
} catch (error) {
  gestionarError(error);
} finally {
  limpiarElEspacioDeTrabajo();
}
```

```js
try {
  hacerElTrabajo();
} catch (error) {
  gestionarError(error);
}

limpiarElEspacioDeTrabajo();
```

¿En qué caso no se ejecutaría la limpieza del segundo fragmento? Modifica el `catch` para que relance un error desconocido y explica por qué `finally` resulta más adecuado.
