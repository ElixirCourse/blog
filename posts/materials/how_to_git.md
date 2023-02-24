---
title_image_path: git_logo.jpg
category: Програма
created_at: 2023-02-24T17:00:00
tags:
  - git
  - github
  - homework
---

# Получаване и изпращане на домашни

За домашните в курса *Функционално програмиране с Elixir* се използва [Github Classroom](https://classroom.github.com/).
Използвайки тази платформа, за да получите и предадете домашно, трябва да направите следните стъпки:

- Да имате профил в [Github](https://github.com).
- Да посетите [сайта на курса](https://elixir-lang.bg/), където ще бъде публикуван линк към домашното.
- Когато посетите този линк и натиснете бутона `Accept this assignment`, ще бъде създадено специално хранилище, до което достъп имате само Вие, в следната [Github организация](https://github.com/orgs/ElixirCourseHW/repositories).
- В хранилището ще намерите условието и инструкциите за изпълнение на домашното.
- Преди да изтече крайният срок, Вашето решение трябва да бъде добавено в хранилището.

# Работа с Github

## Използвайки графичния интерфейс

Най-лесният начин за работа с Github е с помощта на графичния интерфейс.
Може да добавяте нови файлове с бутона `Add files`.
Създадени файлове може да редактирате като ги отворите и натиснете бутона `Edit` (с иконка на молив/химикалка), направите желаните промени
чрез предоставения редактор и натиснете бутона `Commit changes`.

**Не** ви препоръчваме да използвате графичния интерфейс. Уменията за работа с git от командния ред са необходими умения за всеки програмист.

## Използвайки командния ред

### Github CLI

Най-лесният начин за интеракция с Github от командния ред е с помощта на [Github CLI](https://cli.github.com/).
След като сте го инсталирали Github CLI, може да преминете към автентикацията.

#### Автентикация

Отворете терминал и изпълнете следната команда:

```bash
$ gh auth login
```

Ще видите следното:

```bash
? What account do you want to log into?  [Use arrows to move, type to filter]
> GitHub.com
  GitHub Enterprise Server
```

Изберете `Github.com`.
След това ще бъдете помолени да изберете начин за автентикация:

```bash
? What is your preferred protocol for Git operations?  [Use arrows to move, type to filter]
> HTTPS
  SSH
```

Изберете `HTTPS`. На следващата стъпка ще бъдете помолени да се съгласите да използвате Github за удостоверяване. Съгласете се.

```bash
Authenticate Git with your GitHub credentials?  [Y/n] y
```

На следащата стъпка изберете автентикация чрез уеб браузър. Ще ви бъде показан код (например `ABCD-1234`), който трябва да въведете в браузъра.

```bash
? How would you like to authenticate GitHub CLI?  [Use arrows to move, type to filter]
> Login with a web browser
  Paste an authentication token
```

При правилно въвеждане на кода и следване на останалите стъпки в браузъра, във Вашия терминал трябва да видите следното:

```bash
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as IvanIvanoff
```

Това означава, че автентикацията е успешна.
Ако желаете да смените протокола, може да изпълните:

```bash
gh config set -h github.com git_protocol ssh
```

#### Работа с Github CLI

След като сте настроили Github CLI, за да клониранете вашето хранилище може да изпълните: следната команда, използвайки Вашето хранилище:

```bash
$ gh repo clone ElixirCourseHW/homework-1-year-2023-IvanIvanoff
```

Преди да започнем работа, трябва да създадем нов клон в нашето хранилище, което ще съдържа промените:

```bash
$ cd homework-1-year-2023-IvanIvanoff
$ git checkout -b  my-solution
```

Нека да добавим файл `hello.txt` в хранилището:

```bash
$ touch hello.txt
$ echo "Hello, world!" > hello.txt
```

След това може да изпълним следната команда, за да съхраним файла в нашето локално хранилище:

```bash
$ git add hello.txt
$ git commit -m "Adding hello.txt"
```

За да качим промените в Github хранилището, може да изпълним следната команда:

```bash
$ gh pr create
➜  homework-1-year-2023-IvanIvanoff git:(my-solution) gh pr create
? Where should we push the 'my-solution' branch?  [Use arrows to move, type to filter]
> ElixirCourseHW/homework-1-year-2023-IvanIvanoff
  Create a fork of ElixirCourseHW/homework-1-year-2023-IvanIvanoff
  Skip pushing the branch
  Cancel
```

Изберете първата възможност, въведете заглавие, не въвежайте допълнително описание (или въведете, ако искате) и натиснете `Enter`.

```bash
Creating pull request for my-solution into main in ElixirCourseHW/homework-1-year-2023-IvanIvanoff

? Title my solution
? Body <Received>
? What's next?  [Use arrows to move, type to filter]
  Submit
  Submit as draft
> Continue in browser
  Add metadata
  Cancel
```

В показаното меню изберете `Continue in browser`, което ще отвори нова страница в браузъра, където можете да видите Вашите промени, да промените заглавието или описанието и да създадете Pull Request.

След създаването на този Pull Request, може да натиснете бутона `Merge pull request` и вашите промени ще бъдат добавени към `main` клона. С извършването на тази стъпка се счита, че Вашето домашно е предадено успешно. Може да правите промени неограничен брой пъти преди изтичането на крайния срок. Когато крайният срок изтече, правата за писане в това хранилище Ви се отнемат автоматично.

За да синхронизирате локалното хранилище с хранилището в Github, може да изпълните следната команда:

```bash
$ gh repo sync
```

и ще видите следното:

```bash
✓ Synced the "main" branch from ElixirCourseHW/homework-1-year-2023-IvanIvanoff to local repository
```

За да започнете работа върху нова задача, върнете се обратно в `main` клона и създайте нов клон и изпълнете стъпките отново. Нужно е да го направите от `main` клона, за да започнете новата задача от състоянието, което съдържа последните промени.

```bash
$ git checkout main
$ git checkout -b fix-known-bugs-in-solution
```

Забележка: Това не е единственият начин за работа с `git` и `Github`, но е един от най-лесните. При добро познаване на `git`, работният процес може да протече по различен начин.

### SSH автентикация

Настройването на SSH автентикация е една идея по-сложно от използването на Github CLI и няма да го описваме подробно тук.
Инструкции може да намерите [тук](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)