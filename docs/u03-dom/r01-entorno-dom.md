# UD03: Entorno del navegador y DOM

## 1. JavaScript en el navegador

JavaScript se ejecuta en distintos entornos. El navegador, por ejemplo, proporciona objetos y funciones que no forman parte del núcleo del lenguaje y que permiten interactuar con la página.

El objeto global del navegador es `window`. También representa la ventana o pestaña y da acceso a APIs del navegador:

```js
console.log(window.innerWidth);
console.log(window.location.href);

// En el navegador, window es también el objeto global.
console.log(window.document === document); // true
```

El **DOM** (*Document Object Model*) representa el documento HTML como un árbol de objetos. El objeto `document` es el punto de entrada para consultar y modificar la página.

El **BOM** (*Browser Object Model*) comprende APIs del navegador que no representan directamente el documento, como `location`, `navigator`, `history`, temporizadores y cuadros de diálogo.

| API | Para qué sirve |
| --- | --- |
| `document` | Consultar y modificar el documento HTML |
| `location` | Consultar la URL o navegar a otra dirección |
| `navigator` | Consultar información del navegador y sus capacidades |
| `setTimeout()` | Programar una función para más adelante |
| `alert()` / `confirm()` / `prompt()` | Mostrar cuadros de diálogo del navegador |

Las capacidades disponibles dependen del entorno. Por ejemplo, `window` y `document` no están disponibles en Node.js sin una implementación del DOM.

---

## 2. El documento como árbol de nodos

El navegador analiza el HTML y crea un árbol de nodos. Las etiquetas se convierten en **nodos de elemento**; el texto y los comentarios también se representan como nodos.

```html
<body>
  <h1>Noticias</h1>
  <!-- Sección principal -->
  <p>Últimas noticias</p>
</body>
```

En forma simplificada, el árbol contiene `body`, sus elementos `h1` y `p`, nodos de texto y un nodo de comentario.

Los espacios y saltos de línea del HTML pueden convertirse en nodos de texto. El navegador también corrige ciertas omisiones al construir el documento, por ejemplo crea `html`, `head`, `body` y, en tablas, puede insertar `tbody`. Por eso el DOM inspeccionado puede no ser idéntico al texto fuente.

Los nodos más habituales son:

| Nodo | Ejemplo | Descripción |
| --- | --- | --- |
| `Document` | `document` | El documento completo |
| `Element` | `<p>`, `<div>`, `<input>` | Una etiqueta HTML |
| `Text` | `Hola` dentro de `<p>Hola</p>` | Un nodo de texto |
| `Comment` | `<!-- nota -->` | Un comentario del documento |

Las herramientas de desarrollo del navegador permiten inspeccionar el DOM, modificar estilos y propiedades temporalmente y revisar los errores de JavaScript. El panel Elements representa el DOM creado, no solo el marcado original.

---

## 3. Acceder a los nodos y recorrer el árbol

Los nodos principales se pueden obtener desde `document`:

```js
console.log(document.documentElement); // <html>
console.log(document.head);            // <head>
console.log(document.body);            // <body>
```

Un script solo puede acceder a elementos que ya existen cuando se ejecuta. Para ejecutar código cuando se haya analizado el HTML, se puede cargar el script con `defer`:

```html
<script src="app.js" defer></script>
```

También se puede esperar a `DOMContentLoaded`:

```js
document.addEventListener("DOMContentLoaded", () => {
  console.log(document.body);
});
```

### Propiedades para navegar por todos los nodos

Estas propiedades pueden incluir nodos de texto y comentarios:

```js
const list = document.querySelector("ul");

console.log(list.parentNode);
console.log(list.childNodes);
console.log(list.firstChild);
console.log(list.lastChild);
console.log(list.previousSibling);
console.log(list.nextSibling);
```

`childNodes` es una colección iterable, pero no es un array. Se puede convertir si hacen falta métodos de array:

```js
const children = Array.from(list.childNodes);
children.forEach(node => console.log(node));
```

### Propiedades que solo recorren elementos

Para ignorar espacios, nodos de texto y comentarios, suelen ser más prácticas las propiedades específicas de elementos:

```js
console.log(list.children);
console.log(list.firstElementChild);
console.log(list.lastElementChild);
console.log(list.previousElementSibling);
console.log(list.nextElementSibling);
console.log(list.parentElement);
```

| Todos los nodos | Solo elementos |
| --- | --- |
| `parentNode` | `parentElement` |
| `childNodes` | `children` |
| `firstChild` / `lastChild` | `firstElementChild` / `lastElementChild` |
| `previousSibling` / `nextSibling` | `previousElementSibling` / `nextElementSibling` |

Las tablas también ofrecen colecciones específicas, como `table.rows`, `table.tBodies`, `row.cells`, `row.rowIndex` y `cell.cellIndex`:

```html
<table id="grades">
  <tbody>
    <tr><td>Ana</td><td>8</td></tr>
    <tr><td>Luis</td><td>6</td></tr>
  </tbody>
</table>
```

```js
const table = document.querySelector("#grades");
const secondRowSecondCell = table.rows[1].cells[1];

secondRowSecondCell.textContent = "7";
console.log(table.rows.length); // 2
```

---

## 4. Buscar elementos

### `querySelector` y `querySelectorAll`

Los métodos de consulta CSS sirven para buscar por etiqueta, clase, id, atributos y combinaciones:

```js
const firstItem = document.querySelector("ul li");
const allItems = document.querySelectorAll("ul li");
const checked = document.querySelectorAll('input[type="checkbox"]:checked');
```

- `querySelector(selector)` devuelve el primer elemento que coincide o `null`.
- `querySelectorAll(selector)` devuelve una colección estática (`NodeList`) con todos los resultados.

```js
const title = document.querySelector("#page-title");

if (title) {
  title.textContent = "Bienvenidas y bienvenidos";
}
```

Comprueba el resultado de `querySelector` antes de usarlo, porque el selector puede no encontrar ningún elemento.

### Otras consultas

`getElementById` es una búsqueda directa por id:

```js
const menu = document.getElementById("main-menu");
```

También existen `getElementsByTagName`, `getElementsByClassName` y `getElementsByName`. Sus resultados suelen ser colecciones **vivas** que se actualizan cuando cambia el documento. `querySelectorAll`, en cambio, devuelve una colección estática.

| Método | Busca por | Resultado habitual |
| --- | --- | --- |
| `querySelector` | Selector CSS | Primer elemento o `null` |
| `querySelectorAll` | Selector CSS | `NodeList` estática |
| `getElementById` | Id | Elemento o `null` |
| `getElementsByTagName` | Nombre de etiqueta | Colección viva |
| `getElementsByClassName` | Clase | Colección viva |
| `getElementsByName` | Atributo `name` | Colección de elementos |

No dependas de variables globales creadas automáticamente a partir de `id`. Utiliza consultas explícitas para evitar conflictos con otros nombres:

```js
const searchForm = document.getElementById("search-form");
```

### Comprobar relaciones entre elementos

`matches` comprueba si un elemento coincide con un selector; `closest` busca desde el propio elemento hacia arriba; `contains` comprueba si un nodo está dentro de otro:

```js
const chapter = document.querySelector(".chapter");

if (chapter?.matches("li.chapter")) {
  console.log(chapter.closest(".book"));
}

const book = document.querySelector(".book");
console.log(book?.contains(chapter)); // true si chapter está dentro
```

---

## 5. Leer y modificar el contenido

### `textContent`: texto sin interpretar como HTML

`textContent` obtiene o reemplaza el texto de un elemento:

```js
const message = document.querySelector("#message");

message.textContent = "Hola, mundo";
console.log(message.textContent);
```

Es la opción adecuada para mostrar texto proporcionado por una persona usuaria. Las etiquetas se mostrarán literalmente y no se interpretarán como HTML:

```js
const userInput = '<img src=x onerror="alert(1)">';
message.textContent = userInput; // Se muestra como texto
```

### `innerHTML`: contenido HTML

`innerHTML` lee o reemplaza el contenido HTML interior:

```js
const notice = document.querySelector("#notice");
notice.innerHTML = "<strong>Importante:</strong> lee el aviso.";
```

No insertes datos no confiables con `innerHTML`: el navegador los interpretaría como marcado y podrían introducir contenido o código no deseado. Además, `innerHTML += ...` vuelve a analizar y reemplazar todo el contenido, no es una adición eficiente.

Para añadir elementos de forma segura, utiliza `createElement`, `textContent` y `append`:

```js
const item = document.createElement("li");
item.textContent = "Nuevo elemento";
document.querySelector("#items").append(item);
```

### `outerHTML`, nodos de texto y visibilidad

`outerHTML` representa el elemento completo y su contenido. Al asignarle un valor, el elemento se sustituye en el documento; la variable original sigue apuntando al nodo antiguo:

```js
const oldElement = document.querySelector("#status");
oldElement.outerHTML = "<p id='status'>Listo</p>";

const replacement = document.querySelector("#status");
console.log(oldElement === replacement); // false
```

Para leer o cambiar directamente un nodo de texto o comentario se puede utilizar su propiedad `data`:

```js
const textNode = document.querySelector("p").firstChild;
console.log(textNode.data);
```

La propiedad `hidden` muestra u oculta un elemento:

```js
const details = document.querySelector("#details");
details.hidden = true;
details.hidden = false;
```

### Propiedades habituales de elementos

Los elementos tienen propiedades específicas de su clase:

```js
const link = document.querySelector("a");
const input = document.querySelector("input");

console.log(link.href);
console.log(input.value);

input.value = "Nuevo valor";
```

La propiedad `style` permite modificar estilos en línea. Para cambios de presentación mantenibles suele ser mejor alternar clases:

```js
const panel = document.querySelector(".panel");
panel.classList.add("is-active");
panel.classList.remove("is-hidden");
panel.classList.toggle("is-expanded");
```

---

## 6. Resumen y práctica

| Necesidad | Propiedad o método |
| --- | --- |
| Obtener la raíz HTML, cabecera o cuerpo | `document.documentElement`, `document.head`, `document.body` |
| Buscar uno o varios elementos | `querySelector`, `querySelectorAll` |
| Recorrer todos los nodos hijos | `childNodes` |
| Recorrer solo elementos hijos | `children` |
| Leer o escribir texto de forma segura | `textContent` |
| Insertar HTML conocido y de confianza | `innerHTML` |
| Crear y añadir un elemento | `createElement`, `append` |
| Consultar o cambiar el estado visible | `hidden` |

### Tarea 1: buscar elementos

Dado este documento:

```html
<div>Usuarios:</div>
<ul id="users">
  <li>Ana</li>
  <li>Luis</li>
</ul>
```

1. Obtén el `div`.
2. Obtén la lista `ul`.
3. Obtén el segundo `li`.
4. Cambia el texto del segundo elemento sin reemplazar el resto del documento.

### Tarea 2: marcar la diagonal de una tabla

Obtén las celdas de una tabla y cambia el color de fondo de las celdas cuya fila y columna tengan el mismo índice. Comprueba los límites de la tabla antes de acceder a cada celda.
