---
title_image_path: keys.jpg
category: Програма
created_at: 2017-03-24T02:01:21
tags:
  - elixir
  - map
  - pattern matching
  - dict
  - struct
  - module
---

# Речници и структури

Отново ще си говорим за структури от данни и модулите около тях.
В `Elixir` за асоциативен списък, речник или ключ-стойност структура могат да
се използват няколко неща. Поради развитието на `Erlang` и на `Elixir` някои
от тях отпаднаха като предпочитани опции и в момента се използват `Map`-ове.
С помощта на тези структури можем да създаваме нещо като свои собствени типове
в `Elixir`.

## Малко история

В `Erlang` до версия `17` няма `map`-ове. Има ги `keywor list`-ите, които са
просто списъци от наредени `2`-ки елементи във вида `{атом, стойност}`. Има и
`records`, които са подобни на структурите в `Elixir`, донякъде.

Във версия`17` се появяват `map`-овете.
Те са имплементирани като два масива - един ключове, сортирани, а другият - стойности.
Можете да се досетите че като имате повечко елементи те ще са бавни.
Поради тази причина `Elixir` си има собствена имплементация на речник - `HashDict`
и единен модул за работа с речници - `Dict`. Този еликсирски речник e по-използваем
и много по бърз от вградения в `Erlang` при множество елементи.

От версия `18` на `Erlang/OTP`, `Erlang` има нова имплементация за `map`-ове с много елементи.
Тази имплементация е подобна на имплементациите ползвани в `Clojure` и `Scala` да речем.
Използва бърза за достъп и добра при засичане на колизии структура - `Hash Array Mapped Trie`.
Малко повече може да прочетете за тази структура [тук](https://idea.popcount.org/2012-07-25-introduction-to-hamt).

Тази нова имплементация на `map` идваща от `Erlang` е доста [по-бърза](https://gist.github.com/BinaryMuse/bb9f2cbf692e6cfa4841)
от `HashDict`, затова `HashDict` и `Dict` в момента са _deprecated_.

Ние ще си говори за `Map`, която се създава с `%{}`. Не използвайте `HashDict` и `Dict`.
Избягвайте да изпозлвате `keyword lists` вместо `map`.

## Създаване и достъп до Map

Създаваме `map` така:

```elixir
iex> %{} # Празна
%{}
iex> %{name: "Пешо"} # С ключ и стойност
%{name: "Пешо"}
```

Ключове могат да бъдат всякакви типове, дори други `Map`-ове.
Ние ще използваме най-често атоми и низове. Когато можем да използваме атоми
е най-добре да използваме само атоми.

`Map` с ключове атоми:

```elixir
pesho = %{
  name: "Пешо",
  age: 35,
  hobbies: {:drink, :smoke, :xxx, :eurofootball},
  job: "шлосер"
}

# Четене на стойности

iex> pesho[:name]
"Пешо"
iex> pesho.name
"Пешо"
iex> Map.fetch(pesho, :name)
{:ok, "Пешо"}
iex> Map.fetch!(pesho, :name)
"Пешо"
iex> Map.get(pesho, :name)
"Пешо"
```

Ще си кажете - доста фунции, а пък правят едно и също. Разликата е в поведението
им, когато ключът не съществува:

```elixir
iex> pesho[:full_name] # В този случай връща nil ако го няма ключа
nil
iex> pesho.full_name # KeyError
** (KeyError) key :full_name not found

iex> Map.fetch(pesho, :full_name) # :error ако го няма ключа
:error
iex> Map.fetch!(pesho, :full_name) # Подобно на pesho.full_name
** (KeyError) key :full_name not found

iex> Map.get(pesho, :full_name) # Map.get работи с deafault стойност, която е nil, ако не е зададена
nil
iex> Map.get(pesho, :full_name, "Петър Петров")
"Петър Петров"

iex> Map.get_lazy(pesho, :full_name, fn -> "Петър Петров" end)
"Петър Петров"
# Ползва се, ако default стойността е скъпа за смятане.
```

Доста различни функции за четене на ключ. Подходящи са за различни ситуации.
Използваме `Map.get*` ако искаме да имаме стойност-по-подразбиране. Използваме `Map.fetch!` или `.<атом>`
ако искаме да получим грешка при четене на несъществуващ ключ.
Ако искаме да имаме проверка от типа `{:ok, <value>}` при успех или `:error` при липсващ ключ, ползваме
`Map.fetch`.

След лекцията, която направихме по тази статия излезе въпрос - защо имаме `Map.get_lazy/3`, защо не е просто `Map.get/3` дефиниран
с гардове да проверява стойността по подразбиране дали не е функция.
Замислете се, ако имате `Map` със стойности функции и искате да имате стойност по подразбиране някаква функция, не искате тя да се изпълни,
искате тя да бъде върната като стойност по подразбиране. Именно поради случай като този имаме друго име за 'lazy' поведението.

Ако ключовете не са `atom`-и, синтаксисът е малко по-различен:

```elixir
pesho = %{
  "name" => "Пешо",
  "age" => 35,
  "hobbies" => {:drink, :smoke, :xxx, :eurofootball},
  "job" => "шлосер"
}
```

Достъпът `map.<atom>` също не работи в този случай:

```elixir
iex> pesho["age"]
35
iex> pesho.age
** (KeyError) key :age not found
iex> pesho."age"
** (KeyError) key :age not found
```

## Промяна на Map

Премахване на ключове:

```elixir
iex> Map.pop(pesho, :name)
{"Пешо", %{age: 35, hobbies: {:drink, :smoke, :xxx, :eurofootball}, job: "шлосер"}}
iex> Map.pop(pesho, :full_name, "Петър Панов")
{"Петър Панов", %{age: 35, hobbies: {:drink, :smoke, :xxx, :eurofootball}, job: "шлосер", name: "Пешо"}}
iex> Map.pop_lazy(pesho, :nick, fn -> "PI4A" end)
{"PI4A", %{age: 35, hobbies: {:drink, :smoke, :xxx, :eurofootball}, job: "шлосер", name: "Пешо"}}
```

С други думи `pop` функциите са много подобни на `get`, но за разлика от `get` връщат наредена 2-ка.
Първият елемент е този който е търсен или стойност по подразбиране, а вторият - нов `map` без дадените ключ и стойност.

```elixir
iex> Map.delete(pesho, :name)
%{age: 35, hobbies: {:drink, :smoke, :xxx, :eurofootball}, job: "шлосер"}
iex> Map.delete(pesho, :full_name)
%{age: 35, hobbies: {:drink, :smoke, :xxx, :eurofootball}, job: "шлосер", name: "Пешо"}
```

Функцията `Map.delete/2` просто връща `map` без указания ключ или същия ако ключът не съществува.
Функцията `Map.drop/2` приема списък от ключове, които да се премахнат:

```elixir
iex> Map.drop(pesho, [:hobbies, :job])
%{age: 35, name: "Пешо"}
```

## Промяна и добавяне на стойности

Има много начини за `update` на `map` (тоест да се генерира нов, с някаква разлика спрямо оригинала).
Набързо няколко от тях:

```elixir
iex> pesho = %{ name: "Пешо", age: 35 }
%{age: 35, name: "Пешо"}

# Map.put/3 добавя дадена стойност за даден ключ, ако ключът съществува ресултатът е с  променена стойност.
iex> Map.put(pesho, :full_name, "Петрун Петрунов")
%{age: 35, full_name: "Петрун Петрунов", name: "Пешо"}
```

Функциите `Map.put_new/3` и `Map.put_new_lazy/3` правят същото като `Map.put/3`,
с тази разлика, че ако указаният ключ съществува, `Map`-а който бива върнат е същият, без променена стойност.

Интересн начин за промяна, но само на вече съществуващи в речника ключове е следният:

```elixir
pesho = %{
  name: "Пешо",
  age: 35,
  hobbies: {:drink, :smoke, :xxx, :eurofootball},
  job: "шлосер"
}

iex> %{pesho | hobbies: :none}
%{age: 35, hobbies: :none, job: "шлосер", name: "Пешо"}

iex> %{pesho | drink: :rakia} # Ако не съществува - грешка.
** (KeyError) key :drink not found in:
```

По този начин могат да се променят множество от ключове,
а ако искаме да променим и едновременно добавим нови ключове, ползваме `Map.merge/2`:

```elixir
iex> Map.merge(pesho, %{hobbies: :just_drinking, drink: :rakia})
%{age: 35, drink: :rakia, hobbies: :just_drinking, job: "шлосер", name: "Пешо"}
```

В модула `Map` има още няколко функции за `update`, които няма да разгледаме тук,
но са налични в [документацията](https://hexdocs.pm/elixir/Map.html).

## Речници и съпоставяне на образци

Речниците имат много интересно поведение при съпоставяне. Не трябва изцяло да съвпадат лявата и дясната страна.
Важното е ключовете и стойностите от ляво да съвпадат с под-множество на тези от дясно.

```elixir
iex> pesho = %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}
%{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}

iex> %{name: x} = pesho
%{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}
iex> x
"Пешо"
```

Можем да направим проверка че дадени ключове съществуват така:

```elixir
iex> %{name: _, age: _} = pesho
%{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}
iex> %{name: _, age: _, sex: _} = pesho
** (MatchError) no match of right hand side value: %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}
```

Pattern matching-a не може да се приложи на ключове:
```elixir
iex> %{x => "Пешо"} = %{"name" => "Пешо"}
** (CompileError) illegal use of variable x inside map key match, maps can only match on existing variable by using ^x
```

Удобно приложение е подаване на функции:
```elixir
defmodule A do
  def f(%{name: name} = person) do
    IO.puts(name)
    person
  end
end

iex> A.f(pesho)
Пешо
%{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}
```

Както виждате, прихванахме името от `map`-a и успяхме на го принтираме, както и
върнахме оригиналния `map`.

## Операции върху вложени речници

Нека имаме следния `Map`:

```elixir
data = %{
  proboscidea: %{
    elephantidae: %{
      elephas: ["Asian Elephant", "Indian Elephant", "Sri Lankan Elephant"],
      loxodonta: ["African bush elephant", "African forest elephant"]
    },
    mammutidae: %{
      mammut: ["Mastodon"]
    }
  }
}
```

Най-лесният начин да добавим нещо на по-дълбоко ниво е `Kernel.put_in/3`:

```elixir
iex> put_in(data, [:proboscidea, :elephantidae, :fictional], ["Jumbo"])
%{
  proboscidea: %{
    elephantidae: %{
      elephas: ["Asian Elephant", "Indian Elephant", "Sri Lankan Elephant"],
      fictional: ["Jumbo"],
      loxodonta: ["African bush elephant", "African forest elephant"]
    },
    mammutidae: %{ mammut: ["Mastodon"] }
  }
}

# Същият резултат, ако ключът 'fictional' съществуваше:
iex> put_in(data.proboscidea.elephantidae.fictional, ["Jumbo"])
```

По подобен начин можем да прочетем дълбоко вложена стойност:

```elixir
iex> get_in(data, [:proboscidea, :elephantidae, :loxodonta])
["African bush elephant", "African forest elephant"]
```

## Структури

Структурите всъщност са речници. Ограничени речници.
Ограничени по няколко признака:
1. Ключовете им са атоми. Задължително атоми.
2. Ключовете им са предефинирани.
3. Много свойства на речниците не са валидни за структури. Да речем достъп от сорта на `some_struct[:some_atom]` е невъзможен.

Структурите се дефинират в модул. Името на модула е името на структурата:

```elixir
defmodule Person do
  defstruct [:name, :age, location: "Far away", chldren: []]
end
```

Създадохме структура `Person` с четири полета.
Полетата `name` и `age` нямат зададена определена стойност по подразбиране, затова са `nil` по подразбиране.
Създаваме инстанция на структурата, подобно на както създаваме `map`-ове:

```elixir
iex> pesho = %Person{name: "Пешо", age: 35, location: "Горен Чвор"}
%Person{age: 35, chldren: [], location: "Горен Чвор", name: "Пешо"}
```

Всъщност лесно можем да видим какво представлява `pesho`:

```elixir
iex> inspect(pesho, structs: false)
"%{__struct__: Person, age: 35, chldren: [], location: \"Горен Чвор\", name: \"Пешо\"}"
```

Както виждаме, има скрит ключ `__struct__` със стойност името на структурата.
Другата разлика с динамичните речници е, че структурите не имплементират някои протоколи, които ще видим в следващата статия.
Хубава новина е, че `Map` модулът работи със структури:

```elixir
iex> Map.put(pesho, :name, "Стойчо")
%Person{age: 35, chldren: [], location: "Горен Чвор", name: "Стойчо"}
```

Операторът за обновяване също работи:

```elixir
iex> %{pesho | name: "Стойчо"}
%Person{age: 35, chldren: [], location: "Горен Чвор", name: "Стойчо"}
```

Също:

```elixir
iex> is_map(pesho)
true
iex> is_map(%{"key" => "value"})
true

iex> map_size(pesho)
5
iex> map_size(%{"ключ" => "стойност"})
1
```

Тези две `Kernel` функции проверяват съответно дали стойност е речник и колко ключ-стойности има даден речник.
Функцията `is_map` връща `true` и за структури. А функцията `map_size` връща `5`, макар `pesho` да има само `4` дефинирани ключа.
Това е така защото тя брои и скрития `__struct__` ключ.

Можем да съпоставяме структури с речници и други структури от същия тип:

```elixir
iex> %{name: x} = pesho
%Person{age: 35, chldren: [], location: "Горен Чвор", name: "Пешо"}
iex> x
"Пешо"

# Същото поведение:
iex> %Person{name: x} = pesho
%Person{age: 35, chldren: [], location: "Горен Чвор", name: "Пешо"}
iex> x
"Пешо"

#Но това не е валидно:
iex> %Person{} = %{}
** (MatchError) no match of right hand side value: %{}
```

Последното поведение е нормално, защото `Person` има `__struct__` ключ да речем, както
и стойности по подразбиране. Поради това никоя структура от ляво не може да се съпостави с речник от дясно.
Обратното е възможно.

## За какво и как да ползваме структури

Структурите се дефинират в модул с идеята че са нещо като тип дефиниран от нас.
В модула би трябвало да напишем специални функции, които да работят с този тип.
Така имаме на едно място дефиницията на типа и функциите за работа с него.

Пример е `MapSet`:

```elixir
iex> inspect(MapSet.new([2, 2, 3, 4]), structs: false)
"%{__struct__: MapSet, map: %{2 => true, 3 => true, 4 => true}}"
```

Това е структура. Вътрешно е представена с `map`.
За да има уникални стойности - да е множество, използва ключовете на този речник като свои стойности.
При ключовете на речник, както знаем, няма повторения, затова работи. И стойностите са просто `true`.

```elixir
iex> MapSet.union(MapSet.new([1, 2, 3]), MapSet.new([2, 3, 4]))
#MapSet<[1, 2, 3, 4]>
```

Какво споменахме самият модул `MapSet` дефинира набор от функции за работа със структурата `MapSet`.

Това не означава че можем да гледаме на структурите като класове и функциите в модулите им като на техни методи.
Не. Структурите не са класове. Elixir не е обектно-ориентиран език. Структурите са речници, имат предварително
дефинирана структура. Това е.

Да речем, ако създадем структура `Post`, която представлява блог пост, ние просто създаваме речник
с предварително дефинирани полета, представляващ пост. В модула му ще има функции за създаване на `Post` от
файл или от друг источник, и за някаква валидация - това е.
Няма нужда от сложни заимодейсвия между структури. Всичко e `immutable`.
Действията с които разполагаме са прости, малки функцийки, които композираме.
