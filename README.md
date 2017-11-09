# prebid_docs
## Как работает prebid.js

1. Мы добавляем код prebid.js на страницу паблишера. Вместе со всеми конфигами для получения доступа. 
Пример конфига:
````
 var PREBID_TIMEOUT = 700;
        var adUnits = [{
            code: 'div-gpt-ad-1460505748561-1',
            sizes: [[300, 250], [300,600]],
            bids: [{
                bidder: "criteo",
                params: {
                    zoneId: "1091692"
                }}]}
        ];
        var pbjs = pbjs || {};
        pbjs.que = pbjs.que || []; 
        
````

2. При загрузке страницы prebid.js асинхронно отправит запрос всем участникам аукциона о том,  сколько они хотят заплатить за показ. 
3. Чтобы страница не зависала, если участники аукциона очень долго отвечают на запрос, prebid.js позволяет поставить таймер  ожидания ответа для Эд сервера. 
Если время ожидания превышено  - аукцион пропускается и показ отправляется на сервер. 
4. Когда ставки получены, prebid.js добавляет цену и идентификатор кратива к запросу на ваш эд сервер как набор параметров строки. 
5. Внутри эд сервера, позиции( line items ) настроены в соответсвии с ценами ставок, что позволяет участникам аукциона на основании цены бороться за другие позиции или интегрированые показы. 
6. В каждой Prebid позиции прописан javascript чтобы отображать креатив. Пример: 
````
var w = window;
for (i = 0; i < 10; i++) {
  w = w.parent;
  if (w.pbjs) {
    try {
      w.pbjs.renderAd(document, '%%PATTERN:hb_adid%%');
      break;
    } catch (e) {
      continue;
    }
  }
}
````
Когда позиция програматика будет выбрана эд сервером, эти строчки кода обьясняют prebid.js которого участника аукциона обслужить. 



## Путь Ad Ops
1. ##Определение дробности ставок

C этой технологией вам нужно прописать на эд сервере какая цена от участника аукциона вам интерестна. Все делается через позиции ключ-значение(key-values)

Пример:

* Prebid.js отправляет запрос на ставку всем участникам аукциона, потом передает эд серверу эти данные в строке запроса. Вам нужно сопоставить ставку с позицией на сервере. 
* Если у вас одна позиция для каждой ставки с  дробностью в один цент, то для того, чтобы закрыть ставки от 0$ до 10$, вам понадобиться сделать 1000 позиций.
* Создавать 1000 позиций довольно напряжно. Потому паблишеры используют `price buckets` для группировки ценового диапазона.  Для примера, можно использовать ценовую группу с шагом 10 центов. Таким образом ставки в 1.06$ и 1.02$ будут округлены к 1$


Для начала, начните использовать дробность в 1 доллар или 10 центов. 

     
2. ## Один набор позиций для всех участников аукциона VS для каждого участника аукциона

а. Один набор позиций для все участников аукциона. 
Этот метод рекомендуется для настройки ваших позиций. 

* Этот  метод быстрее и проще, потому что создается один набор позиций. 
* Им легче управлять, так как добавление участников аукциона не требует изменения в наборе позиций. 
* Этот метод более отказоустойчивый, так как нужно ипользовать только три ключевых слова. 

 | Значение по умолчанию  | Необходимость    | Описание                                                                                     | Пример|
 |-------------|----------|-------------------------------------------------------------------------------------------------|---------|
 | **hb_pb**       | Необходим |  Ценовой диапазон. Используется для кампании                                          | 2.10    |
 | **hb_adid**     | Необходим | ID Рекламы. Используется эд сервером для рендера рекламы.                                          | 234234  |
 | **hb_bidder**   | Необходим | Код участника аукциона. Используется для логирования и составления отчетов о том кто из участников аукциона имеет лучший CPM | rubicon |
 
 Для инструкции по установке одного набора позиций для всех участников аукциона нажмите на 
 [ссылку](http://prebid.org/adops/step-by-step.html)
 
 
 
 б. Один набор позиций для КАЖДОГО участника аукциона 
 
 Воспользуйтесь это возможностью если:
 * Вам нужны полагаться на отчеты по позициям, а не по строкам запросов, чтобы получить выигравшую ставку
   * При использовании одного набора позиций для всех участников аукциона, prebid.js отправляет только выигравшую ставку на эд сервер. Но если вам нужна информация о всех ставках используйте вариант позиций для каждого биддера  
 * Необходима полная информация для партнеров которые используют Header bidding
    * При использовании одного набора позиций для всех участников аукциона, prebid.js отправляет информацию об участнике аукциона( о том, который участник аукиона имеет наивысшую цену) через ключевое слово `bidder=bidder_name`. Для отправки отчета о выигранной ставке участникам аукциона вы полагаетесь на отчеты ключевых слов с вашего эд сервера. DFP поддерживает такую функцию, но некоторые сервера - нет. DFP не поддерживает отправку отчетов из более чем двух ключевых слов. Поэтому, если у вас уже существующие отчеты основанные на ключевых словах, и вы хотите добавить выиграшную ставку по размеру участника аукциона используйте набор позиций для каждого участника аукциона 
    
    
    
 | Значение по умолчанию | Необходимость | Описание                                                                                                                                                               | Пример                       |
 |-----------------------|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
 | hb_pb_BIDDERCODE      | Необходимо    |  Ценовой диапазон. Используется позицией для таргетинга. Чувствительно регистру и обрезается до 20 символов. BIDDERCODE задокументирован в параметрах биддера(ссылка)  | hb_pb_rubicon = 2.10         |
 | hb_adid_BIDDERCODE    | Необходимо    | ID рекламы используется эд сервером для рендера креатива. Чувствительно регистру и обрезается до 20 символов.                                                          | hb_adid_indexExchang= 234234 |
 | hb_size_BIDDERCODE    | Опционально   | Нет необходимости для Эдопсов                                                                                                                                          | hb_size_appnexus = 300x250   |                   
        
        
