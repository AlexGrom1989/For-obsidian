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