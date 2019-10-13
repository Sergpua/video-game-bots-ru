# Внедрение данных на уровне ОС

## Windows API

Главная задача любой ОС – это управление программными и аппаратными ресурсами компьютера, а также предоставление к ним доступа для запущенных процессов. Аппаратные ресурсы мы уже рассматривали. Это – память, процессорное время, периферийные устройства. К программным ресурсам относятся все приложения и компоненты ОС, установленные на компьютере. Примером этого типа ресурсов являются системные библиотеки Windows, предоставляющие алгоритмы для решения различных задач.

В этой книге мы рассматриваем только ОС Windows. На ней вы сможете запустить [все приведённые примеры](https://github.com/ellysh/practical-video-game-bots). В дальнейшем для простоты под ОС всегда будем подразумевать Windows.

Иллюстрация 2-1 демонстрирует интерфейс ОС, через который предоставляется доступ к её ресурсам. Каждый запущенный процесс может обратиться к Windows с запросом на выполнение какого-то действия (например создание нового окна, отправки сетевого пакета, выделения дополнительной памяти и т.д.). Для каждого из таких запросов у ОС есть соответствующая [**функция**](https://ru.wikipedia.org/wiki/%D0%A4%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F_%28%D0%BF%D1%80%D0%BE%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5%29) (или подпрограмма). Функции, которые решают задачи из одной области (например работа с сетью), собраны в отдельные системные библиотеки.

![Windows API](windows-api.png)

_**Иллюстрация 2-1.** Доступ к ресурсам ОС через WinAPI_

Способ, которым процесс может вызвать системную функцию, строго определён, хорошо задокументирован и остаётся неизменным для данной версии ОС. Такое взаимодействие можно сравнить с юридическим договором: если процесс выполняет предварительные условия для вызова функции, ОС гарантирует указанный в документации результат. Такой договор называется **интерфейс прикладного программирования Windows** (Windows API или WinAPI).

Программное обеспечение очень гибко и легко меняется согласно возникающим требованиям. Так каждое обновление Windows вносит изменения в некоторые детали реализации ОС (например, в какую-то системную библиотеку). Эти детали реализации связаны между собой: типичный случай – одна библиотека вызывает функции другой. Таким образом, даже небольшое изменение может оказать значительное влияние на систему в целом. То же самое справедливо и для игрового приложения. Единственное, что позволяет программному обеспечению работать в этом море постоянных изменений – это надежные интерфейсы. Именно WinAPI гарантирует согласованное состояние системы и обеспечивает совместимость между новыми версиями ОС и приложения.

На иллюстрации 2-1 приведены два типа приложений. **Win32-приложение** взаимодействует с подмножеством системных библиотек через WinAPI интерфейс. Win32 – это историческое название, которое возникло в первой 32-битной версии Windows (Windows NT). Библиотеки, доступные через WinAPI, также известные как **WinAPI библиотеки**, содержат функции, оперирующие сложными абстракциями: элемент управления, файл и т.д.

Второй тип приложений называется **нативные** (native, поэтому иногда переводится как родной). Они взаимодействуют с более низкоуровневыми библиотеками и ядром Windows через **Native API**. Преимущество этих библиотек в том, что они становятся доступны на раннем этапе загрузки системы, когда многие функции ОС еще не работоспособны. Функции этих библиотек оперируют простыми абстракциями, такими как страница памяти, процесс, поток и т.д. Примеры нативных приложений: утилита для разбивки жёсткого диска, антивирус до старта ОС, программа восстановления Windows.

Библиотеки WinAPI вызывают функции низкоуровневых библиотек. Такой подход позволяет составлять сложные абстракции из более простых. Низкоуровневые библиотеки в свою очередь вызывают функции ядра.

Драйвера предоставляют упрощённое представление устройств для системных библиотек. Это представление включает в себя набор функций, которые выполняют характерные для данного устройства действия. WinAPI и низкоуровневые библиотеки обращаются к драйверам через функции ядра.

**Слой аппаратной абстракции** (Hardware Abstraction Layer или HAL) – это модуль ядра ОС, который предоставляет универсальный доступ к различному аппаратному обеспечению. HAL нужен, чтобы облегчить портирование и сопровождение Windows на новых аппаратных платформах. Функции HAL используются ядром ОС и драйверами устройств.

## Симуляция нажатий клавиш

Теперь мы рассмотрим технику симуляции нажатий клавиш. Это наиболее простой метод контроля ботом игрового приложения.

### Нажатия клавиш в активном окне

Рассмотрим, какие возможности предлагает AutoIt для решения нашей задачи. В [списке доступных функций](https://www.autoitscript.com/autoit3/docs/functions.htm) есть функция `Send`. Мы воспользуемся ею в тестовом скрипте, который будет нажимать клавишу "a" в окне приложения Notepad (Блокнот).

Алгоритм работы нашего скрипта выглядит следующим образом:

1. Найти окно Notepad среди всех открытых окон.

2. Переключится на него.

3. Симулировать нажатие клавиши "a".

Для поиска окна приложения мы воспользуемся функцией `WinGetHandle`. Её первый параметр является обязательным и может быть как заголовком окна, так и его классом. Функция возвращает **дескриптор** (handle) окна. Дескриптор – это структура данных, которая представляет некоторый ресурс или объект ОС. Большинство функций WinAPI оперируют этими структурами при работе с объектами.

Указывать класс окна при вызове функции `WinGetHandle` предпочтительнее. Всегда есть вероятность, что окна некоторых работающий приложений будут иметь одинаковые заголовки (например пустые).

Для чтения класса окна Notepad необходимо выполнить следующие шаги:

1. Запустить приложение Au3Info. Вы можете найти его в каталоге установки AutoIt. Путь к приложению по умолчанию: `C:\Program Files (X86)\AutoIt3\Au3Info.exe`.

2. Перетащить иконку "Finder Tool" на окно Notepad и отпустить.

Вы увидите результат, приведённый на иллюстрации 2-2.

![Приложение AutoIt3 Info](au3info.png)

_**Иллюстрация 2-2.** Приложение AutoIt3 Info_

Класс окна Notepad отображается на панели "Basic Info Window". Этот класс – "Notepad".

Скрипт `Send.au3`, представленный в листинге 2-1, симулирует нажатие клавиши "a".

_**Листинг 2-1.** Скрипт `Send.au3`_
```AutoIt
$hWnd = WinGetHandle("[CLASS:Notepad]")
WinActivate($hWnd)
Send("a")
```
В первой строке мы получаем дескриптор окна Notepad с помощью функции `WinGetHandle`. Далее мы переключаем фокус ввода на это окно функцией `WinActivate`. Последнее действие – симуляция нажатия клавиши "a".

Чтобы запустить этот скрипт, создайте в вашем редакторе исходного кода файл с именем `Send.au3` и скопируйте в него приведенный выше код. Запустите скрипт двойным щелчком по иконке этого файла.

### Функция Send

Функция `Send` представляет собой обертку над WinAPI вызовом. Мы можем выяснить что это за вызов с помощью приложения API Monitor, которое перехватит все обращения к WinAPI скрипта `Send.au3`.

Для подключения API Monitor к работающему процессу выполните следующие шаги:

1. Запустите 32-битную версию API Monitor.

2. Переключитесь на панель "API Filter" щелчком мыши. Нажмите комбинацию клавиш *Ctrl+F*, чтобы открыть диалог поиска. Введите в поле "Find what:" текст "Keyboard and Mouse Input" и нажмите кнопку "Find Next". Закройте диалог поиска и активируйте найденный флажок (check box) "Keyboard and Mouse Input".

3. Нажмите *Ctrl+M* для открытия диалога "Monitor New Process". Выберите приложение `AutoIt3.exe` в поле "Process" и нажмите кнопку "OK". По умолчанию путь к этому приложению должен быть следующий: `C:\Program Files (x86)\AutoIt3\AutoIt3.exe`.

4. В открывшемся диалоге "Run Script" выберите скрипт `Send.au3`. Сразу после этого начнётся его выполнение.

5. Переключитесь на панель "Summary" окна API Monitor. По нажатию *Ctrl+F* откройте диалог поиска и с его помощью найдите текст 'a' (с одинарными кавычками).

Иллюстрация 2-3 демонстрирует ожидаемый результат. Согласно перехваченным вызовам, `VkKeyScanW` – это единственная WinAPI функция, получившая символ "a" в качестве параметра. Если мы обратимся к [официальной документации WinAPI](https://docs.microsoft.com/en-us/windows/desktop/apiindex/windows-api-list), выяснится что эта функция не выполняет нажатия клавиши. Она вместе с функцией `MapVirtualKeyW` только подготавливает параметры для вызова `SendInput`, который и симулирует нажатие.

![Приложение API Monitor](api-monitor.png)

_**Иллюстрация 2-3.** Перехват вызовов WinAPI с помощью API Monitor_

Мы узнали достаточно, чтобы симулировать нажатие клавиши "a" напрямую через WinAPI вызовы. Удалим третью строчку скрипта `Send.au3` и заменим её новым блоком кода. При этом оставим первые два вызова `WinGetHandle` и `WinActivate` без изменений. Листинг 2-2 демонстрирует получившийся результат.

_**Листинг 2-2.** Скрипт `SendInput.au3`_
```AutoIt
$hWnd = WinGetHandle("[CLASS:Notepad]")
WinActivate($hWnd)

Const $KEYEVENTF_UNICODE = 4
Const $INPUT_KEYBOARD = 1
Const $iInputSize = 28

Const $tagKEYBDINPUT = _
    'word wVk;' & _
    'word wScan;' & _
    'dword dwFlags;' & _
    'dword time;' & _
    'ulong_ptr dwExtraInfo'

Const $tagINPUT = _
    'dword type;' & _
    $tagKEYBDINPUT & _
    ';dword padding;'

$tINPUTs = DllStructCreate($tagINPUT)
$pINPUTs = DllStructGetPtr($tINPUTs)
$iINPUTs = 1
$Key = AscW('a')

DllStructSetData($tINPUTs, 1, $INPUT_KEYBOARD)
DllStructSetData($tINPUTs, 3, $Key)
DllStructSetData($tINPUTs, 4, $KEYEVENTF_UNICODE)

DllCall('user32.dll', 'uint', 'SendInput', 'uint', $iINPUTs, _
        'ptr', $pINPUTs, 'int', $iInputSize)
```
Здесь мы используем функцию AutoIt `DllCall`. С её помощью можно вызвать код **динамической библиотеки** (DLL), написанной на языке C или C++. В данном случае мы делаем WinAPI вызов `SendInput`. Его входные параметры должны иметь типы, согласно документации WinAPI. Некоторые из этих типов AutoIt не поддерживает на уровне синтаксиса. Поэтому нам нужны дополнительные шаги, чтобы подготовить эти параметры.

Таблица 2-1 демонстрирует входные параметры функции `DllCall`.

_**Таблица 2-1.** Входные параметры функции `DllCall`_

| Параметр | Описание |
| --- | --- |
| `user32.dll` | Имя библиотеки, функцию которой требуется вызвать. |
| `uint` | Тип возвращаемого значения функции. |
| `SendInput` | Имя функции. |
| `uint`, `$iINPUTs`<br/>`ptr`, `$pINPUTs`<br/>`int`, `$iInputSize` | Пары тип-переменная. Переменные являются входными параметрами функции. |

Согласно WinAPI документации, декларация функции `SendInput` выглядит следующим образом:
```C++
UINT SendInput(UINT cInputs, LPINPUT pInputs, int cbSize);
```
Строчку вызова функции `DllCall` на AutoIt можно представить эквивалентом на языке C++:
```C++
SendInput(iINPUTs, pINPUTs, iInputSize);
```
Рассмотрим входные параметры, переданные нами в `SendInput`:

1. `iINPUTs` – количество структур типа `INPUT`, которые передаются вторым параметром.

2. `pINPUTs` – указатель на массив структур типа `INPUT` из одного элемента. Этот массив подготавливается в несколько этапов. Сначала мы объявляем строки `KEYBDINPUT` и `INPUT` с описанием полей соответствующих структур. При этом `KEYBDINPUT` является вторым полем `INPUT`. Такое отношение называется **вложенные структуры** (nested structure). На следующем шаге создаются структуры в формате языка C++ через вызов `DllStructCreate`. Результат сохраняется в переменной `tINPUTs`. С помощью функции `DllStructGetPtr` мы получаем указатель на эту структуру и помещаем его в `pINPUTs`. Запись значений полей C++ структуры происходит через вызов `DllStructSetData`. Обратите внимание, что вторым параметром в `DllStructSetData` передаётся номер поля, начиная с единицы. В случае вложенных структур их поля нумеруются последовательно. То есть элемент 1 соответствует полю `dword type` структуры `INPUT`, а элемент 3 – полю `word wScan` структуры `KEYBDINPUT`.

3. `iInputSize` – размер одной структуры `INPUT` в байтах. В нашем случае это константное значение, рассчитанное по формуле:
```
dword + (word + word + dword + dword + ulong_ptr) + dword =
4 + (2 + 2 + 4 + 4 + 8) + 4 = 28
```
Слагаемые в скобках – это размеры полей вложенной структуры `KEYBDINPUT`.

Может быть непонятно, откуда взялись последние четыре байта в приведённой выше формуле. Возможно, вы обратили внимание, что объявленная в скрипте `Send.au3` структура `INPUT` имеет последнее поле типа `dword` с именем `padding` (набивка). Оно не используется и служит для [**выравнивания данных**](https://ru.wikipedia.org/wiki/%D0%92%D1%8B%D1%80%D0%B0%D0%B2%D0%BD%D0%B8%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85). Рассмотрим это поле подробнее.

Определение структуры `INPUT` согласно документации WinAPI выглядит следующим образом:
```C++
typedef struct tagINPUT {
  DWORD type;
  union {
    MOUSEINPUT    mi;
    KEYBDINPUT    ki;
    HARDWAREINPUT hi;
  };
} INPUT, *PINPUT;
```
Вложенная структура `KEYBDINPUT` на самом деле помещена в блок `union` с другими структурами `MOUSEINPUT` и `HARDWAREINPUT`. Это означает, что под все три структуры будет выделена одна и та же область памяти. Но использоваться она будет только одной из них. Так как область одна, её размер должен соответствовать самой большой структуре, которой является `MOUSEINPUT`. Она больше `KEYBDINPUT` на одно поле типа `dword`, т.е. на четыре байта. Именно из-за него мы добавили `padding` в наше определение `KEYBDINPUT` для выравнивания.

Скрипт `SendInput.au3` демонстрирует преимущества высокоуровневых языков, таких как AutoIt. Они скрывают от пользователя множество несущественных деталей. Это позволяет оперировать простыми абстракциями и функциями. Кроме того, приложения, написанные на таких языках, короче и яснее.

### Нажатия клавиш в неактивном окне

Функция AutoIt `Send` симулирует нажатия клавиш в активном окне. Другими словами, вы не можете свернуть это окно или переключится на другое, что в некоторых случаях неудобно. Функция `ControlSend` позволяет обойти такое ограничение. Мы можем переписать скрипт `Send.au3` с использованием `ControlSend`, как демонстрирует листинг 2-3.

_**Листинг 2-3.** Скрипт `ControlSend.au3`_
```AutoIt
$hWnd = WinGetHandle("[CLASS:Notepad]")
ControlSend($hWnd, "", "Edit1", "a")
```
Третьим параметром в функцию `ControlSend` передается [**элемент интерфейса**](https://ru.wikipedia.org/wiki/%D0%AD%D0%BB%D0%B5%D0%BC%D0%B5%D0%BD%D1%82_%D0%B8%D0%BD%D1%82%D0%B5%D1%80%D1%84%D0%B5%D0%B9%D1%81%D0%B0) (control), который получает симулируемое нажатие клавиши. Указать на него можно несколькими способами. В нашем случае, мы передаём класс элемента "Edit1". Его можно узнать с помощью утилиты Au3Info точно так же, как и класс окна.

Применив API Monitor, мы узнаем, что `ControlSend` внутри себя вызывает WinAPI функцию `SetKeyboardState`. В качестве упражнения предлагаю вам переписать скрипт `ControlSend.au3` так, чтобы он вызывал `SetKeyboardState` напрямую.

Скрипт `ControlSend.au3` работает корректно во всех случаях, кроме симуляции нажатия клавиши в развернутом на весь экран окне приложения DirectX. Проблема заключается в том, что такое окно не имеет элементов интерфейса. Чтобы её решить, достаточно просто не указывать третий параметр `controlID` функции `ControlSend`. Листинг 2-4 демонстрирует исправленный скрипт.

_**Листинг 2-4.** Скрипт `ControlSendDirectX.au3`_
```AutoIt
$hWnd = WinGetHandle("Warcraft III")
ControlSend($hWnd, "", "", "a")
```
Этот скрипт ищет окно игры Warcraft 3 по его заголовку и симулирует в нём нажатие клавиши "a". Узнать заголовок окна DirectX-приложения иногда бывает сложно, потому что не всегда можно выйти из полноэкранного режима. В этом случае утилиты вроде Au3Info вам не помогут. Вместо них с этой задачей справится API Monitor. Если в окне приложения вы наведёте курсор мыши на интересующий вас процесс на панели "Running Process", вы увидите заголовок окна этого приложения, как показано на иллюстрации 2-4.

![Заголовок окна приложения в API Monitor](api-monitor-title.png)

_**Иллюстрация 2-4.** Чтение заголовка окна приложения в API Monitor_

Если вы не можете найти нужный процесс на панели "Running Process", попробуйте запустить API Monitor с правами администратора. Если это не помогло, и у вас установлена 64-битная версия Windows, надо запустить обе версии API Monitor – 32- и 64-битную. В одной из них процесс должен появиться.

Заголовок некоторых окон в полноэкранном режиме пустой. Из-за этого вы не сможете передать его в функцию `WinGetHandle` и получить дескриптор. Тогда альтернативным решением будет передавать класс окна. К сожалению, с помощью API Monitor эту информацию не удастся прочитать.

Чтобы получить класс окна, открытого в полноэкранном режиме, вы можете воспользоваться скриптом AutoIt, приведённом в листинге 2-5.

_**Листинг 2-5.** Скрипт `GetWindowTitle.au3`_
```AutoIt
#include <WinAPI.au3>

Sleep(5 * 1000)
$handle = WinGetHandle('[Active]')
MsgBox(0, "", "Title   : " & WinGetTitle($handle) & @CRLF _
       & "Class : " & _WinAPI_GetClassName($handle))
```
После запуска скрипт ждёт пять секунд, в течение которых вы должны переключиться на интересующее вас окно. После этого его заголовок и класс будут выведены в открывшемся диалоговом окне.

Рассмотрим подробнее скрипт `GetWindowTitle.au3`. В первой строке стоит **ключевое слово** (keyword) `include`. С его помощью AutoIt включает содержание указанного скрипта `WinAPI.au3` в текущий. В `WinAPI.au3` реализована нужная нам функция `_WinAPI_GetClassName`. Она возвращает класс окна по его дескриптору. Далее с помощью функции `Sleep` скрипт ждёт пять секунд. После этого дескриптор активного в данный момент окна сохраняется в переменную `handle`. Функция `MsgBox` создаёт диалоговое окно, в котором выводится результат. Заголовок окна возвращает функция `WinGetTitle`.

## Симуляция действий мыши

В некоторых играх для управления персонажем достаточно только клавиатуры. Однако в большинстве случаев игрок должен пользоваться и клавиатурой, и мышью. AutoIt предлагает несколько функций, которые позволят симулировать основные действия мыши: щелчки, перемещение курсора, зажимание кнопки.

### Действия мыши в активном окне

Мы воспользуемся графическим редактором Microsoft Paint для тестирования скриптов, симулирующих действия мыши. Самое простое действие – это однократный щелчок в указанной точке экрана. Листинг 2-6 демонстрирует соответствующий скрипт.

_**Листинг 2-6.** Скрипт `MouseClick.au3`_
```AutoIt
$hWnd = WinGetHandle("[CLASS:MSPaintApp]")
WinActivate($hWnd)
MouseClick("left", 250, 300)
```
Для тестирования этого скрипта выполните следующее:

1. Запустите приложение Paint.

2. Переключитесь на инструмент "Кисти" (Brushes).

3. Запустите скрипт `MouseClick.au3`.

Скрипт нарисует чёрную точку по координатам x = 250, y = 300. Их корректность вы можете проверить с помощью утилиты ColorPix.

Для симуляции щелчка мыши мы использовали функцию AutoIt `MouseClick`. Она принимает пять входных параметров, первые три из которых являются обязательными:

1. Кнопка мыши для щелчка. Основные варианты: левая (left), правая (right), средняя (middle).

2. Координата X позиции курсора.

3. Координата Y позиции курсора.

4. Число последовательных щелчков.

5. Скорость мыши для перемещения курсора в указанные координаты.

Внутри себя `MouseClick` вызывает WinAPI функцию `mouse_event`.

Координаты позиции курсора можно задавать в одном из трёх режимов, представленных в таблице 2-2. 

_**Таблица 2-2.** Режимы координат, поддерживаемые WinAPI_

| Режим | Описание |
| --- | --- |
| 0 | Координаты относительно левой верхней точки активного окна. |
| 1 | Абсолютные координаты экрана. Это режим по умолчанию. |
| 2 | Координаты относительно левой верхней точки клиентской части окна (без заголовка, меню и границ). |

![Режимы координат](mouse-coordinate-types.png)

_**Иллюстрация 2-5.** Режимы координат, поддерживаемые WinAPI_

Рассмотрим иллюстрацию 2-5. Каждый номер соответствует режиму координат из таблицы 2-1. Например, точка с номером "0" демонстрирует режим относительно активного окна. Её координаты X0 и Y0.

Функция AutoIt `Opt`, вызванная с первым параметром `MouseCoordMode`, позволяет выбрать режим координат для текущего скрипта. Листинг 2-7 демонстрирует выбор координат относительно клиентской части окна в скрипте `MouseClick.au3`.

_**Листинг 2-7.** Скрипт `MouseClick.au3` с выбором режима координат_
```AutoIt
Opt("MouseCoordMode", 2)
$hWnd = WinGetHandle("[CLASS:MSPaintApp]")
WinActivate($hWnd)
MouseClick("left", 250, 300)
```
Запустив этот скрипт, вы заметите, что координаты чёрной точки, нарисованной в окне Pain, изменились. Выбранный нами режим обеспечивает более точное позиционирование курсора. При разработке кликеров предпочтительнее использовать именно его. Он одинаково хорошо работает для окон в обычном и полноэкранном режимах. Единственный его недостаток заключается в сложности отладки скриптов. Утилиты вроде ColorPix отображают только абсолютные координаты пикселей.

Одно из распространённых действий мышью в компьютерных играх – перетаскивание (drag-and-drop). Для его симуляции AutoIt предоставляет функцию `MouseClickDrag`. Листинг 2-8 демонстрирует её использование.
 
_**Листинг 2-8.** Скрипт `MouseClickDrag.au3`_
```AutoIt
$hWnd = WinGetHandle("[CLASS:MSPaintApp]")
WinActivate($hWnd)
MouseClickDrag("left", 250, 300, 400, 500)
```
После запуска скрипт `MouseClickDrag.au3` рисует линию в окне Paint. Координаты её начала: x = 250, y = 300. Она заканчивается в точке x = 400, y = 500. Функция AutoIt `MouseClickDrag` делает внутри себя уже знакомый нам WinAPI вызов `mouse_event`. Обе AutoIt функции `MouseClick` и `MouseClickDrag` симулируют действия мыши только в активном окне.

### Действия мыши в неактивном окне

AutoIt предоставляет функцию `ControlClick`, которая симулирует щелчок мыши в неактивном окне. Пример её использования приведён в листинге 2-9.

_**Листинг 2-9.** Скрипт `ControlClick.au3`_
```AutoIt
$hWnd = WinGetHandle("[CLASS:MSPaintApp]")
ControlClick($hWnd, "", "Afx:00000000FFC20000:81", "left", 1, 250, 300)
```
Скрипт `ControlClick.au3` симулирует щелчок мыши в неактивном или свернутом окне Paint. По принципу работы функция `ControlClick` похожа на `ControlSend`. Вы должны указать элемент интерфейса по которому будет выполнен щелчок. В нашем случае – это рабочая область окна Paint, в которой пользователь применяет инструменты рисования (например кисти). Согласно информации от утилиты Au3Info, элемент рабочей области имеет класс "Afx:00000000FFC20000:81".

Ради эксперимента мы можем передать одни и те же координаты курсора в функции `MouseClick` и `ControlClick`. В результате щелчки мыши произойдут в разных точках экрана. Причина в том, что входные параметры функции `ControlClick` – это координаты относительно левого верхнего угла указанного элемента интерфейса. В случае скрипта `ControlClick.au3`, щелчок произойдёт в точке x = 250, y = 300 относительно левого верхнего угла рабочей области. Тогда как режим координат для функции `MouseClick` определяется параметром `MouseCoordMode`.

`ControlClick` дважды вызывает WinAPI функцию `PostMessageW` внутри себя. Иллюстрация 2-6 демонстрирует её вызовы, перехваченные с помощью API Monitor.

![Режимы координат](controlclick-winapi.png)

_**Иллюстрация 2-6.** Внутренние вызовы функции `ControlClick`_

При вызове функции `PostMessageW` первый раз, в неё передаётся параметр `WM_LBUTTONDOWN`. В результате симулируется нажатие кнопки мыши и её удержание. Во втором вызове передаётся параметр `WM_LBUTTONUP`, что соответствует отпусканию кнопки мыши.

Функция `ControlClick` работает ненадежно со свернутыми окнами DirectX. В некоторых случаях щелчок мыши полностью игнорируется. Иногда он отрабатывает не в момент вызова `ControlClick`, а только после того, как свернутое окно будет восстановлено.

## Выводы

Мы рассмотрели функции AutoIt, которые позволяют симулировать наиболее распространённые действия клавиатуры и мыши в окне игрового приложения. Эти функции делятся на два типа. Первый тип симулирует действия устройства только в активном окне. Второй тип работает как с активными, так и с неактивными или свёрнутыми окнами. Главный недостаток функций второго типа – недостаточная надёжность, поскольку некоторые приложения игнорируют симулируемые ими действия. Поэтому для реализации кликеров рекомендуется использовать функции первого типа.