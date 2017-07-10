---
layout: post
title: "OCL.js"
subtitle: "Object Constraint Language im Browser"
language: de
tags:
 - OCL
 - OCL.js
 - JavaScript
 - Lexer
 - Parser
 - Library
---

1. This will become a table of contents (this text will be scraped).
{:toc}

## Einleitung

### Geschäftsprozesse
{:.no_toc}
Mein täglich Brot im Job ist das Arbeiten mit Modellen bzw. genauer: Geschäftsprozessmodellen.

> Ein Geschäftsprozess ist eine immer wiederkehrende Menge auszuführender Aktivitäten, welche sich auf ein bestimmtes Unternehmen und dessen Unternehmensziel bezieht.
> Zur Erreichung des Unternehmensziels müssen die gegebenen, knappen Ressourcen bspw. in Form von Mitarbeitern oder Rohstoffen eingesetzt werden, um einen für den Kunden
> relevanten Mehrwert (Output) zu erzeugen.  
> &ndash; Frank und van Laak[^1] und Richter-von Hagen und Stucky[^2]

Geschäftsprozessmodelle stellen daher die abstrakte Abbildung eines (oder mehrerer) Geschäftsprozesse dar.
Die Abbildung erfolgt über eine domänenspezifische Modellierungssprache (DSML), welche Konzepte und Beziehungen zwischen diesen darstellen kann.
Hierfür existieren eine Vielzahl an Sprachen um einen Geschäftsprozess abzubilden.
Die wohl bekannstesten sind die Event-driven Process Chain (EPC) sowie die Business Process Model and Notation (BPMN).

### Shortcomings von Modellierungssprachen
{:.no_toc}
Doch ähnlich wie die sehr bekannte Modellierungssprache UML haben auch Sprachen zur Geschäftsprozessmodellierung einen zentralen Nachteil:
Die Ausdrücksmöglichkeiten der jeweiligen Sprachen ist begrenzt, sodass verschiedene Sachverhalte nicht durch die verwendete Sprache allein abgebildet werden kann oder dies nur schwer möglich ist.

Das folgende Beispiel soll dies anhand folgender Konstellation verdeutlichen.
Abbildung 1 zeigt ein UML-Diagramm, welches beschreibt, dass eine Person keine oder mehrere Autos besitzen kann.
Ein Auto hat immer nur genau einen Fahrzeughalter.

{% include image.html file="uml-example.png" caption="Abb. 1: UML-Beispiel: Person besitzt Autos" %}

Neben den Attribute der jeweiligen Objekten `Person` und `Auto` ist keinerlei weitere Semantik dem Diagramm zu entnehmen.
Wie aber ist vorzugehen, wenn nun die Bedingung, dass eine Person mindestens 18 Jahre alt sein muss, um Autos zu besitzen abzubilden?

### OCL ftw!
{:.no_toc}
An dieser Stelle greift die OCL.
Sie ergänzt die gegebene Modellierungssprache um weitere Konzepte, welche _Constraints_ - also weitere Einschränkungen - abbilden können.

#### 1. Beispiel: Der Besitzer eines Autos muss älter als 18 Jahre sein
{:.no_toc}
Kommen wir zurück zu dem Beispiel, wo eine Person erst dann Autos besitzen darf, wenn diese über 18 Jahre alt ist.
Mit Hilfe der gegebenen Modellierungssprache ist diese Einschränkung nur schwer oder gar nicht umsetzbar, da für die Prüfung Instanzdaten benötigt werden, welche auf der abstrakten Modellebene noch nicht bekannt sind.

```ruby
context Person
    inv: self.age < 18 implies self.fleet->size() = 0
```

Die obige OCL-Regel legt eine Invariante für den _context_ Person fest, bezieht sich also auf ein Objekt des Typs _Person_.
Die Invariante legt fest, dass wenn eine Person jünger als 18 Jahre alt ist, die Fuhrparkgröße ebenfalls 0 sein muss.

#### 2. Beispiel: Eine Person kann nicht sein eigenes Elternteil sein
{:.no_toc}
Ein weiteres Beispiel stellt folgendes Diagram dar:

{% include image.html file="uml-example-2.png" caption="Abb. 2: UML-Beispiel: Eine Person hat zwei Elternteile" %}

Eine Person hat zwei Elternteile.
Was jedoch eine (biologische) Einschränkung darstellt ist, dass eine Person nicht sich selbst als Elternteil haben kann.
Es ist schwer - wenn nicht unmöglich - dies allein über die gegebenen Konzepte von UML zu lösen.
Mit Hilfe der folgenden, simplen OCL-Regel hingegen, wird die Einschränkung auf intuitive Weise festgehalten.

```ruby
context Person
    inv: self.parents->forAll(p | p <> self)
```

Diese OCL-Regel definiert erneut eine Invariante im _context_ Person, welche prüft, dass für alle Elemente in der Collection _parents_ gilt, dass diese nicht _self_ sind.

## Vom String zum AST
OCL ist eine textbasierte Sprache und wird in Form eines Strings in JavaScript definiert.
Jede OCL-Regel muss daher um von einem Interpreter ausgeführt zu werden zunächst in eine ausführbare Form gebracht werden.
Dies erfolgt über die klassische Verkettung von Lexer und Parser, welche einen String zerlegen und aus den zerlegten Einzelteilen einen Abstract-Syntax-Tree[^3] (AST) erzeugen.

{% include image.html file="vom-string-zum-ast.png" caption="Abb. 3: Vom String zum AST" %}

### Der Lexer
Ein Lexer übernimmt die Aufgabe die eingegebene OCL-Regel in einzelne Token zu zerlegen.
Hierfür wird im einzelnen definiert, welche Teile des eingegebenen Strings in Token übersetzt werden.
Das folgende Listing zeigt einen Auszug aus der OCL.js Lexer-Definition.

```text
\s+                                 /* ignore */
\-?[0-9][0-9]*                      return 'integer'
"context"                           return 'context'
"inv"                               return 'inv'
"->"                                return '->'
"("                                 return '('
")"                                 return ')'
"."                                 return '.'
":"                                 return ':'
[a-zA-Z][a-zA-Z0-9]*                return 'simpleName'
">"                                 return '>'
```

Mit Hilfe der oben angeführten Lexer-Definition, kann folgende OCL-Regel in einzelne Token zerlegt werden.

```ruby
context Car inv:
    self.licensePlateNumber->size() > 0
```

Wurde die OCL-Regel in Token zerlegt, ergibt sich folgendes Bild:

```text
context simpleName inv:				
    simpleName.simpleName->simpleName() > integer
```

### Der Parser
Die Aufgabe des Parsers ist nun, die einzelnen Token gemäß einer gegebenen Grammatik in einen Abstract-Syntax-Tree zu wandeln.
Einen Auszug der OCL-Grammatik ist im folgenden Listing zu sehen.

```text
classifierContextDecl
    : 'context' simpleName inv
        { $$ = new ContextExpression($2, $3) }
    ;

inv
    : 'inv' simpleNameOptional ':' oclExpression
        { $$ = new InvariantExpression($4, $2) }
    ;
oclExpression
    : pathName
        { $$ = new Expression.VariableExpression($1) }
    | oclExpression '.' simpleName
        { $$ = new Expression.VariableExpression([$1.variable, $3])
    | oclExpression '->' simpleName
        { $$ = functionCallExpression($3, $$); }
    ;
```

Trifft eine der gegebenen Grammatikregeln auf die kombination der gegebenen Token zu, wird entsprechend eine Expression instanziiert.
Das nun folgende Listing zeigt den AST in Form der einzelnen Expressions.
Abbildung 4 zeigt den AST hingegen in Form eines Baumdiagramms.

```text
ContextExpression[name:”Car”]
  - InvariantExpression[name:””]
     - OperationCallExpression[operator:”>”]
        - SizeExpression[name=”size”]
           - VariableExpression[variable:“self.licensePlateNumber”]
        - NumberExpression[number:”0”]

```

Die einzelnen Expressions / Bestandteile des AST folgem dem Interpreter-Pattern[^4] und implementieren jeweils eine `evaluate()`-Funktion.

{% include image.html file="ast.png" caption="Abb. 4: Resultierender AST" %}


## Links und weitere Infos
Die beste Dokumentation ist der Code.
Zu finden ist OCL.js auf GitHub im Repository [SteKoe/OCL.js](https://github.com/SteKoe/ocl.js).
Eine kurze Dokumentation in Schriftform ist [hier](https://ocl.stekoe.de/) zu finden. 

[^1]: Frank U, van Laak B (2003), Anforderungen an Sprachen zur Modellierung von Geschäftsprozessen.   
[^2]: Richter-von Hagen C, Stucky W (2004), Business-Process- und Workflow-Management: Prozessverbesserung durch Prozess-Management.
[^3]: https://de.wikipedia.org/wiki/Abstrakter_Syntaxbaum
[^4]: https://de.wikipedia.org/wiki/Interpreter_(Entwurfsmuster)