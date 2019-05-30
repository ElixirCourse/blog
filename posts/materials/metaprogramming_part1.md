---
title_image_path: metaprogramming.jpg
category: Програма
author: valo
tags:
  - elixir
  - metaprogramming
---

# Meta-програмиране в Elixir част 1

Какво всъщност е мета програмиране?
Накратко, това е възможноста да пишем код, който пише код.
Тъй като това е една от онези дефиниции, които не успяват много успешно да обяснят за какво точно става въпрос, нека да видим няколко примера за приложения на мета програмирането.

## Автоматизирано създаване на функции

Мета програмирането позволява да се автоматизира генерирането на множество функции с подобна или една и съща имплементация. Нека например си представим как би изглеждало генерирането на HTML като код на Elixir:

```elixir
html do
  head do
    title do
      text "Hello To Our HTML DSL"
    end
  end
  body do
    h1 class: "title" do
      text "Introduction to metaprogramming"
    end
    p do
      text "Metaprogramming with Elixir is really awesome!"
    end
  end
end
```

Принципно нищо не ни пречи да имплементираме горните функции без мета програмиране, но ще трябва за всеки HTML таг да напишем функция, която да прави едно и също. Много по-лесно би било да имаме списък от всички тагове в един файл и от него да генерираме всички нужни функции. Това може да стане лесно с мета програмиране и няма да ни е нужен никакъв допълнителен инструмент, освен Elixir. В други езици, подобен проблем обикновенно се решава чрез инструменти като Makefiles и допълнителни скриптове.

## Дефиниране на DSL-и

Мета програмирането позволява да дефинираме нови "езици", които решават някакъв специфичен проблем. Нека например видим библиотеката Ecto, която позволява да се пишат SQL заявки като код на Elixir:

```elixir
from o in Order,
where: o.created_at > ^Timex.shift(DateTime.utc_now(), days: -2)
join: i in OrderItems, on: i.order_id == o.id
```

Горният код ще генерира SQL от следният вид:

```SQL
SELECT o.*
FROM orders o
JOIN order_items i ON i.order_id = o.id
WHERE o.created_at > '2017-04-28 14:15:34'
```

Както виждате можем да пишем SQL код директно в Elixir! Забележете, че това е синтактично валиден код, но без да се пренапише няма да може да се компилира. Тук дори и да искаме няма да може да се разминем от мета програмирането за да постигнем горният синтаксис. Това което макроса `from` прави е да пренапише аргументите, които му се подават към специална структура, която после се използва за да се конструира финалният SQL. Преимуществата на горния подход, са че заявките могат много лесно да се разделят на малки, части които да се комбинират:

```elixir
def orders(from_date) do
  from o in Order,
  where: o.created_at > ^from_date
  join: i in OrderItems, on: i.order_id == o.id
end

def user_orders(from_date, user_id) do
  from o in orders(from_date),
  where: o.user_id == ^user_id
end
```

Както виждате във функцията `user_orders`, където искаме да имаме допълнително филтриране по потребителя направил заявката, можем да преизползваме функцията `orders` и единствено да добавим филтрирането по потребител.

Използването на DSL език за заявки към базата данни има и други преимущества:

* Автоматично санитизиране на данните и защита от SQL injections
* Валидиране на заявките по време на компилация
* По-лесна поддръжка на различни бази данни

## Въведение в Abstract Syntax Tree

Мета програмирането в Elixir става чрез дефинирането на макроси. Това са функции, които приемат като аргументи код и връщат код като резултат, като кода е във формата на Abstract Syntax Tree или AST.

Почти всеки един език за програмиране по един или по друг начин използва AST, но в много случаи потребителите на езика нямат достъп до него. Обикновенно AST се използва като междинно представяне преди кода да бъде компилиран до машинни инструкции. В Elixir обаче, не само че имаме достъп до AST, но и това AST е представено чрез стандартните структури от данни, с които вече се запознахме.

Реално мета програмирането се извършва чрез манипулация на AST-то генерирано по време на компилация и затова е и много важно да запомним, че мета програмирането се извършва **само по време на компилация**. Това значи, че веднъж компилиран, кода не може да бъде променян, за разлика от Ruby например. Това може да звучи, като сериозно ограничение, но реално се оказва, че е напълно достатъчно в повечето случаи и при всички положения подобрява бързината на езика многократно. Все още не съм установил ситуация, където да ми е трябвала манипулация на кода по време на изпълнение и мисля, че авторите на езика са направили много добър trade-off.

Нека да разгледаме малко примери за това как изглежда AST. За да получим AST на някакъв Elixir код, можем да използваме макроса `quote`. Например:

```elixir
iex> quote do: 1 + 2
{:+, [context: Elixir, import: Kernel], [1, 2]}
iex> quote do: div(10, 2)
{:div, [context: Elixir, import: Kernel], [10, 2]}
```

Както виждате, AST представлява кортежи от тройки, които имат следният формат:

```
{<име на функция>, <контекст>, <списък от аргументи>}
```

Този формат може много да ви напомни на Lisp и ще сте напълно прави. Реално в Lisp, кода се представя по много подобен начин и от там идва и идеята, че в Lisp кода всъщност са просто данни. За разлика от Lisp обаче, в Elixir имаме приятен синтаксис, който се преобразува в AST от компилатора и само ако искаме да правим мета програмиране се налага да работим със странния "префиксен" начин да представяне на код.

Да видим някои по-сложни примери:

```elixir
iex> quote do: 1 + 2 * 3
{:+, [context: Elixir, import: Kernel],
 [1, {:*, [context: Elixir, import: Kernel], [2, 3]}]}
```

Ако разпишем горното дърво ще видим, че приоритета на операциите е правилен:

```elixir
{:+, _, [
  1,
  {:*, _, [2, 3]}
]}
```

Както виждате умножението е в отделно под-дърво от събирането. Да видим как би изглеждало AST-то на HTML езика, който разгледахме по-рано:

```elixir
iex> quote do
...> html do
...>   head do
...>     title do
...>       text "Hello To Our HTML DSL"
...>     end
...>   end
...> end
...> end
{:html, [],
 [[do: {:head, [],
    [[do: {:title, [], [[do: {:text, [], ["Hello To Our HTML DSL"]}]]}]]}]]}
```

Тъй като нашият DSL е валиден Elixir код, то той може да бъде компилиран до AST. Респективно ние можем да модифицираме това AST към друго валидно AST, което вече компилатора ще преобразува във BEAM byte code. Нека да видим как става това модифициране на AST-то.

## Въведение в макросите

Макросите са функции, които приемат като аргумент AST и връщат AST. Нека да разгледаме един най-прост пример: да дефинираме макрос, който обръща плюс с минус и умножение с деление.

```elixir
defmodule MathChaosMonkey do
  defmacro swap_ops(do: {:+, context, arguments}) do
    {:-, context, arguments}
  end

  defmacro swap_ops(do: {:-, context, arguments}) do
    {:+, context, arguments}
  end

  defmacro swap_ops(do: {:/, context, arguments}) do
    {:*, context, arguments}
  end

  defmacro swap_ops(do: {:*, context, arguments}) do
    {:/, context, arguments}
  end
end
```

Нека сега да тестваме нашия макрос:

```elixir
iex> MathChaosMonkey.swap_ops do
...> 1 + 2
...> end
-1
iex> MathChaosMonkey.swap_ops do
...> 10 * 2 + 1
...> end
19
iex> MathChaosMonkey.swap_ops do
...> 10 * 2
...> end
5.0
```

Както виждаме успяхме да сътворим пълна бъркотия в аритметичните операции. Нещо, което е важно да се отбележи е че за разлика от езици като Ruby, макросите имат много ясно поле на действие, т.е. няма как да направим глобална промяна във runtime-а. Всички макроси имат ефект единствено в модулите, които са require-нати и използвани.

Примерът по-горе е доста опростен и напълно безполезен в реални условия. За да можем да дефинираме използваеми макроси по лесен начин, има помощни функции, които ни позволяват да работим без да слизаме на ниво AST структурата. Това е функцията `unquote`, която взема AST структура и я интерпретира в текущия контекст. Нека да разгледаме един пример:

```elixir
iex> value = 12
12
iex> quote do
...> 1 + 2 * value
...> end
{:+, [context: Elixir, import: Kernel],
 [1, {:*, [context: Elixir, import: Kernel], [2, {:value, [], Elixir}]}]}
iex> quote do
...> 1 + 2 * unquote(value)
...> end
{:+, [context: Elixir, import: Kernel],
 [1, {:*, [context: Elixir, import: Kernel], [2, 12]}]}
```

Нека да разгледаме друг пример, които ще дефинира функции за умножение на числа по някаква стойност:

```elixir
defmodule Multiplier do
  defmacro of(value) do
    quote do
      def unquote(:"multiplier_#{value}")(expr) do
        expr * unquote(value)
      end
    end
  end
end

defmodule Math do
  require Multiplier

  Multiplier.of(5)
end

Math.multiplier_5(2) # => 10
```

Както виждате успяхе да напишем макрос, който да генерира функция, която има поведение зависещо от аргументите, които подадохме на макроса.

## Дефиниция на while

Нека да разгледаме един по-интересен пример: да дефинираме `while` цикъл. Това ще рече, да подкараме следният код в Elixir:

```elixir
defmodule Fib do
  def async_fib(n) do
    spawn(fn -> fib(n) end)
  end

  def sync_fib(n) do
    pid = async_fib(n)
    while(Process.alive?(pid)) do
      IO.puts "Waiting..."
      sleep(1)
    end
    IO.puts "Done!"
  end

  defp fib(0), do: 0
  defp fib(1), do: 1
  defp fib(n), do: fib(n-1) + fib(n-2)
end
```

```elixir
defmodule Loops do
  defmacro while(expression, do: block) do
    quote do
      try do
        for _ <- Stream.cycle([:ok]) do
          if unquote(expression) do
            unquote(block)
          else
            throw :break
          end
        end
      catch
        :break -> :ok
      end
    end
  end
end

defmodule Fib do
  require Loops

  def async_fib(n) do
    spawn(fn -> fib(n) end)
  end

  def sync_fib(n) do
    pid = async_fib(n)
    Loops.while(Process.alive?(pid)) do
      IO.puts "Waiting..."
      Process.sleep(1000)
    end
    IO.puts "Done!"
  end

  defp fib(0), do: 0
  defp fib(1), do: 1
  defp fib(n), do: fib(n-1) + fib(n-2)
end

iex> Fib.sync_fib(40)
Waiting...
Waiting...
Waiting...
Waiting...
Waiting...
Waiting...
Waiting...
Done!
:ok
```

Както виждате успяхме да създадем `while` конструкция, която е подобна на циклите в другите езици. Добре е да се отбележи, че следният код няма да работи:

```elixir
defmodule Bottles do
  def sing(n) do
    i = 0
    while(i < n) do
      IO.puts "#{n} bottles hanging on the wall"
      IO.puts "If one bottle crashes on the floor, there will be..."
      i = i - 1 # Won't change the binding in the condition of the loop
    end
    IO.puts "No bottles hanging on the wall"
  end
end
```

Тъй като променливите има строго дефиниран scope и данните са неизменими, в горния пример `i = i - 1` няма да промени условието `i < 10`, тъй като `unquote` ще вземе стойността на `i` преди `while` макроса и ако променим тази стойност вътре в болка, то тя няма да се отрази при втория цикъл. За да илюстрираме по-ясно това нека да пренапишем горния макрос с рекурсия:

```elixir
defmodule Loops do
  defmacro while(expr, do: block) do
    quote do
      Loops.run_loop(fn -> unquote(expr) end, fn -> unquote(block) end)
    end
  end

  def run_loop(expr_body, loop_body) do
    case expr_body.() do
      true ->
        loop_body.()
        run_loop(expr_body, loop_body)
      _ ->
        :ok
    end
  end
end
```

В горната имплементация си разделяме цикъла на условие и на тяло и ги "обвиваме" във функции, за да можем да ги изпълняваме когато искаме. Всяка от тази функции си има локален binding на променливите с нея и този binding не може да бъде променян от външните функции.
