### BeautifulSoup

>Парсинг html-страниц. Основной принцип - выборка нужных блоков и ВОЗВРАЩЕНИЕ их в качестве объектов. Это проявляется в том, что можно несколько раз подряд применять одни и те же методы на результаты работы других методов.

```python
import re
from bs4 import BeautifulSoup

soup = BeautifulSoup(src:=open('file.html'), "xml")

# get text from all 'p'-blocks
text_arr = [i.text for i in soup.find_all('p')]

# get link from div block
a_block = soup.find('div', {'class': 'user__name', 'id': 'user__id'}).find('a', class_='a__class')
href = a_block.get('href')
href = a_block['href'] # same way

# get parent
parent = soup.find('div', class_='user__name').find_parent() # first parent in one higher level
parent_n = soup.find('div', class_='user__name').find_parent('span') # first span-parent

# find next element (not necessary in same level)
nextt = soup.find(class_='user__name').find_next()

# find next or previos block in same level
next_sib = soup.find({'class': 'user__name'}).find_next_sibling()
prev_sib = soup.find({'class': 'user__name'}).find_previous_sibling()

# find block by text
text = soup.find('a', text=r'Продукт номер \d+') # catch only full text or regular expression


# get site page
import requests
url = 'https://vk.com'
text = requests.get(url).text
with open('file.html', 'w') as file:
    file.write(text)
```

### SQLAlchemyORM

> Пример целостной программы 

Перед выполнением кода с использованием SQLAlchemy и PostgreSQL необходимо выполнить несколько важных шагов:

1. Установить PostgreSQL
```bash
sudo apt install postgresql postgresql-contrib
```
 
 2. Создать базу данных и пользователя
```bash
sudo -iu postgres
createdb your_db_name
psql your_db_name
```
```sql
CREATE USER your_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE your_db_name TO your_user;
```
 
 3. Установить необходимые Python-пакеты
```bash
pip install sqlalchemy psycopg2-binary
```

4. Создать файл .env (опционально, но рекомендуется)
```bash
pip install python-dotenv
```
```ini.env
# .env file
DATABASE_URL=postgresql://your_user:your_password@localhost:5432/your_db_name
```
```python
from dotenv import load_dotenv
import os

load_dotenv()  # По умолчанию ищет файл .env в текущей директории

DATABASE_URL = os.getenv("DATABASE_URL")
```

> После подготовительной части начинается сама работа с SQLAlchemyORM
```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey, func
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship, and_
from sqlalchemy.exc import SQLAlchemyError

# Определяем базовый класс для моделей
Base = declarative_base()

# Определяем модели (сущности)
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True, nullable=False)
    role = Column(String(20), nullable=False)
    email = Column(String(100))
    
    articles = relationship("Article", back_populates="author")
    
    def __repr__(self):
        return f"<User(id={self.id}, username='{self.username}', role='{self.role}')>"

class Article(Base):
    __tablename__ = 'articles'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(100), nullable=False)
    content = Column(String)
    user_id = Column(Integer, ForeignKey('users.id'))
    
    author = relationship("User", back_populates="articles")
    
    def __repr__(self):
        return f"<Article(id={self.id}, title='{self.title}')>"

# Настройка подключения к PostgreSQL. 
# port(5432) можно посмотреть в psql через "SHOW port;"
# url можно вытащить из .env (см. выше)
DATABASE_URL = "postgresql://postgres:postgres@localhost:5432/sqlalchemy_demo"
engine = create_engine(DATABASE_URL) # служебная информация о движке(СУБД) на котором будет исполняться работа. В нашем случае на PostgreSQL, он это понимает из DATABASE_URL

# Создаем таблицы в базе данных
def create_tables():
    try:
        Base.metadata.create_all(engine) # просто создание таблиц User и Articles (наследников Base). Аналог CREATE TABLE ...
        print("Таблицы успешно созданы")
    except SQLAlchemyError as e:
        print(f"Ошибка при создании таблиц: {e}")

# Создаем сессию для работы с базой данных
Session = sessionmaker(bind=engine) # Пометка, что все дальнейшие сессии будут на движке engine по умолчанию.

def main():
    # Создаем таблицы
    create_tables()
    
    # Создаем пользователей
    with Session() as session:
        try:
            # Проверяем, есть ли уже пользователи
            if not session.query(User).filter(User.username.in_(["default_user", "admin"])).count():
                # Создаем default_user
                default_user = User(
                    username="default_user",
                    role="user",
                    email="default@example.com"
                )
                
                # Создаем admin
                admin = User(
                    username="admin",
                    role="admin",
                    email="admin@example.com"
                )
                
                session.add_all([default_user, admin])
                session.commit()
                print("Пользователи успешно созданы")
            else:
                print("Пользователи уже существуют")
        except SQLAlchemyError as e:
            session.rollback()
            print(f"Ошибка при создании пользователей: {e}")
    
    # Работа с default_user
    with Session() as session:
        try:
            # INSERT для default_user (добавляем статью)
            default_user = session.query(User).filter_by(username="default_user").first()
            if default_user:
                new_article = Article(
                    title="Первая статья default_user",
                    content="Это содержание первой статьи.",
                    user_id=default_user.id
                )
                session.add(new_article)
                session.commit()
                print("Статья default_user успешно добавлена")
            
            # SELECT для default_user (с where, order by, like)
            print("\nSELECT для default_user:")
            articles = session.query(Article).filter(
                and_(
                    Article.user_id == default_user.id,
                    Article.title.like("%статья%")
                )
            ).order_by(Article.title).all()
            print(f"Найдено статей: {len(articles)}")
            for article in articles:
                print(article)
            
            # UPDATE для default_user
            if articles:
                articles[0].title = "Обновленный заголовок первой статьи"
                session.commit()
                print("Статья default_user успешно обновлена")
        
        except SQLAlchemyError as e:
            session.rollback()
            print(f"Ошибка при работе с default_user: {e}")
    
    # Работа с admin
    with Session() as session:
        try:
            # INSERT для admin (добавляем статью)
            admin = session.query(User).filter_by(username="admin").first()
            if admin:
                new_article = Article(
                    title="Административная статья",
                    content="Важная информация для админов.",
                    user_id=admin.id
                )
                session.add(new_article)
                session.commit()
                print("Статья admin успешно добавлена")
            
            # SELECT для admin (с group by, having)
            print("\nSELECT для admin (группировка по ролям):")
            role_stats = session.query(
                User.role,
                func.count(User.id).label("user_count")
            ).group_by(User.role).having(func.count(User.id) > 0).all()
            
            print("Статистика по ролям:")
            for role, count in role_stats:
                print(f"{role}: {count} пользователей")
            
            # UPDATE для admin
            if admin:
                admin.email = "new_admin@example.com"
                session.commit()
                print("Email admin успешно обновлен")
        
        except SQLAlchemyError as e:
            session.rollback()
            print(f"Ошибка при работе с admin: {e}")
    
    # Демонстрация связи между таблицами
    with Session() as session:
        try:
            print("\nДемонстрация связи между таблицами:")
            users_with_articles = session.query(User).join(Article).all()
            for user in users_with_articles:
                print(f"Пользователь: {user.username}")
                print(f"Количество статей: {len(user.articles)}")
                for article in user.articles:
                    print(f"  - {article.title}")
        except SQLAlchemyError as e:
            print(f"Ошибка при демонстрации связи: {e}")

if __name__ == "__main__":
    main()
```

### Numpy

#numpy^1
> [!Info] numpy
> Библиотека для работы с N-мерными массивами одного типа данных. Основной Класс - `np.ndarray`

```python
import numpy as np
np.random.seed(42)
```

> `ndarray` и `empty`. Основной класс `NDArray`. Сам по себе это макет для хранения  n-мерных массивов. 

```python
np.ndarray((1, 2), dtype=np.float16) # -=> array([[1.53e-05, 0.00e+00]], dtype=float16)

'Заполнение по умолчанию через np.empty(--||--) околонулевыми значениями'
```

> `array`. Обычный массив 

```python
arr = np.array([[1,2], [3,4]], dtype=float) # -=> <class 'numpy.ndarray'>

arr.dtype # -=> dtype('float64')

arr.shape # -=> (2, 2)

arr[1,1] # -=> 4.
```

> `load` и `save`. Загрузка/выгрузка массива из файла

```python
data = np.load('../data/bernoulli.npy')

data = np.random.binomial(n=1, p=p, size=size) # см. Random

np.save('bernoulli.npy', data)
```

> `eye`, `zeros` и `ones`. Единичный\нулевой массив

 ```python
 np.eye(3, 3) # -=> единичная матрица 
 np.ones((2, 3), dtype=int) # -=>  массив из единиц
 np.zeros((2, 3), dtype=bool)
```

> `arange` и `linspace`. Послед-ть и Точки на интервале

```python
np.arange(3, 10, 2) # -=> array([3, 5, 7, 9])   аналог range
np.linspace(3, 10, 2) # -=> array([3., 10.])   две точки на [3, 10]
```

> `append` и `concatenate`. Добавление и Объединение

```python
a = np.array([[1, 2], [3, 4]], float)
b = np.array([[5, 6], [7,8]], float)

np.append(a,b) # Добавление в строку [1., 2., 3., 4., 5., 6., 7., 8.]

np.concatenate((a,b), axis=0) # Объединение [[1., 2.],
						#	         [3., 4.],
						#	         [5., 6.],
						#	         [7., 8.]]
```

> `np.vstack()` и `np.hstack()`.  Объединение (читаемость лучше чем у conatenate)

```python
np.vstack((a, b)) # === np.concatenate((a, b), axis=0)
np.hstack((a, b)) # === np.concatenate((a, b), axis=1)
```

> `reshape` и `transpose`. Изменение формы и Транспанирование

```python
arr = np.arange(6).reshape((2, 3)) # -=> array([[0, 1, 2],
								   #	        [3, 4, 5]])
arr.T # -=> array([[0, 3],
	#		       [1, 4],
	#		       [2, 5]])
```

> `flip`. Перевернуть массив вдоль оси (по умолчанию вдоль всех осей)

```python
mat = np.array([[1, 0, 0], 
			    [0, 2, 0], 
			    [0, 0, 3]])
			  
np.flip(mat)
3 0 0
0 2 0
0 0 1

np.flip(mat, axis=0)          np.flip(mat, axis=1)
0 0 3                         0 0 1
0 2 0                         0 2 0
1 0 0                         3 0 0
```

> `vectorize` и `where`. Аналог `apply` и Аналог фильтра. 

```python
vec_func = np.vectorize(lambda x: x**2 if x>0 else 0, otypes=float)
res = vec_func(arr)

res = np.where(arr>0, arr**2, 0) # -=> результат тот же, но работает быстрее
```

> `[::, ::]` и `[arr > 3 & arr < 6]`. Слайсинг  и маски

```python
arr = np.array([[0, 2, 3], [4, 5, 6]], float)
arr[::, ::2] # -=> array([[0., 3.],
					  #   [4., 6.]])
arr[(arr > 1) & (arr % 2 == 0)] # -=> array([2., 4., 6.])
```

> Математически операции

1. **`np.sum()`** – Сумма элементов.
    
2. **`np.mean()`** – Среднее значение.
    
3. **`np.min() / np.max()`** – Минимум и максимум.
    
4. **`np.std()`** – Стандартное отклонение.
    
5. **`np.var()`** – Дисперсия.
    
6. **`np.abs()`** – Модуль.
    
7. **`np.sin(), np.cos(), np.exp(), np.log()`** – Тригонометрия и экспоненты.
    
8. **`np.dot()` / `@`** – Скалярное и матричное умножение.
	
9. **`np.round()`** - Округление

> Линейная алгебра

10. **`np.linalg.inv()`** – Обратная матрица.
    
11. **`np.linalg.det()`** – Определитель матрицы.
    
12. **`np.linalg.eig()`** – Собственные значения и векторы.

##### `Numpy.Random`

```python
np.random.random((2, 3)) # -=> матрица 2x3 с эл. равн-ого распр. на [0, 1)
np.random.rand(2, 3) # -=> то же самое

np.random.randint(1, 100, size=(10)) # -=> 10 рандомных целых из [1, 100)

np.random.choice(['a', 'b', 'c'], size=(5), p=[0.2, 0.4, 0.4]) # -=> весовая выборка

np.random.shuffle([1, 2, 3, 4]) # -=> перемешивание
```

---
> uniform - Равномерное распр.
```python
np.random.uniform(low=0., high=1., size=(2, 3))
# kop = np.random.uniform(low=0., high=1., size=(10000))
# plt.hist(kop, bins=30, edgecolor='black')
```

![[Pasted image 20250407194409.png]]

---
> normal - Нормальное распр.
```python
kop=np.random.randn(size=(1000)) # -=> стандартное норм. распр.
kop=np.random.normal(loc=0, scale=1, size=(1000)) # -=> с мат. ож. и дисп.
# plt.hist(kop, bins=30, edgecolor='black')
```
![[Pasted image 20250407194832.png]]

---
> binomial - Биномиальное распр.
```python
kop=np.random.binomial(n=100, p=0.2, size=(1000)) # n=число_опытов, p=вер-ть_успеха_в_одном_опыте, size=размер_массива_наблюдений -=> [20 16 ... 21 18], где каждый эл. - число успехов за одно отдельное наблюдение
# plt.hist(kop, bins=30, edgecolor='black')
```
![[Pasted image 20250407195814.png]]

---
> `poisson` - Пуассоновское распр. 
>  $P(X = k) = \frac{\lambda^k e^{-\lambda}}{k!}, \quad k = 0, 1, 2, \ldots$

+ Моделирует редкие события за фиксированный интервал времени.
Условия:
+ События происходят независимо друг от друга.
+ Средняя частота событий (λ, интенсивность) известна и постоянна.
+ Вероятность более одного события за малый промежуток времени стремится к нулю.
```python
kop=np.random.poisson(lam=10, size=(1000))
plt.hist(kop, bins=np.unique(kop), edgecolor='black')
```
![[Pasted image 20250407201445.png]]

---
> `exponential` - Экспоненциальное распр.
>  $f(x) = \lambda e^{-\lambda x}, \quad x \geq 0$

+ Описывает **время между событиями** в пуассоновском процессе.
+ Характеризует **время ожидания** до следующего события.    
Условия:    
+ События происходят **непрерывно и независимо** с постоянной средней скоростью (**λ**).
```python
kop=np.random.exponential(scale=1/10, size=(1000))
# plt.hist(kop, bins=30, edgecolor='black')
```
![[Pasted image 20250407202417.png]]

### Pandas

#pandas^2
> [!Info] pandas  
> Библиотека для работы с табличными данными. Основные классы - `DataFrame` (2D) и `Series` (1D)

```python
import pandas as pd
import numpy as np
```

> `DataFrame` - таблица с индексами и названиями колонок

```python
df = pd.DataFrame({
    'A': [1, 2, 3],
    'B': ['x', 'y', 'z']
}, index=['row1', 'row2', 'row3'])
```

> `Series` - одномерный массив с индексами

```python
s = pd.Series([10, 20, 30], index=['a', 'b', 'c'])
```

Основные атрибуты DF и S:
```python
df.shape    # размерность (строки, столбцы)
df.columns  # список колонок
df.index    # индексы
df.dtypes   # типы данных
df.values   # numpy-массив данных
```

> `read_csv` и `to_csv`. Ввод/вывод

```python
pd.read_csv('file.csv')   # чтение CSV
df.to_csv('output.csv')   # запись в CSV
pd.read_excel('file.xlsx') # чтение Excel
```

> `df[[]]` и `df.loc[]`. Выбор по колонке и позиции

```python
df[['A', 'B']]  # выбор нескольких колонок → DataFrame
df.loc['row1']  # выбор по индексу → Series
df.iloc[0]      # выбор по позиции → Series
```

> `df[df.A>1]` и `query`. Условия и фильтрация

```python
df[df['A'] > 1]              # фильтр по значениям
df.query('A > 1 & B == "y"') # SQL-подобный синтаксис
```

> `select_dtypes`.  Выбрать колонки определенного типа.

```python
df.select_dtypes(include=[int, float]) 
```

> `drop`. Добавление/удаление

```python
df['C'] = [4, 5, 6]     # добавить колонку
df.drop('A', axis=1)     # удалить колонку
```

> `groupby` и `agg`. Группировка и агрегация

```python
df.groupby('B').sum()           # группировка по колонке

df.groupby('department').agg(
	{'income': ['mean', 'max']} # несколько агрегаторов
).round(2)                      

df.groupby('department').agg(
    {'age': lambda x: x.std() / x.mean()} # кастомный агрегатор
)            
```

> `dropna` и `isna`. Работа с пропусками

```python
df.isna()       # проверка на NaN
df.dropna()     # удаление строк с NaN
df.fillna(0)    # заполнение нулями
```

> `merge` и concat. Объединение данных

```python
pd.merge(df1, df2, on='user_id', how='inner')   # SQL-подобное соединение
pd.merge(df1, df2, left_on='user_id', right_on='renter_id', how='left')

pd.concat([df1, df2])          # вертикальное объединение
```

> `sort_values`, `sample` и `describe`. Просмотр данных\статистик

```python
df.head()       # первые 5 строк
df.tail()       # последние 5 строк
df.sample(7)     # рандомные 7 строк
df.describe()   # статистика
df.info()      # общие характеристики данных

df.sort_values('A')  # сортировка
df.performance.sort_values(ascending=False).round(3).head().to_list()
df.sort_values(['age', 'bonus']).bonus.round(2).head().to_list()
```

> `apply` и `agg`. Функция/Агрегация над элементами

```python
df.apply(lambda x: x*2)

df.income.agg(lambda x: x.std()/x.mean())
```

> `value_counts`. Частотный столбец

```python
df['Fruits'].value_counts(normalize=True, ascending=True)

#   яблоко     3
#   банан      2
#   апельсин   1
#   dtype: int64
```

> `pivot_table`. Сводная таблица 

```python
df = pd.DataFrame({
    'Город': ['Москва', 'Москва', 'СПб', 'СПб', 'Москва'],
    'Товар': ['Яблоки', 'Апельсины', 'Яблоки', 'Апельсины', 'Яблоки'],
    'Продажи': [100, 200, 150, 50, 120],
    'Месяц': ['Янв', 'Янв', 'Фев', 'Фев', 'Мар']
})

' Задача: Посчитать средние продажи по городам и товарам. '

pd.pivot_table(df, values='Продажи', index='Город', columns='Товар', aggfunc='mean')
```

Пояснение:
```python
''' Для подсчета первой ячейки выбираются все строки, где index=Город=Москва & columns=Товар=Апельсины, из этих строк выбираются values=Продажи, дл которых считается функция агрегации. '''

#   Товар	Апельсины	Яблоки
#   Город		
#   Москва	200	        110
#   СПб	    50	        150
```

> `crosstab`. Сводная частотная таблица. Хотя можно исп. как pivot_table

```python
pd.crosstab(df['Город'], df['Товар'], margins=True)
```

| Товар   | Апельсины | Бананы | Яблоки | All |
| ------- | --------- | ------ | ------ | --- |
| Москва  | 1         | 0      | 2      | 3   |
| СПб     | 0         | 1      | 1      | 2   |
| **All** | 1         | 1      | 3      | 5   |

```python
pd.crosstab(
    index=df['Город'],
    columns=df['Товар'],
    values=df['Продажи'],
    aggfunc='sum' # !!!!!! т.е. то же самое что pivot_table
)
```

| Товар  | Апельсины | Бананы | Яблоки |
| ------ | --------- | ------ | ------ |
| Москва | 200       | NaN    | 220    |
| СПб    | NaN       | 50     | 150    |

> qcut. Разбиение на квантили

```python
data = [1, 7, 5, 4, 10, 15]

kop = pd.qcut(data, q=4) # Разделили выборку на 4 квантиля
# -=>  [(0.999, 4.25], (0.999, 4.25], (4.25, 6.0], (6.0, 9.25], (9.25, 15.0], (9.25, 15.0]]

kop.unique() # -=>  [(0.999, 4.25], (4.25, 6.0], (6.0, 9.25], (9.25, 15.0]]

kop = pd.qcut(data, q=3, labels=['Низкий', 'Средний', 'Высокий'])
# -=> ['Низкий', 'Средний', 'Низкий', 'Низкий', 'Высокий', 'Высокий']
```

> `plot`. Визуализация

```python
df.plot()                  # линейный график
df.plot.bar()              # столбчатая диаграмма
df.plot.scatter(x='A', y='B')  # точечный график
и тд тп
```

### Scipy

> `scipy.stats`

пример:
```python
from scipy import stats

.pmf P(X=k) для ДИСКРЕТНЫХ сл.в.      # ф. вероятности
.pdf P(a<=X<b) для НЕПРЕРЫВНЫХ сл.в.  # ф. плотности
.cdf P(X<=x) для ЛЮБЫХ сл.в.          # ф. распределения

(1 - stats.binom.cdf(k=59, n=100, p=bernoulli_sample.mean()))

stats.poisson.pmf(k=3, mu=poisson_sample.mean())

(1 - stats.poisson.cdf(k=5, mu=poisson_sample.mean()))
```
