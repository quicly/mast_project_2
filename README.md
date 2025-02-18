# Прогнозирование повторных покупок клиентов интернет-магазина

## Цель проекта

Разработать модель машинного обучения для предсказания вероятности совершения покупки клиентом в течение следующих 90 дней. Целевой показатель качества модели - метрика ROC-AUC.

## Описание данных

В работе использованы следующие наборы данных:

- **apparel-purchases**: История покупок клиентов
  - client_id - идентификатор клиента
  - quantity - количество единиц товара
  - price - цена товара
  - category_ids - вложенные категории товара
  - date - дата покупки
  - message_id - идентификатор сообщения из рассылки

- **apparel-messages**: История рекламных рассылок
  - bulk_campaign_id - идентификатор рекламной кампании
  - client_id - идентификатор клиента
  - message_id - идентификатор сообщения
  - event - тип действия (отправка, открытие, клик и т.д.)
  - channel - канал рассылки
  - date - дата рассылки
  - created_at - точное время создания

- **target**: Целевая переменная
  - client_id - идентификатор клиента
  - target - факт совершения покупки в целевом периоде

## Препроцессинг данных

1. **Обработка истории покупок**:
   - Агрегация данных по клиентам с созданием признаков:
     - Общее количество купленных товаров
     - Средняя цена покупки
     - Общая сумма покупок
     - Количество уникальных дней с покупками
     - Дата последней покупки
     - Дата первой покупки
   - Расчет дополнительных временных признаков:
     - Количество дней с последней покупки
     - Длительность клиентского опыта (дни между первой и последней покупкой)

2. **Обработка категорий товаров**:
   - Преобразование вложенных категорий в бинарные признаки
   - Отбор категорий с частотой появления более 2%

3. **Обработка рассылок**:
   - Кластеризация рассылок на основе поведенческих метрик (открытия, клики, покупки)
   - Агрегация действий пользователей по типам событий
   - Расчет активности по каналам коммуникации

## Процедура отбора и обучения модели

1. **Разделение данных**:
   - Train/Validation/Test split в пропорции 60/20/20
   - Стратификация по целевой переменной

2. **Отбор модели**:
   - Тестирование различных алгоритмов (LogisticRegression, DecisionTree, LightGBM, CatBoost)
   - Оптимизация гиперпараметров через Optuna
   - Выбор CatBoost как финальной модели

3. **Параметры финальной модели**:
   - iterations: 308
   - learning_rate: 0.0306
   - depth: 3
   - l2_leaf_reg: 6.39
   - min_data_in_leaf: 38
   - subsample: 0.66
   - auto_class_weights: 'SqrtBalanced'

## Финальные показатели и интерпретация

1. **Метрики качества**:
   - ROC-AUC на тестовой выборке: 0.76
   - Precision для класса 1: 0.18
   - Recall для класса 1: 0.04

2. **Ключевые инсайты**:
   - Модель существенно лучше случайного угадывания (AUC 0.76 vs 0.5)
   - Наиболее важные предикторы:
     - Количество дней с последней покупки
     - Количество кликов по рассылкам
     - Длительность клиентского опыта
     - Количество push-уведомлений
     - Количество открытий писем

3. **Практическое применение**:
   - Модель хорошо идентифицирует клиентов, которые не вернутся за покупками
   - При пороге классификации 0.2 модель способна выявить около 46% клиентов, которые совершат покупку
   - Рекомендуется использовать модель для:
     - Оптимизации маркетинговых расходов
     - Выявления потенциально активных клиентов
     - Разработки таргетированных предложений

4. **Направления улучшения**:
   - Сбор дополнительных данных о клиентах
   - Разработка признаков на основе контента рассылок
   - Учет сезонности и праздничных периодов
