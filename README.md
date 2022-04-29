# tiapiv2

Кукбук по Tinkoff Invest APIv2 на python


Небольшое начало кукбука по TI APIv2 на python - не обессудьте, много не успел.


Тщемта, давно пора на официальном гитхабе https://github.com/Tinkoff/invest-python/tree/main/examples добавить примеров с нормальными описаниями, но пока есть, что есть.

 А есть, прежде всего видео ( ненавижу видеуроки, но пришлось) от @Azzrael и неоф тг разработчиков : https://t.me/c/1436923108/12187


Там и потырено - донаты шлите @Azzrael 


Получаем ключ API - не маленькие, сумеете


дальше уже у себя в консоли

pip install tinkoff-investments

Или тупо в pycharm добавляем модуль tinkoff-investments


Поехали. 

Прописываем в начале программы 

export INVEST_TOKEN='YOUR_TOKEN' 

Или даже прячем в отдельный файл для красоты


В начале файла 

import creds ( или как хотите назвать файл с API ключами и id )


В файле creds


account_id_1 ='2168492213'

token_1: str ='YOUR_TOKEN' 


Что за account_id_1 ?

Счас получим


Берем 

https://github.com/Tinkoff/invest-python/blob/main/examples/main.py

скармливаем наш токен, получаем в консольку и id c расшифровкой, и лимиты.


Если что , выдает аккаунты этот код 


accounts = client.users.get_accounts()

       print("\nСписок текущих аккаунтов\n")

       for account in accounts.accounts:

           print("\t", account.id, account.name, account.access_level.name)



Итого , id получили прописали..


Если вы настоящий <s>сварщик</s> скальпер, дальше вам надо работать в стриме , меня не читайте, но я начинающий, так что реквест - респонса мне, возможно и вам пока хватит.

https://music.yandex.ru/album/6684321/track/48920053



Собсно с апи можно работать только внутри этой странной конструкции

 with Client(TOKEN) as client:


Как вы помните, вложенность в питоне делается табом (мрак для человека больше всего кода написавшего в Дельфи, ну придется терпеть). Хотя, возможно я просто плохо знаю питон.


Теперь нам нужно знать, с какой бумажкой будем работать. Тут разработчики Тинькоф извернулись, не взяли стандартные тикеры, а придумали некие id бумаг figi - ибо мало ли что.

figi можно посмотреть в чужом гитхабе, можно купить одну бумажку и подсмотреть подобным кодом


  response = client.operations.get_positions(account_id=creds.account_id_1)

  print(response.securities)

  

 Ну и ответ например.

 

 [PositionsSecurities(figi='BBG000QQPXZ5', blocked=0, balance=1001)]


Есть варианты подсмотреть и поискать не покупая цитирую доком "Методы поиска инструментов по идентификатору (BondBy, CurrencyBy, EtfBy, FutureBy, ShareBy) позволяют получить информацию об инструменте зная его Figi или связку ticker + class_code." но мне пока не горело.



Итак ид инструмента получили, присвоили ручками констанут FIGI='BBG000QQPXZ5'


Дописали , теперь получим баланс инструмента в переменную balance


       response = client.operations.get_positions(account_id=creds.account_id_1)

       if response.securities[0].figi == FIGI:

           balance = response.securities[0].balance

           print('Остаток', balance)


Такс, получим стакан по инструменту, заодно и цены на границе стакана 

      try:

           book = client.market_data.get_order_book(figi=FIGI, depth=50)

           print(book)

           fast_price_sell, fast_price_buy = book.asks[0], book.bids[0] # центр стакана, мин спред

           print(fast_price_sell, fast_price_buy)

       except:

           print('стакан совсем пуст')

           

Стакан вывалится в виде простыни 

Нам интересны обычно края например предложения bids=[Order(price=Quotation(units=4, nano=248000000), quantity=4840) и спроса asks=[Order(price=Quotation(units=4, nano=250000000) quantity=3278)

Цены тут в неких юнитах и нано, перевод в обычные цифры вот такой подпрограммой.


def cast_money(v):

   return v.units + v.nano / 1e9

   

А можно тупо через nano сравнивать - но можно как нить капитально ошибиться

Как видим, спред 0.2 копейки, где то 0.04% - на граничном боте не разгуляться ... Хотя, если побегать вверх вниз по очереди, ловить проколы..


Ну что же , можно теперь покупать


Лимитная заявка на покупку


               r = client.orders.post_order(

                   order_id=str(datetime.utcnow().timestamp()),

                   figi=FIGI,

                   quantity=1000,

                   price=fast_price_buy.price,

                   account_id=creds.account_id_1,

                   direction=OrderDirection.ORDER_DIRECTION_BUY,

                   order_type=OrderType.ORDER_TYPE_LIMIT

               )

               print(r)

           except:

               print('чето не покупается')

               

               

И на продажу


           try:

               # заявка на продажу

               r = client.orders.post_order(

                   order_id=str(datetime.utcnow().timestamp()),

                   figi=FIGI,

                   quantity=1000,

                   price=fast_price_sell.price,

                   account_id=creds.account_id_1,

                   direction=OrderDirection.ORDER_DIRECTION_SELL,

                   order_type=OrderType.ORDER_TYPE_LIMIT

               )


               print(r)

           except:

               print('нечего продать еще')

               

               

На этом пока все, мои маленькие друзья ботописатели.

