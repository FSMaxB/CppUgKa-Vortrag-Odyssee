<!-- sectionTitle: Wieder Serialisierung -->

# Wieder Serialisierung

---

## Protobuf C++-Library
* API benutzt std::string für bytes
  * ⇒ Alle keys auf normalem Heap
* "hübsche API" macht Allocator-Tausch sinnlos

---

## Arena-Allocator

* Anfangs selbst implementiert
* Später mit Google-Protobuf-Library
* Eine Arena pro Serialisierung-/Deserialisiserung
* `sodium_malloc` plötzlich eine Option
* `zeroed_malloc` gelöscht
