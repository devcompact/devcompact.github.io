.. title: Как передать отрицательное число в качестве параметра консольной программе erlang
.. slug: kak-peredat-otritsatelnoe-chislo-v-kachestve-parametra-konsolnoi-programme-erlang
.. date: 2019-10-22 21:58:48 UTC+03:00
.. tags:
.. category: erlang
.. link:
.. description:
.. type: text

Недавно писал консольную программу, принимающую в качестве аргументов два числа. Столкнулся с тем, что в Erlang не так очевидно и просто передать аргументом отрицательное число.
Представим, что у нас есть программа prog, которая вызывается так:
erl -noshell -s program 1 -2

В Erlang всё, что начинается с символа "-" рассматривается как флаг:
Any argument starting with character - (hyphen) is interpreted as a flag http://erlang.org/doc/man/erl.html#exports

.. TEASER_END

Есть несколько вариантов решения, из них наиболее приемлемые:

*1. Использовать флаг -extra.*

Всё, что следует за этим флагом, рассматривается как "простые" аргументы и может быть получено через init:get_plain_arguments/0.
В свою очередь init:get_plain_arguments() возвращает список. Далее список/строка может быть сконвертировано в число при помощи list_to_integer().

По сути вызов из примера выше необходимо преобразовать в
erl -noshell -s program -extra 1 -2

Обработка в функции может быть примерно такой:

.. code:: erlang

    args_to_ints() ->
        [FirstArg|SecondArg] = init:get_plain_arguments(),
        FirstNum = list_to_integer(FirstArg),
        SecondNum = list_to_integer(SecondArg),
        [FirstNum, SecondNum].

*2. Использовать флаг -eval*

-s флаг работает для простых сценариев, но если нужны какие-то более сложные вызовы, то можно попробовать -eval флаг.
-s конвертирует все аргументы в атомы, поэтому для чисел это может быть не очень удобно. Можно использовать -run флаг, чтобы передать строки (например, -run program -2, что вызовет program:start("-2")), но передавая строки их придётся парсить самостоятельно. Самый простой и в то же время гибкий способ - это использовать -eval:

erl -noshell -eval "program:start(1, -2)"

Тогда можно исполнять любое Erlang выражение и система самостоятельно распарсит значения.
