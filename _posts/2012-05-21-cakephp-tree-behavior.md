---
layout: post
title: "CakePHP: Tree Behavior"
date: 2012-05-21
tags:
---
Im einführenden Beitrag zu CakePHP habe ich bereits kurz über das Tree Behavior gesprochen und kurz angedeutet, wofür dies verwendet werden kann. Da dieses Behavior zum einen ein Kern Behavior ist (also bei CakePHP standardmäßig dabei), aber auch regen Gebrauch findet, widme ich diesem teilweise komplizierten Behavior einen dezidierten Beitrag mit Anwendungsbeispiel und ausführlichen Erklärungen.

<!--more-->

Das Tree Behavior findet immer dann Anwendung, wenn eine sortierte oder verschachtelte Liste (nested set) in einer Datenbank gespeichert werden soll. Hierfür muss das Tree Behavior zum einen an das jeweilige Modell angehängt werden

[code]public $actsAs = array('Tree');[/code]

und die Datenbank vorbereitet werden, da das Tree Behavior drei zusätzliche Felder benötigt. Dies sind <em>parent_id</em>, <em>lft</em> (left) und <em>rght</em> (right).

Das Feld <em>parent_id</em> adressiert das übergeordnete Element in einer verschachtelten Liste oder <em>NULL,</em> wenn das aktuelle Element auf oberster Ebene sein soll. Die Felder <em>lft</em> und <em>rght</em> dienen dazu die Reihenfolge der Elemente festzulegen. Ebenso ist durch die beiden Felder eine einfache Suche und Abfrage über die Liste möglich (warum dies so ist, wird später deutlich).

Der genaue Nutzen der Felder <em>lft </em>und <em>rght</em> wird deutlich, wenn man sich die Theorie dazu anschaut: Das Tree Behavior stützt sich auf das in der Informatik bewährte Konzept des Tiefensuchbaums oder auch <a href="http://en.wikipedia.org/wiki/Depth-first_search" target="_blank">Depth First Search (DFS)</a>.
<h2>Anwendungsbeispiel</h2>
Da man eine Webseitennavigation als verschachtelte Liste auffassen kann, bauen wir unser Beispiel auf folgende Liste auf:
<ul>
	<li>Home</li>
	<li>Universities</li>
<ul>
	<li>University of Duisurg-Essen</li>
	<li>University of Köln</li>
	<li>University of Munich</li>
</ul>
	<li>Impress</li>
	<li>Team</li>
<ul>
	<li>Administrators</li>
	<li>Coordinators</li>
	<li>Content-Assistant</li>
</ul>
</ul>
Die Elemente <em>Home</em>, <em>Universities</em>, <em>Impress</em> und <em>Team</em> befinden sich auf oberster Ebene der Liste und haben daher keine Elternknoten. Das Feld <em>parent_id</em> ist <em>NULL </em>(vgl. Tabelle unten). Überträgt man <em>alle</em> Elemente in eine Tabelle mit <em>Datensatz-ID</em>, <em>Titel</em> und <em>parent_id</em> Feld, erhält man folgendes Ergebnis:
<table class="datalist sortable">
<thead>
<tr>
<th align="center">id</th>
<th>title</th>
<th align="center">parent_id</th>
<th align="center">lft</th>
<th align="center">rght</th>
</tr>
</thead>
<tbody>
<tr>
<td align="center">1</td>
<td>Home</td>
<td align="center">NULL</td>
<td align="center">1</td>
<td align="center">2</td>
</tr>
<tr>
<td align="center">2</td>
<td>Universities</td>
<td align="center">NULL</td>
<td align="center">3</td>
<td align="center">10</td>
</tr>
<tr>
<td align="center">3</td>
<td>Impress</td>
<td align="center">NULL</td>
<td align="center">11</td>
<td align="center">12</td>
</tr>
<tr>
<td align="center">4</td>
<td>Team</td>
<td align="center">NULL</td>
<td align="center">13</td>
<td align="center">20</td>
</tr>
<tr>
<td align="center">5</td>
<td>University of Duisburg-Essen</td>
<td align="center">2</td>
<td align="center">4</td>
<td align="center">5</td>
</tr>
<tr>
<td align="center">6</td>
<td>University of Cologne</td>
<td align="center">2</td>
<td align="center">6</td>
<td align="center">7</td>
</tr>
<tr>
<td align="center">7</td>
<td>University of Munich</td>
<td align="center">2</td>
<td align="center">8</td>
<td align="center">9</td>
</tr>
<tr>
<td align="center">8</td>
<td>Administrators</td>
<td align="center">4</td>
<td align="center">14</td>
<td align="center">15</td>
</tr>
<tr>
<td align="center">9</td>
<td>Coordinators</td>
<td align="center">4</td>
<td align="center">16</td>
<td align="center">17</td>
</tr>
<tr>
<td align="center">10</td>
<td>Content-Assistants</td>
<td align="center">4</td>
<td align="center">18</td>
<td align="center">19</td>
</tr>
</tbody>
</table>
Doch wie kommen die Felder <em>lft </em>und <em>rght</em> zustande? Dies ist nicht ganz einfach zu erklären. Zunächst muss man die verschachtelte Liste als Baum aufzeichnen (siehe Bild). Ausgehend von der Wurzel (<em>NULL)</em>, welche selbst nicht explizit in der Datenbank gespeichert wird, muss man jeweils den Kind-Knoten "besuchen", welcher noch nicht besucht wurde und am weitesten links liegt. Um dies zu verdeutlichen durchlaufen wir ein paar Elemente des Baumes:

Der erste linke nicht besuchte Knoten ist "Home". Navigiert man nun also von <em>NULL </em>zu Home, muss im Feld <em>lft</em> eine 1 eingetragen werden, da man erst einen Schritt gegangen ist. Home selbst hat keine Kind-Knoten, sodass man nun zum Elternknoten zurückgehen muss. Hierfür muss jedoch immer der Umweg über sich selbst genommen werden. Dieser Umweg wird ebenfalls als ein Schritt gezählt und die Anzahl im Feld <em>rght</em> festgehalten. Daraus resultiert der Wert 2 für das Feld <em>rght</em> des Home-Knotens.

<a class="lightbox" href="http://www.systemoutprintln.de/wp-content/uploads/Treebehavior.png"><img class="alignleft size-thumbnail wp-image-835" title="Treebehavior" src="http://www.systemoutprintln.de/wp-content/uploads/Treebehavior-150x150.png" alt="" width="150" height="150" /></a>Gehen wir weiter, gelangt man zum Knoten "Universities". Das ist der dritte Weg, den man gehen musste, also bekommt der Knoten "Universities" im Feld <em>lft</em> den Wert 3. Universities hat Kinderknoten. Hier muss jeweils wieder der am weitesten links gelegene gewählt werden: University of Duisburg-Essen. Wenn man dort hingeht, sind wir 4 Wege gegangen. <em>lft</em> ist also 4 für diesen Kind-Knoten. Der Knoten selbst hat keine Kinder und man muss erneut den Umweg über ihn selbst gehen, <em>rght</em> ist daher 5. – Das Prinzip sollte nun klar sein.

Sortiert man die Einträge anhand des Feldes <em>lft</em> oder <em>rght</em>, kann eine sortierbare bzw. sortierte Liste realisiert werden. Möchte man bspw. Impressum und Team tauschen, so ändern sich die <em>lft</em> und <em>rght</em>-Werte für Impress, Team und alle Unterknoten von Team. Da dies automatisch vom Framework über das Tree Behavior berechnet wird, muss man sich jedoch keine Gedanken machen. Es reicht der Aufruf der Methode <em>moveUp($id)</em> und das Framework berechnet alle Werte selbstständig neu.

Durch diese beiden Felder ist es zum Beispiel auch möglich alle Kinder-Knoten eines Elternknotens zu ermitteln: Gebe mir alle Knoten zurück, wo die <em>lft-</em>Werte größer oder gleich dem <em>lft-</em>Wert und die <em>rght-</em>Werte kleiner oder gleich dem <em>rght-</em>Wert des Elternknotens sind.