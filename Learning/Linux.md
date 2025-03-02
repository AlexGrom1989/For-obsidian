```powershell
Базовые консольные команды для работы с папками и файлами{

pwd => текущая директория (pwd Documents)
ls => список вложеных файлов и папок (ls -r -p) (ls -l) (ls -a #в том числе скрытые файлы)
cd => переход между папками (cd ~/Documents/)
touch => создание файлов любого расширения (touch main.cpp main.py)
nano => редактирование файлов (nano main.cpp)
mkdir => создание катологов (mkdir MyDirectory)
cp => копирование и вставка файла (cp main.cpp MyDirectory/my_main.cpp)
mv => перемещение папки/файла (mv MyDirectory/my_main.cpp ~/Documents)
rm => удалить файл(ы) (rm main.cpp) (rm MyDirectory/*.txt) (rm MyDirectory/test.*) (rm *)
rm -rf => удалить папку (rm -rf MyDerictory)
tree => дерево файлов (tree)
uname -a => полное название операционной системы
}

Работа с админкой и доступом{

sudo .... => выполнение от лица суперпользователя/администратора (sudo nano test.txt)
sudo su => переход в режим админа
su .... => смена юзера (su alex)
sudo chown => смена владельца, но не доступ (sudo chown root:alex test.txt)
sudo chmod => смена модификации/доступа (sudo chmod --help) (sudo chmod 664 test.txt #доступ -rw-rw-r-- #напиши ls -l до и после, чтобы посмотреть что изменилось #цифры 0-7 определяют доступ)
Для работы с папками в вышенаписаннах командах надо добавить -R
}

Текстовый редактор gedit и VIM и пакетный менеджер apt{
gedit => открытие текстового редактора linux (аналог блокнота)
pico => то же, что и nano

apt => типо help для apt (apt)
apt install => установка покетов (apt install vim)
vim => запуск приложения vim (vim)
apt policy => проверка приложения на наличие (apt policy firefox)
apt search => поиск в репозитории (apt search firefox)

echo tar00 > bar00.txt => перезаписать в файл (если файл не существует, то создать)
echo tar00 >> bar00.txt => запись в файл
}

Поиск{

find --help => почитать про find #Лучше использовать с sudo
find путь -type f фильтр => поиск файла (find . -type f -perm 0664) (find snap/ -type f -not -name "fil*.txt")
find путь -type d фильтр => поиск директории/каталога/папки (sudo find snap/ -type d -name "discord")

cat => прочесть файл (cat main.cpp)
grep => найти текст в файле(ах) (grep "cout" main.cpp my_main.cpp test.txt)

Мультипоиск (-exec):
find . -type f -iname "*.cpp" -exec grep -i -n "cou*" {} + => выполнится поиск find, затем -exec .... {} + установит границы выполнения следующей команды и выполнит ее
}

Отсортированный текст из файла{

sort => вывод в консоль отсортированное содержимое файла (sort file.txt)
sort ... > ... => сохранение отсортированного содержимого в новый файл (sort file.txt > my_file.txt)
}

Работа с памятью{

top => аналог диспетчера задач (top) #чтобы выйти, надо нажать q
free => информация о памяти компьютера (free -h)
ps => список процессов (ps -aux | grep ale)
}

Сжатие данных{

zip -r => сжатие в .zip архив (zip -r folder.zip MyDirectory)
unzip => разархивация (unzip folder.zip)

tar cf => создание архива.tar(не сжатого) (tar cf my_folder.tar MyDirectory)
tar tf => просмотреть содержимое архива.tar (tar tf my_folder.tar)
tar xf => разархивация (tar xf my_folder.tar)

xz => сжатие .tar архивов (xz my_folder.tar)
unxz => разархивация до .tar (unxz my_folder.tar.xz)

gzip => сжатие .tar архивов (gzip my_folder.tar)
tar czf => ---||--- (tar czf my_folder.gz MyDirectory) #без .tar
gunzip => разархивация до .tar (gunzip my_folder.tar.gz)
}

Скачивание сторонних ресурсов{

wget => скачивание по ссылке (wget https://itproger.com/img/tests/python.svg)

sudo dpkg -i => запуск имеющегося установщика (sudo dpkg -i google-chrome-stable_curent_awd64.dep)
}

Работа с сайтами и серверами{

ping => системная информация о сатйе (ping www.google.com) (ping -c 10 www.google.com)
host => запрос на ip информацию (host www.google.com)
sudo ufw allow 22 => освободить порт 22
ssh => защищенный протокол подключения (ssh 5bc59b5f5bc5.vps.myjino.ru)
w => подключенные пользователи (w)
}

Bash Shell files{

nano .....sh => создание .bash файла (nano bash_script.sh) #содержимое bash выполняется в консоли при необходиомости
bash => запуск .bash файла (bash bash_script.sh)

Содержимое .bash файла{
	echo => вывод строки (echo "Hello World!")
	my_number=50927 => переменная число
	my_string="olleH dlroW!" => переменная строка
	my_command=`tree -f` => переменная команда
	echo "$my_string is my phone number! $my_command" => вывод с вызовом переменных
	input_num_1=$1 => чтение передаваемых при вызове параметров (например передали "bash b_file.sh 10 12", тогда $0 - это b_file.sh, $1 - это 10, $2 - это 12)
	input_num_2=$2
	res=$((input_num1+input_num_2)) => сложение чисел
	read -p "Input something" input_num_1 => чтение в переменную input_num_1

	#Условный оператор if 
	if [ 5 == "$input_num_1" ]; then 
		echo "Все правильно, кайфуй!"
	elif [ "$input_num_1" == 1 ]; then
		echo "$input_num_1 = one"
	else
		tree
		echo "ljnjcsjncj"
	fi
	
	#Циклы
	for (( i=1; i<=10; i++ )); do
		echo "$i"
	done
	
	for i in {2..5}; do
		echo "$i";
		sleep 1;
	done;
	
	i=0
	while [ $i < 10 ]; do
		echo "$i"
		i=$(($i+1))
	done

	#Функции
	divide() {
		echo "Params: $1 $2"
		if [ "$2" == 0 ]; then 
			echo "Error"
		else
			res=$(($1/$2))
			echo "Res: $res"
		fi
	}
	divide 15 3
	divide 10 0
	}
}
```
