---
title_image_path: basics.jpg
category: Програма
created_at: 2023-02-27T10:00:00
author: valo, meddle, ivan
tags:
  - elixir
  - integer
  - float
  - string
  - tuple
  - list
  - map
  - keyword list
  - order
  - immutability
---

# Основни типове

Да си говорим какви типове предлага даден език не е много интересно, но пък е нужно да знаем с какво разполагаме.
Ще се опитаме да ви представим някои от типовете набързо, за да можем да продължим с по-интересни неща.
Списъци, речници, структури и низове ще разгледаме по-подробно в следващи публикации, но ще ги споменем и тук.

## Числа

Elixir предлага цели числа и числа с плаваща запетая:

```elixir
1    # В десетична бройна система
#=> 1
10_000
#=> 10000

0x53 # В шестнадесетична
#=> 83
0o53  # В осмична
#=> 43
0b11 # В двоична
#=> 3

3.14 # С плаваща запетая
#=> 3.14
1.0e-10 # С плаваща запетая
#=> 1.0e-10
```

Операторът `/` връща като резултат число с плаваща запетая. Целочислено деление се извършва с функцията `div(nom, den)`.

```elixir
1 + 41
#=> 42
21 * 2
#=> 42

54 / 6 # Връща резултат с плаваща запетая
#=> 9.0
div(33, 10) # Целочислено деление
#=> 3
rem(33, 10) # А ето как получаваме остатъка
#=> 3

```
Езикът съдържа и операции за сравнение:

```elixir
1 < 2
#=> true
1 <= 2
#=> true
1 >= 1
#=> true
1 > 1
#=> false
1 != 2
#=> true
1 == 2
#=> false
1 == 1.0 # Операторът == сравнява по стойност
#=> true
1 === 1.0 # Операторът === сравнява по стойност И тип
#=> false
1 !== 1.0 # Операторът !== сравнява по стойност И тип
#=> true
```

> Съществува частична наредба на **всички** типове в Elixir. Тя е следната:
> ```
> number < atom < reference < function < port < pid < tuple < map < list < binary
> ```
> Това означава, например, че всяко число е по-малко от всеки атом, както и че всеки атом е по-малък от всяка референция и т.н.
>
> Наредбата е частична, а не пълна, защото целите числа и числата с плаваща запетая се конвертират преди сравнение. Двете сравнения `1 < 1.0` и `1.0 < 1` връщат `false`, както и `1 === 1.0` също връща `false`. Числото с по-ниска прецизност се конвертира към числото с по-висока прецизност.

Няколко полезни функции, свързани с числа:

```elixir
is_integer(3)  # За проверка на типа - дали е цяло число
#=> true
is_integer(3.0)
#=> false
is_float(3.0) # Дали е с плаваща запетая
#=> false
is_number(3.0) # Има и проверка дали е число изобщо
#=> true
is_number(3)
#=> true

round(3.2) # За закръгляне
#=> 3
round(3.8) # За закръгляне
#=> 4
trunc(3.8) # За отрязване
#=> 3
```


Съществуват специални модули - `Integer` и `Float`, предлагащи функции за работа
с числа. В тях може да намерите още подобни функции. Няма ограничение за големината на целите числа.

> Ако се опитате да извикате директно `Integer.is_odd(5)`, то ще получите грешка. Това е така, защото `Integer.is_odd/1` не е функция, а макрос. За да извикате макрос от даден модул, то трябва да изпълните `require Integer` или `import Integer` във дадения модул или iex сесия. Повече информация за макросите може да намерите в [тази](https://elixir-lang.bg/archive/posts/metaprogramming_part1) и [тази](https://elixir-lang.bg/archive/posts/metaprogramming_part2) статия, свързани с метапрограмирането

## Булеви стойности: true/false

```elixir
true
#=> true
false
#=> false

is_boolean(true)
#=> true
is_boolean(0)
#=> false

true == false
#=> false
```

Операторите за работа с булеви стойности са `and`, `or`, `&&`, `||`, `not` и `!`.
Разликата между съотвестващите оператори: `and` и `&&`, `or` и `||`, `not` и `!` e, че `and`, `or` и `not` приемат само булеви стойности.
Те са стриктни и ако им се подаде нещо различно от `true` и `false` (`:true` и `:false`, както ще видим след малко), ще се получи `ArgumentError`.

Това поведение се илюстрира в следните примери:

```elixir
true or false
#=> true
false and true
#=> false
not true
#=> false
5 or 42
#=> ** (BadBooleanError) expected a boolean on left-side of "or", got: 5
nil || {:error, "unexpected nil result"}
#=> {:error, "unexpected nil result"}
```

В Elixir само `false` и `nil` се приемат за _falsey_ стойности.

## Атоми

Атомите са константи, чието име е стойността им.

* Булевите стойности `true` и `false`, както и `nil` всъщност са атомите `:true`, `:false` и `:nil`.
* Имената на модули в Elixir също са атоми. `MyModule` е съкратена версия (syntax sugar) за `:"Elixir.MyModule"` (Ако атомът съдържа специално симпволи, то той се загражда в "")
* Модули, идващи от Erlang, са реферирани от атоми.
* Удобни са за ползване като ключове в *Map*-ове.
* Задължителна част от *keyword lists*.
* Често се използват в кортежи за означаване на резултат от функция. Пример - `{:ok, 2}` или `{:error, "Something went wrong}`.
* Освен ако не са в двойни кавички, атомите могат да съдържат подчертавки, цифри и латински букви, както и **@**.
* Атомите могат да завършват на `!` или на `?`.
* Идеални за *pattern matching* (съпоставянето им е еквивалентно на съпоставяне на числа). Това е така поради тяхната имплементация. При създаването на нов атом, в специална хеш таблица (`Atom table`) се създава нов запис и на този атом се съпоставя числова стойност, която се използва от виртуалната машина при операции с този атом.

Атомите са важна част от Elixir. За запознатите с Ruby – еквивалентни са на символите.

> Атомите никога не се изчистват от Garbage Collector механизма. Затова е важно тяхната употреба да е разумна. Важно правило е, че никога не трябва да конвертирате стойност към атом, ако тази стойност е подадена от потребителя или външния свят. Неконтролираното създаване на нови атоми може да доведе то прекратяване на работата на програмата и виртуалната машина поради изчерпване на паметта.
> Функцията `String.to_existing_atom/1` може да бъде използване, за да се конвертира низ към атом, ако той вече съществува. Ако не съществува се вдига грешка `ArgumentError`.

```elixir
:atom
#=> :atom

:true
#=> true

:anoter_atom
#=> :anoter_atom

SomeModule # Може и да не е дефиниран
#=> SomeModule
SomeModule == :"Elixir.SomeModule" # Това е истинското име на модула
#=> true

is_atom(:atom)
#=> true
is_atom(true)
#=> true

true == :true
#=> true

:"atom with a space" # Могат да се дефинират и така
#=> :"atom with a space"
```

## Низове

Низовете в Elixir се дефинират с двойни кавички и са с *UTF-8 encoding*:

```elixir
"Здрасти"
#=> "Здрасти"
"Здрасти #{:Pesho}" # Интерполация
#=> "Здрасти Pesho"

"""
Един
стринг
на
повече
от един ред
"""
#=> "Един\nстринг\nна\nповече\nот един ред" # Поддържа множество редове

is_binary("Здрасти") # Низовете представляват поредица от байтове
#=> true
String.length("Здрасти") # Брой на символи
#=> 7
byte_size("Здрасти") # Брой на байтове. Един символ на кирилица е представен от 2 байта в UTF-8 кодировката
#=> 14

"Бял" <> " мерцедес!" # Конкатенация
#=> "Бял мерцедес!"
```

За удобна работа с _UTF-8_ низове съществува модул `String`.
В една от [следващите публикации](https://elixir-lang.bg/archive/posts/binaries) ще се запознаем с имплементацията на низовете и защо те съществено се различават от познатите имплементации на низове. Тази имплементация и изразителна мощ е породена от факта, че една от основите целите `Erlang` е била имплементацията на протоколи в телеком индустрията (`diameter`, `smtp` и др.). Затова имплементацията на binary протоколи е толкова лесна в Elixir.

## Списъци

Дефинират се така:

```elixir
[1, 2, "три", 4.0] # Не са хомогенни
#=> [1, 2, "три", 4.0]
length [1, 2, 3, 5, 8] # Дължината
#=> 5
hd [1, 2, 3, 5, 8] # Връща първия елемент (head)
#=> 1
tl [1, 2, 3, 5, 8] # Връща списък с елементите без първия (tail)
#=> [2, 3, 5, 8]
is_list([1, 2])
#=> true
```

* Списъците са важна структура, има специален модул `List` за работа с тях.
* Не държат стойностите си подредени в паметта.
* Намирането на дължината им, четене на стойност по index, добавяне на стойност на index и триене на стойност на index са линейни операции.

В една от [следващите публикации](https://elixir-lang.bg/archive/posts/lists_streams_recursion) ще говорим по-подробно за тях.

## Кортежи

Българският превод на `tuple` е `кортеж`. 
Кортежите представляват наредени n-ори от елементи. Подобно на списъците, могат да съдържат стойности от всякакъв тип.

```elixir
{:ok, 7}
#=> {:ok, 7}
tuple_size({:ok, 7, 5})
#=> 3
is_tuple({:ok, 7, 5})
#=> true
```

* Кортежите съхраняват елементите си подредени един след друг в паметта.
* Достъпът до елемент по индекс и взимането на дължината им са константни операции.
* Ползват се за много неща:
  * Заедно с атомите за връщане на множество стойности от функция.
  * За *pattern matching* - ще видим в следващата публикация.
  * Read-only колекция, защото писането в тях е скъпа операция.
* Кортежите **не** са персистентна структура от данни. При опит за добавяне/премахване/редакция на елемент от кортеж, нито една част от оригиналния кортеж не се преизползва.
  
## Keyword lists

За тези списъци е по-добре да ползваме английското понятие (иначе са _асоциативни списъци_). Като цяло това са списъци, които съдържат *tuple*-и от по два елемента.
Всеки такъв *tuple* има за първи елемент атом – ключ.

```elixir
[{:one, 1}, {:two, 2}]
#=> [one: 1, two: 2]  # Както виждате, има специален синтаксис за тях. Това е същото:
 [one: 1, two: 2]
 #=> [one: 1, two: 2]
```

Главно се използват за _keyword_ аргументи на функции. Ако keyword list е последен
аргумент на функция, можем да пропуснем квадратните скоби при извикване:
```elixir
f(1, 2, three: 3, four: 4)
```

Ключовете им могат да се повтарят.

Пример `String.split/3`:
```elixir
String.split("one,two,,,three,,,four", ",", trim: true)
#=> ["one", "two", "three", "four"] # Няма празни низове заради опцията trim: true.
```

## Речници

Колекции от ключове и стойности.

* *Map*-овете в Elixir не позволяват еднакви ключове.
* За ключове може да се използва всичко и дори няма нужда да бъдат един и същи тип, но обикновено се използват низове или атоми.
* Малки речници с по-малко от 32 елемента са имплементирани чрез двойка списъци.
* При наличието на повече от 32 елемента са имплементирани са чрез [HAMT](https://en.wikipedia.org/wiki/Hash_array_mapped_trie)
* Достъпът до елемент по ключ и взимането на дължината им са операции с логаритмична сложност.

```elixir
%{"one" => 1, "two" => 2}
#=> %{"one" => 1, "two" => 2}
%{one: 1, two: 2} # Ако ключовете са атоми - има опростен начин за създаване.
#=> %{one: 1, two: 2}
```

## Двоичен тип (Binaries)

Представляват поредици от битове и байтове.

```elixir
<< 2 >> # Цялото число 2 в 1 байт
#=> <<2>>
byte_size << 2 >>
#=> 1

<< 255 >> # Цялото число 255 в 1 байт
#=> <<255>>
<< 256 >> # Превърта и става 0
#=> <<0>>
<<1, 2>> # Две цели числа в два байта.
#=> <<1, 2>>

byte_size << 1, 2 >>
#=> 2
```

Не е задължително едно поле да е един байт, това може да се управлява:

```elixir
<< 5::size(3), 1::size(1), 5::size(4) >>
#=> <<181>>
0b10110101
#=> 181

byte_size << 5::size(3), 1::size(1), 5::size(4) >>
#=> 1

is_bitstring << 5::size(3), 1::size(1) >>
#=> true
is_binary << 5::size(3), 1::size(1) >> # Не е цял байт. Трябва броят на битовете да е кратен на 8.
#=> false
```

Цялото *binary* по-горе е един байт:

1. Числото *5* е пакетирано в *3* бита -> *101*.
2. Числото *1* – един бит -> *1*.
3. Числото *5*  е пакетирано в *4* бита -> *0101*

Общо *8* бита - *1* байт, и същото като `0b10110101` или `181`.

Интересен факт – низовете в Elixir са имплементирани като *binary* тип.
Спомняте си, че `is_binary("Стринг")` връщаше `true`.

## Анонимни функции

```elixir
fn (x) -> x + 1 end
#=> #Function<6.52032458/1 in :erl_eval.expr/5>

(fn (x) -> x + 1 end).(4) # Извикване
#=> 5

is_function((fn (x) -> x + 1 end))
#=> true
```

Има и друг начин за дефиниране на анонимни функции:

```elixir
&(&1 + 1) # Тук &(тяло) е дефиницията на функцията. &1 в тялото значи 'първи параметър'
#=> #Function<6.52032458/1 in :erl_eval.expr/5>
(&(&1 + 1)).(4)
#=> 5
```

Само първият начин позволява дефиницията на анонимна функция без аргументи.

## Други типове

Други типове са *Port*, който се използва за комуникация с OS-level процеси и IO (като цяло с външния свят), *Reference*, който се използва за уникални стойности в рамките на виртуалната машина, и *PID*, който се използва с Erlang/Elixir процеси.

Езикът съдържа синтаксис за регулярни изрази и работа с тях (`~r/\w+/im`), както и ranges (`1..1000`), но това не са отделни типове, а структури, които пък са речници. Ще ги разгледаме в следваща публикация.

Тази статия има за цел да представи различните типове на едно базово ниво.
Когато навлезем в езика, ще се запознаем по-подробно с тях.
Ще има публикации (както споменахме) за определени типове, а като цяло ще ги ползваме в различни упражнения.

Добро начало за опознаване на възможностите на езика е документацията на `Kernel` модула, която можете да намерите [тук](https://hexdocs.pm/elixir/Kernel.html).

## Сравняване и типове

Ето и пълен списък с операторите за сравнение, както е дефиниран в [документацията на Erlang](http://erlang.org/doc/reference_manual/expressions.html):

| Операция |                             Какво прави                             |
| :------: | :-----------------------------------------------------------------: |
| ** ==**  |                Сравнява аргументите си дали са равни                |
| ** !=**  |              Сравнява аргументите си дали са различни               |
| ** =<**  | Връща `true`, ако първият ѝ аргумент е по-малък или равен на втория |
| ** < **  |      Връща `true`, ако първият ѝ аргумент е по-малък от втория      |
| ** >=**  | Връща `true`, ако първият ѝ аргумент е по-голям или равен на втория |
| ** > **  |      Връща `true`, ако първият ѝ аргумент е по-голям от втория      |
| ** ===** |     Сравнява аргументите си – дали са равни и от един и същ тип     |
| ** !==** |   Сравнява аргументите си – дали са различни и от различни типове   |
{: .table .table-striped .table-bordered .table-hover}

Има подредба между типовете:

|        Тип         | Подредба спрямо долния ред |
| :----------------: | :------------------------: |
|       число        |          ** < **           |
|        атом        |          ** < **           |
| reference стойност |          ** < **           |
|      функция       |          ** < **           |
|        порт        |          ** < **           |
|        pid         |          ** < **           |
|       кортеж       |          ** < **           |
|       речник       |          ** < **           |
|        nil         |          ** < **           |
|       списък       |          ** < **           |
|  двоична стойност  |                            |
{: .table .table-striped .table-bordered .table-hover}

Списъците се сравняват елемент по елемент, кортежите първо по големина, а ако са еднакво големи, също елемент по елемент.
Речниците подобно на *tuple*-ите се сравняват първо по големина (брой на ключовете), а после по самите ключове, като тук ключове цели числа се смятат за по-малки от ключове от тип числа с плаваща запетая.

Примери с различни типове:
```elixir
1 > :c
#=> false

%{1 => 3} > %{1.0 => 3}
#=> false

%{one: 1, four: 4} == %{one: 1.0, four: 2.0}
#=> false

%{one: 1, four: 4} == %{one: 1.0, four: 4}
#=> true
```

## Неизменимост (Immutablility)

В идеалния свят, когато извикаме функция с една и съща стойност хиляда пъти, трябва да получим един и същи резултат хиляда пъти. И светът не трябва да се променя тайно от нас.
Трябва да остане същият – познат и предвидим. На практика обаче не всичко е идеално.
Това не ни спира да се стремим към своите идеали.

Обикновено искаме да направим нещата колкото се може по-просто и бързо, и те да работят.
Колкото повече неща се променят в нашата програма, докато тя върви към целта си, за толкова повече неща трябва да мислим, да дебъгваме, да ги търсим из кода и паметта.

Защо да си го причиняваме, когато нашата програма може да е просто множество композирани функции, които не изменят състояния?
Тази идея е залегнала във функционалното програмиране.
Трудно е да не променяме целия свят (макар и не невъзможно, може да имаме поредица от познати светове), но винаги е по-лесно да не променяме това, което знаем как да променим или да не променим.

Веднъж създадена, една структура от данни не трябва да може да бъде променяна.
Хубаво е да имаме функции, които създават една структура с база друга – и това е всичко, от което се нуждаем.
След всичко казано по-горе е време да ви споделим един факт.
Всички типове, които видяхте дотук, всички структури и колекции от данни са точно такива – непроменими (*immutable*).

Сега въпросът е – това не е ли неефективно? Да речем, имаме си един списък от хиляда елемента, искаме да вдигнем всеки от тях на квадрат. Как става това? Ами строим нов списък с квадратите, а старият си остава непроменен.

Но не всеки път нещата са такива. Elixir прeизползва каквото може от базовата структура, когато прави нова.
Все пак базовата структура също е *immutable*, няма да се промени с времето.

```elixir
base_list = [1, 2, 3]
#=> [1, 2, 3]
new_list = [0 | base_list]
#=> [0, 1, 2, 3]
```

В примера новият списък преизползва всичките елементи от първия, освен първия.
Важното е да запомните, че функциите, идващи от модули, като `List`, `Enum`, `String` винаги трансформират аргументите си, като създават нови структури, никога не ги модифицират.

Това беше най-базовото, което трябва да знаете, преди да започнете да използвате езика по-сериозно.
Не беше малко, но не е много сложно.
Ако досега не сте се занимавали с функционален език, може да ви е доста странно.
Не забравяйте, всяко непознато нещо винаги е леко страшно отначало.
Това не означава, че не е правилната стъпка напред във вашето развитие.
