---
"$title": Verwenden Sie AMP als Datenquelle für Ihre PWA
"$order": '1'
description: Wenn Sie in AMP investiert, aber noch keine Progressive Web App erstellt haben, können Ihre AMP-Seiten die Entwicklung Ihrer Progressive Web App erheblich vereinfachen.
formats:
  - Webseiten
author: pbakaus
---

Wenn Sie in AMP investiert, aber noch keine Progressive Web App erstellt haben, können Ihre AMP-Seiten die Entwicklung Ihrer Progressive Web App erheblich vereinfachen. In diesem Leitfaden erfahren Sie, wie Sie AMP in Ihrer Progressive Web App nutzen und Ihre vorhandenen AMP-Seiten als Datenquelle verwenden.

## Von JSON zu AMP

Im gängigsten Szenario ist eine Progressive Web App eine Einzelseitenanwendung, die über Ajax eine Verbindung zu einer JSON-API herstellt. Diese JSON-API gibt dann Datensätze zurück, um die Navigation zu steuern, und den tatsächlichen Inhalt zum Rendern der Artikel.

Teststrecke 1

Sie würden dann fortfahren und den Rohinhalt in verwendbares HTML konvertieren und auf dem Client rendern. Dieser Prozess ist kostspielig und oft schwer zu warten. Stattdessen können Sie Ihre bereits vorhandenen AMP-Seiten als Inhaltsquelle wiederverwenden. Das Beste daran ist, dass AMP dies in nur wenigen Codezeilen trivial macht.

Teststrecke 2

## Füge "Shadow AMP" in deine Progressive Web App ein

Der erste Schritt besteht darin, eine spezielle Version von AMP, die wir „Shadow AMP“ nennen, in Ihre Progressive Web App einzubinden. Ja, das ist richtig – Sie laden die AMP-Bibliothek auf der Seite der obersten Ebene, aber sie steuert nicht den Inhalt der obersten Ebene. Es "verstärkt" nur die Teile unserer Seite, denen Sie es mitteilen.

Fügen Sie Shadow AMP in den Kopf Ihrer Seite ein, wie folgt:

[sourcecode:html]
<!-- Asynchronously load the AMP-with-Shadow-DOM runtime library. -->
<script async src="https://cdn.ampproject.org/shadow-v0.js"></script>
[/sourcecode]

### Woher wissen Sie, wann die Shadow AMP API einsatzbereit ist?

Wir empfehlen Ihnen, die Shadow AMP-Bibliothek mit dem `async` Attribut zu laden. Das bedeutet jedoch, dass Sie einen bestimmten Ansatz verwenden müssen, um zu verstehen, wann die Bibliothek vollständig geladen und einsatzbereit ist.

Das richtige Signal, das Sie beobachten sollten, ist die Verfügbarkeit der globalen `AMP` Variablen, und Shadow AMP verwendet einen „ [asynchronen Funktionsladeansatz](http://mrcoles.com/blog/google-analytics-asynchronous-tracking-how-it-work/) “, um dabei zu helfen. Betrachten Sie diesen Code:

[sourcecode:javascript]
(window.AMP = window.AMP || []).push(function(AMP) {
  // AMP is now available.
});
[/sourcecode]

Dieser Code funktioniert und eine beliebige Anzahl von Callbacks, die auf diese Weise hinzugefügt werden, wird tatsächlich ausgelöst, wenn AMP verfügbar ist, aber warum?

Dieser Code bedeutet übersetzt:

1. „Wenn window.AMP nicht existiert, erstellen Sie ein leeres Array, um seine Position einzunehmen“
2. "Dann schieben Sie eine Callback-Funktion in das Array, die ausgeführt werden soll, wenn AMP bereit ist"

Es funktioniert, weil die Shadow AMP-Bibliothek beim tatsächlichen Laden erkennt, dass unter `window.AMP` , und dann die gesamte Warteschlange verarbeitet. Wenn Sie die gleiche Funktion später erneut ausführen, funktioniert sie immer noch, da Shadow AMP window.AMP durch sich selbst und eine benutzerdefinierte `push` Methode ersetzt, die den Callback einfach `window.AMP`

[tip type="tip"] TIPP **–** Um das obige Codebeispiel praktikabel zu machen, empfehlen wir, es in ein Promise zu packen und dann immer dieses Promise zu verwenden, bevor Sie mit der AMP-API arbeiten. Sehen Sie sich unseren [React-Democode](https://github.com/ampproject/amp-publisher-sample/blob/master/amp-pwa/src/components/amp-document/amp-document.js#L20) für ein Beispiel an. [/tip]

## Behandeln Sie die Navigation in Ihrer Progressive Web App

Sie müssen diesen Schritt immer noch manuell implementieren. Schließlich liegt es an Ihnen, wie Sie Links zu Inhalten in Ihrem Navigationskonzept präsentieren. Mehrere Listen? Ein Haufen Karten?

In einem gängigen Szenario rufen Sie JSON ab, das geordnete URLs mit einigen Metadaten zurückgibt. Am Ende sollten Sie einen Funktionsrückruf erhalten, der ausgelöst wird, wenn der Benutzer auf einen der Links klickt, und dieser Rückruf sollte die URL der angeforderten AMP-Seite enthalten. Wenn Sie das haben, sind Sie bereit für den letzten Schritt.

## Verwenden Sie die Shadow AMP API, um eine Seite inline zu rendern

Wenn Sie schließlich Inhalte nach einer Benutzeraktion anzeigen möchten, ist es an der Zeit, das entsprechende AMP-Dokument abzurufen und Shadow AMP übernehmen zu lassen. Implementieren Sie zunächst eine Funktion zum Abrufen der Seite, ähnlich dieser:

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

[tip type="important"] **WICHTIG –** Um das obige Codebeispiel zu vereinfachen, haben wir die Fehlerbehandlung übersprungen. Sie sollten immer darauf achten, Fehler korrekt abzufangen und zu behandeln. [/tip]

Da wir nun unser gebrauchsfertiges `Document` Objekt haben, ist es an der Zeit, es von AMP übernehmen und rendern zu lassen. Rufen Sie einen Verweis auf das DOM-Element ab, das als Container für das AMP-Dokument dient, und rufen `AMP.attachShadowDoc()` wie folgt auf:

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

[tip type="tip"] TIPP **–** Bevor Sie das Dokument tatsächlich an AMP übergeben, ist es der perfekte Zeitpunkt, um Seitenelemente zu entfernen, die sinnvoll sind, wenn die AMP-Seite eigenständig, aber nicht im eingebetteten Modus angezeigt wird: zum Beispiel Fußzeilen und Kopfzeilen. [/tip]

Und das ist es! Ihre AMP-Seite wird als untergeordnetes Element Ihrer gesamten Progressive Web App gerendert.

## Mach hinter dir auf

Es besteht die Möglichkeit, dass Ihr Benutzer innerhalb Ihrer Progressive Web App von AMP zu AMP navigiert. Wenn Sie die vorherige gerenderte AMP-Seite verwerfen, informieren Sie AMP immer wie folgt:

[sourcecode:javascript]
// ampedDoc is the reference returned from AMP.attachShadowDoc
ampedDoc.close();
[/sourcecode]

Dadurch wird AMP mitgeteilt, dass Sie dieses Dokument nicht mehr verwenden und Speicher- und CPU-Overhead freigeben.

## Sehen Sie es in Aktion

[video src="/static/img/docs/pwamp_react_demo.mp4" width="620" height="1100" loop="true", control="true"]

Sie können das Muster "AMP in PWA" in dem von uns erstellten [React-Beispiel in Aktion sehen.](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa) Es zeigt sanfte Übergänge während der Navigation und wird mit einer einfachen React-Komponente geliefert, die die obigen Schritte umschließt. Es ist das Beste aus beiden Welten – flexibles, benutzerdefiniertes JavaScript in der Progressive Web App und AMP zur Steuerung des Inhalts.

- Holen Sie sich den Quellcode hier: [https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa](https://github.com/ampproject/amp-publisher-sample/tree/master/amp-pwa)
- Verwenden Sie die React-Komponente eigenständig über npm: [https://www.npmjs.com/package/react-amp-document](https://www.npmjs.com/package/react-amp-document)
- Sehen Sie es hier in Aktion: [https://choumx.github.io/amp-pwa/](https://choumx.github.io/amp-pwa/) (am besten auf Ihrem Telefon oder Handy-Emulation)

Sie können auch ein Beispiel von PWA und AMP mit dem Polymer-Framework anzeigen. Das Beispiel verwendet [amp-viewer](https://github.com/PolymerLabs/amp-viewer/) zum Einbetten von AMP-Seiten.

- Holen Sie sich den Code hier: [https://github.com/Polymer/news/tree/amp](https://github.com/Polymer/news/tree/amp)
- Sehen Sie es hier in Aktion: [https://polymer-news-amp.appspot.com/](https://polymer-news-amp.appspot.com/)
