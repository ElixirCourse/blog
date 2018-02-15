category: Домашно
tags:
- elixir
- homework

--------

# Задача 2

Втората задача се състои от две малки под-задачи и е за общо **9 точки**.

Крайният срок за предаване е 23:59 на 02.04.2017 г.

## Информация за задачата
Ако все още не сте се записали за получаване на домашни, можете да го направите, като изпратите имейл до адрес **register@elixir-lang.bg**, който трябва да отговаря на следните условия:
* В Относно (Subject) трябва да напишете **register**.
* В съдържанието на мейла трябва да напишете: име и фамилия, факултетен номер и да сложите линк към GitHub акаунта ви.

Ако сте записани, сте получили мейл с линк към заданието за новото домашно. То съдържа основната структура на проекта, в който тряба да работите. В него има директория `lib`, която ще съдържа два файла съответно за двете части на задачата. Свободни сте да създавате колкото искате допълнително файлове, стига те да се намират в една от директориите `lib` или `test`. В репото ще има няколко базови теста, които да ви съпътстват при писането на задачата (Пускайте тестовете често, за да следите прогреса си). Често "къмитвайте" промените във вашето локално хранилише, също така често "пушвайте" към съответното в GitHub. Ще се постараем да върнем обратна връзка в рамките на един ден (ако не получите такава, може да пишете мейл на **course@elixir-lang.bg**, за да ни напомните). Всичко, предадено в последния ден от срока за предаване, няма да получат обратна връзка, преди да бъдат окончателно проверени.

## Deque

Първата задача е да направите структура, която реализира структурата от данни **Deque**.

Структурата трябва да е дефинирана в модул с името **Deque** (вече имате създаден такъв във файла `lib/deque.ex`). Всички функции за работа с нея трябва да се намират в същия модул.

### Какво представлява Deque

Декът е структура, подобна на динамичен масив, с малката разлика, че освен да добавяме елементи в края ѝ може да го правим и в началото на "масива". Друга характеристика е, че лесно можем да достъпваме и променяме елементите на дека, чрез техния индекс (индексацията на елементите в "дек" винаги започва от 0). Всички тези операции трябва да са сравнително "бързи" (да не отнемат линейно време) - ако не спазите това ограничение, най-вероятно няма да получите максималния брой точки.

### Как трябва да изглежда вашата имплементация

Структурата, която дефинирате, може да има каквито прецените полета, но трябва да можем да създаваме "дек", посредством **%Deque{}**. (В обяснението на функциите е дадена една примерна реализация за по-лесно илюстриране, тя обаче не е много добра, така че помислете за нещо по-добро)

### Какви функции трябва да има в модула Deque

Всички публични функции са описани в следващите параграфи. Вие можете да добавяте колкото искате не-експортирани функции (**defp**). Всяка публична функция от модула трябва да приема като първи аргумент инстанция на **Deque**.

#### Deque.new()

Аналогично на `%Deque{}` създава празден "дек". Например, ако разгледаме реализация на "дек", съдържаща едно поле (:content) в което държим списък.

```elixir
iex> %Deque{}
%Deque{content: []}
iex> Deque.new()
%Deque{content: []}
iex> %Deque{} == Deque.new()
true
```

#### Deque.size(deque)

Връща броя елементи съдържащи се в **deque**.

```elixir
iex> %Deque{content: [1, 2.0, "three"]} |> Deque.size
3
```

#### Deque.push_back(deque, element)

Връща нов "дек", подобен на **deque**, но с добавен последен елемент **element**.

```elixir
iex> %Deque{content: [1, 2.0, "three"]} |> Deque.push_back(:four)
%Deque{content: [1, 2.0, "three", :four]}
```

#### Deque.push_front(deque, element)

Връща нов "дек", подобен на **deque**, но с добавен първи елемент **element**.

```elixir
iex> %Deque{content: [1, 2.0, "three"]} |> Deque.push_front(:zero)
%Deque{content: [:zero, 1, 2.0, "three"]}
```

#### Deque.pop_back(deque)

Връща нов "дек", подобен на **deque**, но с премахнат последен елемент. Ако елементите на новия "дек" са 0, то да се върне `%Deque{}`. Ако **deque** няма елемент да се върне **deque**.

```elixir
iex> %Deque{content: [1, 2.0, "three"]} |> Deque.pop_back
%Deque{content: [1, 2.0]}
iex> %Deque{content: []} |> Deque.pop_back
%Deque{content: []}
iex> ( %Deque{content: [1]} |> Deque.pop_back ) == %Deque{}
true
```

#### Deque.pop_front(deque)

Връща нов "дек", подобен на **deque**, но с премахнат първи елемент. Ако елементите на новия "дек" са 0, то да се върне `%Deque{}`. Ако **deque** няма елемент да се върне **deque**.

```elixir
iex> %Deque{content: [1, 2.0, "three"]} |> Deque.pop_front
%Deque{content: [2.0, "three"]}
iex> %Deque{content: []} |> Deque.pop_front
%Deque{content: []}
iex> ( %Deque{content: [1]} |> Deque.pop_front ) == %Deque{}
true
```

#### Deque.last(deque)

Връша последния елемент в **deque**, ако "декът" е празен връща `nil`.

```elixir
iex> %Deque{content: [1, 2.0, "three"]} |> Deque.last
"three"
iex> Deque.new() |> Deque.last
nil
```

#### Deque.first(deque)

Връша първия елемент в **deque**, ако "декът" е празен връща `nil`.


```elixir
iex> %Deque{content: [1, 2.0, "three"]} |> Deque.first
1
iex> Deque.new() |> Deque.first
nil
```

#### Deque.access_at(deque, n)

Връща елемента на позиция **n** от **deque**. Ако "декът" няма позиция **n** функцията връща `nil`, а ако **n** не е цяло число се случва грешка (каквато и да е).

```elixir
iex> deque = %Deque{content: [1, 2.0, "three"]}
%Deque{content: [1, 2.0, "three"]}
iex> deque |> Deque.access_at(0)
1
iex> deque |> Deque.access_at(1)
2.0
iex> deque |> Deque.access_at(2)
"three"
iex> deque |> Deque.access_at(3)
nil
iex> Deque.access_at(deque, "2")
** (FunctionClauseError) no function clause matching in ...
```

#### Deque.assign_at(deque, n, element)

Връща нов "дек", подобен на **deque**, но елемента на позиция **n** е сменен с **element**. Ако няма елемент с индекс **n** или **n** не е цяло число се случва грешка (каквато и да е).


```elixir
iex> deque = %Deque{content: [1, 2.0, "three"]}
%Deque{content: [1, 2.0, "three"]}
iex> deque |> Deque.assign_at(0, :zero)
%Deque{content: [:zero, 2.0, "three"]}
iex> deque |> Deque.assign_at(1, :one)
%Deque{content: [1, :one, "three"]}
iex> deque |> Deque.assign_at(2, :two)
%Deque{content: [1, 2.0, :two]}
iex> deque |> Deque.assign_at(3)
** (FunctionClauseError) no function clause matching in ...
iex> deque |> Deque.assign_at("2")
** (FunctionClauseError) no function clause matching in ...
```

#### Имплементация на протоколи

За структурата **Deque** да се имплементират протоколите **Collectable** и **Enumerable**. За целта прочетете внимателно документацията на двата модула.

След като имплементирате **Collectable**, трябва да имате следното поведение за структурата ви:

```elixir
iex> deque = 0..5 |> Enum.into Deque.new
%Deque{content: [0, 1, 2, 3, 4, 5]}
iex> [:a, :b, :c] |> Enum.into deque
%Deque{content: [0, 1, 2, 3, 4, 5, :a, :b, :c]}
```

Протокола **Enumerable** трябва да ви позволи да подавате структурата, като първи аргумент на всички други функции от модула **Enum**:


```elixir
iex> deque = %Deque{content: [0, 1, 2, 3, 4, 5]}
%Deque{content: [0, 1, 2, 3, 4, 5]}
iex> deque |> Enum.take 2
[0, 1]
iex> deque |> Enum.drop 3
[3, 4, 5]
iex> deque |> Enum.map &(&1*&1)
[0, 1, 4, 9, 16, 25]
iex> deque |> Enum.reverse
[5, 4, 3, 2, 1, 0]
```

## Sequence

Втората задача е да направите структура, която да имплементира генератор на редица.

Структурата трябва да е дефинирана в модул с името **Sequence** (вече имате създаден такъв във файла `lib/sequence.ex`). Всички функции за работа с нея трябва да се намират в същия модул.

### Как трябва да изглежда вашата имплементация

Структурата трябва да има точно две полета, които да са задължитени:
- :state e състоянието на редицата
- :generator е функция която приема "състояние" и връща наредана двойка от генериран елемент и ново състояние

Например така бихме дефинирали генератор на естествените числа:

```elixir
%Sequence{state: 0, generator: &{&1, &1 + 1}}
```

### Какви функции трябва да имa в модула Sequence

Всички публични функции са описани в следващите параграфи. Вие можете да добавяте колкото искате не-експортирани функции (**defp**). Всяка публична функция от модула трябва да приема, като първо аргумент инстанция на **Sequence**.

#### Sequence.generate(seq)

Функцията връща наредена двойка, първия елемент от която е генерираният елемент, а вторият - нова редица със същия "генератор", но променено състояние.

```elixir
iex> func = &{&1, &1 + 1}
iex> %Sequence{state: 0, generator: func} |> Sequence.generate
{0, %Sequence{state: 1, generator: func}}
```
За да можем да генерираме крайни редици ще приемем, че ако състоянието ни е `nil`, то вместо да върнем наредена двойка ще върнем `nil` и ще приемаме, че е редицата е приключила.

```elixir
iex> %Sequence{state: nil, generator: fn(_) -> 1 end} |> Sequence.generate
nil
```

#### Sequence.generate_value(seq)

Прави същото като `Sequence.generate/1`, но връща само генерирания елемент без новата редица.

```elixir
iex> %Sequence{state: 0, generator: &{&1, &1 + 1}} |> Sequence.generate_value
0
```

Ако редицата е приключила се случва грешка:

```elixir
iex> %Sequence{state: nil, generator: fn(_) -> 1 end} |> Sequence.generate_value
** (FunctionClauseError) no function clause matching in ...
```

#### Sequence.generate_next(seq)

Прави същото като `Sequence.generate/1`, но връща само новата редица без генерирания елемент.

```elixir
iex> func = &{&1, &1 + 1}
iex> %Sequence{state: 0, generator: func} |> Sequence.generate_next
%Sequence{state: 1, generator: func}
```

Ако редицата е приключила се връща `nil`

```elixir
iex> %Sequence{state: nil, generator: fn(_) -> 1 end} |> Sequence.generate_next
nil
```

#### Sequence.advance(seq, n)

Функцията приема редица **seq** и цяло неотрицателно число **n** и връща нова редица, която е резултат от n кратното прилагане на `Sequence.generate_next/1` върху редицата **seq**.

```elixir
iex> func = &{&1, &1 + 1}
iex> seq = %Sequence{state: 0, generator: func}
iex> seq |> Sequence.advance 5
%Sequence{state: 5, generator: func}
iex> seq |> Sequence.advance 0
%Sequence{state: 0, generator: func}
iex> seq |> Sequence.advance -1
** (FunctionClauseError) no function clause matching in ...
```

Ако в някой момент от прилагането на `Sequence.generate_next/1` бива върнат `nil`, то резултатът от `Sequence.advance/2` също да е `nil`.


```elixir
iex> %Sequence{state: nil, generator: fn(_) -> 1 end} |> Sequence.advance(5)
nil
```

#### Sequence.limit(seq, n)

Функцията приема редица **seq** и цяло неотрицателно число **n** и връща нова редица, която може да произведе не повече от **n** елемента.

Пример за лимитиране на безкрайна редица:

```elixir
iex> seq = %Sequence{state: 0, generator: &{&1, &1 + 1}} |> Sequence.limit 2
iex> {n, seq} = seq |> Sequence.generate; n
0
iex> {n, seq} = seq |> Sequence.generate; n
1
iex> seq |> Sequence.generate
nil
```
Пример за лимитиране на крайна редица с по-малко елементи от лимита:

```elixir
iex> seq = %Sequence{state: 0, generator: &{&1, nil}} |> Sequence.limit 2
iex> {n, seq} = seq |> Sequence.generate; n
0
iex> seq |> Sequence.generate
nil
```

#### Sequence.cycle(seq)

Функцията приема редица **seq** и връща нова. Ако дадената редица е крайна, новата редица трява да прдставлява първата с разликата, че всеки път когато достигне края си тя се връща в началното си състояние.

Например, ако **seq** генерира 1, 2, 3 то **Sequence.cycle(seq)** трябва да генерира 1, 2, 3, 1, 2, 3...

Ако редицата не е крайна, то новата редица генерира същите елементи, като старата (това не означава, че е непроменена).

#### Sequence.repeat(seq, n)

Функцията приема редица **seq** и цяло неотрицателно число **n** и връща нова. Ако дадената редица е крайна, новата редица, трябва да повтаря дадената **n** пъти.

Например, ако **seq** генерира 1, 2, 3 то **Sequence.repeat(seq, 3)** трябва да генерира 1, 2, 3, 1, 2, 3, 1, 2, 3.

Ако **n** не е цяло чило или е по-малко от 0, то се случва грешка. Ако редицата не е крайна, то новата редица генерира същите елементи, като старата (това не означава, че е непроменена).

#### Sequence.concatenate(seq_one, seq_two)

Функцията приема две редици **seq_one** и **seq_two** и връща нова. Ако **seq_one** е крайна върнатата редица се генерират първо елементите от **seq_one**, а после и тези от **seq_two**

Например, ако **seq_one** генерира 1, 2, 3, а **seq_two** генерира "one", "two", "three", то **Sequence.concatenate(seq_one, seq_two)** трябва да генерира 1, 2, 3, "one", "two", "three"


#### Имплементация на протоколи

За структурата трябва да имплементирате протокола **Enumerable**. За целта прочетете внимателно документацията на протокола.

Протоколът **Enumerable** трябва да ви позволи да подавате структурата, като първи аргумент на всички функции от модула **Enum**:

```elixir
iex> seq = %Sequence{state: 0, &{&1, &1 + 1}}
iex> seq |> Enum.take 2
[0, 1]
iex> seq |> Enum.empty?
false
iex> seq |> Enum.any?(&(&1 > 5))
true
iex> seq |> Enum.at(5)
5
```

Някои от тях обаче няма да работят за безкрайни редици:


```elixir
iex> seq = %Sequence{state: 0, &{&1, &1 + 1}}
iex> seq |> Enum.map(&(&1*&1))
безкрайна рекурсия
iex> seq |> Enum.any?(&(&1 < -2))
безкрайна рекурсия
iex> seq |> Enum.take_random()
първо тряябва да знае колко елементи има и чак тогава избира произволен от тях
```

За крайни редици обаче всички функции трябва да работят.
