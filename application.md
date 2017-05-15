title_image_path: application.jpg
category: Програма
tags:
  - elixir
  - process
  - gen_server
  - OTP
  - GenServer
  - start_link
  - supervision
  - application
  - app

--------

# Процеси и OTP : Application

`Application` е още едно поведение, което идва с `OTP`.
Трите поведения - `GenServer`, `Supervisor` и `Application` са свързани.
Обикновено създаваме `Application`, който идва със свое дърво от `Supervisor`-и,
което както казахме в [предишната статия](/posts/supervision.md), има за листа
`GenServer` процеси.

## Какво е Application?

Както и при предишните две абстракции, които разгледахме, имаме логика, която работи
с модул, имплементиращ дадено поведение.

`Application` е компонент в `Elixir/Erlang`, който може да бъде спиран и стартиран като едно цяло.
Също така може да бъде използван от други `Apllication`-и.
Един `Application` се грижи за едно `supervision` дърво и средата, в която то върви.

Винаги, когато виртуалната машина стартира, специален процес, наречен 'application_controller' се стартира с нея.
Този процес стои над всички `Application`-и, които вървят в тази виртуална машина, можем да го наречем `Supervisor` на `Application`-ите.
Разбира се самите `Application`-и могат да се управляват с различни стратегии, които ще разгледаме по-късно.

Както казахме можем да приемем `application_controller` процеса за коренът на дървото от процеси в един `BEAM node` (макар и това да не е цялата истина, има известни изключения, за които, поне засега, няма да говорим).
Под този 'корен' стоят различните `Application`-и, които както казахме са абстракция, обвиваща `supervision` дърво, която може да се стартира и спира като едно цяло.
Представете си ги като мега-процеси, управлявани от `application_controller`-a. Поне отвън те изглеждат като един процес за който имаме функции `start` и `stop`.

Когато се стартира един `Application` се създават два специални процеса, които заедно са наречени 'application master'.
Тези два процеса създават `Application`-а и стоят между `application_controller` процеса и `Supervisor`-а служещ за корен на `supervision` дървото на дадения `Application`.

Сега идва времето да разгледаме поведението `Application` и какво представлява и ни дава то.
Ще започнем, както и в предишните две статии, с пример.

## Пример : Blogit

Досега давахме за пример различни части от `Blogit`.
Видяхме, че неговият `root Supervisor` се грижи за 'мозъка' му - `Blogit.Server` и
за друг `Supervisor`, управляващ малки компоненти, които могат да са достъпни за заявки отвън.

Нека сега да видим къде се създава този `root Supervisor` - `Blogit Application`-а:

```elixir
defmodule Blogit do
  use Application

  alias Blogit.Components.Posts
  alias Blogit.Components.PostsByDate
  alias Blogit.Components.Configuration

  @repository_provider Application.get_env(
    :blogit, :repository_provider, Blogit.RepositoryProviders.Git
  )

  def start(_type, _args) do
    Blogit.Supervisor.start_link(@repository_provider)
  end

  def list_posts(from \\ 0, size \\ 5) do
    GenServer.call(Posts, {:list, from, size})
  end

  def list_pinned(), do: GenServer.call(Posts, :list_pinned)

  def filter_posts(params, from \\ 0, size \\ 5) do
    GenServer.call(Posts, {:filter, params, from, size})
  end

  def posts_by_dates, do: GenServer.call(PostsByDate, :get)

  def post_by_name(name), do: GenServer.call(Posts, {:by_name, name})

  def configuration do
    GenServer.call(Configuration, :get)
  end
end
```

Практика е `Application`-а да се слага в основния модул на проекта.
За проекта `Blogit`, този модул е `Blogit`.

Както казахме, `Application` е поведение и отново, чрез мета-програмиране, има имплементация по подразбиране за някои негови функции.
Всъщност `callback` функциите му са само две - `Application.start/2` и `Application.stop/1`.
`use Application` задава имплементация по подразбиране само за `Application.stop/1` - просто връща `:ok`.
Ние трябва да зададем имплементация на `Application.start/2`.

`Application.start/2` има два аргумента - тип, който обикновено е `:normal` (ако програмата е дистрибутирана може и да е нещо друго) и аргументи, които след малко ще видим откъде идват.
Обикновено в тази функция построяваме `supervision` дървото и го стартираме.
В `Blogit` примера правим точно това. Стартираме `Blogit.Supervisor`, който както видяхме в [предишната статия](/posts/supervision.md) е коренът за `Blogit`.

В този пример можем да видим, че всеки `Application` си има `environment`, който можем да конфигурираме и използваме.
Да речем тук `@repository_provider`, който представлява способ за четене от някакво `repository` с публикации, се определя с помощта на `Application.get_env/3`.

Тази функция взима за първи аргумент атом, представляващ `Application`, за втори аргумент ключ към `environment keyword list`-а на този `Application` и за трети аргумент
стойност по подразбиране. След малко ще видим как задаваме настройки, които влизат в този `environment keyword list`.

Друга практика е клиентски функции, които правят заявки към различни процеси на `Application`-а да се дефинират тук.
Това е нещо като публичен интерфейс на `Application`-а. Можем да видим че `Blogit` ни дава възможност:
* Да вземем списък от публикации, който поддържа `pagination`.
* Да вземем списък от важни 'pinned' публикации.
* Да филтрираме публикации.
* Да вземем информация за публикациите по месец и година.
* Да вземем публикация по нейното име-идентификатор.
* Да вземем конфигурацията на блога.

Основните три компонента на `Blogit` стоят зад тези функции-заявки, но за публичния интерфейс, имплементацията не е важна.
Тук е добре да има добра и пълна документация, която ние сме премахнали за да направим примера кратък.

Ето какво представлява `Application` дървото на `Blogit`, представено от инструмента, който може да пуснете с `:observer.start/0`.

<br />
![alt text](/custom/assets/blogit.png ":observer.start") {: .inline-image}
<br />

Процесите с `pid`-ове `0.186.0` и `0.187.0` са `application master` абстракцията, под тях виждаме `supervision` дървото,
за което говорихме в [предишната статия](/posts/supervision.md). Има един процес `Elixir.Earmark.Global.Messages`, който
идва от библиотеката с която преобразуваме `markdown` към `HTML` - `Earmark`, който се създава автоматично от нея и се `link`-ва към процеса,
който я ползва - `Blogit.Server`. Всичко останало е както го описахме.

Нека сега разгледаме двете функции на поведението `Application`.

## Поведението Application

Както казахме това поведение има само две `callback` функции - `start` и `stop`.

### start/2

Извиква се при стартиране на `Application`-а.
Очаква се да стартира процеса-корен на програмата, обикновено това е `root` или `top-level` `Supervisor`, зависи откъде го гледаме.

Очаква се да върне `{:ok, pid}`, `{:ok, pid, state}` или `{:error, reason}`, в зависимост от това дали и как се е стартирал този основен процес.
Този `state` може да е каквото и да е, по подразбиране може да бъде пропуснат и е `[]` (празен списък).
Той се подава на `stop/1` `callback` функцията при спиране.

На `start/2` се подават два аргумента. Първият обикновено е атома `:normal`, но при дистрибутирана програма би могъл да е `{:takeover, node}` или `{:failover, node}`.
Вторият са аргументи за програмата, които се задават при конфигурация, както ще видим по-късно.

### stop/1

Когато `Application`-а бъде спрян, тази функция се извиква със състоянието върнато от `start/2` или ако няма такова с `[]`.
Използва се за изчистване на ресурси и има имплементация по подразбиране, която просто връща `:ok`.

## Функциите на модула Application

Ще разгледаме няколко от функциите на модула, които управляват или дават информация за `Application`.

### Application.load/1

Зарежда `Application` в паметта. Зарежда `environment` данните му и други свързани `Application`-и.
Не го стартира.

### Application.start/2

Стартира `Application`. За първи аргумент взима атом идентифициращ `Application`-а, а за втори типа на `Application`-а.
Извиква `Application.load/1` ако програмата не е заредена в паметта.

Важно е, други `Application`-и, от които зависи стартираната да са стартирани преди това, иначе функцията връща `{:error, {:not_started, app}}`.

Типът на програмата може да бъде:
* `:permanent` - Ако `Application`-ът умре, всички други `Application`-и на `node`-а също умират. Няма значение дали `Application`-а е завършил нормално или не.
* `:transient` - Ако `Application`-ът умре с `:normal` причина, ще видим `report` за това, но другите `Application`-и на `node`-а няма да бъдат терминирани. Ако причината обаче е друга, всички други `Application`-и и целия `node` ще бъдат спрени.
* `:temporary` - Това е типът по подразбиране. С каквато и причина да спре един `Application`, другите ще продължат изпълнение.

`Application` стратегиите не се опитват да спасят или рестартират нищо. Те задават дали и другите `Application`-и трябва да бъдат терминирани при проблем.
На този етап, ако `Application`-а 'умира' няма спасение.

Ако спрем ръчно `Application` с функцията `Application.stop/1`, тези стратегии няма да се задействат.

### Application.ensure_all_started/2

Прави същото като `Application.start/2` и взима същия тип аргументи, но допълнително стартира всички други `Application`-и, конфигурирани като зависимости на подадената ѝ като първи аргумент.

### Application.get_application/1

Връща атом представляващ `Application`. Няма значение дали този `Application` е активен или не. Важно е да е специфициран (ще видим как става това след малко).
Взима модул, който представлява `Application`, тоест има `use Application`. При `Blogit`:

```elixir
Application.get_application(Blogit)
# :blogit
```

### Функции за четене и писане на Application environment

Както казахме `Application environment` е `keyword list`, който се конфигурира при дефиниране на `Application`.
Има няколко функции свързани с него:
* `Application.fetch_env(app :: atom, key) :: {:ok, value} | :error` - Взима стойност по `Application` атом и ключ, ако няма такъв ключ връща `:error`. Има версия `fetch_env!`, която при липса на ключ 'вдига' `ArgumentError`.
* `Application.get_all_env(app :: atom) :: [{key, value}]` - Връща целия `environment` списък за `Application` по атомът, който го представлява.
* `Application.get_env(app :: atom, key, value) :: value` - Видяхме го в примера по-горе. Връща стойност по атом и ключ, или зададена стойност по подразбиране, която може да се пропусне и тогава ще е `nil`.
* `Application.put_env(app :: atom, key, value, [timeout: timeout, persistent: boolean]) :: :ok` - Добавя стойност към `environment` списъка на `Application`.
* `Application.delete_env(app :: atom, key, [timeout: timeout, persistent: boolean]) :: :ok` - Трие стойност от `environment` списъка.

Пример:

```elixir
Application.get_all_env(:blogit)
# [
#   assets_path: "assets", polling: true,
#   repository_url: "git@github.com:meddle0x53/elixir-blog.git",
#   posts_folder: ".", included_applications: [], poll_interval: 60000
# ]

Application.get_env(:blogit, :repository_url)
# "git@github.com:meddle0x53/elixir-blog.git"
```

### Application.spec

Тази функция има две версии. Първата взима само `Application` и връща цялата му спецификация, а втората `Application` и ключ в спецификацията, за да върне част от нея.

### Application.started_applications/0 и Application.loaded_applications/1

Връщат информация за `Application`-ите на `node`-а.

### Application.stop/1

Спира `Application`, без да задейства стратегията му. `Application`-ът остава зареден в паметта.

### Application.unload/1

Премахва от паметта спрян `Application` и неговите зависимости, зададени като `included_applications`.


Тези функции могат да бъдат разгледани по-подробно в [документацията](https://hexdocs.pm/elixir/Application.html#functions).
Обикновено няма да ползвате тези, които не са свързани с `environment`-а. Всичко би трябвало да бъде поето за вас от `mix` и `Elixir`.
Нека да видим как се създава `OTP Application mix` проект и как се конфигурира един `Application`.

## Създаване и конфигурация на Application

Тези `OTP Application` поведения и логиката около тях идват от `Erlang/OTP`.
Те се конфигурират със специален `.app` файл, написан на `Erlang`, който се слага
при `.beam` файловете, които описва и след това може да се зареди на `node`, който има в пътя си
директорията с него и тези `.beam` файлове. Както сигурно се досещате, това става с `Application.load/1`.

Ето го и `.app` файла на `Blogit`:

```erlang
{application,blogit,
             [{description,"  Blogit is an OTP application for generating blog posts from a git\n  repository containing markdown files.\n"},
              {modules,['Elixir.Blogit',
                        'Elixir.Blogit.Components.Configuration',
                        'Elixir.Blogit.Components.Posts',
                        'Elixir.Blogit.Components.PostsByDate',
                        'Elixir.Blogit.Components.Supervisor',
                        'Elixir.Blogit.Logic.Search',
                        'Elixir.Blogit.Logic.Updater',
                        'Elixir.Blogit.Models.Configuration',
                        'Elixir.Blogit.Models.Post',
                        'Elixir.Blogit.Models.Post.Meta',
                        'Elixir.Blogit.RepositoryProvider',
                        'Elixir.Blogit.RepositoryProviders.Git',
                        'Elixir.Blogit.RepositoryProviders.Memory',
                        'Elixir.Blogit.RepositoryProviders.Memory.RawPost',
                        'Elixir.Blogit.Server','Elixir.Blogit.Supervisor']},
              {registered,[]},
              {vsn,"0.7.3"},
              {applications,[kernel,stdlib,elixir,logger,yaml_elixir]},
              {mod,{'Elixir.Blogit',[]}}]}.
```

Разбира се ние сме `Elixir` програмисти и няма защо да пишем този файл ръчно.
Можем да си конфигурираме тези неща в `mix` проект.

Ако генерираме `mix` проект с:

```bash
mix new <app_project_name> --sup
```

ще получим структура подходяща за `OTP Application`. Можем да използваме `mix compile.app` в директорията на този проект за да се генерираме `.app` файла.

### mix файла на един Application

При създаване на нов `--sup` проект имаме по специфичен `mix.exs` файл, който трябва да допълним.
Нека разгледаме завършения файл за `Blogit`:

```elixir
defmodule Blogit.Mixfile do
  use Mix.Project

  def project do
    [app: :blogit,
     version: "0.7.3",
     elixir: "~> 1.4",
     build_embedded: Mix.env == :prod,
     start_permanent: Mix.env == :prod,
     docs: [readme: true, main: "README.md"],
     description: """
       Blogit is an OTP application for generating blog posts from a git
       repository containing markdown files.
     """,
     aliases: aliases(),
     deps: deps()]
  end

  def application do
    [applications: [:logger, :yaml_elixir],
     mod: {Blogit, []}]
  end

  defp deps do
    [
      {:git_cli, "~> 0.2"},
      {:earmark, "~> 1.1"},
      {:yaml_elixir, "~> 1.3.0"},
      {:calendar, "~> 0.16.1"},
      {:ex_doc, ">= 0.15.0", only: :dev},
      {:dialyxir, "~> 0.5", only: [:dev], runtime: false}
    ]
  end

  defp aliases do
    [test: "test --no-start"]
  end
end
```

Както виждате `vsn` и `description`, които видяхме в `blogit.app` файла идват от
`project` конфигурацията. Атомът, който идентифицира `Application`-а също. Задава се чрез `app`.

Интересна е функцията `application/0`. Тя задава специфични опции свързани с `OTP` програми.
Ключът `mod` сочи към наредена двойка - модулът на `Application`-а, който съдържа `use Application` и `start(type, args)` и списък от аргументите `args`, които се задават при извикване на `start(type, args)`.
Списъкът зададен под ключа `applications` съдържа други `Application`-и от които нашият зависи. Към тях при генериране на `.app` файла се добавят и други, които са задължителни -  `kernel`, `stdlib` и `elixir`.
Тук можем да зададем и `keyword list` с ключ `:env` за да дефинираме началния `environment` на `Application`-а, но има и по-добър начин.

Модулите част от `.app` файла са всички модули дефинирани в `mix` проекта.

Винаги когато стартираме кодът си чрез `mix`, да речем с `iex -S mix` или с `mix test`, `Application`-ът ще се стартира и всичките зависимости изброени в `:extra_applications` и `:applications` ще се стартират преди него.
Това поведение може да се избегне, като подадем `--no-start` на командата. Това правим и при `mix test` при примера горе.
Казваме всеки пък когато изпълним `mix test`, всъщност да изпълняваме `mix test --no-start`.

### Конфигурация на Application

Казахме, че `environment`-а може да се дефинира по по-добър начин от това да го слагаме направо в `mix.exs` файла.

Това става чрез дефинирането му в `config/config.exs` файла, който може да зарежда различни конфигурации, в зависимост от `mode`-а в който е пуснат `node`-а.
Така когато си пускаме тестовете в `test mode` ще имаме една конфигурация, а в `dev mode` или `prod mode` друга.

Ето как изглежда `dev` конфигурацията на `Blogit`:

```elixir
use Mix.Config

config :blogit,
  repository_url: "git@github.com:meddle0x53/elixir-blog.git",
  polling: true, poll_interval: 60_000,
  posts_folder: ".", assets_path: "assets"
```

Използваме функцията `config`, идваща от `Mix.Config`, подаваме атома на `Application`-а, в случая `:blogit` и списък от ключове и стойности.
Тези тук ги видяхме и когато извикахме `Application.get_all_env(:blogit)`.

### Как стартираме един Application?

Освен с `iex` или ръчно, с `Application` функциите, можем да пуснем `Application` и с `elixir` командата, отново с помощта на `mix`:

```bash
elixir -S mix run
```

Можем да си задаваме `Application`-ите като зависимости на други `Application`-и.
Прието е една библиотека в `Elixir`, която прави нещо в повече от един процес да е `OTP Application`.
Така нейните процеси ще могат да се ползват от други `Application`-и.

## Заключение

Приемете че един `OTP Application` може да бъде както библиотека, така и `executable` в `Elixir`.
Много от зависимостите, които ще задавате в `mix` проект ще са също така `Application`-и.
Даже `web framework`-а `Phoenix` е всъщност няколко `OTP Application`-и.

Следващата статия ще е кратка и ще разгледа абстракцията за създаване на малки `backgorund` задачи в `Elixir` - `Task`.
С нея ще приключим (засега) темата за процесите.
