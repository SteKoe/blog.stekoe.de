---
layout: post
title: "GMF Editor: Bind delete-key to \"delete from model\" action"
date: 2011-06-24
tags:
 - GMF
 - GEF
 - Eclipse
 - EMF
---
Im Rahmen meiner Tätigkeit rund um die Entwicklung eines Diagramm-Editors mit Hilfe der Eclipse Framworks EMF, GEF und GMF trat ein Feature-Request auf, welcher nicht leicht zu implementieren ist: Das Löschen von Modellelementen über das Diagramm (allgemein bekannt unter "Delete from Model") mit Hilfe der Delete-Taste bzw.
der Backspace-Taste.
Die Eclipse-Newsgroups sind voll an Nachfragen zur Umsetzung dieser Problematik.
Das einzige was ausbleibt ist natürlich eine Antwort.
Doch die gibt es jetzt in diesem Artikel.

<!--more-->

Wie in vielen Fällen ist die eigentliche Implementierung weniger kompliziert, als das Finden der richtigen Stelle.
DiagrammEditoren, im speziellen der <tt>org.eclipse.gmf.runtime.diagram.ui.parts.DiagramEditor</tt>, von welchem die bei uns verwendeten Editoren erben, definieren bereits einige Standard Key-Bindings die man in der eigenen Implementierung überschreiben kann.
Die vordefinierten Bindings sind in der Methode <tt>getKeyHandler()</tt> (<a href="http://publib.boulder.ibm.com/infocenter/rsmhelp/v7r0m0/topic/org.eclipse.gmf.doc/reference/api/runtime/org/eclipse/gmf/runtime/diagram/ui/parts/DiagramEditor.html#getKeyHandler()">Link zur API</a>) definiert.
Zwar kann die Methode nicht direkt überschrieben werden, da diese als protected deklariert ist, aber man kann in der eigenen Implementierung eines Diagramm-Editors (<tt>XXXDiagramEditor extends DiagramEditor</tt>) innerhalb der Methode <tt>configureGraphicalViewer()</tt> auf die bereits defnierten Key-Bindings zugreifen.
Da bei der Initialisierung des <tt>XXXDiagrammEditors</tt> zunächst im Supertyp <tt>DiagramEditor</tt> die Standard Key-Bindings festgelegt werden, können diese abgefragt und manipuliert werden.

Mit den Methoden <tt>getDiagramGraphicalViewer().getKeyHandler()</tt> werden die für den aktuellen Diagramm-Editor registrierten Key-Handler zurückgegeben.
Nun ist es möglich die Key-Bindings zu manipulieren und an eigene Bedürfnisse anzupassen, so dass der Delete-Key eine vom Standard abweichende <tt>IAction</tt> aufruft.
Hier ist ein kleines Code-Beispiel:

{% gist b68dd80e6ec2a0725664 %}

Dieses Beispiel legt fest, dass die Keys Delete und Backspace die <tt>IAction</tt>  "delete from model" aufrufen.
&mdash; Im Prinzip war dies auch bereits die gesamte Arbeit, um das Key-Binding von "delete from diagram" auf "delete from model" zu mappen.
Das einzige Problem welches sich an dieser Stelle auftut ist, dass kein Keyboard-Shortcut mehr für die Aktion "delete from diagramm" existiert.
Das löschen von Diagrammelementen funktioniert hingegen jedoch wie gewohnt weiterhin über das Kontextmenü.