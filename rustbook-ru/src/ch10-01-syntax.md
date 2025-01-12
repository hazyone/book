## Обобщённые типы данных

Мы используем обобщённые типы данных для объявления функций или структур, которые затем можно использовать с различными конкретными типами данных. Давайте сначала посмотрим, как объявлять функции, структуры, перечисления и методы, используя обобщённые типы данных. Затем мы обсудим, как обобщённые типы данных влияют на производительность кода.

### В объявлении функций

Когда мы объявляем функцию с обобщёнными типами, мы размещаем обобщённые типы в сигнатуре функции, где мы обычно указываем типы данных аргументов и возвращаемого значения. Используя обобщённые типы, мы делаем код более гибким и предоставляем большую функциональность при вызове нашей функции, предотвращая дублирование кода.

Рассмотрим пример с функцией `largest`. Листинг 10-4 показывает две функции, каждая из которых находит самое большое значение в срезе своего типа. Позже мы объединим их в одну функцию, использующую обобщённые типы данных.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

<span class="caption">Листинг 10-4: две функции, отличающиеся только именем и типом обрабатываемых данных</span>

Функция `largest_i32` уже встречалась нам: мы извлекли её в листинге 10-3, когда боролись с дублированием кода — она находит наибольшее значение типа `i32` в срезе. Функция `largest_char` находит самое большое значение типа `char` в срезе. Тело у этих функций одинаковое, поэтому давайте избавимся от дублируемого кода, используя параметр обобщённого типа в одной функции.

Для параметризации типов данных в новой объявляемой функции нам нужно дать имя обобщённому типу — так же, как мы это делаем для аргументов функций. Можно использовать любой идентификатор для имени параметра типа, но мы будем использовать `T`, потому что по соглашению имена параметров в Rust должны быть короткими (обычно длиной в один символ), а именование типов в Rust делается в нотации CamelCase. Сокращение слова «type» до одной буквы `T` является стандартным выбором большинства программистов, использующих язык Rust.

Когда мы используем параметр в теле функции, мы должны объявить имя параметра в сигнатуре, чтобы компилятор знал, что означает это имя. Аналогично когда мы используем имя типа параметра в сигнатуре функции, мы должны объявить это имя раньше, чем мы его используем. Чтобы определить обобщённую функцию `largest`, поместим объявление имён параметров в треугольные скобки `<>` между именем функции и списком параметров, как здесь:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Объявление читается так: функция `largest` является обобщённой по типу `T`. Эта функция имеет один параметр с именем `list`, который является срезом значений с типом данных `T`. Функция `largest` возвращает значение этого же типа `T`.

Листинг 10-5 показывает определение функции `largest` с использованием обобщённых типов данных в её сигнатуре. Листинг также показывает, как мы можем вызвать функцию со срезом данных типа `i32` или `char`. Данный код пока не будет компилироваться, но мы исправим это к концу раздела.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

<span class="caption">Листинг 10-5: функция <code>largest</code>, использующая параметры обобщённого типа; пока ещё не компилируется</span>

Если мы скомпилируем программу сейчас, мы получим следующую ошибку:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

В подсказке упоминается `std::cmp::PartialOrd`, который является *типажом*. Мы поговорим про типажи в следующем разделе. Сейчас ошибка в функции `largest` указывает, что функция не будет работать для всех возможных типов `T`. Так как мы хотим сравнивать значения типа `T` в теле функции, мы можем использовать только те типы, данные которых можно упорядочить: можем упорядочить — значит, можем и сравнить. Чтобы можно было задействовать сравнения, стандартная библиотека имеет типаж `std::cmp::PartialOrd`, который вы можете реализовать для типов (смотрите дополнение С для большей информации про данный типаж). Следуя совету в сообщении компилятора, ограничим тип `T` теми вариантами, которые поддерживают типаж `PartialOrd`, и тогда пример успешно  скомпилируется, так как стандартная библиотека реализует `PartialOrd` как для типа `i32`, так и для типа `char`.

### В определении структур

Мы также можем определить структуры, использующие обобщённые типы в одном или нескольких своих полях, с помощью синтаксиса `<>`. Листинг 10-6 показывает, как определить структуру `Point<T>`, чтобы хранить поля координат `x` и `y` любого типа данных.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

<span class="caption">Листинг 10-6: структура <code>Point</code>, содержащая поля <code>x</code> и <code>y</code> типа <code>T</code></span>

Синтаксис использования обобщённых типов в определении структуры очень похож на синтаксис в определении функции. Сначала мы объявляем имена типов параметров внутри треугольных скобок сразу после названия структуры. Затем мы можем использовать обобщённые типы в определении структуры в тех местах, где ранее мы указывали бы конкретные типы.

Так как мы используем только один обобщённый тип данных для определения структуры `Point<T>`, это определение означает, что структура `Point<T>` является обобщённой с типом `T`, и <em>оба</em> поля `x` и <code>y</code> имеют одинаковый тип, каким бы он не являлся. Если мы создадим экземпляр структуры `Point<T>` со значениями разных типов, как показано в листинге 10-7, наш код не скомпилируется.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

<span class="caption">Листинг 10-7: поля <code>x</code> и <code>y</code> должны быть одного типа, так как они имеют один и тот же обобщённый тип <code>T</code></span>

В этом примере, когда мы присваиваем целочисленное значение 5 переменной `x` , мы сообщаем компилятору, что обобщённый тип `T` будет целым числом для этого экземпляра `Point<T>`. Затем, когда мы указываем значение 4.0 (имеющее тип, отличный от целого числа) для `y`, который по нашему определению должен иметь тот же тип, что и `x`, мы получим ошибку несоответствия типов:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Чтобы определить структуру `Point`, где оба значения `x` и `y` являются обобщёнными, но различными типами, можно использовать несколько параметров обобщённого типа. Например, в листинге 10-8 мы изменим определение `Point` таким образом, чтобы оно использовало обобщённые типы `T` и `U`, где `x` имеет тип `T` а `y` имеет тип `U`.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

<span class="caption">Листинг 10-8: структура <code>Point&lt;T, U&gt;</code> обобщена для двух типов, так что <code>x</code> и <code>y</code> могут быть значениями разных типов</span>

Теперь разрешены все показанные экземпляры типа `Point`! В объявлении можно использовать сколь угодно много параметров обобщённого типа, но если делать это в большом количестве, код будет тяжело читать. Если в вашем коде требуется много обобщённых типов, возможно, стоит разбить его на более мелкие части.

### В определениях перечислений

Как и структуры, перечисления также могут хранить обобщённые типы в своих вариантах. Давайте ещё раз посмотрим на перечисление `Option<T>`, предоставленное стандартной библиотекой, которое мы использовали в главе 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Это определение теперь должно быть вам более понятно. Как видите,  перечисление `Option<T>` является обобщённым по типу `T` и имеет два варианта: вариант `Some`, который содержит одно значение типа `T`, и вариант `None`, который не содержит никакого значения. Используя перечисление `Option<T>`, можно выразить абстрактную концепцию необязательного значения — и так как `Option<T>` является обобщённым, можно использовать эту абстракцию независимо от того, каким будет тип необязательного значения.

Перечисления также могут использовать несколько обобщённых типов. Определение перечисления `Result`, которое мы упоминали в главе 9, является примером такого использования:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Перечисление `Result` имеет два обобщённых типа: `T` и `E` — и два варианта:  `Ok`, который содержит тип `T`, и `Err`, содержащий тип `E`. С таким определением удобно использовать перечисление `Result` везде, где операции могут быть выполнены успешно (возвращая значение типа `T`) или неуспешно (возвращая ошибку типа `E`). Это то, что мы делали при открытии файла в листинге 9-3, где `T` заполнялось типом `std::fs::File`, если файл был открыт успешно, либо `E` заполнялось типом  `std::io::Error`, если при открытии файла возникали какие-либо проблемы.

Если вы встречаете в коде ситуации, когда несколько определений структур или перечислений отличаются только типами содержащихся в них значений, вы можете устранить дублирование, используя обобщённые типы.

### В определении методов

Мы можем реализовать методы для структур и перечислений (как мы делали в главе 5) и в определениях этих методов также использовать обобщённые типы. В листинге 10-9 показана структура `Point<T>`, которую мы определили в листинге 10-6, с добавленным для неё методом `x`.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

<span class="caption">Листинг 10-9: Реализация метода с именем <code>x</code> у структуры <code>Point&lt;T&gt;</code>, которая будет возвращать ссылку на поле <code>x</code> типа <code>T</code></span>

Здесь мы определили метод с именем `x` у структуры `Point<T>`, который возвращает ссылку на данные в поле `x`.

Обратите внимание, что мы должны объявить `T` сразу после `impl` .  В этом случае мы можем использовать `T` для указания на то, что реализуем метод для типа `Point<T>`. Объявив `T` универсальным типом сразу после `impl` , Rust может определить, что тип в угловых скобках в `Point` является универсальным, а не конкретным типом. Мы могли бы выбрать другое имя для этого обобщённого параметра, отличное от имени, использованного в определении структуры, но обычно используют одно и то же имя. Методы, написанные внутри раздела `impl` , который использует обобщённый тип, будут определены для любого экземпляра типа, независимо от того, какой конкретный тип в конечном итоге будет подставлен вместо этого обобщённого.

Мы можем также указать ограничения, какие обобщённые типы разрешено использовать при определении методов. Например, мы могли бы реализовать методы только для экземпляров типа `Point<f32>`, а не для экземпляров `Point<T>`, в которых используется произвольный обобщённый тип. В листинге 10-10 мы используем конкретный тип `f32`, что означает, что мы не определяем никакие типы после `impl`.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

<span class="caption">Листинг 10-10: блок <code>impl</code>, который применяется только к структуре, имеющей конкретный тип для параметра обобщённого типа <code>T</code></span>

Этот код означает, что тип `Point<f32>` будет иметь метод с именем `distance_from_origin`, а другие экземпляры `Point<T>`, где `T` имеет тип, отличный от `f32`, не будут иметь этого метода. Метод вычисляет, насколько далеко наша точка находится от точки с координатами (0.0, 0.0), и использует математические операции, доступные только для типов с плавающей точкой.

Параметры обобщённого типа, которые мы используем в определении структуры, не всегда совпадают с аналогами, использующимися в сигнатурах методов этой структуры. Чтобы пример был более очевидным, в листинге 10-11 используются обобщённые типы `X1` и `Y1` для определения структуры `Point` и типы `X2` `Y2` для сигнатуры метода `mixup`. Метод создаёт новый экземпляр структуры `Point`, где значение `x` берётся из `self` `Point` (имеющей тип `X1`), а значение `y` - из переданной структуры `Point` (где эта переменная имеет тип `Y2`).

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

<span class="caption">Листинг 10-11: метод, использующий обобщённые типы, отличающиеся от типов, используемых в определении структуры</span>

В функции `main` мы определили тип `Point`, который имеет тип `i32` для `x` (со значением `5` ) и тип `f64` для `y` (со значением `10.4`). Переменная `p2` является структурой `Point`, которая имеет строковый срез для `x` (со значением `«Hello»`) и `char` для `y` (со значением `c`). Вызов `mixup` на `p1` с аргументом `p2` создаст для нас экземпляр структуры `p3`, который будет иметь тип `i32` для `x` (потому что `x` взят из `p1`). Переменная `p3` будет иметь тип `char`  для  `y` (потому что `y` взят из `p2`). Вызов макроса `println! ` выведет `p3.x = 5, p3.y = c`.

Цель этого примера — продемонстрировать ситуацию, в которой некоторые обобщённые параметры объявлены с помощью `impl`, а некоторые объявлены в определении метода. Здесь обобщённые параметры `X1` и `Y1` объявляются после `impl`, потому что они относятся к определению структуры. Обобщённые параметры `X2` и `Y2` объявляются после `fn mixup`, так как они относятся только к методу.

### Производительность кода, использующего обобщённые типы

Вы могли бы задаться вопросом, возникают ли какие-нибудь дополнительные издержки при использовании параметров обобщённого типа. Хорошая новость в том, что при использовании обобщённых типов ваша программа работает ничуть ни медленнее, чем если бы она работала с использованием конкретных типов.

В Rust это достигается во время компиляции при помощи мономорфизации кода, использующего обобщённые типы. *Мономорфизация* — это процесс превращения обобщённого кода в конкретный код путём подстановки конкретных типов, использующихся при компиляции. В этом процессе компилятор выполняет шаги, противоположные тем, которые мы использовали для создания обобщённой функции в листинге 10-5: он просматривает все места, где вызывается обобщённый код, и генерирует код для конкретных типов, использовавшихся для вызова в обобщённом.

Давайте посмотрим, как это работает при использовании перечисления `Option<T>` из стандартной библиотеки:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Когда Rust компилирует этот код, он выполняет мономорфизацию. Во время этого процесса компилятор считывает значения, которые были использованы в экземплярах `Option<T>`, и определяет два вида `Option<T>`: один для типа `i32`, а другой — для `f64`. Таким образом, он разворачивает обобщённое определение `Option<T>` в два определения, специализированные для `i32` и `f64`, тем самым заменяя обобщённое определение конкретными.

Мономорфизированная версия кода выглядит примерно так (компилятор использует имена, отличные от тех, которые мы используем здесь для иллюстрации):

<span class="filename">Файл: src/main.rs</span>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Обобщённое `Option<T>` заменяется конкретными определениями, созданными компилятором. Поскольку Rust компилирует обобщённый код в код, определяющий тип в каждом экземпляре, мы не платим за использование обобщённых типов во время выполнения. Когда код запускается, он работает точно так же, как если бы мы продублировали каждое определение вручную. Процесс мономорфизации делает обобщённые типы Rust чрезвычайно эффективными во время выполнения.


