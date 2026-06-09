Задание 2. Мониторинг

Сайт «Александрит» подключён к Яндекс Метрике. Но с тех пор как бизнес начал предоставлять оформление заказов через API, данные Яндекс Метрики уже не дают полной картины. Чтобы начать улучшать систему, вам нужно от чего-то отталкиваться. В этом задании вы запланируете внедрение мониторинга.
Вам нужно определить, что вы хотите измерять и как вы будете это делать. А затем —  постараться обосновать свои решения для бизнеса. Не забывайте: бизнесу не всегда очевидно, что мониторинг стоит того, чтобы выделять на него ресурсы команды.
Что нужно сделать
Создайте в директории Task2 текстовый файл и назовите его «Выбор и настройка мониторинга в системе». Вы будете оформлять своё решение в виде технического документа.
Проанализируйте систему компании и C4-диаграмму в контексте планирования мониторинга.
Добавьте в файл раздел «Мотивация». Напишите здесь, почему в систему нужно добавить мониторинг и что это даст компании.
Добавьте раздел «Выбор подхода к мониторингу». Выберите, какой подход к мониторингу вы будете использовать: RED, USE или «Четыре золотых сигнала». Для разных частей системы можно использовать разные подходы.
Опишите, какие метрики и в каких частях системы вы будете отслеживать. Перед вами список метрик. Выберите метрики, которые вы считаете нужным отслеживать. Для выбранных метрик напишите:
Зачем нужна эта метрика.
Нужны ли ярлыки для этой метрики. Если ярлыки нужны, опишите, какие именно вы планируете добавить.
Вы можете не ограничивать себя только этим списком. Если вы видите, что стоит добавить какие-то ещё метрики, — добавьте и их тоже.

Список возможных метрик для отслеживания

Number of dead-letter-exchange letters in RabbitMQ
Number of message in flight in RabbitMQ
Number of requests (RPS) for internet shop API
Number of requests (RPS) for CRM API
Number of requests (RPS) for MES API
Number of requests (RPS) per user for internet shop API
Number of requests (RPS) per user for CRM API
Number of requests (RPS) per user for MES API
CPU % for shop API
CPU % for CRM API
CPU % for MES API
Memory Utilisation for shop API
Memory Utilisation for CRM API
Memory Utilisation for MES API
Memory Utilisation for shop db instance
Memory Utilisation for MES db instance
Number of connections for shop db instance
Number of connections for MES db instance
Response time (latency) for shop API
Response time (latency) for CRM API
Response time (latency) for MES API
Size of S3 storage
Size of shop db instance
Size of MES db instance
Number of HTTP 200 for shop API
Number of HTTP 200 for CRM API
Number of HTTP 200 for MES API
Number of HTTP 500 for shop API
Number of HTTP 500 for CRM API
Number of HTTP 500 for MES API
Number of HTTP 500 for shop API
Number of simultanious sessions for shop API
Number of simultanious sessions for CRM API
Number of simultanious sessions for MES API
Kb tranferred (received) for shop API
Kb tranferred (received) for CRM API
Kb tranferred (received) for MES API
Kb provided (sent) for shop API
Kb provided (sent) for CRM API
Kb provided (sent) for MES API


Добавьте раздел «План действий». Напишите высокоуровнево, какие задачи вы видите для реализации. Это будет драфт технического задания. Например, «Создать инстанс time-series базы с использованием такой-то технологии».
Дополнительное задание. Выберите показатели насыщенности — определите, что является пороговым значением насыщенности и почему нужно использовать именно такие показатели. Опишите, что должно происходить в системе в случае, если эти параметры будут превышены. Например, нужно завести тикет, добавить инстансов, написать письмо в саппорт, добавить автоматическую «звонилку» и так далее. Если вы сдадите работу без этого пункта, это не повлияет на проверку задания ревьюером.