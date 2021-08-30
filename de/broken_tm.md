---
"$title": Integrieren Sie Ihre Anzeigentechnologien in AMP
order: '3'
formats:
  - Anzeigen
teaser:
  text: |2-

    Wenn Sie ein Anbieter von Anzeigentechnologie sind, der AMP HTML integrieren möchte,
    Bitte beachten Sie die Richtlinien unten.
toc: wahr
---

<!--
This file is imported from https://github.com/ampproject/amphtml/blob/master/ads/_integration-guide.md.
Please do not change this file.
If you have found a bug or an issue please
have a look and request a pull request there.
-->

Wenn Sie ein Anbieter von Anzeigentechnologie sind, der AMP-HTML integrieren möchte, lesen Sie die folgenden Richtlinien. Um eine minimale Latenz und Qualität zu gewährleisten, befolgen Sie bitte die [hier](https://github.com/ampproject/amphtml/blob/master/ads/../3p/README.md#ads) aufgeführten Anweisungen, bevor Sie eine Pull-Anfrage an das AMP-Open-Source-Projekt senden. Allgemeine Anleitungen zu den ersten Schritten mit Beiträgen zu AMP finden Sie unter [CONTRIBUTING.md](https://github.com/ampproject/amphtml/blob/master/ads/../CONTRIBUTING.md) .

## Anzeigenserver<a name="ad-server"></a>

*Beispiele : DFP, A9*

Als Ad-Server enthalten die von Ihnen unterstützten Publisher eine von Ihnen bereitgestellte JavaScript-Bibliothek und platzieren verschiedene "Anzeigen-Snippets", die auf der JavaScript-Bibliothek basieren, um Anzeigen abzurufen und auf der Website des Publishers zu rendern.

Da AMP Publishern nicht erlaubt, beliebiges JavaScript auszuführen, müssen Sie zum AMP-Open-Source-Code beitragen, damit das `amp-ad` Tag Anzeigen von Ihrem Ad-Server anfordern kann.

Beispiel: Der Amazon A9-Server kann mit der folgenden Syntax aufgerufen werden:

[sourcecode:html]
<amp-ad
  width="300"
  height="250"
  type="a9"
  data-aax_size="300x250"
  data-aax_pubname="test123"
  data-aax_src="302"
>
</amp-ad>
[/sourcecode]

Beachten Sie, dass jedes der Attribute, die auf `type` folgen, von den Parametern abhängt, die der A9-Server von Amazon erwartet, um eine Anzeige zu liefern. Die [Datei a9.js](https://github.com/ampproject/amphtml/blob/master/ads/./a9.js) zeigt Ihnen, wie die Parameter einem JavaScript-Aufruf zugeordnet sind, der den A9-Server über die URL `https://c.amazon-adsystem.com/aax2/assoc.js` Die entsprechenden Parameter, die vom AMP-Anzeigen-Tag übergeben werden, werden an die URL angehängt, um eine Anzeige zurückzugeben.

Ausführliche Informationen zum Integrieren Ihres Werbenetzwerks in AMP finden Sie unter [Integrieren von Werbenetzwerken in AMP](https://github.com/ampproject/amphtml/blob/master/ads/README.md) .

## Supply Side Platform (SSP) oder eine Ad Exchange<a name="supply-side-platform-ssp-or-an-ad-exchange"></a>

*Beispiele : Rubicon, Criteo ODER Appnexus, Ad-Exchange*

Wenn Sie eine Sell-Side-Plattform sind, die direkt von der Webseite eines Publishers aufgerufen werden möchte, müssen Sie die oben aufgeführten Anweisungen für die Integration in einen Ad Server befolgen. Hinzufügen eigener `type` Wert an den Verstärker-Anzeigen - Tag können Sie Ihren Tag direkt an den Verlag verteilen, so dass sie Ihre Tags direkt in ihre AMP Seiten einfügen.

Häufiger arbeiten SSPs mit dem Publisher zusammen, um das Trafficking der Anzeigen-Tags der SSP in ihrem Ad-Server durchzuführen. Stellen Sie in diesem Fall sicher, dass alle Assets, die von Ihrem Skript in das Creative des Ad-Servers geladen werden, über HTTPS erstellt werden. Bei einigen Anzeigenformaten wie Expandables gibt es einige Einschränkungen. Wir empfehlen Ihnen daher, die am häufigsten gelieferten Creative-Formate mit Ihren Publishern zu testen.

## Werbeagentur<a name="ad-agency"></a>

*Beispiele : Essenz, Omnicom*

Arbeiten Sie mit Ihrem Publisher zusammen, um sicherzustellen, dass die von Ihnen entwickelten Creatives AMP-konform sind. Da alle Creatives in Iframes geliefert werden, deren Größe beim Aufrufen der Anzeige bestimmt wird, stellen Sie sicher, dass Ihr Creative nicht versucht, die Größe des Iframes zu ändern.

Stellen Sie sicher, dass alle Assets, die Teil des Creatives sind, über HTTPS angefordert werden. Einige Anzeigenformate werden derzeit nicht vollständig unterstützt und wir empfehlen, die Creatives in einer AMP-Umgebung zu testen. Einige Beispiele sind: Rich Media Expandables, Interstitials, Page Level Ads.

## Videoplayer<a name="video-player"></a>

*Beispiele : Brightcove, Ooyala*

Ein Videoplayer, der in normalen HTML-Seiten funktioniert, funktioniert nicht in AMP und daher muss ein spezielles Tag erstellt werden, das es der AMP-Laufzeit ermöglicht, Ihren Player zu laden. Brightcove hat ein benutzerdefiniertes [amp-brightcove-](https://github.com/ampproject/amphtml/blob/master/extensions/amp-brightcove/amp-brightcove.md) Tag erstellt, mit dem Medien und Anzeigen auf AMP-Seiten wiedergegeben werden können.

Ein Brightcove-Player kann wie folgt aufgerufen werden:

[sourcecode:html]
<amp-brightcove
  data-account="1290862519001"
  data-video-id="ref:amp-docs-sample"
  data-player="S1Tt8cgaM"
  layout="responsive"
  width="480"
  height="270"
>
</amp-brightcove>
[/sourcecode]

Anweisungen zum Entwickeln eines Amp-Tags wie Brightcove finden Sie in [dieser Pull-Anfrage](https://github.com/ampproject/amphtml/pull/1052) .

## Video-Werbenetzwerk<a name="video-ad-network"></a>

*Beispiele : Tremor, Brightroll*

Wenn Sie ein Video-Werbenetzwerk sind, arbeiten Sie mit Ihrem Publisher zusammen, um sicherzustellen, dass:

- Alle Video-Assets werden über HTTPS bereitgestellt
- Der Videoplayer des Publishers bietet AMP-Unterstützung

## Datenmanagementplattform (DMP)<a name="data-management-platform-dmp"></a>

*Beispiele : KRUX, Bluekai*

Erfahren Sie, [wie Sie die benutzerdefinierte Anzeigenkonfiguration verbessern](https://amp.dev/documentation/components/amp-ad#enhance-incoming-ad-configuration) .

Sie können einen ähnlichen Ansatz verwenden, um den Anzeigenaufruf anzureichern, indem Sie Zielgruppensegmente, die Sie vom Nutzer-Cookie erhalten, an den Anzeigenaufruf übergeben.

## Sichtbarkeitsanbieter<a name="viewability-provider"></a>

*Beispiele : MOAT, Integral Ad Science*

Sichtbarkeitsanbieter integrieren sich in der Regel über die Creative-Wrapper des Ad-Servers in Publisher. Stellen Sie in diesem Fall sicher, dass der Creative-Wrapper alle Assets über HTTPS lädt.

Stellen Sie beispielsweise für MOAT sicher, dass `http://js.moatads.com` `https://z.moatads.com` umgestellt ist

Siehe auch den Ansatz zur Verwendung des [Schnittmuster-Beobachtermusters](https://github.com/ampproject/amphtml/blob/master/ads/README.md#ad-viewability) .

## Content-Empfehlungsplattform<a name="content-recommendation-platform"></a>

*Beispiele : Taboola, Outbrain*

Nützlich, wenn Sie heute JavaScript auf der Publisher-Website eingebettet haben, der Ansatz jedoch nicht auf AMP-Seiten funktioniert. Wenn Sie Inhalte auf einer AMP-Seite empfehlen möchten, empfehlen wir Ihnen, die [`amp-embed` Erweiterung](https://amp.dev/documentation/components/amp-ad) zu verwenden, um die Inhaltsdetails anzufordern. Sehen Sie sich das [Taboola-](https://github.com/ampproject/amphtml/blob/master/ads/taboola.md) Beispiel an.
