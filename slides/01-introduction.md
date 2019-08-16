<!-- sectionTitle: Einführung -->

## Worum gehts?

<!-- note
* Persönliche Odyssee
-->
* [Molch](https://github.com/1984Not-GmbH/molch)-Library
* [Double Ratchet](https://signal.org/docs/specifications/doubleratchet/)-Implementierung
  * Krypto-Messenger-Protokoll
  * ehemals Axolotl
* Projektstart: Mitte 2015

---

## Anforderungen
* Plattformübergreifend
  * Desktop
  * Android
  * iOS
* Keys sicher im Speicher
* State (de-)serialisierbar

---

<!-- sectionTitle: Sprachen -->

## Sprachen
* Java
* Go
* Rust
* C++
* C
<!-- note
Meine Argumente 2015
-->

---

## Java
* iOS? (-)
* Speicher löschen? (-)
* Garbage Collector? (-)
* Speichersicher (+)

---

## Go
* recht jung (-)
* Speicher löschen? (-)
* Cross-Kompilierung? (-)
* Libraries? (-)
* Speichersicher (+)

---

## Rust
<!-- note
15. Mai 2015
-->
* extrem Jung (-)
* Cross-Kompilierung? (-)
* Libraries? (-)
* Speichersicher (+)

---

## C++
* sehr Komplex (-)
* unsicher (-)
* plattformübergreifend (+)

---

## C
* unsicher (-)
* plattformübergreifend (+)
* einfach (+) 🤔

---

## Entscheidung
* C/C++ damals alternativlos
* C++ zu komplex
* ⇒ **C**
