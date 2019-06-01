# Приёмы защиты от кликеров

Мы узнали основные принципы работы кликеров. Теперь попробуем поменять сторону и посмотреть на них с позиции разработчиков систем защиты. Как можно обнаружить ботов этого типа? На этот вопрос мы попробуем ответить в этом разделе.

В первой главе мы рассмотрели архитектуру типичного игрового приложения. Как вы помните, оно имеет две части: клиентскую и серверную. Зачастую, система защиты придерживается той же архитектуры и разделена на две части. Клиентская часть контролирует точки перехвата и внедрения данных на стороне пользователя (драйвера, ОС, приложение). Серверная часть следит за взаимодействием игрового приложения и сервером. Большинство техник по обнаружению кликеров работают на клиентской стороне.

Главная цель любой системы защиты - обнаружить факт несанкционированного чтения или модификации игровых данных. Именно этим занимаются все боты.

Когда обнаружить нарушение удалось, у системы защиты есть несколько вариантов реакции:

1. Уведомить администратора игрового сервера о подозрительных действиях игрока. Для этого достаточно сделать запись в лог файл на стороне сервера.

2. Разорвать соединение между подозрительным пользователем и сервером.

3. Заблокировать подозрительного игрока по IP адресу. Это предотвратит его дальнейшие попытки подключения к серверу.

Мы рассмотрим только алгоритмы обнаружения ботов, но не способы блокировки их работы. Также уделим внимание возможным уязвимостям этих алгоритмов.