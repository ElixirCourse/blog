---
category: Домашно
created_at: 2018-05-16T10:03:25
author: Reductions
tags:
  - elixir
  - homework
---

# Задача 2

Задачата е за общо **15 точки**.

Крайният срок за предаване е 23:59 на 04.06.2018 г.

Ако имате въпрос по домашното пишете на: course@elixir-lang.bg

Ето и линк към заданието:

https://classroom.github.com/a/vXu6wVow

## Lambda

В предоставенто ви репо има начален скелет на **OTP application** с име `Lambda.Application`. За реализацията на функционалността описана по-долу можете да правите всякакви променяте в **Супервижън дървото** на тази апликация. Апликацията трябва да реализира функционалност за изпълнението на функция с дадени аргументи и да кешира резултата от нея. При всяко следващо извикване на кеширана функция резултатът не трябва да бъде пресмятан отново, а да се връща кеширания резултат.

Прочетете цялата задача преди да почнете да пишете по нея, включително забележката в края на документа.

### Интерфейс

Част от следващите функции трябва да са синхронни, а останалите асинхронни. Ще ги бележа съответно с **[call]** и **[cast]**.

#### Lambda.register(module, function) [cast]

Тази функция казва на апликация, че от тук нататък може да очаква извикването на функцията `function` от модула `module`.

Тук точно като при `spawn/3` `module` и `function` са атоми.

#### Lambda.run(module, function, arguments, timeout \\ 5_000) [call]

Изпълнява функцията `function` от модула `module` с аргументи `arguments` (списък от аргументите на функцията)
  - ако функцията не е регистрирана преди това, отговора на сървъра трябва да е `{:error, :not_registered}`
  - ако функцията се run-ва за първи път с тези аргументи тя се изпълнява и се връща резултата във наредена двойка `{:ok, result}`
  - ако функцията бива изпълнена повторно с тези аргументи, връщаме кешираната стойност отново във вида `{:ok, result}`

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
iex> Lambda.run(Test, :wait, [1])
{:error, :not_registered}
iex> Lambda.register(Test, :wait)
:ok
iex> Lambda.run(Test, :wait, [1])
{:ok, 1}
iex> :timer.tc fn -> Lambda.run(Test, :wait, [1000]) end
{1024969, {:ok, 1000}}
iex> :timer.tc fn -> Lambda.run(Test, :wait, [1000]) end
{36, {:ok, 1000}}
```

Както можете да видите от примера, вторият път, когато извикаме `Test.wait` с аргумента `1000`, функцията изобщо не се изпълнява, а просто се връща резултата от предходното изпълнение. Затова извикването завършва много по-бързо втория път (изчакването `Process.wait(1000)` не се случва).

Ако функцията не успее да завърши за `timeout` трябва да върнем :timeout (това не би трябвало да се случва в случай, че резултата е кеширан)

```elixir
iex> Lambda.run(test, :wait, [10_000])
:timeout
iex> Lambda.run(test, :wait, [10_000], 100_000)
{:ok, 10_000}
iex> Lambda.run(test, :wait, [10_000])
{:ok, 10_000}
```

Ако при извикване на `run` възникне грешка от каквото и да е естество, резултатът от функцията трябва да е `{:error, :execution_error}`.

```elixir
iex> Lambda.register(Test, :crasher)
:ok
iex> Lambda.run(Test, :crash, [])
{:error, :execution_error}
```

#### Lambda.fetch_cached(module, function, arguments) [call]

Проверява дали има кеширан резултат от изпълнението на функцията `function` от модула `module` с аргументи `arguments`:
  - ако функцията не е регистрирана преди това, отговорът трябва да е `{:error, :not_registered}`
  - ако резултатът от функцията не е кеширан, за тези аргументи се връща `{:error, :not_cached}`
  - ако резултатът е кеширан се връщa `{:ok, result}`


```elixir
iex> Lambda.fetch_cached(Test, :wait, [1])
{:error, :not_registered}
iex> Lambda.register(Test, :wait)
:ok
iex> Lambda.fetch_cached(Test, :wait, [1])
{:error, :not_cached}
iex> Lambda.run(Test, :wait, [1])
{:ok, 1}
iex> Lambda.fetch_cached(Test, :wait, [1])
{:ok, 1}
```

#### Lambda.clear_cache(module, function) [cast]

Трябва да изчиства цялата кеширана информация за функцията `function` от модула `module`. Ако тя е била регистрирана преди това, то тя трябва да остане такава и след това. Ако не е била регистрирана не се случва нищо.


```elixir
iex> Lambda.register(Test, :wait)
:ok
iex> Lambda.run(Test, :wait, [1])
{:ok, 1}
iex> Lambda.fetch_cached(Test, :wait, [1])
{:ok, 1}
iex> Lambda.clear_cache(Test, :wait)
:ok
iex> Lambda.fetch_cached(Test, :wait, [1])
{:error, :not_cached}
```

#### Lambda.clear_cache(modlue, function, arguments) [cast]

Трябва да изчиства кешираната информация за функцията `function` от модула `module` при аргументи `arguments`.

```elixir
iex> Lambda.register(Test, :wait)
:ok
iex> Lambda.run(Test, :wait, [1])
{:ok, 1}
iex> Lambda.run(Test, :wait, [2])
{:ok, 2}
iex> Lambda.clear_cache(Test, :wait, [1])
:ok
iex> Lambda.fetch_cached(Test, :wait, [1])
{:error, :not_cached}
iex> Lambda.fetch_cached(Test, :wait, [2])
{:ok, 2}
```

#### Lambda.unregister(module, function) [cast]

Тази функция казва на апликацията, че не се очаква извикване на функцията `function` от модула `module` и че не е необходимо да се съхранява повече информацията за функцията.

```elixir
iex> Lambda.register(Test, :wait)
:ok
iex> Lambda.run(Test, :wait, [1])
{:ok, 1}
iex> Lambda.unregister(Test, :wait)
:ok
iex> Lambda.run(Test, :wait, [1])
{:error, :not_registered}
iex> Lambda.register(Test, :wait)
:ok
iex> Lambda.fetch_cached(Test, :wait, [1])
{:error, :not_cached}
```

### Забележка

За получаване на пълния брой точки **Е НЕОБХОДИМО** апликацията ви да може да изпълнява повече от една функция по едно и също време. Показаното по-долу е правилното поведение:

```elixir
iex> Lambda.register(Test, :wait)
:ok
iex> task =
...> fn(pid, seconds) ->
...>   result = Lambda.run(Test, :wait, [seconds*1000], :infinity)
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
{:ok, 1000} # По-бързата заявка е приключила първа и връща отговор.
:ok
iex> Process.sleep(8000)
:ok
iex> flush()
{:ok, 10000} # По-бавната заявка завършва също и връща отговор.
:ok
```

Друго нещо за което бихме тествали е, че ако в близко време еднакви заявкa пристигне два пъти, то тя ще бъде изпълнена само веднъж.

```elixir
iex> defmodule Test do
...>   def wait(time) do
...>     IO.puts "Tra la la"
...>     Process.sleep(time)
...>     time
...> end
iex> Lambda.register(Test, :wait)
:ok
iex> task =
...> fn(pid, seconds) ->
...>   result = Lambda.run(Test, :wait, [seconds*1000], :infinity)
...>   send(pid, result)
...> end
#Function<...>
iex> three_second_task = fn -> task(self(), 3) end
#Function<...>
iex> Process.spawn(three_second_task, [])
#PID<0.123.0>
iex> Process.spawn(three_second_task, [])
#PID<0.126.0>
iex> Process.sleep(5000)
"Tra la la"
:ok
iex> flush()
{:ok, 3000} 
{:ok, 3000} # И двете заявки са върнали отговор, но функцията е изпълнена само веднъж.
:ok
```

Напомняме, че използването на `if`, `unless`, `cond`, `raise` и `throw` е забранено. За целта на това домашно ще трябва да спазвате правилото да не стартирате не наблюдавани процеси.
