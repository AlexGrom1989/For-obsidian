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