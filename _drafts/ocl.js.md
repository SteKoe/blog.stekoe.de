---
layout: post
title: "OCL.js"
subtitle: "Object Constraint Language im Browser "
tags:
 - OCL
 - Library
---
## Geschäftsprozesse
Mein täglich Brot im Job ist das Arbeiten mit Modellen bzw. genauer: Geschäftsprozessmodellen.

> Ein Geschäftsprozess ist eine immer wiederkehrende Menge auszuführender Aktivitäten, welche sich auf ein bestimmtes Unternehmen und dessen Unternehmensziel bezieht.
 Zur Erreichung des Unternehmensziels müssen die gegebenen, knappen Ressourcen bspw. in Form von Mitarbeitern oder Rohstoffen eingesetzt werden, um einen für den Kunden 
 relevanten Mehrwert (Output) zu erzeugen.
 
Geschäftsprozessmodelle stellen daher die abstrakte Abbildung eines (oder mehrerer) Geschäftsprozesse dar.
Die Abbildung erfolgt über eine domänenspezifische Modellierungssprache (DSML), welche Konzepte und Beziehungen zwischen diesen darstellen kann.
Hierfür existieren eine Vielzahl an Sprachen um einen Geschäftsprozess abzubilden.
Die wohl bekannstesten sind die Event-driven Process Chain (EPC) sowie die Business Process Model and Notation (BPMN).

## Shortcomings von Modellierungssprachen
Doch ähnlich wie die sehr bekannte Modellierungssprache UML haben auch Sprachen zur Geschäftsprozessmodellierung einen zentralen Nachteil:
Die Ausdrücksmöglichkeiten der jeweiligen Sprachen ist begrenzt, sodass verschiedene Sachverhalte nicht durch die verwendete Sprache allein abgebildet werden kann oder dies nur schwer möglich ist.

Das folgende Beispiel soll dies anhand folgender Konstellation verdeutlichen.
Abbildung 1 zeigt ein UML-Diagramm, welches beschreibt, dass eine Person keins oder mehrere Autos besitzen kann.
Ein Auto hat immer nur genau einen Fahrzeughalter.
{% include image.html file="uml-example.png" caption="Abb. 1: UML-Beispiel: Person besitzt Autos" %}

Neben den Attribute der jeweiligen Objekten `Person` und `Auto` ist keinerlei weitere Semantik dem Diagramm zu entnehmen.
Wie aber ist vorzugehen, wenn nun die Bedingung, dass eine Person mindestens 18 Jahre alt sein muss, um Autos zu besitzen abzubilden?

## OCL ftw!
An dieser Stelle greift die OCL.
Sie ergänzt die gegebene Modellierungssprache um weitere Konzepte, welche _Constraints_ - also weitere Einschränkungen - abbilden können.

### 1. Beispiel: Der Besitzer eines Autos muss älter als 18 Jahre sein
Kommen wir zurück zu dem Beispiel, wo eine Person erst dann Autos besitzen darf, wenn diese über 18 Jahre alt ist.
Mit Hilfe der gegebenen Modellierungssprache ist diese Einschränkung nur schwer oder gar nicht umsetzbar, da für die Prüfung Instanzdaten benötigt werden, welche auf der abstrakten Modellebene noch nicht bekannt sind.

```ruby
context Person
    inv: self.age < 18 implies self.fleet->size() = 0
```



### 2. Beispiel: Eine Person kann nicht sein eigenes Elternteil sein
{% include image.html file="uml-example-2.png" caption="Abb. 2: UML-Beispiel: Eine Person hat zwei Elternteile" %}

```ruby
context Person
    inv: self.parents->forAll(p | p <> self)
```