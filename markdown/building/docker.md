---
type: lecture-cg
title: "Einführung in Docker"
menuTitle: "Docker"
author: "Carsten Gips (FH Bielefeld)"
weight: 5
readings:
  - key: "Ullenboom2016"
  - key: "Inden2013"
tldr: |
    Container sind im Gegensatz zu herkömmlichen VMs eine schlanke Virtualisierungslösung.
    Dabei laufen die Prozesse direkt im Kernel des Host-Betriebssystems, aber abgeschottet
    von den anderen Prozessen durch Linux-Techniken wie `cgroups` und `namespaces` (unter
    Windows kommt dafür der WSL2 zum Einsatz, unter macOS wird eine kleine Virtualisierung
    genutzt).

    Container sind sehr nützlich, wenn man an mehreren Stellen eine identische Arbeitsumgebung
    benötigt. Man kann dabei entweder die Images (fertige Dateien) oder die Dockerfiles
    (Anweisungen zum Erzeugen eines Images) im Projekt verteilen. Tatsächlich ist es nicht
    unüblich, ein Dockerfile in das Projekt-Repo mit einzuchecken.

    Durch Container hat man allerdings im Gegensatz zu herkömmlichen VMs keinen Sicherheitsgewinn,
    da die im Container laufende Software ja direkt auf dem Host-Betriebssystem ausgeführt wird.

    Es gibt auf DockerHub fertige Images, die man sich ziehen und starten kann. Ein solches
    gestartetes Image nennt sich dann Container und enthält beispielsweise Dateien, die
    in den Container gemountet oder kopiert werden. Man kann auch eigene Images bauen, indem
    man eine entsprechende Konfiguration (Dockerfile) schreibt. Jeder Befehl bei der Erstellung
    eines Images erzeugt einen neuen Layer, die sich dadurch mehrere Images teilen können.

    In der Konfiguration einer Gitlab-CI-Pipeline kann man mit `image` ein Docker-Image
    angeben, welches dann in der Pipeline genutzt wird.

    VSCode kann über das Remote-Plugin sich (u.a.) mit Containern verbinden und dann im
    Container arbeiten (editieren, compilieren, debuggen, testen, ...).

    In dieser kurzen Einheit kann ich Ihnen nur einen ersten Einstieg in das Thema geben.
    Wir haben uns beispielsweise nicht Docker Compose oder Kubernetes angeschaut, und auch
    die Themen Netzwerk (zwischen Containern oder zwischen Containern und anderen Rechnern)
    und Volumnes habe ich außen vor gelassen. Dennoch kommt man in der Praxis bereits mit
    den hier vermittelten Basiskenntnissen erstaunlich weit ...
outcomes:
  - k2: "Unterschied zwischen Containern und VMs"
  - k2: "Einsatzgebiete für Container"
  - k2: "Container laufen als abgeschottete Prozesse auf dem Host -- kein Sandbox-Effekt"
  - k3: "Container von DockerHub ziehen"
  - k3: "Container starten"
  - k3: "Eigene Container definieren und bauen"
  - k3: "Einsatz von Containern im Gitlab-CI"
  - k3: "Einsatz von VSCode und Containern"
quizzes:
  - link: "XYZ"
    name: "Quiz XXX (ILIAS)"
assignments:
  - topic: sheet01
youtube:
  - link: ""
    name: "VL "
  - link: ""
    name: "Demo "
  - link: ""
    name: "Demo "
fhmedia:
  - link: ""
    name: "VL "
---



## Motivation Gitlab-CI: WFM (*Works For Me*)

![](images/ci.png){width="60%"}

::: notes
Auf dem CI-Server muss man eine Arbeitsumgebung konfigurieren und bereitstellen, für
Java-basierte Projekte muss beispielsweise ein JDK existieren und man benötigt Tools
wie Maven oder Gradle, um die Buildskripte auszuführen. Je nach Projekt braucht man
dann noch weitere Tools und Bibliotheken. Diese Konfigurationen sind unabhängig vom
CI-Server und werden üblicherweise nicht direkt installiert, sondern über eine
Virtualisierung bereitgestellt.

Selbst wenn man keine CI-Pipelines einsetzt, hat man in Projekten mit mehreren
beteiligten Personen häufig das Problem "*WFM*" ("works for me"). Jeder Entwickler
hat sich auf ihrem Rechner eine Entwicklungsumgebung aufgesetzt und nutzt in der Regel
seine bevorzugte IDE oder sogar unterschiedliche JDK-Versionen ... Dadurch kann es
schnell passieren, dass Probleme oder Fehler auftreten, die sich nicht von allen
Beteiligten immer nachvollziehen lassen. Hier wäre eine einheitliche Entwicklungsumgebung
sinnvoll, die in einer "schlanken" Virtualisierung bereitgestellt wird.

Als Entwickler kann man zeitgleich in verschiedenen Projekten beteiligt sein, die
unterschiedliche Anforderungen an die Entwicklungstools mit sich bringen. Es könnte
beispielsweise passieren, dass man zeitgleich drei bestimmte Python-Versionen benötigt.
In den meisten Fällen schafft man es (mit ein wenig Aufwand), diese Tools nebeneinander
zu installieren. Oft ist das in der Praxis aber schwierig und fehleranfällig.

In diesen Fällen kann eine Virtualisierung helfen.
:::


## Virtualisierung: Container vs. VM

::: center
![](images/virtualisierung.png){width="90%"}
:::

::: notes
Wenn man über Virtualisierung auf dem Desktop spricht, kann man grob zwei Varianten
unterscheiden. In beiden Fällen ist die Basis die Hardware (Laptop, Desktop-Rechner)
und das darauf laufende (Host-) Betriebssystem (Linux, FreeBSD, macOS, Windows, ...).
Darauf läuft dann wiederum die Virtualisierung.

Im rechten Bild wird eine herkömmliche Virtualisierung mit virtuellen Maschinen (*VM*)
dargestellt. Dabei wird in der VM ein komplettes Betriebssystem (das "Gast-Betriebssystem")
installiert und darin läuft dann die gewünschte Anwendung. Die Virtualisierung (VirtualBox,
VMware, ...) läuft dabei als Anwendung auf dem Host-Betriebssystem und stellt dem
Gast-Betriebssystem in der VM einen Rechner mit CPU, RAM, ... zur Verfügung und übersetzt
die Systemaufrufe in der VM in die entsprechenden Aufrufe im Host-Betriebssystem. Dies benötigt
in der Regel entsprechende Ressourcen: Durch das komplette Betriebssystem in der VM ist eine
VM (die als Datei im Filesystem des Host-Betriebssystems liegt) oft mehrere 10GB groß. Für die
Übersetzung werden zusätzlich Hardwareressourcen benötigt, d.h. hier gehen CPU-Zyklen und RAM
"verloren" ... Das Starten einer VM dauert entsprechend lange, da hier ein komplettes
Betriebssystem hochgefahren werden muss. Dafür sind die Prozesse in einer VM relativ stark
vom Host-Betriebssystem abgekapselt, so dass man hier von einer "Sandbox" sprechen kann: Viren
o.ä. können nicht so leicht aus einer VM "ausbrechen" und auf das Host-Betriebssystem zugreifen
(quasi nur über Lücken im Gast-Betriebssystem kombiniert mit Lücken in der Virtualisierungssoftware).

Im linken Bild ist eine schlanke Virtualisierung auf Containerbasis dargestellt. Die Anwendungen
laufen direkt als Prozesse im Host-Betriebssystem, ein Gast-Betriebssystem ist nicht notwendig.
Durch den geschickten Einsatz von `namespaces` und `cgroups` und anderen in Linux und FreeBSD
verfügbaren Techniken werden die Prozesse abgeschottet, d.h. der im Container laufende Prozess
"sieht" die anderen Prozesse des Hosts nicht. Die Erstellung und Steuerung der Container übernimmt
hier beispielsweise Docker. Die Container sind dabei auch wieder Dateien im Host-Filesystem.
Dadurch benötigen Container wesentlich weniger Platz als herkömmliche VMs, der Start einer Anwendung
geht deutlich schneller und die Hardwareressourcen (CPU, RAM, ...) werden effizient genutzt.
Nachteilig ist, dass hier in der Regel ein Linux-Host benötigt wird (für Windows wird mittlerweile
der Linux-Layer (*WSL*) genutzt; für macOS wurde bisher eine Linux-VM im Hintergrund hochgefahren,
mittlerweile wird aber eine eigene schlanke Virtualisierung eingesetzt). Außerdem steht im Container
üblicherweise kein graphisches Benutzerinterface zur Verfügung. Da die Prozesse direkt im
Host-Betriebssystem laufen, stellen Container keine Sicherheitsschicht ("Sandboxen") dar!

In allen Fällen muss die Hardwarearchitektur beachtet werden: Auf einer Intel-Maschine können
normalerweise keine VMs/Container basierend auf ARM-Architektur ausgeführt werden und umgekehrt.
:::


## Getting started

*   DockerHub: fertige Images => [hub.docker.com/search](https://hub.docker.com/search?q=&type=image)

\smallskip

*   Image downloaden: `docker pull <IMAGE>`
*   Image starten: `docker run <IMAGE>`

::: notes
### Begriffe

*   Docker-File: Beschreibungsdatei, wie Docker ein Image erzeugen soll.
*   Image: Enthält die Dinge, die lt. dem Docker-File in das Image gepackt werden sollen.
    Kann gestartet werden und erzeugt damit einen Container.
*   Container: Ein laufendes Images (genauer: eine laufende Instanz eines Images). Kann
    dann auch zusätzliche Daten enthalten.

### Beispiele
:::

::: slides
\bigskip
\smallskip

**Beispiele**
:::

```
docker pull debian:stable-slim
docker run --rm -it debian:stable-slim /bin/sh
```

::: notes
`debian` ist ein fertiges Images, welches über DockerHub bereit gestellt wird. Mit dem
Postfix `stable-slim` wird eine bestimmte Version angesprochen.

Mit `docker run debian:stable-slim` startet man das Image, es wird ein Container
erzeugt. Dieser enthält den aktuellen Datenstand, d.h. wenn man im Image eine Datei
anlegt, wäre diese dann im Container enthalten.

Mit der Option `--rm` wird der Container nach Beendigung automatisch wieder gelöscht. Da
jeder Aufruf von `docker run <IMAGE>` einen neuen Container erzeugt, würden sich sonst
recht schnell viele Container auf dem Dateisystem des Hosts ansammeln, die man dann manuell
aufräumen müsste. Man kann aber einen beendeten Container auch erneut laufen lassen ...
(vgl. Dokumentation von `docker`). Mit der Option `--rm` sind aber auch im Container angelegte
Daten wieder weg! Mit der Option `-it` wird der Container interaktiv gestartet und man landet
in einer Shell.

Bei der Definition eines Images kann ein "*Entry Point*" definiert werden, d.h. ein Programm,
welches automatisch beim Start des Container ausgeführt wird. Häufig erlauben Images aber auch,
beim Start ein bestimmtes auszuführendes Programm anzugeben. Im obigen Beispiel ist das `/bin/sh`,
also eine Shell ...
:::

\smallskip

```
docker pull openjdk:latest
docker run --rm -v "$PWD":/usr/src -w /usr/src openjdk:latest javac Hello.java
docker run --rm -v "$PWD":/usr/src -w /usr/src openjdk:latest java Hello
```

::: notes
Auch für Java gibt es vordefinierte Images mit einem JDK. Das Tag "`latest`" zeigt dabei auf die
letzte stabile Version des `openjdk`-Images. Üblicherweise wird "`latest`" von den Entwicklern immer
wieder weiter geschoben, d.h. auch bei anderen Images gibt es ein "`latest`"-Tag. Gleichzeitig ist
es die Default-Einstellung für die Docker-Befehle, d.h. es kann auch weggelassen werden:
`docker run openjdk:latest` und `docker run openjdk` sind gleichwertig. Alternativ kann man hier auch
hier wieder eine konkrete Version angeben.

Über die Option `-v` wird ein Ordner auf dem Host (hier durch `"$PWD"` dynamisch ermittelt) in den
Container eingebunden ("gemountet"), hier auf den Ordner `/usr/src`. Dort sind dann die Dateien sichtbar,
die im Ordner `"$PWD"` enthalten sind. Über die Option `-w` kann ein Arbeitsverzeichnis definiert
werden.

Mit `javac Hello.java` wird `javac` im Container aufgerufen auf der Datei `/usr/src/Hello.java`
im Container, d.h. die Datei `Hello.java`, die im aktuellen Ordner des Hosts liegt (und in den
Container gemountet wurde). Das Ergebnis (`Hello.class`) wird ebenfalls in den Ordner `/usr/src/`
im Container geschrieben und erscheint dann im Arbeitsverzeichnis auf dem Host ... Analog kann
dann mit `java Hello` die Klasse ausgeführt werden.
:::

[[Kurze Demo]{.bsp}]{.slides}


## Images selbst definieren

```{.docker size="footnotesize"}
FROM debian:stable-slim

ARG USERNAME=pandoc
ARG USER_UID=1000
ARG USER_GID=1000

RUN apt-get update && apt-get install -y --no-install-recommends            \
        apt-utils bash wget make graphviz biber                             \
        texlive-base texlive-plain-generic texlive-latex-base               \
    #
    && groupadd --gid $USER_GID $USERNAME                                   \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME    \
    #
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

WORKDIR /pandoc
USER $USERNAME
```

\bigskip

`docker build -t <NAME> -f <DOCKERFILE> .`

::: notes
`FROM` gibt die Basis an, d.h. hier ein Image von Debian in der Variante `stable-slim`,
d.h. das ist der Basis-Layer für das zu bauende Docker-Image.

Über `ARG` werden hier Variablen gesetzt.

`RUN` ist der Befehl, der im Image (hier Debian) ausgeführt wird und einen neuen Layer
hinzufügt. In diesen Layer werden alle Dateien eingefügt, die bei der Ausführung des
Befehls erzeugt oder angelegt werden. Hier im Beispiel wird das Debian-Tool `apt-get`
gestartet und weitere Debian-Pakete installiert.

Da jeder `RUN`-Befehl einen neuen Layer anlegt, werden die restlichen Konfigurationen
ebenfalls in diesem Lauf durchgeführt. Insbesondere wird ein nicht-Root-User angelegt,
der von der UID und GID dem Default-User in Linux entspricht. Die gemounteten Dateien
haben die selben Rechte wie auf dem Host, und durch die Übereinstimmung von UID/GID
sind die Dateien problemlos zugreifbar und man muss nicht mit dem Root-User arbeiten
(dies wird aus offensichtlichen Gründen als Anti-Pattern angesehen). Bevor der `RUN`-Lauf
abgeschlossen wird, werden alle temporären und später nicht benötigten Dateien von
`apt-get` entfernt, damit diese nicht Bestandteil des Layers werden.

Mit `WORKDIR` und `USER` wird das Arbeitsverzeichnis gesetzt und auf den angegebenen User
umgeschaltet. Damit muss der User nicht mehr beim Aufruf von außen gesetzt werden.

Über `docker build -t <NAME> -f <DOCKERFILE> .` wird aus dem angegebenen Dockerfile und
dem Inhalt des aktuellen Ordners ("`.`") ein neues Image erzeugt und mit dem angegebenen
Namen benannt.


**Hinweis zum Umgang mit Containern und Updates**:
Bei der Erstellung eines Images sind bestimmte Softwareversionen Teil des Images geworden.
Man kann prinzipiell in einem Container die Software aktualisieren, aber dies geht in dem
Moment wieder verloren, wo der Container beendet und gelöscht wird. Außerdem widerspricht
dies dem Gedanken, dass mehrere Personen mit dem selben Image/Container arbeiten und damit
auch die selben Versionsstände haben. In der Praxis löscht man deshalb das alte Image einfach
und erstellt ein neues, welches dann die aktualisierte Software enthält.
:::

[Beispiel: [debian-latex.df](https://github.com/PM-Dungeon/PM-Lecture/blob/master/markdown/building/src/debian-latex.df)]{.bsp}


## CI-Pipeline

```yaml
default:
  image: openjdk:17

job1:
    tags:
        - ivyjava
    script:
        - echo "Java Version:"
        - java -version
        - javac Hello.java
        - java Hello
        - ls -lags
```

::: notes
In den Gitlab-CI-Pipelines (analog wie in den GitHub-Actions) kann man Docker-Container
für die Ausführung der Pipeline nutzen.

Mit `image: openjdk:11` wird das Docker-Image `openjdk:11` vom DockerHub geladen und durch
den Runner mit dem Tag `ivyjava` für die Stages als Container ausgeführt. Die Aktionen im
`script`-Teil, wie beispielsweise `javac Hello.java` werden vom Runner an die Standard-Eingabe
der Shell des Containers gesendet. Im Prinzip entspricht das dem Aufruf auf dem lokalen Rechner:
`docker run openjdk:11 javac Hello.java`.
:::

[[Beispiel: Gitlab, cagix/wuppie]{.bsp}]{.slides}

<!-- TODO: Beispiel GH? -->


## VSCode und das Plugin "Remote - Containers"

![](images/vscode-remote.png){width="90%"}

::: notes
1.  VSCode (Host): Plugin "Remote - Containers" installieren
2.  Docker (Host): Container starten mit Workspace gemountet
3.  VSCode (Host): Attach to Container => neues Fenster (Container)
4.  VSCode (Container): Plugin "Java Extension Pack" installieren
5.  VSCode (Container): Dateien editieren, kompilieren, debuggen, ...
:::

[[Kurze Demo]{.bsp}]{.slides}


::::::::: notes
## Link-Sammlung

*   [Wikipedia: Docker](https://en.wikipedia.org/wiki/Docker_(software))
*   [Wikipedia: Virtuelle Maschinen](https://en.wikipedia.org/wiki/Virtual_machine)
*   [Docker: Überblick, Container](https://www.docker.com/resources/what-container)
*   [Docker: HowTo](https://docs.docker.com/get-started/)
*   [DockerHub: Suche nach fertigen Images](https://hub.docker.com/search?q=&type=image)
*   [Docker und Java](https://docs.docker.com/language/java/)
*   [Dockerfiles: Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
*   [Gitlab, Docker](http://git01-ifm-min.ad.fh-bielefeld.de/help/ci/docker/using_docker_images.md#overriding-the-entrypoint-of-an-image)
*   [VSCode: Entwickeln in Docker-Containern](https://code.visualstudio.com/docs/remote/containers)
*   @DockerInAction und @DockerInPractice
:::::::::







<!-- DO NOT REMOVE - THIS IS A LAST SLIDE TO INDICATE THE LICENSE AND POSSIBLE EXCEPTIONS (IMAGES, ...). -->
::: slides
## LICENSE
![](https://licensebuttons.net/l/by-sa/4.0/88x31.png)

Unless otherwise noted, this work is licensed under CC BY-SA 4.0.
:::
