---
filename: learnd-de.d
contributors:
    - ["Nick Papanastasiou", "www.nickpapanastasiou.github.io"]
translators:
    - ["Dominik Süß", "www.thesuess.me"]
---

```d
// Es war klar, dass das kommt...
module hello;

import std.stdio;

// Argumente sind optional
void main(string[] args) {
    writeln("Hello, World!");
}
```

Wenn du so wie ich bist und viel Zeit im Internet verbringst, stehen die Chancen
gut, dass du schonmal über [D](http://dlang.org/) gehört hast.
Die D-Sprache ist eine moderne, überall einsetzbare Programmiersprache die von
Low bis High Level verwendet werden kann und dabei viele Stile anbietet.

D wird aktiv von Walter Bright und Andrei Alexandrescu entwickelt, zwei super schlaue,
richtig coole Leute. Da das jetzt alles aus dem Weg ist - auf zu den Beispielen!

```d
import std.stdio;

void main() {

    // Logische Ausdrücke und Schleifen funktionieren wie erwartet
    for(int i = 0; i < 10000; i++) {
        writeln(i);
    }

    auto n = 1; // auto um den Typ vom Compiler bestimmen zu lassen

    // Zahlenliterale können _ verwenden für Lesbarkeit
    while(n < 10_000) {
        n += n;
    }

    do {
        n -= (n / 2);
    } while(n > 0);

    // For und while sind ja schön und gut aber D bevorzugt foreach
    // Die '..' erstellen eine Spanne von Zahlen, inklusive dem ersten Wert
    // jedoch ohne dem letzten
    foreach(i; 1..1_000_000) {
        if(n % 2 == 0)
            writeln(i);
    }

    // Es gibt auch ein 'foreach_reverse' wenn du rückwerts gehen willst.
    foreach_reverse(i; 1..int.max) {
        if(n % 2 == 1) {
            writeln(i);
        } else {
            writeln("No!");
        }
    }
}
```

Neue Typen können mit `struct`, `class`, `union`, und `enum` definiert werden.
Structs und unions werden as-value (kopiert) an Methoden übergeben wogegen
Klassen als Referenz übergeben werden. Templates können verwendet werden um
alle Typen zu parametrisieren.

```d
// Hier, T ist ein Type-Parameter, Er funktioniert wie Generics in C#/Java/C++
struct LinkedList(T) {
    T data = null;
    LinkedList!(T)* next; // Das ! wird verwendet, um T zu übergeben. (<T> in C#/Java/C++)
}

class BinTree(T) {
    T data = null;

    // Wenn es nur einen T Parameter gibt, können die Klammern um ihn weggelassen werden
    BinTree!T left;
    BinTree!T right;
}

enum Day {
    Sunday,
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
}

// Aliase können verwendet werden, um die Entwicklung zu erleichtern

alias IntList = LinkedList!int;
alias NumTree = BinTree!double;

// Funktionen können genau so Templates beinhalten

T max(T)(T a, T b) {
    if(a < b)
        return b;

    return a;
}

// Steht ref vor einem Parameter, wird sichergestellt, dass er als Referenz
übergeben wird. Selbst bei Werten wird es immer eine Referenz sein.
void swap(T)(ref T a, ref T b) {
    auto temp = a;

    a = b;
    b = temp;
}

// Templates können ebenso Werte parametrisieren.
class Matrix(uint m, uint n, T = int) {
    T[m] rows;
    T[n] columns;
}

auto mat = new Matrix!(3, 3); // Standardmäßig ist T vom Typ Integer
```

Wo wir schon bei Klassen sind - Wie wäre es mit Properties! Eine Property
ist eine Funktion, die wie ein Wert agiert. Das gibt uns viel klarere Syntax
im Stil von `structure.x = 7` was gleichgültig wäre zu `structure.setX(7)`

```d
// Diese Klasse ist parametrisiert mit T, U

class MyClass(T, U) {
    T _data;
    U _other;

}

// Ihre Getter und Setter Methoden sehen so aus
class MyClass(T, U) {
    T _data;
    U _other;

    // Konstruktoren heißen immer `this`
    this(T t, U u) {
        data = t;
        other = u;
    }

    // getters
    @property T data() {
        return _data;
    }

    @property U other() {
        return _other;
    }

    // setters
    // @property kann genauso gut am ende der Methodensignatur stehen
    void data(T t) @property {
        _data = t;
    }

    void other(U u) @property {
        _other = u;
    }
}
// Und so kann man sie dann verwenden

void main() {
    auto mc = MyClass!(int, string);

    mc.data = 7;
    mc.other = "seven";

    writeln(mc.data);
    writeln(mc.other);
}
```

Mit properties können wir sehr viel Logik hinter unseren gettern
und settern hinter einer schönen Syntax verstecken

Andere Objektorientierte features sind beispielsweise
`interface`s, `abstract class` und `override`.
Vererbung funktioniert in D wie in Java:
Erben von einer Klasse, so viele Interfaces wie man will.

Jetzt haben wir Objektorientierung in D gesehen, aber schauen
wir uns noch was anderes an.
D bietet funktionale Programmierung mit _first-class functions_
puren Funktionen und unveränderbaren Daten.
Zusätzlich können viele funktionale Algorithmen wie z.B
map, filter, reduce und friends im `std.algorithm` Modul gefunden werden!

```d
import std.algorithm : map, filter, reduce;
import std.range : iota; // builds an end-exclusive range

void main() {
    // Wir wollen die Summe aller Quadratzahlen zwischen
    // 1 und 100 ausgeben. Nichts leichter als das!

    // Einfach eine Lambda-Funktion als Template Parameter übergeben
    // Es ist genau so gut möglich eine normale Funktion hier zu übergeben
    // Lambdas bieten sich hier aber an.
    auto num = iota(1, 101).filter!(x => x % 2 == 0)
                           .map!(y => y ^^ 2)
                           .reduce!((a, b) => a + b);

    writeln(num);
}
```

Ist dir aufgefallen, wie wir eine Haskell-Style Pipeline gebaut haben
um num zu berechnen?
Das war möglich durch die Uniform Function Call Syntax.
Mit UFCS können wir auswählen, ob wir eine Funktion als Methode oder
als freie Funktion aufrufen. Walters Artikel dazu findet ihr
[hier.](http://www.drdobbs.com/cpp/uniform-function-call-syntax/232700394)
Kurzgesagt kann man Funktionen, deren erster Parameter vom typ A ist, als
Methode auf A anwenden.

Parallel Computing ist eine tolle Sache, findest du nicht auch?

```d
import std.stdio;
import std.parallelism : parallel;
import std.math : sqrt;

void main() {
    // Wir wollen die Wurzel von jeder Zahl in unserem Array berechnen
    // und dabei alle Kerne verwenden, die wir zur Verfügung haben
    auto arr = new double[1_000_000];

    // Wir verwenden den Index und das Element als Referenz
    // und rufen einfach parallel auf!
    foreach(i, ref elem; parallel(arr)) {
        ref = sqrt(i + 1.0);
    }
}
```
