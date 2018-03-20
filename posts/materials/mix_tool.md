---
title_image_path: mixing_tools.jpg
created_at: 2018-03-05T23:57:23
tags:
  - elixir
  - introduction
  - mix
  - tool
  - dependencies
  - http client
  - json
  - functional programming
---

# Инструментът Mix. Създаване и тестване на проект
Заедно с `elixir`, `elixirc` и `iex` при инсталацията на Elixir получаваме и `mix`. Това е инструмент, който автоматизира и улеснява работата ни за:
- създаване на приложение/библиотека
- компилиране
- тестване
- управление на dependencies
- форматиране на кода
- изпълнение на задачи (tasks)

#### Създаване на проект
Освен в редки случаи, винаги ще създаваме нашите проекти/библиотеки с mix:
```
$ mix new github_client
```
Резултатът от изпълнението на тази команда е:
```
* creating README.md
* creating .formatter.exs
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/github_client.ex
* creating test
* creating test/test_helper.exs
* creating test/github_client_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd github_client
    mix test

Run "mix help" for more commands.
```

`mix` създава за нас**** основната структура, която изглежда така:
- `.formatter.exs` - използва се от `mix format` за да разбере кои файлове трябва да форматира. Поддържа ограничен набор от възможни настройки, най-съществените от които са дължина на реда и избор на кои функции да не бъдат поставяни скоби около аргументите.
- `mix.exs` съдържа конфигурацията на проекта. В него се определят името, версията зависимосттите на проекта и др.
- `config` папката съдържа конфигурации, използвани от кода на проекта. Пример за такава конфигурация е:
```elixir
config :github_client, GithubClient.Store,
  host: {:system, "INFLUXDB_HOST", "localhost"},
  port: {:system, "INFLUXDB_PORT", 8086},
  pool: [max_overflow: 10, size: 20],
 ```
 - `lib` папката съдържа кода на проекта
 - `test` папката съдържа тестовете на проекта

#### Други команди
За да видим списъка с всички команди, поддържани от mix трябва да изпълним:
```
mix help
```
Ще видим списък с команди и кратко тяхно описание:
```
...
mix compile           # Compiles source files
mix deps              # Lists dependencies and their status
mix deps.clean        # Deletes the given dependencies' files
mix deps.compile      # Compiles dependencies
...
```

За да достъпим по-подробно описание за някоя команда използваме:
```
mix help <command_name>
```

Някои от командите, които най-често ще използваме са:

`mix compile` компилира приложението.


`mix run` стартира всички зависимости и самото приложение. По подразбиране само `mix` изпълнява командата `mix run`


`iex -S mix` е по-интересна команда. Това не е точно `mix` команда, но е полезно да я споменем. Документацията за `iex` намираме чрез:
```
iex --help
```

За `-S` намираме следното:
```
Finds and executes the given script in PATH
```
Това означава, че ще се стартира iex и в него ще бъде изпълнен подадения скрипт, който в нашия случай е `mix run`. За разлика от `mix run`, който приключва веднага след изпълнението си, то с `iex -S mix` имаме възможност да взаимодействаме с нашето приложение.

#### Тестване

При създаването на проекта видяхме, че `mix` ни предлага да влезем в папката и да изпълним тестовете с `mix test`. Нека преди това добавим в `test/github_client_test.exs` още един тест:
```elixir
test "two lists are equal" do
  assert [1,2,3,4,5] == [1,2,3,4,5,6,7]
end
```

И изпълняваме `mix test`. Забелязваме, че освен кода, генерирал грешката, се показва и разликата между лявата и дясната страна, оцветени в червено или зелено.
![mix test output](https://raw.githubusercontent.com/IvanIvanoff/blog/master/assets/mix_test_output.png)


#### Dependencies
По подразбиране  външните зависимости (dependencies) се инсталират чрез [Hex package manager](https://hex.pm/). Можем вместо това да подадем път към git хранилище или път към папка на вашия компютър. При първото инсталиране на  пакети, mix автоматично ще инсталира и Hex.

За нашите нужди ще ни трябва библиотека за JSON и библиотека, предоставяща HTTP клиент. Това са [Poison](https://hex.pm/packages/poison) и [HTTPoison](https://hex.pm/packages/httpoison). Всички пакети се намират на [hex.pm](https://hex.pm/). Добавяме ги към `mix.exs` функцията `deps`  и тя вече изглежда така:
```elixir
  defp deps do
    [
      {:poison, "~> 3.1"},
      {:httpoison, "~> 1.0"},
    ]
  end
  ```
Сега трябва само да изпълним `mix deps.get`.

Tук e моментът да споменем и как mix се справя с различните версии на един пакет. За всеки пакет съществува единствена версия. Това означава, че всички, които зависят от даден пакет, трябва да се съгласят за точно една определена версия.

Най-често се използва `~>` за задаване на диапазон от позволени версии на дадения пакет. Tой работи като фиксира всички цифри от версията, без последната, която евентуално може да бъде по-висока. Пълно описание на начините за фиксиране на версияата на пакет може да намерите [тук](https://hexdocs.pm/elixir/Version.html).

Нека да видим какво се случва като добавим два HTTP клиента, които вътрешно зависят от една библиотека (hackney):
```
{:tesla, "~> 0.10"},
{:httpoison, "~> 1.0"},
```

В `mix.lock` намираме следните два реда:
```
"httpoison": {:hex, :httpoison, "1.0.0", "1f02f827148d945d40b24f0b0a89afe40bfe037171a6cf70f2486976d86921cd", [:mix], [{:hackney, "~> 1.8", [hex: :hackney, repo: "hexpm", optional: false
"tesla": {:hex, :tesla, "0.10.0", "e588c7e7f1c0866c81eeed5c38f02a4a94d6309eede336c1e6ca08b0a95abd3f", [:mix], [{:exjsx, ">= 0.1.0", [hex: :exjsx, repo: "hexpm", optional: true]}, {:fuse
"hackney": {:hex, :hackney, "1.10.1", "c38d0ca52ea80254936a32c45bb7eb414e7a96a521b4ce76d00a69753b157f21", [:rebar3], [{:certifi, "2.0.0", [hex: :certifi, repo: "hexpm", optional: false]
```
Виждаме, че `httpoison` има изискване `:hackney, "~> 1.8"`, а `tesla` има изискване `:hackney, "~> 1.6"`.
Тъй като последната цифра във версията може да бъде по-голяма, то инсталираната версия на `hackney` е `1.10.1` и тя удовлетворява изискванията и на двата пакета.

Веднъж щом се случи това определяне на дадените версии и те бъдат записани в `mix.lock`, то единственият вариант те да бъдат променени е експлицитно да се обновят. Затова е изключително важно да разпространявате `mix.lock` заедно с останалата част от кода. Той гарантира, че абсолютно същите версии ще бъдат инсталирани всеки път и от всеки, които го използва.

#### Как да използваме всичко споменато до тук

По-рано създадохме проекта `github_client` и добавихме две библиотеки към него. Използвайки всичко научено досега, нашият `github_client.ex` изглежда така:
```elixir
defmodule GithubClient do
  @moduledoc """
    Fetch information from the github API
  """
  require Logger

  @github_url "http://api.github.com/"
  @seconds_in_day 60 * 60 * 24

  defp issues_url(org, repo) do
    @github_url <> "repos/#{org}/#{repo}/issues"
  end

  def issues(org, repo, days_old \\ 30) do
    case HTTPoison.get(issues_url(org, repo), [], follow_redirect: true, max_redirect: 5) do
      {:ok, %HTTPoison.Response{status_code: 200, body: body}} ->
        body
        |> Poison.decode!()
        |> Enum.filter(fn %{"created_at" => datetime_iso8601} ->
          {:ok, datetime, _} = DateTime.from_iso8601(datetime_iso8601)
          DateTime.diff(DateTime.utc_now(), datetime) < @seconds_in_day * days_old
        end)
        |> Enum.map(fn %{"title" => title} -> title end)

      {:ok, %HTTPoison.Response{status_code: 404}} ->
        Logger.warn("Github issues for '#{org}/#{repo}' not found")
        []

      error ->
        Logger.warn("Error #{inspect(error)} getting github issues")
        []
    end
  end
end
```

В `@moduledoc` поставяме документацията на нашия модул.

Двата модулни атрибута `github_url` и `seconds_in_day` използваме като константи, които са достъпни в целия модул.

Във функцията `issues_url/2` използваме два различни начина за конкатенация/интерполация на стрингове - чрез `<>` и `#{}`:
```elixir
defp issues_url(org, repo) do
  @github_url <> "repos/#{org}/#{repo}/issues"
end
```

Нека сега да разгледаме функцията `issues/3` .

```elixir
HTTPoison.get(issues_url(org, repo), [], follow_redirect: true, max_redirect: 5)
```

С това извикване на функцията `get` от модула `HTTPoison` (помним, че това е библиотека, която добавихме в `mix.exs`) правим [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) заявка за към `issues_url(org, repo)` без допълнителни headers (`[]`), като позволяваме най-много 5 пренасочвания (redirects).

Резулатът от функцията съпоставяме на
```elixir
{:ok, %HTTPoison.Response{status_code: 200, body: body}}
```

Ще успеем само ако първият елемент е `:ok`, а статус кодът на отговора е 200. `body` е променлива, която след успешно съпоставяне ще съдържа [тялото](https://github.com/IvanIvanoff/blog/blob/master/assets/elixirlang-issues.json) на отговора. В останалите два случая обработваме възможните грешки.

В случая на статус код 200 използваме силата на pipe оператора `|>`. Последователно
- Декодираме JSON до Elixir Map (`Poison.decode!/1`)
- Оставяме само елементи с дата на създаване през последните 30 дни (`days_old` се приема като трети аргумент на `issues/3` и има стойност по подразбиране 30).
- Трансформираме данните от списък с всички данни до списък само от заглавията (`Enum.map(fn %{"title" => title} -> title end)`). По този начин съпоставяме всеки map, който има ключ `title` и връщаме само този `title`.

#### Как да тестваме всичко това?
Ще разгледаме два начина този код да бъде изтестван, но имплементацията им е оставена като упражнения за читателя.

Първият вариант е свързван с добавянето на няколко помощни функции. Вместо цялата работа да се извършва в `issues/3`, то може да разделим логиката на две части - функция, която прави HTTP заявката и връща [тялото](https://github.com/IvanIvanoff/blog/blob/master/assets/elixirlang-issues.json) на отговора и функция, която обработва тези данни. В този случай `issues/3` би изглеждала така:
```elixir
  def issues(org, repo, days_old \\ 30) do
    json = get_issues(org, repo)
    process_issues(json, days_old)
  end
```

По този начин можем в нашия тест директно да изтестваме `proccess_issues/2` без нуждата от интернет, mock и т.н. Този подход има може би повече недостатъци, от колкото предимства. Можем да извикваме само публични функции от даден модул като това важи с пълна сила и за тестовете. При този подход сме принудени да направим публична една функция, която би трябвало да не е. Освен, че добавяме към видимите функции на модула ненужни функции, то не сме изтествали функцията `issues/3` - нашият тест не хваща потенциално опасни промени.

Вторият вариант е свързан с използване на [mock](https://stackoverflow.com/questions/2665812/what-is-mocking). Нашата цел е да направим тестването независимо от зависимости (интернет и сайтът към който правим заявки да работи).

> Забележка: Има противоречия за това как се прави mock. Mнението на създателя на езика, Jose Valim, може прочете [тук](http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/). В нашия пример използваме `Mockery` за образователни цели, а не защото това е единственото и най-добро решение.

Можем да постигнем това като използваме библиотеката [Mockery](https://hex.pm/packages/mockery). Комуникацията с въшния свят се случва през `HTTPoison`. Затова декларираме модулен атрибут с произволно и описателно име:
```elixir
@http_client Mockery.of("HTTPoison")
```
Сега променяме всички извиквания на функции от `HTTPoison` да се случват през `@http_client`:
```elixir
case @http_client.get(issues_url(org, repo), [], follow_redirect: true, max_redirect: 5) do
```

След като направим това, то в нашия тест можем да върнем предетерминирам резултат:
```elixir
...
import Mockery

test "github issues" do
  mock(
    HTTPoison,
    :get,
    {:ok,
      %HTTPoison.Response{
        body: var_containing_actual_body,
        status_code: 200
      }}
  )
...
end
```

Едно нещо прави впечатление. Изглежда трябва да се погрижим за прекалено много неща - да знаем формата на отговора, да добавим ръчно статус кода и т.н. Също така - какво се случва, ако имаме повече от едно извикване на `HTTPoison.get/3`? Трябва да четем документацията, да видим как се параметризира този mock - трябва да направим разграничение според използвания URL.

Това ни навежда на мисълта, че проблемът е на друго място. Добавяме нов модул `GithubApi` през който се случва цялата комуникация с github. В него вече ще имаме различни функции - `get_issues`, `get_pull_requests`, `get_all_repositories`, и др. Сега нашият mock изглежда по-просто:
```elixir
mock(
  GithubApi,
  :get_issues,
  {:ok, var_containing_actual_body}
)
```

Сега вече ако се наложи да направим две или повече HTTP GET заявки, то няма това да се случи през една функция, която трябва да параметризираме (`HTTPoison.get/3`), ами това ще бъдат отделни функции от `GithubApi` модула.