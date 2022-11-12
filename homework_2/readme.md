1. создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом. Поставить на нем Docker Engine.
- Инструкция: https://docs.docker.com/engine/install/ubuntu/
- Комментарий: The convenience script isn’t recommended for production environments, but it’s useful for creating a provisioning script tailored to your needs.
- ``curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER``
- ``sudo docker network create pg-net``
2. сделать каталог /var/lib/postgres
3. развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres
- ``sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14``
4. развернуть контейнер с клиентом postgres
- ``sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres``
5. проверим, что все ок:
- ``sudo docker ps -a``

| CONTAINER ID  |  IMAGE      |                         COMMAND |            CREATED |            STATUS |                                             PORTS | NAMES |
|:--------------|:-----------:|--------------------------------:|-------------------:|------------------:|--------------------------------------------------:| ---------:|
| c998ef1022bd  | postgres:14 |          "docker-entrypoint.s…" | About a minute ago | Up About a minute |                                          5432/tcp | pg-client |
| b5c4aa4ca136  | postgres:14 |          "docker-entrypoint.s…" |     7 minutes ago  |     Up 7 minutes  |         0.0.0.0:5432->5432/tcp, <br/>:::5432->5432/tcp | pg-server|

6. подключится из контейнера с клиентом к контейнеру с сервером и сделать
таблицу с парой строк.
- ``otus=# SELECT * FROM test;``

```sql
    i | amount
   ---+--------
   1 |    100
   2 |    500
   (2 rows)
```
7. подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
- ``psql -p 5432 -U postgres -h 158.160.40.12 -d postgres -W``
8. удалить контейнер с сервером
- ``sudo docker stop b5c4aa4ca136``
- ``sudo docker rm b5c4aa4ca136``

9. создать его заново
- ``sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14``
10. подключится снова из контейнера с клиентом к контейнеру с сервером
- ``sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres``
11. проверить, что данные остались на месте:
- ```sql select * from test;```
- видим также 100 и 500.

