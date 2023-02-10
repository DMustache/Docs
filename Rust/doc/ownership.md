[Back](../README.md)

# Ownership

Владение - набор правил, определяющих, как программа Rust управляет памятью. Все программы так или иначе должны испольщовать память компьютера во время работы. В Rust программа проходит через систему владения с _набором правил_. Если на этой стадии программа нарушит хотя бы одно из правил, то программа не будет скомпилирована.
Понимание этой особенности даст больше понимание работы rust.

---

# Stack

Стек является областью в памяти компьютера, где хранятся данные известной длины. Работает стек по принципу: "Кто последним пришел, тот первым вышел".

---

# Heap

Более медленный тип памяти называется куча. Здесь данные разбросаны в любом порядке. Когда мы обращаемся к `&str`, которая не имеет известной длины, мы в куче ищем то место, где данные это переменной могут поместиться и резервируем для этой переменной. Работает медленнее, так как не надо просто положить данные, а найти место, где данные могут уместиться и резервируем эту область.

---

Особенности `String` заключаются в том, что этот тип является неким вектором, хранящим индексы к куче. `String` хранит индексы в стеке, а индексы показывают, где элементы находится в куче.

Если передать одной строке другую, то первую строку использовать не удастся, так строка скопирует индекс к памяти в куче, выйти из области видимости этой переменной, но строка, которая получила строку будет _живой_, при удалении первой переменной, должно очиститься место в куче. По этому Rust как бы заглушает переменную, которая была родителем.

```rust
fn main() {
    let s1 = String::from("hello"); // ее видно
    let s2 = s1; // s1 теперь не видно

    println!("{}, world!", s1);
}

```

---

# Scope

Правила определения области действия.

Эти правила указывают компилятору, когда заимствование допустимо, когда ресурсы могут быть освобождены и когда создаются или уничтожаются переменные

## RAII

(Resource Acquisition Is Initialization - Получение ресурса есть инициализация)
Переменные помимо хранения данных владеют ресурсами, rust применяет это правило каждый раз, как выходит из области видимости, вызывая деструктор на ту область. Это предотвращает утечку ресурсов, беспокоиться об этом не стоит.

## Ownership and move

Владение и передача. Поскольку переменные отвечают за освобождение своих собственных ресурсов, у ресурсов может быть только один владелец. При этом не все переменные владеют ресурсами. Например ссылки, так как простые ссылки не могут менять данные своей переменной. Так как rust предотвращает образование висячих ссылок, то например такие методы, как функции получают данные используя перемещение, так, одни данные хранятся только в одном месте.

Следующий пример показывает изменчивость данных при передаче права владения.

```rust
fn main() {
    let immutable_box = Box::new(5u32);

    println!("immutable_box contains {}", immutable_box);

    // Mutability error
    //*immutable_box = 4;

    // *Move* the box, changing the ownership (and mutability)
    let mut mutable_box = immutable_box;

    println!("mutable_box contains {}", mutable_box);

    // Modify the contents of the box
    *mutable_box = 4;

    println!("mutable_box now contains {}", mutable_box);
}
```

В этим примере есть интересная строчка `*mutable_box`, которая неявно приводит к `*const mutable_box`. `&` имеет идентичное представление с необработанным указателем `*`. Идти от `*T` к `&T` небезопасно, так как необработанный указатель должен указывать на действительный экземпляр `T`, во то время как ссылка только дает чтение на переменной, если сделать ссылку изменяемой, то ей не удастся работать с неизменяемой переменной. Вот безопасный способ имплементации:

```rust
// Explicit cast:
let i: u32 = 1;
let p_imm: *const u32 = &i as *const u32;

// Implicit coercion:
let mut m: u32 = 2;
let p_mut: *mut u32 = &mut m;

unsafe {
    let ref_imm: &u32 = &*p_imm;
    let ref_mut: &mut u32 = &mut *p_mut;
}
```

### Partial moves

При деструктурировании переменной могут использоваться как метод передачи, так и метод ссылки. Это приведет к частичному перемещению переменной. Некоторые части переменной переместятся, а некоторые останутся. Из-за этого родительская переменная не может использоваться целиком, Однако части, которые ссылаются, все еще могут использоваться.

```rust
fn main() {
    #[derive(Debug)]
    struct Person {
        name: String,
        age: Box<u8>,
    }

    let person = Person {
        name: String::from("Alice"),
        age: Box::new(20),
    };

    // `name` is moved out of person, but `age` is referenced
    let Person { name, ref age } = person;

    println!("The person's age is {}", age);

    println!("The person's name is {}", name);

    // Error! borrow of partially moved value: `person` partial move occurs
    //println!("The person struct is {:?}", person);

    // `person` cannot be used but `person.age` can be used as it is not moved
    println!("The person's age from person struct is {}", person.age);
}
```

Удаление `ref` в примере приведет к ошибке. Если бы возраст персоны хранился бы в стеке, `ref` бы не требовалось, поскольку `age` будет копировать данные возраста без их перемещения.

### Разница между `&` и `ref`

- `&` означает, что код ожидает ссылку на объект. Следовательно `&`, это часть указанного кода.
- `ref` указывает, что нам нужна ссылка на заимствованное значение.

[source](https://doc.rust-lang.org/std/keyword.ref.html)

- `&` указывает на то, какое значение мы ожидаем при сопоставлении. Программа, видя `&` понимает, что мы хотим сопоставить ссылки на некии объекты, а не сами объекты.
- `ref`, в отличии от `&`, не обозначает нечто, с чем мы производим сопоставление. Оно никоим образом не влияет на то, будет ли значение сопоставлено с данным паттерном или нет. Единственное, что оно отличит, это то, как части сопоставленного значения будут захвачены пересмотренными паттернами.

`ref` обладает методом заимствования/владения, в то время как `&` является перемещением.

Резюмируем это на примере:

```rust
for &(ref name, ref value) in &query_params {
    println!("{}={}", name, value);
}
```

Так как мы знаем, что `ref` не влияет на то, сопоставится ли паттерн или нет, мы могли бы просто сделать замену на `&(name, value)`. Да, такой вариант будет более читабельным, но не для компилятора. Проблема в том, что без `ref` мы пытаемся переместить значения кортежа внутрь цикла, но так как мы итерирует `&querry_params`, мы лишь заимствуем каждый из кортежей, по этому такое перемещение на самом деле невозможно. По сути, это было бы классической попыткой перемещения значения из контекста заимствования. Но так как нам нужно только печатать значения, то мы получаем доступ к значениям по ссылке и нам этого достаточно. Это `ref` нам и дает. Вернув обратно данное ключевое слово, мы вернемся обратно от перемещения к заимствованию.

- `&` обозначает, что паттерн **ожидает ссылку на объект**. Таким образом `&` является частью паттерна. Отсюда `&Foo` будет сопоставляться иначе, чем `Foo`
- `ref` отмечает, что вы хотите получить ссылку на раскованное значение. Оно не участвует непосредственно в сопоставлении паттерна. Поэтому `Foo(ref foo)`
 будет сопоставляться тому же объекту, что и `Foo(foo)`

[source](https://habr.com/ru/post/306582/)

## Borrowing

Иногда мы не хотим получить данные без становления их владельцем. Для этого rust использует механизм заимствования. Вместо передачи объектов по значению `T`, объекты можно передать по ссылке `&T`. Компилятор гарантирует, что ссылки всегда указывают на допустимые объекты, по этому мы можем не беспокоиться о висячих ссылках, а так же мы уверены, что пока ссылки на объект существуют, объект не может быть уничтожен.

- При использовании переменной, данные которой хранятся в куче, требуется использовать клонирование, если необходимо использовать и родителя, и дочернюю переменную в одном выражении.
- При приравнивании переменной, компилятором по умолчанию будет использоваться копирование, если это безопасно, иначе будет использоваться заимствование.



### Mutability
(Изменчивость)

Изменяемые данные могут быть заимствованны с правом изменения через `&mut T`. Это называется изменяемой ссылкой и предоставляют право на чтение и изменение.


Нельзя использовать две изменяемые ссылки на одни и те же данные в один момент. Это предотвращает _гонку данных_.
Гонка данных возникает в следующих трех случаях:

- два или больше указателей используют одни и те же данные в одно и то же время.
- Минимум один указатель используется для записи данных
- Отсутствуют механизмы для синхронизации доступа к данным.

Но можно использовать одновременно больше одного указателя в том случае, когда они работают в разных областях видимости.
Комбинировать изменяемые и неизменяемые ссылки.

```rust
fn main() {
    let mut s = String::from("hello");

    let r3 = &mut s;
    println!("{}", r3);
}
```

Так делать можно.

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
}
```

А так нельзя. Если создать ссылочную неизменяемую переменную и ссылочную изменяемую переменную одновременно, то возникнет ошибка. Но если один тип ссылок использовать, а потом использовать другой тип ссылок, то ошибки не будет

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
}
```

**Два правила работы с ссылками:**

- В один момент времени может существовать сколько угодно неизменяемых ссылок либо одна изменяемая ссылка.
- Все ссылки должны быть действительными.

---

## Lifetimes

Время жизни - конструкция, которую использует средство проверки заимствования в компиляторе rust. Время жизни переменной начинается во время ее объявления и заканчивается во время ее уничтожения.

Например, мы заимствует переменную через `&`, затем начинается ее срок жизни, который определится тем, где она объявлена. Затем идет срок жизни, который определяется тем, где он объявлен. В результате заем действителен до тех пор, пока родитель не будет уничтожен. Однако границы заимствования определяются тем, где используется ссылка.

**Важно** Никакие имена или типы не контролируют время жизни. Это ограничивает возможности использования `lifetimes`.

### Explicit annotation

Средство проверки заимствования использует явные аннотации времени жизни, чтобы определить, как долго должны ссылки жить. В тех случаях, когда временем жизни переменной можно пренебречь является [elision](https://doc.rust-lang.org/rust-by-example/scope/lifetime/elision.html)(неявно аннотирует время жизни и поэтому отличается), Rust требует явных аннотаций, чтобы определить рамки жизни ссылки. Вот пример объявления явной аннотации:

```rust
foo<`a> // 'foo' has lifetime parameter '`a'
```

Для большего количества параметров мы в стрелочных скобках прописываем параметры через запятую. Синтаксис подобен [замыканию](https://doc.rust-lang.org/rust-by-example/fn/closures/anonymity.html), использование времени жизни требует дженериков. Кроме того, синтаксис времени жизни указывает, что время жизни связанной функции не может превышать `'a`. Явная аннотация имеет форму `&'a T`, где `'a` уже была ведена. В случае, если меток времени больше 1, используется логические или.

### Functions

Игнорируя [elision](https://doc.rust-lang.org/rust-by-example/scope/lifetime/elision.html), сигнатуры функций с временем жизни имеют несколько ограничений:

- Любая ссылка должна иметь аннотированное время жизни.
- Любая возвращаемая ссылка должна иметь то же время жизни, что и ее родитель, или быть статической.

В дополнение к этим правилам, возврат ссылок без родителя запрещен, если это приведет к возврату ссылок на недопустимые данные, что создаст _висячую ссылку_. Вот пример нескольких допустимых форм функций, работающих с `lifetime`.

```rust
// One input reference with lifetime `'a` which must live
// at least as long as the function.
fn print_one<'a>(x: &'a i32) {
    println!("`print_one`: x is {}", x);
}

// Mutable references are possible with lifetimes as well.
fn add_one<'a>(x: &'a mut i32) {
    *x += 1;
}

// Multiple elements with different lifetimes. In this case, it
// would be fine for both to have the same lifetime `'a`, but
// in more complex cases, different lifetimes may be required.
fn print_multi<'a, 'b>(x: &'a i32, y: &'b i32) {
    println!("`print_multi`: x is {}, y is {}", x, y);
}

// Returning references that have been passed in is acceptable.
// However, the correct lifetime must be returned.
fn pass_x<'a, 'b>(x: &'a i32, _: &'b i32) -> &'a i32 { x }

//fn invalid_output<'a>() -> &'a String { &String::from("foo") }
// The above is invalid: `'a` must live longer than the function.
// Here, `&String::from("foo")` would create a `String`, followed by a
// reference. Then the data is dropped upon exiting the scope, leaving
// a reference to invalid data to be returned.

fn main() {
    let x = 7;
    let y = 9;

    print_one(&x);
    print_multi(&x, &y);

    let z = pass_x(&x, &y);
    print_one(z);

    let mut t = 3;
    add_one(&mut t);
    print_one(&t);
}
```

### Methods/Structs

Аннотации методов/структур аналогичны аннотациям функций:

Методы:

```rust
struct Owner(i32);

impl Owner {
    // Annotate lifetimes as in a standalone function.
    fn add_one<'a>(&'a mut self) { self.0 += 1; }
    fn print<'a>(&'a self) {
        println!("`print`: {}", self.0);
    }
}

fn main() {
    let mut owner = Owner(18);

    owner.add_one();
    owner.print();
}
```

Структуры:

```rust
// A type `Borrowed` which houses a reference to an
// `i32`. The reference to `i32` must outlive `Borrowed`.
#[derive(Debug)]
struct Borrowed<'a>(&'a i32);

// Similarly, both references here must outlive this structure.
#[derive(Debug)]
struct NamedBorrowed<'a> {
    x: &'a i32,
    y: &'a i32,
}

// An enum which is either an `i32` or a reference to one.
#[derive(Debug)]
enum Either<'a> {
    Num(i32),
    Ref(&'a i32),
}

fn main() {
    let x = 18;
    let y = 15;

    let single = Borrowed(&x);
    let double = NamedBorrowed { x: &x, y: &y };
    let reference = Either::Ref(&x);
    let number    = Either::Num(y);

    println!("x is borrowed in {:?}", single);
    println!("x and y are borrowed in {:?}", double);
    println!("x is borrowed in {:?}", reference);
    println!("y is *not* borrowed in {:?}", number);
}
```

### Traits

Аннотации трейтов в основном аналогичны функцией за исключением того, что `impl` также может иметь аннотацию времени функции

```Rust
// A struct with annotation of lifetimes.
#[derive(Debug)]
struct Borrowed<'a> {
    x: &'a i32,
}

// Annotate lifetimes to impl.
impl<'a> Default for Borrowed<'a> {
    fn default() -> Self {
        Self {
            x: &10,
        }
    }
}

fn main() {
    let b: Borrowed = Default::default();
    println!("b is {:?}", b);
}
```

### Bounds

Точно так же, как универсальные типы могут быть ограниченны, время жизни так же используют границы, хотя они и универсальные. Символ `:` здесь имеет немного другое значение, `+` аналогично. Пример чтения следующего:

- `T: 'a` - все ссылки в `T` должны существовать больше `'a`.
- `T: Trait + 'a` - Тип `T` должен реализовывать трейт `Trait` и все ссылки в `T` должны существовать больше `'a`.

В примере показан синтаксис в действии, используемый после ключевого слова `where`

```rust
use std::fmt::Debug; // Trait to bound with.

#[derive(Debug)]
struct Ref<'a, T: 'a>(&'a T);
// `Ref` contains a reference to a generic type `T` that has
// an unknown lifetime `'a`. `T` is bounded such that any
// *references* in `T` must outlive `'a`. Additionally, the lifetime
// of `Ref` may not exceed `'a`.

// A generic function which prints using the `Debug` trait.
fn print<T>(t: T) where
    T: Debug {
    println!("`print`: t is {:?}", t);
}

// Here a reference to `T` is taken where `T` implements
// `Debug` and all *references* in `T` outlive `'a`. In
// addition, `'a` must outlive the function.
fn print_ref<'a, T>(t: &'a T) where
    T: Debug + 'a {
    println!("`print_ref`: t is {:?}", t);
}

fn main() {
    let x = 7;
    let ref_x = Ref(&x);

    print_ref(&ref_x);
    print(ref_x);
}
```

### Coercion

Принудительно, более длинное время жизни может быть заменено более коротким, чтобы оно работало в той области, в которой обычно не работало. Это работает благодаря правилу принуждения в компиляторе rust, а так же с помощью объявления разницы времени жизни.

```rust
// Here, Rust infers a lifetime that is as short as possible.
// The two references are then coerced to that lifetime.
fn multiply<'a>(first: &'a i32, second: &'a i32) -> i32 {
    first * second
}

// `<'a: 'b, 'b>` reads as lifetime `'a` is at least as long as `'b`.
// Here, we take in an `&'a i32` and return a `&'b i32` as a result of coercion.
fn choose_first<'a: 'b, 'b>(first: &'a i32, _: &'b i32) -> &'b i32 {
    first
}

fn main() {
    let first = 2; // Longer lifetime
    {
        let second = 3; // Shorter lifetime

        println!("The product is {}", multiply(&first, &second));
        println!("{} is the first", choose_first(&first, &second));
    };
}
```

### Static

Rust имеет несколько зарезервированных под время жизни времен. Одно из них `'static`, его используют в двух случаях:

```rust
// A reference with 'static lifetime:
let s: &'static str = "hello world";

// 'static as part of a trait bound:
fn generic<T>(x: T) where T: 'static {}
```

Эти примеры похожи, но не полностью, по этому периодически возникает путаница между ними.

Время жизни статической ссылки делает данные, доступные по этой ссылке будут доступны всегда во время работы программы, если не принудить эту переменную к более короткому времени жизни.

#### Reference lifetime

Есть два способа объявить переменную с `'static` временем жизни, она будет храниться в бинарнике и доступна только для чтения.

- Объявить переменную константой `static`: `static NUM: i32 = 18;`
- Сделать `string` переменную, которая имеет тип `&'static str`

```rust
// Make a constant with `'static` lifetime.
static NUM: i32 = 18;

// Returns a reference to `NUM` where its `'static`
// lifetime is coerced to that of the input argument.
fn coerce_static<'a>(_: &'a i32) -> &'a i32 {
    &NUM
}

fn main() {
    {
        // Make a `string` literal and print it:
        let static_string = "I'm in read-only memory";
        println!("static_string: {}", static_string);

        // When `static_string` goes out of scope, the reference
        // can no longer be used, but the data remains in the binary.
    }

    {
        // Make an integer to use for `coerce_static`:
        let lifetime_num = 9;

        // Coerce `NUM` to lifetime of `lifetime_num`:
        let coerced_static = coerce_static(&lifetime_num);

        println!("coerced_static: {}", coerced_static);
    }

    println!("NUM: {} stays accessible!", NUM);
}
```

#### Trait bound

Границы трейта означают, что тип не поддерживает какие либо не статические ссылки. К примеру, получатель может хранить тип сколько угодно и никогда не будет вызывать ошибку отсутствия данных на нем, пока его не деструктурировать.

Это важно для понимания, так как это означает, что любая полученная информация всегда проходит `'static` границу времени жизни, но ссылки нет.

```rust
use std::fmt::Debug;

fn print_it( input: impl Debug + 'static ) {
    println!( "'static value passed in is: {:?}", input );
}

fn main() {
    // i is owned and contains no references, thus it's 'static:
    let i = 5;
    print_it(i);

    // oops, &i only has the lifetime defined by the scope of
    // main(), so it's not 'static:
    print_it(&i);
}
```
```rust
error[E0597]: `i` does not live long enough
  --> src/lib.rs:15:15
   |
15 |     print_it(&i);
   |     ---------^^--
   |     |         |
   |     |         borrowed value does not live long enough
   |     argument requires that `i` is borrowed for `'static`
16 | }
   | - `i` dropped here while still borrowed
```

---

# Allocation

(Выделение памяти)

Использование `NonNull` ломает функцию в `Vec`, а так же остальные стандартные коллекции. При создании пустого вектора память не выделяется. Это аналогично созданию пустого блока памяти - приводит к неопределенному поведению. Так, при создании пустого вектора `Vec::new` мы кладем туда мусор, чтобы избежать неопределенного поведения.

В случае, если мы можем получить в переменной, связано с длинами [составных](datatypes/compound.md) переменных `0` или, еще хуже `null`, мы можем воспользоваться `NonNull::dangling()`[doc.rust-lang.org](https://doc.rust-lang.org/std/ptr/struct.NonNull.html#method.dangling), что позволит избежать проблем с пустой переменной.

В основе управления памятью лежит идея владения ссылкой на объект и [заимствования](#borrowing), Если каждый объектом владеет только одна переменная, то как только кончается срок ее жизни в конце блока, все на что она указывала модно рекурсивно очистить. Также ссылки можно одалживать для чтения и записи. Здесь работает принцип - [один писатель, много читателей](#borrowing).

---

# Move/Clone/Copy

## Move

Функции перемещают возвращаемую переменную, если она на это рассчитана. Нет смысла копировать переменную, так как функция при возврате перестает существовать до вызова ее заново.


## Clone

Если требуется скопировать, например `String`, которая заглушает переменные родители, можно использовать метод полного копирования с копированием данных в куче.

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
}
```

## Copy

В оличии от [Clone](#clone), этот метод применяется к тем типам, которые не требуют клонирования. К таким типам относятся:

- Все целочисленные типы: [Integer](datatypes/scalar.md#integer), [Float](datatypes/scalar.md#float).
- Логический тип [Bool](datatypes/scalar.md#bool)
- Символьный тип [Char](datatypes/scalar.md#char)
- Кортежы, но только те, которые хранят только типы, находящиеся в этом списке [Tuples](datatypes/compound.md#tuples)

При использовании функций, происходит копирование переменных, если функция принимает переменные.

---

# References

Ссылки позволяют избавиться от копирования/клонирования данных при передачи в функцию. Для того чтобы передать ссылку, используется префикс `&`, чтобы ссылка была изменяемой, то необходимо добавить `mut` в месте получения данных.
Пример использования ссылок, чтобы избежать копирования/клонирования:

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len); // The length of 'hello' is 5.
}

fn calculate_length(s: &mut String) -> usize {
    s.len()
}
```

![Как работает данный пример](https://doc.rust-lang.ru/book/img/trpl04-05.svg)

Нельзя использовать две изменяемые ссылки на одни и те же данные в один момент. Это предотвращает _гонку данных_.
Гонка данных возникает в следующих трех случаях:

- два или больше указателей используют одни и те же данные в одно и то же время.
- Минимум один указатель используется для записи данных
- Отсутствую механизмы для синхронизации доступа к данным.

Но можно использовать одновременно больше одного указателя в том случае, когда они работают в разных областях видимости.
Комбинировать изменяемые и неизменяемые ссылки.

```rust
fn main() {
    let mut s = String::from("hello");

    let r3 = &mut s;
    println!("{}", r3);
}
```

Так делать можно.

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
}
```

А так нельзя. Если создать ссылочную неизменяемую переменную и ссылочную изменяемую переменную одновременно, то возникнет ошибка. Но если один тип ссылок использовать, а потом использовать другой тип ссылок, то ошибки не будет

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
}
```

**Два правила работы с ссылками:**

- В один момент времени может существовать сколько угодно неизменяемых ссылок либо одна изменяемая ссылка.
- Все ссылки должны быть действительными.

## &

Заимствует значение при `&expr`, `&mut expr`.
Представляет заимствованный тип указателя при `&type`, `&mut type`

## *

Разыменовывает значение при `*expr`.
Представляет необработанный указатель при `*const type`, `*mut type`
```rust
let a: &i32 = &5;
let mut b:i32 = *a;
b = b + 1;
println!("{} {}", a, b);~
```

## ref

Указывает, что мы хотим хранить в этой переменной ссылку на значение

## Difference

Заметим, что у `&` и `ref` различны только способы написания:

```rust
let y = 'y';

// `ref` on the left side of an assignment is like adding `&` on the right side
let ref x1 = y;
let x2 = &y;

println!("{}", x1 == x2);
```

Теперь определим разницу при использовании `&` и `*`. В то время как `&` просто ссылается на другую переменную, `*` ссылается и распаковывает переменную на один уровень.


## Dangling pointers/Висячие ссылки

В языках с указателями легко ошибочно создать висячий указатель, он ссылается на место в памяти, которое могло быть передано кому-то другому после освобождения этой части памяти, сохраняя при этом указатель из нее. Rust защищает от висячих ссылок.
Пример висячей ссылки

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

---

# Slices

Срез является ссылкой часть памяти. Срез позволяет использовать данные, которые хранятся как в стеке, так и в куче. Размер среза вычисляется во время выполнения. Срез получает данные по ссылке, использует метод [заимствования](#borrowing)

```rust
fn main() {
    let n1 = String::from("value");
    //---------------------01234
    println!("{}", n1.len()); // 5
    let c1 = &n1[2..4];

    println!("{}", c1); // lu
}
```

## Mutable slices

(Изменяемые срезы) чтобы срез был изменяемым, необходимо добавить префикс `&mut` в выражение:

```rust
fn main() {
    let mut data = [1,2,3,4,5];

    println!("main {:?}", data); // [1, 2, 3, 4, 5]
    use_slice(&mut data);
    println!("main {:?}", data); // [1200, 2, 3, 4, 5]
}

fn use_slice(slice: &mut [i32]) {
    println!("{}", slice.len()); // 5
    println!("{:?}", &slice[2..]); // [3, 4, 5]
    slice[0] = 1200;
}
```

Срезы похожи на массивы, но их размер неизвестен в момент компиляции. По сути, срез является списком ссылок на элементы массива.

---