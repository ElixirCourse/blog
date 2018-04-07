---
title_image_path: links.jpg
category: Програма
created_at: 2017-05-09T21:33:57
tags:
  - elixir
  - link
  - process
  - monitor
  - unlink
  - system process
  - signals
  - kill
  - normal
  - messages
---

# Връзки между процеси

В тази статия ще си говорим за създаване на връзки между процеси. Ще видим и
как да наблюдаваме процеси, както и как да реагираме на грешки в процесите или
на завършването на логиката им.

## Когато процес 'умре'

Какво става когато един процес завърши изпълнението си?
Бива изчистен от паметта и просто спира да съществува.
Кой научава за това. По подразбиране никой.

Има начини процесите да са свързани помежду си, така че ако един от тях завърши изпълнението си,
другия да разбере за това. Нека имаме:

```elixir
defmodule Quitter do
  def run do
    Process.sleep(3000)

    exit(:i_am_tired)
  end
end
```

Нека да го пуснем, и след 4-5 секунди да проверим дали е жив:

```elixir
pid = spawn(Quitter, :run, [])

# След няколко секунди
Process.alive?(pid) #false
```

Това е нормално поведение, текущият процес не разбира кога `Quitter` процесът е завършил.
Нека сега стартираме със `spawn_link/3`:

```elixir
spawn_link(Quitter, :run, [])

# След 3 секунди -> ** (EXIT from #PID<0.202.0>) :i_am_tired
```

Това което се случва е, че `Quitter` процесът излиза, но с това убива и текущия процес.
Те са свързани. Това означава, че ако единият процес излезе с грешка, и другият ще излезе.

Разбира се ако единият процес завърши без грешка, няма да убие другия:

```elixir
defmodule Quitter do
  def run_no_error do
    Process.sleep(3000)
  end
end

pid = spawn_link(Quitter, :run_no_error, [])

# След няколко секунди
Process.alive?(pid) #false
```

Има начин за премахване на връзка - `Process.unlink/1`:

```elixir
pid = spawn_link(Quitter, :run, [])

Process.unlink(pid)

# Няна грешка
```

Подобно на това, можем да се свържем два съществуващи процеса с `Process.link/1`.

```elixir
defmodule Simple do
  def run do
    receive do
      {:link, pid} when is_pid(pid) ->
        Process.link(pid)
        run()
      :links ->
        IO.puts(inspect(Process.info(self(), :links)))
        run()
      :die ->
        IO.puts("#{inspect self()} is hit!")
        exit(:ouch)
    end
  end
end
```

Нека направим пет процеса със `Simple.run/0`:

```elixir
pid1 = spawn(Simple, :run, [])
#PID<0.270.0>
pid2 = spawn(Simple, :run, [])
#PID<0.272.0>
pid3 = spawn(Simple, :run, [])
#PID<0.274.0>
pid4 = spawn(Simple, :run, [])
#PID<0.276.0>
pid5 = spawn(Simple, :run, [])
#PID<0.278.0>
```

Нека сега да ги свържем така: 

```
pid1 -> pid2 -> pid3 -> pid4 -> pid5
```

```elixir
send(pid1, {:link, pid2}) # {:link, #PID<0.272.0>}
send(pid2, {:link, pid3}) # {:link, #PID<0.274.0>}
send(pid3, {:link, pid4}) # {:link, #PID<0.276.0>}
send(pid4, {:link, pid5}) # {:link, #PID<0.278.0>}
```

Да проверим връзките им:

```elixir
send(pid1, :links) # {:links, [#PID<0.272.0>]}
send(pid2, :links) # {:links, [#PID<0.270.0>, #PID<0.274.0>]}
send(pid3, :links) # {:links, [#PID<0.272.0>, #PID<0.276.0>]}
send(pid4, :links) # {:links, [#PID<0.274.0>, #PID<0.278.0>]}
send(pid5, :links) # {:links, [#PID<0.276.0>]}
```

Както споменахме по-горе, е видно че тези връзки са двупосочни. Това е по правилно:

```
pid1 <-> pid2 <-> pid3 <-> pid4 <-> pid5
```

Какво ще стане, когато убием `pid5`?

```elixir
send(pid5, :die) #PID<0.278.0> is hit!

Process.alive?(pid1) # false
Process.alive?(pid2) # false
Process.alive?(pid3) # false
Process.alive?(pid4) # false
Process.alive?(pid5) # false
```

Това означава, че грешката се разпространява транзитивно по `link`-овете.
Въпросът е какво представлява тази грешка и можем ли да реагираме от някой от
свързаните процеси на нея така, че той да не бъде ликвидиран?

## Съобщени при грешка

Всъщност тази грешка, която транзитивно убива свързаните процеси е специално
съобщение. Процеси могат да си комуникират само със съобщения.

Тези специални съобщения се наричат сигнали. `Exit` сигналите са 'тайни' съобщения,
които автоматично убиват процесите, които ги получат.

За да имаме `fault-tolerant` система е добре да имаме способ да убиваме процесите и
и да разбираме кога процес е бил ликвидиран. Тези `link`-ове вършат тази работа.
Второто много важно изискване за тази `fault-tolerant` система е да има начин да рестартира процеси,
когато те 'умрат'.

В `Elixir` има специален вид процеси - системни процеси. Това са нормални процеси,
които могат да трансформират `exit` сигналите, които получават в нормални съобщения.
По този начин можем да имаме процеси, които разбират кога други процеси се терминират.
Това са системни процеси, които държат `link`-ове.

## Системни процеси

Нормален процес може да стане системен с извикването на `Process.flag(:trap_exit, true)`:

```elixir
defmodule Quitter do
  def run do
    Process.sleep(3000)
    exit(:i_am_tired)
  end
end

Process.flag(:trap_exit, true)

spawn_link(Quitter, :run, [])
receive do msg -> IO.inspect(msg); end

# {:EXIT, #PID<0.95.0>, :i_am_tired}
```

И по точно този начин, ние можем да си построим `fault tolerant` система.
Един системен процес може да 'наблюдава' други процеси и да слуша за `exit` сигнали от тях.
Ако получи такъв сигнал, той може да реши да ги рестартира или да направи нещо по-умно.

## Грешки и поведение

Ще разгледаме при какви случаи на грешка или терминиране на процес, какво би било поведението на свързан процес.

```elixir
action = <?>
system_process = <?>

Process.flag(:trap_exit, system_process)
pid = spawn_link(action)

receive do
  msg -> IO.inspect(msg)
end
```

### Случай 1 : При `action = fn -> :nothing end`
1. При `system_process = false` : Нищо няма да се случи. Текущият процес продължава да чака.
2. При `system_process = true`  : Ще видим '`{:EXIT, <pid>, :normal}`' на екрана. Текущият процес продължава изпълнение.

При излизане без грешка, сигналът `EXIT` е със статус `:normal`. Този статус не убива не-системните свързани процеси

### Случай 2 : При `action = fn -> exit(:stuff) end`
1. При `system_process = false` : Текущият процес умира с грешка `** (EXIT from <pid>) :stuff`.
2. При `system_process = true`  : Ще видим '`{:EXIT, <pid>, :stuff}`' на екрана. Текущият процес продължава изпълнение.

### Случай 3 : При `action = fn -> exit(:normal) end`
1. При `system_process = false` : Нищо няма да се случи. Текущият процес продължава да чака.
2. При `system_process = true`  : Ще видим '`{:EXIT, <pid>, :normal}`' на екрана. Текущият процес продължава изпълнение.

Поведението е аналогично на _Случай 1_. По този начин можем да излезем нормално от процес ръчно.

### Случай 4 : При `action = fn -> raise("Stuff") end`
1. При `system_process = false` : Текущият процес умира с грешка `[error] Process <pid> raised an exception ** (RuntimeError) Stuff`.
2. При `system_process = true`  : Ще видим '`{:EXIT, <pid>, {%RuntimeError{message: "Stuff"}, [{:erl_eval, :do_apply, 6, [file: 'erl_eval.erl', line: 668]}]}}`' на екрана. Текущият процес продължава изпълнение.

### Случай 5 : При `action = fn -> throw("Stuff") end`
1. При `system_process = false` : Текущият процес умира с грешка `[error] Process <pid> raised an exception ** (ErlangError) erlang error: {:nocatch, "Stuff"}`.
2. При `system_process = true`  : Ще видим '`{:EXIT, <pid>, {{:nocatch, "Stuff"}, [{:erl_eval, :do_apply, 6, [file: 'erl_eval.erl', line: 668]}]}}`' на екрана. Текущият процес продължава изпълнение.

### Заключение

Тези пет поведения покриват всички възможни случаи. Както виждате, засега няма начин системния процес да бъде терминиран с какъвто и да е сигнал.
Истината, обаче е, че има начин. Има един много специален статус на `exit` сигнал, който може да убие и системен процес.

## Функцията `Process.exit/2`

Тази функция може да се използва, когато искаме от един процес да ликвидираме друг.

```elixir
defmodule HelloPrinter do
  def start_link do
    Process.sleep(5000)

    IO.puts("Hello")

    start_link()
  end
end

Process.flag(:trap_exit, true)

pid = spawn_link(HelloPrinter, :start_link, [])
# Започва да печата "Hello" на всеки 5 секунди

Process.exit(pid, :spri_se_be)
# Спира да печата Hello

receive do
  msg -> IO.inspect(msg)
end
# {:EXIT, #PID<0.162.0>, :spri_se_be}
```

По този начин можем да убием всеки не-системен процес. Нека сега да имаме:

```elixir
defmodule HiPrinter do
  def start_link do
    Process.flag(:trap_exit, true)

    print()
  end

  defp print do
    receive do
      {:EXIT, pid, _} -> send(pid, "Няма да стане, батка!")
    after
      6000 -> IO.puts("Hi!")
    end

    print()
  end
end
```

Това е системен процес. Той лови `exit` сигнали и изпраща съобщение, че не му
действат. На всеки 6 секунди ни казва `Hi!`. Искаме да го ликвидираме, но как?
Да видим:

```elixir
pid = spawn_link(HiPrinter, :start_link, [])
# Почва да печата `Hi!`

exit(pid, :spri)
# Продължава да печата `Hi!`

receive do
  msg -> IO.inspect(msg)
end
# 'Няма да стане, батка'

Process.exit(pid, :normal)
# Продължава да печата `Hi!`

receive do
  msg -> IO.inspect(msg)
end
# 'Няма да стане, батка'

Process.exit(pid, :kill)
# Спира да печата!

receive do
  msg -> IO.inspect(msg)
end
# {:EXIT, #PID<0.175.0>, :killed}
```

И така `exit` с атома `kill`, може да убие всякакъв процес, даже системен.
И съобщението има статус `killed`.

Забелязвате че статусът от `kill` стана `killed`. Това е добре. Нашият процес
беше свързан с този, който 'убихме', а не искаме сигналът ни да рикушира обратно.
Сигналът `kill` не е транзитивен към връзките.

Нека сега да изследваме ситуациите, които могат да се получат, когато използваме `Process.exit/2`.

## Ликвидиране и поведение

Имаме:

```elixir
action = <?>
system_process = <?>
status = <?>

Process.flag(:trap_exit, system_process)
pid = spawn_link(action)

Process.exit(pid, status)

receive do
  msg -> IO.inspect(msg)
end
```

### Случай 1 : При `action = fn -> Process.sleep(20_000) end` и `status = :normal`
1. При `system_process = false` : Процесът с pid `pid` умира след 20 секунди със статус `:normal`. Текущият процес не е системен и не получава съобщение, затова продължава да чака неопределено време да получи някакво съобщение в `receive`.
2. При `system_process = true`  : След 20 секунди получаваме съобщение `{:EXIT, <pid>, :normal}`.

### Случай 2 : При `action = fn -> Process.sleep(20_000) end` и `status = :stuff`
1. При `system_process = false` : Грешка : `** (EXIT from <pid>) :stuff`. Текущият процес 'умира'.
2. При `system_process = true`  : Получаваме съобщение `{:EXIT, <pid>, :stuff}`.

### Случай 3 : При `action = fn -> Process.sleep(20_000) end` и `status = :kill`
1. При `system_process = false` : Грешка : `** (EXIT from <pid>) killed`. Текущият процес 'умира'.
2. При `system_process = true`  : Получаваме съобщение `{:EXIT, <pid>, :killed}`.

### Случай 4 : При `action = fn -> Process.exit(self(), :kill) end` и `status = <каквото-и-да-е>`
1. При `system_process = false` : Грешка : `** (EXIT from <pid>) killed`. Текущият процес 'умира'.
2. При `system_process = true`  : Получаваме съобщение `{:EXIT, <pid>, :killed}`.

### Случай 5 : При `action = fn -> exit(:kill) end` и `status = <каквото-и-да-е>`
1. При `system_process = false` : Грешка : `** (EXIT from <pid>) killed`. Текущият процес 'умира'.
2. При `system_process = true`  : Получаваме съобщение `{:EXIT, <pid>, :kill}`. Странно, нали?

`exit(:reason)` е различен от `Process.exit(<pid>, :reason)`.
`exit(:reason)` е нещо като `throw`, предизвиква 'хвърляне', което, ако не е хванато, 'убива' текущия процес.
Когато два процеса са свързани, от тази грешка се произвежда сигнал със съобщение `:reason`, който ги 'убива'.
Ако тази причина е `:kill`, тя се променя на `:killed` за да не убива системни процеси, които са свързани.
Системните процеси прихващат сигнала.
В случай 5, `exit(:kill)` може да бъде хванат с `catch` в процеса-източник, който го вика, но не е хванат, затова процесът 'умира' с причина `:kill`.
Свързаният системен процес получава сигнал `:killed` със съобщение оригиналната `exit` причина - `:kill`, а не `kill`-сигнал.

Разбира се е възможно да изпращаме сигнали до себе си. Да речем `Process.exit(self(), :kill)`, ще ликвидира
текущия процес, независимо дали е системен.

## Наблюдаване на процеси

Свързването на процеси е двупосочно.
Ако единият от тях 'умре' другият ще получи `EXIT` сигнал.
Този сигнал ще 'убие' всички свързани процеси, ако те не са системни.

Често искаме един процес да наблюдава друг без да му праща сигнал, ако случайно 'умре'.
Също така искаме да наблюдаваме процеси без текущият процес да е системен.
Ако те 'умрат', просто искаме да бъдем нотифицирани с нормално съобщение, за да предприемем нещо.

Това е възможно с монитори:

```elixir
pid = spawn(fn -> Process.sleep(3000) end)

Process.monitor(pid)

receive do
  msg -> IO.inspect(msg)
end
# След 3 секунди ще получим нещо такова
# {:DOWN, #Reference<0.0.3.3915>, :process, #PID<0.348.0>, :normal}
```

Когато наблюдаваме процес чрез монитор, получаваме `DOWN` съобщение ако процесът
прекрати изпълнението си или не съществува.

Добавяме монитор към процес с `Process.monitor(pid)`. Тази фунцкия връща референция.
Референциите са специален тип в `ELixir`, всяка от тях е уникална за текущия `node`.
Когато получим `DOWN` съобщението, тази референция ще е втория му елемент.
Четвъртият е `PID`-а на процеса, който е завършил изпълнението си, а петият - статус.
Това са същите тези статуси, като при сигналите при свързани процеси.

Наблюдението е еднопосочно. Монитор може да се премахне чрез `Process.demonitor(monitor_ref)`:

```elixir
pid = spawn(fn -> Process.sleep(3000) end)

ref = Process.monitor(pid)
Process.demonitor(ref)

receive do
  msg -> IO.inspect(msg)
after
  4000 -> IO.puts("Няма съобщения...")
end

# Ще видим 'Няма съобщения...'
```

Има и версия на `spawn`, която създава нов процес и автоматично му добавя монитор - `spawn_monitor`:

```elixir
{pid, ref} = spawn_monitor(fn -> Process.sleep(3000) end)
Process.exit(pid, :kill)

receive do
  msg -> IO.inspect(msg)
end
# {:DOWN, <ref>, :process, <pid>, :killed
```

Най-добре ползвайте `spawn_monitor` (както и `spawn_link`) вместо `Process.monitor` (`Process.link`).
Има някаква възможност процеса, който искате да наблюдавате (или да свържете с текущия) да не съществува вече,
когато извикате `Process.monitor` (`Process.link`) и така текущият процес никога да не получи съобщение (сигнал).
Можете да приемете това като първият `race condition`, който ви показваме в `Elixir`.
Ползвайте `spawn_*` функциите, те са атомарни.

## Заключение

Разгледахме как да свържем два процеса или да наблюдаваме един от друг.
Когато даден процес завърши изпълнение, чрез тези връзки или монитори, друг може
да предприеме нещо. Това ни дава възможност да построим система, която при проблем
знае как да реагира и да остане функционираща.
`OTP` ни дава механизми на по-високо ниво за постигаме на `fault tolerance`, но те
също ползват връзки и монитори.

В следващата статия ще разгледаме как един процес може да пази състояние и как да
достъпваме и променяме това състояние.
