[Back](../../README.md)

---

# Scalar

Скалярные типы - тпы, которые имеют только одну величину.

---

# Integer

Целое число без дробной части. Бывает 2 типа: `i`(с знаком) и `u`(без знака)

| количество бит 	| Со знаком 	| Без знака 	|
|----------------	|-----------	|-----------	|
| 8              	| i8        	| u8        	|
| 16             	| i16       	| u16       	|
| 32             	| 132       	| u32       	|
| 64             	| i64       	| u64       	|
| 128            	| i128      	| u128      	|
| arch           	| isize     	| usize     	|

границы для знаковых значений вычисляются по формулам: от $-(2^{n-1})$ до $2^{n-1} - 1$
границы для незнаковых значений вычисляются по формуле от $0$ до $2^n-1$
Целочисленные литералы можно записывать в любой из форм представленных ниже:

| Числовые литералы | Пример      |
|----------------	|-------------|
| Десятичный	    | 98_222      |
| Шестнадцатеричный | 0xff        |
| Восьмеричный	    | 0o77        |
| Двоичный	        | 0b1111_0000 |
| Байт (только u8)  | b'A'        |

Основной случай, когда требуется использовать `isize` и `usize` - индексация какой-либо коллекции.

## Переполнение целых чисел

если для переменной `u8`, что включает от 0 до 255 попытаться вписать 256, то возникнет паника при компиляции в режиме отладки `cargo build`. Но если запустить сборку с `--release`, то ошибка не появится. В этом случае, число, больше допустимого станет остатком на деление от допустимого. Так, число 256, введенное в `u8` станет 1.
Если требуется достичь переполнения в программе, то стоит использовать следующие методы:

- `wrapping_*`, например `wrapping_add`
- Возвращение другой переменное, если `checked_*` сработал.
- Возвращение числа и логического значения, с помощью `overflowing_*`.
- Установить минимальное или максимальное значение при использовании методов `saturating_`.

## Определение знака переменной `i`

Для того, чтобы понять, какой знак имеет переменная, языку нужно посмотреть на последний(левый) бит числа, если `1`, то число будет отрицательным, иначе положительным

---

# Float

Числа с плавающей запятой могут быть только `f32` и `f64`. `f32` работает быстрее ненамного, а `f64` будет более точным.
Числа с плавающей точкой представлены согласно стандарту [IEEE-754](https://ru.wikipedia.org/wiki/IEEE_754-2008). Тип `f32` является числом с плавающей точкой одинарной точности, а `f64` имеет двойную точность.
![сравнение точности f32 и f64](https://upload.wikimedia.org/wikipedia/commons/thumb/1/18/IEEE754.svg/446px-IEEE754.svg.png)


## Числовые операции

Rust поддерживает основные математические операции, которые ожидаются для всех типов чисел: сложение(`+`), вычитание(`-`), умножение(`*`), деление(`/`) и остаток(`%`). Целочисленное деление округляется до ближайшего целого числа.

---

# Bool

В rust логический тип данных занимает 1 байт, обозначается как `bool` и принимает значения: `true` и `false`.

---

## Char

Примитивный алфавитный тип языка. Может принимать любой один символ. Символ оборачивается одинарными кавычками. занимает 4 байта. Обозначается `char`. Скалярные значения Unicode находятся в диапазоне от `U+0000` до `U+D7FF` и от `U+E000` до `U+10FFFF` включительно.

---