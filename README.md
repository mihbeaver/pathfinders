# pathfinders

Игра про поиск сокровищ в лабиринте. По лабиринту раскиданы золотые монеты, которые собирают полностью автоматические роботы. Задача игроков - написать программу управления роботом для сбора монет. Надо помнить, что другие роботы не стоят на месте и тоже ищут сокровища. Побеждает тот, чей робот собрёт больше монет.

Оригинальная идея игры заимствована из проекта [mipt-cs-labyrinth-challenge](https://github.com/avasyukov/mipt-cs-labyrinth-challenge)

## Как запустить игру?

Для работы игра требует python2.7 и [Flask](http://flask.pocoo.org/) для web интерфейса. Рекомендую поставить Flask из репозитария вашего дистрибутива Linux. Тестирование производилось на Linux (Fedora 25). Работоспособность на Windows не гарантируется.

1. Склонировать репозитарий из Github
2. Запустить из командной строки 

```
$ ./pathfinders.py 
Process 'gosha' started
Process 'masha' started
...
```

3. Открыть в браузере [http://127.0.0.1:5000/](http://127.0.0.1:5000/) На странице должен отобразится состояние игрового поля.
4. Для завершения игры необходимо завершить процесс `pathfinder.py` в консоли при помощи Ctrl-C

## Принципы игры

При старте приложения игра загружает карту лабиринта (настраивается в конфигурационном файле) и случайным образом раскидывает сокровища по карте. Из директории с программами для роботов загружаются все файлы с расширением .py. Каждый файл - программа для отдельного робота. Путь к файлу карты и директории с программами игроков настраиваемся в конфигурационном файле.

Программа управления роботом (далее просто "программа") является обычным python файлом с расширение .py. Имя игрока (в web интерфейсе) будет таким же как и имя файла программы. В программе должна быть обязательно определена функция `move(info, ctx)`. Это основная функция управления роботом. В неё передается текущая информация о карте, расположении сокровищ и других игроков. Функция должна вернуть число от 0 до 3, которое обозначает направление следующего хода. Подробности о написании программ управления роботом можно узнать из соответсвующего раздела. Начальные положения роботов задаются случайно.

Игра вызывает программу управления роботом через определенные интервалы времени. Если функция `move()` не успеет выполнить нужные расчеты за указанное время, то робот остается не месте и никуда не двигается на текущем ходу. Если в программе робота случается ошибка, то он выходит из строя до конца игры и больше не может собирать сокровища.

В некоторых случаях некорректно написанная программа управления роботом не позволит игре запуститься. Например, если в файле программы не определена функция move(). В этом случае надо либо исправить ошибку в программе робота, либо удалить файл глючной программы. 

## Интерфейс

Web интерфейс доступен после запуска игры по адресу [http://127.0.0.1:5000/](http://127.0.0.1:5000/). Отображается игровое поле и таблица результатов. Единственное неочевидное поле в таблице - timeout. Это количество ходов, которое робот пропустил из-за того, что функция `move()` выполнялась дольше необходимого лимита.

## Настройки игры

Все настройки игры хранятся в файле `config`. Его нельзя переименовывать и перемещать. В файле конфигурации две секции: `pathfinders` и `web`. Секция `pathfinders` содержит настройки самой игры.

* `map_file` - путь к файлу с картой. По соглашению карты лежат в директории `maps`.
* `move_timeout` - количество секунд на 1 ход для каждого игрока. Может быть дробным.
* `players` - директория с программами для роботов. Менять не нужно.
* `num_coins` - количество монет на карте. Монеты располагаются на карте в случайном порядке.
* `max_moves` - максимальное количество ходов в игре. После достижения максимального количество ходов игра прекращается даже если ещё остались монеты.

Секция `web` содержит только один параметр: `field_size` - размер клетки игрового поля в пикселях в web интерфейсе. Если поле не вмещается на один экран этот параметр можно уменьшить.

После изменения параметров игры необходимо перезапустить игру и, в ряде случаев, обновить интерфейса страницу в браузере.

## Собственные карты

Составлять собственные карты легко. Необходимо создать файл с картой (лучше в директории `maps`) и прописать путь к файлу карты в конфигурационный файл игры.

Файл игры выглядит так:
```
#......#........#
#...###...####.#.
....#.....#..#...
....#.......#...#
..#....##........
.......#....###..
.......#....#....
.......#....#....
....####.........
```

`.` - пустое поле
`#` - стена

После изменения карты в конфигурационном файле следует не только перезапустить игру, но и обновить страницу в браузере. 

## Программирование робота

Для добавления робота достаточно положить файл программы в директорию `players` (Директорию можно изменить в конфиге). Требования к программе:

1. Файл должен иметь расширение `.py`
2. Файл должен содержать синтаксически верный python код.
3. В файле должна быть определена функция с двумя агрументами `move(info, ctx)`

Если все три условия соблюдены, то при запуске игры появится игрок имя которого будет совпадать с именем файла (без `.py`). Передвижение робота на каждом ходе задается при помощи функции `move(info, ctx)`. Функция должна возвращать значение от 0 до 3, которое обозначает направление следующего шага робота:
 
* 0 - вверх
* 1 - вниз
* 2 - влево
* 3 - вправо

Если функция вернет любое другое значение, то робот останется на месте на этом ходу, но может продолжить движения на последующих ходах. Если в функции возникнет ошибка (вызовется исключение), то робот до конца игры останется на месте.

Для вычисления следующего хода функция может использовать информацию о текущей игровой ситуации. Информация о карте лабиринта, расположении монет и соперников содержится в переменной info, который является первым аргументом функции. info - это словарь, который может содержать следующие поля:
 
* `info["x"]` - текущая координата робота на игровом поле по горизонтали (отсчитывается слева начинается с 0)
* `info["y"]` - аналогично по вертикали (отсчитывается сверху начинается с 0)
* `info["coins"]` - координаты монет. Список tuples (x, y), например `info["coins"] = [(1, 3), (3, 0), (0, 4)]`
* `info["players"]` - координаты других игроков. Список tuples (x, y), например `info["playes"] = [(2, 2)]`
* `info["map"]` - карта лабиринта. Представлена в виде двумерного массива из 0 и 1. 1 - стена, 0 - свободное поле.

Все координаты отсчитываются от верхнего левого угла поля начиная с (0, 0). В последующих редакциях игры состав полей в словаре `info` может меняться.

Необходимо отдельно отметить устройство карты. Карта это двумерный массив, то есть список списков. Например:

```python
info["map"] = [
[0, 0, 0, 1, 0],
[0, 0, 0, 0, 0],
[0, 0, 0, 0, 0],
[1, 1, 0, 1, 0]
]
```
Карта - это список "строк", поэтому чтобы узнать, что находится в точке с координатами x, y необходимо обратиться к элементу `info["map"][y][x]`, а не к `info["map"][х][y]`.

Второй агрумент функции - пустой словарь `ctx` в который можно записывать произвольные поля. При следующем вызове функции эти данные стануться в словаре `ctx`. Такми образом его можно использовать для хранения результатов вычислений между последующими вызовами функции `move()`.

## Примеры программ управления роботом

### "Всегда вверх"

```python
def move(info, ctx):
    return 0 # Движемся вверх всегда
```

### "Наблюдатель"

```python
def move(info, ctx):
    # Ничего не делаем печатаем карту и список сокровищ
    print info["map"]
    print info["coins"] 
    # Робот никуда не двинется, проигнорирует ошибочное значение
    return None
```

### "Глючный робот"

```python
def move(info, ctx):
    return 10/0
```
После старта игры вываливается ошибка.
```
ERROR. Process 'robot' on move function. integer division or modulo by zero.
```
Робот больше не двигается, но для остальных игра продолжается.

### "ЕГГОГ"

Некорректный код на Python: нет двоеточия, табуляция. 

```python
def move(info, ctx)
return 0
```
Игра не запускается с ошибкой:
```
ERROR: Error loading player robot: invalid syntax (robot.py, line 3)
```
Необходимо или поправить ошибку, или удалить файл с ошибкой из директории с программами роботов.

### "Случайные блуждания"

В диретории `players` уже есть два примера программ для роботов: chuck и rocky. В обоих случаях направление следующего хода выбирается случайно. Но в алгоритме chuck при этом проверяется то, что соответсвующий ход возможен. Обратите внимание на результаты соревнований двух алгоритмов.