# UD02: Arrays en JavaScript

## 1. Arrays

Un array representa una **colección ordenada** de elementos.

```js
const fruits = ["Apple", "Orange", "Plum"];
```

Los índices empiezan en `0`:

```js
console.log(fruits[0]); // Apple
console.log(fruits[1]); // Orange

fruits[2] = "Pear";
console.log(fruits.length); // 3
```

Un array puede contener valores de distintos tipos, aunque normalmente resulta más claro utilizarlo como una colección homogénea.

### Último elemento con `at`

```js
const fruits = ["Apple", "Orange", "Plum"];

console.log(fruits.at(-1)); // Plum
console.log(fruits[fruits.length - 1]); // Plum
```

Con índices negativos, `at()` cuenta desde el final.

---

## 2. Añadir y quitar elementos

| Método | Acción | Modifica el array |
| --- | --- | --- |
| `push()` | Añade al final | Sí |
| `pop()` | Extrae del final | Sí |
| `unshift()` | Añade al principio | Sí |
| `shift()` | Extrae del principio | Sí |

```js
const fruits = ["Orange"];

fruits.push("Pear");
fruits.unshift("Apple");

console.log(fruits); // ["Apple", "Orange", "Pear"]

const last = fruits.pop();
const first = fruits.shift();
```

`push` y `pop` suelen resultar más eficientes que `unshift` y `shift`, porque trabajar al principio obliga a recolocar índices.

### Arrays y referencias

Los arrays son objetos y se asignan por referencia:

```js
const fruits = ["Apple"];
const copy = fruits;

copy.push("Pear");
console.log(fruits); // ["Apple", "Pear"]
```

Para obtener una copia superficial:

```js
const copy1 = fruits.slice();
const copy2 = [...fruits];
```

---

## 3. Recorrer arrays

### `for`

```js
for (let i = 0; i < fruits.length; i++) {
  console.log(i, fruits[i]);
}
```

### `for...of`

Cuando solo necesitamos los valores:

```js
for (const fruit of fruits) {
  console.log(fruit);
}
```

En general no se debe usar `for...in` para arrays: está pensado para recorrer propiedades de objetos, no elementos de una colección ordenada.

### `forEach`

```js
fruits.forEach((fruit, index) => {
  console.log(index, fruit);
});
```

`forEach` ejecuta la callback para cada elemento y no crea un nuevo array con los resultados.

### Tarea 1: operaciones básicas

Partiendo de:

```js
const styles = ["Jazz", "Blues"];
```

1. Añade `"Rock-n-Roll"` al final.
2. Sustituye el elemento central por `"Classics"`.
3. Extrae y muestra el primer elemento.
4. Añade `"Rap"` y `"Reggae"` al principio.

---

## 4. `splice`, `slice` y `concat`

### `splice`: modificar el array

`splice(start, deleteCount, ...items)` puede eliminar, sustituir o insertar elementos.

```js
const languages = ["HTML", "CSS", "PHP"];
languages.splice(2, 1, "JavaScript");

console.log(languages); // ["HTML", "CSS", "JavaScript"]
```

Para insertar sin borrar:

```js
languages.splice(2, 0, "Git");
```

### `slice`: copiar una parte

`slice(start, end)` devuelve un nuevo array. `end` no se incluye.

```js
const values = [10, 20, 30, 40];
const part = values.slice(1, 3); // [20, 30]
```

### `concat`: combinar

```js
const front = ["HTML", "CSS"];
const back = ["JavaScript"];
const stack = front.concat(back);
```

También es habitual utilizar propagación:

```js
const stack = [...front, ...back];
```

---

## 5. Buscar elementos

### `indexOf`, `lastIndexOf` e `includes`

```js
const roles = ["user", "editor", "admin", "editor"];

roles.indexOf("editor");     // 1
roles.lastIndexOf("editor"); // 3
roles.includes("admin");     // true
```

### `find` y `findIndex`

Son especialmente útiles con arrays de objetos:

```js
const users = [
  { id: 1, name: "Ana" },
  { id: 2, name: "Luis" },
];

const user = users.find(item => item.id === 2);
const index = users.findIndex(item => item.id === 2);
```

`find` devuelve el primer elemento que cumple la condición o `undefined`. `findIndex` devuelve su índice o `-1`.

### `filter`

Devuelve un nuevo array con todos los elementos que cumplen la condición:

```js
const numbers = [1, 2, 3, 4, 5, 6];
const even = numbers.filter(number => number % 2 === 0);
```

### Tarea 2: filtrar un rango

Implementa `filterRange(arr, a, b)` para devolver un **nuevo array** con los valores comprendidos entre `a` y `b`, ambos incluidos. El array original no debe modificarse.

---

## 6. Transformar arrays

### `map`

`map` crea un nuevo array transformando cada elemento:

```js
const numbers = [1, 2, 3];
const doubled = numbers.map(number => number * 2);
```

Con objetos:

```js
const users = [
  { id: 1, name: "Ana" },
  { id: 2, name: "Luis" },
];

const names = users.map(user => user.name);
```

### `sort`

`sort()` **modifica el array** y, por defecto, compara los elementos como texto.

```js
const numbers = [1, 15, 2];
numbers.sort(); // [1, 15, 2]
```

Para ordenar números:

```js
numbers.sort((a, b) => a - b);
```

Para conservar el original podemos copiar antes de ordenar:

```js
const sorted = [...numbers].sort((a, b) => a - b);
```

### `reverse`

`reverse()` invierte el orden y también modifica el array:

```js
const values = [1, 2, 3];
values.reverse(); // [3, 2, 1]
```

---

## 7. `split` y `join`

`split` pertenece a las cadenas y permite obtener un array:

```js
const text = "Ana,Luis,Marta";
const names = text.split(",");
```

`join` hace el recorrido inverso:

```js
const message = names.join(" | ");
```

### Tarea 3: `camelize`

Crea `camelize(str)`:

```js
camelize("background-color"); // "backgroundColor"
camelize("list-style-image"); // "listStyleImage"
camelize("-webkit-transition"); // "WebkitTransition"
```

Pista: combina `split`, `map` y `join`.

---

## 8. `reduce`

`reduce` calcula un único resultado acumulando los elementos del array:

```js
const numbers = [1, 2, 3, 4];

const total = numbers.reduce((accumulator, number) => {
  return accumulator + number;
}, 0);
```

Versión corta:

```js
const total = numbers.reduce((sum, number) => sum + number, 0);
```

Es recomendable proporcionar un valor inicial, especialmente cuando el array puede estar vacío.

Ejemplo con objetos:

```js
const cart = [
  { name: "Keyboard", price: 40 },
  { name: "Mouse", price: 20 },
];

const total = cart.reduce((sum, item) => sum + item.price, 0);
```

### `some` y `every`

```js
const values = [2, 4, 6];

values.some(value => value > 5);   // true
values.every(value => value % 2 === 0); // true
```

- `some` comprueba si **algún** elemento cumple la condición.
- `every` comprueba si **todos** la cumplen.

---

## 9. Comprobar y crear arrays

`typeof` no distingue un array de otros objetos:

```js
typeof []; // "object"
```

Para comprobarlo:

```js
Array.isArray([]); // true
Array.isArray({}); // false
```

`Array.from` crea un array real a partir de un iterable o un objeto *array-like*:

```js
const chars = Array.from("Hola");
// ["H", "o", "l", "a"]
```

También puede transformar durante la conversión:

```js
const doubled = Array.from([1, 2, 3], number => number * 2);
```

---

## 10. Iterables

Un **iterable** es un valor que JavaScript puede recorrer mediante `for...of`. Los arrays y las cadenas ya son iterables.

```js
for (const char of "Hola") {
  console.log(char);
}
```

Técnicamente, un iterable implementa `Symbol.iterator`. El iterador generado dispone de un método `next()` que va devolviendo objetos con las propiedades `value` y `done`.

Para el trabajo habitual es más importante reconocer qué estructuras pueden emplearse directamente con `for...of` que implementar iteradores propios.

---

## 11. Desestructuración

La desestructuración permite extraer valores de arrays y objetos de forma declarativa.

### Arrays

```js
const coordinates = [10, 20];
const [x, y] = coordinates;
```

Podemos utilizar valores predeterminados y el operador rest:

```js
const [first = 0, second = 0, ...rest] = [10, 20, 30, 40];
```

### Objetos

```js
const user = {
  name: "Ana",
  age: 20,
};

const { name, age } = user;
```

Podemos cambiar el nombre de una variable y proporcionar un valor predeterminado:

```js
const { name: userName, isAdmin = false } = user;
```

### Parámetros de función

La desestructuración resulta muy útil cuando una función recibe varias opciones:

```js
function createCard({ title, width = 300, visible = true } = {}) {
  console.log(title, width, visible);
}

createCard({ title: "JavaScript", width: 400 });
```

---

## 12. Resumen y práctica

### ¿Qué método elegir?

| Necesidad | Método habitual |
| --- | --- |
| Añadir al final | `push` |
| Quitar del final | `pop` |
| Añadir al principio | `unshift` |
| Quitar del principio | `shift` |
| Extraer una copia parcial | `slice` |
| Insertar/eliminar en una posición | `splice` |
| Saber si existe un valor | `includes` |
| Encontrar un elemento | `find` |
| Obtener varios que cumplan una condición | `filter` |
| Transformar todos los elementos | `map` |
| Ejecutar una acción por elemento | `forEach` |
| Obtener un único resultado acumulado | `reduce` |
| Comprobar alguno/todos | `some` / `every` |
| Comprobar si es un array | `Array.isArray` |

### Mutación: atención

Métodos como `splice`, `sort` y `reverse` modifican el array original. Métodos como `slice`, `concat`, `map` y `filter` devuelven un nuevo array.

### Reto final: estadísticas de estudiantes

Dado un array:

```js
const students = [
  { id: 1, name: "Ana", grade: 8 },
  { id: 2, name: "Luis", grade: 4 },
  { id: 3, name: "Marta", grade: 7 },
];
```

Obtén, sin modificar el array original:

1. Los estudiantes aprobados.
2. Un array que contenga únicamente sus nombres.
3. La nota media.
4. El estudiante con mayor nota.
5. Una copia ordenada de mayor a menor nota.
6. Un objeto indexado por `id` utilizando `reduce`.
