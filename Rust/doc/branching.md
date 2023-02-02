[Back](../README.md)

---

# Branching

Одно из основных средств в программировании. С помощью ветвления можно разветвлять код, чтобы он выполнял разные выражения в зависимости от условий.

---

## `if`

Выражение `if` позволяет разветвлять код в заисимости от условий. Если условие было соблюдено, то выполнится этот блок кода.

```rust
fn main() {
    let number = 3;

    if number <= 5 { // 3 меньше или равно 5
        println!("Now you work");
    } else { // так как одно из предыдущих выражений было выполнено, то этот блок кода не выполняется.
        println!("Chill")
    }
}
```

---

## `else if`

Если мы хотим, чтобы у программы было больше двух реакций, то мы используем между `if` и `else` конструкцию `else if`, где передаем следующие условия.

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

Ответом будет `number is divisible by 3` так как это выраждение стоит выше `else if number % 2 == 0`. Если какое либо условие выполняется, то следующие пропускаются.
Аналогом для ветвлением является `match`, которая желательна при использовании более 2 конструкций `if`

Так же доступны тернанрные опперации:

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}"); // The value of number is: 5
}
```

Но тернарные опперации должны приводить к выводу одного типа, если это условие проигнорировать, то мы получим ошибку `'if' and 'else' have incompatible types`

---

# `while`

Ключевое слово `while` обозначает для rust блок цикла. `while` являектся разновидностью цикла, эта конструкция позволяет не писать `break`, а указать условие, которое будет запускать итерации до тех пор, пока условие истинно.
Пример перебора всех элементов массива:

```rust
fn main() {
    let array_numbers = [10,20,30,40,50];
    let mut i = 0;
    while i < array_numbers.len() {
        println!("{}", array_numbers[i]);
        i += 1;
    }
}
```

---

# `loop`

Ключевое слово `loop` обозначает для rust блок цикла.

```Rust
fn main() {
    loop {
        println!("again!");
    }
}
```

```rust
fn main() {
    let mut counter = 0;
    let result: i32;
    loop {
        counter += 1;
        if counter == 10 {
            break result = counter * 2;
        }
    };
    println!("{}", result);
}
```

Аналогично

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```

Иногда следует использовать метку цикла. Она нужна чтобы указать, к чему применяется `break` и `continue`. Метка цикла начинается с одинарной кавычки:

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop { <--------------------------\
        println!("count = {count}");                |
        let mut remaining = 10;                     |
        loop {                                      |
            println!("remaining = {remaining}");    |
            if remaining == 9 {                     | break связан
                break;                              | с эти циклом
            }                                       |
            if count == 2 {                         |
                break 'counting_up; <---------------/
            }
            remaining -= 1;
        }
        count += 1;
    }
    println!("End count = {count}");
}
```

## `break`, `continue`

`break` - остававливает выполнение цикла. Если не указанно метки цикла, `break` остановит остановит выполнение цикла.
`continue` - пропустит следующий код в цикле и перейдет к следующей итерации.

---