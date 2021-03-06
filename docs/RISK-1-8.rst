﻿##################################################
RISK-1-8. Розмір забезпечення тендерної пропозиції/ пропозиції визначено з порушенням встановлених Законом граничних значень під час закупівлі товарів або послуг.
##################################################

***************
Суть індикатора
***************

Даний індикатор виявляє ситуації, коли замовником встановлено надто великий розмір забезпечення тендерної пропозиції при закупівлі товарів та послуг (розмір тендерного забезпечення перевищує 3% від очікуваної вартості).

************************************
Законодавче обґрунтування індикатора
************************************

Перевищення розміру тендерного забезпечення є порушенням частин 1 і 2 статті 24 Закону України "Про публічні закупівлі".

********************************
Підстава для розробки індикатора
********************************

Цей індикатор було розроблено, оскільки система електронних закупівель не передбачає перевірки порушення розмірів тендерного забезпечення.

*********************************
Методологія розрахунку індикатора
*********************************

Етап існування процедури
========================
Індикатор розраховується, коли процедура знаходиться на етапі *тендерингу*.

Рівень розрахунку
=================
Індикатор розраховується на рівні *лота*.

Джерела даних для розрахунку
============================

Для розрахунку індикатора вікористовуються наступні джерела даних:

- API модуля тендеринга електронної системи закупівель

- `API курсу валют Національного Банку України <https://bank.gov.ua/control/uk/publish/article?art_id=38441973#exchange>`_


Типи процедур
=============

Індикатор розраховується для наступних типів процедур:

- ``aboveThresholdUA`` - *відкриті торги*
- ``aboveThresholdEU`` - *відкриті торги з публікацією англійською мовою*

Типи замовників
===============

Індикатор розраховується для замовників які в системі визначені як:
 + ``authority`` - Орган державної влади, місцевого самоврядування або правоохоронний орган
 + ``central`` - Юридична особа, що здійснює закупівлі в інтересах замовників (ЦЗО)
 + ``general`` - Юридична особа, яка забезпечує потреби держави або територіальної громади
 + ``social`` -	Орган соціального страхування
 + ``special`` - Юридична особа, яка здійснює діяльність в одній або декількох окремих сферах господарювання

Категорії товарів
=================

Індикатор розраховується для процедур закупівлі *товарів* або *послуг* відподівно до значення поля ``data.mainProcurementCategory = 'goods'`` або ``data.mainProcurementCategory = 'services'``.

Стадії процедур
===============

Подія, що вмикає розрахунок індикатора
--------------------------------------

Подія, що вмикає розрахунок індикатора - публікація замовником у електронній системі оголошення про проведення закупівлі.

Подія, що вимикає розрахунок індикатора
---------------------------------------

Розрахунок індикатора вимикається тоді, коли процедура переходить у статус ``active.auction``.

Статуси процедур
----------------

Виходячи з подій, що вмикають та вимикають розрахунок індикатора, маємо наступні умови розрахунку:

- Індикатор розраховується для наступних статусів процедур:

  - ``active.tendering``
  - ``active.enquiries``

Частота розрахунку
==================

Індикатор розраховується при будь-якій зміні json-документа, що відповідає процедурі, якщо присутні всі умови для його розрахунку.

Окрім цього індикатор перераховується раз на добу незалежно від змін у json-документі, що відповідає процедурі, якщо присутні всі умови для його розрахунку.

Поля для розрахунку
===================

Для розрахунку індикатора використовуються наступні поля з API модуля тендеринга:

- ``data.guarantee``
- ``data.guarantee.amount``
- ``data.guarantee.currency``
- ``data.value.amount``
- ``data.value.currency``
- ``data.lots.guarantee``
- ``data.lots.guarantee.amount``
- ``data.lots.guarantee.currency``
- ``data.lots.value.amount``
- ``data.lots.value.currency``
- ``data.lots.status``
- ``data.enquiryPeriod.startDate``

Для розрахунку індикатора використовуються наступні поля з API курсу валют Національного Банку України:

- ``cc``
- ``rate``
- ``exchangedate``

Формула розрахунку
==================

Індикатор розраховується наступним чином:

Якщо у json-документі, що відповідає процедурі відсутні блоки ``data.guarantee`` або ``data.lots.guarantee``, індикатор приймає значення ``-2``. Розрахунок завершується.

Якщо у json-документі, що відповідає процедурі присутні блоки ``data.guarantee`` або ``data.lots.guarantee``, то йдемо на наступний крок.

У випадку, якщо процедура багатолотова:

1. Для кожного лота, де ``data.lots.status = 'active'`` між собою порівнюються ``data.lots.value.currency`` та ``data.lots.guarantee.currency``. Якщо вони не співпадають, то значення ``data.lots.value.amount`` та ``data.lots.guarantee.amount`` мають бути приведені до спільної валюти за допомогою API курсу валют на дату ``data.enquiryPeriod.startDate``. Якщо дане поле відсутнє у процедурі, то для розрахунку слід використати ``data.tender.date``.

2. Для початкових (або приведених до спільної валюти) значень ``data.lots.guarantee.amount`` ``data.lots.value.amount`` рахується, який відсоток від ``data.lots.value.amount`` становить ``data.lots.guarantee.amount``.

3. Якщо цей відсоток перевищує 3.00001%, то індикатор приймає значення ``1``.

У випадку, якщо процедура однолотова, то вищенаведені дії проводяться аналогічно для ``data.guarantee.amount``, ``data.guarantee.currency``, ``data.value.amount``, ``data.value.currency``.

Фактори, що впливають на неточність розрахунку
==============================================

1. Індикатор може бути порахований неточно у випадках, коли замовники в окремих сферах господарювання і організації, що не є замовниками, помилково визначають себе в системі як загальні замовники.

2. Індикатор може бути порахований неточно у випадках, коли замовником неправильно визначено тип процедури.

3. Індикатор може бути порахований неточно у випадках, коли замовником помилково визначено валюту тендерного забезпечення.
