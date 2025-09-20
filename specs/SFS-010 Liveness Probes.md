# SFS-010 Liveness Probes

Приложение позволяет делать пробы своей работоспособности по HTTP или TCP-порту. 

При выполнении Liveness/Readiness проб приложение НЕ должно открывать новые сетевые соединения или
делать запросы по сети. We probably shouldn't be checking the availability of our dependencies in readiness checks due to the potential for cascading failures. 

Startup-проба может быть умной, сложной. Liveness проба должна быть очень простой. Относитесь к Liveness пробле как с способу узнать нужно ли приложние перезагрузать.
Readiness.

Приложение пишел в log когда приходит за прос на пробу и результат ответа, но только в log-level=`debug`

## Виды проб

* Liveness (работоспособности) - для того, чтобы узнать что контейнер жив. Если эта проба не проходит, то pod перезапустят. Делается по пути `/live`
* Readiness (готовности) - для того, чтобы узнать когда проверяемый контейнер будет готов принимать сетевой трафик. Если эта проба не проходит, то с pod-а снимут трафик. Делается по пути `/ready`
* Startup (запущено) - для того чтобы определить когда приложение запустилось и можно переходить к проверкам liveness/readiness. Пока эта проба не состоится, pod не пойдет дальше по жизненному циклу.

## HTTP-пробы

Если приложение поддерживает HTTP-пробы, то порт отвечающего сервера указыватся через переменную окружения `HEALTH_PORT`

* Liveness-проба делается по пути `/live`. Данный вид пробы желателен, но не обязателен. Если приложение не поддерживает такой endpoint, то шедулером используется Readiness-проба
* Readiness-проба делается по пути `/ready`
* Startup-проба делается по пути `/startup`. Желателен, но не обязателен.

Удачным считается любой ответ со статусом 200.

HTTP-запрос должен укладываться в `30ms` и не зависеть от нагрузки на сервис.

## TCP-пробы

* Порт для liveness-пробы указывается через переменную окружения `LIVENESS_PORT`
* Порт для readiness-пробы указывается через пременную окружения `HEALTH_PORT`

Успешной считается любая проба которая удачно открыла соединине на нужном порту.

## gRPC-пробы

Подробнее: https://github.com/grpc/grpc/blob/master/doc/health-checking.md

* Порт для liveness-пробы указывается через переменную окружения `LIVENESS_PORT`
* Порт для readiness-пробы указывается через пременную окружения `HEALTH_PORT`


# Больше информации

<img src="https://andrewlock.net/content/images/2020/k8s_probes.svg" />

* SFS-021 Liveness probes over commands - https://github.com/Brandymint-com/wiki/blob/main/docs/specs/SFS-021%20Leveness%20probes%20over%20command.md

* [What is difference in Liveness Probe, Readiness Probe and Startup Probe in Kubernetes?](https://medium.com/@edu.ukulelekim/what-is-difference-in-liveness-probe-readiness-probe-and-startup-probe-in-kubernetes-e116c4563c13)
* https://faun.pub/the-difference-between-liveness-readiness-and-startup-probes-781bd3141079
* https://andrewlock.net/deploying-asp-net-core-applications-to-kubernetes-part-6-adding-health-checks-with-liveness-readiness-and-startup-probes/

# Комментарии

* Если readiness пропало, а live живой то с нео просто снимают трафик и ждут когда появится readiness.
* Если liveness пропало, то, вне зависитимости от того что возвращает readiness, контейнер тупо прибьют.
* Важное свойство liveness-пробы - она безусловная, то есто всегда отдает OK.

