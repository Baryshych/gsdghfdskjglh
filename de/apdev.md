---
"$title": Erstellen Sie ein UI-Widget mit benutzerdefiniertem JavaScript
"$order": '101'
formats:
  - Webseiten
tutorial: wahr
author:
  - morsssss
  - CrystalOnScript
description: Für Web-Erlebnisse, die ein hohes Maß an Anpassung erfordern, hat AMP amp-script entwickelt, eine Komponente, die die Verwendung von beliebigem JavaScript auf Ihrer AMP-Seite ermöglicht, ohne die Leistung der Seite zu beeinträchtigen.
---

In this tutorial, you'll learn how to use `<amp-script>`, a component that allows developers to write custom JavaScript in AMP. You'll use this to build a widget that checks the contents of a password input field, only allowing it to be submitted when certain requirements are met. AMP already provides this functionality with `<amp-form>`, but `<amp-script>` will empower you to create a custom experience.1

## In this tutorial, you'll learn how to use `<amp-script>`, a component that allows developers to write custom JavaScript in AMP. You'll use this to build a widget that checks the contents of a password input field, only allowing it to be submitted when certain requirements are met. AMP already provides this functionality with `<amp-form>`, but `<amp-script>` will empower you to create a custom experience.

- Ein moderner Webbrowser
- Grundkenntnisse in HTML, CSS und JavaScript
- Entweder:
    - ein lokaler Webserver und ein Code-Editor wie [SublimeText](https://www.sublimetext.com) oder [VSCode](https://code.visualstudio.com/)
    - *oder* [CodePen](https://codepen.io/) , [Glitch](https://glitch.com/) oder ein ähnlicher Online-Spielplatz

## Hintergrund

AMP zielt darauf ab, Websites für Benutzer schneller und stabiler zu machen. Übermäßiges JavaScript kann eine Webseite verlangsamen. Manchmal müssen Sie jedoch Funktionen erstellen, die AMP-Komponenten nicht bieten. In solchen Fällen können Sie die [`<amp-script>`](../../../documentation/components/reference/amp-script.md) , um benutzerdefiniertes JavaScript zu schreiben.

Lass uns anfangen!

# Einstieg

Um den Startcode zu erhalten, laden Sie[dieses Github-Repository](https://github.com/ampproject/samples/tree/master/amp-script-tutorial) herunter oder klonen Sie es. Sobald Sie dies getan haben, `cd` Sie mit cd in das von Ihnen erstellte Verzeichnis. : Sie werden zwei Verzeichnisse sehen `starter_code` und `finished_code` . `finished_code` enthält das, was Sie in diesem Tutorial erstellen. Schauen wir uns das also noch nicht an. Stattdessen `cd` in `starter_code` . Dies enthält eine Webseite, die unser Formular nur mit [`<amp-form>`](../../../documentation/components/reference/amp-form.md) implementiert, ohne Hilfe von `<amp-script>` .

Um diese Übung durchzuführen, müssen Sie einen Webserver auf Ihrem Computer ausführen. Wenn Sie dies bereits tun, sind Sie bereit! Wenn ja, können Sie je nach Einrichtung auf die Starter-Webseite zugreifen, indem Sie in Ihren Browser eine URL wie `http://localhost/amp-script-tutorial/starter_code/index.html` .

Alternativ können Sie einen schnellen lokalen Server mit etwas wie [serve](https://www.npmjs.com/package/serve) einrichten, einem [Node.js-](https://nodejs.org/) basierten statischen Inhaltsserver. Wenn Sie Node.js nicht installiert haben, laden Sie es [hier](https://nodejs.org/) herunter. Sobald Node installiert ist, geben Sie `npx serve` in Ihre Befehlszeile ein. Hier gelangen Sie dann zu Ihrer Website:

`http://localhost:5000/`

Sie können auch einen Online-Spielplatz wie [Glitch](https://glitch.com/) oder [CodePen nutzen](https://codepen.io/) . <a href="itch%5D(https://glitch.com/~grove-thankful-ragdoll" target="_blank">Dieses</a> enthält den gleichen Code wie das Github-Repository, und Sie können stattdessen dort beginnen, wenn Sie möchten!

Sobald Sie dies getan haben, sehen Sie unsere Starter-Webseite:

{{ image('/static/img/docs/tutorials/custom-javascript-tutorial/starter-form.jpg', 600, 325, layout='intrinsic', alt='Webformular mit E-Mail- und Passworteingaben', align ='Mitte') }}

Öffnen Sie `starter_code/index.html` in Ihrem bevorzugten Code-Editor. Sehen Sie sich den HTML-Code für dieses Formular an. Beachten Sie, dass das Kennwort `<input>` dieses Attribut enthält:

```html
on="tap:rules.show; input-debounced:rules.show"
```

Dies weist AMP an, die Regeln `<div>` wenn der Benutzer auf das Passwort `<input>` tippt oder darauf klickt, und auch, nachdem er dort ein beliebiges Zeichen eingegeben hat. Wir würden es vorziehen , die verwenden `focus` Veranstaltung, die auch den Fall abdecken würde , wo die Benutzer mit der Tabulatortaste in den Eingang. Zumindest zum Zeitpunkt der Erstellung dieses Tutorials gibt AMP dieses Ereignis nicht weiter, daher haben wir diese Option nicht. Mach dir keine Sorge. Wir sind dabei, das mit `<amp-script>` zu beheben!

Das Passwort `<input>` enthält ein weiteres interessantes Attribut:

```html
pattern="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^a-z\d]).{8,}$"
```

Dieser reguläre Ausdruck kombiniert eine Reihe kleinerer regulärer Ausdrücke, von denen jeder eine unserer Validierungsregeln ausdrückt. AMP [lässt das Formular erst dann absenden,](../../../documentation/components/reference/amp-form.md#verification) wenn der Inhalt der Eingabe übereinstimmt. Wenn der Benutzer es versucht, wird eine Fehlermeldung mit einigen Details angezeigt:

{{ image('/static/img/docs/tutorials/custom-javascript-tutorial/starter-form-error.jpg', 600, 442, layout='intrinsic', alt='Webformular mit Fehlermeldung', align ='Mitte') }}

[tip type="note"] Da der von uns bereitgestellte Code keinen Webservice enthält, der das Senden von Formularen verarbeitet, bringt das Absenden des Formulars nichts. Natürlich können Sie diese Funktion gerne zu Ihrem eigenen Code hinzufügen! [/tip]

Diese Erfahrung ist akzeptabel - aber leider kann AMP nicht erklären, welche unserer Überprüfungsregeln fehlgeschlagen sind. Es kann nicht wissen, da wir die Regeln in einen einzigen regulären Ausdruck quetschen mussten.

Lassen Sie uns nun `<amp-script>` , um eine benutzerfreundlichere Erfahrung zu erstellen!

# Neuaufbau mit &lt;amp-script&gt;

Um `<amp-script>` , müssen wir sein eigenes JavaScript importieren. Öffnen Sie `index.html` und fügen Sie Folgendes zum `<head>` .

```html
<head>
 ...
  <script async custom-element="amp-script" src="https://cdn.ampproject.org/v0/amp-script-0.1.js"></script>
  ...
</head>

```

`<amp-script>` lässt uns unser eigenes JavaScript inline oder in eine externe Datei schreiben. In dieser Übung schreiben wir genug Code, um eine separate Datei zu verdienen. Erstellen Sie ein neues Verzeichnis mit dem Namen `js` , und fügen Sie eine neue Datei mit dem Namen , um es `validate.js` .

`<amp-script>` erlaubt Ihrem JavaScript, seine DOM-Kinder zu manipulieren - die Elemente, die die Komponente einschließt. Es kopiert diese DOM-Kinder in ein virtuelles DOM und gewährt Ihrem Code Zugriff auf dieses virtuelle DOM. In dieser Übung möchten wir, dass unser JavaScript unser `<form>` und seinen Inhalt steuert. Also wickeln wir das `<form>` in eine `<amp-script>` -Komponente ein, wie folgt:

```html
<amp-script src="js/validate.js" layout="fixed" sandbox="allow-forms" height="500" width="750">
  <form method="post" action-xhr="#" target="_top" class="card">
    ...
  </form>
</amp-script>
```

Unser `<amp-script>` enthält das Attribut `sandbox="allow-forms"` . Das teilt AMP mit, dass das Skript den Inhalt des Formulars ändern darf.

Da AMP darauf abzielt, eine schnelle, visuell stabile Benutzererfahrung zu garantieren, lässt es unser JavaScript zu keiner Zeit uneingeschränkte Änderungen am DOM vornehmen. Ihr JavaScript kann weitere Änderungen vornehmen, wenn sich die Größe der `<amp-script>` -Komponente nicht ändern kann. Es ermöglicht auch umfangreichere Änderungen nach einer Benutzerinteraktion. Details finden Sie in [der Referenzdokumentation](../../../documentation/components/reference/amp-script.md) . Für dieses Tutorial genügt es zu wissen , dass wir angegeben haben `layout` - Typ, der nicht ist `container` , und wir haben verwendeten HTML - Attribute der Komponente Größe zu sperren. Dies bedeutet, dass alle DOM-Manipulationen auf einen bestimmten Bereich der Seite beschränkt sind.

Wenn Sie die [Chrome-Erweiterung AMP Validator verwenden](https://chrome.google.com/webstore/detail/amp-validator/nmoffdblmcmgeicmolmhobpoocbbmknc) , wird jetzt eine Fehlermeldung angezeigt:

{{ image('/static/img/docs/tutorials/custom-javascript-tutorial/relative-url-error.png', 600, 177, layout='intrinsic', alt='Fehler bezüglich relativer URL', align= 'Center' ) }}

[tip type="note"] Wenn Sie diese Erweiterung nicht haben, fügen Sie `#development=1` an Ihre URL an und AMP gibt Validierungsfehler an Ihre Konsole aus. [/tip]

Was bedeutet das? Wenn Ihr `<amp-script>` sein JavaScript aus einer externen Datei lädt, erfordert AMP die Angabe einer absoluten URL. Wir könnten dies beheben, indem wir `http://localhost/js/validate.js` . AMP erfordert aber auch die Verwendung von [HTTPS](https://developers.google.com/web/fundamentals/security/encrypt-in-transit/why-https) . Wir würden also immer noch einen Validierungsfehler erhalten, und das Einrichten von SSL auf unserem lokalen Webserver ist nicht Gegenstand dieses Tutorials. Wenn Sie dies tun möchten, können Sie den Anweisungen in [diesem Beitrag](https://timonweb.com/posts/running-expressjs-server-over-https/) folgen.

Als nächstes können wir das `pattern` und seinen regulären Ausdruck aus unserem Formular entfernen: Wir brauchen es nicht mehr!

Wir werden auch das `on` Attribut entfernen, das derzeit verwendet wird, um AMP anzuweisen, unsere Passwortregeln anzuzeigen. Wie oben angedeutet, werden wir stattdessen verwenden `<amp-script>` des Browsers erfassen `focus` Ereignis.

```html
pattern="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^a-z\d]).{8,}$"
on="tap:rules.show; input-debounced:rules.show"
```

Stellen wir nun sicher, dass unser `<amp-script>` funktioniert. Öffnen `validate.js` und fügen Sie eine Debug-Nachricht hinzu:

```js
console.log("Hello, amp-script!");
```

Gehen Sie zu Ihrem Browser, öffnen Sie die Konsole und laden Sie die Seite neu. Stellen Sie sicher, dass Sie Ihre Nachricht sehen!

{{ image('/static/img/docs/tutorials/custom-javascript-tutorial/hello-amp-script.png', 600, 22, layout='intrinsic', alt='Hallo amp-script message in Console' , align='center' ) }}

## Wo ist mein JavaScript?

`<amp-script>` führt Ihr JavaScript in einem Web Worker aus. Web-Worker können nicht direkt auf das DOM zugreifen, daher `<amp-script>` dem Worker Zugriff auf eine virtuelle Kopie des DOM, die mit dem echten DOM synchronisiert wird. `<amp-script>` bietet Emulationen vieler gängiger DOM-APIs, die Sie fast alle wie gewohnt in Ihrem JavaScript verwenden können.

Wenn Sie Ihr Skript zu irgendeinem Zeitpunkt debuggen müssen, können Sie Breakpoints in JavaScript in einem Web Worker auf die gleiche Weise wie in jedem anderen JavaScript setzen. Sie müssen nur wissen, wo Sie es finden.

Öffnen Sie in den Chrome DevTools die Registerkarte "Quellen". Unten sehen Sie eine lange hexadezimale Zeichenfolge wie die unten gezeigte. Erweitern Sie das, erweitern Sie dann den Bereich "keine Domain", und Sie sehen Ihr Skript:

{{ image('/static/img/docs/tutorials/custom-javascript-tutorial/script-in-sources.png', 303, 277, layout='intrinsic', alt='amp-script JavaScript im DevTools-Quellbereich ', ausrichten='center') }}

# Hinzufügen unseres JavaScripts

Da wir nun wissen, dass unser `<amp-script>` funktioniert, schreiben wir etwas JavaScript!

Das erste, was wir tun möchten, ist, die DOM-Elemente, mit denen wir arbeiten werden, zu packen und in Globals zu verstauen. Unser Code verwendet die Passworteingabe, die Schaltfläche zum Senden und den Bereich, der die Passwortregeln anzeigt. Fügen Sie diese drei Erklärungen zu `validate.js` :

```js
const passwordBox = document.getElementById("passwordBox");
const submitButton = document.getElementById("submitButton");
const rulesArea = document.getElementById("rules");
```

Beachten Sie, dass wir normale DOM-API-Methoden wie `getElementById()` . Obwohl unser Code in einem Worker ausgeführt wird und die Worker keinen direkten Zugriff auf das DOM haben, `<amp-script>` eine virtuelle Kopie des DOM und emuliert einige gängige APIs, die [hier](https://github.com/ampproject/worker-dom/blob/main/web_compat_table.md) aufgeführt sind . Diese APIs geben uns genügend Tools, um die meisten Anwendungsfälle abzudecken. Es ist jedoch wichtig zu beachten, dass nur eine Teilmenge der DOM-API unterstützt wird. Andernfalls wäre das in `<amp-script>` enthaltene JavaScript enorm und würde die Leistungsvorteile von AMP zunichte machen!

Wir müssen diese IDs zu zwei der Elemente hinzufügen. Öffnen Sie `index.html` , suchen Sie das Passwort `<input>` und den Submit `<button>` und fügen Sie die IDs hinzu. Fügen Sie auch ein `disabled` Attribut zum Absenden `<button>` , um zu verhindern, dass der Benutzer darauf klickt, bis dies gewünscht wird.

```html
<input type=password
       id="passwordBox"

...

<button type="submit" id="submitButton" tabindex="3" disabled>Submit</button>
```

Seite aktualisieren. Sie können überprüfen, ob diese Globals korrekt festgelegt wurden, indem Sie in der Konsole nachsehen, genau wie bei JavaScript für Nicht-Worker:

{{ image('/static/img/docs/tutorials/custom-javascript-tutorial/global-set.png', 563, 38, layout='intrinsic', alt='Konsolenmeldung zeigt an, dass der SubmitButton gesetzt ist', align= 'Center' ) }}

Außerdem fügen wir jedem `<li>` in `<div id="rules">` . Jede davon enthält eine individuelle Regel, deren Farbe wir steuern möchten. Und wir entfernen jede Instanz von `class="invalid"` . Unser neues JavaScript wird das bei Bedarf hinzufügen!

```html
<ul>
  <li id="lower">Lowercase letter</li>
  <li id="upper">Capital letter</li>
  <li id="digit">Digit</li>
  <li id="special">Special character (@$!%*?&)</li>
  <li id="eight">At least 8 characters long</li>
</ul>
```

## Implementierung unserer Passwortprüfungen in JavaScript

Als Nächstes entpacken wir die regulären Ausdrücke aus unserem `pattern` . Jede Regex repräsentierte eine unserer Regeln. Lassen Sie uns in den unteren Teil eines Objekts Karte `validate.js` , die jede Regel mit dem Kriterium es Kontrollen zuordnet.

```js
const checkRegexes = {
  lower: /[a-z]/,
  upper: /[A-Z]/,
  digit: /\d/,
  special: /[^a-zA-Z\d]/i,
  eight: /.{8}/
};
```

Wenn diese Globals gesetzt sind, können wir die Logik schreiben, die das Passwort überprüft und die Benutzeroberfläche entsprechend anpasst. Wir fügen unsere Logik in eine Funktion namens `initCheckPassword` , die ein einzelnes Argument benötigt – das DOM-Element des Passworts `<input>` . Bei diesem Ansatz wird das DOM-Element praktischerweise in einem Verschluss versteckt.

```js
function initCheckPassword(element) {

}
```

Als Nächstes `initCheckPassword` wir initCheckPassword mit den Funktionen und Ereignis-Listener-Zuweisungen, die wir benötigen. Fügen Sie zunächst eine kleine Funktion hinzu, die eine einzelne Regel `<li>` grün färbt, wenn die Regel erfolgreich ist - und eine andere, die sie rot färbt, wenn sie fehlschlägt.

```js
function initCheckPassword(el) {
  const checkPass = (el) => {
    el.classList.remove("invalid");
    el.classList.add("valid");
  };

  const checkFail = (el) => {
    el.classList.remove("valid");
    el.classList.add("invalid");
  };
};
```

Lassen Sie uns diese `valid` und `invalid` Klassen tatsächlich grün oder rot färben. Gehen Sie zurück zu `index.html` und fügen Sie diese beiden Regeln zum Tag `<style amp-custom>`

```css
li.valid {
  color: #2d7b1f;
}

li.invalid {
  color:#c11136;
}
```

Jetzt können wir die Logik hinzufügen, die den Inhalt des Passworts `<input>` mit unseren Regeln überprüft. Fügen Sie direkt vor der schließenden Klammer eine neue Funktion namens `checkPassword()` zu `initCheckPassword()`

```js
const checkPassword = () => {
  const password = element.value;
  let failed = false;

  for (const check in checkRegexes) {
    let li = document.getElementById(check);

    if (password.match(checkRegexes[check])) {
      checkPass(li);
    } else {
      checkFail(li);
      failed = true;
    }
  }

  if (!failed) {
    submitButton.removeAttribute("disabled");
  }
};
```

Diese Funktion macht Folgendes:

1. Erfasst den Inhalt des Passworts `<input>` .
2. Erstellt ein Flag namens `failed` , initialisiert auf `false` .
3. Durchläuft jede unserer Regexes und testet jede mit dem Passwort:
    - Wenn das Passwort einen Test nicht besteht, rufen Sie `checkFail()` auf, um die entsprechende Regel rot zu färben. Außerdem wurde `failed` auf `true` .
    - Wenn das Passwort einen Test besteht, rufen Sie `checkPass()` auf, um die entsprechende Regel grün zu machen.
4. Wenn schließlich keine Regel fehlgeschlagen ist, ist das Passwort gültig und wir aktivieren die Schaltfläche Senden.

Jetzt brauchen wir nur noch ein paar Event-Listener. Denken Sie daran , wie wir waren nicht in der Lage zu verwenden `focus` Ereignis in AMP? In `<amp-script>` können wir das. Jedes Mal , wenn das Passwort `<input>` die empfängt `focus` - Ereignis, werden wir die Regeln anzuzeigen. Und immer wenn der Benutzer eine Taste in dieser Eingabe drückt, rufen wir `checkPassword()` .

Fügen Sie diese beiden Ereignis-Listener am Ende von `initCheckPassword()` direkt vor der schließenden Klammer hinzu:

```js
element.addEventListener("focus", () => rulesArea.removeAttribute("hidden"));
element.addEventListener("keyup", checkPassword);
```

Schließlich am Ende des `validate.js` , fügen Sie eine Zeile , die initialisiert `initCheckPassword` mit dem Passwort `<input>` DOM - Element:

```js
initCheckPassword(passwordBox);
```

Unsere Logik ist nun abgeschlossen! Wenn das Passwort allen unseren Kriterien entspricht, werden alle Regeln grün und unser Senden-Button wird aktiviert. Sie sollten jetzt in der Lage sein, eine Interaktion wie diese zu haben:

<figure class="alignment-wrapper margin-">
  <amp-video width="762" height="564" layout="responsive" autoplay loop noaudio>
    <source src="/static/img/docs/tutorials/custom-javascript-tutorial/finished-project.mp4" type="video/mp4">
    <source src="/static/img/docs/tutorials/custom-javascript-tutorial/finished-project.webm" type="video/webm">
  </amp-video>
</figure>

Wenn Sie nicht weiterkommen, können Sie immer auf dem Arbeits Code verweisen im `finished_code` Verzeichnis.

# Herzliche Glückwünsche!

Sie haben gelernt, wie Sie mit `<amp-script>` Ihr eigenes JavaScript in AMP schreiben. Es ist Ihnen gelungen, die `<amp-form>` -Komponente mit Ihrer eigenen benutzerdefinierten Logik und UI-Funktionen zu erweitern! Fühlen Sie sich frei, Ihrer neuen Seite mehr Funktionen hinzuzufügen! Und um mehr über `<amp-script>` zu erfahren, lesen Sie [die Referenzdokumentation](../../../documentation/components/reference/amp-script.md) .
