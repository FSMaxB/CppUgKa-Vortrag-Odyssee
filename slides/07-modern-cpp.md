<!-- sectionTitle: Modern C++ -->

# Modern C++

---

## Switch Case

### Alt

```cpp
char digit;
switch (ternary) {
  case 0:
    digit = '0';
    break;

  case 1:
    digit = '1';
    break;

  case 2:
    digit = '2';
    break;

  default:
    std::terminate();
}
```

---

## Switch Case

<!-- notes
Immediately Invoked Function Expression
-->

### Neu (IIFE)

```cpp
char digit = [=]{
  switch (ternary) {
    case 0:
      return '0';

    case 1:
      return '1';

    case 2:
      return '2';

    default:
      std::terminate();
  }
}();
```

---

## if-Expression

```cpp
char digit = [=]{
  if (binary) {
    return '1';
  } else {
    return '0';
  }
}();
```

---

## std::optional

**Clunky und Fehleranfällig**
```cpp
auto data_optional = get_data();
if (not data_optional.has_value()) {
  return std::nullopt;
}
auto& data = *data_optional;
```

---

## std::span

* Laufzeit-Check unmöglich.
* `subspan` ist undefined wenn out of range

---

## max_align_t (C++17)

* `alignof(max_align_t)` Maximal mögliches Alignment
* Früher simuliert durch `int_max_t` aber nicht durch Standard garantiert.
