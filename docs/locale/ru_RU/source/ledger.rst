Реестр
======

Реестр это упорядоченная, защищенная от вмешательства запись всех изменений состояния. Изменения
состояния появляются в результате вызова чейнкода (транзакция), поданных к исполнению участниками.
Набор пар "ключ-значение" записывается в реестр в формате "создать", "обновить" или "удалить", как
результат исполнения транзакции.

Реестр образован блокчейном ("цепь") для хранения неизменяемой структурированной записи,
состоящей из блоков и базой данных состояний для поддержки текущего состояния, по одному реестру
на канал. Каждый одноранговый узел поддерживает одну копию реестра для каждого канала, членом которого
он (узел) является.

Блокчейн (цепь)
-----

Блокчейн это журнал транзакций, структурированный как хэш-связанные блоки, в котором каждый блок
содержит последовательность N-транзакций. Заголовок блока включает хэш транзакций блока, а также
хэш предыдущего заголовка блока. Таким образом, все транзакции в реестре упорядочены и криптографически
связаны друг с другом. Иными словами, исказить данные реестра невозможно без разрушения хэш-связей.
Хэш самого позднего по времени создания блока представляет каждую транзакцию, совершенную прежде, чем
обеспечивается нахождение всех одноранговых узлов в согласованном состоянии подтвержденного доверия.


Блокчейн хранится в файловой системе одноранговых узлов (в локальной или связанной памяти),
таким образом эффективно поддерживая режим "разрешена только запись" данных блокчейна.

База данных состояний
--------------

Данные текущего состояния представляют собой последние значения всех ключей, представленных в журнал
транзакций блокчейна. Поскольку текущее состояние представляет собой все актуальные по времени значения
ключей, известные каналу, его иногда называют Глобальное Состояние.

Вызовы чейнкода исполняют транзакции в данных текущего состояния. Чтобы придать взаимодействиям
с чейнкодом предельно эффективными, в базе данных состояний хранятся последние значения всех ключей.
База данных состояний является просто проиндексированным снимком транзакционного журнала блокчейна. Она
может быть воссоздана из блокчейна в любой момент. Прежде, чем будут приняты транзакции, база данных
состояний автоматически восстановится (или заново сгенерируется, если нужно) после перезапуска однорангового узла.

База данных состояний опционально может быть LevelDB или CouchDB. По умолчанию базой данных является LevelDB,
встроенная в процессы одноранговых узлов, и она хранит данные чейнкода в виде пар "ключ-значение". Как вариант,
приемлема CouchDB, которая обеспечивает дополнительную поддержку запросов (поддерживая расширенные
запросы JSON-контента), необходимую, если ваш чейнкод смоделирован как JSON. Дополнительную информацию о
CouchDB можно увидеть в :doc:`couchdb_as_state_database`.

Транзакционный поток
На высоком уровне абстракции транзакционный поток состоит из запроса на транзакцию, посланного клиентским приложением
конкретным одноранговым узлам для одобрения. Одобряющие узлы проверяют подпись клиента и исполняют функцию чейнкода
для симуляции транзакции. На выходе - результаты чейнкода, набор версий "ключ-значение", прочитанных в чейнкода
(набор чтения) и набор "ключ-значение", записанный в чейнкод (набор записи). Ответ на запрос отсылается обратно
клиенту вместе с подписью одобрения.

Клиент составляет из одобрений так называемую "транзакционную полезную нагрузку" и сообщает их в службу
упорядочения. Служба упорядочения доставляет очередь транзакций в виде блоков всем одноранговым узлам в канале.

Перед записью транзакций одноранговые узлы должны их валидировать. Сначала они проверяют правила одобрения,
чтобы обеспечить корректное распределение конкретных узлов, подписавших результаты, и они авторизуют подписи
в сравнении с транзакционной полезной нагрузкой.

Затем узлы проверяют номер версии в сравнении с "набором записи" транзакции, чтобы обеспечить сохранность
данных и защиту от "задвоения". В Hyperledger Fabric встроен контроль параллельного выполнения, посредством
которого для ускорения потока транзакции запускаются параллельно, а по итогам записи (всеми одноранговыми узлами)
каждая транзакция верифицируется для того, чтобы убедиться в том, что ни одна другая транзакция не изменила
прочитанные ею данные. Иными словами, он гарантирует, что данные, которые были прочитаны во время исполнения
чейнкода, не изменились со времени исполнения (одобрения), и тем самым гарантирует, что результаты исполнения
до сих пор действительны и могут быть записаны в базу данных состояния реестра. Если прочитанные данные успели
измениться как результат действия другой транзакции, тогда транзакция в блоке помечается как недействительная
и не записывается в базу данных реестра. Клиентскому приложению выдается предупреждение, и оно может обработать
ошибку или запустить новую попытку, по необходимости.

См. разделы :doc:`txflow`, :doc:`readwrite`, и :doc:`couchdb_as_state_database` для более глубокого
изучения структуры транзакций, контроля параллелизма и баз данных состояний.

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
