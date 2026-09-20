# UF1.1: Introducción a JavaScript

---

## 1. JavaScript en la web

JavaScript es un lenguaje de programación que permite crear **comportamiento e interacción** en una página web. Puede responder a eventos, modificar el contenido y los estilos del documento, validar formularios, animar elementos y comunicarse con servidores y APIs.

- **HTML** estructura el contenido.
- **CSS** presenta el contenido.
- **JavaScript** aporta comportamiento e interacción.

### JavaScript, ECMAScript y el navegador

- **ECMAScript** es el estándar que define el lenguaje.
- **JavaScript** es la implementación y el ecosistema más conocido alrededor de ese estándar.
- El navegador incorpora un motor JavaScript, como V8, SpiderMonkey o JavaScriptCore.
- El código puede ejecutarse directamente en el navegador; no es necesario compilarlo manualmente.
- Los motores modernos pueden optimizar y compilar partes del código durante la ejecución.

> JavaScript no es Java: son lenguajes diferentes.

### Cómo probar JavaScript

En la consola del navegador:

```js
const message = "Hola, PW";
console.log(message);
```

En una página HTML:

```html
<script>
  alert("Hola desde JavaScript");
</script>
```

En un archivo externo:

```html
<script src="app.js" defer></script>
```

El atributo `defer` descarga el script sin bloquear el análisis del HTML y lo ejecuta cuando el documento ya se ha analizado.

---

## 2. Variables y constantes

Una variable es un nombre asociado a un valor.

```js
let message;
message = "Hola";

console.log(message);
```

También se puede declarar y asignar en una sola línea:

```js
let userName = "Javier";
let age = 18;
```

Se recomienda declarar una variable por línea y utilizar nombres que expliquen su contenido.

### `let`, `const` y `var`

| Palabra | Uso recomendado | ¿Se puede reasignar? |
| --- | --- | --- |
| `let` | Variable cuyo valor cambiará | Sí |
| `const` | Referencia que no se reasignará | No |
| `var` | Código antiguo | Sí, pero se debe evitar |

```js
let score = 0;
score = 10;

const school = "IES Benigasló";
// school = "Otro centro"; // TypeError
```

`const` debe recibir un valor en el momento de declararse.

### Nombres de variables

Reglas principales:

- Pueden contener letras, dígitos, `_` y `$`.
- No pueden comenzar por un dígito.
- Distinguen mayúsculas y minúsculas.
- No pueden coincidir con palabras reservadas como `let`, `class`, `return` o `function`.

```js
let currentUserName = "Javier";
let planetName = "Tierra";
```

Se usa `camelCase` para variables y nombres descriptivos.

### Constantes con nombre significativo

Las constantes que representan valores conocidos antes de ejecutar el programa suelen escribirse en mayúsculas:

```js
const COLOR_ORANGE = "#ff7f00";
const MAX_LOGIN_ATTEMPTS = 3;

let selectedColor = COLOR_ORANGE;
```

No todas las constantes tienen que escribirse en mayúsculas:

```js
const birthday = "18.04.1986";
const age = calculateAge(birthday);
```

La convención en mayúsculas es especialmente útil para **valores fijos y difíciles de recordar**.

### Tarea 1: variables

1. Declara `admin` y `name`.
2. Asigna `"Javier"` a `name`.
3. Copia el valor de `name` a `admin`.
4. Muestra `admin` con `alert`.
5. Declara una variable para el nombre del planeta.
6. Declara otra para el usuario actual de un sitio web.

---

## 3. Tipos de datos

JavaScript tiene ocho tipos básicos:

| Tipo | Ejemplo | Uso |
| --- | --- | --- |
| `number` | `42`, `3.14` | Números |
| `bigint` | `123n` | Enteros muy grandes |
| `string` | `"Hola"` | Texto |
| `boolean` | `true` | Verdadero o falso |
| `null` | `null` | Ausencia intencionada |
| `undefined` | `undefined` | Sin valor asignado |
| `symbol` | `Symbol("id")` | Identificador único |
| `object` | `{ name: "Ana" }` | Estructuras complejas |

Los siete primeros, salvo `object`, se consideran tipos primitivos.

### `number` y `bigint`

`number` representa enteros y decimales de doble precisión:

```js
const total = 19.95;
const divisionByZero = 1 / 0; // Infinity
const invalidResult = "texto" / 2; // NaN
```

Los enteros seguros llegan hasta `Number.MAX_SAFE_INTEGER`. Para enteros de tamaño arbitrario se usa `bigint`:

```js
const hugeNumber = 123456789012345678901234567890n;
```

No se mezclan directamente `number` y `bigint` en una misma operación.

### Cadenas de texto

```js
const doubleQuotes = "Hola";
const singleQuotes = 'Hola';
const template = `Hola`;
```

Los *backticks* permiten interpolar valores y expresiones:

```js
const name = "Ilya";
console.log(`Hola ${name}`);
console.log(`El resultado es ${1 + 2}`);
```

`"name"` es texto literal; `${name}` utiliza el contenido de la variable.

### Booleanos, `null` y `undefined`

```js
const isLoggedIn = true;
const formWasSent = false;

let selectedUser = null; // todavía no hay usuario elegido
let result; // undefined: no se ha asignado valor
```

- `null`: ausencia de valor decidida explícitamente.
- `undefined`: valor que todavía no ha sido asignado.
- No conviene usar `undefined` como sustituto general de `null`.

### El operador `typeof`

`typeof` devuelve una cadena con el tipo detectado:

```js
typeof undefined; // "undefined"
typeof 42; // "number"
typeof 10n; // "bigint"
typeof true; // "boolean"
typeof "texto"; // "string"
typeof Symbol("id"); // "symbol"
typeof {}; // "object"
typeof null; // "object"
typeof alert; // "function"
```

El resultado `typeof null === "object"` es un comportamiento histórico del lenguaje. `typeof` es un **operador**, no una función.

### Tarea 2: interpolación

¿Cuál es la salida de cada `alert`?

```js
let name = "Ilya";

alert(`Hola ${1}`);
alert(`Hola ${"name"}`);
alert(`Hola ${name}`);
```

Antes de ejecutar, razona qué parte es texto literal y cuál es una expresión JavaScript.

---

## 4. Conversión de tipos

JavaScript convierte valores automáticamente cuando una operación lo necesita. También se puede convertir de forma explícita:

```js
String(value); // a texto
Number(value); // a número
Boolean(value); // a booleano
```

Tres preguntas útiles:

1. ¿Qué tipo tiene el valor ahora?
2. ¿Qué tipo necesita la operación?
3. ¿La conversión puede producir `NaN`, `null` o `undefined`?

### Conversión a `string`

```js
String(true); // "true"
String(42); // "42"
String(null); // "null"
String(undefined); // "undefined"
```

`alert` muestra cualquier valor como texto, pero no cambia necesariamente el tipo original:

```js
const value = true;
alert(value);
console.log(typeof value); // "boolean"
```

### Conversión a `number`

```js
Number("123"); // 123
Number("  123  "); // 123
Number(""); // 0
Number("12px"); // NaN
Number(true); // 1
Number(false); // 0
Number(null); // 0
Number(undefined); // NaN
```

El operador unario `+` también puede convertir:

```js
const first = "2";
const second = "3";
const total = +first + +second; // 5
```

### Conversión a `boolean`

Valores falsos (*falsy*):

```js
false, 0, -0, "", null, undefined, NaN
```

El resto son verdaderos (*truthy*), incluido el texto `"0"` y un array vacío.

```js
Boolean(1); // true
Boolean(0); // false
Boolean("hola"); // true
Boolean(""); // false
Boolean("0"); // true
```

Esta conversión aparece automáticamente en `if`, `while` y operadores lógicos.

### Tarea 3: conversión y suma

Indica el resultado de cada expresión:

```js
"" + 1 + 0
"" - 1 + 0
true + false
6 / "3"
"2" * "3"
4 + 5 + "px"
"$" + 4 + 5
"4" - 2
"4px" - 2
" -9 " + 5
" -9 " - 5
null + 1
undefined + 1
" \t \n" - 2
```

Corrige también este programa para que sume números:

```js
const a = prompt("¿Primer número?", 1);
const b = prompt("¿Segundo número?", 2);
alert(a + b); // "12"
```

---

## 5. Operadores

### Operadores aritméticos

| Operador | Significado | Ejemplo |
| --- | --- | --- |
| `+` | Suma | `2 + 3` |
| `-` | Resta | `5 - 2` |
| `*` | Multiplicación | `3 * 4` |
| `/` | División | `8 / 2` |
| `%` | Resto | `8 % 3` |
| `**` | Exponenciación | `2 ** 3` |

```js
5 % 2; // 1
2 ** 3; // 8
4 ** (1 / 2); // 2
```

El operador `+` también concatena cadenas:

```js
"1" + 2; // "12"
2 + "1"; // "21"
2 + 2 + "1"; // "41"
"1" + 2 + 2; // "122"
```

Cuando la intención sea numérica, se debe convertir de forma explícita para que el código sea claro.

### Precedencia y paréntesis

Los operadores se ejecutan según su precedencia. Los paréntesis permiten expresar la intención:

```js
const result = (1 + 2) * 2; // 6
```

Orden simplificado:

1. Paréntesis.
2. Operadores unarios: `+x`, `-x`, `!x`.
3. Exponenciación: `**`.
4. Multiplicación, división y resto.
5. Suma y resta.
6. Comparaciones.
7. Operadores lógicos.
8. Asignación.

Ante la duda, usa paréntesis.

### Asignación e incremento

La asignación devuelve el valor asignado:

```js
let a = 1;
let b = 2;
let c = 3 - (a = b + 1);
// a vale 3 y c vale 0
```

Las asignaciones compuestas combinan operación y asignación:

```js
let n = 2;
n += 5;
n *= 2;
n -= 3;
n /= 2;
```

Prefijo y sufijo modifican la variable, pero no devuelven el mismo valor:

```js
let counter = 1;
const prefix = ++counter; // counter: 2, prefix: 2

counter = 1;
const suffix = counter++; // counter: 2, suffix: 1
```

### Tarea 4: prefijo, sufijo y asignación

Calcula los valores finales:

```js
let a = 1;
let b = 1;
let c = ++a;
let d = b++;

let x = 2;
let y = 1 + (x *= 2);
```

Indica el valor de cada variable y explica el orden de evaluación.

---

## 6. Comparaciones y condiciones

Las comparaciones devuelven un booleano:

```js
5 > 4; // true
5 <= 4; // false
5 == "5"; // true: conversión de tipo
5 === "5"; // false: tipos diferentes
5 != "5"; // false
5 !== "5"; // true
```

Se recomienda utilizar `===` y `!==`, salvo que se necesite expresamente la conversión de `==` o `!=`.

### Comparar cadenas

Las cadenas se comparan carácter a carácter en orden lexicográfico:

```js
"Z" > "A"; // true
"Glow" > "Glee"; // true
"Bee" > "Be"; // true
"a" > "A"; // true en Unicode
```

Para ordenar texto según el idioma, suele ser más adecuado `localeCompare`.

### `null` y `undefined` al comparar

```js
null == undefined; // true
null === undefined; // false

null > 0; // false
null == 0; // false
null >= 0; // true
```

`undefined` no es comparable numéricamente de forma útil. Comprueba `null` y `undefined` por separado cuando puedan aparecer.

### `if`, `else` y `else if`

```js
const year = Number(prompt("¿En qué año se publicó ECMAScript 2015?"));

if (year < 2015) {
  alert("Demasiado pronto");
} else if (year > 2015) {
  alert("Demasiado tarde");
} else {
  alert("¡Exactamente!");
}
```

Solo se ejecuta el primer bloque cuya condición es verdadera. Se recomienda usar llaves incluso cuando el bloque tenga una sola instrucción.

### Operador ternario `? :`

Sirve para elegir un valor según una condición:

```js
const age = 20;
const accessAllowed = age >= 18 ? true : false;
```

Como la comparación ya devuelve un booleano, se puede escribir:

```js
const accessAllowed = age >= 18;
```

El ternario sirve para **calcular un valor**. Para ejecutar bloques de código distintos es preferible `if`.

### Tarea 5: comparaciones y condicionales

Predice el resultado:

```js
5 > 4
"apple" > "pineapple"
"2" > "12"
undefined == null
undefined === null
null == "\n0\n"
null === +"\n0\n"
```

Después, pide un número y muestra `1`, `-1` o `0` según sea positivo, negativo o cero.

---

## 7. Operadores lógicos

JavaScript dispone de `||` (OR), `&&` (AND), `!` (NOT) y `??` (fusión de nulos). Pueden trabajar con valores de cualquier tipo y devolver un valor que no tiene por qué ser booleano.

### OR `||`

Con otros valores, devuelve el primer valor *truthy* o el último si todos son *falsy*:

```js
null || 2 || undefined; // 2
0 || "" || null; // null
```

Usa cortocircuito: los operandos se evalúan de izquierda a derecha y se detiene al encontrar un valor verdadero.

```js
const firstName = "";
const lastName = "";
const nickName = "SuperCoder";

const displayName = firstName || lastName || nickName || "Anónimo";
```

### AND `&&`

Devuelve el primer valor *falsy* o el último si todos son *truthy*:

```js
1 && 5; // 5
1 && null && 2; // null
0 && "cualquier"; // 0

const user = null;
user && user.showProfile();
```

`&&` tiene más precedencia que `||`.

### NOT `!`

Convierte el operando a booleano y devuelve el contrario:

```js
!true; // false
!0; // true
!""; // true

Boolean("texto"); // true
Boolean(null); // false
```

### `??`: primer valor definido

`a ?? b` devuelve `a` salvo que `a` sea `null` o `undefined`:

```js
null ?? "valor por defecto"; // "valor por defecto"
undefined ?? "valor por defecto"; // "valor por defecto"
0 ?? 100; // 0
"" ?? "valor por defecto"; // ""
false ?? true; // false
```

A diferencia de `||`, conserva `0`, `""` y `false`:

```js
const height = 0;
height || 100; // 100
height ?? 100; // 0
```

No se debe mezclar `??` directamente con `&&` o `||` sin paréntesis:

```js
const value = (firstName || lastName) ?? "Anónimo";
```

### Tarea 6: lógica y cortocircuito

Predice la salida:

```js
alert(null || 2 || undefined);
alert(alert(1) || 2 || alert(3));
alert(1 && null && 2);
alert(alert(1) && alert(2));
alert(null || 2 && 3 || 4);
```

Escribe también condiciones para comprobar que `age` está entre 14 y 90, inclusive, y que no está entre esos valores.

---

## 8. Resumen y práctica

### Variables y tipos

- Declara con `let` lo que vaya a cambiar.
- Usa `const` cuando no vayas a reasignar la variable.
- Evita `var` en código nuevo.
- Usa nombres descriptivos en `camelCase`.
- Convierte de forma explícita con `String`, `Number` y `Boolean` cuando sea necesario.

### Operadores y condiciones

- `+` puede sumar o concatenar.
- Los paréntesis hacen visible el orden de evaluación.
- `===` compara sin conversión y suele ser preferible a `==`.
- `||` busca el primer valor *truthy*; `&&`, el primero *falsy*.
- `??` busca el primer valor no nulo ni indefinido.
- Usa llaves en los bloques condicionales y divide las expresiones complejas.

### Reto final: calculadora de acceso

Crea un script que:

1. Pregunte el nombre del usuario.
2. Pregunte su edad.
3. Convierta la edad a número.
4. Indique si puede acceder si tiene al menos 18 años.
5. Muestre un nombre alternativo con `??` si no se ha introducido nombre.
6. Indique con `typeof` los tipos de los valores recibidos.
7. Evite aceptar edades que no sean números válidos.

Entrega el código con nombres claros y, al menos, cinco casos de prueba.
