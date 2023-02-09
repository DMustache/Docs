[Back](../README.md)

---

# Function

Основной функцией в rust является функция `main`. Из этой функции вызываются другие учаски программы. Для определения функции, необходимо указать `fn` за которым идет имя функции и набор передаваемых значений в крглых скобках. Фигурные скобки указывают компиляторы на тело функции. Функции можно указать до определения или после.
Определение функция является операторами, по этому нельзя присвоить значение оператора `let` другой переменной, так как этот оператор не возвращает значений.

```rust
fn main() {
    another_function(5);
}

fn another function(x: i32) {
    println!("It`s {}", x);
}

```

Пример выражения в новой области видимости:

```rust
fn main() {
    let y = { // оператор - ; стоит
        let x = 3; // оператор - ; стоит
        x + 1 // выражение - ; не стоит
    };

    println!("The value of y is: {y}"); // The value of y is: 4
}
```

---

# Taking arguments

Для передачи перенных в функцию неообходимо указать имя переменной в функции и тип используемых аргументов.

```rust
fn five(number: i32){
    println!("{}", number)
}

fn main() {
    five(5); // 5
}
```

---

# Returning arguments

Для возврата значений, функции необходимо указать имп возвращаемой переменной после ():

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}"); // 5
}
```

Чтобы переменная возвращалась, у нее не должно быть `;`. Но это делает код ужасно плохо читаемым. Лучший способ вернуть значение, это использовать `return`

Пример с `return`:
```rust
fn function_name() -> return_type {
   //statements
   return value;
}
```

Пример без `return`, с использованием метода реализации без `;`:

```rust
fn function_name() -> return_type {
   // statements
   value //no semicolon means this value is returned
}
```

---

# [Arrow functions](https://doc.rust-lang.org/rust-by-example/fn/closures.html)

Стрелочные функции могут быть использованны как в структурах, так и в функциях. Они могут использовать переменные, которые находятся в окружении, где инициалищированна стрелочная функция.

```rust
fn main() {
    let temp = 43;

    let num_plus = |i: i32| -> i32 {i + 1};
    let minus_one = |i: i32| -> i32 {i - 1};

    println!("{}", num_plus(temp)); // 44
    println!("{}", minus_one(temp)); // 42
}
```


JVT 0 Token
GRPC WEB
