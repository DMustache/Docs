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