category: Домашно

tags:
- elixir
- homework

--------

# Задача 4

Четвъртата задача е за **9 точки**.

Крайният срок за предаване е 23:59 на 29.05.2017 г.

## Информация за задачата
Ако все още не сте се записали за получаване на домашни, можете да го направите, като изпратите имейл до адрес **register@elixir-lang.bg**, който трябва да отговаря на следните условия:
* В Относно (Subject) трябва да напишете **register**.
* В съдържанието на мейла трябва да напишете: име и фамилия, факултетен номер и да сложите линк към GitHub акаунта ви.

Ако сте записани, сте получили мейл с линк към заданието за новото домашно. То съдържа основната структура на проекта, в който трябва да работите. Свободни сте да създавате колкото искате допълнителни файлове, стига те да се намират в една от директориите `lib` или `test` (не пишете във вече съществуващите файлове в `test`). В репото ще има няколко базови теста, които да ви съпътстват при писането на задачата (Пускайте тестовете често, за да следите прогреса си). Често "къмитвайте" промените във вашето локално хранилише, също така често "пушвайте" към съответното в GitHub. Ще се постараем да върнем обратна връзка в рамките на един ден (ако не получите такава, може да пишете мейл на **course@elixir-lang.bg**, за да ни напомните). Всички работи, предадени в последния ден от срока за предаване, няма да получат обратна връзка, преди да бъдат окончателно проверени.

## Lambda

Задачата е проста (на пръв поглед). Трябва да направите **OTP application** `Lambda.Application`, която да стартира **Supervisor** `Lambda.Supervisor` с поне едно дете **GenServer** регистриран с име `Lambda.Server`. Сървърът трябва да регистрира изпълнението на функция с дадени аргументи и да кешира резултата от нея. При всяко следващо извикване резултатът не трябва да бъде пресмятан отново, ами да се връща кеширания резултат от сървъра.

За реализацията на функционалността можете да добавите още неща в **Супервижън дървото** на вашата апликация.

Прочетете цялата задача преди да почнете да пишете по нея, включително забележката в края на документа.

### Интерфейс на клиента

Част от следващите функции трябва да са синхронни, а останалите несинхронни. Ще ги бележа съответно с **[call]** и **[cast]**.

#### Lambda.Server.register(module, function) [cast]

Тази функция казва на сървъра, че от тук нататък може да очаква извикването на функцията `function` от модула `module`.

Тук точно като при `spawn/3` `module` и `function` са атоми.
Трябва да връща `:ok` при успешна регистрация.

#### Lambda.Server.run(module, function, arguments) [call]

Изпълнява функцията `function` от модула `module` с аргументи `arguments` (списък от аргументите на функцията)
  - ако функцията не е регистрирана преди това, отговора на сървъра трябва да е `{:error, "Function is not registered."}`
  - ако функцията се изпълнява за първи път с тези аргументи тя се изпълнява и се връща резултата във наредена двойка `{:ok, result}`
  - ако функцията се извиква повторно с тези аргументи, то се връща кешираната стойност отново във вида `{:ok, result}`

За да можем да разгледаме няколко примера нека си представим, че имаме следния модул:

```elixir
defmodule Test do
  def wait(time) when time > 0 and is_integer(time) do
    Process.wait(time)
    time
  end

  def crasher() do
    raise "I will crash you!"
  end
end
```

Тогава:

```elixir
iex> Lambda.Server.run(Test, :wait, [1])
{:error, "Function is not registered"}
iex> Lambda.Server.register(Test, :wait)
:ok
iex> Lambda.Server.run(Test, :wait, [1])
{:ok, 1}
iex> :timer.tc fn -> Lambda.Server.run(Test, :wait, [1000]) end
{1024969, {:ok, 1000}}
iex(4)> :timer.tc fn -> Lambda.Server.run(Test, :wait, [1000]) end
{36, {:ok, 1000}}
```

Както можете да видите от примера, вторият път, когато извикаме `Test.wait` с аргумента `1000`, функцията изобщо не се изпълнява, а просто се връща резултата от предходното изпълнение. Затова извикването завършва много по-бързо втория път (изчакването `Process.wait(1000)` не се случва).

Ако при извикване на `run` възникне грешка от каквото и да е естество, резултатът от функцията трябва да е `{:error, "Execution error"}`.

```elixir
iex> Lambda.Server.register(Test, :crasher)
:ok
iex> Lambda.Server.run(Test, :crash, [])
{:error, "Execution error"}
```

#### Lambda.Server.cached(module, function, arguments) [call]

Проверява дали има кеширан резултат от изпълнението на функцията `function` от модула `module` с аргументи `arguments`:
  - ако функцията не е регистрирана преди това, отговорът на сървъра трябва да е `{:error, "Function is not registered."}`
  - ако резултатът от функцията не е кеширан, за тези аргументи се връща `{:error, "Not cached"}`
  - ако резултатът е кеширан се връщa `{:ok, result}`


```elixir
iex> Lambda.Server.cached(Test, :wait, [1])
{:error, "Function is not registered"}
iex> Lambda.Server.register(Test, :wait)
:ok
iex> Lambda.Server.cached(Test, :wait, [1])
{:error, "Not cached"}
iex> Lambda.Server.run(Test, :wait, [1])
{:ok, 1}
iex> Lambda.Server.cached(Test, :wait, [1])
{:ok, 1}
```

#### Lambda.Server.clear_cache(module, function) [cast]

Функцията e асинхронна (не чака резултат от сървъра). Трябва да изчиства цялата кеширана информация за функцията `function` от модула `module`. Ако тя е била регистрирана преди това, то тя трябва да остане такава и след това. Ако не е била регистрирана не се случва нищо.


```elixir
iex> Lambda.Server.register(Test, :wait)
:ok
iex> Lambda.Server.run(Test, :wait, [1])
{:ok, 1}
iex> Lambda.Server.cached(Test, :wait, [1])
{:ok, 1}
iex> Lambda.Server.clear_cache(Test, :wait)
:ok
iex> Lambda.Server.cached(Test, :wait, [1])
{:error, "Not cached"}
```

#### Lambda.Server.clear_cache(modlue, function, arguments) [cast]

Функцията e асинхронна (не чака резултат от сървъра). Трябва да изчиства кешираната информация за функцията `function` от модула `module` при аргументи `arguments`.

```elixir
iex> Lambda.Server.register(Test, :wait)
:ok
iex> Lambda.Server.run(Test, :wait, [1])
{:ok, 1}
iex> Lambda.Server.run(Test, :wait, [2])
{:ok, 2}
iex> Lambda.Server.clear_cache(Test, :wait, [1])
:ok
iex> Lambda.Server.cached(Test, :wait, [1])
{:error, "Not cached"}
iex> Lambda.Server.cached(Test, :wait, [2])
{:ok, 2}
```

#### Lambda.Server.unregister(module, function) [cast]

Тази функция казва на сървъра, че не се очаква извикване на функцията `function` от модула `module` и че не е необходимо да се съхранява повече информацията за функцията.

```elixir
iex> Lambda.Server.register(Test, :wait)
:ok
iex> Lambda.Server.run(Test, :wait, [1])
{:ok, 1}
iex> Lambda.Server.unregister(Test, :wait)
:ok
iex> Lambda.Server.run(Test, :wait, [1])
{:error, "Function is not registered"}
iex> Lambda.Server.register(Test, :wait)
:ok
iex> Lambda.Server.cached(Test, :wait, [1])
{:error, "Not cached"}
```

### Забележка

За получаване на пълния брой точки **НЕ Е** необходимо сървърът да изпълнява повече от една заявка на наведнъж. Показаното по-долу е съвсем приемливо поведение:

```elixir
iex> Lambda.Server.register(Test, :wait)
:ok
iex> task =
...> fn(pid, seconds) ->
...>   result = Lambda.Server.run(Test, :wait, [seconds*1000])
...>   send(pid, result)
...> end
#Function<...>
iex> one_second_task = fn -> task(self(), 1) end
#Function<...>
iex> ten_second_task = fn -> task(self(), 10) end
#Function<...>
iex> Process.spawn(ten_second_task, [])
#PID<0.123.0>
iex> Process.spawn(one_second_task, [])
#PID<0.126.0>
iex> Process.sleep(8000)
:ok
iex> flush() # нито една от заявките не е върнала резултат
:ok          # защото втората чака първата да приключи
iex> Process.sleep(8000)
:ok
iex> flush()
{:ok, 10000} # вече и двете заявки са приключили
{:ok, 1000}  # двата отговора идват в реда в който са изпратени
:ok
```

Ако сървърът ви има възможност да обработва повече от една заявка ще получите бонус точки (**4.5т**). Тогава горният сегмент би изглеждал така:


```elixir
iex> Lambda.Server.register(Test, :wait)
:ok
iex> task =
...> fn(pid, seconds) ->
...>   result = Lambda.Server.run(Test, :wait, [seconds*1000])
...>   send(pid, result)
...> end
#Function<...>
iex> one_second_task = fn -> task(self(), 1) end
#Function<...>
iex> ten_second_task = fn -> task(self(), 10) end
#Function<...>
iex> Process.spawn(ten_second_task, [])
#PID<0.123.0>
iex> Process.spawn(one_second_task, [])
#PID<0.126.0>
iex> Process.sleep(8000)
:ok
iex> flush()
{:ok, 1000} # По-бързата заявка е приключила първа и връща отговор
:ok
iex> Process.sleep(8000)
:ok
iex> flush()
{:ok, 10000} # По-бавната заявка завършва също и връща отговор
:ok
```
