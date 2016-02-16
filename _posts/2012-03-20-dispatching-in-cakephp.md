---
layout: post
title: "Dispatching in CakePHP"
date: 2012-03-20
tags:
---
Bei intensiver Auseinandersetzung mit CakePHP stellt sich – zumindest mir – nach einiger Zeit die Frage nach dem "wie wird was, wann ausgeführt?". Dies ist vor allem dann relevant, wenn man eigene Komponenten, Behavior oder Helper programmiert. Diesen Vorgang nennt man auch Dispatching des Frameworks.

Im Endeffekt stößt der Besucher einer mit CakePHP entwickelten Seite "nur" ein PHP-Script an, welches dann die Ausführung des Frameworks initiiert. CakePHP ist in weiten Teilen objekt-orientiert Entwickelt und arbeitet mit verschiedensten Klassen. Auf ein Objekt-Relationales-Mapping wurde bisher jedoch verzichtet. Das heißt, dass die Felder einer Datenbanktabelle nicht in Form von Instanzvariablen in einem Objekt bereit gehalten werden, sondern in Form von Arrays. Die Schlüssel des Arrays entsprechen dann den Feldern in der Datenbank.

Doch zurück zum Dispatching: Oftmals ist es wichtig zu wissen, welche Teile des Frameworks wie aufgerufen werden. Statt dass man sich die jeweiligen Aufrufe in der Dokumentation heraussucht, habe ich ein kleines Sequenzdiagramm erstellt, welches den Aufruf der wichtigsten Methoden vom Controller, Component und View darstellt.

<!--more-->

![Dispatching CakePHP](Dispatching.png)