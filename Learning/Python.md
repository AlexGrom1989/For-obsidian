[[Frameworks & Libraries]]
### String and Format

> Оператор %

```python
print('%d %s, %d %s' % (6, 'bananas', 10, 'lemons'))
print('%+10s' % 'Hello') # -> '     Hello'
print('%-10s' % 'Hello') # -> 'Hello     '
```

> .format()

```python
print('{1}, {2}, {0}'.format('a', 'b', 'c')) # -> 'b, c, a'
print('{2}, {1}, {0}'.format(*'abc')) # -> 'c, b, a'

print('{a}, {b}'.format(a='A', b='B')) # -> 'A, B'

coord = {a:'A', b:'B'}
print('{a}, {b}'.format(**coord)) # -> 'A, B'
```

> F-строки

```python
a = 'hello'
print(f'{a} mr Broski') # -> 'hello mr Broski'
```

> Спецификация формата

 >[!abstract] Спецификация
`[[fill]align][sign][#][0][width][,][.precision][type]`
`fill` - то чем будут заполнены пропуски ( символ кроме { и } )
`align` - выравнивание: `<` - по левому краю, `>` - по правому краю, `^` - по центру
`sign` - знак (должен быть у чисел)
`width` - ширина строки ( число )
`.precision` - точность дробного числа ( .число )
`type` - тип данных (`i` - int, `s` - str, `f` - float )

Пример: 
```python
print('{:*^12}'.format('center')) # -> '***center***'
print('{:~<+10.2f}'.format(123.1)) # -> '+123.10~~~'

a = 123.1
print(f'{a:$<+10.2f}') # -> '+123.10$$$'
```

> Редкие методы строк

```python
s = 'a123b'
s.center(width=10, fill='$') # центрирование -> $$$a123b$$$
s.partition('12') # разделение по первому шаблону -> ('a', '12', '3b')
s.zfill(width=10) # строка не меньше width с заполением нулями слева -> '00000a123b'
```
---

### Nonlocal veriable

> Ошибка при попытке работы с внешней переменной

```python
def foo():
	a = 10
	def goo():
		print(a)
	goo()
foo() # -> 10

def foo():
	a = 10
	def goo():
		a += 1
		...
	goo()
foo() # -> UnboundLocalError: cannot access local variable 'a'

def foo():
	a = 10
	def goo():
		a = a
		...
	goo()
foo() # -> UnboundLocalError: cannot access local variable 'a'
```

Чтобы ошибки не возникало, необходимо указать `nonlocal`:

```python
def foo():
	a = 10
	def goo():
		nonlocal a
		a += 1
		print(a)
	goo()
foo() # -> 11

def foo():
	a = 10 
	def goo():
		v = a
		print(v)
	goo()
foo() # -> 10
```

---

### Decorators

> Декоратор с аргументами

```python
def event_handler(mes: str):
	def call(func):
		print(mes)
		def inner_decor(*args):
			print('inner_decor', args)
			func(*args)
		return inner_decor
	return call
# Есть отличие, но тоже работает. Нет обработки аргументов func
# def event_handler(mes: str):
	# def call(func):
		# print(mes, 'decor')
		# return func
	# return call
  
@event_handler("btn")
def func1(*args):
	print('func11111')
  
def func2(*args):
	print('func22222')
tmp = event_handler('etc')
func2 = tmp(func2)
  
def main():
	func1(1, 2, "func1")
	func2(1, 2, "func2")

main()
```
output:
```
btn
etc
inner_decor (1, 2, 'func1')
func11111
inner_decor (1, 2, 'func2')
func22222
```

> Двойной декоратор

```python
# Будь внимателен, все не так очевидно
# Посмотри на вывод, сначала сработают принты внешнего декоратора!
# потому что связывание функции с декоратором, это то же самое, что
# вызов самого декоратора с передачей в него функции, как в способе 2.
def decor(vfunc):
	def callf(*args, **kwargs):
		print('Execute function', vfunc.__name__)
		return vfunc(*args, **kwargs)
	return callf
  
def external_decor(vfunc):
	print(external_decor.__name__)
	return vfunc

@external_decor
@decor # 1 способ
def func():
	print('func')
  
def func_1(): # 2 способ
	print('func_1')
func_1 = external_decor(decor(func_1))
  
def main():
	func()
	print()
	func_1()

main()
```
output:
```
external_decor
external_decor
Execute function func
func

Execute function func_1
func_1
```

> Декоратор для генераторов/сопрограмм

 см. ниже [[Python#^965795]]

> Фабрика декораторов и `wraps`

```python
''' Задача - логировать вызовы функций '''

from functools import wraps # для копирования описания функции
from inspect import getcallargs # для получения словаря передаваемых в функцию аргументов
from datetime import datetime


def logging_decorator(logger):
	def decor(f):
		@wraps(f)
		def wrapper(*args, **kwargs):
			r = f(*args, **kwargs)
			logger.append(
				{
					'name':wrapper.__name__,
					'arguments': getcallargs(f, *args, **kwargs),
					'call_time': datetime.now(),
					'result': r
				}
			)
			return r
		return wrapper
	return decor

logger = [] # этот массив будет хранить наш "лог"
  
@logging_decorator(logger) # в аргументы фабрики декораторов подается логгер
def test_simple(a, b=2):
	return 127
  
test_simple(1) # при вызове функции в список logger должен добавиться словарь с
# информацией о вызове функции
  
print(logger)
# -> [{'name': 'test_simple', 'arguments': {'a': 1, 'b': 2}, 'call_time': datetime.datetime(2025, 2, 1, 1, 1, 47, 56658), 'result': 127}]
```
---

### Generators

> Ленивые вычисления

```python
a = (i for i in range(2, 8)) # генератор
print(next(a)) # -> 2
print(a.__next__()) # -> 3
for i in a:
	print(i) # -> 4 5 6 7
```

> Генератор через `yield`

>[!Info]
> Функция с `yield` возвращает объект-генератор. С каждым новым `.__next__()` функция продолжает свое выполнение до встречи со следующим `yield`.
```python
def func(start, end):
	while start < end: 
		yield start
		start += 1
	
a = func(1, 4)
for i in a:
	print(i) # -> 1 2 3
```

### Сопрограммы

>[!Info]
>В отличие от генераторов, не возвращают, а принимают значения в (yield)
```python
def counter(num: int=9):
	while True:
		tmp = (yield)
		if num == tmp % 3: 
			print(tmp)
  
def main():
	Counter = counter(2)
	Counter.__next__() # !!!Перемещение до первой инструкции yield
	for i in range(100):
		input()
		Counter.send(i)
	Counter.close()
```

> Сложный пример работы генератора-сопрограммы `line = (yield result)`:
^965795

```python
def decor_for_gener(func): # декоратор нужен, чтобы не вызывать next()
	def call(*args, **kwargs):
		tmp = func(*args, **kwargs)
		next(tmp) # ! Перемещение до первой инструкции yield
		return tmp
	return call
  
@decor_for_gener
def gener(mes:str):
	print('Im ready for splitting.', mes)
	result = None
	while True:
		line = (yield result) # при вызове .send(smth) вызывается .__next__(), а для line присваивается smth
		result = line.split()
  
def main():
	runner = gener('message')
	print(runner.send("my big message !"))
	runner.close()
  
main()
```

> `yiled from` - позволяет делегировать вычисления одного генератора на другие. Также можно использовать для рекурсии или c итерируемыми объектами:

```python
def sub_generator():
    yield 1
    yield 2

def main_generator():
    yield from sub_generator()  # Делегируем выполнение подгенератору
    yield from [3, 4] # т.е. не нужно писать цикл, это просто упрощает код
    yield 5

for num in main_generator():
    print(num, end=' ')
```
output:
```
1 2 3 4 5
```

---

### Datetime

```python
import datetime
```

> date - класс даты 

```python
yest = datetime.date(2025, 1, 30)
today = datetime.date.today() # -> 2025-01-31
print(today.day, today.month, today.year) # -> 31 01 2025
today.weekday() # пн=0 вт=1 ... -> 4
```

> time - класс времени

```python
t = datetime.time() # -> 00:00:00
t = datetime.time(16, 25) # -> 16:25:00

```

> datetime - класс даты со временем

```python
dt = datetime.datetime(2017, 5, 10, 4, 30)
dt_now = datetime.datetime.now() # -> 2025-01-31 22:16:24.138565
```

> timedelta - класс периода времени. Описаны операции + и -

```python
td = datetime.timedelta(days=2, hours=19)
print(datetime.date.today() + td) # -> 2025-02-02
print(datetime.datetime.now() + td) # -> 2025-02-03 17:24:56.997619
print(td + td) # -> 5 days, 14:00:00
```

> `.strptime(str, format)` and `.strftime(time, format)`

> [!Info] strptime and strftime
> .strptime(str, format) преобразует строку в datetime по шаблону format.
> .strftime(time, format) преобразует datetime в строку по шаблону format.
```python
print(datetime.datetime.strptime("22/05/2017 12:30", "%d/%m/%Y %H:%M"))
# -> 2017-05-22 12:30:00
print(datetime.datetime.strftime(datetime.datetime.now(), "%d.%m.%y %H:%M:%S"))
# -> 31.01.25 22:28:28
```

### Time

> Модуль `time`

```python
import time

print(time.gmtime(0)) # Время начала эпохи: 1970-01-01

print(time.ctime(seconds=0)) # секунды от эпохи в дату-> Thu Jan 1 03:00:00 1970
print(time.ctime()) # -> Fri Jan 31 22:42:02 2025

print(time.strftime("%Y-%m-%d %H.%M.%S", time.localtime())) # -> 2025-01-31 22.47.38

print(time.time()) # время в секундах от эпохи -> 1738352935.4128742
```
---
### Regular Expressions

```python
import re
```

> Составные части регулярного выражения

| Шаблон | Описание               |
| ------ | :--------------------- |
| n*     | 0 или более символов n |
| n+     | 1 или более символов n |
| n?     | 0 или 1 символ n       |
| n{2}   | ровно 2 символа n      |
| n{2,}  | 2 или более символа n  |
| n{2,4} | 2, 3 или 4 символа n   |

| Шаблон     | Описание                                                                                                    |
| ---------- | :---------------------------------------------------------------------------------------------------------- |
| .          | Любой символ, кроме новой строки (`\n`)                                                                     |
| (A\|B)     | A или B                                                                                                     |
| (...)      | Группа символов (каждой из них соответствует порядковый номер, на который можно ссылаться - \1, \2, ... \n) |
| [ABC]      | A, B или C                                                                                                  |
| [**^**ABC] | Не(A, B или C)                                                                                              |
| [A-Z]      | Символы от A до Z, верхний регистр                                                                          |
| [0-9]      | Цифры от 0 до 9                                                                                             |
| [A-Z0-9]   | Символы от A до Z ицифры от 0 до 9                                                                          |
| \n         | ссылка на группу                                                                                            |

| Шаблон | Описание      |
| ------ | :------------ |
| ^      | Начало строки |
| $      | Конец строки  |

| Шаблон | Описание                                          |
| ------ | :------------------------------------------------ |
| \w     | Word (a-z, A-Z, 0-9, включая `_`))                |
| \W     | Non-word                                          |
| \d     | Digit (0-9)                                       |
| \D     | Non-digit                                         |
| \s     | Пробел (включая табуляцию и прочие виды отступов) |
| \S     | Не пробел                                         |
| \n     | Новая строка                                      |

> Методы работы с RegEx

```python
res = re.match(pattern=r'AV\d', string='AV1 AVVo') # поиск шаблона в начале
print(res) # -> <re.Match object; span=(0, 3), match='AV1'>
print(res.group()) # -> 'AV1'
```

```python
# group() или group(0) выведет весь паттерн
# group(n) выведет скобочную группу, если она есть
res = re.match(r'AV(.{3})', 'AV Analytics Vidhya AV') # В первую группу попадут любые 3 символа после AV
print(res.group()) # -> 'AV An'
print(res.group(1)) # -> ' An'
```

```python
res = re.search(r'(Analytics)(\d)', 'AV Analytics1 Vidhya AV')
print(res.group()) # -> 'Analytics1'
print(res[0]) # -> 'Analytics1'
print(res[1]) # -> 'Analytics'
print(res[2]) # -> '1'
```

```python
res = re.findall(r'\w+', 'AV Analytics Vidhya AV')
print(res) # -> ['AV', 'Analytics', 'Vidhya', 'AV']
```

```python
result = re.split(r'\d', 'a1b2c3d45e6')
print(result) # -> ['a', 'b', 'c', 'd', '', 'e', '']
```

```python
result = re.sub(pattern=r'India', repl='the World', string='brain of India')
print(result) # -> 'brain of the World'
```
---

### Classes

> Методы и атрибуты класса

```python
class Foo(object):
	value = 10

	def __new__(cls): # Конструктор
		pass

	def __del__(self): # Деструктор
		pass

	def instance_method(self): # self - ссылка на экземляр класса Foo
		pass
	
	@classmethod
	def class_method(cls): # cls - ссылка на сам класс Foo
		pass

	def method_with_no_args(): # нельзя вызвать через экземпляр класса
		pass

	@staticmethod
	def static_method(): # можно вызвать через экземпляр класса
		pass
```

> Перегрузка операторов

```python
class Kop:
	def __init__(self, num):
		self.num = num
	
	def __repr__(self): return f"Kop({self.num})"
	def __str__(self): return "-> %s <-" % (self.num)
	  
	def __add__(self, val): return Kop(self.num + val)
	# right add => instance after value
	def __radd__(self, val): return self.__add__(val)

	def __mul__(self, val): return self.num * val
	# right mul => instance after value
	def __rmul__(self, val): return self.__mul__(val)

	def __truediv__(self, val): return self.num / val
	  
	def __sub__(self, val): return Kop(self.num - val)
	# right sub => instance after value
	def __rsub__(self, val): return Kop(val - self.num)

	def __eq__(self, other): return other.num == self.num
	
	def __lt__(self, other): return self.num < other.num

	def __or__(self, val): return self.num or val
```

>  Магический метод `.__call__()`

```python
class Kop:
	def __call__(self):
		print('call method react')

kop = Kop()
kop() # -> 	'call method react'
```

> Магические методы `.__enter__(self)` и `.__exit__(self)` 

см. ниже [[Python#^095ae6]]

> Магические методы `.__eq__(self, other)` и `.__hash__(self)`. Нужны для определения поведения хэширования.

```python
class A:
	def __init__(self, l, w):
		self.length = l
		self.word = w

	def __eq__(self, other):
		return other.length == self.length and  other.word == self.word

	def __hash__(self):
		return hash((self.length, self.word))

a = A(10, 'ada')
print(hash(a)) # -> 3889907546743416439

'''---Но даже если не определить функцию хэширования python разберется---'''
class A:
	def __init__(self, l, w):
		self.length = l
		self.word = w

a = A(10, 'ada')
print(hash(a)) # -> 8764103095174
```
#### Property, setter, getter и deleter

```python
class Foo(object):
	def __init__(self, name:str):
		self.__name = name
	
	@property # геттер
	def name(self): return self.__name

	@name.setter # сеттер
	def name(self, new_name:str): self.__name = new_name

	@name.deleter # делиттер
	def name(self): raise(Exception('Impossible to delete attribute.'))
  
def main():
	f = Foo('Alex')
	print(f.name)
	f.name = 'Alexey'
	print(f.name)
	try:
		del f.name
	except Exception as e:
		print(e)
	finally:
		print("(END)")
main()
```
output:
```
Alex 
Alexey 
Impossible to delete attribute. 
(END)
```
---
### Dataclasses

```python
from dataclasses import dataclass

@dataclass
class Book:
	title: str
	author: str
	desc: str = None
  
	def __post_init__(self): # вызывается после инициализации title и author
		self.desc = self.desc or "`%s` by %s" % (self.title, self.author)

Book("Название", "Автор").desc
```
### Контекстный менеджер `with`
^095ae6

```python
import sqlite3

class DBConnection:
    def __init__(self, db_name):
        self.db_name = db_name
    
    def __enter__(self):
        self.conn = sqlite3.connect(self.db_name)
        return self.conn
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()

with DBConnection('MyDBase.db') as conn: 
	# принимает в conn результат работы метода .__enter__(self)
	conn.execute('SELECT * FROM table_name')
	result = conn.fetchall()
	print(result)
	# вызывает .__exit__(self)
```

> Альтернатива к with

```python
try:
	conn = sqlite.connect('MyDBase.db')
	conn.execute('SELECT * FROM table_name')
	result = conn.fetchall()
	print(result)
finally:
	conn.close()
```

> Асинхронная реализация
   (Про асинхронное программирование см. ниже __ )

```python
import asyncio

class AsyncTimer:
	async def __aenter__(self):
		self.start_time = await asyncio.get_event_loop().time()
		return self        
	
	async def __aexit__(self, exc_type, exc_val, exc_tb):     
		elapsed_time = asyncio.get_event_loop().time() - self.start_time   
		await asyncio.sleep(5)
	    print(f"Elapsed time: {elapsed_time} seconds")

# Пример использования асинхронного контекстного менеджера
async def example():   
	async with AsyncTimer() as timer:   
		# Асинхронный блок кода      
		await asyncio.sleep(2)# Запуск асинхронной функции

asyncio.run(example())
```
---

### Descriptor

> [!Info]
> Дескриптор - это класс для описания поведения атрибутов другого класса.
```python
class TypedProperty(object):
	def __init__(self, name_, type_, value_=None):
		self.name = "_" + name_
		self.type = type_
		self.value = value_ if value_ else type()
	  
	def __get__(self,instance,cls):
		return getattr(instance,self.name,self.value)
	
	def __set__(self,instance,value):
		if not isinstance(value,self.type):
			raise TypeError("“Значение должно быть типа %s”" % self.type)
		setattr(instance,self.name,value)
	  
	def __delete__(self,instance):
		raise AttributeError("“Невозможно удалить атрибут”")

class Foo(object):
	name = TypedProperty("name",str) # инициализация атрибутов
	num = TypedProperty("num",int,42)

	def __init__(self, name, num) -> None:
		self.name = name
		self.num = num
	  
	def __str__(self) -> str:
		return self.name + self.num.__str__()
  
	def main():
	f = Foo('alex', 12)
	d = Foo("andrey", 13)
	print(f, d) # -> alex12 andrey13
	print(f._name, f.name) # -> alex alex
main()
```
---

### Logging 

> Пример:
```python
# Подготовка логгера

import logging
  
logging.basicConfig(
	filename='logging.log', # файл записи
	filemode='w', # режим записи
	level=logging.INFO, # Уровень важности
	format="%(levelname)-9s %(asctime)-30s LineNum:%(lineno)-7d %(message)s"
)
log = logging.getLogger('main')
  
class FilterFunc(logging.Filter):

	def __init__(self,name):
		self.funcName = name
	
	def filter(self, record):
		if self.funcName == record.funcName: return False
		else: return True
  
log.addFilter(FilterFunc('func')) # Игнорировать все сообщения из функции func()



# Ниже - исполняемый код
  
def func(num:int):
	def call():
		log.error("We have big troubles in call function") # will react, despite it laced in func()
	if num == 10: call()
	else: log.info(f"Don't worry, num equal {num}")
  
 
def main():
	func(10)
	func(12)
	log.debug("debug msg")
	log.info("info msg")
	log.warning("warning msg")
	log.error("error msg")
	log.critical("critical msg")
```
Если хочется писать в разные файлы, то это сложно) там какие-то хэндлеры и сложности всякие. Как оказалось, хэндлеры это обычное дело, хоть и не очень удобное, когда надо будет - разберешься

---

### Asincio

> [!Info] Потоки и процессы
 ЯДРО умеет выделять НЕзависимые друг от друга ПРОЦЕССЫ.
ПРОЦЕССЫ могут выделять внутри себя ПОТОКИ, которые в свою очередь могут обращаться к одним данным из ПРОЦЕССА.
Несколько ПОТОКОВ могут увеличивать число ссылок на данные, откуда появляются проблемы со сборщиком мусора. 
Т.о. сборщик мусора глобальный и многопоточность в Python все равно реализует последовательное выполнение при работе с ОБЩИМИ данными.

> Threading - (thread = поток)

```python
import threading
  
def delay(delay, message):
	print(f"{message} received")
	time.sleep(delay)
	print(f"Printing {message}")
	return message
	  
def main():
	print("Main started")
	threads = [
		threading.Thread(target=delay, args=(3, "THREE SECONDS DELAY")),
		threading.Thread(target=delay, args=(2, "TWO SECONDS DELAY")),
	]
	for thread in threads:
		thread.start()
	print("Главный поток тоже продолжает исполняться")

	for thread in threads:
		thread.join() # Ждем пока завершаться вторичные потоки, прежде чем закрывать главный!
	print("Main Ended")

main()
```
output:
```
Main started
THREE SECONDS DELAY received
TWO SECONDS DELAY received
Главный поток тоже продолжает исполняться
Printing TWO SECONDS DELAY
Printing THREE SECONDS DELAY
Main Ended
```


> Futures
> Футуры - не закрывают поток после выполнения, а оставляют открытым для других задач (экономим на открытии и закрытии потоков)

```python
import concurrent.futures as cf
  
def main():
	with cf.ThreadPoolExecutor(max_workers=2) as executor:
		future_to_mapping = {
			executor.submit(delay, 3, "THREE SECONDS DELAY"): "3 secs",
			executor.submit(delay, 2, "TWO SECONDS DELAY"): "2 secs",
			executor.submit(delay, 4, "FOUR SECONDS DELAY"): "4 secs",
		}
		for future in cf.as_completed(future_to_mapping):
			logging.info(f"{future.result()} Done")

main()
```
output:
```
THREE SECONDS DELAY received
TWO SECONDS DELAY received
Printing TWO SECONDS DELAY
FOUR SECONDS DELAY received
TWO SECONDS DELAY Done
Printing THREE SECONDS DELAY
THREE SECONDS DELAY Done
Printing FOUR SECONDS DELAY
FOUR SECONDS DELAY Done
```


> `threading.Lock()` - При работе с одними данными параллельно во избежании коализий можно разблокировать данные только для одного потока обернув кусок кода, где мы работаем с общей переменной:
> ПС: В asyncio  нет такой проблемы

```python
class Value:
	def __init__(self, v):
		self.value = v
	
	def update_value():
		...
		with threading.Lock(): 
			tmp = self.value + 1
			print('Updating Value')
			self.value = tmp
			print('Done')
		...
```


> ASYNCIO

> [!tip] Asyncio
> Три основные сущности: 
> 1. __Корутины__ - специальные функции, от которых ожидается передача управления в цикл событий, из которого они были запущены
>2. __Цикл событий__ - корутина, которая управляет вызовом остальных корутин.
>3. __Awaitable объект__ - объект, который умеет "ожидать" результат.

Проще говоря: корутины - исполняемые параллельно функции, а цикл событий - это цикл управления корутинами.

> Корутина через сопрограмму:

```python
import time

def cor_func():
	counter = 0
	while True:
		counter += 1
		lt = (yield)
		time.sleep(2) # полностью блокирует поток
		print('%5i) %s.' % (counter, lt) )
  
corutine = cor_func()
corutine.__next__()
corutine.send('my first message')
corutine.send('my second message')
```
output: 
```
     1) my first message.
     2) my second message.
```

> Корутина через `async def`:

```python
import asyncio 

counter = 0

async def corutine(lt):
	global counter
	counter += 1
	await asyncio.sleep(2)
	print(f'{counter:>5}) {lt}.')

async def main():
	task_1 = asyncio.create_task(corutine('my first message'))
	task_2 = asyncio.create_task(corutine('my second message'))

	print(f'We have only {len(asyncio.all_tasks())}') # 3 in jupyter

	await task_1
	await task_2

# asyncio.run(main()) # if event loop is not created
await main() # in jupyter, because loop has created by default
```
output will take 2.0s

> asyncio.gather() - запустить список корутин 

```python
async def main():
	tasks = asyncio.gather(*[corutine(f'mes {i}') for i in range(5)])
	await tasks	
```


> [!Info] asyncio и блокировка ресурсов
> asyncio при исполнении забирает все ресурсы ПРОЦЕССА. Тогда если в исполняемой корутине будет `time.sleep(n)`, то новые корутины не будут создаваться и вызываться, а ранее вызванные все-таки доисполнятся.
ПС: темка мутная какая-то 

> Асинхронный [генератор | итератор]**.** **[async for | .__aiter__() и .__anext__()) ]**

Пример с `async for`:
```python
import asyncio
  
 
async def mygen(u: int = 10):
	i = 0
	while i < u:
		yield i
		i += 1
		await asyncio.sleep(1)
  
async def main():
	async for i in mygen(5):
		print(i)
	print("after async for")
  
asyncio.gather(main(), return_exceptions=True)
```
output will take 5.0s:
```
0
1
2
3
4
after async for
```
---

### Smth in functools and itertools

> `filter(function, iterable)` - выборка из iterable. `function -> bool`

```python
arr = [1, 2, 3, 5]
filtered_arr = filter(lambda x: x%2==1, arr)
# -> [1, 3, 5]

arr = [0, 1, False, True, "", "hello", None, [], [1, 2, 3]]
true_arr = filter(None, arr)
# -> [1, True, 'hello', [1, 2, 3]]
```

> `functools.reduce(function, iterable, initializer=None)` - применяет функцию `function` каждого следующего элемента `iterable` к предыдущему результату работы function, начиная с первого элемента `iterable` или с `initializer`.

```python
from functools import reduce # -> с англ. "уменьшить"

arr = [1, 2, 3]
res1 = reduce(lambda x, y: x + y, arr) # -> 6
res2 = reduce(lambda x, y: x + y, arr, 1) # -> 7
```

> `itertools.chain(*iterables)` - Объединение итерируемых объектов

```python
from itertools import chain

arr = [[1, 2, 3], [4, 5, 6], [[1, 2]]]
chain(*arr) # -> [1, 2, 3, 4, 5, 6, [1, 2]]
```

> `itertools.groupby(iterable, key=None)` - группировка элементов на основе ключа заранее отсортированного`iterable`. ` -> итератор из кортежей (ключ, итератор по группе эл-ов)`. 

```python
from itertools import groupby

arr = ['apple', 'banana', 'cherry', 'date', 'strawberry', 'fig']
arr.sort(key=len)
res = groupby(arr, key=len)

[(i[0], list(i[1])) for i in res] # -> 
# [ (3, ['fig']),
#   (4, ['date']),
#   (5, ['apple']),
#   (6, ['banana', 'cherry']),
#   (10, ['strawberry']) ]
```