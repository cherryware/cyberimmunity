# Кибериммунное устройство мониторинга оборудования (КУМО)

## Disclaimer 

This is a demo project and shall not be used in production.
The code is distributed under MIT license (see the LICENSE file).

## Краткое описание продукта

Кибериммунное устройство мониторинга оборудования (КУМО) получает данные от промышленного оборудования из внутренней сети, обрабатывает эти данные, в случае превышения пороговых значений генерирует события, которые пользователь из внешней сети может получить для дальнейшего анализа. Должна быть возможность осуществить безопасное обновление КУМО (полное или частичное).

Предполагается использовать КУМО в том числе на заводе по производству лекарств.

## Ценности продукта и непримемлемые события в их отношении

| Ценность | Непримемлемые события |
|---|---|
| confidential data | unauthorised access	|
| data integrity | loss or corruption of data	|
| code protection | unauthorised access, code modification |

## Цели и предположения безопасности (ЦПБ)

### Цели безопасности:
1. Доступ к информации только авторизованными пользователями.
2. Данные защищены от компрометации.
3. Применяются только правильные обновления. (Правильные – прошедшие некий набор проверок, который мы определили как достаточный).

### Предположения безопасности:
1. Потенциальный злоумышленник не имеет физического доступа к устройству.
2. Не рассматриваются проблемы доставки обновления на устройство.
3. Оборудование и внутренняя сеть предприятия - безопасны.

## Архитектура решения

### Компоненты

| Название | Назначение | Комментарий |
|----|----|----|
|*File server* | хранит файлы с обновлением | внешний по отношению к системе сервис, имитатор сервера с обновлениями в интернете |
|*Data input* | сервис с бизнес-логикой: получение входящих данных | бывшее приложение |
|*Data processing* | сервис с бизнес-логикой: обработка входящих данных | бывшее приложение |
|*Data output* | сервис с бизнес-логикой: передача данных внешнему сервису | бывшее приложение |
|*Updater* | непосредственно применяет обновление | работает в одном контейнере с *Data processing*, чтобы иметь возможность обновлять его файлы |
|*Downloader*  | скачивает данные из сетей общего пользования. Только этот сервис может выходить во внешние сети | в примере скачивает данные с *File server*.  |
|*Manager* | оркестрирует весь процесс обновления | |
|*Verifier* | проверят корректность скачанного обновления | |
|*Storage* | осуществляет хранение скачанного обновления | |
|*Security monitor*| авторизует операцию, если она удовлетворяет заданным правилам или блокирует её в противном случае| |
|*Message bus* | шина сообщений и брокер - сервис передачи сообщений от источника получателям | kafka+zookeeper |

### Алгоритм работы решения

![Алгоритм работы решения](./misc/diag1.png?raw=true "Алгоритм работы решения")

### Описание cценариев (последовательности выполнения операций), при которых ЦБ нарушаются

#### Негативный сценарий 1. Система аутентификации не верифицирует пользователя

![Негативный сценарий 1](./misc/diag1n.png?raw=true "Система аутентификации не верифицирует пользователя")

Результат: недостижение цели безопасности №1 - доступ к информации неавторизованными пользователями.

#### Негативный сценарий 2. Повреждение или потеря данных в КУМО

![Негативный сценарий 2](./misc/diag2n.png?raw=true "Повреждение или потеря данных в КУМО")

Результат: недостижение цели безопасности №2 - потеря данных.

#### Негативный сценарий 3. Менеджер не проверяет обновление или игнорирует результаты проверки

![Негативный сценарий 3](./misc/diag3n.png?raw=true "Менеджер не проверяет обновление или игнорирует результаты проверки")

Результат: недостижение цели безопасности №3 - обновление не проверено.

## Запуск приложения и тестов

Для запуска и проверки тестов имеются вспомогательные скрипты, запускаемые с помощью 'Makefile'. При вызове комманды 'make' (без аргументов) выводится список доступных скриптов.