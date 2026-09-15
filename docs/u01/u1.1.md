# UD01: Programación web en el servidor

---

## 1. Modelo cliente-servidor

El modelo cliente-servidor es la base de la web. Un **cliente** (navegador) solicita recursos a un **servidor**, que procesa la petición y devuelve una respuesta. La comunicación se realiza mediante el protocolo **HTTP** o su versión segura **HTTPS**, usando direcciones **URL**.

```mermaid
graph LR
  Cliente[Cliente] -->|HTTP / HTTPS| Servidor[Servidor]
  Servidor -->|Consulta| BaseDatos[(Base de datos)]
  Servidor -->|Respuesta| Cliente
```

- El **cliente** habitualmente un navegador como Chrome ![Chrome](https://api.iconify.design/logos:chrome.svg), Firefox ![Firefox](https://api.iconify.design/logos:firefox.svg) o Edge ![Edge](https://api.iconify.design/logos:microsoft-edge.svg), solicita recursos y muestra contenido, a partir de código HTML, CSS y JavaScript.
- El **servidor** procesa las peticiones y devuelve contenido, habitualmente HTML, JSON o archivos estáticos.

Pasos en la petición de una página web:

- El usuario introduce una URL o hace clic en un enlace.
- El navegador envía una petición HTTP al servidor.
- El servidor procesa la petición, consulta la base de datos si es necesario y genera una respuesta
- El navegador recibe la respuesta y renderiza la página para el usuario.

Cada interacción del usuario puede generar nuevas peticiones al servidor, que pueden ser síncronas (recargando la página) o asíncronas (actualizando parte del contenido sin recargar).

## 2. Aplicaciones web estáticas y dinámicas

### 2.1 Web estática

Una **web estática** sirve HTML, CSS y JavaScript desde el servidor sin modificar el contenido entre peticiones.

```mermaid
sequenceDiagram
  participant Usuario
  participant Navegador
  participant Servidor
  Usuario->>Navegador: solicita página
  Navegador->>Servidor: GET /index.html
  Servidor-->>Navegador: HTML/CSS/JS
  Navegador-->>Usuario: muestra página
```

Las páginas estáticas son rápidas y fáciles de servir, pero no cambian según el usuario o los datos. Solo varían si se actualiza el archivo en el servidor. Son adecuadas para sitios informativos, blogs simples o landing pages.

Ventajas y características principales:

- El contenido se almacena en archivos finales (.html, .css, .js, imágenes) y se sirve tal cual al cliente.
- No es necesario programar para crear contenidos básicos; basta con editar los archivos.
- Consumen menos recursos en el servidor y suelen ofrecer mejor rendimiento y SEO cuando el contenido no varía.
- Útiles para secciones que no requieren interacción ni datos dinámicos: contacto, términos, información estática.

Limitaciones:

- Imposible personalizar contenido por usuario o por contexto sin generar páginas adicionales.
- Actualización manual cuando cambia la información, lo que incrementa mantenimiento si hay mucho contenido.

## 2.2 Web dinámica

Una **web dinámica** construye contenido en el servidor en cada petición, usando datos de una base de datos o lógica del servidor.

```mermaid
sequenceDiagram
  participant Usuario
  participant Navegador
  participant Servidor
  participant BaseDatos
  Usuario->>Navegador: solicita página
  Navegador->>Servidor: GET /productos
  Servidor->>BaseDatos: consulta productos
  BaseDatos-->>Servidor: devuelve datos
  Servidor-->>Navegador: HTML/CSS/JS con datos
  Navegador-->>Usuario: muestra página
```

Las páginas dinámicas permiten personalización, interacción y actualización de datos en tiempo real. Son adecuadas para tiendas online, redes sociales o aplicaciones web complejas.

Características y funcionamiento:

- El servidor ejecuta código en lenguajes como PHP, Python, Java, Node.js, etc., y genera HTML al vuelo.
- El contenido puede depender de la hora, del usuario autenticado, de acciones previas o de consultas a bases de datos.
- Al recibir la petición el servidor analiza el archivo solicitado (por ejemplo index.php), ejecuta el código del lenguaje de servidor, accede a la base de datos o a otros recursos y construye el HTML que finalmente se envía al cliente.

Extensiones y ejemplos:

- Archivos típicos: .php, .py, .js (Node), .jsp, .asp.
- Adecuadas para comercios electrónicos, blogs con gestión de usuarios, paneles de administración (back-office) y aplicaciones con lógica de negocio.

Inconvenientes:

- Mayor complejidad de desarrollo y mayor consumo de recursos en el servidor.
- Requieren cuidado adicional para SEO y rendimiento (cache, optimización de consultas, etc.).

## 2.3 Comparación entre web estática y dinámica

| Característica | Web estática | Web dinámica |
| --- | --- | --- |
| Contenido | Fijo, no cambia entre peticiones | Generado al vuelo, puede variar según usuario, hora o datos |
| Lenguajes | HTML, CSS, JS | PHP, Python, Node.js, Java, etc. |
| Base de datos | No requiere | Requiere para almacenar y recuperar datos |
| Rendimiento | Rápida, menos carga en servidor | Más lenta, depende de la lógica y consultas |
| SEO | Fácil de optimizar | Requiere cuidado adicional (renderizado, metaetiquetas dinámicas) |
| Mantenimiento | Simple, editar archivos | Más complejo, requiere gestión de código y base de datos |
| Casos de uso | Blogs simples, landing pages, portafolios | Tiendas online, redes sociales, aplicaciones web interactivas |

## 3. Definiciones clave

- **HTML**: lenguaje de marcado que estructura una página.

  ```html
  <h1>Mi página</h1>
  <p>Bienvenido al sitio.</p>
  ```

- **CSS**: hojas de estilo que definen el aspecto visual.

  ```css
  body {
    font-family: Arial, sans-serif;
    background: #f9f9f9;
  }
  ```

- **JavaScript**: lenguaje que añade comportamiento e interactividad en el navegador.

  ```js
  document.querySelector('button').addEventListener('click', () => {
    alert('¡Has pulsado el botón!');
  });
  ```

- **DOM**: representación en memoria del árbol de elementos HTML.

  ```js
  const titulo = document.getElementById('titulo');
  titulo.textContent = 'Nuevo título';
  ```

- **Framework**: conjunto de herramientas y librerías que facilitan el desarrollo de aplicaciones web, proporcionando estructura y funcionalidades predefinidas.

- **Frontend**: parte de la aplicación que se ejecuta en el cliente (navegador) y que interactúa con el usuario. Consiste en HTML, CSS y JavaScript, y puede usar frameworks como React, Vue o Angular.

  ```jsx
  import React, { useState } from 'react';

  function PulsaBoton() {
    const [count, setCount] = useState(0);
    return (
      <button onClick={() => setCount(count + 1)}>
        Pulsado {count} {count === 1 ? 'vez' : 'veces'}
      </button>
    );
  }

  export default PulsaBoton;
  ```

![Frontend](./frontend.png)

- **Backend**: parte de la aplicación que se ejecuta en el servidor y que procesa datos, lógica y almacenamiento. Puede estar implementado en PHP, Python, Node.js, Java, etc., y suele usar frameworks como Laravel, Django, Express o Spring.

  ```js
  // Node.js / Express
  app.get('/api/mensaje', (req, res) => {
    res.json({ mensaje: 'Hola desde el servidor' });
  });
  ```

- **Full Stack**: desarrollador o aplicación que abarca tanto frontend como backend.

- **Backoffice**: interfaz de administración de la aplicación, generalmente accesible solo para usuarios autorizados.

![Backoffice](./backoffice.png)

- **HTTP**: protocolo de comunicación entre cliente y servidor. Utiliza métodos para indicar la acción deseada y códigos de estado para informar del resultado.
  Los métodos HTTP más utilizados son:

    - **GET**: solicitar un recurso. Seguro y sin efecto secundario (idempotente cuando no altera estado).
    - **POST**: enviar datos al servidor para crear un recurso o procesar información (no idempotente).
    - **PUT**: reemplazar o crear un recurso en una ubicación concreta (idempotente).
    - **PATCH**: aplicar modificaciones parciales a un recurso (no necesariamente idempotente).
    - **DELETE**: eliminar un recurso (idempotente en la práctica cuando el recurso desaparece).

  Los rangos de códigos de estado son:

    - 1xx Informativos: indican comunicación en progreso (100 Continue, ...).
    - 2xx Éxito: la petición se completó correctamente (200 OK, 201 Created, 204 No Content, ...).
    - 3xx Redirecciones: se requiere acción adicional para completar la petición (301 Moved Permanently, 302 Found, ...).
    - 4xx Errores del cliente: la petición es incorrecta o no autorizada (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, ...).
    - 5xx Errores del servidor: fallo en el servidor al procesar la petición (500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, ...).

  Las respuestas incluyen un código de estado que indica el resultado y, opcionalmente, un cuerpo con más información.

- **JSON**: formato de datos ligero usado para intercambiar información, consistente en pares clave-valor.

  ```json
  {
    "productos": [
      {
        "producto": "Camiseta",
        "precio": 19.99
      },
      {
        "producto": "Pantalones",
        "precio": 39.99
      }
    ]
  }
  ```

- **API REST**: interfaz que permite al Frontend comunicarse con el Backend para intercambiar recursos en formato JSON mediante el protocolo HTTP.

  Ejemplo de petición:
  
  ```http
  GET /api/productos HTTP/1.1
  Host: ejemplo.com
  ```

  Ejemplo de respuesta:

  ```http
  HTTP/1.1 200 OK
  Content-Type: application/json; charset=utf-8
  Content-Length: 48

  {
    "productos": [
      { "id": 1, "nombre": "Camiseta", "precio": 19.99 },
      { "id": 2, "nombre": "Pantalones", "precio": 39.99 }
    ]
  }
  ```

---

## 4. Comunicación síncrona vs. asíncrona y AJAX

En la web tradicional, la comunicación entre cliente y servidor es **síncrona**. Cuando el usuario realiza una acción (por ejemplo, enviar un formulario o hacer clic en un enlace), el navegador bloquea la interfaz, realiza la petición y espera a que el servidor devuelva un documento HTML completo para recargar toda la página.

```mermaid
sequenceDiagram
  participant Usuario
  participant Navegador
  participant Servidor
  Usuario->>Navegador: clic en enlace
  Navegador->>Servidor: GET /ruta
  Servidor-->>Navegador: HTML completo
  Navegador-->>Usuario: muestra página
```

Para evitar estas recargas completas y ofrecer una experiencia más fluida, surge **AJAX** (*Asynchronous JavaScript and XML*). AJAX permite hacer peticiones asíncronas desde JavaScript, de forma que la página puede actualizar parte de su contenido sin recargar todo el documento.

```mermaid
sequenceDiagram
  participant Usuario
  participant Navegador
  participant Servidor
  Usuario->>Navegador: pulsa botón
  Navegador->>Servidor: AJAX /api/datos
  Servidor-->>Navegador: JSON
  Navegador-->>Usuario: actualiza vista
```

Algunos ejemplos típicos de AJAX incluyen formularios que se envían sin recargar la página, actualizaciones de contenido en tiempo real (como botones "me gusta" o notificaciones), autocompletado en buscadores y carga de datos adicionales al hacer scroll (*infinite scroll*).

Entre las ventajas de AJAX destacan:

- Mejor experiencia de usuario (UX), ya que la interfaz no se bloquea.
- Reducción de ancho de banda, al no recargar recursos estáticos.
- Permite crear aplicaciones web más interactivas y dinámicas.

Para que una comunicación asíncrona AJAX funcione eficientemente, el cliente y el servidor necesitan un "idioma común" para intercambiar datos estructurados. Para ello, se utiliza **JSON** con interfaces **API REST**, donde el cliente hace peticiones HTTP a rutas específicas y el servidor responde con datos en formato JSON.

```mermaid
graph LR
    Cliente[Cliente / Frontend] -->|Petición HTTP: GET /api/productos| Server[Servidor / Backend]
    Server -->|Consulta| DB[(Base de Datos)]
    DB -->|Retorna filas| Server
    Server -->|Respuesta HTTP 200 OK + Payload JSON| Cliente
```

---

## 5. Modelos de arquitectura de renderizado

La forma en que combinamos la generación de HTML (servidor vs. cliente) y el tipo de comunicación (síncrona vs. asíncrona) da lugar a dos grandes filosofías de desarrollo web:

```mermaid
graph TD
    subgraph MPA [MPA - Multi-Page Application]
        M1[Navegación tradicional] --> M2[El Servidor genera todo el HTML]
        M2 --> M3[Recarga completa en cada clic]
    end
```

```mermaid
graph TD
    subgraph SPA [SPA - Single-Page Application]
        S1[Carga inicial de un único HTML y JS] --> S2[Navegación interna por JS]
        S2 --> S3[Consumo de API REST vía AJAX]
        S3 --> S4[Modificación dinámica del DOM]
    end
```

Vídeo recomendado:

<div align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/2z0FChkphvo" title="Aplicaciones SPA vs MPA ¿Qué son y cual elegir?" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>

<!-- ![Aplicaciones SPA vs MPA ¿Qué son y cual elegir?](https://www.youtube.com/embed/2z0FChkphvo){ width="560" height="315" } -->

### 5.1 MPA (Multi-Page Application)

Es el modelo clásico del desarrollo web. Cada vez que el usuario navega a una nueva sección, el servidor procesa la petición, consulta la base de datos y renderiza en el servidor (SSR, Server Side Rendering) una nueva página HTML completa.

```mermaid
sequenceDiagram
    participant Usuario
    participant Navegador
    participant Servidor as Servidor (MPA)
    participant BaseDatos as Base de datos

    Usuario->>Navegador: accede a una sección
    Navegador->>Servidor: solicita /productos
    Servidor->>BaseDatos: consulta los datos
    BaseDatos-->>Servidor: devuelve resultados
    Servidor->>Servidor: renderiza la vista HTML
    Servidor-->>Navegador: envía la página completa
    Navegador-->>Usuario: muestra la nueva vista
```

Algunas de las tecnologías backend que se usan en los servidores MPA son:

| Framework servidor | Lenguaje |
| --- | --- |
| Laravel ![Laravel](https://api.iconify.design/logos:laravel.svg) | PHP ![PHP](https://api.iconify.design/logos:php.svg) |
| Spring ![Spring](https://api.iconify.design/logos:spring.svg) | Java ![Java](https://api.iconify.design/logos:java.svg) |
| Django ![Django](https://api.iconify.design/logos:django.svg) | Python ![Python](https://api.iconify.design/logos:python.svg) |
| Express ![Express](https://api.iconify.design/logos:express.svg) | Node.js ![Node.js](https://api.iconify.design/logos:nodejs.svg) |
| Ruby on Rails ![Ruby on Rails](https://api.iconify.design/logos:rails.svg) | Ruby ![Ruby](https://api.iconify.design/logos:ruby.svg) |
| ASP.NET ![ASP.NET](https://api.iconify.design/logos:dotnet.svg) | .NET ![.NET](https://api.iconify.design/logos:dotnet.svg) |

Entre las características de MPA destacan:

- El servidor genera el HTML de cada página.
- El navegador recibe contenido listo para mostrar.
- La navegación recarga el documento completo.
- Es adecuado para sitios informativos y aplicaciones con formularios tradicionales.

Ejemplo de una ruta MPA con Laravel. El servidor consulta los productos y devuelve una vista HTML completa en cada petición:

```php
// routes/web.php
use App\Models\Producto;
use Illuminate\Support\Facades\Route;

Route::get('/productos', function () {
  return view('productos', ['productos' => Producto::all()]);
});
```

```blade
{{-- resources/views/productos.blade.php --}}
<h1>Productos</h1>
<ul>
  @foreach ($productos as $producto)
    <li>{{ $producto->nombre }} - {{ $producto->precio }} €</li>
  @endforeach
</ul>
```

### 5.2 SPA (Single Page Application)

Es una aplicación web de una sola página. El servidor entrega un HTML sin datos inicial junto con un paquete de JavaScript (usando frameworks como React, Vue o Angular). A partir de ahí, el navegador mantiene la página activa: cuando el usuario navega, JavaScript simula el cambio de página y pide datos al backend mediante AJAX a una API REST, modificando el DOM al vuelo.

En una SPA, el navegador carga una sola página inicial y el cliente maneja el enrutado y las vistas.

```mermaid
sequenceDiagram
    participant Usuario
    participant Frontend as Frontend (SPA)
    participant Backend as Backend (API REST)
    participant BaseDatos as Base de datos

    Usuario->>Frontend: accede a la aplicación
    Frontend->>Frontend: carga HTML, CSS y JS iniciales
    Frontend->>Backend: solicita /api/productos
    Backend->>BaseDatos: consulta los datos
    BaseDatos-->>Backend: devuelve resultados
    Backend-->>Frontend: responde JSON
    Frontend->>Frontend: renderiza la interfaz
    Frontend-->>Usuario: muestra la página actualizada
```

Algunas de las tecnologías frontend que se basan en el funcionamiento SPA son:

| Framework cliente | Lenguaje |
| --- | --- |
| React ![React](https://api.iconify.design/logos:react.svg) | JavaScript ![JavaScript](https://api.iconify.design/logos:javascript.svg) |
| Vue ![Vue](https://api.iconify.design/logos:vue.svg) | JavaScript ![JavaScript](https://api.iconify.design/logos:javascript.svg) |
| Angular ![Angular](https://api.iconify.design/logos:angular-icon.svg) | TypeScript ![TypeScript](https://api.iconify.design/logos:typescript-icon.svg) |

Entre las características de SPA destacan:

- El servidor entrega un HTML base y scripts.
- El cliente usa JavaScript y AJAX para actualizar contenido.
- La primera carga puede ser más pesada.
- Buena para aplicaciones con mucha interacción y estado en el cliente.

Ejemplo sencillo de una SPA con Node.js. Node.js sirve el HTML inicial y expone una API; después, JavaScript actualiza el DOM sin recargar la página:

```js
// server.js
const express = require('express');
const app = express();

app.use(express.static('public'));

app.get('/api/productos', (req, res) => {
  res.json([
    { nombre: 'Teclado', precio: 29.99 },
    { nombre: 'Ratón', precio: 14.99 }
  ]);
});

app.listen(3000, () => console.log('http://localhost:3000'));
```

```html
<!-- public/index.html -->
<h1>Productos</h1>
<ul id="productos"></ul>
<script>
  fetch('/api/productos')
    .then(respuesta => respuesta.json())
    .then(productos => {
      document.querySelector('#productos').innerHTML = productos
        .map(producto => `<li>${producto.nombre} - ${producto.precio} €</li>`)
        .join('');
    });
</script>
```

### Comparación de modelos

| Criterio | MPA (Multi-Page Application) | SPA (Single-Page Application) |
| --- | --- | --- |
| Generación de HTML | En el servidor (Server-Side Rendering) | En el cliente mediante JavaScript |
| Navegación | Recarga de página completa | Transición fluida sin recarga de navegador |
| Manejo de Estado | El servidor gestiona el estado (Sesiones) | El cliente guarda el estado en memoria |
| Posicionamiento SEO | Excelente de forma nativa | Requiere configuraciones adicionales |
| Complejidad de desarrollo | Menor (ideal para la base del módulo) | Mayor (requiere separar Frontend y Backend) |
| Casos de uso ideales | Sitios corporativos, blogs, e-commerce, paneles de gestión | Redes sociales, plataformas SaaS, dashboards muy interactivos |

---

## 6. Arquitecturas backend

Existen distintas arquitecturas del backend que definen cómo se distribuyen y ejecutan los componentes del sistema en los servidores.

```mermaid
graph TD
    subgraph Monolith [1. Arquitectura Monolítica]
        M1[Todos los módulos empaquetados juntos] --> M2[(Única Base de Datos)]
    end

    subgraph Microservices [2. Arquitectura de Microservicios]
        MS1[Servicio Usuarios] --> DB1[(BD 1)]
        MS2[Servicio Catálogo] --> DB2[(BD 2)]
        MS3[Servicio Pagos] --> DB3[(BD 3)]
    end

    subgraph Serverless [3. Arquitectura Serverless / FaaS]
        API[API Gateway Cloud] --> F1[Función Lambda 1]
        API --> F2[Función Lambda 2]
    end
```

### 6.1 Arquitectura Monolítica (Monolito)

Todos los componentes de la aplicación (gestión de usuarios, catálogo, cobros, vistas HTML...) se compilan, empaquetan y despliegan como una sola unidad de software.

Características:

- Todos los módulos internos comparten la misma base de datos.
- Si se modifica una sola línea de código, hay que volver a desplegar la aplicación completa.

Ejemplo: una e-commerce donde la gestión de productos, el carrito de la compra y la pasarela de pagos residen en el mismo repositorio y se ejecutan en un único servidor Apache/Nginx.

Pros y Contras:

- ✅: Muy fácil de desarrollar, probar y desplegar en proyectos pequeños o medianos; menor complejidad operativa.
- ❌: Si el código crece mucho, puede volverse difícil de mantener; escalar requiere duplicar todo el bloque aunque solo una parte tenga mucho tráfico.

### 6.2 Arquitectura de Microservicios

La aplicación se divide en un conjunto de pequeños servicios independientes y desacoplados. Cada microservicio se encarga de un único dominio de negocio y se comunica con los demás a través de la red (mediante API REST).

Características:

- Autonomía: Cada microservicio se puede desplegar, actualizar y escalar de forma totalmente independiente.
- Base de Datos propia (Database per Service): Un servicio no puede acceder directamente a la BD de otro.
- Políglota: Se pueden usar distintos lenguajes o bases de datos según las necesidades de cada microservicio (ej. Node.js para un chat en tiempo real y Python para recomendaciones).

Ejemplo: una plataforma más compleja como Netflix o Amazon, que necesita de servicios independientes para la gestión del motor de recomendaciones, facturación, autenticación y catálogo de vídeos.

Pros y Contras:

- ✅: Excelente escalabilidad horizontal; gran tolerancia a fallos (si cae un servicio, el resto sigue funcionando).
- ❌: Muy alta complejidad operativa (requiere orquestadores como Kubernetes ![Kubernetes](https://api.iconify.design/logos:kubernetes.svg), Docker ![Docker](https://api.iconify.design/logos:docker.svg) y monitorización avanzada); latencia añadida por las llamadas en red.

### 6.3 Arquitectura Serverless (Sin Servidor)

Modelo de ejecución en la nube (Backend as a Service - BaaS) donde el equipo de desarrollo no gestiona ni aprovisiona servidores. El código se escribe en forma de funciones efímeras que el proveedor cloud (AWS ![AWS](https://api.iconify.design/logos:aws.svg), Google Cloud ![Google Cloud](https://api.iconify.design/logos:google-cloud.svg), Azure ![Azure](https://api.iconify.design/logos:azure.svg)) ejecuta únicamente cuando ocurre un evento específico (ej. una petición HTTP a un endpoint de una API REST).

Características:

- Escalado de cero a infinito: Si no hay peticiones, no hay código ejecutándose. Si entran miles de peticiones simultáneas, el proveedor lanza miles de instancias en milisegundos.
- Pago por uso exacto: Se factura únicamente por el tiempo exacto de ejecución del código en el procesador del servidor (medido en milisegundos).
- Mantenimiento cero: No hay que preocuparse por actualizaciones de sistema operativo, parches de seguridad ni reinicios del servidor, porque el proveedor cloud se encarga de todo.

Ejemplo: Una función en AWS Lambda que se activa únicamente cuando un usuario sube una imagen para redimensionarla y guardarla en la nube.

Pros y Contras:

- ✅: Cero mantenimiento de infraestructura; costes mínimos para aplicaciones con tráfico intermitente.
- ❌: Dependencia directa del proveedor en la nube (Vendor Lock-in).

## 7. Patrón MVC (Modelo-Vista-Controlador)

MVC es un patrón de diseño de software que organiza el código en tres componentes principales, separando la lógica de negocio (Modelo), la presentación (Vista) y el control de flujo (Controlador).

- **Modelo**: representa los datos y el acceso a la base de datos.
- **Vista**: genera la plantilla HTML que ve el usuario.
- **Controlador**: recibe la petición, pide datos al modelo y devuelve la vista.

Actualmente, el patrón MVC es el estándar más utilizado para organizar el código en aplicaciones **monolíticas** (como las creadas con Laravel, Django o Spring Boot) o para organizar la capa interna de un **microservicio**.

MVC mejora el mantenimiento y facilita el trabajo colaborativo.

```mermaid
graph TD
    Client([1. Cliente / Navegador]) -->|2. Petición HTTP| Ctrl[Controlador]
    
    subgraph Aplicación MVC
        Ctrl -->|3. Solicita / Modifica datos| Mod[Modelo]
        Mod <-->|4. Consultas SQL| DB[(Base de Datos)]
        Mod -->|5. Retorna Datos / Objetos| Ctrl
        Ctrl -->|6. Envía datos a la plantilla| Vis[Vista]
    end
    
    Vis -->|7. Devuelve HTML renderizado| Client
```

### Modelo (Model)

Gestiona los datos, las reglas de negocio y la comunicación con la Base de Datos.

En Laravel, se representa por clases de Eloquent ORM (ej. Product.php, User.php).

```php
// app/Models/Product.php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
  protected $fillable = ['name', 'price'];
}
```

### Vista (View)

Es la capa de presentación. Se encarga de maquetar y formatear la información que se mostrará al usuario final.

En Laravel, se utilizan plantillas Blade (ej. index.blade.php).

```blade
{{-- resources/views/products/index.blade.php --}}
<h1>Productos</h1>

<ul>
  @foreach ($products as $product)
    <li>{{ $product->name }} — {{ $product->price }} €</li>
  @endforeach
</ul>
```

### Controlador (Controller)

Es el intermediario/orquestador. Recibe las peticiones HTTP del cliente, le pide al Modelo los datos necesarios, aplica la lógica oportuna y le entrega los datos a la Vista para generar la respuesta.

En Laravel, se utilizan clases de tipo Controller (ej. ProductController.php).

```php
// app/Http/Controllers/ProductController.php
namespace App\Http\Controllers;

use App\Models\Product;

class ProductController extends Controller
{
  public function index()
  {
    $products = Product::all();

    return view('products.index', compact('products'));
  }
}
```

## 8. Entorno de desarrollo

En UD01 el entorno preferido es:

- **Backend**: PHP + Laravel
- **Servidor web**: Apache
- **Base de datos**: MySQL/MariaDB o SQLite en desarrollo
- **Editor**: VSCode

```mermaid
graph LR
  VSCode --> PHP[PHP / Laravel]
  PHP --> Apache[Apache]
  Apache --> DB[(Base de datos)]
  VSCode --> Git[Control de versiones]
```

Laravel es un framework MVC que encaja bien con el enfoque de este módulo.

---

## 9. Conclusión

Esta propuesta simplifica los modelos de renderizado a los tres más relevantes para UD01 y da importancia al patrón MVC en las arquitecturas backend. El objetivo es ofrecer una base clara para distinguir entre aplicaciones estáticas y dinámicas, entender cuándo usar cada modelo y cómo organizar el backend con Laravel y VSCode.
