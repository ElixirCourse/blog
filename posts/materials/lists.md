---
title_image_path: lists.jpg
category: Програма
author: valo, meddle
created_at: 2018-04-25T11:00:00
tags:
  - elixir
  - fold
  - reduce
  - immutable
  - list
  - charlist
---

# Списъци

Вече разгледахме основните типове като числа и атоми, разгледахме и речниците, време е да разгледаме още един тип в *Elixir*, който представлява поредица от данни.
Това е списъкът (*list*).

## Какво представляват списъците

Един списък може да се представи рекурсивно като главата на списъка (*head*), в която е стойността на първия му елемент и указател към следващия елемент.
Поради това представяне, можем да гледаме на списъците като на кутийки, в които има стойност и всяка кутийка има връзка към следваща кутийка.
Единствено последната кутийка няма връзка към следваща:

```
+-+-+   +-+-+   +-+-+   +-+-+   +-+-+
|1| |-->|2| |-->|3| |-->|4| |-->|5| |
+-+-+   +-+-+   +-+-+   +-+-+   +-+-+
```

Това е списъкът `[1,2,3,4,5]`. Главата му съдържа `1` и сочи към следващият елемент, който съдържа `2` и има указател към следващият елемент, който има стойност `3` и т.н.
Само последният елемент, който съдържа `5`, няма указател към следващ елемент.
Това е често срещано представяне на списък във много функционални езици.

В Еликсир подобна зависимост може да се запише така:

```elixir
[1 | [2 | [3 | [4 | [5 | []]]]]]
#=> [1, 2, 3, 4, 5]
```

В горния пример, използваме специален оператор (`|`) за да опишем каква е стойността на елемента и каква е опашката на списъка.
Горният ред казва: главата на списъка съдържа `1` и сочи към опашката на списъка: `[2 | [3 | [4 | [5 | []]]]]`,
който е списък с глава, която съдържа `2` и сочи към опашката на списъка: `[3 | [4 | [5 | []]]]` и т.н.
Последният елемент е `[5 | []]`, който съдържа `5` и сочи към опашка, който е празен списък. Това маркира края на данните.

Друг начин за да се опише горната структура е: `[head | tail]`, където `head` е стойността на главата на списъка, а `tail` е опашката.
Това е и синтаксисът, ако искаме да *pattern match*-ваме списъци:

```elixir
[a | b] = [1, 2, 3]
#=> [1, 2, 3]

a
#=> 1
b
#=> [2, 3]
```

Горният пример няма да работи ако списъкът е празен:

```elixir
[a | b] = []
#=> ** (MatchError) no match of right hand side value: []
```

Ако списъкът съдържа един елемент, обаче, ще работи:

```elixir
[a | b] = [25]
#=> [25]

a
#=> 25
b
#=> []
```

Това е пряко следствие от представянето на списъка. Списък от един елемент е *head* със стойност елемента и указател към празния списък.
Точно това *match*-нахме.


## Рекурсивно обхождане на списък

Горната дефиниция ни дава възможност да пишем функции, които работят със списъци.
Нека например напишем функция, която връща дължината на списък:

```elixir
defmodule ListUtils do
  def length([]), do: 0
  def length([_head | tail]), do: 1 + length(tail)
end
```

Имаме 2 версии на функцията `ListUtils.length/1`:

* Когато списъкът е празен, знаем че неговата дължина е **0** (това е и дъното на рекурсията).
* Ако списъкът не е празен, то можем да го разделим на глава и опашка. Дължината му ще бъде **1 + <дължината на опашката>**.

Ако извикаме горната функция за един списък, операциите които ще се извършат са:

```elixir
ListUtils.length(["cat", "dog", "fish", "horse"])
= 1 + ListUtils.length(["dog", "fish", "horse"])
= 1 + 1 + ListUtils.length(["fish", "horse"])
= 1 + 1 + 1 + ListUtils.length(["horse"])
= 1 + 1 + 1 + 1 + ListUtils.length([])
= 1 + 1 + 1 + 1 + 0
= 4
```

Можем да имплементираме доста операции или обхождане и събиране на даден резултат чрез рекурсия върху списък.
Обикновено имаме версия на действието върху празен списък, която се явява дъно на рекурсията или край на операцията и връщане на резултат
и една или повече версии на функцията, които прилагат действието върху главата на списъка и рекурсивно извикват операцията над опашката.
Това е много често използван шаблон при обработването на списъци.

## Рекурсивно изграждане на списък

Аналогично можем и да изграждаме списъци чрез рекурсия, като ги дефинираме като "глава" и "опашка":

```elixir
a = [2, 3]
#=> [2, 3]

b = [1 | a]
#=> [1, 2, 3]

c = [1 | []]
#=> [1]

d = [a | [1]]
#=> [[2, 3], 1]
```

Нека напишем функция, която повдига всички елементи от един списък на квадрат:

```elixir
defmodule Square do
  def of([]), do: []
  def of([head | tail]), do: [head * head | of(tail)]
end

Square.of([1, 2, 3, 4, 5])
#=> [1, 4, 9, 16, 25]
```

Логиката е следната:

* Списъкът от квадрати на празен списък е празен списък.
* Списъкът от квадрати на не-празния списък `[head | tail]` е списък с първи елемент `head`<sup>2</sup> и опашка - списък от квадратите на `tail`.

Така рекурсивно можем да изграждаме списъци от други списъци.

Ако имаме един списък `list` и направим два нови списъка, използвайки `list` за опашка - `[1, 2 | list]` и `[3, 4 | list]`, то ние ще преизползваме `list` - той няма да бъде копиран и в паметта ще съществува само веднъж.

```elixir
а = [1, 2, 3, 4, 5, 6]
#=> [1, 2, 3, 4, 5, 6]

b = [0 | a]
#=> [0, 1, 2, 3, 4, 5, 6]
c = [-1 | a]
#=> [-1, 0, 1, 2, 3, 4, 5, 6]
```

Вътрешно при представянето на `b` и на `c` не се прави копие на `a`, а се използва указател към `a`, което е много ефективно откъм памет, но има *trade-off*-а,
че за да установим дължината на списъка трябва да го обходим.

В този контекст, винаги се опитвайте да добавяте елементи към началото на списък, тъй като така не се създават копия на списъка. Например **не правете** следното:

```elixir
defmodule Fib do
  def bad_of(n), do: bad_fib(n, 0, 1, [])

  defp bad_fib(0, _current, _next, seq), do: seq

  defp bad_fib(n, current, next, seq) do
    bad_fib(n - 1, next, current + next, seq ++ [current])
  end
end
```

При горната имплементация ще се генерира копие на списъка `seq` когато се направи `seq ++ [current]`.
Това се дължи на неизменимостта (*immutability*) на данните в *Elixir*.
Тъй като няма как да се промени последният елемент на списъка да сочи към нов елемент се налага да се направи копие на списъка.
По-оптималният вариант е следния:

```elixir
defmodule Fib do
  def of(n), do: fib(n, 0, 1, [])

  defp fib(0, _current, _next, seq), do: seq |> Enum.reverse()

  defp fib(n, current, next, seq) do
    fib(n - 1, next, current + next, [current | seq])
  end
end
```

Така няма да имаме нито едно копиране докато изграждаме списъка и накрая просто се налага да го обърнем, което е нов списък, но изграден само веднъж.

Използването на конкатенация при списъци (`list1 ++ list2`) не е винаги толкова лошо нещо.
За повече информация прочетете [секция 2.2 от The Seven Myths of Erlang Performance](http://erlang.org/doc/efficiency_guide/myths.html).
Ако, обаче, използваме конкатенацията без да се замислим ни грози опасност от лоша производителност.
Прочетете повече за това [тук](https://makandracards.com/neoukelixirtips/22101-++-operator-performance-gotcha)

## Имплементация на функцията map(list, fun)

Нека да видим как бихме имплементирали стандартния за функционалните езици `map`, използвайки горните похвати.
Нека припомним, че `map` е функция, която приема списък и функция с един аргумент и връща нов списък с резултата от извикването на функцията върху всеки елемент от списъка:

```elixir
defmodule ListUtils do
  def map([], _func), do: []
  def map([head | tail], func), do: [func.(head) | map(tail, func)]
end

ListUtils.map([1, 2, 3, 4, 5], fn x -> x * x end)
#=> [1, 4, 9, 16, 25]
```

Както виждате горната функция е обобщение на това, което направихме по-горе при рекурсивното изграждане на списъците.
Затова и тази операция е стандартна във функционалното програмиране и е включена в стандартната библиотека чрез функцията `Enum.map/2`.

## Имплементация на reduce(list, acc, func)

Друга интересна операция и обобщение на това което правихме при смятането на дължината на списъка е `reduce` (или `accumulate`, `foldl`, `inject`).
Това е функция, която приема списък, начална стойност - акумулатор и функция и извиква функцията с аргументи акумулатора и всеки елемент от списъка.
трябва да върне новата стойност на акумулатора:

```elixir
defmodule ListUtils do
  def reduce([], acc, _func), do: acc
  def reduce([head | tail], acc, func), do: reduce(tail, func.(head, acc), func)
end

ListUtils.reduce(["cat", "dog", "horse"], 0, fn _head, acc -> 1 + acc end)
#=> 3
```

Това което ще се случи при горното извикване е следното:

```elixir
add_one = fn _head, acc -> 1 + acc end
ListUtils.reduce(["cat", "dog", "horse"], 0, add_one)
= ListUtils.reduce(["dog", "horse"], 1, add_one)
= ListUtils.reduce(["horse"], 2, add_one)
= ListUtils.reduce([], 3, add_one)
= 3
```

Интересното е, че всичко това са стандартни примери за код където имаме нужда от някакъв променящ се *state* и за да избегнем писането на цикли използваме рекурсия.
Преимуществото на рекурсията е, че имаме много добра дефиниция кое е състоянието, което се прехвърля между отделните итерации - аргументите на функцията.
Няма как да използваме състояние (*state*), което е извън тези аргументи.
Това прави кода ни по-четим, по-модулен и по-лесен за промяна.

Операцията `reduce`, също както `map`, е основна при работата със списъци в един функционален език и затова е включена в стандартната библиотека под името `Enum.reduce`.
Също така, `Enumerable` протоколът изисква да бъде имплементирана единствено функцията `reduce` за даден тип, за да може всички функции от модула `Enum` да се свеждат до нея (каквато и да е рекурсивна операция може да се изрази с `reduce`).
Повече информация на [тук](https://hexdocs.pm/elixir/Enum.html#reduce/3) и [тук](https://hexdocs.pm/elixir/Enumerable.html).

## Модула List

*Elixir* идва с модул съдържащ функции за работа със списъци.
Този модул е `List`.

### Функции за достъп

Модулът съдържа функции за лесен достъп до първия `List.first/1` и последния `List.last/1` елемент.
Интересно е, че няма функция за достъп до произволен елемент.
Такива функции, които предполагат линейно обхождане можем да намерим в модула `Enum`, за който ще говорим в следваща публикация.

### Функции за "модификация"

За сметка на това, че нямаме функции за произволен достъп, имаме такива за промяна или добавяне на елемент на произволна позиция или изтриването му.
Това са `List.delete_at/2`, `List.update_at/3` която приема функция като действие за промяна и по-простата ѝ версия `List.replace_at/3`, която приема стойност и `List.insert_at/3`.
Тези функции работят и с отрицателни индекси. Няколко примера с тях и други подобни функции от модула:

```elixir
list = [:one, :two, :three, :four]
#=> [:one, :two, :three, :four]

list = List.insert_at(list, -1, :five)
#=> [:one, :two, :three, :four, :five]

list = List.insert_at(list, 20, :six)
#=> [:one, :two, :three, :four, :five, :six]

list = List.replace_at(list, 1, 2)
#=> [:one, 2, :three, :four, :five, :six]
list = List.replace_at(list, 2, 3)
#=> [:one, 2, 3, :four, :five, :six]

list = List.update_at(list, 2, &(&1 * 2))
#=> [:one, 2, 6, :four, :five, :six]

list = List.delete_at(list, -2)
#=> [:one, 2, 6, :four, :six]

list = List.delete(list, :four)
#=> [:one, 2, 6, :six]

{value, list} = List.pop_at(list, 0)
#=> {:one, [2, 6, :six]}
```

### Функциите foldl и foldr

Интересно е, че `List` съдържа две *reduce* функции, които задължително приемат начален елемент.
Функцията `List.foldl/3` се държи като `reduce`, която имплементирахме в предишната секция, а `List.foldr/3` обхожда елементите на списъка в обратен ред:

```elixir
# 1 - 1 = 0; 2 - 0 = 2; 3 - 2 = 1; 4 - 1 = 3
List.foldl([1, 2, 3, 4], 1, fn el, acc -> el - acc end)
#=> 3

# 4 - 1 = 3; 3 - 3 = 0; 2 - 0 = 2; 1 - 2 = -1
List.foldr([1, 2, 3, 4], 1, fn el, acc -> el - acc end)
#=> -1
```

Как бихме имплементирали `foldr`?
Ето една идея:

```elixir
defmodule ListUtils do
  defdelegate foldl(list, acc, func), to: __MODULE__, as: :reduce

  def foldr(list, acc, func), do: list |> Enum.reverse() |> reduce(acc, func)

  def reduce([], acc, _func), do: acc
  def reduce([head | tail], acc, func), do: reduce(tail, func.(head, acc), func)
end
```

Тук пре-използваме `ListUtils.reduce/3`, която имплементирахме в предишната секция.
Нашата версия на `foldr` е просто да предадем обърнат списък на `reduce`.
За версията на `foldl` използваме нов синтаксис. Операцията `defdelegate` дефинира функция, която ще делегира изпълнението си на друга функция и ще предаде аргументите си към нея.
И наистина:

```elixir
ListUtils.foldl([1, 2, 3, 4], 1, fn el, acc -> el - acc end)
#=> 3

ListUtils.foldr([1, 2, 3, 4], 1, fn el, acc -> el - acc end)
#=> -1
```

Както казахме `fold` функциите са много мощен инструмент и всичко може да бъде изразено чрез тях.
Имаме техни версии в `List` защото версията в `Enum` ще работи за всякакви колекции, а тези в `List` са оптимизирани за списъци.
Ето и известните функции `map` и `filter`, имплементирани с *fold/reduce*:

```elixir
defmodule ListUtils do
  defdelegate foldl(list, acc, func), to: __MODULE__, as: :reduce
  def foldr(list, acc, func), do: list |> Enum.reverse() |> reduce(acc, func)
  def reduce([], acc, _func), do: acc
  def reduce([head | tail], acc, func), do: reduce(tail, func.(head, acc), func)

  def map(list, func) do
    foldr(list, [], fn el, acc -> [func.(el) | acc] end)
  end

  def filter(list, predicate) do
    f = fn
      el, acc, true -> [el | acc]
      _, acc, false -> acc
    end

    foldr(list, [], fn el, acc -> f.(el, acc, predicate.(el)) end)
  end
end

ListUtils.map([1, 2, 3], &(&1 * &1))
#=> [1, 4, 9]

ListUtils.filter([1, 2, 3, 4, 5, 6], & rem(&1, 2) == 0)
#=> [2, 4, 6]
```

### Други полезни функции

В модула има още няколко функции, за които е добре да знаем, защото могат да ни влязат в употреба.

Чест случай е, когато искаме да напишем функция, която работи както за единична стойност, така и за списък от стойности.
Да речем:

```elixir
defmodule TestSuite do
  def run(test), do: <implementation>
  def run(tests) when is_list(tests), do: <implementation>
end
```

В такива моменти бихме делегирали единичната имплементация към тази за списък:

```elixir
defmodule TestSuite do
  def run(test), do: run([test])
  def run(tests) when is_list(tests), do: <implementation>
end
```

Но има и още по-прост вариант, да използваме `List.wrap/1`, която ако приеме списък го връща непроменен, но ако приеме друга стойност я слага в списък:


```elixir
defmodule TestSuite do
  def run(tests) do
    tests
    |> List.wrap()
    |> <implementation-for-lists>
  end
end
```

Функцията `List.duplicate/2` ни помага да си изградим списък от множество повтарящи се елементи:

```elixir
List.duplicate(:me, 2)
#=> [:me, :me]

List.duplicate([1, 2, 3], 2)
#=> [[1, 2, 3], [1, 2, 3]]
```

А пък с `List.flatten/1` можем да направим списък от списъци прост списък от стойности, премахвайки дълбочината:

```elixir
[1, 2, 3] |> List.duplicate(2) |> List.flatten()
#=> [1, 2, 3, 1, 2, 3]

List.flatten([1, [2, [2, [4]], [4]]])
#=> [1, 2, 2, 4, 4]
```

Последната функция, която ще разгледаме тук е `List.zip/1`, която взима списък от списъци и комбинира елементите на тези списъци на еднакви позиции:

```elixir
digits = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
lists = [1, 2, 3, 4, 5] |> List.duplicate(3)
#=> [[1, 2, 3, 4, 5], [1, 2, 3, 4, 5], [1, 2, 3, 4, 5]]

List.zip([digits | lists])
#=> [{0, 1, 1, 1}, {1, 2, 2, 2}, {2, 3, 3, 3}, {3, 4, 4, 4}, {4, 5, 5, 5}]
```

Интересно е, че `List` имплементира повечето си функции използвайки *Erlang*-ския модул `:lists`.
Този модул има много повече полезни функции, за които можете да прочетете [тук](http://erlang.org/doc/man/lists.html).
Да речем имплементациите на `foldl` и `foldr` се съдържат в `:lists`.
Също така повечето такива имплементации са *[BIF](http://rvirding.blogspot.bg/2009/10/what-are-bifs.html)*-ове (*Built-In Functions*) и имат добра производителност.

## Списъци от символи (charlists)

Списък от числа, които са валидни *ASCII condepoint*-и ще бъде принтиран като поредица от символите, които отговарят на тях:

```elixir
[83, 79, 83]
#=> 'SOS'
```

Всъщност ако низа `'SOS'` с единични кавички е точно това - списък:

```elixir
is_list('SOS')
#=> true
```

Един списък от числа ще бъде винаги представян като низ в единични кавички, ако всички тези числа са *printaple ASCII condepoint*-и:

```elixir
[83, 79, 83]
#=> 'SOS'

[1, 83, 79, 83]
#=> [1, 83, 79, 83]
```

В модула `List` има функции свързани с *charlist* списъци:

```elixir
# List.to_atom/1 и List.to_existing_atom/1 ще опитат да върнат атом от charlist:

List.to_atom([83, 79, 83])
#=> :SOS
List.to_atom([83, 79, 83, 1])
#=> :"SOS\x01"
List.to_atom([83, 79, 83, 1, :s])
#=> ** (ArgumentError) argument error

# List.to_integer/1 и List.to_float/1 ще опитат да върнат числа от charlist:

List.to_integer('54')
#=> 54
List.to_float('45.2')
#=> 45.2

List.to_string([83, 77, 69, 82, 67, 72])
#=> "SMERCH"
```

Можем да проверим дали списък е *printable* с `List.ascii_printable?/1` или част от него с `List.ascii_printable?/2`:

```elixir
List.ascii_printable?([83, 79, 83])
#=> true
List.ascii_printable?([83, 79, 83, 1])
#=> false
List.ascii_printable?([83, 79, 83, 1], 2)
#=> true
List.ascii_printable?([83, 79, 83, 1], 3)
#=> true
List.ascii_printable?([83, 79, 83, 1], 4)
#=> false
```

Много *Erlang*-ски функции приемат *charlist* като аргумент вместо низ.
Това е така, защото низовете в *Erlang* са нещо сравнително ново.
Като цяло низовете използват *binary* типа и са трудно съпоставими с *charlist*-ите, които са просто списъци.
Важно е да знаем, че низове, дефинирани между единични кавички и низове, дефинирани между двойни са две съвсем различни неща.
Запомнете, че когато използвате *Erlang*-ски функции, които би трябвало да изискват низ, те ще изискват *charlist* и може да се наложи трансформация от стринг към *charlist*. Има функция, която може да прави това : `String.to_charlist/1`.
Пример за функция от *Erlang*-ски модул, която работи с *charlist* можете да намерите на много места, да речем в модула `:file` : `:file.read_file('/tmp/some_file.txt')`.

## Неправилни (improper) списъци

Както казахме в началото на тази публикация, списъкът представлява поредица от
*head* стойност, която може да бъде каквато и да е стойност и *tail* стойност, която трябва да друг такъв списък.

```
 H T     H T     H T     H T     H T
+-+-+   +-+-+   +-+-+   +-+-+   +-+-+
|1| |-->|2| |-->|3| |-->|4| |-->|5| |
+-+-+   +-+-+   +-+-+   +-+-+   +-+-+
```

или

```elixir
[1 | [2 | [3 | [4 | [5 | []]]]]]
#=> [1, 2, 3, 4, 5]
```

Стойността *head* може да бъде абсолютно всичко.
Стойността *tail* е добре да бъде друг списък.
Но може и да не бъде друг списък. Тя също може да е всичко:

```elixir
[1 | [2 | [3 | [4 | [5 | 6]]]]]
#=> [1, 2, 3, 4, 5 | 6]
```

В този случай имаме *improper list*.
Функциите, които работят със списъци обикновено не проверяват дали *tail* стойността е списък, а го очакват.
Това би довело до грешки ако списъкът е неправилен:

```elixir
list = [1 | [2 | [3 | [4 | [5 | 6]]]]]
#=> [1, 2, 3, 4, 5 | 6]

length(list)
#=> ** (ArgumentError) argument error
List.last(list)
#=> ** (FunctionClauseError) no function clause matching in List.last/1
```

Хубаво е да знаем, че `Kernel.is_list/1` връща `true` даже ако списъкът е неправилен.

Най-лесно можете да получите *improper list* (при това без да го разберете) при конкатенация:

```elixir
[1, 2, 3] ++ 4
#=> [1, 2, 3 | 4]
```

Няма функция, която проверява дали списък е *proper*, но лесно можем да си напишем такава:

```elixir
defmodule ListUtils do
  def proper?(l) when is_list(l), do: is_proper?(l)

  defp is_proper?([]), do: true
  defp is_proper?([_h | t]), do: is_proper?(t)
  defp is_proper?(_), do: false
end


ListUtils.proper?([1, 2, 3])
#=> true
ListUtils.proper?([1, 2, 3] ++ 4)
#=>#=>  false
ListUtils.proper?([])
#=> true
ListUtils.proper?([] ++ [])
#=> true
ListUtils.proper?([] ++ [] ++ 4)
#=> false
ListUtils.proper?([4])
#=> true
ListUtils.proper?([4 | 5])
#=> false
```

За какво можем да ползваме неправилни списъци?

Едно интересно приложение е да си генерираме безкрайни списъци:

```elixir
defmodule ListUtils do
  def new_infinite(val, fun) do
    fn ->
      [val | new_infinite(fun.(val), fun)]
    end 
  end
end

lazy_ints = ListUtils.new_infinite(1, &(&1 + 1))
#=> #Function<0.26159329/0 in ListUtils.new_infinite/2>

[head | tail] = lazy_ints.()
#=> [1 | #Function<0.26159329/0 in ListUtils.new_infinite/2>]

head
#=> 1

[head | tail] = tail.()
#=> [2 | #Function<0.26159329/0 in ListUtils.new_infinite/2>]

head
#=> 2
```

В примера създаваме *improper list*, чиято опашка е е функция, която генерира списък.
По този начин опашката не се създава веднага, а само когато функцията-опашка се извика при поискване.
Това поведение прилича на поведението на потоците, за които ще говорим в следваща публикация.

Друго интересно приложение имаме, когато използваме списъците като `iolist`-и за *IO* операции.
За това ще си говорим в публикацията на тема *IO*.

## Заключение

Опитахме се да покрием максимално подробно темата за списъците, работата с тях и тяхната имплементация.
В следващата публикация ще представим типа *tuple* или кортеж и типа *keyword list*, който представлява списък от двуелементни кортежи във вида `{атом, стойност}`.

Списъците са много важна структура във функционалното програмиране, която е дефинирана рекурсивно и е доста интуитивна за работа.
Ще ползваме списъци често, а понякога ще се налага (*Erlang* модули) да използваме списъци и като поредица от символи.
Съветваме ви да ги разучите подробно за да сте максимално запознати с тях и как и кога да **не** ги ползвате (внимавайте с конкатенациите).