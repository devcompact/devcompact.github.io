.. title: Кириллица в erlang
.. slug: otobrazhenie-kirillitsy-v-erlang
.. date: 2019-10-20 18:05:08 UTC+03:00
.. author: Igor Korolev
.. tags:
.. category: erlang
.. link:
.. description:
.. type: text


На первый порах изучения Erlang может быть не очевидно, каким образом выводить кириллицу. В интернете в основном находил информацию о том, как это происходит в интерактивной оболочке `erl`. По крайней мере, мне быстро найти ответ не удалось.

.. TEASER_END

В моём случае необходимо было запустить консольную утилиту примерно так:

``erl -noshell -s some_module func arg1 arg2``

Вне shell эрланга достаточно написать простую функцию:

.. code:: erlang

    cyrillic(String) ->
        io:format(unicode:characters_to_binary(String)).
