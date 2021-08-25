---
"$title": Utilice AMP como fuente de datos para su PWA
"$order": '1'
description: Si ha invertido en AMP pero aún no ha creado una aplicación web progresiva, sus páginas AMP pueden simplificar drásticamente el desarrollo de su aplicación web progresiva.
formats:
  - sitios web
author: pbakaus
---

Si ha invertido en AMP pero aún no ha creado una aplicación web progresiva, sus páginas AMP pueden simplificar drásticamente el desarrollo de su aplicación web progresiva. En esta guía, aprenderá cómo consumir AMP dentro de su aplicación web progresiva y cómo utilizar sus páginas AMP existentes como fuente de datos.

## De JSON a AMP

En el escenario más común, una aplicación web progresiva es una aplicación de una sola página que se conecta a una API JSON a través de Ajax. Esta API JSON luego devuelve conjuntos de datos para impulsar la navegación y el contenido real para representar los artículos.

Segmento de prueba 1

Luego procedería y convertiría el contenido sin procesar en HTML utilizable y lo renderizaría en el cliente. Este proceso es costoso y, a menudo, difícil de mantener. En su lugar, puede reutilizar sus páginas AMP ya existentes como fuente de contenido. Lo mejor de todo es que AMP hace que sea trivial hacerlo con solo unas pocas líneas de código.

Segmento de prueba 2

## Incluya "Shadow AMP" en su aplicación web progresiva

El primer paso es incluir una versión especial de AMP que llamamos "Shadow AMP" en su aplicación web progresiva. Sí, es correcto: carga la biblioteca AMP en la página de nivel superior, pero en realidad no controlará el contenido de nivel superior. Solo "amplificará" las partes de nuestra página a las que le indique.

Incluya Shadow AMP en el encabezado de su página, así:

[sourcecode:html]
<!-- Asynchronously load the AMP-with-Shadow-DOM runtime library. -->
<script async src="https://cdn.ampproject.org/shadow-v0.js"></script>
[/sourcecode]

### ¿Cómo saber cuándo está lista para usar la API Shadow AMP?

Le recomendamos que cargue la biblioteca Shadow AMP con el `async` en su lugar. Eso significa, sin embargo, que debe usar un cierto enfoque para comprender cuándo la biblioteca está completamente cargada y lista para usarse.

La señal correcta para observar es la disponibilidad de la `AMP` global, y Shadow AMP utiliza un " [enfoque de carga de función asíncrona](http://mrcoles.com/blog/google-analytics-asynchronous-tracking-how-it-work/) " para ayudar con eso. Considere este código:

[sourcecode:javascript]
(window.AMP = window.AMP || []).push(function(AMP) {
  // AMP is now available.
});
[/sourcecode]

Este código funcionará, y cualquier cantidad de devoluciones de llamada agregadas de esta manera se activará cuando AMP esté disponible, pero ¿por qué?

Este código se traduce en:

1. "Si window.AMP no existe, crea una matriz vacía para tomar su posición"
2. "luego inserta una función de devolución de llamada en la matriz que debe ejecutarse cuando AMP esté listo"

Funciona porque la biblioteca Shadow AMP, tras la carga real, se dará cuenta de que ya hay una serie de devoluciones de llamada en `window.AMP` y luego procesará toda la cola. Si luego ejecuta la misma función nuevamente, seguirá funcionando, ya que Shadow AMP reemplaza `window.AMP` por sí mismo y un `push` personalizado que simplemente activa la devolución de llamada de inmediato.

[tip type="tip"] **SUGERENCIA:** para que el ejemplo de código anterior sea práctico, le recomendamos que lo incluya en una Promesa y, a continuación, utilice siempre dicha Promesa antes de trabajar con la API de AMP. Mire nuestro [código de demostración de React](https://github.com/ampproject/amp-publisher-sample/blob/master/amp-pwa/src/components/amp-document/amp-document.js#L20) para ver un ejemplo. [/tip]

## Manejar la navegación en su aplicación web progresiva

Aún deberá implementar este paso manualmente. Después de todo, depende de usted cómo presentar los enlaces al contenido en su concepto de navegación. ¿Varias listas? ¿Un montón de cartas?

En un escenario común, obtendría JSON que devuelve URL ordenadas con algunos metadatos. Al final, debe terminar con una devolución de llamada de función que se activa cuando el usuario hace clic en uno de los enlaces, y dicha devolución de llamada debe incluir la URL de la página AMP solicitada. Si tiene eso, está listo para el paso final.

## Utilice la API Shadow AMP para renderizar una página en línea

Finalmente, cuando desee mostrar contenido después de una acción del usuario, es hora de buscar el documento AMP relevante y dejar que Shadow AMP se haga cargo. Primero, implemente una función para buscar la página, similar a esta:

[sourcecode:javascript]
function fetchDocument(url) {

  // unfortunately fetch() does not support retrieving documents,
  // so we have to resort to good old XMLHttpRequest.
  var xhr = new XMLHttpRequest();

  return new Promise(function(resolve, reject) {
    xhr.open('GET', url, true);
    xhr.responseType = 'document';
    xhr.setRequestHeader('Accept', 'text/html');
    xhr.onload = function() {
      // .responseXML contains a ready-to-use Document object
      resolve(xhr.responseXML);
    };
    xhr.send();
  });
}
[/sourcecode]

[tip type="important"] **IMPORTANTE:** para simplificar el ejemplo de código anterior, omitimos el manejo de errores. Siempre debe asegurarse de detectar y manejar los errores con elegancia. [/tip]

Ahora que tenemos nuestro `Document` listo para usar, es hora de dejar que AMP se haga cargo y lo procese. Obtenga una referencia al elemento DOM que sirve como contenedor para el documento AMP, luego llame a `AMP.attachShadowDoc()` , así:

[sourcecode:javascript]
// This can be any DOM element
var container = document.getElementById('container');

// The AMP page you want to display
var url = "https://my-domain/amp/an-article.html";

// Use our fetchDocument method to get the doc
fetchDocument(url).then(function(doc) {
  // Let AMP take over and render the page
  var ampedDoc = AMP.attachShadowDoc(container, doc, url);
});
[/sourcecode]

[tip type="tip"] **SUGERENCIA:** antes de entregar el documento a AMP, es el momento perfecto para eliminar los elementos de la página que tienen sentido cuando se muestra la página AMP de forma independiente, pero no en modo incrustado: por ejemplo, pies de página y encabezados. [/tip]

¡Y eso es! Su página AMP se representa como un elemento secundario de su aplicación web progresiva general.

## Limpia después de ti mismo

Lo más probable es que su usuario navegue de AMP a AMP dentro de su aplicación web progresiva. Al descartar la página AMP renderizada anteriormente, asegúrese siempre de informarle a AMP, así:

[sourcecode:javascript]
// ampedDoc is the reference returned from AMP.attachShadowDoc
ampedDoc.close();
[/sourcecode]

Esto le dirá a AMP que ya no está utilizando este documento y liberará memoria y sobrecarga de CPU.

## Míralo en acción

[video src = "/ static / img / docs / pwamp_react_demo.mp4" width = "620" height = "1100" loop = "true", controls = "true"]

Puede ver el patrón "AMP en PWA" en acción en la [muestra de React](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa) que hemos creado. Demuestra transiciones suaves durante la navegación y viene con un componente React simple que envuelve los pasos anteriores. Es lo mejor de ambos mundos: JavaScript personalizado y flexible en la aplicación web progresiva y AMP para impulsar el contenido.

- Coge el código fuente aquí: [https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa)
- Utilice el componente React de forma independiente a través de npm: [https://www.npmjs.com/package/react-amp-document](https://www.npmjs.com/package/react-amp-document)
- Véalo en acción aquí: [https://choumx.github.io/amp-pwa/](https://choumx.github.io/amp-pwa/) (mejor en su teléfono o emulación móvil)

También puede ver una muestra de PWA y AMP usando el marco de Polymer. La muestra usa [amp-viewer](https://github.com/PolymerLabs/amp-viewer/) para insertar páginas AMP.

- Coge el código aquí: [https://github.com/Polymer/news/tree/amp](https://github.com/Polymer/news/tree/amp)
- Véalo en acción aquí: [https://polymer-news-amp.appspot.com/](https://polymer-news-amp.appspot.com/)
