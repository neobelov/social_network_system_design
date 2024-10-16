# Social Network - System Design
Пример дизайна системы для курса по [Системному дизайну](https://balun.courses/courses/system_design)

### Функциональные требования:
- публикация постов из путешествий с фотографиями, небольшим описанием и привязкой к конкретному месту путешествия;
- оценка и комментарии постов других путешественников;
- подписка на других путешественников, чтобы следить за их активностью;
- поиск популярных мест для путешествий и просмотр постов с этих мест;
- просмотр ленты других путешественников;

### Нефункциональные требования:
- Доступность сервиса 99.95%
- Регион СНГ
- DAU: 10 000 000 уникальных пользователей в день
- Линейный прирост пользователей
- Сезонность: в период отпусков нагрузка выше, соотношение максимальной нагрузки к средней 20:1
- Время ответа менее 1 сек по читающим и пишущим ручкам
- Прочие требования по подсистемам ниже

### Расчет нагрузки:
1. Публикация постов:
    - В среднем 3 поста в неделю от одного пользователя
    - В среднем 200 просмотров новых постов из разных источников (лента, поиск)
    - JSON ~8 КБ:
        - Описание: 4 КБ (2000 символов)
        - Местоположение: 8 байт (4 байта на координату)
        - Ссылка на картинку в CDN и прочие данные: 4 КБ
    - Запись:
        - RPS: 10 000 000 * 3 / 7 / 86400 = 50 RPS
        - RPS (пик): 50 * (20:1) = 1000 RPS
        - Трафик: 8 КБ * 50 RPS = 400 КБ/сек
        - Трафик (пик): 400 * 20 = 8000 КБ/сек
    - Чтение:
        - RPS: 50 * 200 = 1000 RPS
        - RPS (пик): 1000 * (20:1) = 20000 RPS
        - Трафик: 8 КБ * 1000 RPS = 8 МБ/сек
        - Трафик (пик): 8 МБ/сек * 20 = 1.6 ГБ/сек
2. Оценка постов:
    - В среднем 100 лайков на пост пользователя
    - JSON ~6 байт:
        - ID поста: 4 байта
        - ID реакции: 2 байта
    - RPS: 50 RPS (пост) * 100 = 5000 RPS
    - RPS (пик): 5000 * 20 = 10000 RPS
    - Трафик: 6 Б * 5000 RPS = 30 КБ/сек
    - Трафик (пик): 30 * 20 = 6 МБ/сек
    - Просмотр счетчика в рамках чтения поста
3. Комментарии постов:
    - В среднем 10 комментариев на пост пользователя
    - В среднем 10 просмотров комментариев постов
    - В выдаче к посту 10 комментариев
    - JSON ~6 КБ:
        - ID поста: 4 байта
        - Комментарий: 2 КБ (1000 символов)
        - Прочие данные: 4 КБ
    - Запись:
        - RPS: 50 RPS (пост) * 10 = 500 RPS
        - RPS (пик): 500 * 20 = 1000 RPS
        - Трафик: 6 КБ * 500 RPS = 3 МБ/сек
        - Трафик (пик): 3 * 20 = 600 МБ/сек
    - Чтение:
        - RPS: 50 RPS (пост) * 10 * 10 = 5000 RPS
        - RPS (пик): 5000 * 20 = 10000 RPS
        - Трафик: 6 КБ * 5000 RPS = 30 МБ/сек
        - Трафик (пик): 30 * 20 = 6 ГБ/сек
4. Поиск популярных мест для путешествий:
    - В среднем 2 запроса в день от одного пользователя
    - Не более 10 популярных мест за один раз в ответ
    - request JSON ~500 Б:
        - Поисковый запрос: 500 Б (250 символов)
    - response JSON ~12.5 КБ:
        - Название места: 500 Б (250 символов)
        - Координаты места: 8 байт
        - Описание места: 2 КБ (1000 символов)
        - Лимит 10 мест (отображаем без постов, они запрашиваются отдельно по клику на место)
    - RPS: 10 000 000 * 2 / 86400 = 230 RPS
    - RPS (пик): 230 * 20 = 4600 RPS
    - Трафик: 500 Б * 230 RPS = 115 КБ / сек in, 12.5 КБ * 230 = 2875 КБ out
    - Трафик (пик): 115 * 20 = 2300 КБ / сек, 2875 * 20 = 57500 КБ out
5. Просмотр ленты:
    - В среднем 10 запросов в день от одного пользователя
    - Не более 20 постов в ленте за раз
    - response JSON:
      - Пост 8 КБ x 20 = 1.6 МБ 
    - RPS: 10 000 000 * 10 / 86400 = 1200 
    - RPS (пик): 1200 * 20 = 24000 RPS 
    - Трафик: 1200 * 1.6 МБ = 2 ГБ / сек 
    - Трафик (пик): 40 ГБ / сек
6. Трафик на медиа:
   - 5 фото по 2 МБ к одному посту
   - Чтение:
     - RPS: 1000 * 5 = 5000 RPS
     - RPS(пик): 5000 * 20 = 100000 RPS
     - Трафик: 5000 * 2МБ = 10 ГБ / сек
     - Трафик(пик): 10 ГБ/сек * 20 = 2000 ГБ / сек
   - Запись:
     - RPS: 50 * 5 = 250 RPS
     - RPS(пик): 250 * 20 = 5000 RPS
     - Трафик: 250 * 2 = 500 МБ / сек
     - Трафик(пик): 500 * 20 = 10 ГБ / сек


Диск по подсистемам:
   Посты:
      SSD Sata:
         1.6 GB/s / 1000 MB/s = 2 SSD
         21000 RPS / 1000 IOPS = 21 SSD
         400КБ * 86400 * 365 = 12.6 TB
      Итого: 21 SSD Sata по 500 ГБ
   
   Оценки:
      SSD nVME:
         6 МБ/c / 3 ГБ/с = 1 SSD
         10000 RPS / 10000 IOPS = 1 SSD
         30КБ * 86400 * 365 = 1 TB данных
      Итого: 1 SSD nVME на 1 ТБ

   Комментарии:
      SSD nVME:
         6 Гб/с / 3 ГБ/с = 2 SSD
         10000 RPS / 10000 IOPS = 1 SSD
         3МБ * 86400 * 365 = 94 TB
      Итого: 4 SSD nVME по 30 ТБ
   
   Медиа (картинки):
      HDD:
         20 Гб/с / 100 МБ/с = 200 HDD
         100000 RPS / 100 IOPS = 1000 HDD
         500 МБ * 86400 * 365 = 16000 ТБ
      Итого: 500 HDD