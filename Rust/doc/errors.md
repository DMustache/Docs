[Back](../README.md)

# Errors

Обработка ошибок позволяет управлять кодом в зависимости от ошибки, полученной во время ее возникновения. Обнаружение и явное управление ошибками спасе программу от различных ловушек. Существует множество различных способов обработки ошибок в rust, они имеют свои тонкие различия и варианты использования.

Rust в отличии от других яп различает 2 категории ошибки: исправимые и неисправимые.
* Исправимые - ошибка, которую пользователь может решить во время использования программы. Например, программа не видит указанный пользователем файл и сообщает ему это. Программа при этом продолжает свою работу.
* Неисправимые - ошибки, допущенные разработчиком, например ошибка с выходом курсора за пределы массива. В этом случае требуется немедленно остановить программу.

В Rust нет исключений, в место этого он имеет тип `Result<T, E>` ждя исправимых ошибок и `panic!` макрос, который останавливает работу программы в случае неисправимой ошибкой.

Код `panic` полезен для тестов и работы с неисправимыми ошибками. Это полезно для тестовых версий проекта. Например, при работе с функциями, которые еще не были реализованы, но в этих случаях лучше `unimplemented`. В тестах, `panic` - разумный способ указать на провал.

Тип `Option` предназначен для случаев, когда значение является необязательным или отсутствие значения не является ошибкой. Например при работе с `Option` `unwrap` отлично подходит для прототипа, когда абсолютно точно гарантированно наличие значения. Однако `except` более полезен, поскольку позволяет указать сообщение об ошибке на случай, Если что-то все равно пойдет не так.

Когда есть шанс получить баг, `Result` является лучшем решением. Его можно `unwrap` и `except` **только во время разработки прототипа**у

## Unrecoverable Errors with `panic!`

Иногда, плохие вещи могут произойти в коде, и они бывают необратимыми, в этом случае помогает макрос `panic!`. На практике есть 2 способа вызвать панику: вызов элемента массива, не относящегося к нему, явный вызов `panic!`. В обоих случаях мы вызываем панику в программе. По умолчанию эти паники выводят сообщение об ошибке в терминале, раскручиваются, очищают стек и завершают работу программы. С помощью переменной среды можно заставить rust отображать стек вызова при возникновения паники, чтобы простить отслеживания вызова паники.

### Unwinding the Stack or Aborting in Response to a Panic

По умолчанию, когда возникает паника, программа начинает раскручиваться, что означает, что Rust просматривает стек и очищает данные от каждой функции, с которой сталкивается. Тем не менее, это возвращение и уборка - это много работы. Таким образом, Rust позволяет вам выбрать альтернативу немедленного прерывания, которая завершает программу без очистки.

Операционная система должна очистить память, которую использовала программа, то если в проекте нужно сделать результирующий бинарник как можно меньше, вы можете переключиться с раскручивания на прерывание при панике, добавив в `panic = 'abort'` соответствующие `[profile]` секции в вашем файле `Cargo.toml`. Например, если вы хотите прервать панику в режиме деблокирования, добавьте следующее:

```rust
[profile.release]
panic = 'abort'
```

---

Попробуем вызвать панику:

```rust
fn main() {
    panic!("It's panic")
}
```

```
   Compiling guessing_game v0.1.0 (/home/dmustache/projects/rust/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 0.64s
     Running `target/debug/guessing_game`
thread 'main' panicked at 'It's panic', src/main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

В отличии от случая, когда мы сами вызываем панику, библиотечные паники попытаются указать проблему и способ ее решения

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

```
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/guessing_game`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
Нам так же позволяют показать трассировку вызова ошибки

```
$ RUST_BACKTRACE=1 cargo run
```
```
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/guessing_game`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/fc594f15669680fa70d255faec3ca3fb507c3405/library/std/src/panicking.rs:575:5
   1: core::panicking::panic_fmt
             at /rustc/fc594f15669680fa70d255faec3ca3fb507c3405/library/core/src/panicking.rs:64:14
   2: core::panicking::panic_bounds_check
             at /rustc/fc594f15669680fa70d255faec3ca3fb507c3405/library/core/src/panicking.rs:147:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/fc594f15669680fa70d255faec3ca3fb507c3405/library/core/src/slice/index.rs:260:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/fc594f15669680fa70d255faec3ca3fb507c3405/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/fc594f15669680fa70d255faec3ca3fb507c3405/library/alloc/src/vec/mod.rs:2727:9
   6: guessing_game::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/fc594f15669680fa70d255faec3ca3fb507c3405/library/core/src/ops/function.rs:507:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

Это то, что происходит во время паники, чтобы еще больше показать, мы можем поставить `full`, о чем нам говорит последняя строчка. Вывод может отличаться от ОС и версии языка. Если использовать `--release`, то вывод будет меньше.

## Recoverable Errors with Result

Большинство ошибок не достаточно серьезные, чтобы требовать полной остановки программы. Ингода, когда функция дает сбой, это приводит к причине, которые вы можете легко интерпретировать и отреагировать на нее. Например, если вы пытаетесь открыть несуществующий файл, возникнет исправимая ошибка, которую можно исправить путем создания необходимого файла.

Давайте вызовем функцию, которая возвращает `Result` значение, потому что функция может завершиться ошибкой.

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = dbg!( File::open("hello.txt"));
}
```

```
[src/main.rs:4] File::open("hello.txt") = Err(
    Os {
        code: 2,
        kind: NotFound,
        message: "No such file or directory",
    },
)
```

Возвращаемый тип — `File::open` это `Result<T, E>`. Общий параметр `T` был заполнен реализацией `File::open` с типом значения успеха, `std::fs::File` который является дескриптором файла `E`. В значении ошибки используется тип `std::io::Error`. Этот тип возврата означает, что вызов `File::open` может завершиться успешно и вернуть дескриптор файла, из которого мы можем читать или записывать. Вызов функции также может завершиться ошибкой: например, файл может не существовать или у нас может не быть разрешения на доступ к файлу. У `File::open` функции должен быть способ сообщить нам, успешно ли она выполнена или нет, и в то же время дать нам либо дескриптор файла, либо информацию об ошибке. Эта информация именно то, что `Result` передает перечисление.

В случае удачного выполнения программы, мы получим следующее:

```
[src/main.rs:4] File::open("src/filer.txt") = Ok(
    File {
        fd: 3,
        path: ".../guessing_game/src/filer.txt",
        read: true,
        write: false,
    },
)
```

В случае исправимой ошибки мы можем определить дальнейшее поведение кода в зависимости от `Result<T, E>`:

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```

Так же, благодаря `match` и `Result<T, E>` мы можем определять ошибки и дальше:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```

[`Result<T, E>` source](https://doc.rust-lang.org/std/result/)


Альтернативой `match` является `if else`.

### Shortcuts for Panic on Error: `unwrap` and `expect`

Ярлыки являются специфическими методами для решения более специфических задач. `unwrap` метод является сокращенным выражением `match`.

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```

```
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:4:49
```

Аналогично, `except` метод позволяет выбрать сообщение паники.

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```

---

### Propagating Errors

Распространение ошибок дает больший контроль вызывающему коду, где может быть больше информации или логики, определяющих, как следует обрабатывать ошибку, чем то, что реализованно нами.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

Так как возвращающий тип функции `Result<String, io::Error>`, это означает, что функция возвращает значение типа `Result<T, E>`, в котором параметр `T` был заполнен типом `String`, а `E` был заполнен `io::Error`

Если эта функция завершится без проблем, то получит `OK`, содержащее строку - имя пользователя, иначе будет отображена ошибка.

#### A Shortcut for Propagating Errors: the ? Operator

Ярлык `?` позволяет добиться той же реализации с меньшими затратами

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    return Ok(username); // хаха, мы не возвращаем приколы си с 0 или 1, или как там, у нас есть Ok, которые означает, что все четко https://t.me/extremecode/719
}
```

`?` показывает, код будет аналогично прошлому примеру, то есть как `match` выражение. Несмотря на это, код работает похоже, но не идентично. `?` пропускает возвращенные значения через `from` функцию, определенную в `From` свойстве стандартной библиотеки, которая используется для преобразования значений из одного типа в другой. Когда `?` вызывает `from` функцию, полученный тип ошибки преобразуется в тип ошибки, определенный в возвращаемом теле функции. Это полезно, когда функция возвращает один тип ошибки, чтобы представить все возможные сбои функции, даже если некоторые части могут выйти из строя по разным причинам.

Оператор `?` устраняет множество шаблонов и упрощает реализацию этой функции. Мы могли бы еще больше сократить этот код, связав вызовы методов сразу после `?`:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    return Ok(username);
}
```

##### Where The ? Operator Can Be Used

Оператор `?` можно использовать только в функциях, тип возвращаемого значения которых совместим со значением, для которого он используется. Это связанно с тем, что `?` оператор определен для досрочного возврата значения из функции, аналогично выражению `match` в прошлых примерах.

Так, например, если вы вызовем `?` в `main` функции, то получим ошибку, потому что `?` возвращает `Result`, а `main` - `()`. Чтобы исправить эту неисправимую ошибку, то можно либо изменить тип возвращаемого значения, использовании `match` или одного из `Result<T, E>` методов для обработки `Result<T, E>` любым подходящим способом.

Вот один из примеров исправления ошибки

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```

Но мы обязаны указать возвращаемое значение.

Примечание. Функция `main` может возвращать любые типы, реализующие типаж [`типаж std::process::Termination`](https://doc.rust-lang.org/std/process/trait.Termination.html), которые содержит функцию `report`, возвращающую `ExitCode`.


<!-- Когда, что использовать: Паникуем мы когда нельзя восстановить надежность программы, Результы наоборот, могут использоваться, чтобы исправить ошибку путем выбора, что делать, если есть ошибка, если ошибки нет. -->

---
