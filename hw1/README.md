# Домашнее задание #1.

## Задание.

Реализовать простое веб приложение, у которого обрабатывает запросы на два endpoint’а:
- GET /time - этот endpoint должен отдавать текущее время
- GET /statistics - этот endpoint должен отдавать количество обращений к endpoint’у /time в виде одного числа. Соответственно, ваше приложение должно хранить информацию о каждом обращении и отдавать эту информацию по запросу

Далее необходимо реализовать скрипт, который будет обращаться к endpoint’у GET /statistics и записывать полученный результат в файл. Этот скрипт должен будет вызываться раз в 5 секунд.

Все перечисленное выше должно запускаться в Kubernetes. Для этого вам необходимо:
- контейнеризировать ваше приложение и скрипт(docker образы)
- написать манифесты конфигураций для этих двух образов
- bash скрипт для сборки образов и запуска всего внутри Kubernetes

В readme проекта опишите куда надо сделать запрос, чтобы получить результат


## Решение.

В текущей директории (`hw1/`) запускаем скрипт `./refresh.sh`, (возможно, для запуска потребуется прописать `chmod +x refresh.sh`). 
Скрипт делает следующее:
- Удаляет minikube ноду (было необходимо для оперативного тестирования)
- Поднимает новую minikube ноду
- Ставит в качестве окружения docker. Это необходимо, чтобы minikube видела локальные образы и не было необходимости пушить образы на dockerhub. (после данного действия может перестать работать `docker run` / `docker build`, поэтому чтобы "отменить" переключение, необходимо выполнить `eval $(minikube docker-env -u)`)
- Запускает `./run.sh` (возможно, для запуска потребуется прописать `chmod +x run.sh`) - скрипт, который собирает все образы и накатывает настройки на кластер. 

Для проверки, что всё корректно работает, помимо запуска `./refresh.sh`, надо после действий выше отдельно произвести следующие шаги (находясь в этой же директории):
- `kubectl get pods` - получаем список подов, их состояния
- `kubectl get cronjobs` - получаем список cron job, их состояния
- `kubectl get jobs` - получаем список сработавших job'ов, их состояния
- `kubectl describe pod webapp-deployment-**hash**` -  проверяем, что всё работает
- `kubectl logs script-cronjob-**hash** -c script` - видим, что curl запросы происходят (видно классический лог curl'а, ошибок нет)
- `minikube service webapp` - создается туннель для удобного доступа к приложению, здесь мы ожидаем `404 page not found` в открывающемся браузере (так как у нас нет endpoint'а `/`)
- Нас интересует endpoint `/time` - дописываем его в строку поиска, должны увидеть что-то вроде `Mon, 13 May 2024 13:40:27 UTC`
- Аналогично можем зайти на endpoint `/statistics`