---
category: Домашно
created_at: 2018-03-20T07:54:43
author: njichev
tags:
  - elixir
  - homework
---

# Задача 1

Първата задача е за общо **15 точки**.

Крайният срок за предаване е 23:59 на 11.04.2018 г.

Ако имате въпрос по домашното пишете на: course@elixir-lang.bg

## GitHub
Ако нямате [GitHub](github.com) акаунт, време е да си направите такъв, ще ви трябва за домашното.

Селд като кликнете на следващия линк и се логнете в GitHub, за вас ще бъде създадено репо, в което да качвате работата си.

https://classroom.github.com/a/nqor38Gg

Къмитвайте и пушвайте често. Ние ще се стараем да направим ревю на кода ви.

## Тестoве

В репото, което ще получите, има няколко дефинирани теста.
С тях можете да проверявате дали сте имплементирали най-базовата функционалност.

___

# Имплементирайте TestSuite

Задачата е да напишете модул с име `TestSuite`, в който да дефинирате структура, която ще държи набор от тестов.

Тестовете са анонимни функции с арност 0.
Тест се смята за неуспешен, ако след изпълнението си функцията върне `false`, `nil` или хвърли грешка.
Ако функцията върне нещо резлично от `false` или `nil`, то съответният тест е успешен.

Всъщност един тест може да има 5 различни състояния:

  1. Преди да бъде пуснат - ще го бележим с `:pending`
  2. Успешен - `:passed`
  3. Провалил се - `:failed`
  4. Пропуснат - `:skipped`
  5. Не завършил за дадено време - `:timed_out`

## Функции, които TestSuite трябва да съдържа.

#### TestSuite.new()

Създава нов празен `TestSuite`, аналогично на `%TestSuite{}`

```elixir
TestSuite.new() == %TestSuite{}
# => true
```
#### TestSuite.new(initial_tests)
Създав `TestSuite`, който съдържа всички тестове от `initial`.
Ако тестовете са повече от един, то те се добавят в реда, в който ги виждаме. (Това ще е от значение по-нататък и важи също за следващата функция).

```elixir
# Всеки тест е проста анонимна функция.
f = fn -> 3 == 3 end
g = fn -> 3 == 4 end

TestSuite.new([f, g])
```

#### TestSuite.add(test_suite, to_add):
`=to_add` е една анонимна функция или списък от няколко такив.

```elixir
f = fn -> 3 == 3 end
g = fn -> 3 == 4 end

TestSuite.new() |> TestSuite.add(f)
TestSuite.new() |> TestSuite.add([f, g])
```

#### TestSuite.add(test_suite, to_add, tags)

Вторият параметър е същия, като при `TestSuite.add/2`.
Третият съдържа списък от атоми или един атом, това са таговете на добавените тестове чрез тази функция.
Таговете ще използваме по-късно за да филтрираме тестовете при изпълнението на други функции.

```elixir
f = fn -> 3 == 3 end
g = fn -> 3 == 4 end

TestSuite.new() |> TestSuite.add(f, [:slow, :do_not_run])
TestSuite.new() |> TestSuite.add([f, g], :maybe)
```

#### TestSuite.size(test_suite)

Връща размера на TestSuite-а (борая на тестовете в ного).

```
TestSuite.new() |> TestSuite.size()
# => 0
TestSuite.new([fn -> true end]) |> TestSuite.size()
# => 1
```

#### TestSuite.size(test_suite, options)

Връща борая на тестовете филтрирани чевз аргумента `options`.
`options` e асоциативен списък, който може да съдържа следните "филтр":
  - `:only` - ако има такава опция се връща броя на тестовете с даден таг

```elixir
TestSuite.new()
|> TestSuite.add(fn -> true end, :always_true)
|> TestSuite.add([fn -> false end, fn -> nil end], :never_true)
|> TestSuite.size(only: :never_true)
#=> 2
```
  - `:exclude` - ако има такава опция се връща броя на тестовете, които не съдържат даден таг

```elixir
TestSuite.new()
|> TestSuite.add(fn -> true end, :always_true)
|> TestSuite.add([fn -> false end, fn -> nil end], :never_true)
|> TestSuite.size(exclude: :never_true)
#=> 1
```
 - можем да комбинираме двете например:

```elixir
TestSuite.new()
|> TestSuite.add(fn -> true end, :true)
|> TestSuite.add(fn -> false end, :false)
|> TestSuite.add(fn -> rem(:rand.uniform(), 2) == 0 end, [:true, :false])
|> TestSuite.size(only: :true, exclude: :flase)
#=> 1
```
#### TestSuite.run(test_suite, options \\ [])

Изпилнява тестовете и връща модифицирана `TestSuite` с информация за изпълнените тестове.
Информация ще ни трябва по-нататък.

```elixir
f = fn -> 3 == 3 end
g = fn -> 3 == 4 end

TestSuite.new()
|> TestSuite.add(f) # Ще бъде маркиран, като :passed
|> TestSuite.add(g) # Ще бъде маркиран, като :failed
|> TestSuite.run()
```

Първите две опции са като при `TestSuite.size/2` (`:only`, `:exclude`), като всеки тест, който бъде филтриран не се изпълнява, а се маркира, катo `:skipped`

```elixir
f = fn -> 3 == 3 end
g = fn -> 3 == 4 end

TestSuite.new()
|> TestSuite.add(f, :first) # Ще се изпълни
|> TestSuite.add(g, :second) # Ще бъде маркиран, като пропуснат
|> TestSuite.run(only: :first)
```

Другата опция за изпълнението на тестовете е `:timeout` (тя има дефаултна стойност 5000 ms)

Ако един тест работи повече от това време той бива маркиран, като `:timed_out`

```elixir
f = fn ->
  Process.sleep(2000)
  true
end

TestSuite.new()
|> TestSuite.add(f) # Ще бъде маркиран, като :timed_out
|> TestSuite.run(timeout: 500)

TestSuite.new()
|> TestSuite.add(f) # Ще бъде маркиран, като :passed
|> TestSuite.run(timeout: 5000)
```
Можем да run-нем `TestSuite`, който е върнат от `run`, в такъв случай ще изпълним само функциите, които не са маркирани, като `:passed`

Всъщност се очаква да има фукции `timed_out`, `failed` и `skipped`, които правят същото като `passed`, но за съответните състояния.

#### TestSuite.passed(test_suite)
Връща нов `TestSuite` съдържащ само тезтовете, които са били изпълнени и са били успешни

```elixir
f = fn -> 3 == 3 end
g = fn -> 3 == 4 end

passed =
TestSuite.new()
|> TestSuite.add(f)
|> TestSuite.add(g)
|> TestSuite.run()
|> TestSuite.passed()
|> TestSuite.size()

TestSuite.size(passed) # passed съдържа само "теста" f, който е отбелязана като :passed
#=> 1
```
#### TestSuite.failed(test_suite)
#### TestSuite.timed_out(test_suite)
#### TestSuite.skipped(test_suite)
#### TestSuite.pending(test_suite)

Последните 4 са аналогични на `TestSuite.passed/1`, но за съответните тест статуси.

#### TestSuite.ran?(test_suite)

Връща `true`, ако TestSuite-а е върнат от `TestSuite.run/2`

```elixir
TestSuite.new() |> TestSuite.ran?() # => false

TestSuite.new() |> TestSuite.run() |> TestSuite.ran?() # => true

TestSuite.new()
|> TestSuite.run()
|> TestSuite.add(fn -> true end)
|> TestSuite.ran?() # => false
```
#### TestSuite.reset(test_suite)

Връща нов `TestSuite` подобен на входния, само че всички тестове са отбелязани, като `:pending`.

```elixir
reset_suite =
TestSuite.new(fn -> true end)
|> TestSuite.run()
|> TestSuite.reset()

reset_suite
|> TestSuite.size()
#=> 1

reset_suite
|> TestSuite.pending()
|> TestSuite.size()
#=> 1
```

### Протокола `Inspect`

Трябва да имплементирате протокола [`Inspect`](https://hexdocs.pm/elixir/Inspect.html#content) за структурата `Dequue`

Когато "инспецтираме" един празен `TestSuite`, той трябва да получим следния резултат:

```elixir
t = TestSuite.new()

inspect(t) #=> "#TestSuite<0 tests>"
```

Ако имаме тестове нещата са малко по различни

```elixir
f = fn -> 3 == 3 end
g = fn -> 3 == 4 end
h = fn -> Process.sleep(1000) end
j = fn -> true end

test_suite =
TestSuite.new()
|> TestSuite.add(f)
|> TestSuite.add(g)
|> TestSuite.add(h)
|> TestSuite.add(j, :skip)
|> TestSuite.run(exclude: :skip, timeout: 500)
|> TestSuite.add(fn -> nil end)
|> inspect()

inspect(test_suite)
#=> "#TestSuite<5 tests:.FTSP>"
```

Където всеки успешен тест е отбелязан с ".", всеки провали се - с "F", всеки пропуснат - със "S", всеки, който не е минал за допустимо време - "T" и всеки още не е изпълнен - с "P"

Реда в който показваме резултатите е същият в който са били добавени в `TestSuite`-a.

# Бонус точки

Добавете опция на `:parallel` брой на тестове, които да се изпълняват кнкурентно.
Имплементирайте схема за паралелизиране на тестовете.
