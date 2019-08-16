<!-- sectionTitle: Migration -->

# Migration

---

## Schritte
* `-Wc++-compat`
* Datei für Datei
* `throw`-Makro → `THROW`
* `NULL` → `nullptr`
* `-Wzero-as-nullpointer-constant`

---

## Schritte
* `memset` → `std::fill`
* `memcpy` → `std::copy`
* Expliziter cast vor `malloc`
* Standard-Container
  * `std::vector`, `std::vector` und `std::vector` 😁

---

## Structs → Klassen

```c
typedef struct master_keys {
  /* ... */
} master_keys;

return_status master_keys_get_signing_key(
    master_keys * const keys,
    buffer_t * const public_signing_key) {
  return_status status = return_status_init();

  if (keys == NULL) {
    throw(INVALID_INPUT, "keys is NULL")
  }
  /* ... */
}
```

---

## Structs → Klassen

<!-- note
Ja, ich habe tatsächlich diesen Fehler bekommen.
Und ja, das ist undefined behavior.
-->

```cpp
class MasterKeys {
  /* ... */
  return_status master_keys_get_signing_key(buffer_t * const public_signing_key) {
    return_status status = return_status_init();

    if (this == nullptr) { // 🤔
      THROW(INVALID_INPUT, "keys is NULL")
    }

    /* ... */
  }
}

```

---

## Schritte
* Create-Funktionen → statische Methoden → Konstruktor
* Visibility reduzieren (public/private)
* Copy-/Move-Assignment Operatoren/Konstruktoren

---

## Casts
* `-Wold-style-casts`
* `static_cast`
* `reinterpret_cast`
* `const_cast`
* `dynamic_cast` (nie verwendet)

---

## and, or, not

<!-- note
Vor allem "not" ist deutlich sichtbarer
-->

❌ gewohnt
```cpp
if (!(a && b) || c) {
  /* ... */
}
```

✅ schöner
```cpp
if (not (a and b) or c) {
  /* ... */
}
```


---

## alignof

### C
```c
#define ALIGNMENT_OF(type) offsetof( struct { char c; type t;}, t)
```

### C++
```cpp
alignof
```

---

## Headers

### C
```c
#include <stdint.h>
```

### C++
```cpp
#include <cstdint>
```

---

## Referenzen

<!-- note
Viele NULL-Checks werden überflüssig
-->

### C
```c
return_status do_something(buffer_t *buffer) {
  return_status status = return_status_init();
  if (buffer == NULL) {
    throw(INVALID_INPUT, "Buffer is missing");
  }

  /* ... */

cleanup:
  return status;
}
```

### C++
```cpp
void do_something(buffer_t& buffer) {
  /* ... */
}
```

---

## Goto

<!-- note
Semikolon nach Goto-Label
-->

✅ Gültiges C

❌ Ungültiges C++
```c
goto cleanup;
int temporary = 42;
cleanup:
;
```

---

## Goto

✅ Gültiges C++
```c
goto cleanup;
{
  int temporary = 42;
}
cleanup:
;
```
---

## sizeof(character)

x86_64
### ❌ C
```c
printf("%zu", sizeof('\0')); // 4
```

### ✅ C++
```cpp
printf("%zu", sizeof('\0')); // 1
```

---

## static_assert

### C (runtime)
```c
assert(crypto_auth_BYTES == crypto_auth_KEYBYTES);
```

### C++ (compile time)
```cpp
static_assert(crypto_auth_BYTES == crypto_auth_KEYBYTES);
```

---

## RAII

### ❌ C

```c
buffer_t* buffer = buffer_create_on_heap(size, capacity);
/* ... */
cleanup:
  buffer_destroy_from_heap(buffer);
```

### ✅ C++
```cpp
Buffer(size, capacity);
```

---

## RAII

### ❌ C
```c
sodium_mprotect_readonly(master_keys);
/* ... */
sodium_mprotect_noaccess(master_keys);
```

### ✅ C++

<!-- note
Ja ich weiß, das hätte man auch als Methode auf MasterKeys machen können
-->

```c
MasterKeys::Unlocker(master_keys) unlocker;
```

---

## RAII
* `std::unique_ptr`
* `std::shared_ptr`
* ⚠ `std::bad_alloc` fangen!

---

## std::unique_ptr mit anderem Allocator
```cpp
template <typename T>
class SodiumDeleter {
public:
  void operator()(T* object) {
    if (object != nullptr) {
      sodium_free(object);
    }
  }
};
```

```cpp
this->private_keys = std::unique_ptr<PrivateKeys,SodiumDeleter<PrivateKeys>>(sodium_malloc<PrivateKeys>(1));
```

---

## GSL
* C++ Core Guidelines
* Guidelines support library
* `span`
* `byte` (pre C++17)
* `narrow`

---

## GSL

### Vorher

```c
return_status do_something(const char* bytes, size_t bytes_length);
```

### Nachher
```cpp
return_status do_something(span<byte> buffer);
```

* 😕 signed size-Typ (ständige Konvertierung zu/von `size_t`)

---

## GSL

```cpp
auto size = narrow<size_t>(signed_size);
```

* Fehlerverhalten konfigurierbar (z.B. Exception)

---

## templates

### ❌ C
```c
return_status endianness_uint32_to_big_endian(/* ... */);
return_status endianness_int32_to_big_endian(/* ... */);
return_status endianness_uint64_to_big_endian(/* ... */);
return_status endianness_int64_to_big_endian(/* ... */);
/* ... */
```

### ✅ C++
```cpp
template <typename IntegerType>
void to_big_endian(IntegerType integer, span<gsl::byte> output) {
/* ... */
```

---

## templates

### ❌ C
```c
#define free_and_null_if_valid(pointer)\
  if (pointer != NULL) {\
    free(pointer);\
    pointer = NULL;\
  }
```

### ✅ C++
```cpp
template <typename T>
inline void free_and_null_if_valid(T*& pointer) {
  if (pointer != nullptr) {
    free(pointer);
    pointer = nullptr;
  }
}
```


---

## Namespaces

### C
```c
return_status molch_create_user(/* ... */);
return_status molch_list_users(/* ... */);
```

### C++
```c
namespace Molch {
  return_status create_user(/* .. */);
  return_status list_users(/* .. */);
}
```

---

## Enum Class

* Typensicherheit
* Namespacing

```cpp
enum class status_type {
  SUCCESS = 0,
  GENERIC_ERROR,
  INVALID_VALUE,
  INCORRECT_BUFFER_SIZE,
  BUFFER_ERROR,
  /* ... */
};
```

---

## Typensichere Keys

<!-- note

-->

Nie wieder public-/private-Key in API verwechseln
```cpp
enum class KeyType : uint8_t {
  /* .. */
  PublicKey,
  PrivateKey,
  /* ... */
};

template <size_t key_length, KeyType keytype>
class Key : public std::array<std::byte,key_length> {
  /* ... */
  template <typename DerivedKeyType>
  result<DerivedKeyType> deriveSubkeyWithIndex(const uint32_t subkey_counter) const {
    /* ... */
  }
  /* ... */
}

/* ... */
using PublicKey = Key<PUBLIC_KEY_SIZE,KeyType::PublicKey>;
using PrivateKey = Key<PRIVATE_KEY_SIZE,KeyType::PrivateKey>;
```

---

## Alignment

```cpp
std::alignof()
```
Eigene Implementierung fällt weg
