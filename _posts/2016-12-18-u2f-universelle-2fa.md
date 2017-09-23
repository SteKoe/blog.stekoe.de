---
title: "U2F - Universelle Zweifaktor-Authentifizierung"
layout: post
image: true
tags:
 - 2FA
 - U2F
 - FIDO
 - YubiKey
---

U2F ist ein von Google getriebener Standard, welcher unter dem Namen "FIDO" geführt wird. In Zusammenarbeit mit Yubico wurde die FIDO Alliance gegründet, welche fortan die Weiterentwicklung des FIDO-Standards übernimmt und überwacht. Kern dieses Standards ist ein USB-Dongle, welcher in Zusammenarbeit mit einer Software (bspw. Google Chrome) in der Lage ist eine Zweifaktor-Authentifizierung mittels eines Hardware-Dongles im Web zu realisieren. Einige Dienstleister bieten bereits die Möglichkeit an, den Login-Prozess mit Hilfe eines solchen USB-Dongles zusätzlich abzusichern. Neben den klassischen Verfahren wie One-Time-Passwords (OTP), welche ohne spezielle Soft- und Hardwarebindung auskommt, ist der Zugriff des Browsers auf das USB-Gerät für die U2F-Authentifizierung eine notwendige Voraussetzung und wird daher aktuell nicht von allen Browsern nativ unterstützt.

Um die in diesem Beitrag gezeigten Beispiele selbst ausprobieren zu können ist ein FIDO kompatibler Hardware-Dongle wie bspw. der "<a href="https://www.yubico.com/products/yubikey-hardware/yubikey4/">YubiKey</a>" notwendig. Zudem wird eine Node.JS-Umgebung in Version 4 benötigt. Aktuell ist der Zugriff auf den Dongle nur über Google Chrome ohne Zusatzsoftware möglich, sodass es sich anbietet, die Beispiele in diesem Browser auszuprobieren.
<h2>Das Grundgerüst</h2>
Bei dem entwickelten Beispiel handelt es sich um eine rudimentären Webseite, welcher mit Hilfe von Hapi.js auf Node.JS aufgesetzt wurde. Es existiert eine Login-Maske, ein interner Bereich, sowie eine Einstellungen-Seite und ein Logout. Die Einstelleungen-Seite erlaubt es dem Benutzer sein Zugang zusätzlich mit dem Yubikey zu schützen.
<h3>Vorgehen</h3>
Grundsätzlich ist zu beachten, dass die Verwendung von U2F nur funktioniert, sofern eine HTTPS-Verbindung zum Server vorliegt. Dies ist auch dann zu gewährleisten, wenn es sich um einen Test-Server handelt, der über <code>localhost</code> aufgerufen wird. Darüberhinaus ersetzt U2F nicht den klassische Loginansatz. Dieser ist weiterhin notwendig und auch Grundvoraussetzung für die Implementierung von U2F. Um U2F einsetzen zu können und Benutzern zu erlauben einen Dongle mit ihrem Benutzerkonto zu verknüpfen ist daher eine Benutzersession obligatorisch. Ferner werden sowohl im Frontend als auch im Backend entsprechende Bibliotheken benötigt, welche den FIDO-Standard implementieren. Yubico bietet hierfür ein JavaScript an, welches im Frontend eingebunden werden kann: <code>https://demo.yubico.com/js/u2f-api.js. </code> Für den Einsatz im Backend bietet sich die npm-Abhängigkeit <a href="https://www.npmjs.com/package/u2f">u2f</a> an, welche den serverseitigen Part implementiert.
<h4>1. Registration Request</h4>
Soll ein neuer Dongle registriert werden, muss der Server hierfür zunächst ein "registration request" stellen. Das folgende Gist zeigt, wie die Erzeugung eines "registration requests" aussieht (Zeile 2). Der Funktion <em>request(&lt;appId&gt;)</em> wird eine AppId übergeben, welche einem definierten Format folgen muss und auch mehrere "Facetten" abdecken kann.
<blockquote>An (application) facet is how an application is implemented on various platforms. For example, the application PayPal may have an Android app, an iOS app, and a Web app. These are all facets of the PayPal application. (Quelle: <a href="https://developers.yubico.com/U2F/App_ID.html">Yubico</a>)</blockquote>
Ist der "registration request" erstellt, wird dieser der Session angehängt, um den "request" zu einem späteren Zeitpunkt validieren zu können (Zeilen 6-16).

{% gist aec55b34e94bdde175399d31b9494a49 u2f.registration.request.js %}
<h4>2. Registration Result</h4>
Der vom Server erzeugte "registration request" wird an den Client übergeben. Er enthält eine Version, die AppId und eine Challenge. Diese Informationen müssen nun im Frontend in die u2f-Library übergeben werden:

{% gist aec55b34e94bdde175399d31b9494a49 u2f.process-request.js %}

Der Browser spricht nun den Dongle an und aktiviert diesen. Um der Anfrage zuzustimmen muss der Benutzer aktiv einen Knopf auf dem Dongle drücken, um so seine Zustimmung zum Pairing des Dongles mit der Anfrage zu geben. Tut der Benutzer dies nicht, wird ein Fehler geworfen, welcher mit dem Error-Code 5 angibt, dass die Benutzerinteraktion zu lange gedauert hat und ein erneuter "registration request" angefordert werden muss.

Gibt der Benutzer jedoch seine Zustimmung, wird eine Antwort auf die gegebene "Challenge" erstellt, die anschließend wieder an den Server gesendet werden muss.
<h4>3. Registration Verification</h4>
Mit Hilfe des vom Client zurück an den Server gesendeten Payloads kann nun überprüft werden, ob ein Pairing des Dongles mit dem Benutzerkonto erlaubt ist. Hierfür wird zunächst der "registration request" aus der Session ausgelesen und der u2f Bibliothek zusammen mit dem vom Client erzeugten Payload übergeben. Passen die Antwort vom Client zu dem in der Session gehaltenen "registration request" zusammen, wird ein öffentlicher Schlüssel, der den Dongle des Benutzers identifizieren kann, sowie ein <em>keyHandle</em> am Benutzer in der Datenbank abgelegt. Ab sofort kann mit Hilfe des öffentlichen Schlüssels geprüft werden, ob die von einem Dongle gesendete Signatur zu dem entsprechenden Benutzer passt.

{% gist aec55b34e94bdde175399d31b9494a49 u2f.handle.post.js %}
<h4>4. Login Request</h4>
Hat ein Benutzer einen FIDO-Dongle in seinem Benutzeraccount hinterlegt, muss die Login-Logik erweitert werden. Bisher war es ausreichend den Zugriff auf den internen Bereich direkt nach Eingabe der korrekten Kombination aus Benutzername und Passwort zu gewähren. Ist jedoch ein öffentlicher Schlüssel eines Benutzers hinterlegt, muss nach Eingabe der korrekten Benutzerdaten eine weitere Abfrage erfolgen, die den User auffordert den Dongle an den Computer anzuschließen und dem Login-Versuch durch drücken des Knopfes zuzustimmen.

Ähnlich wie bei der Registrierung eines Dongles (vgl. 1. Registration Request), muss vom Server eine Challenge angefordert werden, die auf den Daten des gespeicherten Dongles basiert. Hierfür wird auf Basis des gespeicherten <em>keyHandles</em> ein Sign-Request erstellt, welcher an den Client geschickt wird.

{% gist aec55b34e94bdde175399d31b9494a49 u2f.login.request.js %}
<h4>5. Sign Request</h4>
Nachdem der Sign-Request vom Server an den Client geschickt wurde, muss dieser durch den Benutzer bestätigt werden. Dies erfolgt über folgenden Code:

{% gist aec55b34e94bdde175399d31b9494a49 u2f.ui.login.request.js %}

Der vom Server gesendete <em>authRequest</em> wird der <em>sign(&lt;...&gt;)</em> Funktion übergeben und das Ergebnis zurück an den Server gesandt. Dieser muss nun prüfen, ob die erzeugte Antwort valide ist.
<h4>6. Verify Login</h4>
Die vom Client geschickte Antwort wird nun mit dem gestellten "login request" aus Schritt 4. sowie dem öffentlichen Schlüssel des Benutzers verifiziert. Werden vom u2f-Algorithmus keine Probleme festgestellt, wird die Session des Benutzers so initialisiert, dass der Zugriff auf die internen Seiten gewährt wird.

{% gist aec55b34e94bdde175399d31b9494a49 u2f.login.verification.js %}
<h2>Pitfalls</h2>
Während der Entwicklung der Beispielapplikation bin ich auf mehrere Fehler gestoßen, welche sich durch das Lesen der Dokumentation leider nicht lösen ließen und daher hier dokumentiert sind.
<h4>"Unknown or missing requestId in response."</h4>
Das Ergebnis der Recherche zeigt, dass hier entgegen der im Netz kursierenden Anleitungen die u2f-client-API falsch verwendet wird. Im Falle der Beispielapplikation sah der Code zunächst wie folgt aus:
<pre>u2f.register([req], [], cb);</pre>
Dem Aufruf der Funktion register jedoch muss die AppId als erster Parameter übergeben werden. Der korrekt funktionierende Aufruf gestaltet sich daher wie folgt:
<pre>u2f.register(req<strong>.appId</strong>, [req], [], cb);</pre>
<h4>"Error Code: 2"</h4>
Der Dokumentation ist zu entnehmen, dass bei einem Error Code 2 die AppId oder der Request falsch strukturiert sind. Zunächst ist auch hier wiederum darauf zu achten, dass die Funktion <code>u2f.register</code> korrekt verwendet wird. Besondere Beachtung gilt dem zweiten Parameter, welcher als Array übergeben werden muss. Ferner ist die AppId korrekt zu wählen: Zum einen ist es zwingend erforderlich, dass der Server über SSL, also HTTPS verfügt und die AppId entsprechend gestaltet wird: https://localhost. Ein Slash am Ende der URL führt ebenfalls zu ungewollten Seiteneffekten und Fehlern.