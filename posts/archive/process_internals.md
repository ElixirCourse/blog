---
title_image_path: internals.png
category: Програма
created_at: 2017-04-24T21:50:52
tags:
  - elixir
  - message queue
  - process
  - garbage collection
  - GC
  - actors
  - receive
  - after
  - state
  - messages
  - recursion
---

# Устройство и комуникация между процеси

Продължаваме по темата 'процеси и `Elixir`'. След като разгледахме защо и как
са се появили в `Erlang`, как се създават и как си комуникират, сега е време
да поговорим за тяхната имплементация и по-подробно да поговорим за комуникацията между тях.

## Actor модела и Elixir процесите

Да разгледаме `Actor` модела за конкурентност. Както казахме той е повлиян
от препращането на съобщения в `Smalltalk` и `Simula`. Създаден е от _Carl Hewitt_ през 1973 година.
Идеята е следната:
* Всичко е `Actor`. Подобно на обектно-ориентираната идеология : 'всичко е обект'.
* Всеки `Actor` чака за съобщения. Когато получи съобщение, той може:
    * Да изпрати краен брой съобщения към други `Actor`-и.
    * Да създаде краен брой нови `Actor`-и.
    * Да определи поведение, което ще се изпълни, когато получи следващо съобщение, адресирано към него.

Тези съобщения се предават асинхронно. Всеки `Actor` си има адрес, понякога наричан поща (`mail box`).
По този начин `Actor`-и могат да комуникират само и единствено когато знаят адресите си.
Важно е да се отбележи, че горните поведения нямат определен ред и могат да се извършват паралелно.

Всичко това много прилича на модела на комуникация и поведение на процесите в `Elixir`.
Именно затова често ще чуем или прочетем, че `Erlang/Elixir` имплементират или са базирани на `Actor` модела.
Истината е че и `Erlang` и `Actor` модела са повлияни от едни и същи идеи.

Разликите са следните:
* В `Elixir` не всичко е процес. Всяко парче код се изпълнява в процес, но типовете данни не са процеси. Както казахме има няколко слоя на езика. Функционалният слой който се изпълнява в процесите няма нищо общо с `Actor` модела.
* В `Elixir` при получаване на съобщения, кодът в самия процес е последователен.

Приликите са следните:
* Процесите в `Elixir` се държат като `Actor`-и.
    * Чакат за съобщения от други процеси и реагират на тях.
    * При получено съобщение, могат да изпратят нови съобщения до други процеси.
    * При получено съобщение, могат да създадат краен брой други процеси.
    * При получено съобщение, могат да заложат поведение за следващо съобщение.
* Процесите в `Elixir` имат опашка за получените съобщения - `mail box`.
* Процесите в еликсир имат адреси (`PID`) и само процеси, знаещи адресите си могат да комуникират помежду си.

В заключение, можем да кажем че `Elixir` процесите са `Actor`-и,
но кодът който се изпълнява в тях е функционален и не е базиран на `Actor` модела.

## Устройство на процес

Ще използваме следната диаграма:

<br />
![alt text](/custom/assets/process.png "Устройство на процес") {: .inline-image}
<br />

Можем да кажем че процесът е изграден от следните части:

### Контролен блок

Тук се държи адреса на процес, `PID`-а. Също и състояние - дали чака или пък се изпълнява в момента.
Възможно е да реферираме процес и по име, което също се държи тук. Пример:

```elixir
defmodule Responder do
  def run do
    Process.register(self(), :responder)

    receive do
      {pid, :ping} when is_pid(pid) -> send(pid, :pong)
      {pid, anything} when is_pid(pid) -> send(pid, "I received #{anything}.")
    end
  end
end
```

Сега можем да направим това:

```elixir
spawn(Responder, :run, [])

send(:responder, {self(), :ping})

receive do
  :pong -> IO.puts("PONG!")
end
# PONG!
```

Можем да видим всички регистрирани имена с `Process.registered/0`.

В контролния блок се държат още характеристики на процеса. Можем да ги видим
с `Process.info/1`.

```elixir
spawn(Responder, :run, [])
:responder |> Process.whereis |> Process.info

[
  registered_name: :responder, current_function: {Responder, :run, 0},
  initial_call: {Responder, :run, 0}, status: :waiting, message_queue_len: 0,
  messages: [], links: [], dictionary: [], trap_exit: false,
  error_handler: :error_handler, priority: :normal, group_leader: #PID<0.50.0>,
  total_heap_size: 233, heap_size: 233, stack_size: 2, reductions: 3,
  garbage_collection: [
    max_heap_size: %{error_logger: true, kill: true, size: 0},
    min_bin_vheap_size: 46422, min_heap_size: 233, fullsweep_after: 65535,
    minor_gcs: 0
  ],
  suspending: []
]
```

### Stack

Всеки процес си има собствен стек на изпълняващи се функции, техните параметри
и `return` адреси. При създаването на процеса този стек е изключително малък,
но може да расте. Както виждате на диаграмата, докато има свободно място, `stack`-ът
може да се разширява надолу.

### Heap

При стартиране на процеса, `heap`-ът му също е малък, но може да се разширява
нагоре, докато има свободно пространство.

Тук се намира и опашката от идващи съобщения на процеса. Тук се пазят непроменимите
структури като списъци, кортежи, както и числа с плаваща запетая, малки `binary`-та (под 64 байта).
За по-големите, `Refc binary`-а се пазят само указателите `ProcBin`.
В [`binaries`](/posts/binaries) описахме как работи тази споделена памет, сочена
от тези указатели.

### Опашка от идващи съобщения

Казахме, че така наречената пощенска кутия на процеса е опашка.
Тази опашка може да се разширява, докато има памет за това. В опашката са
съобщенията, които процесът е получил в реда на пристигането си.

Ако няма съобщения в опашката, `receive` блокира процеса и чака, докато се получи поне едно ново съобщение.

Когато пристигне ново съобщени, то се слага в тази опашка. Когато процесът стане
активен, съобщението се `match`-ва към условията в `receive`. Ако има успех, то
се премахва от опашката.

Ако съобщението не успее да се `match`-не, то се запазва за изчакване.
Този алгоритъм се повтаря за следващото съобщение и така, докато опашката стане
празна. В този момент изчакващите съобщения се връщат в опашката, и ще бъдат
съпоставени на клаузите в `receive` при получаване на следващо съобщение.

Колкото повече такива не-`match`-нати съобщения се застоят в опашката,
толкова по-бавен ще става `receive` алгоритъмът, защото при ново съобщение
винаги ще пробва старите първо. Освен това, те ще заемат място в паметта на процеса.

```elixir
spawn(Responder, :run, [])

1..200 |> Enum.map(fn n -> send(:responder, "junk#{n}") end)

:responder |> Process.whereis |> Process.info(:messages)
# {:messages, [...]}
```

В този пример изпращаме `200` нежелани съобщения към `:responder`.
Можем да ги видим като използваме `Process.info(pid, :messages)`.

```elixir
send(:responder, {self(), :ping})
```

При получаване на горното, очаквано от `:responder` съобщение, всички тези `200`
съобщения-боклук, ще трябва да бъдат `match`-нати първо, което не е хубаво.
Именно затова е хубаво да знаете какво очакват процесите, към които изпращате съобщения.
И да не им изпращате неща, които никога няма да се обработят.

Начин за справяне с нежелани съобщения е:

```elixir
defmodule Responder do
  def run do
    Process.register(self(), :responder)

    receive do
      {pid, :ping} when is_pid(pid) -> send(pid, :pong)
      {pid, anything} when is_pid(pid) -> send(pid, "I received #{anything}.")
      anything -> IO.puts("Unexpected message received : #{anything}")
    end
  end
end

spawn(Responder, :run, [])

send(:responder, "junk")
# Ще видим 'Unexpected message received : junk'
```

Понякога обаче, `receive` блокът на процес може да се променя с получаване
на нови съобщения. Тогава не знаем какво ще бъде `match`-нато в бъдеще и е
добре да си пазим не-`match`-натите съобщения. Което ни напомня, че не сме
разглеждали процеси, които могат да `match`-нат повече от едно съобщение, преди
да завършат съществуването си.

## Процеси очакващи множество съобщения

След като съобщение е `match`-нато в `receive`, кодът асоцииран с `match`-натата
клауза се изпълнява. В този момент процесът излиза от `receive` и тъй като няма повече
логика, прекратява съществуването си.

Това значи, че ако искаме процес да започне да чака за ново съобщение, след като
е `match`-нал текущото, трябва пак да сложим `receive`.


```elixir
defmodule Responder do
  def run do
    wait()
    wait()
  end

  defp wait do
    receive do
      {pid, :ping} when is_pid(pid) -> send(pid, :pong)
      {pid, anything} when is_pid(pid) -> send(pid, "I received #{anything}.")
      anything -> IO.puts("Unexpected message received : #{anything}")
    end
  end
end

pid = spawn(Responder, :run, [])

Process.alive?(pid)
# true

send(pid, {self(), "Hey!"})
receive do
  msg -> IO.puts(msg)
end
# Ще видим 'I received Hey!'

Process.alive?(pid)
# true

send(pid, {self(), :ping})
receive do
  msg -> IO.puts(msg)
end
# Ще видим 'pong'

Process.alive?(pid)
# false
```

Примерът не е сложен. `Process.alive?/1` връща `true`, ако процесът с даден `PID` съществува,
иначе връща `false`.
След като изпратим първото съобщение, процесът продължава да съществува и очаква второ.
След второто успешно `match`-нато съобщение, процесът излиза от `run` и прекратява съществуването си.

Като използваме рекурсия можем да имаме дълго живеещи процеси:

```elixir
defmodule Responder do
  def run do
    receive do
      {pid, :ping} when is_pid(pid) ->
        send(pid, :pong)
        run()
      {pid, anything} when is_pid(pid) ->
        send(pid, "I received #{anything}.")
        run()
      anything ->
        IO.puts("Unexpected message received : #{anything}. Exiting!")
    end
  end
end
```

По този начин процесът ще може да отговаря на съобщения постоянно.
Ако получи съобщение, което не е очаквал, ще прекрати изпълнението си.
Тук се възползваме от `tail call` оптимизацията вградена в `Elixir`.

```elixir
1..10
|> Enum.map(fn _ -> send(pid, {self(), :ping}) end)
|> Enum.each(fn _ -> receive do msg -> IO.puts(msg); end end)

# Ще видим 'pong' 10 пъти

send(pid, "Bye")
# Unexpected message received : Bye. Exiting!
Process.alive?(pid)
# false
```

## Използване на `receive` с `timeout`

Нека да направим процес, който на съобщение - положително число, ни връща `N`-тото
число на _Фибоначи_.

```elixir
defmodule Fibonacci do
  def run do
    receive do
      {pid, n} when is_pid(pid) and is_number(n) and n > 0 ->
        send(pid, nth(n))
      {pid, _} when is_pid(pid) ->
        send(pid, "I CAN'T COMPILE!")
      anything ->
        IO.puts(:stderr, "Bad query #{anything}")
    end

    run()
  end

  defp nth(1), do: 1
  defp nth(2), do: 1
  defp nth(n), do: nth(n - 1) + nth(n - 2)
end
```

Функцията `run` е предназначена за стартиране на процес.
Чака се за съобщение. Ако съобщението е `PID` и положително число се смята
резултата от `nth/1` и се праща на процеса с `PID`, подадения `pid`. В другите
случаи сме уведомени че заявката не е валидна. С рекурсия постигаме `run loop` и
процесът ще може да работи при множество извиквания. Нека да видим:

```elixir
pid = spawn(Fibonacci, :run, [])

send(pid, {self(), 10})
receive do msg -> IO.puts(msg); end
# 55
```

Работи! Сега нека да пробваме с по-голямо число - `50`:

```elixir
send(pid, {self(), 50})
receive do msg -> IO.puts(msg); end
```

Тази заявка отнема доста време. Текущият процес забива на `receive`.
Има начин да напишем `receive` така, че ако чакането отнеме прекалено много
време да се изпълни някакъв код и да излезем от `receive`:

```elixir
send(pid, {self(), 50})
receive do
  msg -> IO.puts(msg)
after
  3000 -> IO.puts("Tired of waiting. Bye!")
end
```

Така след `3` секунди (времето е в милисекунди) текущият процес ще се откаже да чака,
ще изпълни кода в `after` и ще излезе от `receive`.

Между другото `after` работи и с атома `:infinity`. Поведението е
абсолютно същото като в случая, в който въобще не слагаме `after`.

С `receive` и `after` можем да имплементираме `sleep` функция:

```elixir
sleep = fn(time) ->
  receive do
  after
    time -> :ok
  end
end

sleep.(2000)
# Текущият процес ще забие за 2 секунди.
```

По този начин е имплементирана `:timer.sleep/1`, която пък се ползва от `Process.sleep/1`.

Друг интересен случай е когато подадем `0` на `after`. Така можем да имплементираме
`recieve`, който извлича всички съобщения на процеса от опашката със съобщения, и ако е празна,
не забива:

```elixir
defmodule Flusher do
  def flush_it do
    receive do
      msg ->
       IO.puts(msg)
       flush_it()
    after
      0 -> :ok
    end
  end
end

pid = spawn(Fibonacci, :run, [])
1..10 |> Enum.each(fn n -> send(pid, {self(), n}) end)
Flusher.flush_it
# Ще видим числата на Фибоначи до 10-тото и процесът няма да забие
```

Всъщност има такава помощна функция в `iex`, която се казва `flush/0`.

## Освобождаване на паметта на процесите (GC)

Всеки процес има свое собствено освобождаване на паметта. Отделно, както сме
споменавали, има обща памет за `Refc` двоичните данни, която си има собствен
`Reference Counting Garbage Collection` алгоритъм.

### GC на `heap`-а на процесите

Освобождаването на паметта (GC) за `heap`-овете на процесите е главно `generational`.
Това значи, че `Garbage Collector`-ът разделя паметта на две поколения - старо и ново.
Това разделение е базирано на идеята, че ако обект в паметта остане след цикъл на `GC`-то,
то шансът да бъде премахнат скоро е малък.

Новото поколение се състои от скоро-създадена информация, а старото от информация,
която не е била премахната след даден брой цикли на `GC`.
Това разделени помага на `GC`, като се намаляват ненужни цикли върху старото поколение информация.

`GC`-то на процесите в `Elixir` използва две стратегии - `generational`, която споменахме и `fullsweep`.
Както споменахме, `generational` стратегията работи само върху новата генерация от обекти в паметта,
докато `fullsweep` стратегията работи върху всички обекти.

Нека разгледаме алгоритъма на освобождение на паметта:

#### Случай 1:
При кратко живеещи процеси, които не използват `heap`, по-голям от `min_heap_size`
стойността на `VM`, няма освобождаване на паметта.
Когато тези процеси 'умрат', цялата паметта е освободена от `GC`.

#### Случай 2:
При процес, който използва памет повече от `min_heap_size`, имаме следното поведение:
1. Процесът е създаден (`spawn`)
2. Първоначално се използва `fullsweep GC` стратегия, тъй като още нямаме ново и старо поколение.
3. На този етап имаме нови и стари поколения, използва се `generational GC`.
4. Повтаряме (3)
5. Процесът се унищожава и паметта му е изчистена.

```
spawn -> fullsweep GC -> generational GC -> generational GC -> ...  -> end
```

Този случай е валиден за процеси, които не живеят извънредно дълго.

#### Случай 3:
Когато процес е активен за прекалено дълго време, `fullsweep` стратегията може да се
активира отново. Това се случва след определен брой `generational GC` цикли. Има флаг
за този брой - `fullsweep_after`. Когато брояч наречен `minor_gcs`, който се увеличава на всеки цикъл,
стигне този брой имаме `fullsweep GC` цикъл.

Други случаи, когато се изпълнява `fullsweep GC`, са когато процесът не е способен
да си освободи достатъчно памет при нужда или когато извикаме `:erlang.garbage_collect(pid)`.

```
spawn -> fullsweep GC -> generational GC -> generational GC -> ... -> fulswwep GC -> ... -> end
```

Тези флагове и стойности могат да се видят за даден процес така:
```elixir
Process.info(pid, :garbage_collection)
{
  :garbage_collection,
  [
    max_heap_size: %{error_logger: true, kill: true, size: 0},
    min_bin_vheap_size: 46422, min_heap_size: 233, fullsweep_after: 65535,
    minor_gcs: 0
  ]
}
```

#### Случай 4:
Когато се наложи `fullsweep GC`, защото няма достатъчно памет и се окаже,
че `GC` не може да освободи нужната памет, `heap`-ът на процеса се увеличава.

В заключение, това че всеки процес си има собствен `GC` прави `Elixir` толкова
`responsive` език. Обикновено `Garbage Collector`-ите на процесите работят с малко данни.
Нищо не ни пречи да използваме само един процес и да натоварваме `Garbage Collector`-а му.
Идеологията около `Erlang/Elixir` поощрява ползването на множество процеси и разделянето на програмата
на малки, комуникиращи си компоненти.

### GC на refc паметта

В тази памет `Garbage Collector`-ът изчиства само обекти чиито референции са нула.
Това е много бърза стратегия, защото е лесно да се отделят обектите за изчистване.
Точно в тази памет, обаче, са възможни `memory leak`-ове по доста лесен начин.

Преди споменахме, че `sub-binary` обектите не са копия на `refc binary` обектите,
а просто указатели, но те също увеличават `reference counter`-а на двоичната структура.
Трябва да се внимава с тях, защото заради някакъв малък низ, част от прочетен в паметта файл,
е възможно целият файл да престои прекалено дълго в паметта.

Друг опасен случай е, когато имаме някой много лек, но дълго-съществуващ процес,
през който е преминало огромно `binary`. Даже и да не се ползва това `binary` в него,
ако данните в паметта му са малко, `Garbage Collection` няма да се случи. Не забравяйте, че той
държи само един много малък и евтин обект - `ProcBin`-указателя в паметта си.
Заради този указател, обаче, цялото `refc binary` може да се задържи за дълго, даже вечно.

Тази споделена памет е важна, най-често съобщенията между процеси са, или включват низове.
Ако тези низове са големи, те не се копират от памет на процес в памет на процес. Само нов
`ProcBin` се създава в процеса-получател. Това е много добра оптимизация, но трябва да бъдем внимателни.

Това е всичко за сега. В следващата статия ще говорим за връзки между процеси и наблюдение между процеси.
Ще си поговорим и какво става, когато възникнат грешки при изпълнение в процеси.