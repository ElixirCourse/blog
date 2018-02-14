---
title_image_path: blueprints.jpg
category: Програма
created_at: 2017-03-29T18:27:02
tags:
  - elixir
  - map
  - protocol
  - dict
  - struct
  - module
  - emumerable
---

# Протоколи

Протоколите са начин за постигане на _полиморфизъм_ в `Elixir`.
Те ни предоставят механизъм, чрез който вече съществуващо поведение може да се имплементира за нов тип от данни.
Използвайки протоколи можем да си построим библиотека, която да бъде разширена
от този, който я ползва.

Протоколите в `Elixir` са имплементирани върху нещо наречено _поведения_ (`Behaviors`).
Те са специфични за езика, за разлика от тези _поведения_, които идват от `Erlang`.

## Дефиниране на протокол

Протокол може да се дефинира с макрото `defprotocol`.
В тази статия, ще дефинираме протокол за преобразуване на тип в `JSON` форма:

```elixir
defprotocol JSON do
  @doc "Converts the given data to its JSON representation"
  def encode(data)
end
```

Дефинирахме протокол `JSON`, той декларира функция `JSON.encode/1`, която
преобразува данни в някаква валидна `JSON` форма.
Например за атома `nil`, трябва да получим `"null"`.

Нека да пробваме:

```elixir
iex> JSON.encode(nil)
** (Protocol.UndefinedError) protocol JSON not implemented for nil
json_protocol.ex:1: JSON.impl_for!/1
json_protocol.ex:3: JSON.encode/1
```

Тази грешка получаваме, защото не сме имплементирари протокола за `nil`.

## Имплементиране на протокол

Протокол се имплементира с макрото `defimpl`. Нека имплементираме `JSON` за атоми:

```elixir
defimpl JSON, for: Atom do
  def encode(true), do: "true"
  def encode(false), do: "false"
  def encode(nil), do: "null"

  def encode(atom) do
    JSON.encode(Atom.to_string(atom))
  end
end
```

Тук разписваме `JSON` протокола за `Atom` данни. Тъй като в
`JSON` формата `"null"`, `"true"` и `"false"` са валидни типове, ние ги връщаме като низ съответно за `nil`, `true` и `false`.
Всички други атоми, най-лелсно могат да се възприемат като низове в `JSON`.

```elixir
iex> JSON.encode(true)
"true"
iex> JSON.encode(false)
"false"
iex> JSON.encode(nil)
"null"
iex> JSON.encode(:name)
** (Protocol.UndefinedError) protocol JSON not implemented for "name"
    json_protocol.ex:1: JSON.impl_for!/1
    json_protocol.ex:3: JSON.encode/1
```

Това е нормално поведение, имаме имплементация за `nil` и булевите стойности, но
не и за низове. Нека имплементираме протокола за низове.
В `Elixir`, това са `binary` структури както знаем.

```elixir
defimpl JSON, for: BitString do
  def encode(<< >>), do: ~s("")
  def encode(str) do
    cond do
      String.valid?(str) -> ~s("#{str}")
      true -> JSON.encode(bitstring_to_list(str))
    end
  end

  defp bitstring_to_list(binary) when is_binary(binary) do
    list_of_bytes(binary, [])
  end

  defp bitstring_to_list(bits), do: list_of_bits(bits, [])

  defp list_of_bytes(<<>>, list), do: list |> Enum.reverse
  defp list_of_bytes(<< x, rest::binary >>, list) do
    list_of_bytes(rest, [x | list])
  end

  defp list_of_bits(<<>>, list), do: list |> Enum.reverse
  defp list_of_bits(<< x::1, rest::bits >>, list) do
    list_of_bits(rest, [x | list])
  end
end
```

За този случай се използва за тип `BitString`. Когато `str` е валиден низ,
всичко е лесно, просто слагаме стойността му в кавички (`""`). Използваме сигила `~s` при
тези дефиниции, за да не ескейпваме кавичките. Когато обаче `str` не е валиден низ,
го трансформираме в списък от байтове, ако е `binary` и в списък от битове ако броят
на битовете му не е кратен на `8`.

```elixir
iex> JSON.encode(:name)
"\"name\""
iex> JSON.encode("")
"\"\""
iex> JSON.encode("some")
"\"some\""
iex> JSON.encode(<< 200, 201 >>)
** (Protocol.UndefinedError) protocol JSON not implemented for [200, 201]
    json_protocol.ex:1: JSON.impl_for!/1
    json_protocol.ex:3: JSON.encode/1
```

Работи както трябва - нормално е да се оплаква че протоколът не е имплементиран за
списък. Нека го имплементираме.

```elixir
defimpl JSON, for: List do
  def encode(list) do
    "[#{list |> Enum.map(&JSON.encode/1) |> Enum.join(", ")}]"
  end
end
```

Имплементацията е доста проста - всеки от елементире на списъка е конвертиран
към `JSON`, след това са събрани в `[]` и разделени със запетая. Да видим дали работи:

```elixir
iex> JSON.encode([nil, true, false])
"[null, true, false]"
iex> JSON.encode(<< 200, 201 >>)
** (Protocol.UndefinedError) protocol JSON not implemented for 200
             json_protocol.ex:1: JSON.impl_for!/1
             json_protocol.ex:3: JSON.encode/1
```

Работи за списък от вече имплементирани типове данни.
Но за списък от цели числа - байтовете, не работи. Трябва да го имплементираме
за цели числа:

```elixir
defimpl JSON, for: Integer do
  def encode(n), do: n
end
```

За цели числа е наистина лесно - просто връщаме числото. Сега следното работи:

```elixir
iex> JSON.encode(<< 200, 201 >>)
"[200, 201]"
```

Последното, което остана е да го имплементираме за `Map`. Този тип данни ще са
`JSON` обекти от типа `{ "key": value }`:

```elixir
defimpl JSON, for: Map do
  def encode(map) do
    "{#{map |> Enum.map(&encode_pair/1) |> Enum.join(", ")}}"
  end

  defp encode_pair(pair) do
    {key, value} = pair

    "#{JSON.encode(to_string(key))}: #{JSON.encode(value)}"
  end
end
```

Отново проста имплентация: за всяка двойка ключ и стойност, превръщаме ключа
в низ и го трансформираме във валиден `JSON` низ, трансформираме стойността в `JSON` и
ги конкатенираме с `:` между тях -> `"key": value`. Всички такива двойки обединяваме
в един низ със запетайки между тях. Този низ слагаме в _'къдрави'_ скоби.

```elixir
iex> data = %{
  name: "Pesho",
  age: 43,
  likes: [:drinking, "eating shopska salad", "да гледа мачове"]
}
iex> IO.puts JSON.encode(data)
{"age": 43, "likes": ["drinking", "eating shopska salad", "да гледа мачове"], "name": "Pesho"}
```

или

```json
{
  "age": 43,
  "likes": [
    "drinking",
    "eating shopska salad",
    "да гледа мачове"
  ],
  "name": "Pesho"
}
```

## Структури и протоколи

Както знаем структурите са просто именовани `Map`-ове. От това би трябвало да
следва, че ако имплементираме протокол за `Map`, той ще е може да се прилага
за структури.

```elixir
defmodule Man do
  defstruct [:name, :age, :likes]
end

kosta = %Man{
  name: "Коста",
  age: 54,
  likes: ["Турбо фолк", "Телевизия", "да гледа мачове"]
}
JSON.encode(kosta)
** (Protocol.UndefinedError) protocol JSON not implemented for %Man{...}
    json_protocol.ex:1: JSON.impl_for!/1
    json_protocol.ex:3: JSON.encode/1
```

Всяка структура е като нов тип. Ако искаме да имплементира протокол ще трябва
да го направим ръчно или да използваме имплементацията за всички случаи.

## Покриване на всички случаи

Вградените типове за които можем да имплементираме протокол са:
* `Atom`
* `BitString`
* `Float`
* `Function`
* `Integer`
* `List`
* `Map`
* `PID`
* `Port`
* `Reference`
* `Tuple`

Има и начин да имплементираме протокол за всички случаи за които не е имплементиран,
използвайки `Any`:

```elixir
defimpl JSON, for: Any do
  def encode(_), do: "null"
end
```

Казваме че за неизвестни типове ще връщаме `null` като `JSON`. Нека да пробваме
с `kosta`:

```elixir
JSON.encode(kosta)
** (Protocol.UndefinedError) protocol JSON not implemented for %Man{...}
    json_protocol.ex:1: JSON.impl_for!/1
    json_protocol.ex:3: JSON.encode/1
```

Същото поведение.

Това е така, защото за да използваме имплементацията за `Any`, трябва да го обявим
при дефиниране на структурата:

```elixir
defmodule Man do
  @derive JSON
  defstruct [:name, :age, :likes]
end

nikodim = %Man{
  name: "Никодим",
  age: 15,
  likes: ["Порно", "GTA V"]
}
JSON.encode(nikodim)
"null"
```

Сега работи с `Any` имплементацията. С тази директива - `@derive`, казваме на
`Man` да имплементира `JSON`, използвайки `Any` имплементацията.

Ако пък не искаме да обявяваме `@derive JSON` за всяка имплементация, можем да
модифицираме самия протокол, така че да се ползва `Any` имплементацията за всеки
тип, който не имплементира протокола.

```elixir
defprotocol JSON do
  @fallback_to_any true

  @doc "Converts the given data to its JSON representation"
  def encode(data)
end
```

Така за тип като `Tuple`, за който протоколът `JSON` не е имплементиран ще получим
имплементацията за `Any`:

```elixir
iex> JSON.encode({:ok, :KO})
"null"
```

## Протоколи идващи с езика

Има няколко протокола, които идват с езика.
Можем да ги видим използвайки `Protocol.extract_protocols/1`.
Модулът `Protocol` съдържа методи за работа с протоколи, ще разгледаме някои от
тях. Нека започнем с този. Нека да го накараме да ни върне списък с вградените протоколи:

```elixir
iex> path = :code.lib_dir(:elixir, :ebin)
iex> Protocol.extract_protocols([path])
[Collectable, Inspect, String.Chars, List.Chars, Enumerable]
```

Използвахме `code:lib_dir/2` да излечем пътя до директорията, съдържаща вградените
модули на `Elixir`. Това е `Erlang` функция, затова получваме `charlist`. След това
с помощта на `Protocol.extract_protocols/1` виждаме кои са.

Да ги разгледаме:
* `Collectable` - това е протоколът, използван от `Enum.into`, за преобразуване на `Enummerable` в дадена колекция.
* `Inspect` - използва се за _pretty printing_.
* `String.Chars` - `Kernel.to_string/1` го използва.
* `List.Chars` - `Kernel.to_charlist/1` го използва.
* `Enumerable` - `Enum` методите очакват това, което им се подава да имплементира този протокол. Също и `Collectable` работи с имплементации на протокола.

Само `5`, но всички тях вече сме ползвали по един или друг начин.

Нека видим кои типове имплементират `Enumerable`. Това можем да направим, използвайки
`Protocol.extract_impls/2`. Този метод взима за първи аргумент протокола на който ще намерим имплементациите,
както и пътища. Ще използваме същия този път до вградените модули на `Elixir`:

```elixir
iex> Protocol.extract_impls(Enumerable, [path])
[Stream, List, Function, Map, IO.Stream, Range, MapSet, HashDict, HashSet,
 GenEvent.Stream, File.Stream]
```

Ето как виждаме кои типове имплементират даден протокол. Както виждате го няма
`BitString`, тоест низовете не са `Enumerable`. Това е нормално, `BitString` може да
е низ, но може да е просто `binary`, с невалидни за `Unicode` байтове или пък поредица от битове.
Ако е низ, то може да го обхождаме по различни начини - по графеми или по `codepoint`-и.

Нека все пак за упражнението да имплементираме `Enumerable` за низове.
Идеята е да ги обхождаме по графеми:

```elixir
defimpl Enumerable, for: BitString do
  def count(str), do: {:ok, String.length(str)}
  def member?(str, char), do: {:ok, String.contains?(str, char)}

  def reduce(_, {:halt, acc}, _fun), do: {:halted, acc}
  def reduce(str, {:suspend, acc}, fun) do
    {:suspended, acc, &reduce(str, &1, fun)}
  end

  def reduce("", {:cont, acc}, _fun), do: {:done, acc}
  def reduce(str, {:cont, acc}, fun) do
    {next, rest} = String.next_grapheme(str)
    reduce(rest, fun.(next, acc), fun)
  end
end
```

Протоколът `Enumerable` изисква имплементацията на `3` функции.

Функцията `Enumerable.count/1` се използва за намиране на големината на дадената енумерация.
Тази функция или връща `{:ok, <count>}` или `{:error, __MODULE__}`.
Първият тип резултат е ръчна имплементация, докато вторият казва на `Enum` да ползва `Enumerable.reduce/3`,
в подразбираща се имплементация с линейно време.
В случая, връщаме `String.length/1` на низа.

```elixir
iex> Enum.count("Далия")
5
```

Следващата функция, която трябва да бъде имплементирана е `Enumerable.member?/2`.
Тя проверява дали елемент е член на дадената енумерация. Възможни резултати са
`{:ok, boolean}` и `{:error, __MODULE__}`. Както за `count`, резултатът с `:error` ще използва `reduce`.
Документацията на тези два метода, препоръчва да ги имплементираме с `{:error, __MODULE__}`, ако нашата
имплементация не е по-добра от линейна имплементация.
За упражнението, ние ги имплементираме ръчно, но на практика в тези случаи е най-добре да се върне `{:error, __MODULE__}`
Тук ползваме `String.contains?/2`.

```elixir
iex> Enum.member?("Далия", "я")
true
```

Последният метод за имплементиране е `Enumerable.reduce/3`.
Сложен метод, който има предвид както `eager`, така и `lazy` енумерации.
Първият му аргумент е енумерацията. В нашия случай - низ.
Вторият е двойка от 'таг' и акумулираната стойност досега.
Третият е функцията - `reducer`, която смята акумулираната стойност.

Нека разгледаме нашите имплементации за низ и да обясним какво и защо се случва.

Първата имплементация е за случая, когато получим 'таг' `halt`.
Това означава че трябва да прекратим каквото правим.
Това би се случило да речем в случая, в който търсим нещо в една енумерация и го намерим.
В този случай трябва да върмен `{:halted, <акумулирана стойност>}`.
Правим точно това.

Втората имплементация е за случая, когато получим 'таг' `suspend`.
Това значи да прекратим изпълнението до тук, но това не означава че то е свършило.
Трябва да върнем тройката - `:suspended`, акумулираната стойност, която да се използва,
когато трябва да продължим и функция която да имплементира продължението.
Това е частично извикана функция, знаеща енумерацията и редуциращата си функция и очакваща
акумулираната стойност. Може да се използва `Enumerable.reduce/3` имплементация, както в примера.

Третата имплементация е за случая, когато получим 'таг' `cont` и празна енумерация.
Трябва да върнем двойката `{:done, <резултата-акумулираната-стойност>}`.
Това е дъното на рекурсията - в нашия случай при празен низ.

Четвъртата ни имплементация е имплементацията на `reduce` за произволна итерация.
Отново идва с тага `cont`. Тук използваме `String.next_grapheme/1` за да вземем
текущата първа графема и остатъка от низа. Извикваме рекурсивно `reduce` с остатъка
като енумерация, нова акумулирана стойност, получена от прилагането на редуциращата функция към текущата графема и текущата акумулирана стойност
и, разбира се, редуциращата функция.

С тези имплементации `Enum` функциите ще работят за низове.

```elixir
"Далия"
|> Enum.filter(fn
     c when c in ~w(а ъ о у е и) -> false
     _ -> true
   end)
|> Enum.join("")
"Для"
```

И така премахваме гласните с помощта на `Enum`.

## Консолидация

Извикването на функции на протоколи е по-бавно от извикването на функции на модули.
За протокол, при извикване, трябва да се мине през всички имплементации, докато не се намери
тази за типа данните-аргумент или докато не се изчерпат.

Така наречената консолидация превръща протокол и неговите имплементациии в прост
модул. При компилация с `mix`, това става автоматично (от версия `1.2`).

## Заключение

Протоколите дават възможност за имплементация на дадено поведение за множество
типове. Чудесни са за случая, когато предлагате собствена библиотека и желаете
този, който я ползва да може да я разшири за собствените си структури.
Добър пример е `JSON` протокола, който дефинирахме, даже има такава библиотека,
която може да се разшири именно защото ползва протокол - [poison](https://github.com/devinus/poison).
Имплементирайте и дефинирайте протоколи само когато се налага, не правете кода си
прекалено разхвърлян, ако знаете точните типове за които искате да работи.
Ако искате кодът ви да работи за безкрайно множество от типове - може да ползвате протоколи.

Полиморфизъм може да се постигне по няколко начина в `Elixir`.
Когато говорим за типове - най-простият е с множество версии на една функция в даден модул,
при ограничен брой имплементации. Най-често това ще е нашият случай.
