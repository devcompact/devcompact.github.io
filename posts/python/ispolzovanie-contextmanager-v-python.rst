.. title: Пишем контекстные менеджеры в python
.. slug: ispolzovanie-contextmanager-v-python
.. date: 2019-10-22 22:08:28 UTC+03:00
.. tags:
.. category: python
.. link:
.. description:
.. type: text

Контекстный менеджер в python - это объект, определяющий, что должно быть сделано "До" и "После"
тела with выражения. Чаще всего контекстные менеджеры используют для управления ресурсами,
получение и освобождение. Тема хорошо объясняется на следующем, довольно распространённом, примере с
функцией open.

Допустим, нам необходимо открыть, прочитать и закрыть файл. Причём закрытие файла должно быть
гарантированным:

.. TEASER_END

.. code:: python

    file_descriptor = open("test_file.txt")
    file_descriptor.read("just example text")
    file_descriptor.close()

В случае возникновения какого-либо исключения в промежутке между открытием и закрытием файла
.close() не отработает - это утечка файлового дескриптора. Этого можно избежать, применив
finally:

.. code:: python

    file_descriptor = open("test_file.txt")
    try:
        file_descriptor.read("just example text")
    finally:
        file_descriptor.close()

Теперь то же самое, но в виде контекстного менеджера:

.. code:: python

    with open("test_file.txt") as file_descriptor:
        content = file_descriptor.read()

Лаконичнее и чище.

|

**Как написать свой:**

Если замечаете, что в вашем коде встречается подобный повторяющийся try…finally, то можно
написать свой контекстный менеджер. Например, вам нужно исполнять sql запрос предварительно
приконнектившись к БД, а по завершении обязательно закрыть коннект(даже если по пути вылезло
исключение). Далее мы такой менеджер и напишем несколькими способами. Для примера возьмём sqlite3.

1) **способ с __exit__ и __enter__:**

Данный способ заключается в написании класса, в котором необходимо определить специальные, указанные
выше, методы:

.. code:: python

    import sqlite3


    class DBManager:

        def __init__(self):
            self.conn = None

        def __enter__(self):
            self.connect()
            return self.conn

        def __exit__(self, *exc_info):
            self.disconnect()

        def connect(self):
            self.conn = sqlite3.connect("test.db")

        def disconnect(self):
           self.conn.close()

И теперь можно использовать так:

.. code:: python

    with DBManager() as db:
        print("2 x 2 = ", db.execute("SELECT 2 * 2;").fetchone()[0])

2) **Способ: декоратор contextlib.contextmanager:**

contextlib - это встроенная библиотека, которая помогает писать понятные контекстные менеджеры.
Ниже реализация аналога примера из пункта 1:

.. code:: python

    import contextlib
    import sqlite3


    @contextlib.contextmanager
    def db_manager():
       conn = sqlite3.connect("test.db")
        try:
            yield conn
        finally:
            conn.close()

Использование:

.. code:: python

    with db_manager() as db:
        print("2 x 2 = ", db.execute("SELECT 2 * 2;").fetchone()[0])

Обратите внимание на использование yield. По сути всё, что в блоке try - это то же самое, что и
__enter__ метод, а в finally это тот самый клинап ресурсов - __exit__.

3) **Способ contextlib.closing:**

В этом способе вам вообще ничего не придётся писать, при условии, что клинап метод у вас называются
close().

Так, в sqlite3 есть метод .close(), который отвечает за закрытие коннекта к БД, а значит мы
можем спокойно использовать данный способ:

.. code:: python

    with contextlib.closing(sqlite3.connect("test.db")) as conn:
        conn.execute("SELECT 2 * 2;").fetchone()[0]

Тема контекстных менеджеров достаточно обширная и рекомендуется, как минимум, самостоятельно
поизучать какие ещё есть встроенные контекстные менеджеры(включая те, что есть в библиотеке
contextlib).
