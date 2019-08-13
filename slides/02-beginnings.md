# Erste Schritte

---

## Keys löschen

❌ *dead store elimination*
```c
memset(buffer, sizeof(char), buffer_size);
```

---

## Keys löschen

✅ gelöstes Problem
```c
sodium_memzero(buffer, buffer_size);
```

---

## Error Handling

❌ errno

---

<!-- sectionTitle: Error Handling -->

## Error Handling

**0 == Erfolg**
```c
int diffie_hellman(/* ... */) {
  int status = 0;
  /* ... */
  status = crypto_scalarmult(/* ... */);
  if (status != 0) {
    return status;
  }
  /* ... */
  return status;
}
```

---

## ⚠ Achtung
<!-- notes
return type nicht gecheckt
-->
```c
/* --> */ diffie_hellman(/* ... */);
if (status != 0) {
  return status;
}
```

---

## Lösung

### C
```c
int diffie_hellman(/* ... */) __attribute__((warn_unused_result));
```
### C++ (17)
```cpp
[[nodiscard]] // Auch für Typen
```

---

## Bequemlichkeit

🙁
```c
if (status != 0) {
  return status;
}
```

---

## Bequemlichkeit

🙂
```c
#define on_error if (status != 0)

on_error {
  return status;
}
```

---

## Ressourcenverwaltung
```c
char *buffer1 = malloc(size1);
/* .. */

on_error {
  free(buffer1);
  return status;
}

char *buffer2 = malloc(size2);

/* .. */

on_error {
  free(buffer1);
  free(buffer2);
  return status;
}

```

---

## ⚠ Memory Leak
```diff
diff --git a/lib/message.c b/lib/message.c
index fb2f0d30d..60897128a 100644
--- a/lib/message.c
+++ b/lib/message.c
@@ -152,6 +152,7 @@ int decrypt_message(
                        key);
        if (status != 0) {
                sodium_memzero(plaintext_buffer, PLAINTEXT_LENGTH);
+               free(plaintext_buffer);
                return status;
        }

@@ -164,6 +165,7 @@ int decrypt_message(
        //copy plaintext to message (output)
        memcpy(message, plaintext_buffer, *message_length);
        sodium_memzero(plaintext_buffer, PLAINTEXT_LENGTH);
+       free(plaintext_buffer);

        //copy header length to output
        *header_length = HEADER_LENGTH;
```

---

## Goto considered beneficial

```c
char *buffer1 = NULL;
char *buffer2 = NULL;
/* .. */
buffer1 = malloc(size1);
/* .. */
on_error {
  goto cleanup;
}
buffer2 = malloc(size2);
/* .. */
on_error {
  goto cleanup;
}
/* .. */
cleanup:
  if (buffer1 != NULL) {
    free(buffer1);
  }
  if (buffer2 != NULL) {
    free(buffer2);
  }

  return status;
```

---

## Mehr Bequemlichkeit

```c
#define free_and_null(pointer)\
  free(pointer);\
  pointer = NULL;

#define free_and_null_if_valid(pointer)\
  if (pointer != NULL) {\
    free_and_null(pointer)\
  }
```

---

## Mehr Bequemlichkeit

```c
char *buffer1 = NULL;
char *buffer2 = NULL;
/* .. */
buffer1 = malloc(size1);
/* .. */
on_error {
  goto cleanup;
}
buffer2 = malloc(size2);
/* .. */
on_error {
  goto cleanup;
}
/* .. */
cleanup:
  free_and_null_if_valid(buffer1);
  free_and_null_if_valid(buffer1); /* BUG verhindert (double free)! */
  free_and_null_if_valid(buffer2);

  return status;
```

---

## Fehlertypen

```c
typedef enum status_type {
  SUCCESS = 0,
  GENERIC_ERROR,
  INVALID_INPUT,
  INVALID_VALUE,
  INCORRECT_BUFFER_SIZE,
  /* ... */
} status_type;
```

---

# Let's get crazy

---

## Stacktraces (oder so ...)
<!-- notes
Wer aus dem Publikum hat schon eine linked list implementiert?
-->

```c
typedef struct error_message error_message;
struct error_message {
  const char * message;
  status_type status;
  error_message *next; // <-- linked list
};

typedef struct return_status {
  status_type status;
  error_message *error;
} return_status;

return_status return_status_init();

status_type return_status_add_error_message(
		return_status *const status_object,
		const char *const message,
		const status_type status) __attribute__((warn_unused_result));
```

---

## Stacktraces (oder so ...)

<!-- notes
Kommt das "keyword" jemandem bekannt vor?
-->

```c
#define throw(status_type_value, message) {\
	status.status = status_type_value;\
	if (message != NULL) {\
		status_type type = return_status_add_error_message(&status, message, status_type_value);\
		if (type != SUCCESS) {\
			status.status = type;\
		}\
	} else {\
		status.error = NULL;\
	}\
\
	goto cleanup;\
}

#define on_error if (status.status != SUCCESS)

#define throw_on_error(status_type_value, message) \
  on_error {\
    throw(status_type_value, message)\
  }
```

---

## Ergebnis

```c
return_status status = return_status_init();
char *buffer = NULL;

/* ... */
status = funktion();
throw_on_error(GENERIC_ERROR, "Aufruf ist fehlgeschlagen")

cleanup:
  free_and_null_if_valid(buffer);

  return status;
```
