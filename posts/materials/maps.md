---
title_image_path: keys.jpg
category: Програма
created_at: 2019-03-06T19:00:00
tags:
  - elixir
  - map
  - pattern matching
  - dict
  - struct
  - module
---

# Речници (Maps)

В *Elixir* за асоциативен списък, речник или ключ-стойност структура могат да
се използват няколко типа.
Поради развитието на *Erlang* и на *Elixir* някои от тях отпаднаха като предпочитани опции и в момента се използват *Map*-ове.

## Малко история

В *Erlang* до версия **17** е нямало *Map*-ове.
В ролята на речници са били използвани съществуващите *keyword list*-ове, които са просто списъци от наредени двойки във вида `{атом, стойност}`.
Друго нещо, което е било ползвано (и се ползва и до днес) е *ETS*, *key-value* база данни в паметта, която е доста оптимизирана за конкурентен достъп и промени.
За речници с определена структора са били използвани *record* структурите, които са подобни на `struct`-овете в *C*. Всъщност тези *record* структури са просто именувани кортежи.
Можете да научите повече за тях [тук](http://erlang.org/doc/reference_manual/records.html) и [в документацията](https://hexdocs.pm/elixir/Record.html) на *Elixir*-ския им *wrapper* модул.

Във версия **17** се появяват речниците.
Те са имплементирани като два масива - единият пази ключовете, които са сортирани, а другият - стойностите.
При такава имплементация, речници с повече елементи стават бавни за писане и инициират множество копирания.
Поради тази причина *Elixir* се сдобива със собствена имплементация на речник, `HashDict`
и единен модул за работа с речници - `Dict`. Този речник е по-използваем
и много по бърз от вградения в *Erlang* когато съдържа множество елементи.

От версия **18** на *Erlang/OTP*, *Erlang* има нова имплементация за *Map*-ове с много елементи.
Тази имплементация е подобна на имплементациите ползвани в *Clojure* и *Scala*.
Използва се бърза за достъп и добра при засичане на колизии структура - *Hash Array Mapped Trie*.
Малко повече можете да прочетете за тази структура [тук](https://idea.popcount.org/2012-07-25-introduction-to-hamt).

Тази нова имплементация на *Map* идваща от *Erlang* е доста [по-бърза](https://gist.github.com/BinaryMuse/bb9f2cbf692e6cfa4841)
от `HashDict`, затова `HashDict` и `Dict` в момента са _deprecated_.

Ние ще си говорим за *Map* типа, който се създава с `%{}`.
Не използвайте `HashDict` и `Dict`.
Избягвайте да използвате *keyword lists* там, където е по-правилно да се използва *Map*.


## Създаване и достъп до Map

Създаваме речник така:

```elixir
%{} # Празен
#=> %{}

%{name: "Пешо"} # С ключ и стойност
#=> %{name: "Пешо"}
```

Ключове могат да бъдат всякакви типове, дори други *Map*-ове.
Най-често се използват атоми и низове.
Когато можем да използваме атоми е най-добре да използваме само атоми, защото те са оптимизиране за *pattern matching*-ване (все едно съпоставяме числа) и ако вече даден атом е дефиниран, той се пре-използва.
Все пак е лошо да конвертираме произволни низове към атоми, защото атомите никога не се зачистват от паметта.

Пример за *Map* с ключове атоми:

```elixir
pesho = %{
  name: "Пешо",
  age: 35,
  hobbies: {:drink, :smoke, :eurofootball},
  job: "шлосер"
}

# Четене на стойности:

pesho[:name]
#=> "Пешо"

pesho.name
#=> "Пешо"

Map.fetch(pesho, :name)
#=> {:ok, "Пешо"}

Map.fetch!(pesho, :name)
#=> "Пешо"

Map.get(pesho, :name)
#=> "Пешо"
```

Разликата между всички тези начини на достъп е в поведението им, когато ключът не съществува:

```elixir
pesho[:full_name] # В този случай връща nil ако го няма ключа
#=> nil

pesho.full_name # KeyError
#=> ** (KeyError) key :full_name not found

Map.fetch(pesho, :full_name) # :error ако го няма ключа
#=> :error

Map.fetch!(pesho, :full_name) # Подобно на pesho.full_name
#=> ** (KeyError) key :full_name not found

Map.get(pesho, :full_name) # Map.get работи с default стойност, която е nil, ако не е зададена
#=> nil
Map.get(pesho, :full_name, "Петър Петров")
#=> "Петър Петров"

Map.get_lazy(pesho, :full_name, fn -> "Петър Петров" end)
#=> "Петър Петров"

# Горната функция се ползва, ако стойността по подразбиране е скъпа за пресмятане.
```

* Използваме `Map.get*` ако искаме да имаме стойност по подразбиране.
* Използваме `Map.fetch!/2` или `.<атом>`, ако искаме да получим грешка при четене на несъществуващ ключ.
* Ако искаме да имаме проверка от типа `{:ok, <value>}` при успех или `:error` при липсващ ключ, ползваме `Map.fetch/2`.

Защо имаме `Map.get_lazy/3`? Защо `Map.get/3` не е дефиниран с гардове за проверка на стойността по подразбиране дали не е функция.
Отговорът е следният: ако имаме *Map* със стойности функции и искаме да имаме стойност по подразбиране някаква функция, не искаме тя да се изпълни.
Тя трябва да бъде върната като стойност по подразбиране. Именно поради случай като този имаме друго име за 'lazy' поведението.

Ако ключовете не са атоми, синтаксисът е малко по-различен:

```elixir
pesho = %{
  "name" => "Пешо",
  "age" => 35,
  "hobbies" => {:drink, :smoke, :eurofootball},
  "job" => "шлосер"
}
```

Достъпът `map.<atom>` също не работи в този случай:

```elixir
pesho["age"]
#=> 35

pesho.age
#=> ** (KeyError) key :age not found
pesho."age"
#=> ** (KeyError) key :age not found
```

## "Промяна" на Map

Премахване на ключове:

```elixir
Map.pop(pesho, :name)
#=> {"Пешо", %{age: 35, hobbies: {:drink, :smoke, :eurofootball}, job: "шлосер"}}

Map.pop(pesho, :full_name, "Петър Панов")
#=> {"Петър Панов", %{age: 35, hobbies: {:drink, :smoke, :eurofootball}, job: "шлосер", name: "Пешо"}}

Map.pop_lazy(pesho, :nick, fn -> "pe60" end)
#=> {"pe60", %{age: 35, hobbies: {:drink, :smoke, :eurofootball}, job: "шлосер", name: "Пешо"}}
```

С други думи `pop` функциите са много подобни на `get` функциите, но за разлика от тях връщат наредена 2-ка.
Първият елемент е този който е търсен или стойност по подразбиране, а вторият e нов *Map* без дадените ключ и стойност.

```elixir
Map.delete(pesho, :name)
#=> %{age: 35, hobbies: {:drink, :smoke, :eurofootball}, job: "шлосер"}

Map.delete(pesho, :full_name)
#=> %{age: 35, hobbies: {:drink, :smoke, :eurofootball}, job: "шлосер", name: "Пешо"}
```

Функцията `Map.delete/2` просто връща *Map* без указания ключ или оригиналния речник, ако ключът не съществува.
Функцията `Map.drop/2` приема списък от ключове, които да се "премахнат":

```elixir
Map.drop(pesho, [:hobbies, :job])
#=> %{age: 35, name: "Пешо"}
```

## "Промяна" и "добавяне" на стойности

Има много начини за "промяна" на речник (тоест да се генерира нов, с някаква разлика спрямо оригинала).
Няколко от тях:

```elixir
pesho = %{ name: "Пешо", age: 35 }
#=> %{age: 35, name: "Пешо"}

# Map.put/3 добавя дадена стойност за даден ключ, ако ключът съществува ресултатът е с променена стойност.
Map.put(pesho, :full_name, "Петрун Петрунов")
#=> %{age: 35, full_name: "Петрун Петрунов", name: "Пешо"}
```

Функциите `Map.put_new/3` и `Map.put_new_lazy/3` правят същото като `Map.put/3`,
с тази разлика, че ако указаният ключ съществува, речникът който бива върнат е същият, без променена стойност.

Интересен начин за промяна, но само на вече съществуващи в речника ключове е следният:

```elixir
pesho = %{
  name: "Пешо",
  age: 35,
  hobbies: {:drink, :smoke, :eurofootball},
  job: "шлосер"
}

%{pesho | hobbies: :none}
#=> %{age: 35, hobbies: :none, job: "шлосер", name: "Пешо"}

%{pesho | drink: :rakia} # Ако не съществува - грешка.
#=> ** (KeyError) key :drink not found in:
```

По този начин могат да се променят множество ключове,
а ако искаме да променим и едновременно добавим нови ключове, ползваме `Map.merge/2`:

```elixir
Map.merge(pesho, %{hobbies: :just_drinking, drink: :rakia})
#=> %{age: 35, drink: :rakia, hobbies: :just_drinking, job: "шлосер", name: "Пешо"}
```

В модула `Map` има още няколко функции за "промяна", които няма да разгледаме тук,
но са налични в [документацията](https://hexdocs.pm/elixir/Map.html).

## Речници и съпоставяне на образци

Речниците имат интересно поведение при съпоставяне.
При съпоставяне не е нужно лявата и дясната страна да съвпадат изцяло.
Важното е ключовете и стойностите от ляво да съвпадат с **под-множество** на тези от дясно.

```elixir
pesho = %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}
#=> %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}

%{name: x} = pesho
#=> %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}

x
#=> "Пешо"
```

Можем да направим проверка, че дадени ключове съществуват по следния начин:

```elixir
%{name: _, age: _} = pesho
#=> %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}

%{name: _, age: _, location: _} = pesho
#=> ** (MatchError) no match of right hand side value: %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}
```

*Pattern matching*-ът не може да се приложи на ключове:

```elixir
%{x => "Пешо"} = %{"name" => "Пешо"}
#=> ** (CompileError) illegal use of variable x inside map key match, maps can only match on existing variable by using ^x
```

Имаме същото поведение, когато речник е подаден като аргумент при извикването на функция:

```elixir
defmodule A do
  def f(%{name: name} = person) do
    IO.puts(name)

    person
  end
end

A.f(pesho)
#output: Пешо
#=> %{age: 35, drink: :rakia, hobbies: :just_drinking, name: "Пешо"}
```

В този пример, `name` стойността на подадения речник беше *match*-ната и използвана в тялото на функцията.

## Операции върху вложени речници

Нека имаме следния речник:

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
put_in(data, [:proboscidea, :elephantidae, :fictional], ["Jumbo"])
#=> %{
#=>   proboscidea: %{
#=>     elephantidae: %{
#=>       elephas: ["Asian Elephant", "Indian Elephant", "Sri Lankan Elephant"],
#=>       fictional: ["Jumbo"],
#=>       loxodonta: ["African bush elephant", "African forest elephant"]
#=>     },
#=>     mammutidae: %{ mammut: ["Mastodon"] }
#=>   }
#=> }

# Имаме същия резултат, ако ключът 'fictional' съществуваше:
put_in(data.proboscidea.elephantidae.fictional, ["Jumbo"])
```

По подобен начин можем да прочетем дълбоко вложена стойност:

```elixir
get_in(data, [:proboscidea, :elephantidae, :loxodonta])
#=> ["African bush elephant", "African forest elephant"]
```
