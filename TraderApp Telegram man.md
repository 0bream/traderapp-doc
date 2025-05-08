
## Управление роботом черезе Telegram

### Стратегии

Точкой входа для работы бота и главной сущностью является стратегия. Стратегия - (S)trategy - это совокупность правил по которым осуществляется автоматическая торговля. Стратегия может находится в двух состояниях, `STARTED` и `PAUSED`.

#### Список стратегий

Для просмотра списка активных(в состоянии `STARTED`) стратегий служит команда `strategy list`:
- `s ls`

Для просмотра списка всех стратегий служит команда `strategy list full`:
- `s lsf`

#### Создание стратегий

Для создания новой стратегии служит команда `strategy generate` следующего формата:

`s g <exchange> <type> ...<params>`
`type = copy|candle` - тип стратегии
`exchange = binance` - торговая площадка

Пддерживаемы типы стратегий и их параметры:
- `copy` - стратегия копирования сделок
	- `name` - имя стратегии
	- `trader_id` - идентификатор трэйдера для копирования
- `candle` - стратегия работающая на закрытиях свечей
	- `name` - имя стратерии
	- `strategy_family` - `tms` | `ass` | `bbrsi`
	- `ticker` - `btc`, `eth`, ...
	- `interval` - `15m`, `1h`

возвращает `NN` - unique number of created strategy or `fail`


> [!example] 
>  
`s g binance copy test1 212`
return - `34`

  > [!example] 
>  
  `s g binance candle test2 tms btc 15m`
return - `35`

Новая стратегия создается в состоянии `PAUSED`


#### Смена состояния стратегии

Для остановки и запуска стратегий использу.тся команды `strategy pause` и `strategy unpause`  с параметром номера стратегии `NN`:
- `s u 34`
- `s p 34`


#### Парметры стратегии

Для просмотра параметров стратегии используется команда `strategy get ` с параметром `NN`
`s get <NN>`

> [!example] 
> `s get 34`

Для установки и изменения параметров стратегии используется команда `strategy set`:
 `s set <NN> <param1:value1> <param2:value2> <param3:value3> ...`

Команда возвращает спсок текущих параметров стратегии или `fail`

Для каждого типа и/или семейства стратегий есть свой набора параметров. Так же есть общие параметры для всех стратегий.

##### Параметры общие для всех стратегий

- `a` - amount in USDT
- `l` - leverage

> [!example] 
>  
  >`s set 34 a:1000 l:25`
   >`set 35 l:25`

###### Параметры для расчета цен закрытия позиций
> [!warning] 
>  Параметры поддерживаются в стратегиях, у которых явно указана поддержка данных параметров(см. ниже описание конкретных стратегий)

- `usePoint` - `true | false` - использовать пункты цены (`def = false`);
- `tpPoint` - размер *take profit* в пунктов(`def = 100`);
- `slPoint` - размер *stop loose* в пунктов(`def = 100`);
- `useATR` - `true | false` - использовать для расчетов индикатор **ATR**(`def = false`);
- `atrLen` - глубина индикаттора **ATR**(`def = 14`);
- `atrTpMult` - `take profit = atrTpMult * ATR` (`def = 1`);
- `atrSlMult` - `stop loose = atrSlMult * ATR` (`def = 1`).

> [!important] 
>  Параметр `useATR` имеет приоритет над параметром  `usePoint`

> [!warning] 
>  Здесь *пункты* это минимальный шаг цены для конкретного инструмента. Параметры `tpPoint` и `slPoint` нужно подбирать для каждого инструмента индивидуально, т.к. минимальный шаг может отличаться на несколько порядков.


##### Параметры `candle` стратегий

###### Семейство `tms`

- `tpx` - Take Profit Multiplier
-  `depthRSI`
- `depthFastMA`
- `depthSlowMA`
- `bandLength`
- `periodK`
- `smoothK`
- `smoothD`

По умолчанию стратегия `candle:tms` работает с параметрами:
```js
{
    depthRSI: 14,
    depthFastMA: 3,
    depthSlowMA: 9,
    bandLength: 34,
    stdDev: 1.6185,
    maType: 'EMA'
    priceType: 'close'
    periodK: 8,
    smoothK: 1,
    smoothD: 3,
    maTypeStochastic: 'SMA'
    priceTypeStochastic: 'close'
}
```


###### Семейство `ass`
- `depthRSI`
- `depthFastMA`
-  `depthSlowMA`
-  `bandLength`
- `actionMoving`
- `trendMoving`
- `slowMoving`
- `useHA`
- `useActionMA`
- `rsiBuyLevel`
-  `rsiSellLevel`
Закрытие позиции определяется либо пунктами цены либо индикатором **ATR**.
По умолчанию стратегия `candle:ass` работает с параметрами:
```javascript
  {
    depthRSI: 13,
    depthFastMA: 2,
    depthSlowMA: 7,
    bandLength: 34,
    stdDev: 1.6185,
    maType: 'EMA' as MAType,
    priceType: 'close' as PriceType,
    actionMoving: 10,
    trendMoving: 200,
    slowMoving: 800,

    useActionMA: false,
    useHA: false,
    rsiBuyLevel: 68,
    rsiSellLevel: 32,
  }
```


###### Семейство `bbrsi`
Стратегия работате на связке индикаторов **Bollinger Band + RSI**
> [!warning] 
> Обязательно указывать параметр закрытия стратегии `usePoint` или `useATR` в `true`

- `enterType` - тип выбора точки входа(см. теорию)`(def = 1)`

Рекомендуемые параметры для `enterType = 2`:
BB 30 2
RSI 13


### Позиции

(P)osition - сущность пренадлежазая стратегии. Идентификатором позиции является её уникальный идентификатор `id`. Позиция может находится в трёх состояниях: `open`, `closed`, `fail`.

Для управления позициями используются команды:
- `p` | `p <count>` - отображает список последних позиций, необязательный параметр `count` задаёт количество отображаемых позиций;
- `p o` | `p o <count>` - отображает список открытых позиций, необязательный параметр `count` задаёт количество отображаемых позиций;
- `p id <id>` - отображает подробную информацию позиции `id`;
- `p s <NN> [<count>]` - позиции стратешии `NN`. Необязательный параметр `count` - количество отображаемых записей;
- `p sd <NN> [<count>]` - то же что и предыдущая команда, но с выводом подробной информации;
- `p c <id>` - принудительное закрытие позиции `id`;
- `p f <id>` - перевод позиции `id` в статус `fail`.


### Маркет специфичаские команды

#### Команды Binance

`tfb b` - отображает баланс фьючерсного счета
`tfb o` - отображает открытые позиции по фьючерсам


## Теория

### Стратегии

#### TMS

Оригинальная стратегия 
- https://www.forexfactory.com/thread/291622-trading-made-simple

Ссылки на различные варианты стратегии
- https://academyfx.ru/article/blogi/4040-prostaya-torgovlya-po-strategii-trading-made-simple-r
- https://rutube.ru/video/6780f167d2d3ad4943383c431416cca6/



#### Another Simple System (ASS)
- https://tlap.com/another-simple-system/
- https://investmana.ru/strategy/another-simple-system

Условия открытия лонга:
```javascript
      // price over trend MA(200)
      curr.src > curr.trendMA &&
      // TDI RSI below 68
      curr.tdi.rsi < rsiBuyLevel &&
      // TDI slow MA crossed up RSI
      prev.tdi.rsi < prev.tdi.slowMA &&
      curr.tdi.rsi > curr.tdi.slowMA &&
      // HA candle has white color
      (useHA ? curr.ha.growth : true) &&
      // price crossed up trend MA(10)
      (useActionMA
        ? prev.src < prev.actionMA && curr.src > curr.actionMA
        : true)

```

- параметр `useHA` включает подтверждение входа по **HeikenAshi** свечам;
- параметр `useActionMA` включает подтверждение входа по пробою ценой быстрого мувинга.


#### BBRSI
- https://www.youtube.com/watch?v=5VTaJqzbdvM&t=6s
- https://www.youtube.com/watch?v=0DNbw0WG68w

Вход в лонг при `enterType = 1`:
```javascript
      prev.candle.low < prev.bb.lineAvg &&
      prev.rsi.value < rsiBuyLevel &&
      curr.candle.high > curr.candle.low &&
      curr.candle.close > prev.candle.close &&
      curr.bb.bandWidth > bandWidthThreshold
```

Вход в лонг при `enterType = 2`:
```javascript
      curr.candle.close < curr.bb.lineDown &&
      curr.rsi.value < rsiBuyLevel &&
```

