title_image_path: metaprogramming.jpg
title: Meta-програмиране в Elixir част 2
category: Програма
tags:
  - elixir
  - metaprogramming
  - use
--------

## [От миналия път](https://elixir-lang.bg/posts/metaprogramming_part1)

Последно се запознахме с AST-то на Elixir, видяхме [как прилича на LISP](https://www.youtube.com/watch?v=IZvpKhA6t8A&feature=youtu.be&t=12m10s) и как да боравим с него. Научихме се да използваме `quote` и `unquote`. Обърнахме операциите плюс и умножение с минус и деление. Накрая си написахме наша версия на `while` цикъл.  

# Какво ще правим във втората част за метапрограмиране?

Днес ще се запознаем с това какво са модулни атрибути, ще разгледаме различни compile-time hooks, ще се научим да използваме `bind_quoted` и `__using__`. Ще поговорим за това как да пишем `чисти` макроси и ще разгледаме `var!`. В процеса на работа, ще научим повече за това как е написан Еликсир и колко лесно можем сами да си разширяваме езика, като сами напишем `unless` макроса и разгледаме други примери в ядрото на Еликсир. Накрая ще изобретим собствена версия на библиотека за тестове, в процеса на което ще се сблъскаме с проблеми около писането на макроси.


# Макросa `unless`.

Нека започнем с нещо лесно, а именно `unless` макроса. На кратко `unless` изпълнява даден блок, когато дадено условие е лъжа(обратното на `if`). Да видим набързо AST-то на един `if`.

```
iex(1)> quote do
...(1)>   if true do
...(1)>     "Hello"
...(1)>   else
...(1)>     "Goodbye"
...(1)>   end
...(1)> end
{:if, [context: Elixir, import: Kernel], [true, [do: "Hello", else: "Goodbye"]]}
```

 От миналия път знаем, че `...макросите са функции, които приемат като аргумент AST и връщат AST...`. Тогава като гледаме AST-то на `if`-а, трябва просто да обърнем стойността на условието, примерно използвайки `!` оператора и да върнем почти същото AST. Нека да видим това как ще стане. Започваме като дефинираме макроса с `defmacro`, името и блока -  като аргументи, които приема. Използваме `quote` и `unquote` - съответно да върнем AST и вземем стойността на условието и блока.

 ```
defmodule Conditions do
  defmacro fmi_unless(condition, do: block) do
    quote do
      if unquote(!condition), do: unquote(block)
    end
  end
end
 ```

 Нека видим как може да ползваме нашето макро в `iex`:

 ```
iex(3)> fmi_unless false, do: IO.puts "Hi"
Hi
:ok
iex(4)> fmi_unless true, do: IO.puts "Hi"
nil
iex(5)> ast = quote do
...(5)>   fmi_unless true, do: "Hello"
...(5)> end
{:fmi_unless, [context: Elixir, import: Conditions], [true, [do: "Hello"]]}
iex(6)> Macro.expand_once ast, __ENV__
{:if, [context: Conditions, import: Kernel], [false, [do: "Hello"]]}
 ```

 Какво виждаме, като веднъж разгънем макроса(като използвахме [`Macro.expand_once`](https://hexdocs.pm/elixir/Macro.html#expand_once/2)), то се е генерирало до нормален `if`. Всъщност може да считаме, че когато Еликсир види макро, почва да го разгъва рекурсивно, докато повече не може. Може да разгледате [`Macro`](https://hexdocs.pm/elixir/1.0.5/Macro.html) и [`Code`](https://hexdocs.pm/elixir/1.0.5/Code.html) модулите за повече информация.

 Обаче тази имплементация е малко проста и ни оставя да желаем повече, ще работи ли, ако добавим `else`, или какво става, ако потребител добави друга клауза освен `do` и `else`? Може да се опитате да се погрижите за тези случаи сами :).  
 Една от причините да харесвам толкова много Еликсир, е че лесно можем да разгледаме хората как се се погрижили за тези неща просто като отворим кода на Еликсир написан на Еликсир. Ако се предавате за `unless`-а, може да видите [тук](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/lib/kernel.ex#L2766).

Не ни вярвате, че if–овете са истина за всичко, освен `nil/false`? Вече няма нужда да ни се доверявате, може сами да разгледате как са реализирани всички яки работи. И ето малко примери:
  - [if](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/lib/kernel.ex#L2719)
  - [&&](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/lib/kernel.ex#L2896)
  - [||](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/lib/kernel.ex#L2931)
  - [!](https://github.com/elixir-lang/elixir/blob/master/lib/elixir/lib/kernel.ex#L1484)

Всъщност, може сами да забележите как `&&` е направен мързеливо да връща `false`
ако още първата част не се оценява до истина.

## Набързо за bind_quoted

`bind_quoted` е една от опциите, които можем да подаваме на `quotе`.
Често в макросите използваме `unquote`, за да оценим някаква променлива в подадения ни контекст:

```
defmodule Hello
  defmacro say(name)
    quote do
      "Здравей #{unquote(name)}, как е?"
    end
  end
end
```

Всъщност това може да го пренапишем така:

```
defmodule Hello
  defmacro say(name)
    quote bind_quoted: [name: name] do
      "Здравей #{name}, как е?"
    end
  end
end
```

И като разцъкаме в `iex`:

```
iex(1)> Hello.say("Ники")
"Здравей Ники, как е?"
iex(2)> name
** (CompileError) iex:4: undefined function name/0
```

Тук забелязваме нещо важно - `name`, което беше дефинирано вътре в макроса не съществува извън него. Това е част от чистотата на макросите, за която ще си говорим малко по-късно.

Можем да видим пълен списък с опции, които приема `quote` [тук](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#quote/2-options)

# Чистота на макросите.

Като пишем макроси в Еликсир не генерираме само код, ние го инжектираме в контекста подаден ни от извикващата функция. Контекстът държи локалния `binding`, вмъкнатите модули и псевдоними. Един вид контекстът е света, който виждаме от макроса - за това е толкова важен.

Нека видим как Еликсир ни предпазва от това да "замърсяваме" средата в която сме, като се опитаме да достъпим външна променлива.

```
iex(1)> ast = quote do
...(1)>   if a == 42 do
...(1)>     "The answer is?"
...(1)>   else
...(1)>     "Mehhh"
...(1)>   end
...(1)> end
iex(2)> Code.eval_quoted ast, a: 42
warning: variable "a" does not exist and is being expanded to "a()", please use parentheses to remove the ambiguity or chang
e the variable name
  nofile:1

** (CompileError) nofile:1: undefined function a/0
    (stdlib) lists.erl:1354: :lists.mapfoldl/3
    (elixir) expanding macro: Kernel.if/2
    nofile:1: (file)
```

Въпреки, че инжектирахме променливата `a` в локалния binding чрез [`Code.eval_quoted`](https://hexdocs.pm/elixir/1.0.5/Code.html#eval_quoted/3), Еликсир не ни позволява неявно да предефинираме локалния binding на променливи в контекста на извикващия. Как може да накараме този пример да работи?

# var!

Като използваме `var!` макроса, можем явно да предефинираме локалния binding в контекста - подаден ни в макроса. По този начин казваме на Еликсир: "Знам какво правя, не се притеснявай, тези външни неща ще ги използвам." Нека накараме предишния пример да проработи:

```
iex(1)> ast = quote do
...(1)>   if var!(a) == 42 do
...(1)>     "The answer is?"
...(1)>   else
...(1)>     "Mehhh"
...(1)>   end
...(1)> end
{:if, [context: Elixir, import: Kernel],
 [{:==, [context: Elixir, import: Kernel],
   [{:var!, [context: Elixir, import: Kernel], [{:a, [], Elixir}]}, 42]},
  [do: "The answer is?", else: "Mehhh"]]}
iex(2)> Code.eval_quoted ast, a: 42
{"The answer is?", [a: 42]}
iex(3)> Code.eval_quoted ast, a: 1
{"Mehhh", [a: 1]}
```

Добре, тук само използваxме променливата в условието на `if`-а, но не я променихме по какъвто и да е начин, нека разгледаме по-опасен пример:

```
iex(1)> defmodule Dangerous do
...(1)>   defmacro rename(new_name) do
...(1)>     quote do
...(1)>       var!(name) = unquote(new_name)
...(1)>     end
...(1)>   end
...(1)> end
{:module, Dangerous, .....
iex(2)> require Dangerous
Dangerous
iex(3)> name = "Слави"
"Слави"
iex(4)> Dangerous.rename("Вало")
"Вало"
iex(5)> name
"Вало"
```

# Наша собствена библиотека за тестове.

Добре, вече знаем как да използваме `quote/unquote/bind_quoted`, `var!`. Ще се опитаме да си напишем собствена библиотека за тестове с малък приятен DSL заимстван от `exunit`. За целта ще се опитаме да предоставим следните неща:
  - удобен начин хората да използват библиотеката ни:
  ```
  defmodule TestUsers do
    use Specs
  end
  ```
  - искаме лесно да може да проверяваш стойности и да ги сравняваш, за целта ще имаме модул Assertion, който ще предоставя тази функционалност.
  ```
  assert value
  assert value == 4
  assert value <= 5
  ```
  - начин да създаваме отделни тестове с кратко описание, което ще изглежда нещо от сорта на: 
  ```
  spec "кратко описание", do: ...block of testing code...
  ```
  - И разбира се, начин да пускаме тестовете.


## __using__

Добре, нека започнем първо с `__using__`  - макро, което ни дава да дефинираме callback, когато някой ни използва модула. На кратко, ако имаме:

```
defmodule UserTest do
  use Assertion, option: "Hello"
end
```

Това ще се компилира до:

```
defmodule UserTest do
  require Assertion
  Assertion.__using__(option: "Hello")
end
```

Това ни позволява лесно да вкараме методите и макросите за тестове, които ще са нужни за потребителите на нашата библиотека.

Как ще го постигнем това? Като предефинираме макроса `__using__`. Просто искаме да `import`-нем нашия модул, можем да използваме [`__MODULE__`](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#__MODULE__/0).

```
defmacro __using__(_options) do
  quote do
    import unquote(__MODULE__)
  end
end
```

Така ще можем да импортираме всички функции от модулите ни, когато човек иска да се възползва от тях. Потребителите на нашата библиотека, биха писали само `use Assertion`.

## Assertion

Как ще позволяваме на хората да проверяват различни твърдения? Нека видим в други езици как е постигнато това:

```
assert value
assert_equal value, 4
assert_operator value, :<, 5
```

Нещо яко - за разлика от други езици, с помощта на pattern-matching ще пишем само:

```
assert value
assert value == 4
assert value < 5
```

Казах ли, че можем да pattern match-ваме по AST-то, точно както pattern match-ваме аргументите в обикновени функции? Ооооо да. Нека видим колко лесно можем да построим модула за твърдения.

```
defmodule Assertion do
  defmacro __using__(_options) do
    quote do
      import unquote(__MODULE__)
    end
  end

  defmacro assert({operator, _context, [lhs, rhs]}) do
    quote bind_quoted: [operator: operator, lhs: lhs, rhs: rhs] do
      do_assert(operator, lhs, rhs)
    end
  end
  defmacro assert(value) do
    quote bind_quoted: [value: value] do
      do_assert(value)
    end
  end


  def do_assert(:<, left, right) when left < right, do: :ok
  def do_assert(:<, left, right) do
    {:error, "Expected left side #{left} to be smaller than right side #{right}"}
  end

  def do_assert(:==, left, right) when left == right, do: :ok
  def do_assert(:==, left, right) do
    {:error, "Expected the left side #{left} to be equal to the right side #{right}"}
  end

  def do_assert(operator, _left, _right) do
    {:error, "Could not recognize operator: #{operator}"}
  end

  def do_assert(value) when value in [false, nil] do
    {:error, "Expected #{value} to be truthy."}
  end
  def do_assert(_value), do: :ok
end
```

Какво направихме? Съпоставяме получените оператори и изпълняваме функцията `do_assert`, която предефинираме за различните оператори. Можем просто да добавим повече клаузи към `do_assert`, ако искаме да добавим още оператори, разбира се трябва да внимаваме да не изпуснем някой случай.
В момента импортваме всичко дефинирано вътре в `Assertion`, как можем да го избегнем това? Това е оставено като упражнение за читателя.

Добре, вече може да проверяваме твърдения, сега трябва да дефинираме начин да пишем тестовете. Накратко - искаме да дефинираме някакво макро `spec`, което ще приема низ(описанието на теста) и блок, който да изпълним след това. Когато потребителят използва `spec` ще дефинираме функция, с името на описанието и ще си записваме някъде всички `spec`-ове, които хората са си дефинирали. Когато потребител иска да стартира тестовете, ще минаваме през всички записи и ще ги изпълняваме. За целта да ги записваме ще използваме модулни атрибути.

## Модулни атрибути

За какво и как се използват модулни атрибути?
  - Пазене на временна информация по време на компилация.
  - Да държат информация за модула, която ще бъде използвана от потребителя или виртуалната машина.
  - Или можем да ги използваме като константи.

Бележим модулните атрибути с префикс `@`.

Как ще пазим всички написани тестове? Ще инициализираме модулен атрибут, който ще е списък, в който ще записваме всеки използван тест. За да не презаписваме предишния тест, ще добавяме в началото всяка двойка от име на тест и описание.

```
defmodule Specs do
  defmacro __using__(_options) do
    quote do
      # Добавяме модула за тестване на твърдения
      use Assertion

      # Инициализираме празен списък като модулен атрибут.
      @specs []

      import unquote(__MODULE__)
    end
  end

  defmacro spec(description, do: spec_block) do
    # def иска атом като първи аргумент
    func_name = String.to_atom(description)
    quote do
      # Добавяме в началото всеки тест.
      @specs [{unquote(func_name), unquote(description)} | @specs]
      # spec просто ще дефинира нормална функция
      def unquote(func_name)(), do: unquote(spec_block)
    end
  end
end
```

Добре, нека да видим дали ни се създават и записват успешно тестовете:


```
iex(1)> defmodule ExampleTests do
...(1)>   use Specs
...(1)>
...(1)>   spec "Test success" do
...(1)>     assert 1 == 1
...(1)>   end
...(1)>
...(1)>   spec "Test failure" do
...(1)>     assert 1 == 2
...(1)>   end
...(1)>
...(1)>   def specs do
...(1)>     @specs
...(1)>   end
...(1)> end
{:module, ExampleTests ...
iex(2)> ExampleTests.specs
["Test failure": "Test failure", "Test success": "Test success"]
iex(3)> apply(ExampleTests, :"Test failure", [])
{:error, "Expected the left side 1 to be equal to the right side 2"}
iex(4)> apply(ExampleTests, :"Test success", [])
:ok
```

Добре, дефинирахме и записахме успешно тестовете. Използвахме [`apply/3`](https://hexdocs.pm/elixir/Kernel.html#apply/3) - така извикахме произволна функцията само по нейното име и модул. Всичко върви прекрасно, само че трябваше ръчно да извикаме всеки тест с `apply`, което не беше много прекрасно. Вместо това ще се опитаме да дефинираме функция `run`, вътре в модула на потребителя, която да извика всички тестове за нас.

Тоест ще искаме да пишем само веднъж `ExampleTests.run` и това да изпълнява всички тестове.

## SpecRunner

Нека реализираме последния нужен модул, който автоматично ще дефинира функцията `run` в ExampleTests. За жалост не може просто да си отворим модула `ExampleTests` и да му добавим функцията `run`, нито пък можем да се отървем само с импортиране на модула ни. Защо не можем? Ако дефинираме `run` и просто го импортиме - няма да знаем кога точно е вмъкна тази функция по време на компилация. С други думи може не всички тестове в `@specs` да са акумулирани.

### __before_compile__

За целта ще използваме [`__before_compile__`](https://hexdocs.pm/elixir/1.0.5/Module.html). `__before_compile__` ни осигурява, че кодът ще се изпълни точно преди модула да бъде компилиран, иначе казано - всички тестове вече ще са били дефиниране и ще ги има в модулния атрибут `@specs`.

```
defmodule SpecRunner do
  defmacro __using__(_options) do
    quote do
      # извикай SpecRunner.__before_compile__(env) преди да се компилира дадения модул.
      @before_compile unquote(__MODULE__)
    end
  end

  # Тук е идеалното място да вкараме нашата функция `run` в ExampleTests
  defmacro __before_compile__(_env) do
    quote do
      def run do
        @specs
        |> Enum.each(fn {spec_func, spec_desc} ->
          IO.puts "Running spec: #{spec_desc}"
          case apply(__MODULE__, spec_func, []) do
            # Връщаме точка когато сме окей.
            :ok -> IO.puts "."
            # Прихващаме грешки.
            {:error, reason} -> IO.puts "Failure in #{spec_desc}: #{reason}"
          end
          IO.puts ""
        end)
      end
    end
  end
end
```

и не забравяме да използваме `SpecRunner` в `Specs`, като променим `__using__` така:

```
defmodule Specs do
  defmacro __using__(_options) do
    quote do
      use Assertion
      use SpecRunner

      @specs []

      import unquote(__MODULE__)
    end
  end

  # ... spec ...
end
```

И вече можем да се радваме на крайния резултат:

```
iex(7)> defmodule ExampleTests do
...(7)>   use Specs
...(7)>
...(7)>   spec "Test success" do
...(7)>     assert 1 == 1
...(7)>   end
...(7)>
...(7)>   spec "Test failure" do
...(7)>     assert 1 == 2
...(7)>   end
...(7)> end
{:module, ExampleTests, ...
iex(8)> ExampleTests.run
Running spec: Test failure
Failure in Test failure: Expected the left side 1 to be equal to the right side 2

Running spec: Test success
.

:ok
```
