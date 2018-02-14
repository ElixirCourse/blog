---
title_image_path: control_flow.jpg
category: Програма
created_at: 2017-04-08T13:19:07
tags:
  - elixir
  - control flow
  - erlang
  - pattern matching
  - case
  - if
  - unless
  - cond
  - core erlang
---

# Control Flow

В тази статия ще си говорим за неща, които избягвахме (нарочно) досега.
Целият код в предишните статии е без разделяне на логиката от `if`-ове.
Това може би е странно за хора, които за първи път се сблъскват с функционален
език, но в `Elixir` такива конструкции често са ненужни.
Ако програмираме, използвайки множество от малки функции, комбинирани с гардове
и `pattern matching`, рядко ще имаме нужда от `if-else-then` код.

## Конструкциите `if` и `unless`

Наше мнение е, че `if` е напълно ненужна конструкция в `Elixir`. Не стига това,
но има и още една, още по-ненужна - `unless`.
Нека ги разгледаме:

```elixir
defmodule Person do
  defstruct [:name, :age, :sex, :location, hobbies: []]
end

pesho = %Person{
  name: "Пешо",
  age: 45,
  sex: :male,
  location: 'Елин Пелин',
  hobbies: ~w(eurofootbal drink xxx турбо-фолк)
}
```

Ще използваме чичо Пешо за нашите експерименти.
Ето го и първият ни `if`:

```elixir
# Ще видим "Чичо Пешо"
if pesho.age > 40, do: "Чичо #{pesho.name}", else: pesho.name
```

Тази конструкция може да се напише и в друг вид:

```elixir
defmodule Questionnaire do
  def drinks?(person) do
    if Enum.member?(person.hobbies, "drink") do
      "Това момче е пиянка!"
    else
      "Дааа беее..."
    end
  end
end

# Познайте:
Questionnaire.drinks?(pesho)
```

Този пример доста прилича на `if`-овете в императивните езици.
И в този случай `pattern matching`-ът е по-сложен за ползване, защото проверяваме
дали списък съдържа елемент.
Може да се направи с `pattern matching` все пак:

```elixir
defmodule Questionnaire do
  def drinks?(%Person{hobbies: hobbies}) do
    check_drinks(hobbies)
  end

  defp check_drinks([]), do: "Дааа беее"
  defp check_drinks(["drink" | _]), do: "Това момче е пиянка!"
  defp check_drinks([_ | rest]), do: check_drinks(rest)
end

milena = %Person{
  age: 25,
  name: "Миленката",
  sex: :female,
  location: 'Ямбол',
  hobbies: ~w(шопинг солариум моренце поп-фолкче)
}

# Прави се, но ако я черпят шотчета, коктейлчета, едно-друго и се раздава.
Questionnaire.drinks?(milena)
```

В повечето случаи можем да не ползваме `if`-подобни конструкции.
Освен `if`, имаме `unless`, какво споменахме, което е същото като `if not <expression>`:

```elixir
unless 1 + 1 == 2, do: "Аз съм БСП!", else: "Защо въобще проверяваме??"
```

Стойността на `if` или на `unless` е стойността на израза, който се оценява.

## Конструкцията `cond`

В тази конструкция дефинираме списък от условия със свързан с тях код.
Всяко от тях се оценява, докато стигнем до някое, което се оцени като `true`.
Когато това стане, се изпълнява асоциираният с него код и това е стойността на `cond`-a.

Да речем можем да разпишем `FizzBuzz` проблема от статията [Модули, функции и рекурсия](/posts/modules_functions_recursion) с `cond`, така:

```elixir
defmodule FizzBuzz do
  def of(n), do: Enum.map(1..n, &number_value/1)

  defp number_value(n) do
    cond do
      rem(n, 3) == 0 and rem(n, 5) == 0 -> "FizzBuzz"
      rem(n, 3) == 0 -> "Fizz"
      rem(n, 5) == 0 -> "Buzz"
      true -> n
    end
  end
end

FizzBuzz.of(17)
# [1, 2, "Fizz", 4, "Buzz", "Fizz", 7, 8, "Fizz", "Buzz", 11, "Fizz", 13, 14, "FizzBuzz", 16, 17]
```

Обикновено последното условие на `cond` е `true`, за да има код, който да се изпълни,
когато никое от условията не се оценява като истина. Ако не го сложим и нищо не се оцени като истина
ще има грешка - `(CondClauseError) no cond clause evaluated to a true value`.
Както `if`, `cond` в много случаи е абсолютно ненужен. Ако заменим императивния код, с който сме
свикнали да боравим от други езици, с декларативен, `cond` може да бъде премахнат.
Да си припомним `FizzBuzz` без `cond`:

```elixir
defmodule FizzBuzz do
  def of(n), do: Enum.map(1..n, &number_value/1)

  defp number_value(n) when rem(n, 3) == 0 and rem(n, 5) == 0, do: "Fizz Buzz"
  defp number_value(n) when rem(n, 3) == 0, do: "Fizz"
  defp number_value(n) when rem(n, 5) == 0, do: "Buzz"
  defp number_value(n), do: n
end
```

Когато можем да използваме `guard`-ове и `pattern matching`, е по-добре да използваме тях.
За разлика от `if`, `cond` е 'специална форма'. Специалните форми са най-базовите градивни единици
в `Elixir`, които не могат да се пренапишат от нас.
Те са специални `macro`-си (за тях в бъдещи статии), които не са написани на езика.
От това следва, че ако толкова много ни трябва `control flow` конструкция, е по добре да не ползваме
`if` и `unless`, а някоя на която са базирани - `cond` или `case`.

Конструкциите `if` и `unless` са добавени за да се справи един императивен програмист с навлизането в `Elixir`,
но според нас са повече вредни, отколкото помагат. Забравете че ги има.
Всъщност забравете и за `cond`. Единственото което ви трябва, когато `guard`-овете и `pattern matching`-а не ви стигат е `case`.

## Конструкцията `case`

Тази конструкция приема израз и списък от стойности и свързан с тях код. Ако изразът се `match`-не
с някоя от тези стойности, кодът към нея се изпълнява и неговата стойност е стойността на `case`.

```elixir
pesho = %Person{
  name: "Пешо",
  age: 45,
  sex: :male,
  location: 'Елин Пелин',
  hobbies: ~w(eurofootbal drink xxx турбо-фолк)
}
milena = %Person{
  age: 25,
  name: "Миленката",
  sex: :female,
  location: 'Ямбол',
  hobbies: ~w(шопинг солариум моренце поп-фолкче)
}
nikodim = %Person{
  age: 15,
  sex: :male,
  name: 'Никичко',
  location: 'Софията, бате',
  hobbies: ~w(xxx gaming хип-хоп)
}

defmodule Questionnaire do
  def asl(person) do
    case person do
      %{sex: :male} ->
        "#{person.age}, М, #{person.location}"
      %{sex: :female} ->
        "#{person.age}, F, #{person.location}"
      _ ->
        "#{person.age}, LGBT, #{person.location}"
    end
  end
end

Questionnaire.asl(milena)  # "25, F, Ямбол"
Questionnaire.asl(nikodim) # "15, М, Софията, бате"
Questionnaire.asl(pesho)   # "45, М, Елин Пелин"
```

Както можете да видите, имаме последна клауза, която match-ва всякакви стойности.
Ако нищо не се match-не се изпълнява тази последна клауза. Ако я няма, ще
имаме грешка - `(CaseClauseError) no case clause matching`.

В `case` изразите, можем да ползваме и `guard`-ове:

```elixir
defmodule Questionnaire do
  def able_to_buy_cigarettes(person) do
    case person do
      %{age: age} when is_number(age) and age > 17 -> true
      _ -> false
    end
  end
end
```

Важно е да знаем, че `case`, също като `cond` е специална форма. И отново да
повторим, ползваме го само ако кодът ни стане прекалено разхвърлян или труден
за разбиране с `pattern matching`. Отново - забравете за `if` и `unless`, показахме ви ги
за да ви кажем да не ги ползвате. По добре използвайте `case`.

Защо `case` е по-добър избор от `cond`?
Проверките които правим в `case`, са свързани с `data`-та която му даваме.
Това ще рече, че лесно можем да видим какво и защо проверяваме, че е по-трудно да вкараме странични ефекти.
Проверките в `cond` могат да са всякакви и да са свързани с всякакви данни - много по-лесно е да напишем код,
в който се чудим кое, от къде идва. Много по-лесно е да имаме странични ефекти.

Всичко това не значи, че в `case` не можем да ползваме нещо дефинирано извън него.
Именно затова е най-добре да избягваме и `case` ако ни е възможно.
Разбира се има случаи в които това ще доведе до много странен код.

В заключение - нека никога не ползваме `if`, `unless` и `cond`. Никога.

Кога да ползваме `case`, тогава? Отговорът е прост, когато няма как да ползваме
`guard` клаузи и `pattern matching` лесно. Когато трябва да правим функции, само и само
да не ползваме `case`.

В една от предишните статии - [Низове](/posts/strings), създадохме функция,
която брои срещанията на буквата 'a' в низ:

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

За да не ползваме `control flow` конструкция,
ние написахме проста функция `next_n`, която взима `true` или `false`.
Това е случай, в който без да ни е съвестно можем да използваме `case`:

```elixir
defmodule ACounter do
  def count_it_with_next_grapheme(str) do
    count_with_next_grapheme(str, 0)
  end

  defp count_with_next_grapheme("", n), do: n
  defp count_with_next_grapheme(str, n) do
    {next_grapheme, rest} = String.next_grapheme(str)
    case next_grapheme do
      "a" -> count_with_next_grapheme(rest, n + 1)
      _ -> count_with_next_grapheme(rest, n)
    end
  end
end
```

## `Case` VS `Pattern Matching`

Каква е разликата между `case` и използването на `pattern matching` и `guard`-ове с функции?
Кой от тези два подхода е по-бърз?

За да разберем, ще направим една разходка през компилацията на `Elixir`, а от там и на `Erlang`.
Първо, какво представлява компилацията на `Elixir`?

```
1. Elixir Source Code =>
2. Elixir Macro Expansion =>
3. Erlang Abstract Format =>
4. Core Erlang =>
5. BEAM VM Bytecode
```

Идеята е да видим кода си в някаква четима форма малко преди да е станал bytecode.
Така ще сравним инструкциите които се ползват за `case` и за `pattern matching` с функции.

При компилация, Еликсир се трансформира в Abstract Syntax Tree (AST) код, за който ще си говорим
повечко в следващи статии. Нещо подобно:

```elixir
quote do: 1 + 1
{:+, [context: Elixir, import: Kernel], [1, 1]}
```

На следващата стъпка, този `AST` код, се трансформира в Erlang-ския абстрактен формат:

```erlang
{op,1,'+',{integer,1,1},{integer,1,1}}
```

На следващата стъпка този абстрактен формат се преобразува в `core erlang`.
Този `core erlang` си е семантично верен `erlang`.
Обикновено езици написани за `BEAM` се трансформират до `core erlang`.
На това ниво можем да видим до какви точно извиквания се преобразуват `case` и `pattern matching + functions` и
да ги сравним.

До тук добре, но не е лесно да преобразуваме `Elixir` код до `core erlang`, поне ние не откриваме
лесен начин. Хубавото, обаче е, че има лесен начин да преобразуваме `Erlang` до `core erlang`.

Нека да разгледаме компилацията на `Erlang`:

```
1. Erlang Source Code =>
2. Erlang Abstract Format =>
3. Core Erlang =>
4. BEAM VM Bytecode
```

Интересно - можем да видим къде се включва `Elixir` при компилация до `beam` - преди `Erlang Abstract Format` фазата.

Както казахме, в `Erlang` има много лесен начин за преобразуване на `erlang` код до `core erlang`.
От това следва, че е нужно нашият `elixir` да се преобразува до `erlang`.
Можем да го направим ръчно, но за упражнението ще напишем функция която де-компилира какъвто и да е `beam` файл до `erlang source`:

```elixir
defmodule BeamToErl do
  def transform(beam_file_name, erl_file_name) do
    case :beam_lib.chunks(to_charlist(beam_file_name), [:abstract_code]) do
      {:ok, {_, [{:abstract_code, {:raw_abstract_v1, abstract_code}}]}} ->
        src = :erl_prettypr.format(:erl_syntax.form_list(tl(abstract_code)))
        {:ok, file} = File.open(erl_file_name, [:write])

        IO.binwrite(file, src)
        File.close(file)
      error -> error
    end
  end
end
```

Има много нови неща, но основото е следното: `BeamToErl.transform/2` приема път до `beam` файл, който
съществува и път до `erl` файл, който все още не съществува.
Функцията генерира `erlang` код от `beam` байт кода.

От документацията на `:erl_prettypr.format/2` (`pretty printer` за erlang код), видяхме следното:
```erlang
{ok,{_,[{abstract_code,{_,AC}}]}} =
  beam_lib:chunks("myfile.beam",[abstract_code]),
io:put_chars(erl_prettypr:format(erl_syntax:form_list(AC)))
```

Това горе-долу ни показва как да вземем абстрактен `erlang` от `bytecode` и да го трансформираме в `Erlang` код.
Това именно ползваме.
Всички `erlang` функции, които работят с низове, ползват `charlist`, затова си преобразуваме аргумента - път до `beam` файл до `charlist`.
Ако успешно вземем абстрактния код, го преобразуваме в `erlang` код и го записваме във файл.

Нека започнем. Ето и примера ни:

```elixir
defmodule Cases do
  def use_case(data) do
    case data do
      1 -> "one"
      2 -> "two"
      _ -> "many"
    end
  end

  def use_func(1), do: "one"
  def use_func(2), do: "two"
  def use_func(_), do: "many"
end
```

Просто разписваме една и съща проста логика по двата начина. Нека я компилираме до `beam`:

```bash
elixirc cases.ex
```

Получаваме `Elixir.Cases.beam`. Сега този файл можем да го преобразуваме до `erlang` код с `BeamToErl.transform/2`:

```elixir
BeamToErl.transform("Elixir.Cases.beam", "cases.erl")
```

Сега имаме следния `erlang` код:

```erlang
-compile(no_auto_import).

-file("cases.ex", 1).

-module('Elixir.Cases').

-export(['__info__'/1, use_case/1, use_func/1]).

-spec '__info__'(attributes | compile | exports |
                 functions | macros | md5 | module |
                 native_addresses) -> atom() |
                                      [{atom(), any()} |
                                       {atom(), byte(), integer()}].

'__info__'(functions) -> [{use_case, 1}, {use_func, 1}];
'__info__'(macros) -> [];
'__info__'(info) ->
    erlang:get_module_info('Elixir.Cases', info).

use_case(data@1) ->
    case data@1 of
      1 -> <<"one">>;
      2 -> <<"two">>;
      _ -> <<"many">>
    end.

use_func(1) -> <<"one">>;
use_func(2) -> <<"two">>;
use_func(_) -> <<"many">>.
```

Малко е подробен, нека изтрием ненужните неща:

```erlang
-module('cases').
-export([use_case/1, use_func/1]).

use_case(Data) ->
  case Data of
    1 -> <<"one">>;
    2 -> <<"two">>;
    _ -> <<"many">>
  end.

use_func(1) -> <<"one">>;
use_func(2) -> <<"two">>;
use_func(_) -> <<"many">>.
```

Точно това можехме да направим ръчно, но за упражнението го направихме автоматично.
Нека да го трансформираме в `core erlang`:

```erlang
erl> c("cases.erl", to_core).
```

Сега имаме `cases.core` файл, който съдържа:

```erlang
module 'cases' ['module_info'/0,
    'module_info'/1,
    'use_case'/1,
    'use_func'/1]
    attributes []
'use_case'/1 =
    %% Line 4
    fun (_cor0) ->
      %% Line 5
      case _cor0 of
        %% Line 6
        <1> when 'true' ->
            #{#<111>(8,1,'integer',['unsigned'|['big']]),
              #<110>(8,1,'integer',['unsigned'|['big']]),
              #<101>(8,1,'integer',['unsigned'|['big']])}#
        %% Line 7
        <2> when 'true' ->
            #{#<116>(8,1,'integer',['unsigned'|['big']]),
              #<119>(8,1,'integer',['unsigned'|['big']]),
              #<111>(8,1,'integer',['unsigned'|['big']])}#
        %% Line 8
        <_cor3> when 'true' ->
            #{#<109>(8,1,'integer',['unsigned'|['big']]),
              #<97>(8,1,'integer',['unsigned'|['big']]),
              #<110>(8,1,'integer',['unsigned'|['big']]),
              #<121>(8,1,'integer',['unsigned'|['big']])}#
      end

'use_func'/1 =
    %% Line 11
    fun (_cor0) ->
      case _cor0 of
        <1> when 'true' ->
            #{#<111>(8,1,'integer',['unsigned'|['big']]),
              #<110>(8,1,'integer',['unsigned'|['big']]),
              #<101>(8,1,'integer',['unsigned'|['big']])}#
        %% Line 12
        <2> when 'true' ->
            #{#<116>(8,1,'integer',['unsigned'|['big']]),
              #<119>(8,1,'integer',['unsigned'|['big']]),
              #<111>(8,1,'integer',['unsigned'|['big']])}#
        %% Line 13
        <_cor2> when 'true' ->
            #{#<109>(8,1,'integer',['unsigned'|['big']]),
              #<97>(8,1,'integer',['unsigned'|['big']]),
              #<110>(8,1,'integer',['unsigned'|['big']]),
              #<121>(8,1,'integer',['unsigned'|['big']])}#
  end

'module_info'/0 =
    fun () ->
  call 'erlang':'get_module_info'
      ('cases')
'module_info'/1 =
    fun (_cor0) ->
  call 'erlang':'get_module_info'
      ('cases', _cor0)
end
```

Изненада! Кодът и за `case` и за функции с `pattern matching` е абсолютно еднакъв.
От тук следва, че изборът е философски. Ние ви представихме нашата философия по-горе.
Вие избирате.
