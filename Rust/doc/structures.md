[Back](../README.md)

# struct

Структуры похожи на классы в других языках программирования. Но сравнивать структуры с классами других языков будет не верно, так как в разных языках Объекты, имеющие поля, методы, приватности элементов называются по разному, по разному реализовываются и имеют порой очень значимые [различия](https://www.reddit.com/r/rust/comments/vpz3kc/comment/iem2ngf/?utm_source=share&utm_medium=web2x&context=3)
Структуры похожи на кортежи, где нам не надо знать расположения элемента и того, что он делает, чтобы создавать реализацию для него. Структуры указывают имя для каждой переменной и ее тип для дальнейшей реализации.

```rust
struct Dog {
    hp: u8,
    attack: i8,
    speed: u8,
    name: String,
}

fn main() {
    let charly = Dog {
        hp:10,
        attack:3,
        speed:32,
        name: String::from("Charly"),
    };
    println!("{} is strong, his is: {}", charly.name, charly.hp);
    println!("{} is fast, his speed is: {}", charly.name, charly.speed);
    println!("{} is strong, his hp is: {}", charly.name, charly.hp);
    println!("{} is dangerous, his attack is: {}", charly.name, charly.attack);
}
```

```
Charly is strong, his is: 10
Charly is fast, his speed is: 32
Charly is strong, his hp is: 10
Charly is dangerous, his attack is: 3
```

Переменная, являющаяся некой структурой работает как и любая другая переменная, то есть если переменная изменяемая, то ее структуру можно изменять, если нет, то и поля нельзя изменять.

Так как в структуре используются поля с неким типом, то мы можем из другой структуры взять переменные и положить в эту структуру

Можно использовать неявное добавление необозначенных полей структуры, для этого надо использовать `..<other_struct>` в конце объявления.

```rust
fn main() {
    let mut charly = Dog {
        hp:10,
        attack:3,
        speed:32,
        name: String::from("Charly"),
    };

    let rex = Dog {
        name: String::from("Rex"),
        ..charly
    };

    println!("attack {}", rex.attack == charly.attack); // True
    println!("speed {}", rex.speed == charly.speed); // True
    println!("hp {}", rex.hp == charly.hp); // True
    println!("name {}", rex.name == charly.name); // False

}
```

В случае, когда применяется объявления структуры, которая получает элементы из другой структуры, то если в новую структуру перейдут элементы, которые хранят свои данные в куче, старую структуру использовать будет запрещено. Мы получим эту ошибку: `note: move occurs because 'charly.name' has type 'String', which does not implement the 'Copy' trait`

```rust
struct Dog<'a> {
    hp: u8,
    attack: i8,
    speed: u8,
    name: &'a str,
}
fn main() {
    let mut charly = Dog {
        hp:10,
        attack:3,
        speed:32,
        name: "Charly",
    };

    let rex = Dog {
        name: "Rex",
        ..charly
    };


    println!("attack {}", rex.attack == charly.attack); // True
    println!("speed {}", rex.speed == charly.speed); // True
    println!("hp {}", rex.hp == charly.hp); // True
    println!("name {}", rex.name == charly.name); // False

}
```

## Tuple struct without named fields to create different types

Кортежи, объявленные как структуры называются кортежными структурами. Они не имеют имен полей. У них есть только типы полей. Это полезно, когда требуется выделить некоторый кортеж.

```rust
struct  Color(u8, u8, u8);

fn main() {
    let black = Color(0, 0, 0);
    let white = Color(255, 255, 255);
}
```

## Unit-like structs without any fields

Можно так же создавать пустые структуры(без полей). Эти Юнит-подобные структуры ведут себя аналогично `()`. Вот реальный пример:

[Chrono](https://docs.rs/chrono/0.2.25/chrono/offset/utc/struct.UTC.html) - `chrono::Utc` имеет `pub struct Utc` который не имеет состояния, но реализует `TimeZone` и другие. Эта структура может использоваться в качестве универсального аргумента для `Date` и `DateTime`.

# Ownership on structs

Структуры могут хранить ссылки на данные, принадлежащие кому-то другому, но для этого надо использовать [время жизни](ownership.md#lifetimes) Если этому не следовать, то мы получим ошибку на стадии компиляции:

```rust
struct User {
    is_active: bool,
    username: &str,
    email: &str,
    sign_in_count: u64,
}

fn main() {
    let user = User {
        is_active: true,
        username: "Dmustache",
        email: "dmustache@gmail.com",
        sign_in_count: 1,
    };
}
```

```
   Compiling guessing_game v0.1.0 (/home/dmustache/projects/rust/guessing_game)
error[E0106]: missing lifetime specifier
 --> src/main.rs:3:15
  |
3 |     username: &str,
  |               ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     is_active: bool,
3 ~     username: &'a str,
  |

error[E0106]: missing lifetime specifier
 --> src/main.rs:4:12
  |
4 |     email: &str,
  |            ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     is_active: bool,
3 |     username: &str,
4 ~     email: &'a str,
  |

For more information about this error, try `rustc --explain E0106`.
error: could not compile `guessing_game` due to 2 previous errors
```
Чтобы избежать проблемы, мы можем перейти к String:

```rust
struct User {
    is_active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let user = User {
        is_active: true,
        username: dbg!(String::from("Dmustache")),
        email: String::from("dmustache@gmail.com"),
        sign_in_count: 1,
    };

    println!("{} {} {} {}", user.email, user.is_active, user.username, user.sign_in_count);
}
```

## Defining methods

Чтобы функции, в которых используется структура, не были доступны публично, мы поместим функции в тело структуры. Для этого после определения структуры мы используем `impl <struct>` и внутри определяем функции.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

Мы можем своим полям и методам давать одинаковые названия, если мы не поставим в вызове скобки - это будет поле, иначе метод.
Эта практика применяется не всегда, когда мы даем методу то же имя, что и полю, мы хотим, чтобы метод возвращал только значение и ничего больше не делал. Такие методы называются __геттерами__, rust реализует их автоматически для полей структуры. Геттеры позволяют делать поля закрытыми, а методы общедоступными, и, таким образом позволяют только читать поля.

Когда вызывается метод с помощью `object.something()`, rust автоматически добавляет `&`, `&mut` или `*`, поэтому `object` соответствует сигнатуре метода. Вот две строчки, которые делают абсолютно одно и то же:

```rust
p1.distance(&p2)
&p1.distance(&p2)
```

Первый вариант выглядит чище. Такая автоматическая ссылка работает, потому что у методов есть четкий получатель - `self`. Зная получателя и имя метода, Rust может определить, определен метод на чтение, изменение или потребление. Тот факт, что rust делает метод заимствования неявным для получателей методов, является важной частью практической эргономичности [владения](ownership.md).

Важно понимать, что такое `self`. Это экземпляр класса. который позволяет обратиться к полям и методам себя.

### Traits

`trait` - набор методов, определенных для некого `Self`. Черты могут получать доступ к другим методам, объявленным в этом же трейте. Черты могут быть реализованны для любого типа данных.

```rust
struct Sheep { naked: bool, name: &'static str }

trait Animal {
    // Associated function signature; `Self` refers to the implementor type.
    fn new(name: &'static str) -> Self;

    // Method signatures; these will return a string.
    fn name(&self) -> &'static str;
    fn noise(&self) -> &'static str;

    // Traits can provide default method definitions.
    fn talk(&self) {
        println!("{} says {}", self.name(), self.noise());
    }
}

impl Sheep {
    fn is_naked(&self) -> bool {
        self.naked
    }

    fn shear(&mut self) {
        if self.is_naked() {
            // Implementor methods can use the implementor's trait methods.
            println!("{} is already naked...", self.name());
        } else {
            println!("{} gets a haircut!", self.name);

            self.naked = true;
        }
    }
}

// Implement the `Animal` trait for `Sheep`.
impl Animal for Sheep {
    // `Self` is the implementor type: `Sheep`.
    fn new(name: &'static str) -> Sheep {
        Sheep { name: name, naked: false }
    }

    fn name(&self) -> &'static str {
        self.name
    }

    fn noise(&self) -> &'static str {
        if self.is_naked() {
            "baaaaah?"
        } else {
            "baaaaah!"
        }
    }

    // Default trait methods can be overridden.
    fn talk(&self) {
        // For example, we can add some quiet contemplation.
        println!("{} pauses briefly... {}", self.name, self.noise());
    }
}

fn main() {
    // Type annotation is necessary in this case.
    let mut dolly: Sheep = Animal::new("Dolly");
    // TODO ^ Try removing the type annotations.

    dolly.talk();
    dolly.shear();
    dolly.talk();
}
```

```
Dolly pauses briefly... baaaaah!
Dolly gets a haircut!
Dolly pauses briefly... baaaaah?
```

#### Derive

Компилятор может реализовать базовые реализации некоторых трейтов через `#[derive]` атрибут. Эти черты можно модифицировать.

Список производных трейтов:

* Сравнение: `Eq`, `PartialEq`, `Ord`, `PartialOrd`
* Копия `&T` из `T`: `Clone`
* Указать, что в данной реализации используется копирование, а не перемещение: `Copy`
* Вычисление хеша из `&T`: `Hash`
* Пустой экземпляр типа данных: `Default`
* Форматирование представления значения с помощью `{:?}`: `Debug`

Черты можно объединять для того чтобы использовать вместе.

```rust
// `Centimeters`, a tuple struct that can be compared
#[derive(PartialEq, PartialOrd)]
struct Centimeters(f64);

// `Inches`, a tuple struct that can be printed
#[derive(Debug)]
struct Inches(i32);

impl Inches {
    fn to_centimeters(&self) -> Centimeters {
        let &Inches(inches) = self;

        Centimeters(inches as f64 * 2.54)
    }
}

// `Seconds`, a tuple struct with no additional attributes
struct Seconds(i32);

fn main() {
    let _one_second = Seconds(1);

    // Error: `Seconds` can't be printed; it doesn't implement the `Debug` trait
    //println!("One second looks like: {:?}", _one_second);
    // TODO ^ Try uncommenting this line

    // Error: `Seconds` can't be compared; it doesn't implement the `PartialEq` trait
    //let _this_is_true = (_one_second == _one_second);
    // TODO ^ Try uncommenting this line

    let foot = Inches(12);

    println!("One foot equals {:?}", foot);

    let meter = Centimeters(100.0);

    let cmp =
        if foot.to_centimeters() < meter {
            "smaller"
        } else {
            "bigger"
        };

    println!("One foot is {} than one meter.", cmp);
}
```

```
One foot equals Inches(12)
One foot is smaller than one meter.
```

#### Returning Traits with `dyn`

Компилятор Rust должен знать, сколько места нужно для каждого возвращаемого значения функции. Это значит, что все функции должны возвращать конкретный тип. В отличии от других яп, если у структуры есть такая черта, как `Animal`, нельзя написать функцию, которая будет возвращать тип `Animal`, Потому что для ее разных реализаций может потребоваться разное количество памяти. Однако, если все же требуется так сделать, вместо того, чтобы возвращать трейт-объект, функция может возвращать `Box`, который будет содержать некоторый `Animal`. Так как `Box` - ссылка на участок памяти в куче, то технически, функция знает размер возвращаемого значения, по-этому ошибка не возникнет.

Так как rust старается быть как можно более явным яп, то для того, чтобы показать, что мы хотим сохранить трейт-объект в куче, то мы должны указать перед трейтом `dyn`: `Box<dyn Animal>`.

```rust
struct Sheep {}
struct Cow {}

trait Animal {
    // Instance method signature
    fn noise(&self) -> &'static str;
}

// Implement the `Animal` trait for `Sheep`.
impl Animal for Sheep {
    fn noise(&self) -> &'static str {
        "baaaaah!"
    }
}

// Implement the `Animal` trait for `Cow`.
impl Animal for Cow {
    fn noise(&self) -> &'static str {
        "moooooo!"
    }
}

// Returns some struct that implements Animal, but we don't know which one at compile time.
fn random_animal(random_number: f64) -> Box<dyn Animal> {
    if random_number < 0.5 {
        Box::new(Sheep {})
    } else {
        Box::new(Cow {})
    }
}

fn main() {
    let random_number = 0.234;
    let animal = random_animal(random_number);
    println!("You've randomly chosen an animal, and it says {}", animal.noise());
}
```

```
You've randomly chosen an animal, and it says baaaaah!
```

### Operator Overloading

В rust, множество операторов может быть перегружено через трейты. То есть некоторые операторы могут использоваться для выполнения различных задач в зависимости от их входных аргументов. Это возможно, потому что операторы являются синтаксическим сахаром для вызова методов. Например `+` вызывает метод `add`. Этот `add` метод является частью черты `Add`. Следовательно `+` может использоваться любым наследником трейта `Add`

Список таких признаков, как `Add` можно найти в [`core::ops`](https://doc.rust-lang.org/core/ops/)

```rust
use std::ops;

struct Foo;
struct Bar;

#[derive(Debug)]
struct FooBar;

#[derive(Debug)]
struct BarFoo;

// The `std::ops::Add` trait is used to specify the functionality of `+`.
// Here, we make `Add<Bar>` - the trait for addition with a RHS of type `Bar`.
// The following block implements the operation: Foo + Bar = FooBar
impl ops::Add<Bar> for Foo {
    type Output = FooBar;

    fn add(self, _rhs: Bar) -> FooBar {
        println!("> Foo.add(Bar) was called");

        FooBar
    }
}

// By reversing the types, we end up implementing non-commutative addition.
// Here, we make `Add<Foo>` - the trait for addition with a RHS of type `Foo`.
// This block implements the operation: Bar + Foo = BarFoo
impl ops::Add<Foo> for Bar {
    type Output = BarFoo;

    fn add(self, _rhs: Foo) -> BarFoo {
        println!("> Bar.add(Foo) was called");

        BarFoo
    }
}

fn main() {
    println!("Foo + Bar = {:?}", Foo + Bar);
    println!("Bar + Foo = {:?}", Bar + Foo);
}
```

```
> Foo.add(Bar) was called
Foo + Bar = FooBar
> Bar.add(Foo) was called
Bar + Foo = BarFoo
```

#### Drop Trait

Трейт `Drop` имеет только один метод: `drop`. который вызывается автоматически, когда объект выходит за границы области видимости. Основное использование трейта `Drop` - освобождение ресурсов после использования:

```rust
struct Droppable {
    name: &'static str,
}

// This trivial implementation of `drop` adds a print to console.
impl Drop for Droppable {
    fn drop(&mut self) {
        println!("> Dropping {}", self.name);
    }
}

fn main() {
    let _a = Droppable { name: "a" };

    // block A
    {
        let _b = Droppable { name: "b" };

        // block B
        {
            let _c = Droppable { name: "c" };
            let _d = Droppable { name: "d" };

            println!("Exiting block B");
        }
        println!("Just exited block B");

        println!("Exiting block A");
    }
    println!("Just exited block A");

    // Variable can be manually dropped using the `drop` function
    drop(_a);
    // TODO ^ Try commenting this line

    println!("end of the main function");

    // `_a` *won't* be `drop`ed again here, because it already has been
    // (manually) `drop`ed
}
```

```
Exiting block B
> Dropping d
> Dropping c
Just exited block B
Exiting block A
> Dropping b
Just exited block A
> Dropping a
end of the main function
```

#### Iterator trait

Черта [`iterator`](https://doc.rust-lang.org/core/iter/trait.Iterator.html) используется для реализации итераторов над коллекциями. такими как [массивы](datatypes/compound.md#array). Черта требует, чтобы для элемента был определен только метод `next`, который может быть определен вручную или автоматически. Для удобства, `for` конструкция превращает некоторые коллекции в итераторы с помощью [`.into_iter()`](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html) метода.

```rust
struct Fibonacci {
    curr: u32,
    next: u32,
}

// Implement `Iterator` for `Fibonacci`.
// The `Iterator` trait only requires a method to be defined for the `next` element.
impl Iterator for Fibonacci {
    // We can refer to this type using Self::Item
    type Item = u32;

    // Here, we define the sequence using `.curr` and `.next`.
    // The return type is `Option<T>`:
    //     * When the `Iterator` is finished, `None` is returned.
    //     * Otherwise, the next value is wrapped in `Some` and returned.
    // We use Self::Item in the return type, so we can change
    // the type without having to update the function signatures.
    fn next(&mut self) -> Option<Self::Item> {
        let current = self.curr;

        self.curr = self.next;
        self.next = current + self.next;

        // Since there's no endpoint to a Fibonacci sequence, the `Iterator`
        // will never return `None`, and `Some` is always returned.
        Some(current)
    }
}

// Returns a Fibonacci sequence generator
fn fibonacci() -> Fibonacci {
    Fibonacci { curr: 0, next: 1 }
}

fn main() {
    // `0..3` is an `Iterator` that generates: 0, 1, and 2.
    let mut sequence = 0..3;

    println!("Four consecutive `next` calls on 0..3");
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());

    // `for` works through an `Iterator` until it returns `None`.
    // Each `Some` value is unwrapped and bound to a variable (here, `i`).
    println!("Iterate through 0..3 using `for`");
    for i in 0..3 {
        println!("> {}", i);
    }

    // The `take(n)` method reduces an `Iterator` to its first `n` terms.
    println!("The first four terms of the Fibonacci sequence are: ");
    for i in fibonacci().take(4) {
        println!("> {}", i);
    }

    // The `skip(n)` method shortens an `Iterator` by dropping its first `n` terms.
    println!("The next four terms of the Fibonacci sequence are: ");
    for i in fibonacci().skip(4).take(4) {
        println!("> {}", i);
    }

    let array = [1u32, 3, 3, 7];

    // The `iter` method produces an `Iterator` over an array/slice.
    println!("Iterate the following array {:?}", &array);
    for i in array.iter() {
        println!("> {}", i);
    }
}
```

```
Four consecutive `next` calls on 0..3
> Some(0)
> Some(1)
> Some(2)
> None
Iterate through 0..3 using `for`
> 0
> 1
> 2
The first four terms of the Fibonacci sequence are:
> 0
> 1
> 1
> 2
The next four terms of the Fibonacci sequence are:
> 3
> 5
> 8
> 13
Iterate the following array [1, 3, 3, 7]
> 1
> 3
> 3
> 7
```

#### Impl Trait

`impl Trait` можно использовать только как тип аргумента или возвращаемый тип.

Это позволяет сделать функцию более общей.

##### Как аргумент

Например, если функция является общей для типажа. Можно упростить объявление функции, используя `impl Trait` в качестве типа аргумента:

```rust
fn parse_csv_document<R: std::io::BufRead>(src: R) -> std::io::Result<Vec<Vec<String>>> {
    src.lines()
        .map(|line| {
            // For each line in the source
            line.map(|line| {
                // If the line was read successfully, process it, if not, return the error
                line.split(',') // Split the line separated by commas
                    .map(|entry| String::from(entry.trim())) // Remove leading and trailing whitespace
                    .collect() // Collect all strings in a row into a Vec<String>
            })
        })
        .collect() // Collect all lines into a Vec<Vec<String>>
}
```

`parse_csv_document` является универсальной функцией, что позволяет ей принимать любой тип, который реализует `BufRead`, такой как `BufRead<File>` или `[u8]`, не важно, какой это `R`, и `R` используется только для объявления типа `src`, по этому функция так же может быть записана как:

```rust
fn parse_csv_document(src: impl std::io::BufRead) -> std::io::Result<Vec<Vec<String>>> {
    src.lines()
        .map(|line| {
            // For each line in the source
            line.map(|line| {
                // If the line was read successfully, process it, if not, return the error
                line.split(',') // Split the line separated by commas
                    .map(|entry| String::from(entry.trim())) // Remove leading and trailing whitespace
                    .collect() // Collect all strings in a row into a Vec<String>
            })
        })
        .collect() // Collect all lines into a Vec<Vec<String>>
}
```

Важно, что `parse_csv_document::<std::io::Empty>(std::io::empty())` не будет работать со вторым примером.

##### Как возвращаемый тип

Если функция возвращает тип, который реализует `Trait`, можно записать тип возвращаемого значения как `impl MyTrait`, это может значительно упростить подписи типов.

```rust
use std::iter;
use std::vec::IntoIter;

// This function combines two `Vec<i32>` and returns an iterator over it.
// Look how complicated its return type is!
fn combine_vecs_explicit_return_type(v: Vec<i32>, u: Vec<i32>) -> iter::Cycle<iter::Chain<IntoIter<i32>, IntoIter<i32>>> {
    return v.into_iter().chain(u.into_iter()).cycle();
}

// This is the exact same function, but its return type uses `impl Trait`.
// Look how much simpler it is!
fn combine_vecs(v: Vec<i32>, u: Vec<i32>) -> impl Iterator<Item=i32> {
    v.into_iter().chain(u.into_iter()).cycle()
}

fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = vec![4, 5];
    let mut v3 = combine_vecs(v1, v2);
    assert_eq!(Some(1), v3.next());
    assert_eq!(Some(2), v3.next());
    assert_eq!(Some(3), v3.next());
    assert_eq!(Some(4), v3.next());
    assert_eq!(Some(5), v3.next());
    println!("all done");
}
```

Так же, некоторые типы не могут быть сохранены. Например у каждого замыкания есть свой безымянные конкретный тип. До появления `impl Trait` синтаксиса, нам приходилось выделять память в куче, чтобы возвращать замыкание, теперь можно сделать все статически:

```rust
// Returns a function that adds `y` to its input
fn make_adder_function(y: i32) -> impl Fn(i32) -> i32 {
    let closure = move |x: i32| { x + y };
    return closure;
}

fn main() {
    let plus_one = make_adder_function(1);
    println!("{}", plus_one(2)); // 3
}
```

#### Clone

При взаимодействии с ресурсами, обычным поведением будет передать их в вызовы. Однако иногда нам нужно также сделать копию ресурсов. Черта `Clone` позволяет это сделать. Для этого нам нужен метод `.clone()`

```rust
// A unit struct without resources
#[derive(Debug, Clone, Copy)]
struct Unit;

// A tuple struct with resources that implements the `Clone` trait
#[derive(Clone, Debug)]
struct Pair(Box<i32>, Box<i32>);

fn main() {
    // Instantiate `Unit`
    let unit = Unit;
    // Copy `Unit`, there are no resources to move
    let copied_unit = unit;

    // Both `Unit`s can be used independently
    println!("original: {:?}", unit);
    println!("copy: {:?}", copied_unit);

    // Instantiate `Pair`
    let pair = Pair(Box::new(1), Box::new(2));
    println!("original: {:?}", pair);

    // Move `pair` into `moved_pair`, moves resources
    let moved_pair = pair;
    println!("moved: {:?}", moved_pair);

    // Error! `pair` has lost its resources
    //println!("original: {:?}", pair);
    // TODO ^ Try uncommenting this line

    // Clone `moved_pair` into `cloned_pair` (resources are included)
    let cloned_pair = moved_pair.clone();
    // Drop the original pair using std::mem::drop
    drop(moved_pair);

    // Error! `moved_pair` has been dropped
    //println!("copy: {:?}", moved_pair);
    // TODO ^ Try uncommenting this line

    // The result from .clone() can still be used!
    println!("clone: {:?}", cloned_pair);
}
```

```
original: Unit
copy: Unit
original: Pair(1, 2)
moved: Pair(1, 2)
clone: Pair(1, 2)
```

#### Supertraits

В rust нет наследования, но можно определить черту как надмножество другой черты, например:

```rust
trait Person {
    fn name(&self) -> String;
}

// Person is a supertrait of Student.
// Implementing Student requires you to also impl Person.
trait Student: Person {
    fn university(&self) -> String;
}

trait Programmer {
    fn fav_language(&self) -> String;
}

// CompSciStudent (computer science student) is a subtrait of both Programmer 
// and Student. Implementing CompSciStudent requires you to impl both supertraits.
trait CompSciStudent: Programmer + Student {
    fn git_username(&self) -> String;
}

fn comp_sci_student_greeting(student: &dyn CompSciStudent) -> String {
    format!(
        "My name is {} and I attend {}. My favorite language is {}. My Git username is {}",
        student.name(),
        student.university(),
        student.fav_language(),
        student.git_username()
    )
}

fn main() {}
```

#### Disambiguating overlapping traits

Тип может реализовывать множество различных признаков. Что, если для двух признаков требуется одно и то же имя? Например, многие трейты могут иметь метод с именем `get()`. Они могут даже иметь разные типы возврата!

Поскольку каждая реализация трейта получает свой `impl` блок, становится ясно, какой `get` будет реализовываться.

```rust
trait UsernameWidget {
    // Get the selected username out of this widget
    fn get(&self) -> String;
}

trait AgeWidget {
    // Get the selected age out of this widget
    fn get(&self) -> u8;
}

// A form with both a UsernameWidget and an AgeWidget
struct Form {
    username: String,
    age: u8,
}

impl UsernameWidget for Form {
    fn get(&self) -> String {
        self.username.clone()
    }
}

impl AgeWidget for Form {
    fn get(&self) -> u8 {
        self.age
    }
}

fn main() {
    let form = Form {
        username: "rustacean".to_owned(),
        age: 28,
    };

    // If you uncomment this line, you'll get an error saying
    // "multiple `get` found". Because, after all, there are multiple methods
    // named `get`.
    // println!("{}", form.get());

    let username = <Form as UsernameWidget>::get(&form);
    assert_eq!("rustacean".to_owned(), username);
    let age = <Form as AgeWidget>::get(&form);
    assert_eq!(28, age);
}
```

---

## Associated functions

Связанными функциями называют те, которые определены в блоке `impl` для некоторой структуры. Бывают функции, которые не имеют параметра `self`, такие функции не являются методами(потому что не нужен экземпляр типа для работы, один из примеров - `String::from`, функция, определенная для `String` типа.)

Связанные функции, не являющиеся методами часто используются для конструкторов, возвращающих новый экземпляр структуры. Их часто называют `new`, но `new` не специальное имя и оно не встроенно в язык. Например мы могли бы представить связанную функцию с именем `square`, которая возвращала бы класс `Rectangle` с одинаковыми высотой и шириной, получая при этом только одно значение.

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```

Для использования этой функции используем `::` синтаксис: `let square = Rectangle::square(3);`. Синтаксис `::` используется помимо этого для пространства имен, созданных модулями.

Для одной структуры может быть использовано множество `impl` блоков.

# Example

Напишем программу, которая вычисляет площадь прямоугольника:

```rust
fn main() {
    let rect = (30, 50);
    println!("{}", area(rect));
}

fn area(dimensions: (u32, u32)) -> u32 {
    return dimensions.0 * dimensions.1;
}
```

Попробуем реализовать ее через структуры:

```rust
struct Rectangle {
    width: u32,
    height: u32,
}
fn main() {
    let rect = Rectangle{
        width: 30,
        height: 50,
    };
    println!("{}", area(&rect));
}

fn area(dimensions: &Rectangle) -> u32 {
    return dimensions.height * dimensions.width;
}
```

Так наш код стал более понятным. Теперь хочется, чтобы наша структура  отображалась при вызове в консоль. Первый способ - добавить атрибут `#[derive(Debug)]` перед определением структуры. Другой - использовать макрос `dbg!` который выполняет операцию и пишет в консоль операцию, результат и строку, где он используется.

Попробуем реализовать метод, проверяющий, может ли в наш прямоугольник вместиться другой прямоугольник:

```rust
impl Rectangle {
    fn area(&self) -> u32 {self.width * self.height}

    fn width(&self) -> bool {self.width > 0}

    fn can_hold(&self, other_rectangle: &Rectangle) -> bool {
        return self.width > other_rectangle.width && self.height > other_rectangle.height;
    }
}

fn main() {
    let rect1 = Rectangle{
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 1,
        height: 1,
    };

    println!("{:?} can hold {}? {}", rect1, rect2, rect1.can_hold(&rect2));
}
```

```
Rectangle { width: 30, height: 50 } can hold Rectangle { width: 1, height: 1 }? true
```

---