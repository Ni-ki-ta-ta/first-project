# Терминал Windows и Linux

## Что такое терминал

Терминал — текстовый интерфейс для управления компьютером командами. Через него запускают программы, работают с файлами, Git, Docker и серверами.

В Windows часто используют PowerShell или CMD. В Linux — Bash или другую оболочку.

## Где я нахожусь?

PowerShell:
```powershell
Get-Location
```
Показывает текущую папку.

Linux:
```bash
pwd
```
Печатает путь текущей папки.

## Посмотреть файлы

PowerShell:
```powershell
Get-ChildItem
Get-ChildItem -Force
```
Второй вариант также показывает скрытые элементы.

Linux:
```bash
ls
ls -la
```
`-l` — подробный список, `-a` — включая скрытые файлы.

## Перейти в папку

```powershell
Set-Location C:\Dev\project
```
```bash
cd /home/user/project
```

`..` означает родительскую папку:
```bash
cd ..
```

## Создать папку и файл

PowerShell:
```powershell
New-Item -ItemType Directory notes
New-Item -ItemType File notes.txt
```

Linux:
```bash
mkdir notes
touch notes.txt
```

## Копирование и перемещение

PowerShell:
```powershell
Copy-Item old.txt copy.txt
Move-Item copy.txt notes/
```

Linux:
```bash
cp old.txt copy.txt
mv copy.txt notes/
```

## Удаление — осторожно

PowerShell:
```powershell
Remove-Item old.txt
```

Linux:
```bash
rm old.txt
```

Удаление папки рекурсивно может стереть много данных. Перед такой командой я проверяю путь и содержимое.

## Поиск текста

PowerShell:
```powershell
Select-String -Path *.java -Pattern "TODO"
```

Linux:
```bash
grep -R "TODO" .
```

Ищет текст `TODO` в файлах текущей папки и вложенных каталогах.

## Процессы

PowerShell:
```powershell
Get-Process
Get-Process -Name java
```

Linux:
```bash
ps aux
top
```

Не завершай незнакомый процесс, пока не понял, за что он отвечает.

## Переменные окружения

PowerShell:
```powershell
$env:JAVA_HOME
$env:PATH
```

Linux:
```bash
echo "$JAVA_HOME"
printenv
```

Переменные окружения часто используют для настройки JDK, приложений и секретов. Не публикуй значения, содержащие пароли или токены.

## Полезные привычки

- Перед удалением я проверяю `pwd` / `Get-Location`.
- Пути с пробелами заключай в кавычки.
- Я использую `--help` или справку, если не уверен в параметрах.
- Я не запускаю неизвестные команды из интернета с правами администратора.

## Аргументы, флаги и справка

В команде `ls -la`:
- `ls` — имя команды;
- `-l` и `-a` — флаги, меняющие поведение;
- путь после команды — аргумент, к которому применяется действие.

Справка:
```bash
command --help
man command
```
В PowerShell:
```powershell
Get-Help Get-ChildItem
Get-Command java
```

## Перенаправление и конвейер

```bash
command > output.txt       # записать вывод, перезаписав файл
command >> output.txt      # дописать вывод в конец
command1 | command2       # передать вывод первой команды второй
```
В PowerShell также используются `>` и `|`, но некоторые команды и типы данных отличаются от Bash.

## Коды завершения

Программа обычно возвращает код завершения: `0` означает успешное выполнение, ненулевой код — ошибку или особый результат. В Bash код последней команды доступен через `$?`; в PowerShell — через `$LASTEXITCODE` для внешних программ.

## Java из терминала

```bash
java -version       # версия среды запуска
javac -version      # версия компилятора
where.exe java      # Windows: где найден java.exe
which java          # Linux/macOS: путь к java
```
Если команда не найдена, я проверяю установку JDK и PATH. `JAVA_HOME` обычно указывает на каталог JDK, а PATH содержит путь к его `bin`.

## Безопасность команд

Перед `rm -r`, `Remove-Item -Recurse`, `git clean -fd` и похожими командами я проверяю текущую папку и точный путь. Не вставляй команды с `sudo` или правами администратора, если не понимаешь каждую часть.

