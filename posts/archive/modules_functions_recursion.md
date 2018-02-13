---
title_image_path: why_elixir.jpg
category: Програма
author: Valentin Mihov
created_at: 2017-03-11T10:26:14
tags:
  - elixir
  - modules
  - functions
---

# Модули, функции и рекурсия

Организацията на кода в Elixir става чрез модули. Модулите просто групират множество функции, като обикновенно идеята е функциите в даден модул да извършват някаква обща работа. Например функциите в модула `List` предоставят различни операции със списъци, а в стандартния модул String, операции с низове.

Така и когато ние пишем програма на Elixir ще разбиваме функционалността на функции и ще ги групираме в модули.

## Дефиниране на модул

Нека да видим един прост пример за модул с една фунцкия:

```elixir
defmodule Times do
  def double(n) do
    n * 2
  end
end
```

Нека запишем горният код във файл `times.exs`, да пуснем iex и да компилираме и заредим модула:

```elixir
% iex
Erlang/OTP 19 [erts-8.2] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Interactive Elixir (1.4.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> c "times.exs"
[Times]
iex(2)> Times.double(10)
20
```

Както виждате вече можем да викаме функциите от този модул, като следваме синтаксиса `<име на модул>.<име на функция>`

Възможно е да имаме функции с едно и също име, но с различен брой параметри. Конвенцията е да се пише броят параметри на една функция след името за да се знае за коя точно функция става въпрос. Например горната функция е `double/1`. Ако имаме функция `double` с 3 параметъра, то тя е `double/3`. Тази конвенция се използва широко в Elixir света и ако четете документацията в https://hexdocs.pm ще я срещнете.

Този вид именуване се изпозлва и в самият език, за да се вземе референция към съществуваща функция и да се използва за композиция или параметър. Например:

```elixir
iex(4)> double_operation = &Times.double/1
&Times.double/1
iex(5)> double_operation.(5)
10
```

Реално можем да имаме 2 функции с еднакво име и различен брой параметри, които да правят 2 коренно различни неща. За езика това са 2 абсолютно различни функции, но на практика не трябва да правим това, защото може да е много объкващо за хората, които използват и четат кода ни.

Пример за функция с различен брой параметри би била `String.split`. Имаме `String.split/1`, която разделя низ по шпациите в него на смисък от поднизове и `String.split/3`, която разделя низа използвайки някакъв шаблон и позволява да се подадат допълнителни опции като трети параметър. Например:

```elixir
iex(1)> String.split("Elixir is awesome. It totally kicks bum")
["Elixir", "is", "awesome.", "It", "totally", "kicks", "bum"]
iex(2)> String.split("Elixir is awesome. It totally kicks bum", ".", parts: 2)
["Elixir is awesome", " It totally kicks bum"]
```

## do..end блокове с код

Може би забелязахте, че при дефинирането на модули и функции имаме така наречените `do..end` блокове. Реално тези блокове групират кода между тях и го подават като параметър на съответната функция (`defmodule` и `def`). От тази гледна точка, `defmodule` и `def` всъщност не са ключови думи в езика, а макроси (мислете за тях като вид функции). Дори можете да прочетете тяхната документация: https://hexdocs.pm/elixir/Kernel.html#def/2 и https://hexdocs.pm/elixir/Kernel.html#defmodule/2. Красотата на цялото нещо е че контролните структури също са макроси:

https://hexdocs.pm/elixir/Kernel.html#if/2
https://hexdocs.pm/elixir/Kernel.SpecialForms.html#case/2
https://hexdocs.pm/elixir/Kernel.SpecialForms.html#cond/1

Всъщност `do..end` блоковете са улеснен синктаксис на това кода да се подава като параметър на макроса, чрез `do:` (което също не е част от езика, а списък от ключови думи, за които ще говорим следващата лекция). Ето един пример как би изглеждала функцията `double` написана чрез `do:`:

```elixir
def double(n), do: n * 2
```

Ако имаме функция от няколко реда можем да я напишем така, но трябва да използваме скоби:

```elixir
def greeter(greeting, name), do: (
  IO.puts greeting
  IO.puts "How are you doing, #{name}?"
)
```

Тъй като горният синктаксис не е много удобен, много по-добре е да използваме `do...end`:

```elixir
def greeter(greeting, name) do
  IO.puts greeting
  IO.puts "How are you doing, #{name}?"
end
```

Реално можем да пишем всичко на един ред, но това не е препоръчително, тъй като кода ще е много нечетим:

```elixir
defmodule Times, do: def double(n), do: n * 2
```

Съкратеният синтаксис се използва само ако имаме кратка функция на един ред.

## Дефиниране на функции и pattern matching

Нещо много интересно, което се използва много често в Elixir е да се напишат няколко имплементации на една функция, като те се разделят чрез pattern matching. Така много елегантно се избягва писането на conditionals (if, unless, case, etc.). Например да видим как би изглеждала функцията за факториел:

```elixir
defmodule Factorial do
  def of(0), do: 1
  def of(n), do: n * of(n - 1)
end
```

И да видим как ще работи:

```elixir
iex(4)> Factorial.of(0)
1
iex(5)> Factorial.of(5)
120
```

Когато се извика функцията `Factorial.of/1` с някакви параметри, elixir ще се опита да pattern match-не параметрите към дефинициите на функцията в реда, в който са дефинирани и когато има съвпадение ще извика тази функция. Затова и реда на дефиниране на функциите е важен. Например така написано:

```elixir
defmodule Factorial do
  def of(n), do: n * of(n - 1)
  def of(0), do: 1
end
```

Elixir ще ни даде предупреждение:

```
warning: this clause cannot match because a previous clause at line 7 always matches
  iex:8
```

Което казва, че втората дефиниция на функцията `of/1` няма да бъде никога извикана, тъй като първата дефиниция ще се изпълни винаги преди нея. Когато компилирате кода си винаги гледайте какви преупреждения дава компилатора, тъй като много често те разкриват грешки в кода.

Този вид описание на логиката е доста приятен, тъй като ни дава възможност да я дефинираме, започвайки от простите случаи и вървейки към сложните, като използваме изолирани парчета логика. Ето например как би изглеждала дефиницията на редицата на фибоначи:

```elixir
defmodule Fibonachi do
  def of(1), do: 1
  def of(2), do: 1
  def of(n), do: of(n - 1) + of(n - 2)
end
```

Така написан кода е доста четим, тъй като лесно виждаме тривиалните случай и накрая се намира и общата логика. Ще се учудите колко много неща, за който сте писали if клаузи до сега могат да се опишат по този начин със малки функции.

## Guard клаузи при дефиниране на функции

Видяхме как Elixir може да pattern match-ва аргументите, които подаваме на функциите за да изпълни определа дефиниция на функция. Понякога обаче това не е достатъчно и имаме нужда да тестваме аргументите за техният тип или вид на техната стойност. В тези случаи на помощ идват guard клаузите. Те се дефинират със `when` когато дефинираме функциите. Например:

```elixir
defmodule Guard do
  def what_is(x) when is_number(x) do
    IO.puts "#{x} is a number"
  end

  def what_is(x) when is_list(x) do
    IO.puts "#{inspect(x)} is a list"
  end

  def what_is(x) when is_atom(x) do
    IO.puts "#{x} is an atom"
  end
end
```

Нека да пробваме горния код:

```elixir
iex(2)> Guard.what_is(42)
42 is a number
:ok
iex(3)> Guard.what_is(:cat)
cat is an atom
:ok
iex(4)> Guard.what_is([1,2,3])
[1, 2, 3] is a list
:ok
```

Guard клаузите могат да проверяват и за други свойства на аргументите. Например можем да проверяваме дали числата подавани на редицата на фибоначи са по-големи от нула:

```elixir
defmodule Fibonachi do
  def of(1), do: 1
  def of(2), do: 1
  def of(n) when is_number(n) and n > 0, do: of(n - 1) + of(n - 2)
end
```

Така ако опитаме да извикаме функцията с отрицателно число ще получим грешка, че такава функция не съществува:

```elixir
iex(6)> Fibonachi.of(-5)
** (FunctionClauseError) no function clause matching in Fibonachi.of/1
    iex:6: Fibonachi.of(-5)
```

Проверката за число също ни помага когато се опитваме да викаме функцията с нещо различно от число:

```elixir
iex(7)> Fibonachi.of("5")
** (FunctionClauseError) no function clause matching in Fibonachi.of/1
    iex:7: Fibonachi.of("5")
```

Този вид проверки ни позволяват да правим хитрини - например функции, които обработват различни видове данни. Можем да имаме функция, която приема както едно число, така и списък от числа:

```elixir
defmodule Accumulator do
  def sum(n) when is_number(n), do: sum([n])
  def sum(list) when is_list(list), do: Enum.reduce(list, 0, &Kernel.+/2)
end
```

Както виждате имаме обща имплементация за списък и частна имплементация за едно число, която се свежда до викане на същата функция със списък от числа:

```elixir
iex(10)> Accumulator.sum(42)
42
iex(11)> Accumulator.sum([1,2,3,4,5,6])
21
```

Видовете проверки, които могат да се извършват в guard клауза са ограничени и списък от тях можете да видите тук: http://elixir-lang.org/getting-started/case-cond-and-if.html#expressions-in-guard-clauses

## Параметри със стойности по подразбиране

Elixir поддържа дефиниране на стойности по подразбиране на аргументите на функциите. Това става със синтаксиса `param \\ value`. Попълването на аргументите става от ляво на дясно, като първо се попълват стойностите на параметрите без стойности по подразбиране. Ето един пример:

```elixir
defmodule Example do
  def func(p1, p2 \\ 2, p3 \\ 3, p4) do
    IO.inspect [p1, p2, p3, p4]
  end
end

Example.func("a", "b") # => ["a", 2, 3, "b"]
Example.func("a", "b", "c") # => ["a", "b", 3, "c"]
Example.func("a", "b", "c", "d") # => ["a", "b", "c", "d"]
```

Горният пример е доста изкуствен. На практика параметрите със стойности по подразбиране са много удобни за аргументи, които държат някакъв вид допълнителни опции. Например функцията `String.split/3`: https://hexdocs.pm/elixir/String.html#split/3, която може да се викне с допълнителни опции:

```elixir
String.split("mississippi", "i") # => ["m", "ss", "ss", "pp", ""]
String.split("mississippi", "i", parts: 3) # => ["m", "ss", "ssippi"]
```

## Private функции

До тук винаги когато дефинирахме функции в модул, тези функции бяха достъпни за ползване от външния свят. Понякога обаче имаме нужда да дефинираме помощни функции, които да не са достъпни извън модула и ни трябват за да имплементираме някаква по-сложна функционалност. Това става като дефинираме функциите със `defp`, вместо `def`. Например ако искаме да решим [fizzbuzz проблема](https://en.wikipedia.org/wiki/Fizz_buzz):

```elixir
defmodule FizzBuzz do
  def of(n), do: Enum.map(1..n, &number_value/1)

  defp number_value(n) when rem(n, 3) == 0 and rem(n, 5) == 0, do: "Fizz Buzz"
  defp number_value(n) when rem(n, 3) == 0, do: "Fizz"
  defp number_value(n) when rem(n, 5) == 0, do: "Buzz"
  defp number_value(n), do: n
end

FizzBuzz.of(20) # => [1, 2, "Fizz", 4, "Buzz", "Fizz", 7, 8, "Fizz", "Buzz", 11, "Fizz", 13, 14, "Fizz Buzz", 16, 17, "Fizz", 19, "Buzz"]
```

## Pipe оператора - `|>`

Тук стигаме до един доста интересен оператор, който отначало ще ви се стори странен, но след като свикнете с него ще ви липсва адски много в другите програмни езици: pipe операторът или `|>`. Pipe операторът взема стойноста от ляво и го вмъква като първи параметър на функцията от дясно. Най-лесно ще е да видим малко примери. Например нека да видим как би изглеждала логика, която трябва да вземе низ със стойности разделени със запетаи, да го направи с главни букви и да замени запетаите със интервали:

```elixir
iex(30)> Enum.join(String.split(String.upcase("name,sex,location"), ","), " ")
"NAME SEX LOCATION"
```

Това е доста трудно за четене тъй като натурално започваме да четем кода от ляво на дясно, но реално той се изпълнява от дясно на ляво, тъй като първо ще се изпълни най-вътрешната функция. Като се замислим, горната операция реално е трансформация на една стойност през няколко етапа. Ето как ще изглежда кода написан със pipe оператора:

```elixir
iex(33)> "name,sex,location" |> String.upcase |> String.split(",") |> Enum.join(" ")
"NAME SEX LOCATION"
```

Доста по-четимо, не мислите ли? А сега си представете, че горната логика трябва да поддържа низове с интервали в тях, т.е. "name, sex, location" трябва да стане на "NAME SEX LOCATION". Горната логика ще направи следното:

```elixir
iex(34)> "name, sex, location" |> String.upcase |> String.split(",") |> Enum.join(" ")
"NAME  SEX  LOCATION"
```

Ако използваме имплементацията без pipe оператора би било доста трудно да преценим къде точно трябва да вмъкнем един `String.strip`, но като имаме кода написан като няколко трансформации, не е трудно да се види, че това трябва да е преди `join`-а:

```elixir
iex(35)> "name, sex, location" |> String.upcase |> String.split(",") |> Enum.map(&String.strip/1) |> Enum.join(" ")
"NAME SEX LOCATION"
```

Когато пишем подобни конструкции обикновенно е доста удобно да разположим отделните стадии на отделен ред:

```elixir
defmodule CsvUtils do
  def upcase_space_transform(csv_line) do
    csv_line
    |> String.upcase
    |> String.split(",")
    |> Enum.map(&String.strip/1)
    |> Enum.join(" ")
  end
end

CsvUtils.upcase_space_transform("name, sex, location") # => "NAME SEX LOCATION"
```

Когато пишете elixir код е препоръчително да използвате pipe оператора колкото може повече. Ще видите, че в много случаи можете да имплементирате идеите си като трансформации на няколко фази и това ще направи кода ви съставен от множество малки функции, които изпълняват конкретна и добре дефинирана цел. Ако разгледате и някой известни elixir библиотеки, като Ecto ще видите, че там pipe оператора се използва за много неща, включително и за дефиниране на SQL транзакции: https://hexdocs.pm/ecto/Ecto.Multi.html#module-example

## Връзки между модули

Когато започнете да разделяте приложенията си на отделни модули, много често се налага тези модули да викат функции един на друг. Също понякога е удачно да влагате модулите един в друг (нещо наричано в други езици namespaces). Реално можете да влагате модулите колко си искате:

```elixir
defmodule Outer do
  defmodule Inner do
    def inner_func do
      "hello world"
    end
  end

  def outer_func do
    "Greeting from the inner func: #{Outer.Inner.inner_func}"
  end
end

Outer.outer_func # => "Greeting from the inner func: hello world"
Outer.Inner.inner_func # => "hello world"
```

Реално влагането на модули единствено добавя името на родителският модул към името на модула. Горният код е еквивалентен на:

```elixir
defmodule Outer.Inner do
  def inner_func do
    "hello world"
  end
end

defmodule Outer do
  def outer_func do
    "Greeting from the inner func: #{Outer.Inner.inner_func}"
  end
end
```

Няма никаква специална зависимост между горните модули. Реално именуването им показва на читателя, че те са свързани по някакъв начин, но за самият език това са просто два модула с различни имена.

Elixir предоставя някой улеснения, когато работим с модули и използваме функции от един модул в друг модул.

### import

import прави всички публични функции от подадения модул достъпни за ползване в текущият модул без да има нужда да се указва името на модула, в който живеят. Например:

```elixir
defmodule CsvUtils do
  import String

  def upcase_space_transform(csv_line) do
    csv_line
    |> upcase
    |> split(",")
    |> Enum.map(&strip/1)
    |> Enum.join(" ")
  end
end
```

Както виждате, когато import-нем модула String в нашия модул, всички функции в него са достъпни без да има нужда да се prefix-ват със `String.`. Функцията import позволява да се зададе и списък от функции, които да се import-нат в модула:

```elixir
defmodule CsvUtils do
  import String, only: [upcase: 1, split: 2, strip: 1]

  def upcase_space_transform(csv_line) do
    csv_line
    |> upcase
    |> split(",")
    |> Enum.map(&strip/1)
    |> Enum.join(" ")
  end
end
```

Също е възможно да се import-не всичко *с изключение* на списък от функции:

```elixir
defmodule CsvUtils do
  import String, except: [capitalize: 1]

  def upcase_space_transform(csv_line) do
    csv_line
    |> upcase
    |> split(",")
    |> Enum.map(&strip/1)
    |> Enum.join(" ")
  end
end
```

Можете да прочетете повече за опциите на `import` в документацията: https://hexdocs.pm/elixir/Kernel.SpecialForms.html#import/2

### alias

`alias` позволява да се 'преименува' модул под ново име, за да се пише по-малко когато се използва в рамките на нишия модул. Пример:

```elixir
defmodule Outer.Inner do
  def inner_func do
    "hello world"
  end
end

defmodule Outer do
  alias Outer.Inner, as: Inner

  def outer_func do
    "Greeting from the inner func: #{Inner.inner_func}"
  end
end
```

Както виждате с горния `alias` можем да използваме модула `Outer.Inner` само чрез `Inner`. В горният пример можем да не пишем `as: Inner`, тъй като по подразбиране `alias` ще използва името след последната точка.

### require

`require` указва, че имаме нужда от даден модул и той трябва да бъде компилиран и зареден. Обикновенно не е нужно да използваме `require`. Единственото изключение е ако използваме макроси от някой модул. Няма да задълбаваме сега какво са макросите. Достатъчно е да знаете, че ако има нужда да използвате require, компилаторът ще ви уведоми за това.

## Модулни атрибути

Когато дефинираме модули понякога е удобно да използваме константи за стойности, които се използват многократно в кода ни. За целта можем да използваме модулни атрибути. Това са стойности, които се изчисляват по време на компилация и не могат да бъдат променяни (подобни на константите в други езици). Например:

```elixir
defmodule Greeter do
  @standard_greeting "Hello, stranger!"

  def greet(nil) do
    IO.puts @standard_greeting
  end

  def greet(name) do
    IO.puts "Hello, #{name}!"
  end
end
```

Важно е да се знае, че стойностите на модулните атрибути се смятат по време на компилация, т.е. този код:

```elixir
defmodule Greeter do
  @standard_time Time.utc_now

  def greet(name) do
    IO.puts "Hello, #{name}! The time is #{@standard_time}"
  end
end
```

винаги ще вади едно време - времето когато кода е бил компилиран.

## Имена на модули и използване на Erlang модули

Тъй като Elixir работи във виртуалната машина на Erlang можем да използваме модули от Erlang. Това става като референцираме модулите като атоми. Например в Erlang има модул за генериране на случайни числа `rand`, който можем да използваме ето така:

```elixir
:rand.uniform(100) # => 98
:rand.uniform(100) # => 41
```

Можете да разгледате документацията на `rand` тук: http://erlang.org/doc/man/rand.html. На същия сайт можете да разгледате и какви други стандартни модули има в Erlang. Друг доста полезен модул позволява да се тества бързината на код:

```elixir
defmodule Fibonachi do
  def of(1), do: 1
  def of(2), do: 1
  def of(n), do: of(n - 1) + of(n - 2)
end

{time, value} = :timer.tc(&Fibonachi.of/1, [40])
IO.puts "Function took #{time} microseconds to run" # => Function took 3687607 microseconds to run
```

Повече информация можете да прочетете на http://erlang.org/doc/man/timer.html
