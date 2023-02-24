[Back](../README.md)

---

## [Инструменты/Tooling](doc/tooling.md)

* [Rustup](doc/tooling.md#rustup)
* [Cargo](doc/tooling.md#cargo)

---

## [Переменные/Variables](doc/variables.md)

* `let`
* `let mut`
* `const`

---

## [Типы данных/Data Types](doc/datatypes.md)

* [Скалярные/Scalar](doc/datatypes/scalar.md)

  * [Integer](doc/datatypes/scalar.md#integer)
  * [Float](doc/datatypes/scalar.md#float)
  * [Boolean](doc/datatypes/scalar.md#bool)

* [Составные/Compound](doc/datatypes/compound.md)

  * [String](doc/datatypes/compound.md#string)
  * [Tuples](doc/datatypes/compound.md#tuples)
  * [Array](doc/datatypes/compound.md#array)

---

## [Функции/Functions](doc/functions.md)

* [Basic](doc/functions.md#function)
* [Taking arguments](doc/functions.md#taking-arguments)
* [Returning arguments](doc/functions.md#returning-arguments)

---

## [Ветвление/Branching](doc/branching.md)

* [`if`, `else`, `else if`](doc/branching.md#branching)
* `while`
* `loop`

---

## [Владение/Ownership](doc/ownership.md)

* [Stack](doc/ownership.md#stack)
  * [Difference](../Other/stack_heap_realization.md#stack)
* [Heap](doc/ownership.md#heap)
  * [Difference](../Other/stack_heap_realization.md#heap)
* [Scope](doc/ownership.md#scope)
  * [RAII](doc/ownership.md#raii)
  * [Ownership and Move](doc/ownership.md#ownership-and-move)
    * [Partial moves](doc/ownership.md#partial-moves)
  * [Borrowing](doc/ownership.md#borrowing)
    * [Mutability](doc/ownership.md#mutability)
  * [Lifetimes](doc/ownership.md#lifetimes)
    * [Explicit annotation](doc/ownership.md#explicit-annotation)
    * [Functions](doc/ownership.md#functions)
    * [Methods/Structs](doc/ownership.md#methodsstructs)
    * [Traits](doc/ownership.md#traits)
    * [Bounds](doc/ownership.md#bounds)
* [Allocation](doc/ownership.md#allocation)
* [Move / Clone / Copy](doc/ownership.md#moveclonecopy)
* [References](doc/ownership.md#references)
* [Borrowing](doc/ownership.md#borrowing)
* [Slices](doc/ownership.md#slices)

---

## [Структуры/Structures](doc/structures.md)

* [struct](doc/structures.md#struct)
  * [Tuple struct without named fields](doc/structures.md#tuple-struct-without-named-fields-to-create-different-types)
  * [Unit-like struct without any fields](doc/structures.md#unit-like-structs-without-any-fields)
* [Ownership on structs](doc/structures.md#ownership-on-structs)
  * [Defining methods](doc/structures.md#defining-methods)
    * [Traits](doc/structures.md#traits)
  * [Associated functions](doc/structures.md#associated-functions)
* [Example](doc/structures.md#example)

---

## [Перечисление/Enum](doc/enum.md#enum)

* [Enums](doc/enum.md#enum)
  * [Option<T>](doc/enum.md#option)
  * [Enum and match](doc/enum.md#enum-and-match)
  * [Explicit let if in match](doc/enum.md#explicit-let-if-in-match)
  * [Universal parameters in match](doc/enum.md#universal-parameters-in-match)

---

## [Code organization](doc/code_organization.md#code-organization)

  * [Packages and crates](doc/code_organization.md#packages-and-crates)
* [Scope on modules](doc/code_organization.md#scope-on-modules)
  * [Group code in linked modules](doc/code_organization.md#group-code-in-linked-modules)
  * [Road for link to the element in modules tree](doc/code_organization.md#road-for-link-to-the-element-in-modules-tree)
    * [Using `use`](doc/code_organization.md#using-use)
    * [Reexport names from `pub` `use`](doc/code_organization.md#reexport-names-from-pub-use)
    * [Using nested paths with `use`](doc/code_organization.md#using-nested-paths-with-use)
  * [Example](doc/code_organization.md#example)
* [Separating Modules into Different Files](doc/code_organization.md#separating-modules-into-different-files)

---

## [Коллекции/Collections](doc/collections.md#collections)

* [Sequences](doc/collections.md#sequences)
  * [Vec](doc/collections.md#vec)
  * [VecDeque](doc/collections.md#vecdeque)
  * [LinkedList](doc/collections.md#linkedlist)
* [Maps](doc/collections.md#maps)
  * [HashMap](doc/collections.md#hashmap)
  * [BTreeMap](doc/collections.md#btreemap)
* [Sets](doc/collections.md#sets)
  * [HashSet](doc/collections.md#hashset)
  * [BTreeSet](doc/collections.md#btreeset)
* [Misc](doc/collections.md#misc)
  * [BinaryHeap](doc/collections.md#binaryheap)

---

## [Ошибки/Errors](doc/errors.md#errors)

* [Unrecoverable Errors with `panic!`](doc/errors.md#unrecoverable-errors-with-panic)
  * [Unwinding the Stack or Aborting in Response to a Panic](doc/errors.md#unwinding-the-stack-or-aborting-in-response-to-a-panic)
* [Recoverable Errors with Result](doc/errors.md#recoverable-errors-with-result)
  * [Shortcuts for Panic on Error: `unwrap` and `expect`](doc/errors.md#shortcuts-for-panic-on-error-unwrap-and-expect)
  * [Propagating Errors](doc/errors.md#propagating-errors)
  * [Catching panic with `catch_unwind`](doc/errors.md#catching-panic-with-catch_unwind)
    * [A Shortcut for Propagating Errors: the `?` Operator](doc/errors.md#a-shortcut-for-propagating-errors-the--operator)
      * [Where The `?` Operator Can Be Used](doc/errors.md#where-the--operator-can-be-used)

---

## [Declarations](doc/declarations.md#declarations)

---
