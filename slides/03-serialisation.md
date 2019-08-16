<!-- sectionTitle: Serialisierung -->

# Serialisierung

---

## Protocol Buffers


```protobuf
syntax = "proto2";

option optimize_for = LITE_RUNTIME;

package Molch.Protobuf;

message Key {
  required bytes key = 1;
}
```

---

## Protobuf-C
<!-- note
Der Name ist noch nicht mal vom Standard erlaubt ...
-->

### generierter Code

```c
typedef struct _Molch__Protobuf__Key Molch__Protobuf__Key;
struct  _Molch__Protobuf__Key
{
  ProtobufCMessage base;
  ProtobufCBinaryData key;
};

void   molch__protobuf__key__init(Molch__Protobuf__Key *message);
size_t molch__protobuf__key__get_packed_size(const Molch__Protobuf__Key *message);
size_t molch__protobuf__key__pack(const Molch__Protobuf__Key *message, uint8_t *out);
/* ... */
```

---

## Keys löschen

❌ *dead store elimination*
```c
memset(buffer, '\0', buffer_size);
```

---

## Keys löschen

✅ gelöstes Problem
```c
sodium_memzero(buffer, buffer_size);
```

---

## sodium_malloc
* Allokiert 4 oder mehr Speicherseiten
  * Guard page davor
  * Nutz-Page
  * Guard page danach
* `sodium_memzero` bei deallokation
* ❌ zu viele kleine Allokationen

---

```
          page 1               page 2               page 3                     page n - 1     page n
+-------------------------+------------+----------------------------+-- ... --+-----------+------------+
| allocation_size | empty | guard page | empty | canary | user_data |   ...   | user_data | guard page |
+-------------------------+------------+----------------------------+-- ... --+-----------+------------+
^                                      ^       ^        ^
|                                      | ------|--------| unprotected size ---------------|
base_ptr                 unprotected_ptr       |        user_ptr
                                               canary_ptr
```
[Aus einem GitHub-Kommentar von mir](https://github.com/jedisct1/libsodium/issues/572#issuecomment-319654932)

---

## zeroed_malloc
* Eigenimplementierung
* Speichert Länge
* Überschreibt beim deallokieren mit 0

---

<!-- note
Vielleicht weiß jemand was jetzt kommt.
-->

## Erste tests auf ARM

---

## Erste tests auf ARM

### Program terminated with signal SIGBUS, Bus error.

---

## Alignment

```c
void *next_aligned_address(void *pointer, size_t alignment){
  //determine the amount of padding for proper alignment
  size_t padding = 0; //padding needed to fit alignment
  if (alignment != 0) {
    padding = (alignment - (((size_t)pointer) % alignment)) % alignment;
  }

  return (char*)pointer + padding;
}
```

---

## Protobuf-C allocator

<!-- note
In C ist sowas relativ einfach
-->

```c
void *protobuf_c_allocator(void *allocator_data __attribute__((unused)), size_t size) {
  return zeroed_malloc(size);
}

void protobuf_c_free(void *allocator_data __attribute__((unused)), void *pointer) {
  zeroed_free(pointer);
}

static ProtobufCAllocator protobuf_c_allocators __attribute__((unused)) = {
  &protobuf_c_allocator,
  &protobuf_c_free,
  NULL
};
```
