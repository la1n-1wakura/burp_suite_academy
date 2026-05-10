---
title: "Write-up: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data" 
author: "Lain Iwakura"
date: "10.05.2026"
difficulty: "Apprentice"
lab: "PortSwigger Web Security Academy — SQL injection"
---

# 1.Резюме

В ходе тестирования веб-приложения `https://lab-target.web-security-academy.net` была обнаружена **SQL-инъекция (SQLi)** в фильтре категорий товаров.

**Тип уязвимости**: `WHERE clause` — некорректная конкатенация пользовательского ввода в SQL-запрос.

**Воздействие**: Злоумышленник может модифицировать логику запроса и получить данные, которые не должны отображаться (включая скрытые, неопубликованные или тестовые товары).

**Результат**: Все скрытые товары (например, `"Gift Card — Secret Edition"`) успешно извлечены без авторизации.

---

# 2. Разведка (Reconnaissance)

## 2.1. Изучение целевого приложения

При переходе на главную страницу отображается каталог товаров с фильтром по категориям (например, `Gifts`, `Pets`, `Clothing`).
*Рисунок 1 — Интерфейс магазина с фильтром категорий*:
![shop interface](images/shop_interface.png)

При клике на категорию `Gifts` URL изменяется на: https://lab-target.web-security-academy.net/filter?category=Gifts
*Рисунок 2 — Интерфейс магазина с применённым фильтром категорий 'Gifts'*:
![shop interface](images/shop_filter_Gifts.png)

## 2.2. Гипотеза (предполагаемый SQL-запрос)

Скорее всего, бэкенд выполняет примерно такой запрос:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```
Где *released = 1* — фильтр, показывающий только опубликованные товары.

# 3. Поиск уязвимости (Vulnerability Discovery)
## 3.1. Тестирование с одинарной кавычкой

Подставим значение Gifts' в параметр category: https://lab-target.web-security-academy.net/filter?category=Gifts'
