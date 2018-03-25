---
layout: post
title: "Angular CLI-Build in Spring Boot Maven-Lifecycle integrieren"
image: true
language: de
tags:
---

> Zu diesen Beitrag existiert ein [GitHub Repository](https://github.com/SteKoe/spring-boot-angular/tree/1-initial-spring-boot-project), welches die einzelnen Abschnitte als Tag gepflegt hat.

Vor einiger Zeit habe ich mir vorgenommen eine Applikation in Angular zu entwickeln, die mit Hilfe von Spring Boot ausgeliefert wird.
Ziel der Übung war es die benötigten CLI Commands für `ng` in den Maven-Lifecycle[^1] so zu integrieren, dass beim Aufruf von `mvn clean package` eine Spring Boot Applikation herausfällt, die eine transpilierte, also mit `ng build --prod` gebaute SPA enthält.
In diesem Blogpost beschreibe ich das grundlegende Vorgehen und lege meine Überlegungen diesbezüglich offen und begründe auch einen Teil meiner Entscheidungen entsprechend.

## 1. Neues Spring Boot Web Projekt anlegen
Um ein neues Spring Boot Web Projekt zu erzeugen habe ich zum Spring Initializrs[^2] gegriffen und mir ein Spring 2.0.0 Projekt generieren lassen.
Als _dependency_ habe ich ausschließlich `Web` eingegeben und anschließend das resultierende ZIP-File als Basis verwendet.
Führt man auf diesem Projekt `mvn spring-boot:run` aus, wird die Spring Boot Applikation unter Port 8080 gestartet und kann im Browser aufgerufen werden.
Da bisher keine HTML-Seite hinterlegt wurde, wird eine vordefinierte Fehlerseite ausgegeben.

## 2. Erste statische Inhalte ausliefern
Spring Boot Web erlaubt es auf verschiedenste Arten statischen Inhalt auszuliefern.
Der einfachste Weg ist es, unter `src/main/resources` einen Ordner anzulegen, der `static` heißt.
In diesem Ordner habe ich testweise eine HTML-Datei `index.html` angelegt, welche den Klassiker `Hello, World!` ausgibt.
Startet man die Applikation nun neu, wird die `index.html` automatisch von Spring Boot als Startseite ausgeliefert und entsprechend im Browser angezeigt.

Wirft man einen Blick in das `target` Verzeichnis des Projektes, findet sich die `index.html`-Datei unter `classes/static` wieder.
Wollen wir also eine Angular Applikation von Spring Boot ausliefern lassen, muss die App nach `classes/static` transpiliert werden.
Im Umkehrschluss heißt das aber auch, dass unsere Angular Sourcen nicht im Ordner `src/main/static` liegen sollten, da diese ansonsten untranspiliert in das JAR einfließen und sie es nur unnötig aufblähen.
Ich verwende daher den Ordner `src/main/webapp` als Basisordner, da dieser semantisch genau das ausdrückt, was die Angular App darstellt und dieser Ordner nach Maven-Konvention genau hierfür vorgesehen ist.

**Anmerkung:** An dieser Stelle sei angemerkt, dass das `webapp`-Verzeichnis nur dann verwendet werden sollte, wenn das maven-war-plugin[^3] nicht verwendet wird, da dies sonst davon ausgeht, dass sich in dem `webapp`-Ordner eine JSP/Java-basierte Webapplikaton befindet und versucht diese zu kompilieren.

## 3. Erzeugen eines initialen Angular CLI-Projekts
Nun da klar ist, dass sich die Angular App Sourcen im `src/main/webapp` Ordner befinden sollen, erzeuge ich entsprechend dort eine neue Angular App über die CLI `ng new webapp`.
In dem neu erzeugten Angular Projekt kann nun wie gewohnt mit der Entwicklung der SPA angefangen werden und diese auch separat vom Spring Boot Projekt entwickelt werden.

## 4. Ausliefern der Angular App
Führt man den Befehl `mvn clean package` aus und schaut in das `target` Verzeichnis, sieht man, dass dort noch keine Spur der Angular App zu finden ist, da das `webapp`-Verzeichnis nicht wie das `static`-Verzeichnis automatisch im `target`-Ordner landet.
Ziel ist es also nun, die Angular App wie in Schritt 2 beschrieben in den Ordner `classes/static` zu transpilieren.
Hierfür reicht es aus in der `angular-cli.json` das Property `outDir` so anpassen, dass es auf das `target`-Verzeichnis zeigt: `../../../target/classes/static`.
Führt man nun `ng build` aus, wird die Angular App in das `static`-Verzeichnis geschrieben und nach einem Neustart der Spring Boot Applikation auch ausgeliefert.

## 5. Automatisieren der Schritte
Wenn man nun das Projekt ausschließlich mit Hilfe von Maven bauen will, muss natürlich auch das `ng build` Command in den Build-Lifecycle von Maven integriert werden.
Dies hat mehrere Vorteile:
Zum einen muss man als Entwickler nicht mehrere Befehle nacheinander ausführen (`mvn package && ng build && maven spring-boot:run`), zum anderen kann die Entwicklung beider Projekte transparent voneinander stattfinden.
Als Java Entwickler brauche ich mich nicht mit den Angular CLI-Befehlen auseinanderzusetzen, da ich meine gewohnten Maven-Befehle benutzen kann und völlig transparent im Hintergrund die Angular CLI-Befehle ausgeführt werden und als Angular Entwickler kann ich mich ausschließlich auf den Ordner `src/main/webapp` konzentrieren und hier meine Angular CLI wie gewohnt nutzen.

Im Hinblick auf die Buildpipeline des Projekts bietet es sich an, die Abhängigkeiten zu benötigten Tools wie Maven, Java, Node, npm und Angular-CLI so weit zu reduzieren wie möglich.
Statt also auf ein global installiertes Angular CLI zu bauen, verwende ich das unter den `node_modules` befindliche Angular CLI-Paket.
Mit Hilfe des Befehls `node node_modules/@angular/cli/bin/ng build --prod` erreiche ich dasselbe Ergebnis wie mit dem Befehl `ng build --prod` mit dem Unterschied, dass ich die Angular CLI im ersten Fall nicht vorher mit dem Befehl `npm install -g @angular/cli` habe installieren müssen.
Für Entwickler, die mit Node, npm oder Angular nicht sehr vertraut sind, wäre es mit diesem Ansatz nun völlig ausreichend Node.JS zu installieren, was genau wie Maven und Java sowieso benötigt würde - der Aufwand neue Tools zu lernen und die Rüstzeit entfallen bzw. werden hierdurch auf ein Minimum reduziert.

Nun bleibt noch die Frage, wann und wie diese Befehle auszuführen sind.

### Welche Lifecycle-Phasen sollen verwendet werden?
Nach Durchsicht des Maven-Lifecycles scheinen sich die Phasen `verify` und `compile` anzubieten.
Die Phase `verify` gewährlistet, dass das Projekt korrekt ist und alle erforderlichen Informationen und Abhängigkeiten vorhanden sind.
In dieser Phase kann also `npm install` eingebaut werden, damit bei fehlenden oder falschen Dependencies im Angular Projekt der Build gemäß des Prinzips "fail fast" frühzeitig abbricht.
Das Transpilieren der Angular App hingegen implementiere ich in die Phase `compile`, da hier auch die Java Klassen kompiliert werden (`compile the source code of the project`).

**Achtung:** Wird mit dem spring-boot-plugin gearbeitet (bspw. `mvn spring-boot:run`), muss das `npm install` in eine Phase zwischen `generate-resources` und `compile` integriert werden, da das spring-boot-plugin nicht nochmal die Phase `verify` aufruft.
Im verlinkten GitHub Repository ist daher der `npm install`-Befehl in die Phase `generate-resources` eingebunden.

### Wie können die erforderlichen Befehle zu einer bestimmten Phase ausgeführt werden?
Um sich in den Maven-Build einzuhängen existiert für das Ausführen von Befehlen das Maven Plugin `exec-maven-plugin`[^4].
Über dieses Plugin werden `executions` definiert, welche zu einer konfigurierbaren `phase` in der Reihenfolge ausgeführt werden, wie sie in der `pom.xml` definiert sind.
Daher werden zwei `executions` definiert: `npm-install` und `ng-build`, welche entsprechend mit den Commands `npm install` bzw. `nb build --prod` konfiguriert werden.

{% gist e41efe09abeaf216841a8c887214a231 npm-install.pom.xml %}
{% gist e41efe09abeaf216841a8c887214a231 ng-build.pom.xml %}

Führt man nun, nachdem das `exec-maven-plugin` eingebunden und konfiguriert wurde, `mvn clean package` aus, erzeugt Maven ein JAR-File im `target`-Verzeichnis welches man mit Hilfe des Befehls `java -jar <filename>.jar` ausführen kann.
Durch die skizzierte Integration der Angular CLI in den Maven-Lifecycle erhalten wir eine Spring Boot Applikation die über einen integrierten Tomcat unsere Angular App ausliefert.
Der Buildlifecycle hierbei ist völlig transparent!

## 6. Noch einen Schritt weiter: Tests
Zwar ist das eigentliche Ziel meines Vorhabens bereits erreicht, doch möchte ich nun auch noch ein weiteres Detail in den Buildlifecycle einbauen: Tests!
Angular erlaubt es Tests zu schreiben und mit Hilfe des Commands `ng test` auszuführen.
Wie in den vorherigen Schritten gezeigt ist es ein Leichtes diesen Aufruf auch noch einzubauen.
Maven bietet die dedizierte Phase `test` an, in welcher man `ng test -sr` implementiert habe.
Das Flat `-sr` steht für `single-run` und führt die Tests einmalig aus.

{% gist e41efe09abeaf216841a8c887214a231 ng-test.pom.xml %}

[^1]: [Maven Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)
[^2]: [Spring Initializt](https://start.spring.io)
[^3]: [Maven War Plugin](https://maven.apache.org/plugins/maven-war-plugin/usage.html)
[^4]: [Maven Exec Plugin](http://www.mojohaus.org/exec-maven-plugin/)