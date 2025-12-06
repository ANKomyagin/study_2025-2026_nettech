---
lang: ru-RU
title: Лабораторная работа 7
subtitle: Адресация IPv4 и IPv6. Настройка DHCP
author:
  - Комягин Андрей Николаевич
institute:
  - Российский университет дружбы народов, Москва, Россия

babel-lang: russian
babel-otherlangs: english

toc: false
toc-title: Содержание
slide_level: 2
aspectratio: 169
section-titles: true
theme: metropolis
header-includes:
  - \metroset{progressbar=frametitle,sectionpage=progressbar,numbering=fraction}
---

# Цель работы

## Цель

Изучение механизмов динамической конфигурации параметров IPv4- и IPv6-сетей с использованием DHCPv4 и DHCPv6, а также взаимодействия DHCP с SLAAC и ICMPv6 в среде моделирования GNS3.

# Задачи

## Основные задачи

1. Развертывание стенда в GNS3 с маршрутизатором VyOS.

2. Настройка DHCPv4-сервера для автоматической выдачи IPv4-адресов.

3. Проверка получения адреса клиентом VPCS.

4. Настройка stateless-DHCPv6 с SLAAC.

5. Настройка stateful-DHCPv6 с полной выдачей адреса.

6. Анализ трафика ICMPv6 и DHCPv6.

# Стенд

## Топология стенда

![Топология: маршрутизатор VyOS с тремя коммутаторами и хостами](image/1.png){width=70%}

## Компоненты стенда

- **Маршрутизатор VyOS** ankomyagin-gw-01;

- **Коммутаторы**: ankomyagin-sw-01, ankomyagin-sw-02, ankomyagin-sw-03;

- **Хосты**:
  - PC1 (VPCS) — DHCPv4 на eth0;
  - PC2 (Linux) — SLAAC + stateless-DHCPv6 на eth0;
  - PC3 (Linux) — stateful-DHCPv6 на eth0.

## Адресация

| Интерфейс | IPv4            | IPv6       |
|-----------|-----------------|------------|
| eth0      | 10.0.0.1/24     | —          |
| eth1      | —               | 2000::1/64 |
| eth2      | —               | 2001::1/64 |

# DHCPv4: Базовая настройка

## Конфигурация VyOS (шаг 1–2)

![Первичная настройка host-name и пользователя](image/2.png){width=60%}

## Конфигурация интерфейсов

![Настройка адресов на eth0, eth1, eth2](image/10.png){width=70%}

# DHCPv4: Конфигурация сервера

## Настройка DHCP-сервера

![Создание DHCPv4-пула ankomyagin с параметрами](image/3.png){width=70%}

## Статистика и пустой список арен

![Проверка конфигурации и статистики DHCP](image/4.png){width=70%}

# DHCPv4: Работа клиента

## Клиент PC1 получает адрес

![Обмен DHCP-сообщениями и выдача IP 10.0.0.2](image/5.png){width=70%}

## Проверка на сервере

![Появление записи об аренде и тестирование связности](image/6.png){width=70%}

# DHCPv6: Stateless (PC2)

## Конфигурация RA и DHCPv6 (stateless)

![Включение RA с other-config-flag и пула ankomyagin-stateless](image/10.png){width=70%}

## Полная конфигурация VyOS

![Вывод всех служб на маршрутизаторе](image/11.png){width=70%}

# DHCPv6: SLAAC и параметры

## Интерфейсы PC2 до DHCPv6

![ifconfig: link-local адреса и отсутствие глобального адреса](image/12.png){width=70%}

## Маршрутизация и первый ping

![route -n -A inet6 и ping 2000::1](image/13.png){width=70%}

# DHCPv6: Получение параметров

## Запуск DHCPv6-клиента (stateless)

![dhclient -6 -S: Information-Request и Reply](image/14.png){width=70%}

## Захват трафика ICMPv6 и NDP

![Wireshark: RA, Echo Request/Reply, Neighbor Solicitation/Advertisement](image/15.png){width=70%}

# DHCPv6: Stateful (PC3)

## Конфигурация RA (managed-flag) и DHCPv6

![Включение managed-flag и пула ankomyagin-stateful (2001::100–2001::199)](image/16.png){width=70%}

## Интерфейсы PC3 до DHCPv6

![ifconfig: только link-local адреса](image/17.png){width=70%}

# DHCPv6: Solicit–Advertise–Request–Reply

## Маршрутизация PC3 до DHCPv6



![route -n -A inet6 на PC3: только link-local и маршрут по умолчанию](image/18.png){width=70%}

## Работа dhclient (stateful)

![Подробный журнал: Solicit, Advertise, Request, Reply с адресом 2001::199](image/19.png){width=70%}

# DHCPv6: Итоги на PC3

## Полученная конфигурация

![ifconfig: глобальный адрес 2001::199/128 и link-local](image/20.png){width=70%}

## Маршрутизация и DNS

![route -n -A inet6 и /etc/resolv.conf с 2001::1](image/21.png){width=70%}

# DHCPv6: Сервер и трафик

## Аренда на DHCPv6-сервере

![show dhcpv6 server leases: активная аренда 2001::199](image/22.png){width=70%}

## Захват DHCPv6 и ICMPv6

![Wireshark: Solicit, Advertise, Request, Reply, Echo ping, NDP](image/23.png){width=70%}

# Результаты

## Проведено

-  DHCPv4: автоматическая выдача адреса PC1;

-  Stateless-DHCPv6: SLAAC + параметры DNS для PC2;

-  Stateful-DHCPv6: полная выдача адреса 2001::199 для PC3;

-  Анализ трафика ICMPv6, NDP и DHCPv6.

## Выводы

- DHCPv4 обеспечивает централизованную выдачу параметров IPv4;

- SLAAC позволяет хостам независимо генерировать адреса в IPv6;

- DHCPv6 дополняет SLAAC, выступая в роли поставщика параметров или полного адреса;

- Протоколы NDP и DHCPv6 обеспечивают корректную работу IPv6-сетей.
