# Why all the hustle?

В этой работе будут рассмотренны базовые методы и практики оптимизации Django для построения высоко-производительных веб-приложений. _Многие_ утверждают, что __Python__ в силу своей природы как _интерпретируемая стек машина_ довольно медленный в отношении компилируемых (и не только) языков. Это действительно так, однако это утверждение содержит фундаментальное заблуждение в оперировании бэкенд задач.

!!! tip "Аксиома"
    __Любую__ вычисляемую __задачу__ можно раздельть на __два__ класса: __CPU-bound__ или __IO-bound__.

Пускай у нас есть задача [__T__], в которой мы хотим _вычислить_ входит ли данное число _n_ в последовательность Фибоначчи:
```python linenums="1"
from typing import Callable
import numpy as np


def is_fib(n: int) -> bool:
"""
>>> is_fib(3)
True
>>> is_fib(55)
True
>>> is_fib(17)
False
"""
  is_perfect_square: Callable[[int, int], bool] = lambda num, root: num == np.power(int(root + 0.5),2)
  value = 5 * np.power(n,2)
  if (root:=np.sqrt(value + 4)) and is_perfect_square(value + 4, root):
    return True
  elif (root:=np.sqrt(value - 4)) and is_perfect_square(value - 4, root):
    return True
  return False
```
В этом примере мы используем только CPU и почти не адрессуемся к RAM или дисковому пространству. Следовательно, мы можем вывести, что данная задача __T:CPU-bound__ [попадает под класс задач, ограниченных CPU], поскольку производительность кода высоко коррелирует с частотой исполнения команд процессора.

Если же задача [__T__] должна прочитать ответы на запросы от _n_ различных ресурсов, это выглядело бы примерно вот так:
```python linenums="1"
from typing import List
import requests

def send_request(url: str) -> bytes:
  return requests.get(url).content

def manage_requests(targets_file: List[str]) -> List[bytes]:
  reponses = []
  with open(targets, "r") as f:
    responses.append(send_request(f.readline()))
  return responses

if __name__ == '__main__':
  results = manage_requests("my_req_files")
  ... # Process results
```
Ужасно. Скрипт не только обратился к диску чтобы подтянуть список url, но еще и отправил __по очереди__ отправил запросы по сети __ожидая__ ответ. Как вы понимаете, это __T:IO-bound__. И это довольно мелденно.

Но в сравнении с Python -- это скоростной болид. Это пример того, почему язык получил широкое распространение в backend сфере. Большая серверных процессов связана с обслуживанием файлов: Запросы БД, JSON парсинг, отправка/обслуживание запросов и т.д. Даже _"очень быстрый" скомпилированный код_ не даст вам большой прирост производительности. Все __эти задачи ограничены производительностью перефирийных устройств__ типа сетевой карты, RTD, жесткого/твердотельного накопителя.  

> Thus, there is no need for hustle.

----

# General optimization strategies

