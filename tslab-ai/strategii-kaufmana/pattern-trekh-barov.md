---
order: 4
title: Паттерн трех баров
---

## TSM 3-Bar Inside Pattern (S)

## 1\. Название стратегии

-  Русское название: Паттерн трех баров

-  Оригинальное название: `TSM 3-Bar Inside Pattern`

## 2\. Источник

-  Автор идеи: Johnan Prathap

-  Публикация: `Three-Bar Inside Bar Pattern`, *Technical Analysis of Stocks & Commodities*, March 2011

-  Исходный файл проекта: `TSM 3-Bar Inside Pattern (S).txt`

## 3\. Краткая идея

Стратегия ищет трехбарную конфигурацию, в которой средний бар является inside bar, а направление первого и третьего сигнальных баров совпадает. После подтверждения стратегия открывает позицию на следующем баре и сразу выставляет процентные тейк-профит и стоп-лосс от цены входа.

## 4\. Данные текущей серверной реализации

-  папка серверного скрипта: `AI/KaufmanStrategies/TSM 3-Bar Inside Pattern (S)`

-  datasource: `BingXPerpetual_Staging`

-  инструмент: `BTC-USDT`

-  таймфрейм: `1h`

## 5\. Реализованная логика

### Вход в long

Long-сигнал активируется, когда:

-  `Close > Close[-1]`

-  `High[-1] < High[-2]`

-  `Low[-1] > Low[-2]`

-  `Close[-2] > Close[-3]`

В текущей реализации вход выполнен блоком `OpenPositionByMarketItem` по булевому условию `LongSignal`.

### Вход в short

Short-сигнал активируется, когда:

-  `Close < Close[-1]`

-  `High[-1] < High[-2]`

-  `Low[-1] > Low[-2]`

-  `Close[-2] < Close[-3]`

В текущей реализации вход выполнен блоком `OpenPositionByMarketItem` по булевому условию `ShortSignal`.

### Выход из long

-  тейк: `EntryPrice * (1 + TpPct / 100)`

-  стоп: `EntryPrice * (1 - SlPct / 100)`

### Выход из short

-  тейк: `EntryPrice * (1 - TpPct / 100)`

-  стоп: `EntryPrice * (1 + SlPct / 100)`

## 6\. Использованные блоки

-  `TradableSecuritySourceItem`

-  `Close`

-  `High`

-  `Low`

-  `RelativeCommission`

-  `ConstGen`

-  `BoolCustomHandlerItem`

-  `DoubleCustomHandlerItem`

-  `OpenPositionByMarketItem`

-  `EntryPrice`

-  `ClosePositionByProfitItem`

-  `ClosePositionByStopItem`

## 7\. Параметры

-  `TpPct = 0.75`

-  `SlPct = 0.75`

-  `Qty1 = 1` Для числовых параметров сервером добавлены optimization mappings.

## 8\. Инженерные решения переноса

-  Вход `next bar on open` перенесен через `OpenPositionByMarketItem` по булевому сигналу.

-  Процентные уровни тейка и стопа считаются от `EntryPrice` через отдельные формульные блоки.

-  Размер позиции в baseline-версии фиксирован и равен `1`.

-  На график выведены линии `EntryPrice`, `TakePrice` и `StopPrice` для long и short.

## 9\. Статус live-проверки

Проверка на серверном артефакте:

-  `authoring-quality`: без структурных блокеров для текущего варианта переноса

-  `validate`: успешно

-  `build`: успешно

-  `load`: успешно

-  `run`: выполнен, данные получены

Результаты последнего прогона:

-  `barsCount = 18434`

-  `allTrades = 1189`

-  `netProfit = 2564.00 USDT`

-  `netProfitPct = 4.07%`

-  `profitFactor = 1.0065`

-  `winTradesPct = 49.62%`

-  `maxDrawdownPct = -40.48%`

Остаточные замечания:

-  generic authoring-quality советует `MissingControlPane` и `OnlyMarketEntries`;

-  главным смысловым допущением остается хостовая трактовка `OpenPositionByMarketItem` как исполнения сигнала `next bar on open`.

## 10\. Итог

Стратегия перенесена в TSLab как серверный граф и исполняется на `BingXPerpetual_Staging / BTC-USDT / 1h`. В текущем состоянии комиссия включена, как вы попросили. Для буквального соответствия ключевой вопрос теперь не в формулах или выходах, а в точной интерпретации хостом TSLab рыночного входа `OpenPositionByMarketItem` относительно конструкции `next bar on open`.

## 7\.5 Комиссия

В текущем варианте графа комиссия оставлена явным блоком `RelativeCommission`.

[TSM 3-Bar Inside Pattern (S).tscript](<TSM 3-Bar Inside Pattern (S).tscript>)