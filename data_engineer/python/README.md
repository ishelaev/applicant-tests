# Задания

## Python

Ниже даны 3 варианта заданий. Решить нужно какое-то одно.
Если успеваете в срок решить несколько — пожалуйста, такое будет плюсом 😀 

### 1 (качество кода, SOLID, принцип единой ответственности, Dependency Injection)

Выполните рефакторинг кода ниже. Реализуйте новый *payment_type* (скажем, *bank*). Убедитесь, что добавление новых *payment_type* осуществляется легко.<br>
*Подсказка: руководствуйтесь принципом единой ответственности. Также можно применить Dependency Injection.*

```python
class Order:

   def __init__(self):
      self.items = []
      self.quantities = []
      self.prices = []
      self.status = 'open'

   def add_item(self, name, quantity, price):
      self.items.append(name)
      self.quantities.append(quantity)
      self.prices.append(price)

   def total_price(self):
      total = 0
      for i in range(len(self.prices)):
         total += self.quantities[i] * self.prices[i]
      return total

   def pay(self, payment_type, security_code):
      if payment_type == 'debit':
         print('Какая-то логика реализации debit...')
         print(f'Верифицируем код: {security_code}')
         self.status = 'paid'
      elif payment_type == 'credit':
         print('Какая-то логика реализации credit...')
         print(f'Верифицируем код: {security_code}')
         self.status = 'paid'
      else:
         raise Exception(f'Неизвестный тип платежа: {payment_type}')


def main() -> None:
   order = Order()
   order.add_item('Keyboard', 1, 50)
   order.add_item('SSD', 1, 150)
   order.add_item('USB cable', 2, 5)
   print(order.total_price())
   order.pay('debit', '0372846')


if __name__ == "__main__":
   main()
```

### 2 (качество кода, Dependency Injection, Dependency Inversion)

Код ниже работает без ошибок, но, возможно, требует определенного рефакторинга.
Внесите изменения в код так, чтобы добавлять новые **LampSwitcher'ы** было проще.

```python
from typing import Union


class GlowLampSwitcher:

    def __init__(self) -> None:
        self.on_state = False

    def on(self) -> None:
        print('Лампа накаливания включена...')
        print('Логика реализации включения лампы накаливания...')
        self.on_state = True

    def turn_off(self) -> None:
        print('Лампа накаливания выключена...')
        print('Логика реализации выключения лампы накаливания...')
        self.on_state = False


class HalogenLampSwitcher:

    def __init__(self) -> None:
        self.on_state = False

    def turn_on(self) -> None:
        print('Галогенная лампа включена...')
        print('Логика реализации включения галогена...')
        self.on_state = True

    def off(self) -> None:
        print('Галогенная лампа выключена...')
        print('Логика реализации выключения галогена...')
        self.on_state = False


class AnotherLampSwitcher:

    def __init__(self) -> None:
        self.on_state = False

    def lamp_on(self) -> None:
        print('Ещё лампа включена...')
        print('Специфическая логика реализации включения лампы...')
        self.on_state = True

    def lamp_off(self) -> None:
        print('Ещё лампа выключена...')
        print('Специфическая логика реализации выключения лампы...')
        self.on_state = False


class ElectricLightSwitchManager:

    def __init__(
        self, switcher: Union[GlowLampSwitcher, HalogenLampSwitcher, AnotherLampSwitcher],
    ) -> None:
        self.switcher = switcher

    def press(self) -> None:
        if isinstance(self.switcher, GlowLampSwitcher):
            if self.switcher.on_state:
                self.switcher.turn_off()
            else:
                self.switcher.on()
        elif isinstance(self.switcher, HalogenLampSwitcher):
            if self.switcher.on_state:
                self.switcher.off()
            else:
                self.switcher.turn_on()
        elif isinstance(self.switcher, AnotherLampSwitcher):
            if self.switcher.on_state:
                self.switcher.lamp_off()
            else:
                self.switcher.lamp_on()


def main() -> None:
    switch = ElectricLightSwitchManager(GlowLampSwitcher())
    switch.press()
    switch.press()
    switch = ElectricLightSwitchManager(HalogenLampSwitcher())
    switch.press()
    switch.press()
    switch = ElectricLightSwitchManager(AnotherLampSwitcher())
    switch.press()
    switch.press()


if __name__ == '__main__':
    main()

```

### 3 (парсинг, агрегация данных)

Необходимо:
1. Получить данные по комментариям и постам с ресурса http://jsonplaceholder.typicode.com/
    * http://jsonplaceholder.typicode.com/posts
    * http://jsonplaceholder.typicode.com/comments

2. Посчитать среднее количество комментариев к посту каждого
   пользователя, результатом должен быть словарь формата:
    * user_id
    * average_comments_per_post
   
3. Результат вывести в stdout (например `print`).
   
```python
import requests

def get_data(url):
    # Функция для отправки GET-запроса по указанному URL и получения данных в формате JSON
    response = requests.get(url)
    return response.json()

def calculate_average_comments(posts, comments):
    # Функция для расчета среднего количества комментариев на пост для каждого пользователя
    user_comments = {}
    # Подсчет количества комментариев для каждого поста
    for comment in comments:
        post_id = comment['postId']
        if post_id not in user_comments:
            user_comments[post_id] = 0
        user_comments[post_id] += 1
    
    user_posts = {}
    # Подсчет общего количества постов и комментариев для каждого пользователя
    for post in posts:
        user_id = post['userId']
        if user_id not in user_posts:
            user_posts[user_id] = {'total_comments': 0, 'total_posts': 0}
        user_posts[user_id]['total_posts'] += 1
        if post['id'] in user_comments:
            user_posts[user_id]['total_comments'] += user_comments[post['id']]
    
    # Расчет среднего количества комментариев на пост для каждого пользователя
    average_comments = {}
    for user_id, data in user_posts.items():
        if data['total_posts'] > 0:
            average_comments[user_id] = data['total_comments'] / data['total_posts']
        else:
            average_comments[user_id] = 0
    
    return average_comments

def main():
    # Основная функция программы
    posts_url = 'http://jsonplaceholder.typicode.com/posts'
    comments_url = 'http://jsonplaceholder.typicode.com/comments'
    
    # Получение данных о постах и комментариях
    posts = get_data(posts_url)
    comments = get_data(comments_url)
    
    # Расчет среднего количества комментариев на пост для каждого пользователя
    average_comments = calculate_average_comments(posts, comments)
    
    # Вывод результатов
    for user_id, avg_comments in average_comments.items():
        print(f"user_id: {user_id}, average_comments_per_post: {avg_comments}")

if __name__ == "__main__":
    main()

```

```
user_id: 1, average_comments_per_post: 5.0
user_id: 2, average_comments_per_post: 5.0
user_id: 3, average_comments_per_post: 5.0
user_id: 4, average_comments_per_post: 5.0
user_id: 5, average_comments_per_post: 5.0
user_id: 6, average_comments_per_post: 5.0
user_id: 7, average_comments_per_post: 5.0
user_id: 8, average_comments_per_post: 5.0
user_id: 9, average_comments_per_post: 5.0
user_id: 10, average_comments_per_post: 5.0
```
