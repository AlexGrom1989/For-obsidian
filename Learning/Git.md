```powershell
Локально { 

git --version => версия гита
git config => глобальная регистрация имени и почты (git config --global user.name alex) (git config --global user.email fordro00@mail.ru)
git init => создание .git папки в рабочей папке(подключение git к проекту) (git init)
git add => добавление файлов\папок (git add .) (git add MyProject)
git status => просмотр индекса
git rm --cached => удаление из индекса (git rm --cached MyProject/main.cpp) (git rm -r --cached MyProject/)
nano .gitignore => создание файла .gitignore, в котором мы перечисляем то, что не должно быть добавлено при вызове add (nano .gitignore #внутри например можно написать *.cpp)
git commit -m "...." => сохранение добавленных ранее файлов\папок с комментарием (git commit -m "First commit")
git commit -a -m "...." => коммит без необходимости git add
git log => просмотр всех коммитов (git log --oneline)

git branch => создание веток (git branch forms_branch) (git branch -a #Все ветки проекта)
git checkout .... => переход на ветку/версию (git checkout forms_branch) (git checkout ab14f3b)
git switch .... => переход на новую ветку (git switch forms_branch)
git checkout -b .... => создание ветки и переход на нее (git checkout -b myProject_branch)
git merge .... => объединение ветки с текущей веткой (git merge myProject_branch)
}

Глобально (связь с репозиторием) {

git remote add origin_1 https://github.com/AlexGrom1989/Git_learning_repository.git
git remote => просмотреть добавленные удаленные репозитории (git remote -v)
git push origin_1 master => отправка ветки master в удаленный репозиторий
git clone https://github.com/AlexGrom1989/Git_learning_repository.git => копирование содержимого из удаленного репозитория к себе
git pull origin_1 master => скачивание изменений в ветке master с удаленного репозитория origin_1
git fetch origin_1 master => извлечение/просмотр удал. репозитория
}

Просмотр изменений {

git log --stat --summary => краткий просмотр
git log HEAD~1 => просмотр состояния проекта на 1 коммит раньше
gitk => визуализация изменений
git log -p HEAD..FETCH_HEAD => просмотр извлеченного репозитория
git log -p master..origin/master => просмотр ветки из удал. репо.
git show .... => просмотр коммита по коду (git show ab14f3b)
git grep "hello" <путь/ветка/сеанс> => поиск по ../../..

}

```
