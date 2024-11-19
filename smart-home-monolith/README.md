## Задание 1. Анализ и планирование.

### Задание 1-1.
Изучите функциональность монолитного приложения:
- Управление отоплением. 
  - Пользователи могут удалённо включать/выключать отопление в своих домах.
- Мониторинг температуры. 
  - Система получает данные о температуре с датчиков, установленных в домах. 
  - Пользователи могут просматривать текущую температуру в своих домах через веб-интерфейс.

Текущий функционал AS IS. Также стоит отметить, что самостоятельно подключить свой датчик к системе пользователь не может (только через вызов специалиста).


### Задание 1-2.
Проанализируйте архитектуру монолитного приложения:
- Язык программирования: Java
- База данных: PostgreSQL
- Архитектура: Монолитная, все компоненты системы (обработка запросов, бизнес-логика, работа с данными) находятся в рамках одного приложения.
- Взаимодействие: Синхронное, запросы обрабатываются последовательно.
- Масштабируемость: Ограничена, так как монолит сложно масштабировать по частям.
- Развёртывание: Требует остановки всего приложения.


### Задание 1-3.
Определите домены и границы контекстов. Например, домен «Управление Устройствами» и так далее. Помимо явно подсвеченных особенностей, упомянутых выше, проанализируйте, на что ещё можно опираться из существующего решения.

В текущем приложении (на основании кодовой базы) можно выделить 1 домен:
- Управление устройствами

Но в рамках описания системы, видится должным, наличие таких доменов как: 
- Управление устройствами
  - подключение устройств
  - получение статуса устройств
  - и тд
- Дом
  - Локация в которой пользователь подключил устройства с датчиками
  - Получение информации о привязанных устройствах для этой локации и тд
- Телеметрия
  - Получение данных от устройств и датчиков. 
- Пользователь
  - Персональные данные
  - Роли
- Billing
  - проведение платежей
  - интеграция с внешними сервисами оплат
- Сервис нотификаций
  - различные виды удомлений пользователя при событиях системы (регистрация пользователя,уведомление об успешно подключенном устройстве и тд)


### Задание 1-4.
Проведите анализ архитектуры монолитного приложения. 
Подробно опишите существующие проблемы монолитного решения в контексте бизнес-задач компании. 
Если вы считаете, что текущее решение не вызывает проблем, аргументируйте свою позицию.

В текущем монолитном решении присутсвуют 2 сущности HeatingSystem и TemperatureSensor.
Они жестко мапятся на таблицы в БД с аналогичными названиями и это будет проблемой,
так как в планах добавить новые виды устройств и датчиков в приложение.

Также в текущей системе вся бизнес логика, отвечающая за текущий функционал, 
сформирована в одном классе, реализующий интерфейс
`smart-home-monolith/src/main/java/ru/yandex/practicum/smarthome/service/HeatingSystemService.java:6`.
Интерфейс содержит методы с семантическими названиями, связанными только с возможными взаимодействиями 
с сущностями HeatingSystem и TemperatureSensor.
В планах компании появление новых устройств и датчиков и функционала, 
НО при добавлении новых устройств мы не сможем переиспользовать имеющийся интерфейс из-за его жесткой привязки к текущему контексту.
Видится полезным унификацию общего интерфейса для всех видов устройств 
и при необходимости расширинем поведения моделей черз дополнительные узкоспециализированные 
интерфейсы (согласно одному из принципов SOLID).

Текущее взаимодействие в системе только асинхронное. Могут/будут проблемы с производительностью при росте числа клиентов и устройств.

Видится правильным переработка текущего приложения с помощьюно оддного из двух подходов:
- модульный монолит (по принципам DDD и слоенной архитектуры).
- микросервисная архитектура.

Конечную архитектуру следует выбрать исходя из:
- целей бизнеса по массштабированию функционала и требуемых количественных показателей итоговой системы (RPS, product time to market, etc.)
- текущей команды проекта: количество сотрудников, уровень компетенций, бюджета на обучение и найм нового персонала)

### Задание 1-5.
Визуализируйте контекст системы. 
Создайте диаграмму контекста (Context diagram) в модели C4 с помощью PlantUML. 
Диаграмма должна наглядно показывать, как монолитное приложение взаимодействует с внешним миром (пользователи, датчики).

Схема контекста 
- /c4-diagrams/as-is-c4-context.puml 

Схема контекста визуально по url
- https://www.plantuml.com/plantuml/png/fLHFRn9V5DtpATwF5KrQsFnMhafROfjGHQ5fN941Njg9mmoPUKJTWQqQGsErQQ8n9YvSkCKMx003_GhtVIFlVOpycJu8ZJ5sPYxldNFkkNysFM6uebwrOl-PTjcgLpWx5gBcFKYbNED5yiWKn_LItUDksR45jqMox5HJDSkmwSw69ogMMKfj_x-Hez31VZPLDJp1NGmLj_66OThSJTO8FvPDFtjyK2WUxXLsYzdCmyoZdSHQB2PCON46b-136wvb0_y6-A_De0yZ6ChNq8M-RE8JGm65yFy0V0pGspMC6DpYDH-38_YDfuVuupbBvzbUFXRBSTTpxCG1Ph3EulI8unE-4uWyWnjyNa6RMEYRoIPURghFGBQOF840DRIWnw23HuFusb_4TALgsgOdN4Cu13tthJW-ufK0TGt7aAYHU4QomH9mXPXTPE-e97h4UmjT-LQvC1XVM5j5JIp_2lsk9WvvALshMMRP4APZ8_rXPeiHETsM3NcA63tP95SHdbmYH-DB3dRGa7FY97VRwD8Wz9Q2OK8edTx8GaeLzOHob1ihQj8cJzLkBp9m5U4Aie7AVFbkfbFmfKCUo0yONfmF0YCGyU4ndreHOxDLmxT0ApnIbtt7TYmSedaZjepoSsvND9qBNLNcgM9Lw772y9NQFnGwK4ctM3gtjxnMOTEGamiBjqxvqiNGNn-JR5DUoaZTh93PwYT6Q4GFGNAQzqvfalcE8OoIuN5Au1DStcU8amn-AZfb36aaNSXAJf-f0i6GafDlo9-SuuaZb-UVPAT2YfcN8h7BqNZxQ1mX7J_ksg1t3qRa23A68Ph4k4A87eRktBWITFGs0gAm6Lq5NgrnN1Xr5CrjJ-qHk9pfEf-5QAgPrGydL7KbuPloqP-KR0ZNYuwZ7K-v5Mx1o5XEjk49aawjHbjF5N0mxWGoVmI_T2tKHVnbyCfjLIfge3wDzWr-j2TylKMwb-DUq22du4Asw4MZJ3zAeHlH3Qu-DTv4raXNkqGml2ezezHdTmbTZUxbK69KZdwRx1OAEKPpGUH8tOgWAKPQ5FUPlJcNxgFZyxoIoHB_HnrQIisEmUbVVMii1TTCfsvK55KVG-_nkRoJzJntjjD5jBTIj4YYGVrsMPtb2xeIqCq9e5wwdcUJsvLwrVe3 




## Задание 2. Проектирование микросервисной архитектуры
## Задание 2-1. Декомпозируйте приложение на микросервисы.
Клиент


бразуер

APi Gateway


Домены:
- Device Service
- House Service
- Telemetry Service
- User Service
- Billing Service
- Notification Service

## Задание 2-2. Определите взаимодействия между.

Api Gateway - входная точка для всех внешних запросов. Перенаправляет запрос во внутренния микросервисы.

Kafka - как шина данных для асинхронного взаимодействия.

БД:
- Clickhouse для хранения информации собранной с датчиков и устройств. Telemetry Service.
- PostgreSQl для остальных микросервисов.


## Задание 2-3. Визуализируйте архитектуры.

Диаграмма контейнеров
- /c4-diagrams/c4-containers.puml

Схема контейнеров визуально по url
- http://www.plantuml.com/plantuml/png/nLZTJXj75BxtKqnzPIJ7hghq9bSXW20gfAGsGhr22qnWbVMFrkj4gAh4JroI4ZMfXDJUeGgfxQKXRB2DRLvXzerwljdnsZlCabMGcbQEFMTspiztldFcB0yRdkLwpOgJ-SgkRZZDJSwsFQ_UU50ikjRBmfRjRJVNcmtkRjIg7gzwXOrQfLXth2hEt6jucuwzNfovVsyPefMbnQBB7MurUF75DuLl2r_JrjeCdRJiAdSBTTZ9P1QdV_n-fRos-hYyiBOuDp_tP3OtbSbujkTmDdEVYMFH4Mtn8TYbV_lqVvk9dXYAGR0lVD4L0-7Jrb2Sis1FT88zAUu7HqoSYZuzTIIwJ5pG0_q26LlcRgDMpQrG27cMnLUMliK9xLy5RyKb58asT62jgmmED9NPgP76wSsANRKRdcjvDQYEhQMDztIaJx4CGfNaDEmCWrsM4-UaqoVh5xGx55tOlQQuVf6XNuK7fj9uaodjD3nUoQtozUbwtR4tBC-kLSdywjmZ1kCu7Um6hu1Ti4TGGZqjiD7EtlBW1G7o3ZQlPIREwPoFMAHG1KIkunSWHAZaZt97ef7M9rFmo7ex2S0qZaCbFNbEPmybo1_44C8XxHC6mMyaJew754CHsQF7Ek0VP1DPerzzi-CgBpzmHx5EWo5Yty780-67SRn2R20yDCA1eJWpOXsqoC9j1lwI-HY6_WJlHlIMEFX65236BeKOj0n6Fs7mL6U9y1DPedYiKW8c8sQ5cLvNPYA9FqGR5X4YAhytGHQ4rqQNG4OAp5JgsCaR_KFUwTS5l55SNdjKQrOtBNSdborLg5akr2ey6yQiLoFxASFelwZZvQRhzhpbyPVM3Zh4ycEcLZXFUwCb2ruZ5-8o8kyvKSiFMlG3aT0sGOhbvJXkX4DkQXPd-GjxWvUuYo-o4gwP4cGbEQB2s4ikx_I9Uv30fpp3MG0kzHtbnzZ6VPvTLrxFMfwrRZNWjlXT73FO53qLRUGp9IaM969QhZMyBPUNdYxUHAJC7LxXdhip1YKI9UAYMEAJ2pwWG3rIeE_9F2gr9S4PgFIJy0obcsVO0FcY6b0NcK-8eQg1BjeAZeQ5jfUo-HX0Yw98ZPiE1SAUeR9wlbsZE_ucT0iGZs6JosJ8pEH04mj-5JuvjA_4n8Gh-kwD8A7rjRb8WOI9ZVcm4deIsciaIYN3bnlA3Zu6t116UgxXwRHKUsGxZbtT6YEc18cOYI6yHxoeed2ieLWknlnHsoJuYFkjmCIVQX82hwAReXVh5A8V6nrGlHEc390eFrE3efvF2GikZJ4cM2K2Wiq7XjiGTJDCeWbQ3Q03OP9x35Id3X0FfupkISspdwchQHpbfFIsaf7f1b6ZYxDByP_gSY2G6BxeIl8WT7IVIm0H74OT1XL8F_UX8sYP6KIFQGo2B8O9H0aUirnfgPHdvVayAosLbrc3ukNkb249ax2aJeO1ONDImWlxQPCtUUuxwzbp2wh_1Pj4EmQ4w4A1V8I9ln56E5u2kH6wOe1ZadZ18Vn1tC5RJG-P_ZaJJYxwb1Ax_1VAXClaqA9DuapwY151zgr1I6VQKHrLPLWspeyZHf3nQ6HAOLqRcgA9wREjgy0dJUdpXpXHq941mum89f7u87W3aQAiz5OVTQIBCKgIVTqwclKNCwwQufSoZvRrVzcMnhLKtvZXg31Zsl06UqYSKlGfXdlGA3fXF8PPBPBSJN6S65ZTqLCzsMfzDwMn569nDtN6csbI-9dgZOKVzTiJkYFEq3x60vxMCzczYI5O-Ec7u08Jn-7xKLiVn76GmGYeBiVnVJhJAP2l1gwQdKroLxs7gxVzrz9Ot_pMqqaPYPb5T_KBLJmnwD6dPIWipTG_eomBNpUGctnNcM80DNryPWQcivmF_zPoIH-wYGdVDtZzDKqVwetgwXFZKUUsloQf5-c7lBhPh3Z_0G00 



Диаграмма компонента микросервис Notifier Service
- /c4-diagrams/c4-component.puml

Схема компонента визуально по url
- http://www.plantuml.com/plantuml/png/nLPVJpj557tlfnZxn2-1zeKddn0anWW6AJwJfHtAnlrJTsTLOaoWZOA1qAWy66CqqKHzBCMcfN_Qht3d6tdchcMtxHA8nBGakxCpS-USUy-zTzW8bHMeo7LoRzbUsOagKfmeLGlVBXO3w_D2rLOdqL4Kog3iUqfwgb3stMBDiJnZinQgYcCV5RVMr_RCrCUxEqNN2fKCC7Mu1GlBzcHGgFrFvxirtpCsV7vEsSgHOcjTq2trg4Nt-WpF8VvQWleqnc22vuWwDD9rCpd4lZvsZVKvTV1kKm-ZblZGL_QnNRQKxNib6NncbwMWji0Qrc7QWML7qEzuZVGLFGZwsnnVo9bVwHG4tSDDF_8gLd2waiyuBRygylHp5hhG3GmwyO4qC5onxI43NZVcuFDln9SvWTzZO5O-i8u_iQ136Bao02P_Y-c4OJ4T2vnmnm6x0w6wlZ3a3EIO5xvDY61Lw4kCWSqRi7m5LbZClydD4TYNDJ-qbH-S_kiv89Cv3iDFyyhWK443VDyR3dYpWmFTH12e2uOzx1zZgaCF9XuSk2PRVKUtu31BuQ2QG1-y9soaMJLWCLuE3sSvxfnI-XjyzwQSilIAsN1C9cPUq6_qmmAnRTUodOGRVow77aCly2kvOSAkj5jQ3ZS0cxJwwh6mjhzGA-cIUdSJNFRyK5K36Nxg22HmowG9K30OcJJLNpzTwzdr1iWOxgD8HZAk8-FoNyOVQitd_MDfqHzn9BYI1bZhOb-RUS0CpW7_Wl41gQyNwoVbv85qJBXTZ1CTw7iJR7qPqnHfPwPXDHLA4sRGnkbzmV67Qqb7VCvhneV-huQ6UY_2uWnyDThsxZllxpoDbzkNpbJA-Sv8lt9BcSHPGICHorp7x8IJkGSACtBf9WspSpaRD8MMRadqepxdL1_eQ-hg1joimop3YcFqNmmXvBDcR3UBaA79Bwc0zwdrP0LFKxoRKV_wXjj4EoNlS4RU-2vZGcdc62-NJ4grf3INHIXBvzHbcortqenwrC6ffbVhvYhJC8p1_Xoa1y0vqmrzmzV-FOr5wX-3HdBHCeSCq8NkJ7-oammBCC6tE9UQlfvfqyQn3UbL8jVv1m00



Диаграмма кода микросервиса Notifier Service
- /c4-diagrams/c4-code.puml

Схема кода визуально по url
- http://www.plantuml.com/plantuml/png/jLJTYXCn5BxFKnnwrIej2buDKdQtEqZLAknq7o1EJsjqaYcJJB68eDvwwnlu0cMri2wrpv2-aMSoKvsq3gYm32GvFz_tdPmaSw8Dpqmk4yQCC0d2u07Op_RQhkpNtNiQD_IjmDxOhVrMB3OKkw78TlV1hcc-ijUqMi6pr8YfYBaHgOemMuWOmLu1nIY--sZNX5o3_KBJZzqd-ns8RcitFSRYX6iDJ_ZqDH-aIkSICtZB0EukKZ61k7IrSO7A19vCMBWwx5r5KSQMKOoAPo9r75t95PzXLL00exh92X7bGCtqlYyotEIQ96kQFg2K1SStd0GGcKoecVEfMkvuEKUYFl2uD1xlJGzjX4Hjk9m7SCuDZidyhKGU7i1fbl6bAmvLBe_ONMdHAA9n_8Y6S7GwVEgd-_ii0PU3GHWLe8lmSJWOrpRlTRFQk2ujNLNpC7nZC5CygRA6YimfZx7FVXtn3CtPiYYmRVxObVeHrq8NE4-rC6csBANZL2cCpVbP0CzVlAALKt3absBIpaafAFlPAGAQBx1ErwuVpvx2IOjzpT2qcVA-QbIJvejKwriazG5JkmD2pXEKbAcRPVvMfvGPp_vBvTz4Gib5SgiwpFyrTBlzWvSUW78sEyAxvDRzrl53yncWr-iVgzTulMGY968t-IyWW9TSCz-tJxxNwzrfDVV2GG-5gFg66nz0hb5JRqweVRbCVW80



## Задание 3. Разработка ER-диаграммы

Диаграмма ER 
- /c4-diagrams/er-diagram.puml

Схема кода визуально по url
- http://www.plantuml.com/plantuml/png/dPB1QiCm38RlVWhHuo27NiiO7UIm3BR1sZw0gSLgY9rWEnkXzDsdM50dgGDRZ_b_VVebnLiVf0kTgOKGQ8CEFQmE7jqApWBW0RI23HmELHc_EdAiD4ZQBZXjlwBQIkz_6gUcAXgfwrdXgR4pbk93vwl4WhYaIJwR6XSdgMQ1ZeglmcMwpYx6v5ln2913NnhAoLEtPv1AEVG-lzzo-pj7A_5R7_yycuhL-xx5IlrZmFAAvOC2ESlSdPljiKQ3mVMb339FyphLH3iwb07LSo1nhmrIWccvclgG-BhxU1zonF0We2YQCmplYvoJz6_Izcmy5iKJf3T3zdgp7Z7nfusW-1qfjiOOcypnJKOW2qocGpDuQwQ-oUR6atkD-eijMjMP-XS0 

=======================================
# Практика Часть 2.
Добавил вторую часть практики - схему api микросервисов. 
Если есть возможность, пожалуйста, посмотрите. Волнуюсь, чтобы успеть до жесткого дедлайна.

В формате OA добавлены описания API микросервисов
- oa-schema/device-microservice-api.yaml
- oa-schema/house-microservice-api.yaml

Описания корректно отображаются в Swagger (https://editor.swagger.io/)


=======================================

# Smart Home Monolith

## Описание

Проект "Smart Home Monolith" представляет собой монолитное приложение для управления отоплением и мониторинга температуры в умном доме. Пользователи
могут удаленно включать/выключать отопление, устанавливать желаемую температуру и просматривать текущую температуру через веб-интерфейс.

## Структура проекта

Проект организован в соответствии со стандартной структурой Maven/Gradle проекта:

- **Основной класс приложения** аннотирован `@SpringBootApplication`.
- **Пакеты**:
    - `controller` - контроллеры для обработки HTTP-запросов.
    - `service` - сервисы для бизнес-логики.
    - `repository` - репозитории для доступа к базе данных.
    - `entity` - сущности JPA.
    - `dto` - объекты передачи данных (DTO).

## Зависимости

Проект использует следующие зависимости:

- Java 17
- Maven >=3.8.1 && <4.0.0
- `spring-boot-starter-web` - для создания веб-приложений.
- `spring-boot-starter-data-jpa` - для работы с JPA и базой данных.
- `postgresql` - драйвер для работы с PostgreSQL.
- `springfox-boot-starter` - для интеграции Swagger.
- `lombok` - для упрощения написания кода.
- `spring-boot-starter-test`, `spring-boot-testcontainers`, `junit-jupiter`, `testcontainers` - для тестирования.

## Запуск

1.

``` shell
docker run --name postgres-db -e POSTGRES_DB=smart_home -e POSTGRES_USER=your_username -e POSTGRES_PASSWORD=your_password -p 5432:5432 -d postgres:13-alpine
```

2.

``` shell
mvn spring-boot:run
```

## Тесты

``` shell
mvn test
```

## Конфигурация

### Подключение к базе данных

Конфигурация подключения к PostgreSQL находится в файле `application.yml`:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/smart_home
    username: user
    password: password
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

## Логирование

В проекте используется SLF4J для логирования.

## Тестирование

В проекте написаны модульные тесты для сервисов и контроллеров.
