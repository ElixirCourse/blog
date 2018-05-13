---
title_image_path: strings.jpg
category: Програма
created_at: 2018-05-03T11:00:00
tags:
  - elixir
  - binaries
  - erlang
  - pattern matching
  - strings
  - unicode
  - utf8
---

# Низове

В тази публикация ще разгледаме низовете в *Elixir*.
Тъй като низовете представляват двоични структури, ще е добре да прочетете първо [за тях](https://elixir-lang.bg/materials/posts/binaries).
Всички операции свързани с *binaries*, които разгледахме, могат да бъдат приложени и върху стрингове.

Низовете в *Elixir* използват *unicode*.
Това значи, че както казахме предтавляват *binary* структури, които са поредица от *unicode codepoint*-и в *utf8* енкодинг.

## Какво е Unicode

*Unicode* е начин на представяне на символи. Свързан е донякъде с *ASCII*.
При *ASCII* всяко от числата от **0** до **127** отговаря на определен символ.
Тези числа, отговарящи на символи наричаме *codepoint*-и.
Ако решим да съхраняваме *ASCII codepoint*-ите в един байт, ще използваме само **7** бита, слагайки една **0** отпред.

При *ASCII* имаме *codepoint*-и само за латинските букви, цифрите и различни символи които можем (или не) да намерим по клавиатурите си.

Къде е кирилицата?
А гръцката азбука, от която се нуждаем толкова много, когато пишем математически формули?
Къде е КСИ (ξ), както и да се пише наистина?
На практика има различки енкодинги като *CP-1252* които допълват с нещо основните 128 кодпойнта до **1 байт**, но те са много на брой и ограничени.

Това, което *unicode* предлага е *codepoint*-и за огромен набор от символи, включващи както кирилицата, така и гръцката азбука, също множество канджи (йероглифи) и символи,vкоито могат да се обединят в йероглифи.
Включва и странни неща като карти за игра, емотикони и алхимични символи...

На теория, *Unicode* включва символи за всичко изразимо на който и да е човешки език.
Има си отклоненията от тази теория на практика, но работи в огромен процент от случаите.

И *Unicode* е като *ASCII*, дефинира числа, съответващи на символи - огормен брой *codepoint*-и.
Разбира се тези числа трябва да се съхраняват по някакъв начин.
Тъй като се съхраняват на компютър, те ще се представят като поредици от нули и единици.
Как точно се съхраняват, зависи от енкодинга, който се ползва.
Има много енкодинги, да речем *utf-32* изполва по **4 байта** за всеки *codepoint*, което не е много оптимално.
В *Elixir*, низовете се пазят в *unicode utf-8* енкодинг.

## UTF-8

При *UTF-8* даден *codepoint* може да се съхранява в от един до четири байта (на практика е възможно и в повече).
Първите **128** кодпойнта съвпадат с *ASCII codepoint*-ите. Te се пазят в един байт, който започва с **0**:

```
0XXXXXXX
```

Има специфики защо даден номер съответства на даден символ при тези първи **128** символа.
Да речем цифрите винаги има префикс от **4 бита** - **0011**:

```elixir
<< 0b0011_0000 >>
#=> "0"
<< 0b0011_0001 >>
#=> "1"
<< 0b0011_0010 >>
#=> "2"
<< 0b0011_0101 >>
#=> "5"
<< 0b0011_1001 >>
#=> "9"
```

Главните латински букви започват след `0b01000000`, тоест имат префикс **010**:

```elixir
<< 0b010_00001 >>
#=> "A"
<< 0b010_11010 >>
#=> "Z"
```

Малките букви са след `0b01100000`, което значи, че ако добавим **100000 (32)** към главна буква получаваме малка.

След като **128**-те възможности за **1** байт се изчерпат, започват да се ползват по **2** байта:

```
110XXXXX 10XXXXXX
```

В този шаблон за двубайтови *codepoint*-и, виждаме че първият байт винаги започва с **110**.
Двете единици значат два байта. Байт започващ с **10**, означава че е част от поредица от байтове.
Тъй като в този порядък се пазят буквите от кирилицата, нека разгледаме какво прдставлява една от тях:

```elixir
?Ъ
#=> 1066
```

Така с въпросителен знак пред символ можем да видим *codepoint*-а на символа.
Нека да видим какво представлява **1066** в двоична бройна система:

```elixir
Integer.to_string(1066, 2)
#=> "10000101010"
```

За целта ползваме `Integer.to_string/2`, която взима за втори аргумент база.
Ще пробваме да вкараме числото `10000101010` в шаблона на *utf8* за два байта:

```
(110)10000 -> Префиксваме с 110 и допълваме до байт с битовете от числото.
(10)101010 -> Префиксваме с 10 и допълваме до байт с битовете, които останаха.
```

От горното следва, че *Ъ* се пази като **11010000 10101010**. Нека да сложим тези два байта в *binary*:

```elixir
<< 0b110_10000, 0b10_101010 >>
#=> "Ъ"
```

Така получаваме низа `"Ъ"`.

Шаблонът за три-байтови *codepoint*-и е:

```
1110XXXX 10XXXXXX 10XXXXXX
```

А шаблонът за четири-байтовите:

```
11110XXX 10XXXXXX 10XXXXXX 10XXXXXX
```

Както виждате броят на единиците, с които започва началния байт определя колко байта е общо *codepoint*-а.

Можем да заключим че има три типа байтове:
1. Единичен - започва с **0** и представлява първите **128** ASCII-compatible *codepoint*-и.
2. Байт-продължение - започва с **10** и следва други байтове в *codepoint* от повече от **1** байт.
3. Байт-начало - започва с **110**, **1110**, **11110**, първи байт от серия байтове представляващи *codepoint*.

Интересно е че при буквите в кирилицата първият байт е винаги **208 (11010000)**.
Главните започват от втори байт **144 (10010000)** a малките от **176 (10110000)**.
Отново разликата между главните и малките букви е **32 (100000)**.

## Графеми и codepoint-и

Символите или графемите не винаги са точно един *codepoint*.
Има графеми, които могат да се представят и от един *codepoint* и от няколко.
Такъв случай е, да речем, когато имаме комбинация с диакритични знаци.
Нека да разгледаме следния код:

```elixir
name = "Николай"
#=> "Николай"

String.codepoints(name)
#=> ["Н", "и", "к", "о", "л", "а", "й"]
String.graphemes(name)
#=> ["Н", "и", "к", "о", "л", "а", "й"]
String.graphemes(name) == String.codepoints(name)
#=> true
```

Започваме да използваме `String` модула.
Той ни дава две функции за превръщането на низ в списък от графеми или *codepoint*-и, които връщат напълно еднакви списъци.
Тогава защо имаме две? Кога ще са различни?

```elixir
name = "Николаи\u0306"
#=> "Николай"

String.codepoints(name)
#=> ["Н", "и", "к", "о", "л", "а", "и", ̆"<не може да се представи>"]
String.graphemes(name)
#=> ["Н", "и", "к", "о", "л", "а", "й"]
```

Това пак е името 'Николай', но графемата 'й' е съставена от два codepoint-а - `и` и диакритичен знак.

```elixir
name1 = "Николай"
#=> "Николай"
name2 = "Николаи\u0306"
#=> "Николай"

name1 == name2
#=> false
String.equivalent?(name1, name2)
#=> true
```

Такa написани, стрингове могат да се сравняват със `String.equivalent?/2` ако ни интересуват само графемите в тях, a не как са представени.

Функции като `String.reverse/1` работят правилно, защото ползват `String.graphemes/1`:

```elixir
String.reverse(name2)
#=> "йалокиН"

# Аналогично:
String.graphemes(name2) |> Enum.reverse |> Enum.join("")
#=> "йалокиН"
```

## Дължина на низ

Тъй като стринговете са *binary*, можем да използваме функциите от `Kernel` - `Kernel.byte_size/1` и `Kernel.bit_size/1`.
Те няма да ни върнат броя на графемите, но са бързи операции **O(1)**.
`String.length/1` ще ни върне броя на графемите и за да го направи, трябва да мине през всички байтове, да ги групира в *codepoint*-и, а тях да групира в графеми.
Това е линейна **O(N)** операция:

```elixir
Kernel.bit_size("Николаи\u0306")
#=> 128
Kernel.byte_size("Николаи\u0306")
#=> 16

String.length("Николаи\u0306")
#=> 7
```

## Под-низове

Има доста функции за взимане на под-низ от низ.
Ако искате кодът ви да бъде оптимален, когато работите с низове съставени от първите **128** *codepoint*-а, използвайте *binary pattern matching* или *binary* функции за обхождане.
Те са по-бързи и по този начин могат да се преизползват *matching context*-и.

Нека да видим как да вземем символ на дадена позиция:

```elixir
String.at("Искам бира!", 6)
#=> "б"
String.at("Искам бира!", -1)
#=> "!"
String.at("Искам бира!", 12)
#=> nil
```

Както виждате, можем да използваме отрицателни индекси за да започнем от края.
Ако индексът е невалиден, получаваме атома `nil`.
Ако искаме да вземем 6-тия символ с *pattern matching*, ще трябва да напишем нещо такова:

```elixir
<< _::binary-size(11), x::utf8, _::binary >> = "Искам бира!"
#=> "Искам бира!"

x
#=> 1073
<< x::utf8 >>
#=> "б"
```

Знаем това по-горе защото имаме **5** символа от по **2** байта и един - шпацията от **1** байт, затова пропускаме първите **11** байта.
Добре е да работим с *binary pattern matching* само когато сме сигурни какво имаме като съдържание на стринга.

Това също работи:

```elixir
<< "Искам ", x::utf8, _::binary >> = "Искам бира!"
#=> "Искам бира!"
```

Други интересни функции:

```elixir
String.next_codepoint("Николаи\u0306")
#=> {"Н", "иколай"}
String.next_codepoint("и\u0306")
#=> {"и", "<нещо което скапва hilighting-a на кода>"}

String.next_grapheme("и\u0306")
#=> {"й", ""}

String.next_grapheme_size("и\u0306")
#=> {4, ""}
String.next_grapheme_size("\u0306")
#=> {2, ""}
```

Горните функции връщат наредени двойки.
`String.next_codepoint/1` връща *tuple* с първи елемент, първият *codepoint* на низа, а втори остатъка.
Ако низът е празен, резултатът ще е атома `nil`.
`String.next_grapheme/1` се справя по-добре с комбинирани *codepoint*-и, но иначе има подобно поведение.
Просто работи с графеми.
Третата функция от примера - `String.next_grapheme_size/1` е по интересна, защото връща големината на следващата графема в байтове.
Тъй като `й` е съставено от два *codepoint*-a от по два байта - имаме `4` байта.
Само по себе си 'краткото' е **2** байта.

Има две функции за взимане на парче от низ - `String.slice/2` и `String.slice/3`:

```elixir
poem = """
  Ще строим завод,
  огромен завод,
  със яки
        бетонни стени!
  Мъже и жени,
  народ,
  ще строим завод
  за живота!
"""

large_factory = String.slice(poem, 18..30)
#=> "огромен завод"
String.slice(poem, 18, 13)
#=> "огромен завод"
```

Едната взима отрязък по *range* от индекс до индекс, а другата от индекс с дължина.

## Обхождане на низове

Нека обходим низ и преброим срещането на буквата 'a' в него.

```elixir
defmodule ACounter do
  def count_it_with_slice(str) do
    count_with_slice(str, 0)
  end

  defp count_with_slice("", n), do: n
  defp count_with_slice(str, n) do
    the_next_n = next_n(String.starts_with?(str, "a"), n)
    count_with_slice(String.slice(str, 1..-1), the_next_n)
  end

  defp next_n(true, n), do: n + 1
  defp next_n(false, n), do: n
end
```

Използваме `String.starts_with?/2` за проверка на текущия първи символ в низа.
Ако е `"a"` увеличаваме `n`. Ако не е `"a"` продължаваме. Накрая върщаме `n`.
Решението работи, но за големи низове се усеща забавяне.
Ние го тествахме с низ с дължина **3_000_000+**:

```elixir
:timer.tc(ACounter, :count_it_with_slice, [str])
#=> {3176592, 589251}
```

Това е около **3.18** секунди, което за едно просто обхождане на низ е доста бавно, даже за низ с такава дължина.
Тук забавянето се получава от това, че на всяка итерация `String.slice/2` създава ново *sub-binary*, a за него се създава нов *match context*, когато използваме `String.starts_with?/2`.
Ще е по-добре ако в възможно *match context*-а да бъде преизползван.

Вътрешно `String.next_grapheme/1` ползва `binary_part` (`:binary.part`), която е достатъчно умна функция за да запазва *match context*-a и да отлага създаването на *sub-binaries*.
Нека да пробваме имплементация с тази функция:

```elixir
defmodule ACounter do
  def count_it_with_next_grapheme(str) do
    count_with_next_grapheme(str, 0)
  end

  defp count_with_next_grapheme("", n), do: n
  defp count_with_next_grapheme(str, n) do
    {next_grapheme, rest} = String.next_grapheme(str)
    count_with_next_grapheme(rest, next_n(next_grapheme == "a", n))
  end

  defp next_n(true, n), do: n + 1
  defp next_n(false, n), do: n
end
```

Да видим сега:

```elixir
:timer.tc(ACounter, :count_it_with_next_grapheme, [str])
#=> {792885, 589251}
```

Същият резултат, но сега за **0.8** секунди.
Около **4** пъти по-бърза имплементация!
Ако сме сигурни че няма да имаме обединени *codepoint*-и, можем да заменим `next_grapheme` с `next_codepoint`.
В този случай ще получим леко по-бързо поведение - около **0.5** секунди.

Можем да имплементираме същото това поведение с *binary patternt matching*:

```elixir
defmodule ACounter do
  def count_it_with_match(str) do
    count_with_match(str, 0)
  end

  defp count_with_match("", n), do: n

  defp count_with_match(<< c::utf8, rest::binary >>, n) when c == ?a do
    count_with_match(rest, n + 1)
  end

  defp count_with_match(<< _::utf8, rest::binary >>, n) do
    count_with_match(rest, n)
  end
end
```

Тази имплементация със същия низ отнема около **0.1** секунди.

Изводът от горните няколко примера е, че както винаги, ако можем да ползваме *pattern matching*, задължително трябва да се възползваме от това.
Сравнението, както и разделянето на низа със съпоставяне са много бързи.

Тази и миналата публикации наблегнаха на това как са имплементирани низовете в *Elixir*.
Показахме ви уловки, които можете да избегнете, както и различни операции.
В `String` модула има още доста функции, които можете да разгледате [тук](https://hexdocs.pm/elixir/String.html).

Ще се върнем на представянето на низове в паметта още веднъж, когато говорим за работа с процеси и уловки при *garbage collection*-а на процесите и на общата *binary* памет.