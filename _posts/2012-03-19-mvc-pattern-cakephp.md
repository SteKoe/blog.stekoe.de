---
layout: post
title: "CakePHPs MVC-Pattern"
date: 2012-03-19
tags:
---
MVC-Pattern sind in aller Munde und bieten eine gute Möglichkeit den logischen Part einer Anwendung von Darstellung und Datenhaltung zu trennen. Auch das kostenfreie unter der <a href="http://de.wikipedia.org/wiki/MIT-Lizenz" target="_blank">MIT-Lizenz</a> angebotene PHP-Framework <strong>CakePHP</strong> setzt auf das MVC-Muster. Für unbedarfte Nutzer mag es ein wenig irritierend und fraglich sein, warum die MVC-Splittung im Endeffekt so wichtig ist. Aber es lohnt sich in den meisten Fällen und es ist nicht schwer die Denkweise zu "erlernen".

<!--more-->Die grobe Idee hinter MVC besteht darin Code in logische Segmente zu zerlegen um so Wiederverwendung, Übersichtlichkeit und Aufgabenteilung zu fördern. MVC ist eine Abkürzung und steht für "Model-View-Controller", ein eng zusammenarbeitendes, (fast) unzertrennliches Tripple. Im folgenden werden die einzelnen Akteure etwas genauer beleuchtet:
## MVC-Pattern kurz erklärt
<strong>Model: </strong>Der überwiegende Teil der Webseiten arbeitet im Hintergrund mit einer Datenbank in welcher alle wichtigen Daten gespeichert werden. Dies sind Postings, Benutzerdaten, Webseiteninformationen wie Titel, URL, etc. pp. – Um eine Schnittstelle zwischen Datenbank und Webanwendung zu erleichtern, bieten viele Frameworks eine Abstraktion von der jeweilige Datenbank in Form eines Modells an. Für jede Tabelle in der Datenbank existiert ein Pendant in der jeweiligen Software als Modell, welches alle Felder der Datenbanktabelle enthält.

<strong>View:</strong> Die View ist die Repräsentationsschicht des MVC-Patterns. Sie stellt die an sie übergebenen Daten grafisch in Form einer GUI (Graphical User Interface, Grafische Benutzeroberfläche) dar und ist bei Webanwendungen nichts anderes als ein HTML-Gerüst mit Platzhaltern (Variablen) für den dynamischen Inhalt.

<strong>Controller:</strong> Die Daten, Anfragen und Berechnungen (Algorithmen) einer Webanwendung wollen natürlich auch noch berücksichtigt werden und brauchen ihren Platz. Da Berechnungen nichts mit Datenhaltung zu tun haben, sondern Daten benötigen, gehören sie nicht in das Modell. Ferner geben Algorithmen nur verarbeitete Daten zurück, sodass sie auch nicht in die View gehören. Vielmehr wurde eine weitere Zwischenschicht eingeführt, der Controller. Dieser bezieht seine Daten aus dem Modell (also der Datenbankabstraktionsschicht), verarbeitet diese nach Wünschen des Programmierers und gibt die Ergebnisse an die View weiter um sie anzuzeigen. Aber auch Eingaben die über die View (in Formularen) getätigt werden, können im Controller abgegriffen und verarbeitet werden.
## MVC in CakePHP
![CakePHP MVC Pattern](CakePHP_MVC.png)
Es gibt keine standardisierte Vorgabe wie das MVC-Pattern zu implementieren ist und daher setzen es die verschiedenen Frameworks leicht differierend um. CakePHP bietet zusätzlich zu den drei Schichten des MVC-Patterns weitere Konzepte an und erweitert damit die Flexibilität und Modularität des Codes. Im Folgenden werden (analog zur linken Grafik) die wichtigsten Konzepte des MVC-Patterns von CakePHP näher erläutert.

### Modelle - Behavior
Behavior sind ein probates Mittel, wenn es darum geht einzelnen Modellen spezielle Eigenschaften oder Funktionsweisen zu implementieren. Möchte man beispielsweise eine Baumstruktur abspeichern (verschachtelte Listen, Seitennavigation, ...), so existiert hierfür ein Tree-Behavior, welches Methoden anbietet die eine Sortierung der verschachtelten Listen (und deren Elemente) ermöglicht und weitere.

### Views - Elemente und Helper
Oftmals werden in Views einzelne Teile redundant verwendet. Wenn eine Tabelle zur Auflistung von Daten an mehreren Stellen benutzt wird, wäre ein naiver Ansatz diese Tabelle via Copy &amp; Paste zu übernehmen. Sollte jedoch anschließend ein Fehler im Code gefunden werden, muss dieser an allen Stellen geändert werden, an denen die Tabelle auftaucht. Hier folgt man besser dem DRY-Prinzip: Don't Repeat Yourself (Wiederhol' dich nicht!) – CakePHP bietet den Entwicklern die Möglichkeit HTML-Code-Fragmente – wie bspw. eine Tabelle – auszulagern und an beliebiger Stelle in einer View zu inkludieren. Diese ausgelagerten Code-Fragmente werden "Element" genannt und können beliebig erstellt werden.

Ferner bietet CakePHP die Verwendung von "Helpern" an. Heller sind ausgelagerte PHP-Algorithmen, die meist HTML-Code ausgeben. In CakePHP werden oftmals Formulare nicht händisch, sondern mit Hilfe eines FormHelpers erstellt. Dieser erzeugt sauberen HTML-Code und kann zusätzlich anhand der ihm übergebenen Daten feststellen, welcher Input-Typ ein Feld besitzen soll. Helper sind also wesentlich komplexer als Elemente und enthalten meist viel logischen Code.

### Controller - Components
Wie bereits in der kurzen Einführung beschrieben verarbeitet der Controller alle Daten einer Anwendung und lenkt diese je nach Verwendung zur View oder in die Datenbank. Auch dort kann es vorkommen, dass bestimmte Algorithmen mehrfach verwendet werden sollen. Solange dies in einem Controller geschieht kann die Redundanz über das Auslagern in eine eigenständige Methode verhindert werden. Sobald jedoch ein Algorithmus in mehreren Controllern verwendet werden soll, kann man diesen entweder in eine Superklasse (AppController) auslagern oder – um die Übersicht zu wahren – eine Komponente schreiben und diese in den jeweiligen Controllern einbinden.

### Übrige Module
Das obige Bild zeigt neben den genannten Modulen weitere Module im unteren Bereich: CakeSession, Cache und Vendors.

Die CakeSession zeigt eindrucksvoll, dass es auch möglich ist über eine Superklasse die gleichen Daten in einer Component sowie einem Helper zu halten. Die in einer Session gespeicherten Daten sollen sowohl in der View abfragbar sein aber auch im Controller. Da die Daten konsistent sein müssen, wird hier über eine Superklasse eine gemeinsame Basis geschaffen.

Caching wird im Controller angeboten und ermöglicht es die Last vom Server zu reduzieren. Im Controller können bestimmte Inhalte in den Cache geschrieben werden. Jedoch ist auch lesen und löschen möglich.

Vendors sind 3rd-Party Klassen, die nicht speziell für die Verwendung in CakePHP abgestimmt wurden. Vendors können einfach in jedem Controller inkludiert und verwendet werden.

## Fazit
Diese kurze Einführung in das MVC-Pattern von CakePHP ist natürlich nicht abschließend. CakePHP bietet selbst eine sehr umfangreiche und auch gute <a href="http://book.cakephp.org/" target="_blank">Dokumentation</a>. Durch meine Arbeit mit diesem Framework folgen noch weitere kurze erklärende Beiträge zu diversen Aspekten des Frameworks. Für heute soll es jedoch an dieser Stelle reichen.