# ansible_asterisk_mariadb
Ansible playbook to setup and scale existing Asterisk and Mariadb deployment

== Версии ПО ==

Данный плейбук тестировался на следующих версиях ПО:

* Ansible 2.6.5
* Centos 7
* Asterisk 16
* Mariadb 10.3 (ODBC-connector 3.0.8)

== Логика работы плейбука: ==

1. Общая настройка дистрибутива (часовой пояс, настройка sshd)
2. Установка СУБД Mariadb и его настройка (создание пользователя, базы данных, в случае наличия нескольких mariadb
серверов - настройка galera cluster и запуск keepalived поднимающего выделенный IP на одной из db-нод)
3. [TODO:] Установка Ejabberd и его настройка (создание пользователя, настройка s2s между несколькими инстансами)
4. Установка asterisk, в случае наличия Mariadb сервера - настройка на его использование, (TODO: в случае наличия
Ejabberd сервера - настройка на его использование.) В случае существования нескольких asterisk инстансов - настройка
IAX2-транка между ними.

Результат выполнения: Установка Asterisk на одной и более нод, с IAX2 транком между ними, возможным H.323 транком до внешней PBX (настраивается через group_vars переменные), готовый базовый Dialplan'ом позволяющий набирать эндпоинты текущей и соседних нод и выходом во вне через объявленный H.323 транк.
При наличии DB-сервера (или Galera кластера с keepalived виртуальным адресом) производится подключение Asterisk нод к DB и настройка нескольких модулей Asterisk на работу с DB.

== Заполнение файла инвентаря: ==

* В файле инвентаря 'hosts' объявлено 3 секции: [asterisk_server], [db_server], [xmpp_server].

Минимальной рабочей конфигурацией является хотя-бы один ip-адрес в секции [asterisk_server] - в результате плейбук
установит Asterisk на этот хост и произведёт минимальную конфигурацию (см. также "Управляющие переменные")

Возможна установка всех трёх компонентов (Asterisk + Mariadb + Ejabberd) на один хост.

При указании более одного хоста в каждой секции, соответствующий компонент будет установлен на указанных хостах,
кроме того, будет произведено масштабирование (IAX2 транки между инстансами Asterisk, создание Galera между нодами Mariadb,
(TODO: установка доверительных s2s отношений между Ejabberd серверами).

== Управляющие переменные: ==

* galera_cluster_always_rebuild - в случае истинности пересоздаёт Galera cluster при каждом запуске плейбука,
имеет смысл включать эту переменную при желании произвести масштабирование количества mariadb узлов.

* iax_trunking - True - образовывать IAX2-транки между asterisk и прописывать в диалпланах возможность набора номеров
из других Asterisk инстансов, False - не создавать или удалить имеющиеся IAX2 транки

* h323_trunking - True - настроить на всех Asterisk инстансах транк с "внешней АТС" по h.323 протоколу, добавить в
диалплан возможность совершать внешние вызовы, False - удалить конфигурацию h.323 транка из конфигурации

* asterisk_dialplan_allow_forward_ext_to_peers - True - разрешить текущей Asterisk ноде маршрутизировать звонки от внешней PBX
на номера других Asterisk нод, False - удалить конфигурацию из Dialplan

== Тэги: ==

* iax-config - при игнорировании данного тега не будет производиться настройка транка между
инстасами asterisk, при выполнении только этого тега будет производиться масштабирование конфигурации
IAX транка между указанными в inventory файле узлами.

* galera-config - при игнорировании данного тэга не будет производиться настройка galera кластера между mariadb инстансами,
каждый сервер будет иметь независимую конфигурацию, при выполнении только этого тега (только если истинна управляющая
переменная 'galera_cluster_always_rebuild'!) будет производиться масштабирование
конфигурации galera-кластера - первая mariadb нода в списке [db_server] станет initial-нодой нового кластера, на других
нодах будет изменен список wsrep_cluster_address и перезапуск mariadb сервиса.

* h323-config, - при применении задач с этим тэгом - переконфигурировать H.323-транк в соответствии с настройками из group_vars

* iax-config - при применении задач с этим тэгом - переконфигурировать IAx2-транк в соответствии с настройками из group_vars

* pjsip-config,, - при применении задач с этим тэгом - переконфигурировать настройки эндпоинтов, с настройками из group_vars
