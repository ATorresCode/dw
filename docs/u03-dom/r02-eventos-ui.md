# UD03: Eventos de la interfaz

## 1. Eventos y manejadores

Un **evento** es una señal de que ha ocurrido algo, como un clic, la pulsación de una tecla o el envío de un formulario. El navegador crea un objeto de evento con información y lo pasa al manejador (*handler*).

Algunos eventos habituales:

| Grupo | Eventos |
| --- | --- |
| Ratón y puntero | `click`, `contextmenu`, `mousedown`, `mouseup`, `mousemove`, `mouseover`, `mouseout` |
| Teclado | `keydown`, `keyup` |
| Formularios | `submit`, `input`, `change`, `focus`, `blur` |
| Documento | `DOMContentLoaded` |
| CSS | `transitionend` |

### Formas de registrar un manejador

Los atributos HTML como `onclick` permiten asociar código en el marcado, pero mezclan comportamiento y contenido. Para código nuevo es preferible registrar los manejadores desde JavaScript:

```html
<button id="greet">Saludar</button>
```

```js
const button = document.querySelector("#greet");

button.onclick = () => {
  console.log("¡Hola!");
};
```

Una propiedad `on...` admite un único manejador; asignar otro reemplaza al anterior. Se puede quitar asignando `null`.

La forma recomendada habitualmente es `addEventListener`, que permite varios manejadores:

```js
function showMessage() {
  console.log("Botón pulsado");
}

button.addEventListener("click", showMessage);
button.addEventListener("click", () => console.log("Otro manejador"));

// Para quitar el manejador, se necesita la misma referencia a la función.
button.removeEventListener("click", showMessage);
```

No invoques la función al registrarla. `button.addEventListener("click", showMessage)` pasa la función; `button.addEventListener("click", showMessage())` la ejecuta inmediatamente.

Opciones útiles:

```js
button.addEventListener("click", showMessage, { once: true });
```

- `once: true` elimina el manejador después de la primera ejecución.
- `capture: true` registra el manejador en la fase de captura.
- `passive: true` indica que no se llamará a `preventDefault()`, útil para ciertos eventos de desplazamiento.

También se puede usar un objeto como manejador si implementa `handleEvent(event)`:

```js
const logger = {
  handleEvent(event) {
    console.log(event.type, event.currentTarget);
  },
};

button.addEventListener("click", logger);
button.removeEventListener("click", logger);
```

---

## 2. El objeto de evento

El objeto recibido contiene información sobre el evento:

```js
button.addEventListener("click", event => {
  console.log(event.type);
  console.log(event.target);
  console.log(event.currentTarget);
});
```

- `event.target`: elemento donde se originó el evento.
- `event.currentTarget`: elemento cuyo manejador se está ejecutando.
- `event.type`: nombre del evento.
- `event.preventDefault()`: cancela la acción predeterminada si es cancelable.
- `event.stopPropagation()`: evita que el evento continúe propagándose a otros elementos.

`target` y `currentTarget` pueden ser distintos: el objetivo puede ser un elemento interior de un botón o contenedor.

---

## 3. Propagación, captura y delegación

Algunos eventos se propagan por el árbol DOM. Simplificando, un evento pasa por estas fases:

1. **Captura**: desde los ancestros hacia el objetivo.
2. **Objetivo**: llega al elemento que lo originó.
3. **Propagación**: desde el objetivo de vuelta hacia los ancestros.

Los manejadores registrados con `addEventListener` se ejecutan en la fase de propagación por defecto; se puede solicitar la fase de captura con `capture: true`.

```js
document.querySelector("#container").addEventListener(
  "click",
  event => console.log("Captura"),
  { capture: true }
);

document.querySelector("#container").addEventListener("click", event => {
  console.log("Propagación");
});
```

`stopPropagation()` detiene el recorrido del evento. No lo uses sin necesidad: puede impedir que otros componentes respondan. Para cancelar la acción predeterminada, como navegar al pulsar un enlace, se usa `preventDefault()`; son operaciones distintas.

### Delegación de eventos

La propagación permite escuchar un evento en un contenedor para gestionar muchos elementos descendientes. Así se evita registrar un manejador por cada elemento:

```html
<ul id="tasks">
  <li><button data-action="remove">Quitar tarea</button></li>
  <li><button data-action="remove">Quitar tarea</button></li>
</ul>
```

```js
const tasks = document.querySelector("#tasks");

tasks.addEventListener("click", event => {
  if (!(event.target instanceof Element)) {
    return;
  }

  const removeButton = event.target.closest('[data-action="remove"]');

  if (!removeButton || !tasks.contains(removeButton)) {
    return;
  }

  removeButton.closest("li")?.remove();
});
```

`mouseover` y `mouseout` se propagan y son útiles para delegar. `mouseenter` y `mouseleave` no se propagan, y no notifican transiciones entre un elemento y sus descendientes.

---

## 4. Eventos del ratón

Una interacción de clic suele producir `mousedown`, `mouseup` y `click`. `dblclick` representa un doble clic; `contextmenu` suele corresponder al menú contextual.

En eventos de ratón:

- `event.button` identifica el botón usado (`0`: principal, `1`: auxiliar, `2`: secundario).
- `event.altKey`, `event.ctrlKey`, `event.shiftKey` y `event.metaKey` indican si había teclas modificadoras pulsadas.
- `clientX` / `clientY` son coordenadas relativas a la ventana.
- `pageX` / `pageY` son coordenadas relativas al documento, incluyendo el desplazamiento.

```js
document.addEventListener("click", event => {
  console.log({
    x: event.clientX,
    y: event.clientY,
    pageX: event.pageX,
    pageY: event.pageY,
    shift: event.shiftKey,
  });
});
```

### `mouseover` y `mouseout`

Estos eventos pueden activarse al pasar de un elemento a un descendiente. `event.relatedTarget` indica el elemento desde el que se llega o al que se va, respectivamente.

Si solo interesa entrar o salir del límite completo de un elemento, usa `mouseenter` y `mouseleave`. Para delegar eventos en muchos descendientes, utiliza `mouseover`/`mouseout` y filtra los movimientos que ocurren dentro del mismo descendiente.

### Resaltar celdas con delegación

```js
const table = document.querySelector("#schedule");

table.addEventListener("mouseover", event => {
  if (!(event.target instanceof Element)) {
    return;
  }

  const cell = event.target.closest("td");
  if (!cell || !table.contains(cell)) {
    return;
  }

  if (event.relatedTarget instanceof Node && cell.contains(event.relatedTarget)) {
    return;
  }

  cell.classList.add("is-highlighted");
});

table.addEventListener("mouseout", event => {
  if (!(event.target instanceof Element)) {
    return;
  }

  const cell = event.target.closest("td");
  if (!cell || !table.contains(cell)) {
    return;
  }

  if (event.relatedTarget instanceof Node && cell.contains(event.relatedTarget)) {
    return;
  }

  cell.classList.remove("is-highlighted");
});
```

La comprobación de `relatedTarget` evita tratar el movimiento entre los elementos interiores de una celda como una salida o entrada de la propia celda.

---

## 5. Arrastrar y soltar con el ratón

Para un elemento arrastrable, el algoritmo básico es:

1. En `mousedown`, guardar la posición inicial del puntero respecto al elemento.
2. En `mousemove`, actualizar la posición del elemento.
3. En `mouseup`, detener el movimiento y limpiar los manejadores temporales.

Este ejemplo básico usa coordenadas de ventana y escucha el movimiento en `document` para no perderlo si el puntero sale del elemento:

```js
const box = document.querySelector("#draggable");

box.addEventListener("mousedown", event => {
  if (event.button !== 0) {
    return;
  }

  const rect = box.getBoundingClientRect();
  const shiftX = event.clientX - rect.left;
  const shiftY = event.clientY - rect.top;

  box.style.position = "fixed";
  box.style.left = `${rect.left}px`;
  box.style.top = `${rect.top}px`;
  box.style.zIndex = "10";

  function move(moveEvent) {
    box.style.left = `${moveEvent.clientX - shiftX}px`;
    box.style.top = `${moveEvent.clientY - shiftY}px`;
  }

  function stop() {
    document.removeEventListener("mousemove", move);
    document.removeEventListener("mouseup", stop);
  }

  document.addEventListener("mousemove", move);
  document.addEventListener("mouseup", stop, { once: true });
  event.preventDefault();
});

box.addEventListener("dragstart", event => event.preventDefault());
```

`getBoundingClientRect()` y `clientX`/`clientY` usan coordenadas relativas a la ventana, por eso las unidades coinciden. En producción, valora la API moderna de eventos de puntero (`pointerdown`, `pointermove`, `pointerup`) para soportar también pantallas táctiles y lápices.

Para detectar un elemento situado debajo de otro que se está arrastrando, puede utilizarse `document.elementFromPoint(clientX, clientY)`. Hay que ocultar temporalmente el elemento arrastrado para que no se detecte a sí mismo:

```js
box.hidden = true;
const underPointer = document.elementFromPoint(event.clientX, event.clientY);
box.hidden = false;
```

---

## 6. Eventos de teclado y acciones predeterminadas

`keydown` se produce al pulsar una tecla y `keyup` al soltarla. Si se mantiene pulsada, `keydown` puede repetirse; `event.repeat` indica esas repeticiones.

- `event.key` representa el carácter o función de la tecla y respeta la distribución del teclado.
- `event.code` representa la ubicación física de la tecla y es útil para atajos que dependen de la posición.

```js
document.addEventListener("keydown", event => {
  if ((event.ctrlKey || event.metaKey) && event.code === "KeyS") {
    event.preventDefault();
    console.log("Guardar documento");
  }
});
```

No uses los antiguos `keypress`, `keyCode`, `charCode` o `which` en código nuevo.

Para saber si cambia el contenido de un campo, prefiere `input` o `change` a los eventos de teclado: el valor puede cambiar al pegar, arrastrar, usar dictado o completar automáticamente.

```js
const nameInput = document.querySelector("#name");

nameInput.addEventListener("input", () => {
  console.log(nameInput.value);
});
```

Se puede impedir una acción predeterminada con `preventDefault()`, pero normalmente es mejor utilizar validación HTML y el evento adecuado:

```js
const form = document.querySelector("#phone-form");

form.elements.phone.addEventListener("input", () => {
  form.elements.phone.setCustomValidity("");
});

form.addEventListener("submit", event => {
  const phone = form.elements.phone.value;

  if (!/^[0-9+() -]+$/.test(phone)) {
    event.preventDefault();
    form.elements.phone.setCustomValidity("Introduce un teléfono válido.");
    form.reportValidity();
  } else {
    form.elements.phone.setCustomValidity("");
  }
});
```

```html
<form id="phone-form">
  <label>
    Teléfono
    <input name="phone" type="tel" required>
  </label>
  <button>Enviar</button>
</form>
```

No filtres caracteres exclusivamente con `keydown`: impediría teclas de edición y no cubriría todos los métodos de entrada. Valida el valor final.

---

## 7. Formularios y ciclo de vida

El evento `submit` se produce al enviar un formulario. `preventDefault()` impide la navegación tradicional cuando se quiere procesar el formulario con JavaScript:

```js
const loginForm = document.querySelector("#login");

loginForm.addEventListener("submit", event => {
  event.preventDefault();

  const formData = new FormData(loginForm);
  console.log(formData.get("email"));
});
```

`DOMContentLoaded` se dispara cuando el HTML se ha analizado y el DOM está disponible:

```js
document.addEventListener("DOMContentLoaded", () => {
  document.querySelector("#status").textContent = "Listo";
});
```

Si el script se carga con `defer` o es un módulo, normalmente puede consultar el DOM directamente al ejecutarse.

---

## 8. Resumen y práctica

| Necesidad | Solución |
| --- | --- |
| Registrar uno o más manejadores | `addEventListener` |
| Retirar un manejador | `removeEventListener` con la misma función |
| Saber dónde se originó el evento | `event.target` |
| Saber qué elemento ejecuta el manejador | `event.currentTarget` |
| Cancelar la acción predeterminada | `event.preventDefault()` |
| Gestionar muchos elementos descendientes | Delegación de eventos |
| Detectar cualquier cambio en un campo | Evento `input` |
| Detectar una pulsación o atajo | `keydown` / `keyup` |

### Tarea 1: ocultar un mensaje

Crea un botón que oculte un bloque al pulsarlo. Añade otro botón para volver a mostrarlo. Utiliza `hidden` o una clase CSS.

### Tarea 2: lista seleccionable

Usa delegación de eventos en una lista para alternar una clase en el elemento pulsado. Permite seleccionar varios elementos y evita que un clic en un botón interior cambie la selección.

### Tarea 3: atajo de teclado

Implementa `runOnKeys(callback, ...codes)` para ejecutar una función al mantener pulsadas simultáneamente las teclas indicadas. Limpia el estado al recibir `keyup` y ten en cuenta `event.code`.
