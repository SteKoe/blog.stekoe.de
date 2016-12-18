---
title: "Jenkins CI - Integrationstests mit Docker"
layout: post
tags:
 - Jenkins
 - Docker
 - Integrationstests
---
Jenkins ist eines der weltweit führenden Werkzeuge das sich auf Continuous Integration (CI) fokussiert. CI ist ein etabliertes Vorgehensmuster, welches Codeänderungen nachdem diese in ein Source-Code-Repository überspielt wurden automatisiert kompiliert und testet. Auch bei GBTEC werden verschiedene Instanzen von Jenkins CI eingesetzt, um den Source-Code unserer Produkte zu kompilieren und einer Qualitätskontrolle zu unterziehen. Um die Instanzen schnellstmöglich aufsetzen zu können, wird hierfür Docker verwendet. In einem entsprechenden Dockerfile werden die Jenkins-Version, benötigte Plugins sowie verschiedene Groovy-Scripts eingesetzt, um eine schnelle Installation einer neuen Jenkins-Instanz zu ermöglichen. In diesem Artikel wird vorgestellt, wie mit Hilfe von Jenkins und Docker Datenbanken für Integrationstests aufgesetzt und anschließend wieder entfernt werden.

<!--more-->
<h2>Docker-Umgebung in Jenkins anmelden</h2>
Um innerhalb von Jenkins Docker Container starten zu können, muss Jenkins Zugriff auf einen Docker-Host bekommen. Dies wird erreicht, indem sowohl der Docker-Socket, als auch die Docker-Executable beim Start des Jenkins Containers in diesem gemounted werden. Natürlich sind auch andere Möglichkeiten denkbar, wie beispielsweise das Einbinden einer externen Docker-Machine oder eines Amazon S3 Zugangs. Für unsere Zwecke ist das genannte Vorgehen jedoch ausreichend gut geeignet, um das angestrebte Ziel zu erreichen.

{% gist 5891c56c387ae1260d3827811760f88d run-docker.sh %}

Durch das Mounten des Sockets ist es nun möglich innerhalb von Jenkins mit Hilfe der Docker-CLI Befehle an den äußeren Docker-Host zu senden und neue Container zu starten bzw. zu stoppen.
<h2>Verwendung von Datenbanken für Integrationstestzwecke</h2>
Um Integrationstests laufen zu lassen, werden eine oder mehrere Instanzen verschiedener Datenbanktechnologien benötigt. Bisher wurden hierfür dedizierte virtuelle Maschinen im Netzwerk eingerichtet, auf denen jeweils eine Datenbank-Instanz läuft. Aus wirtschaftlicher Sicht ist dies ein verbesserungswürdiger Ansatz, da die Kapazitätsauslastung der einzelnen VMs moderat ist, da Integrationstests ausschließlich Nachts durchgeführt werden. Durch den Einsatz von Docker können Datenbank-Instanzen immer genau dann erzeugt werden, wenn diese benötigt werden und anschließend wieder entfernt werden. Durch das Entfernen der nicht mehr benötigten Instanzen können Ressourcen gespart und die Anzahl benötigter Host-Maschinen reduziert werden. Dies ist eine erhebliche Verbesserung gegenüber dem Einsatz nativer VM-Systeme, die permanent laufen oder als Image unnötig Speicherplatz belegen.

Für die Entwicklung der BIC Cloud werden bspw. die Datenbanken MySQL und OrientDB verwendet. Um Integrationstests gegen eine OrientDB laufen zu lassen, wird hierfür das offizielle Image aus dem Docker-Hub heruntergeladen und hochgefahren:

{% gist 5891c56c387ae1260d3827811760f88d run-orientdb.sh %}

Das Aufsetzen der benötigten Testdaten erfolgt autonom über die Integrationstestumgebung. Im Detail heißt dies, dass Integrationstests die benötigten Testdaten stets selbst definieren und keinen existierenden Datenbestand voraussetzen.

Sind die Tests durchgelaufen, wird über den Befehl <code>docker rm $JOB_NAME </code>die OrientDB-Docker-Instanz wieder entfernt. Durch die Verwendung einer Variable ist eine Kollision zwischen mehreren Jobs ausgeschlossen und es erlaubt parallel mehrere OrientDB-Instanzen zu starten. Durch das Entfernen der Instanz nach Durchführung der Tests wird der verwendete Plattenplatz wieder freigegeben und Seiteneffekte zwischen einzelnen Testdurchläufen verhindert.
