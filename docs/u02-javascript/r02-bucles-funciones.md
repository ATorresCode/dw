# UD02: Bucles y funciones en JavaScript

## 1. Bucles

Los bucles permiten **repetir un bloque de código** mientras se cumpla una condición o durante un número determinado de iteraciones.

### `while`

`while` comprueba la condición **antes** de cada iteración:

```js
let i = 0;

while (i < 3) {
  console.log(i);
  i++;
}
```

Si la condición nunca pasa a ser falsa, se crea un bucle infinito. Por eso debemos comprobar que el cuerpo del bucle modifica aquello de lo que depende la condición.

### `do...while`

`do...while` ejecuta primero el cuerpo y después comprueba la condición. Por tanto, el cuerpo se ejecuta **al menos una vez**.

```js
let i = 0;

do {
  console.log(i);
  i++;
} while (i < 3);
```

### `for`

`for` reúne la inicialización, la condición y la actualización en una única estructura:

```js
for (let i = 0; i < 3; i++) {
  console.log(i);
}
```

| Parte | Ejemplo | Momento de ejecución |
| --- | --- | --- |
| Inicialización | `let i = 0` | Una sola vez, al comenzar |
| Condición | `i < 3` | Antes de cada iteración |
| Cuerpo | `console.log(i)` | Mientras la condición sea verdadera |
| Actualización | `i++` | Después de cada iteración |

Cualquiera de las tres partes puede omitirse, pero los dos `;` siguen siendo necesarios:

```js
let i = 0;

for (; i < 3;) {
  console.log(i++);
}
```

---

## 2. `break` y `continue`

`break` termina por completo el bucle actual:

```js
for (let i = 1; i <= 10; i++) {
  if (i === 6) {
    break;
  }
  console.log(i);
}
```

`continue` termina únicamente la iteración actual y pasa a la siguiente:

```js
for (let i = 1; i <= 10; i++) {
  if (i % 2 === 0) {
    continue;
  }
  console.log(i); // 1, 3, 5, 7, 9
}
```

JavaScript permite usar etiquetas con `break` y `continue` para controlar bucles anidados, aunque conviene reservarlas para casos en los que realmente mejoren la claridad.

### Tarea 1: bucles

1. Muestra los números pares del 2 al 10 con `for`.
2. Reescribe el siguiente código usando `while` sin cambiar su salida:

```js
for (let i = 0; i < 3; i++) {
  console.log(`Número ${i}`);
}
```

3. Pide números hasta que el usuario introduzca uno mayor que 100, cancele o deje la entrada vacía.
4. Muestra todos los números primos entre 2 y `n`.

---

## 3. `switch`

`switch` es útil cuando una misma expresión debe compararse con varios valores.

```js
const role = "editor";

switch (role) {
  case "admin":
    console.log("Administración");
    break;
  case "editor":
    console.log("Edición");
    break;
  default:
    console.log("Usuario");
}
```

La comparación de cada `case` es estricta. Si se omite `break`, la ejecución continúa por los casos siguientes.

También se pueden agrupar casos:

```js
switch (browser) {
  case "Chrome":
  case "Firefox":
  case "Safari":
    console.log("Navegador compatible");
    break;
  default:
    console.log("Compatibilidad no comprobada");
}
```

### Tarea 2: `switch`

Crea un `switch` que reciba un número del 1 al 7 y muestre el día de la semana correspondiente. Añade un `default` para valores no válidos.

---

## 4. Funciones

Una función agrupa instrucciones para realizar una tarea. Permite **reutilizar código, dividir problemas y expresar mejor la intención**.

```js
function showMessage() {
  console.log("Hola");
}

showMessage();
```

### Parámetros y argumentos

Los **parámetros** aparecen en la declaración de la función. Los **argumentos** son los valores usados al llamarla.

```js
function greet(name, message) {
  console.log(`${message}, ${name}`);
}

greet("Ana", "Hola");
```

Podemos definir valores predeterminados:

```js
function greet(name, message = "Hola") {
  return `${message}, ${name}`;
}
```

### Variables locales y externas

Una variable declarada dentro de una función es local a esa función:

```js
function calculateTotal() {
  const tax = 0.21;
  return 100 * (1 + tax);
}
```

Es preferible que una función reciba los datos mediante parámetros y devuelva un resultado, en lugar de depender innecesariamente de variables externas.

### `return`

`return` devuelve un resultado y termina inmediatamente la ejecución de la función:

```js
function min(a, b) {
  if (a < b) {
    return a;
  }
  return b;
}
```

Si una función finaliza sin devolver expresamente un valor, su resultado es `undefined`.

### Nombres de funciones

Una función representa una acción. Conviene utilizar verbos y nombres descriptivos:

- `getUser()` obtiene un valor.
- `calculateTotal()` calcula un resultado.
- `createCard()` crea algo.
- `checkAccess()` comprueba una condición.
- `showMessage()` muestra información.

Una función pequeña que hace una sola cosa suele ser más fácil de leer, probar y reutilizar.

### Tarea 3: funciones

Implementa:

```js
min(2, 5);  // 2
min(3, -1); // -1
min(1, 1);  // 1
```

Después crea `pow(x, n)` para calcular `x` elevado a un número natural `n` mediante repetición.

---

## 5. Las funciones son valores

En JavaScript una función también es un valor: puede almacenarse en una variable, copiarse o pasarse como argumento.

```js
const sayHi = function () {
  console.log("Hola");
};

const anotherFunction = sayHi;
anotherFunction();
```

### Declaración y expresión de función

Declaración:

```js
function sum(a, b) {
  return a + b;
}
```

Expresión:

```js
const sum = function (a, b) {
  return a + b;
};
```

Las declaraciones de función pueden utilizarse antes de aparecer textualmente en su bloque. Las expresiones solo están disponibles después de que se haya ejecutado su asignación.

### Callbacks

Una función puede recibirse como argumento para ejecutarla cuando corresponda:

```js
function processValue(value, callback) {
  return callback(value);
}

const result = processValue(5, function (n) {
  return n * 2;
});
```

La función recibida en `callback` es una **función callback**.

---

## 6. Funciones flecha

Las funciones flecha ofrecen una sintaxis compacta:

```js
const sum = (a, b) => a + b;
const double = n => n * 2;
const sayHi = () => console.log("Hola");
```

Si el cuerpo contiene una única expresión, su resultado se devuelve implícitamente. Con varias instrucciones necesitamos llaves y, si queremos devolver un valor, `return`:

```js
const sumAndDouble = (a, b) => {
  const sum = a + b;
  return sum * 2;
};
```

### Tarea 4: funciones flecha

Reescribe con funciones flecha:

```js
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(function (number) {
  return number * 2;
});
```

---

## 7. Resumen y práctica

- `while` comprueba antes de iterar; `do...while`, después.
- `for` resulta adecuado cuando la inicialización, condición y actualización forman una secuencia clara.
- `break` termina un bucle y `continue` salta a la siguiente iteración.
- `switch` compara una expresión con varios casos mediante igualdad estricta.
- Las funciones encapsulan acciones y pueden recibir parámetros y devolver valores.
- En JavaScript las funciones son valores y pueden utilizarse como callbacks.
- Las funciones flecha son especialmente cómodas para callbacks y transformaciones cortas.

### Reto final: analizador de números

Crea un programa que:

1. Pida números hasta que el usuario cancele o deje la entrada vacía.
2. Los almacene en un array.
3. Use funciones separadas para calcular suma, mínimo y cantidad de números pares.
4. Muestre los resultados al terminar.
5. Utilice al menos una función flecha.
