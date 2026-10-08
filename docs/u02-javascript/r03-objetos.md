# UD02: Objetos en JavaScript

## 1. Objetos y propiedades

Un objeto permite agrupar datos relacionados mediante pares **clave: valor**.

```js
const user = {
  name: "Ana",
  age: 20,
  isAdmin: false,
};
```

Las propiedades pueden almacenar valores de cualquier tipo.

### Leer, crear, modificar y eliminar propiedades

```js
console.log(user.name);
user.city = "Valencia";
user.age = 21;
delete user.isAdmin;
```

Para claves dinámicas o que no son identificadores válidos usamos corchetes:

```js
const key = "name";
console.log(user[key]);

user["favourite color"] = "blue";
```

### Propiedades calculadas y atajos

```js
const field = "score";
const name = "Ana";

const student = {
  name,
  [field]: 10,
};
```

Cuando el nombre de la propiedad coincide con el de la variable, `name` equivale a `name: name`.

---

## 2. Comprobar y recorrer propiedades

Leer una propiedad inexistente produce `undefined`:

```js
console.log(user.email); // undefined
```

Para comprobar de forma explícita si una propiedad existe podemos utilizar `in`:

```js
if ("name" in user) {
  console.log(user.name);
}
```

### `for...in`

`for...in` recorre las claves enumerables de un objeto:

```js
for (const key in user) {
  console.log(key, user[key]);
}
```

No debe confundirse con `for...of`, que veremos al trabajar con arrays e iterables.

### Tarea 1: objeto básico

1. Crea un objeto `user` vacío.
2. Añade `name` con el valor `"John"`.
3. Añade `surname` con el valor `"Smith"`.
4. Cambia `name` a `"Pete"`.
5. Elimina `name`.
6. Crea `isEmpty(obj)`, que devuelva `true` cuando un objeto no tenga propiedades enumerables.

---

## 3. Referencias y copias

Los primitivos se copian como valores independientes. Al asignar un objeto a otra variable, ambas variables hacen referencia al **mismo objeto**.

```js
const user = { name: "Ana" };
const admin = user;

admin.name = "Eva";
console.log(user.name); // "Eva"
```

Dos objetos distintos no son iguales por tener el mismo contenido:

```js
const a = {};
const b = {};

console.log(a === b); // false

const c = a;
console.log(a === c); // true
```

Que una variable esté declarada con `const` impide reasignar la referencia, no modificar las propiedades del objeto.

### Copia superficial

Podemos crear una copia superficial con `Object.assign` o con propagación:

```js
const original = { name: "Ana", age: 20 };
const copy1 = Object.assign({}, original);
const copy2 = { ...original };
```

Si existen objetos anidados, una copia superficial mantiene referencias compartidas a esos objetos internos.

### Copia profunda con `structuredClone`

```js
const original = {
  name: "Ana",
  address: { city: "Valencia" },
};

const copy = structuredClone(original);
copy.address.city = "Castellón";

console.log(original.address.city); // "Valencia"
```

`structuredClone` admite muchas estructuras de datos, pero no puede clonar funciones.

### Tarea 2: referencias

Predice la salida antes de ejecutar:

```js
const first = { score: 1 };
const second = first;
second.score++;

console.log(first.score);
console.log(first === second);
```

Después crea una variante en la que `second` sea una copia independiente.

---

## 4. Métodos y `this`

Una función almacenada como propiedad de un objeto se denomina **método**.

```js
const user = {
  name: "Ana",
  sayHi() {
    console.log(`Hola, soy ${this.name}`);
  },
};

user.sayHi();
```

En una llamada con la forma `obj.method()`, `this` referencia normalmente al objeto situado antes del punto.

```js
function showName() {
  console.log(this.name);
}

const firstUser = { name: "Ana", showName };
const secondUser = { name: "Luis", showName };

firstUser.showName();  // Ana
secondUser.showName(); // Luis
```

### Funciones flecha y `this`

Las funciones flecha **no crean su propio `this`**. Lo toman del ámbito exterior. Por ello no son un reemplazo directo de los métodos que necesitan un `this` dinámico.

```js
const user = {
  name: "Ana",
  sayHi() {
    const show = () => console.log(this.name);
    show();
  },
};
```

### Tarea 3: calculadora como objeto

Crea un objeto `calculator` con:

- `read()`: pide dos valores y los guarda como `a` y `b`.
- `sum()`: devuelve su suma.
- `mul()`: devuelve su producto.

Usa `this` para acceder a las propiedades.

---

## 5. Funciones constructoras y `new`

Cuando necesitamos crear varios objetos semejantes podemos emplear una función constructora. Por convención, su nombre empieza por mayúscula.

```js
function User(name) {
  this.name = name;
  this.isAdmin = false;
}

const ana = new User("Ana");
const luis = new User("Luis");
```

Al ejecutar `new User(...)`, JavaScript crea un nuevo objeto, establece `this` a ese objeto, ejecuta el cuerpo de la función y devuelve normalmente ese `this`.

También podemos añadir métodos:

```js
function User(name) {
  this.name = name;

  this.sayHi = function () {
    console.log(`Hola, soy ${this.name}`);
  };
}
```

> Más adelante, las clases proporcionan otra sintaxis para modelar la creación de objetos.

### Tarea 4: acumulador

Crea `Accumulator(startingValue)` de forma que:

- `value` comience con `startingValue`.
- `read()` pida un número y lo acumule en `value`.

---

## 6. Encadenamiento opcional `?.`

El encadenamiento opcional permite acceder de forma segura a una propiedad cuando el valor anterior puede ser `null` o `undefined`.

```js
const user = {};

console.log(user.address?.street); // undefined
```

Formas habituales:

```js
obj?.prop
obj?.[key]
obj.method?.()
```

Podemos encadenarlo:

```js
const city = user.address?.location?.city;
```

Debemos utilizar `?.` solamente cuando la ausencia del valor sea una posibilidad válida. Usarlo indiscriminadamente puede ocultar errores en el programa.

### `?.` y `??`

Es frecuente combinar acceso opcional con un valor por defecto:

```js
const city = user.address?.city ?? "Sin ciudad";
```

---

## 7. Resumen y práctica

- `{}` crea un objeto literal.
- `obj.prop` es cómodo para claves conocidas; `obj[key]` permite claves dinámicas.
- `in` comprueba la existencia de una propiedad.
- `for...in` recorre las claves de un objeto.
- Las variables de objeto almacenan referencias.
- `{ ...obj }` y `Object.assign` realizan copias superficiales.
- `structuredClone` permite realizar muchas copias profundas.
- Un método es una función almacenada en una propiedad.
- `this` depende de cómo se llama a la función.
- Las funciones flecha no tienen su propio `this`.
- `new` permite crear objetos a partir de funciones constructoras.
- `?.` evita errores cuando una parte válida de una cadena de acceso puede no existir.

### Reto final: carrito

Crea un objeto `cart` que:

1. Tenga una propiedad `items` con varios productos `{ name, price, quantity }`.
2. Disponga de un método `getTotal()`.
3. Permita añadir productos con `addItem(item)`.
4. Permita consultar de forma segura `customer?.address?.city`.
5. Cree una copia profunda antes de aplicar cambios para conservar un estado anterior.
