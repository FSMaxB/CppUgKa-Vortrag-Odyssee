<!-- C++ Error Handling -->

# Wieder Error Handling

---

## Wrapper um return_status

```cpp
struct Error {
  status_type type{type::status::SUCCESS};
  std::string message;

  Error(const status_type type, const std::string& message);
  /* ... */
}

class Exception : public std::exception {
private:
  std::deque<Error> error_stack;
  mutable std::string printed;

public:
  Exception(const Error& error);
  Exception(const status_type type, const std::string& message);
  Exception(return_status& status);
}
```

---

## Entfernen des Error-Stacks

---

<!-- note
Because: Android is weird
-->

## Die Katastrophe passiert
### Ich kann keine Exceptions mehr verwenden
* SEGFAULT beim throwen

---

## Ich kann keine Exceptions mehr verwenden
* Keine Fehler mehr in Konstruktoren möglich

---

## Boost Outcome to the rescue
```cpp
template <typename Result>
using result = outcome::result<Result,Error>;
```

---

## Private unvollständige Konstruktoren

<!-- note
Marker um vom Default-Konstruktor zu unterscheiden
-->

```cpp
// Marker-Typ
struct uninitialized_t {
 explicit uninitialized_t() = default;
};

class MasterKeys {
  /* ... */
private:
  MasterKeys(uninitialized_t uninitialized) noexcept;
  /* ... */
public:
		static result<MasterKeys> create(/* ... */);
}
```

---

## Fehlerpropagierung

```cpp
OUTCOME_TRY(do_something());
OUTCOME_TRY(value, calculate_something());
```
