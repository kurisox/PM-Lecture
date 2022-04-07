---
type: lecture-cg
title: "Testbarkeit und Testdriven Development (TDD)"
menuTitle: "TDD"
author: "Carsten Gips (FH Bielefeld)"
weight: 6
readings:
  - key: "Martin2009"
  - key: "Dietz2018"
  - key: "Fowler2011"
  - key: "Beck2014"
  - key: "Passig2013"
  - key: "googlestyleguide"
tldr: |
  TODO

  Test-Driven Development (_TDD_)

outcomes:
  - k3: "Einsatz von TDD bei der Softwareentwicklung"
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
sketch: true
---


## Design für Testbarkeit

::: notes
### Schlechtes Beispiel
:::

```java
public void Ausgabe(...) {
    ... // mache komplexe Berechnungen
    ... // berechne nette Formatierungen
    System.out.println(...);
}
```

\pause
\bigskip
\hrule
\smallskip

::: notes
### Etwas verbessertes Beispiel
:::

```java
private A berechneA(...)  {
    return ...;  // mache komplexe Berechnungen
}
private String formatiereA(A a) {
    return ...;  // berechne nette Formatierungen
}
public void Ausgabe(...) {
    System.out.println(formatiereA(berechneA(...)));
}
```

## Anzeichen für schlecht testbares Design

*   Keine Rückgabewerte
*   Direkte Ausgaben
*   Mehr als ein Zweck:
    *   Berechnung plus Ausgabe
    *   Mehrere unterschiedliche Berechnungen
    *   ...
*   Seiteneffekte, beispielsweise Berechnung und direkte Veränderung von Zuständen **anderer** Objekte


## Private Methoden sind nicht (direkt) testbar

1.  Gar nicht testen????

    ::: notes
    Private Methoden sind nicht ohne Grund "privat": Sie gehören nicht zur
    Schnittstelle einer Klasse. Deshalb können sie jederzeit geändert oder
    sogar entfernt werden ...

    Andererseits steckt hier Funktionalität drin, die man in irgendeiner
    Weise absichern sollte. In "Softwarequalität" werden wir dieses Thema
    weiter diskutieren. Oft ist es aber hilfreich, auch für die privaten
    Methoden ausreichend Unit-Tests zu haben (Stichwort "Refactoring").
    :::

\smallskip

2.  Sichtbarkeit auf `protected` setzen; Tests im selben Package unterbringen

    ::: notes
    *   Testklassen und getestete Klassen im selben Ordner, oder
    *   Wahl unterschiedlicher Oberordner: `src` für Sourcecode und
        `test` für Testklassen und Spiegelung der Package-Struktur
        (Eclipse: Testordner über Project-Properties als Source-Ordner hinzufügen!)
    :::

\smallskip

3.  JUnit 4/5: [Testmethoden via]{.notes} Annotationen
    => Testmethoden mit Produktionscode mischen

\smallskip

4.  Reflection nutzen (vgl. spätere VL)

    ::: notes
    [**Nachteile**]{.alert}:

    *   Umgang mit Strings statt Klassen!
    *   Keine Compiler-/IDE-Unterstützung!
    *   Kein Refactoring!
    :::


## Was ist ein guter Test?

*   Läuft schnell. Läuft schnell. Läuft schnell!!!

    ::: notes
    Langsame Tests werden nicht oft (genug) ausgeführt!
    :::

\smallskip

*   Testet genau einen Aspekt, wenige Assertions

    ::: notes
    Wenn der Test fehlschlägt, ist offensichtlich, wo nach dem Problem gesucht werden muss.
    :::

\smallskip

*   Separiert Abhängigkeiten (Datenbanken, Netzwerk, ...)

    ::: notes
    Ein Test, der von Datenbanken o.ä. abhängt, ist normalerweise nicht schnell
    (vgl. Regel Eins). Außerdem können bei einem Test, der noch von anderen
    Dingen wie Datenbanken o.ä. abhängt, Fehler auftreten, die mit dem eigentlichen
    Tests nichts zu tun haben.

    Dazu werden Stubs und Mocks (vgl. Wahlfach "Softwarequalität") eingesetzt, um
    diese Abhängigkeiten "selbst in der Hand zu haben", d.h. zu ersetzen und zu
    simulieren.
    :::

\smallskip

*   Absicht ist klar zu verstehen

    ::: notes
    Jeder Entwickler kann sich den Test anschauen und dadurch verstehen, was
    vom Produktionscode erwartet wird.
    :::


## TDD oder der "Red-Green-Refactor"-Zyklus

![](images/tdd.png){width="70%" web_width="50%"}

::: notes
### Test-Driven Development (_TDD_)

1.  Schreibe Test => schlägt fehl: "Red"
    *   Überlegen Sie, wie das Stück Software, welches Sie erstellen wollen,
        funktionieren soll und wie Sie das testen würden, wenn es schon
        existieren würde.
    *   Stellen Sie sich vor, wie der (zu testende) Code aufgerufen wird und
        schreiben Sie den Testfall, als ob der Code schon existieren würde.
    *   Eclipse wird "meckern", weil es den aufgerufenen Code noch nicht gibt.
        Nutzen Sie einfach die vorgeschlagene Lösung von Eclipse und lassen
        sich den Methodenrumpf für die zu testende Methode(!) von Eclipse generieren.
    *   Jetzt compiliert der Code und der Test, aber der Test ist immer noch
        "rot".

2.  Schreibe genau soviel Code, dass Test OK wird: "Green"
    *   Implementieren Sie jetzt die Methode, d.h. füllen Sie den generierten
        Methodenrumpf mit "Leben". Aber nur genau so viel Code schreiben, bis
        der Test "grün" ist.

3.  Überarbeite Code-Struktur: "Refactor" (Refactoring ist Thema einer
    `[späteren VL]({{< ref "/coding/refactoring" >}})`{=markdown})
    *   Durch das reine Erfüllen des neuen Test gibt es im Laufe der Zeit sehr
        wahrscheinlich doppelten Code oder zu große Methoden o.ä. ... Dies wird
        durch diesen Schritt konsequent aufgeräumt.
    *   Eventuell müssen auch (frühere) Testfälle angepasst werden, die durch
        die eben implementierte neue Funktionalität ungültig werden.
:::

\vfill

*Anmerkung*: Jeder Zyklus sollte sehr kurz sein!
[=> Pro Stunde schafft man i.d.R. viele Rot/Grün/Refactoring-Zyklen!]{.notes}

<!-- TODO
[Demo: Taschenrechner (Addition zweier Ints)]{.bsp}
-->

::: notes
### Vorteile bei konsequenter Anwendung von TDD

*   Kein ungetesteter Code mehr:
    Es entstehen automatisch viele Unit-Tests und gibt keinen Grund, keine Tests zu schreiben
*   Kein unnötiger Code auf Vorrat: Es wird immer genau ein Test erfüllt
*   Konzentration auf das Wesentliche
*   Vermeidung von Redundanzen durch Refactoring
*   Die Testfälle dienen als Dokumentation, da man von erwartetem Verhalten ausgegangen ist
    Veraltet im Gegensatz zu separater Dokumentation nicht
*   Da der Entwickler zuerst einen Test schreiben muss, ist die Gefahr, dass
    er nicht versteht, was der Produktionscode machen soll, minimiert
:::


## Wrap-Up

<!-- TODO -->
Test-Driven Development (_TDD_)







<!-- DO NOT REMOVE - THIS IS A LAST SLIDE TO INDICATE THE LICENSE AND POSSIBLE EXCEPTIONS (IMAGES, ...). -->
::: slides
## LICENSE
![](https://licensebuttons.net/l/by-sa/4.0/88x31.png)

Unless otherwise noted, this work is licensed under CC BY-SA 4.0.
:::
