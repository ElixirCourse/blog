---
category: Програма
title_image_path: input_output.gif
tags:
  - elixir
  - exceptions
  - throw
  - pattern matching
  - raise
  - try
  - catch
  - IO
  - file
  - path
  - puts
  - read
  - write
  - stream
---

# Грешки и Вход-Изход

Видяхме какво представлява Elixir вътре в процесите.
Имаме някаква идея какво представлява Elixir като множество комуникиращи си процеси (ще разберем повече в следващата публикация).
Тъй като всяка програма си комуникира по някакъв начин с външния свят ще обърнем внимание на Elixir и в контекста на външния свят.

Ще разгледаме какво представляват грешките в Elixir.
Най-често те биха възникнали когато програмата ни няма контрол над даден проблем.
В повечето случаи такъв проблем възниква именно извън нея.

Ще разгледаме и модулите свързани с писане и четене на файлове както и принтирането и четенето от терминала.

## Грешки

Грешките в Elixir са предназначени за ситуации, които никога не би трябвало да се случат
при нормални обстоятелства.
Когато програмата зависи нещо външно (*service*-и, файлова система), а то не е достижимо,
когато имаме проблем с ресурсите и конфигурацията, от които нашата програма зависи.

Да речем грешен *input* от потребител не е такава ситуация.

### 'Вдигане' на грешка

Грешка се 'вдига' с `raise`:

```elixir
raise "Ужаст!"
#=> (RuntimeError) Ужаст!
```

По подразбиране, ако не подадем тип на грешката на `raise`, тя е `RuntimeError`.
Можем да 'вдигнем' грешка и с тип, без съобщение:

```elixir
raise RuntimeError
#=> (RuntimeError) runtime error
```

Както и с тип на грешката и съобщение:

```elixir
raise ArgumentError, message: "Грешка, брато!"
#=> (ArgumentError) Грешка, брато!
```

`Elixir` идва с набор от грешки за различни случаи, които можете да видите в
[документацията](https://hexdocs.pm/elixir) под *EXCEPTIONS*.

Грешките в Elixir не са препоръчителни за употреба.
Имат славата на *GOTO*/спагети програмиране и наистина е хубаво да помислим дали има нужда от тях в дадена ситуация.

Прието е функции, при които има проблем, да връщат `{:error, <причина>}`, а ако се изпълняват с успех и
имат резултат - `{:ok, <резултат>}`.

Имената на функции, които биха могли да 'вдигнат' грешка, обикновено завършват на **!**.
Ако имаме `SomeModule.some_function/1`, която връща `{:ok, result}` или `{:error, reason}`
и искаме да дефинираме същата, но при успех връщаща `result`, а при неуспех, 'вдигаща' грешка,
ще я именоваме `SomeModule.some_function!/1`.
В секцията за вход-изход ще видим, че функциите идващи с езика следват тази конвенция.

### Прихващане на грешка

Можем да 'прихванем' грешка, използвайки `try/rescue` блок:

```elixir
try do
  1 / 0
rescue
  [RuntimeError, ArgumentError] ->
    IO.puts("Няма да стигнем до тук.")
  error in [ArithmeticError] ->
    IO.puts("На нула не се дели, #{error.message}")
  any_other_error ->
    IO.puts("Лошаво... #{any_other_error.message}")
else
  IO.puts("Няма грешка.")
after
  IO.puts("Finally!")
end
```

Примерът по горе показва няколко вида прихващане. Прихващането е *pattern match*-ване.

Първият пример е чрез списък от тип грешки, докато във втория, виждаме как от този списък
да си вземем грешката в променлива.
Накрая хващаме всички типове, които не сме описали досега в променливата `any_other_error`.

Имаме и `after` клауза, която винаги ще се изпълни след като `try` функцията завърши, няма значение,
дали е имало грешка или не. Другата интересна клауза в примера е `else`. Тялото ѝ се изпълнява само
ако не е възникнала грешка в тялото на `try`.

### Създаване на нови типове грешки

Можем да създадем и нови типове грешки. Подобно на структурите, нова грешка се
дефинира като част от модул:

```elixir
defmodule VeryBadError do
  defexception message: "Лошо!!!"
end
```

Сега можем да я 'вдигнем':

```elixir
try do
  raise VeryBadError
rescue
  error in VeryBadError ->
    IO.puts(inspect(error, structs: false))
end
#=> %{__exception__: true, __struct__: VeryBadError, message: "Лошо!!!"}
```

Както виждате, грешките са структури с още едно тайно поле - `__exception__`.
То има стойност `true`.

### Throw/Catch

Тези две конструкции **НЕ ТРЯБВА** да се ползват. Има библиотеки, които поради някакво
стечение на обстоятелствата е възможно да ги ползват, но ги избягвайте. Ние ви ги
показваме за да ви кажем: *не ги ползвайте*!

С `throw` 'подхвърляме' стойност, която може да се 'хване' по-късно:

```elixir
try do
  throw 5
catch
  x -> IO.puts(x)
end
#=> 5
```

Сега - забравете за тях и не ги ползвайте.

### В заключение

Тази секция е доста кратка. Идеята ѝ е да ни запознае с грешките в Elixir и да ни
каже - **не ги ползвайте**.

Два факта:
1. В кода на *mix* няма прихващане на грешки.
2. В кода на компилатора на Elixir има точно пет прихващания. Все пак това е компилатор и там се случват доста магически неща.

Ако започнете да създавате нов тип за грешки се замислете.

В Erlang/Elixir кодът върви в изолирани процеси. Идеологията е - '*остави го да се счупи*'.

Това е така защото тези процеси са наистина изолирани и не споделят състояние.
Ако един падне, друг ще бъде вдигнат на негово място и така ако е имало проблем, възникнал *runtime*,
той ще се изчисти. Няма защо ние да правим това.

Важно е да запомните, че писането на нови типове грешки е нещо, което Elixir програмистите **НЕ** правят.
Същото се отнася и за 'прихващането' им.

## Вход-изход

Ще разгледаме няколко модула от стандартната библиотека, като `IO` и `File` и ще видим
как да ги ползваме за да четем и пишем.

### Изход с IO.puts/2 и IO.write/2

Досега ползвахме `IO.puts/2` за да извеждаме текст на стандартния изход.
Може би се чудите защо написахме функцията като функция на два аргумента.
Това е защото тя е такава, просто първият, `device` има стойност по
подразбиране. Засега няма да разглеждаме тази стойност.
Тя е свързана с комуникацията между процеси.
Това, което ще направим е да подадем други стойности:

```elixir
IO.puts("По подразбиране пишем на стандартния изход.")
IO.puts(:stdio, "Можем да го направим и така.")
IO.puts(:stderr, "Или да пишем в стандартния изход за грешки.")
```

Всъщност `puts` се държи по същия начин като друга функция в `IO` - `write`.
Разликата е, че `puts` слага нов ред след текста, който ѝ е подаден.

```elixir
IO.write(:stderr, "Това е грешка!")
```

### chardata

Както казахме, първият аргумент на `write` и `puts` е `device`.
Вторият е нещо от тип *chardata*.

Какво е *chardata*?
В статията [Списъци и потоци](/posts/TODO) споменахме за *iolist*, по-познат като *iodata* в Elixir.
Този тип е доста подобен.

Следните стойности са *chardata*:
* Низ, да речем `"Далия"`.
* Списък от *codepoint*-и, да речем `[83, 79, 0x53]` или `[?S, ?O, ?S]` или `'SOS'`.
* Списък от *codepoint*-и и низове - `[83, 79, 83, "mayday!"]`.
* Списък от *chardata*, тоест списък от нещата в горните три точки : `[[83], [79, ["dir", 78]]]`.

Подавайки *chardata* на функции като `IO.puts/2` и `IO.write/2`, можем да избегнем конкатенация на
низове, което е винаги хубаво нещо. Така няма копиране в паметта, данните се изпращат направо където е нужно.
Като цяло това е едно от чудесата на Erlang/Elixir, ползвайте го вместо конкатенирани или интерполирани низове, когато можете.

В `IO` има функция, която трансформира *chardata* в низ.

```elixir
IO.chardata_to_string([1049, [1086, 1091], "!"])
# "Йоу!"
```

### Вход с IO.read/2, IO.gets/2, IO.getn/2 и IO.getn/3

Функцията `read` също взима `device` като първи аргумент (което е или атом, да речем `:stdio` или *PID* на процес).
Вторият аргумент може да бъде:
* Атомът `:all` - значи да се изчете всичко идващо от `device`-а, докато не се достигне *EOF*, тогава се връща празен низ.
* Атомът `:line` - прочита се всичко до нов ред или *EOF*. При *EOF*, функцията връща `:eof`.
* Цяло число, по голямо от нула - прочита толкова символа от `device` или колкото може преди да достигне *EOF*.

Функцията връща прочетеното:

```elixir
IO.read(:line)
Хей, Хей<enter>
#=> "Хей, Хей\n"
```

Много подобна е и функцията `IO.gets/2`. Тя приема *prompt* като втори аргумент и чете до нов ред:

```elixir
IO.gets("Кажи нещо!\n")
Кажи нещо!
Нещо!<enter>
#=> "Нещо!\n"
```

Двете `getn` функции прочитат брой байтове или *unicode codepoint*-и, в зависимост от типа на `device`-а.
Когато говорим за файлове ще разгледаме как можем да отворим файл в различни *mode*-ове.

### iodata

Подобно на *chardata*, *iodata* може да се дефинира като списък.
За разлика от *chardata*, *iodata* списъкът е от цели числа които представляват байтове (0 - 255),
*binary* с елементи със *size*, кратен на **8** (могат да превъртат) и такива списъци.

Има функции, които боравят с *iodata* - `IO.binwrite` и  `IO.binread`.
Тези функции са по-бързи от не-`bin*` вариантите им.
Не трансформират това което получават в *utf8*.

В `IO` има две функции за боравене с *iodata*.
Функцията `IO.iodata_length/1` връща дължината на *iodata* в байтове:

```elixir
IO.iodata_length([1, 2 | <<3, 4>>])
#=> 4
```

Функцията `IO.iodata_to_binary/1` трансформира *iodata* в *binary*:

```elixir
IO.iodata_to_binary([1, << 2 >>, [[3], 4]])
#=> <<1, 2, 3, 4>>
```

### Файлове

Модулът `File` съдържа функции за работа с файлове. Някои от тях ни позволяват да отваряме
файловете за писане и четене. По подразбиране всички файлове се отварят в *binary mode* и
функциите `IO.binwrite/2` и `IO.binread/2` трябва да се използват за работа с тях.

Файл може да бъде отворен и в *utf8 mode*. По този начин байтовете, записани или прочетени,
ще се интерпретират като *utf8 codepoint*-и.

```elixir
{:ok, file} = File.open("test.txt", [:write])
#=> {:ok, #PID<0.855.0>}

IO.binwrite(file, "some text!")
#=> :ok

File.close(file)
#=> :ok
```

Функцията `File.open/2` връща наредена двойка - `{:ok, device}`.
Ако имаше някаква грешка щяхме да получим `{:error, reason}`.
Това е нормално при повечето функции свързани с файлове.
Разбира се има и функции, които хвърлят грешка при проблем и връщат резултата направо.
Те имат същите имена, но завършващи на **!**. Пример: `File.open!/2`.
Споменахме тази конвенция по-горе, когато говорихме за грешки и имена на функции.

В модула има много функции за създаване и триене на файлове и директории, за проверки дали съществуват,
за промяна и показване на съдържанието на директория.
Прочетете за тях в [документацията](https://hexdocs.pm/elixir/File.html).

### Процеси и файлове

Така наречения `device` всъщност е *PID* на процес или атом, който е ключ, сочещ към *PID* на процес.
Когато отваряме файл се създава нов процес, който знае *file descriptor*-а на файла
и управлява писането и четенето към и от него.

Това е много хубаво нещо. От една страна това означава, че `IO` функциите на един *node*,
могат да четат/пишат файл на друг *node*, или един компютър да управлява файлове на друг.
От друга, означава че можем да си създаваме лесно свои `device`-и, чрез процеси, които
знаят какво съобщение да очакват.

Разбира се това означава, че всяка операция с файла минава през комуникация между процеси.
Когато искаме оптимално писане/четене на файл, това не е плюс.

Именно за това има функции, които направо работят с файлове, като `File.read/1`, `File.read!/1`,
`File.write/3`, `File.write!/3`.

Тези функции отварят файла и пишат/четат в/от него като една операция, след това го затварят.

### Потоци и файлове

Ако не искаме да прочетем цял файл в паметта, можем да го отворим и да си направим поток към него:

```elixir
{:ok, file} = File.open("program.txt", [:read])
#=> {:ok, #PID<0.82.0>}

IO.stream(file, :line)
|> Stream.map(fn line -> line <> "!" end)
|> Stream.each(fn line -> IO.puts(line) end)
|> Stream.run()
```

Това ще прочете файла ред-по-ред, трансформирайки редовете и ще ги изведе на стандартния изход.
Разбира се `IO.stream/2` има и `IO.binstream/2` версия.

Ако искаме по бързо четене/писане, без преминаване през комуникация между процеси, ползваме
`File.stream!/2`:

```elixir
File.stream!(filename, read_ahead: 10_000)
```

По подразбиране, когато използваме `File.stream!/2`, файловете се отварят в *raw binary read_ahead mode*.
Това означава, че няма трансформация към *UTF8 codepoint*-и има буфериране в паметта. В примера по горе,
показваме как можем да зададем големина на буфера.

Ако искаме наистина бързо четене от файл на части, трансформиране и записване в друг файл
е добре да следваме следния шаблон:

```elixir
File.stream!(<input_name>, read_ahead: <buffer_size>)
|> Stream.<transform-or-filter>
...
|> Stream.into(File.stream!(<output_name>, [:delayed_write]))
|> Stream.run
```

По този начин комбинирайки `:read_ahead` и `:delayed_write` се получава буфериране с
добра скорост. Повече по темата [тук](http://cloudless.studio/articles/12-elixir-vs-ruby-file-i-o-performance-updated).

### Модула IO.ANSI

Този модул съдържа функции които контролират цвета, и форматирането в теминала.
Много добре се комбинират в *chardata* списък с текст:

```elixir
IO.puts [IO.ANSI.blue(), "text", IO.ANSI.reset()]
# Ще отпечата 'text' в синьо, ако терминалът ви поддържа ANSI цветове
```

### Модула StringIO и файлове в паметта

Използвайки този модул, ние можем да четем/пишем от/в низове в паметта:

```elixir
{:ok, pid} = StringIO.open("data")
#=> PID<0.136.0>}

StringIO.contents(pid)
#=> {"data", ""}

IO.write(pid, "doom!")
#=> :ok

StringIO.contents(pid)
#=> {"data", "doom!"}

IO.read(pid, :line)
#=> "data"

StringIO.contents(pid)
#=> {"", "doom!"}

StringIO.close(pid)
#=> {:ok, {"", "doom!"}}
```

Както виждаме в паметта се държат два низа - един за вход, един за изход.
Можем да четем от изхода, докато стане празен и да пишем във входа.

Това не е точно поведението при един истински файл, за който нямаме две пространства, а само едно.

Ако искаме псевдо-файл в паметта, който се държи като истински файл, можем да го направим така:

```elixir
{:ok, file} = File.open("data", [:ram])
#=> {:ok, {:file_descriptor, :ram_file, #Port<0.1578>}}
IO.binread(file, :all)
#=> "data"
```

Опцията при отваряне `:ram`, създава файл в паметта със съдържание първия аргумент на функцията `open`.
Ако сега направим:

```elixir
IO.binread(file, :all)
#=> ""
```

Ще получим празен низ. Това е защото сме в края на файла, можем да променим това, с Erlang функцията `:file.postion/2`.

```elixir
:file.position(file, :bof)
#=> {:ok, 0}

IO.binread(file, :all)
#=> "data"
```

Така отиваме на позиция `:bof` - *beginning of file* и четем. В Elixir няма *random access* функции, но могат да се ползват
тези от Erlang.

### Модула Path

Много от функциите във `File` изискват пътища.
Модулът `Path` ни предоставя спомагателни функции за работа с пътища:

```elixir
Path.join("some", "path")
#=> "some/path"

Path.expand("~/development")
#=> "/home/meddle/development"
```

По-добре е да си строим пътищата с функции от `Path`.
Те се справят с различията в операционните системи - знаят на какво вървят.

## Заключение

Повечето програми използват външни ресурси. Обикновено нямаме пълен контрол над тези ресурси, затова могат да възникнат грешки при работа с тях,
които не зависят от логиката на нашата програма.
Най-добре е да използваме функции, коит връщат `{:ok, result}` или `{:error, reason}`, когато работим с такива ресурси.
Видяхме, че функциите за *IO* имат такива версии и точно те се ползват най-често.
Ако сме убедени, че процес трябва да *crash*-не, ако даден ресурс не е наличен, можем да използваме *!* версиите на тези функции.

Научихме как да 'вдигаме' грешки, но най-добре е, когато пишем напи функции да не го правим.
По-добре е да връщаме `{:error, reason}`. По-лесно се чете и поддържа такъв код.