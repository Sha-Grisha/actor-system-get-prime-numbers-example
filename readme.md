# Веб-сервис вычисления N-го простого числа и получения текущей погоды

Веб-приложение позволяет получить погоду в некотором городе или найти N-ое простое число.

Погода получается с сервиса [http://apixu.com/](apixe.com).

N-ое простое число считается наиболее медленным способом: проверяются все числа от 2 до N, каждое проверяется на простоту перебором всех делителей за O(n).

Приложение может работать в многопоточном режиме, а также использовать для вычислений удалённые машины.

Приложение реализовано в модели акторов, используя среду выполнения [https://github.com/untu/comedy](Comedy).

## Почему модель акторов

Приложение на NodeJS по умолчанию работает в однопоточном режиме. Это означает, то если запросить простое 50 000-ое число, то сервер будет занят получением простого числа. Если в это время параллельно в сервера запросить 1-ое простое число, он не сможет ответить до тех пор, пока не ответит на предыдущий запрос.

Работа в режиме `forked` позволяет очень легко сделать сервер многопоточным, тогда каждое вычисление простого числа будет выполняться в отдельном потоке, и не будет блокировать сервер.

Этого же эффекта можно было добиться, используя, например, модуль cluster, или самостоятельно запуская вычисления в отдельных потоках, но с использованием системы акторов такая задача решается ещё проще.

При необходимости можно распараллелить вычисления не только между ядрами одной машины, но и между множеством машин. Для этого достаточно перейти в режим `remote`, указать адреса узлов сети акторов и запустить на удалённой машине узел среды выполнения акторов **comedy-node**.

При этом: 
1. Мы не запускаем какой-либо наш веб-сервис на удалённых машинах вручную, а просто запускаем узел сети акторов Comedy. В момент запуска узел даже не знает о существовании простых чисел. Позже актор "отправится" с нашего сервера на удалённый узел.
1. Для того, чтобы наша система начала работать многопоточно на нескольких машинах мы просто меняем пару строк в конфигурации.

Если решение нашей вычислительной задачи также можно было бы распараллелить, то мы могли бы также легко и её выполнять при помощи множества акторов.

## Развёртывание

1. Установить [http://nodejs.org/](NodeJS)
1. В директории проекта выполнить `npm install`

## Запуск

1. `npm start` - запускает сервер

## Использование

1. Сервер доступен по [http://localhost:4000](http://localhost:4000)
1. GET [http://localhost:4000/weather?city=Perm](http://localhost:4000/weather?city=Perm) - Получение погоды в Перми
1. GET [http://localhost:4000/prime?num=5](http://localhost:4000/prime?num=5) - Получение 5-го простого числа

## Конфигурирование

Файл **actors.config.json**

* `mode` - режим работы, может принимать значения:
    * `in-memory` - все акторы находятся в памяти в основном потоке приложения (однопоточный режим)
    * `forked` - каждый актор существует в отдельном потоке (многопоточный режим)
    * `remote` - каждый актор существует в отдельном потоке, и расположен на удалённых машинах (многопоточный режим, распределённый в сети)
* `clusterSize` - размер кластера актора, количество одновременно существующих акторов
* `host` - массив хостов с портами. Если порта нет, то по умолчанию 6161
* `balancer` - режим работы, может принимать значения:
    * `round-robin` - нагрузка будет распределяться между акторами по очереди (1-2-3-4-1-2-3-4...)
    * `random` - нагрузка будет распределяться между акторами в случайном порядке

## Запуск в режиме `remote`

Для запуска в режиме `remote` требуется добавить удалённые машины к среде, в которой существуют акторы. Для этого требуется на удалённой машине запустить узел среды акторов:

```
npx comedy-node PORT 0.0.0.0
```
Например, для запуска на порту 4001

```
npx comedy-node 4001 0.0.0.0
```

Данную утилиту можно использовать:
* Взяв из данного приложения (развернув приложение на удалённой машине, но не запуская сервер)
* Установив через `npm init` + `npm install comedy`
* Используя скомпилированный под нужную ОС бинарник (см. [nexe](https://github.com/nexe/nexe))

## Описание реализации

* `ComputingActor.js` - Класс ComputingActor, имеющий методы получения простого числа и текущей погоды. Обычный класс, не знающий о том, что он будет актором. В дальнейшем используется как реализация актора
* `actorSystem.js` - Инициализация системы акторов с конфигурацией в **actors.config.json**
* `server.js` - Запуск сервера на Express, обработка запросов и их выполнение с помощью акторов из actorSystem.

## P.S.

Данное приложение разработано для демонстрации примера разработки приложения, использующего модель акторов. Порт сервера и токен для получения погоды зашиты в приложении. Отсутствует контроль ошибок. Отсутствует контроль отключения сервера для автоматического уничтожения акторов. Так делать не нужно :)