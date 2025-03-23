### Ввод/Вывод. Числа и массивы

>`malloc(size)` - выделить указателю `ptr_1` память размера `size` 
>`realloc(ptr_1, size * n)` - увеличить для `ptr_1` объем выделенной памяти до `size * n`
> 
>`calloc(n, size)` - выделить указателю `ptr_2` память размера `size * n`

```cpp
#include <iostream>

#include <stdlib.h> // стандартная библиотека
#include <stdio.h> // стандартный ввод/вывод

int main(void){
    // Выделение памяти для массива из n элементов. 3 Способа

    // 1) C++
    int n;
    std::cin >> n;
    int a[n];
    std::cout << a << ' ' << a[1] << std::endl; // -> 0x55555556b6c0, 32767


    // 2.1) C
    int n;
    scanf("%d", &n); // Ввод
    int* a = (int*)calloc(n, sizeof(int)); // Выделение памяти размера n * sizeof(int)
    printf("%p, %i\n", a, a[1]); // Вывод -> 0x55555556b6c0, 0

    // 2.2) C
    int n;
    scanf("%i", &n);
    int* a = (int*)malloc(sizeof(int)); // Указатель на память размера int
    a = static_cast<int *>(realloc(a, sizeof(int)*(n))); // Увеличение выделенной памяти для уже имеющегося указателя a
    printf("%p, %d\n", a, a[1]); // -> 0x55555556b6c0, 0

    return 0;
}
```


> Переменные стандартных типов это объекты => указывают на область памяти, проще говоря - ведут себя как указатели. 

Пример:
```cpp
int main(){
    int n = 9; // для объекта n выделилась память, в нее записалась 9
    // int n; // выше - то же что это
    // *(&n) = 9;
    int* d = &n; // в указатель d записываем ссылку на область памяти объекта n
    std :: cout << *d << std::endl; // -> 9
    *d = 1; // меняем значение в области памяти на 1, => n тоже мняет значение
    std::cout << *d << " " << n << std::endl; // -> 1 1
}
```
---

### Класс для демонстрации

>Удобно, чтобы вспомнить основные концепты написания класса

```cpp
#include <iostream>

class A{
  // приватные атрибуты:
    int *x; // атрибут-указатель
    int y = 2000; 
    static int counter; // атрибут класса

  public:
    A(int *x_): x(x_) {
        counter += 1;
        this->y = *(this->x); // обращение через указатель на экзмепляр
    }

    friend std::ostream& operator<<(std::ostream& out, A& a); 

    int getx() {return *x;};
    int getx2();
};

int A::getx2(){return (*x)*(*x);} // определение метода вне тела класса

std::ostream& operator<<(std::ostream& out, A& a){ // перегрузка вывода
    return out << a.getx()  +  (&a)->getx() + a.counter + a.y*1000; // получили доступ к y, т.к. функция дружественная
}

int A::counter = 0; // инициализация атрибута класса

int main(void){
    int n = 10;
    A a(&n); // можно передать ссылку на константу
    std::cout << a << std::endl;
    return 0;
}
```
---

### Union

> union (unnamed) - выделение для нескольких переменных разных(можно и одинаковых) типов одной области памяти.
> union (named) - то же самое, только указывается имя объединения и обращение к переменным происходит через точку

```cpp
#include <iostream>

int main(){

    union {
        int a;
        char c=90;
        int* d;
    }; // } union_name_1, u_n_2; // if named

    std::cout << d << " " << c << " " << a << std::endl; // -> 0x5a Z 90

    a = 40;
    c = a * 2;
    d = &a;
    std::cout << d << " " << c << " " << a << std::endl; // -> 0x7fffffffd780 � -10368
}
```
---

### Анонимная функция. Демонстрация

```cpp
#include <iostream>
#include <vector>

void my_sort(auto &a, auto cmp, auto my_swap){ // не сортирует, см. вывод
    auto prev = std::begin(a);
    for (int i = 0; i <= a.size(); ++i)
        for (auto j = std::begin(a); j != std::end(a); ++j){
            if (cmp(*j, *prev))
                my_swap(*j, *prev);
        }
    }
int main(){
    std::vector<int> a;
    int n = 10;
    for (int i = 0; i < n; ++i)
        a.push_back(i);

    my_sort(a,
        [](auto a, auto b){return a > b;}, // анонимная функция
        [](auto &a, auto &b){a ^= b ^= a ^= b;} // и это тоже. (swap оч крутой!)
        );                                      // ^ - это XOR

    for (int i = 0; i < n; ++i)
        std::cout << a[i] << ' '; // -> 9 0 1 2 3 4 5 6 7 8
    return 0;
}
```
---

### Template 

> template - шаблон

```cpp
#include <iostream>

template <typename T1, typename T2>
T1 func(T1 a, T2 b){
	return a + b;
}
```
---

### Header files.h

>[!Info] Файлы заголовков нужны для хранения объявлений.

Описание файла заголовков
```cpp
// my_class.h
#ifndef MY_CLASS_H // описаны ли уже эти объявления в файле my_programm.cpp?
#define MY_CLASS_H

namespace N
{
    class my_class
    {
    public:
        void do_something();
    };
}

#endif /* MY_CLASS_H */
```
Реализация объявлений из файла заголовков
```cpp
// my_class.cpp
#include "my_class.h" // header in local directory
#include <iostream> // header in standard library

using namespace N;
using namespace std;

void my_class::do_something()
{
    cout << "Doing something!" << endl;
}
```
Подключения и использование файла заголовков в файле проекта
```cpp
// my_program.cpp
#include "my_class.h"

using namespace N;

int main()
{
    my_class mc;
    mc.do_something();
    return 0;
}
```

### Пример программы с классом, merge-сортировкой, вектором, стат переменной и algorithm

```cpp
#include <iostream>
#include <cstring>

using namespace std;

const int K = 3000; // Макс число деталей
const int STRING_SIZE = 5; // длина строки

struct Detail {
    char partCode[STRING_SIZE];        // Код детали
    char materialCode[STRING_SIZE];   // Код материала
    char unit[STRING_SIZE];          // Единица измерения
    int workshopNumber;             // Номер цеха
    double consumptionRate;        // Норма расхода

    bool operator<(const Detail& other) const {
        return strcmp(partCode, other.partCode) < 0;
    }

    bool operator==(const Detail& other) const {
        return strcmp(partCode, other.partCode) == 0;
    }
};

void printTable(const Detail table[], int size) {
    cout << "Код детали | Код материала | Единица измерения | Номер цеха | Норма расхода\n";
    cout << "-----------------------------------------------------------------\n";
    for (int i = 0; i < size; i++) {
        cout << table[i].partCode << "         | " << table[i].materialCode << "           | "
             << table[i].unit << "               | " << table[i].workshopNumber << "          | "
             << table[i].consumptionRate << endl;
    }
    cout << endl;
}

int binarySearch(const Detail table[], int size, const char* partCode) {
    int l = 0;
    int r = size - 1;
    while (l <= r) {
        int m = (l + r) / 2;
        int cmp = strcmp(table[m].partCode, partCode);
        if (cmp == 0) return m;
        if (cmp < 0) l = m + 1;
        else r = m - 1;
    }
    return -1;
}

void merge(Detail table[], int l, int m, int r) {
    int n1 = m - l + 1;
    int n2 = r - m;

    Detail L[n1], R[n2];

    for (int i = 0; i < n1; i++)
        L[i] = table[l + i];
    for (int i = 0; i < n2; i++)
        R[i] = table[m + 1 + i];

    int i = 0, j = 0, k = l;

    while (i < n1 && j < n2) {
        if (L[i] < R[j]) {
            table[k] = L[i];
            i++;
        } else {
            table[k] = R[j];
            j++;
        }
        k++;
    }

    while (i < n1) {
        table[k] = L[i];
        i++;
        k++;
    }

    while (j < n2) {
        table[k] = R[j];
        j++;
        k++;
    }
}

void mergeSort(Detail table[], int l, int r) {
    if (l < r) {
        int m = l + (r - l) / 2;
        mergeSort(table, l, m);
        mergeSort(table, m + 1, r);
        merge(table, l, m, r);
    }
}

void markDeleting(Detail table[], int& size, const char* partCode, int threshold = 3) {
    static int volume = 0;

    for (int i = 0; i < size; i++) {
        if (strcmp(table[i].partCode, partCode) == 0) {
            strcpy(table[i].partCode, "DELETED");
            volume++;
        }
    }

    if (volume >= threshold) {
        int newSize = 0;
        for (int i = 0; i < size; i++) {
            if (strcmp(table[i].partCode, "DELETED") != 0) {
                table[newSize] = table[i];
                newSize++;
            }
        }
        size = newSize;
        volume = 0;
    }
}

int main() {
    Detail table[K] = {
        {"005", "M123", "кг", 1, 2.5},
        {"002", "M456", "шт", 2, 10},
        {"003", "M789", "м", 1, 3.0},
        {"004", "M123", "кг", 3, 1.8},
        {"001", "M999", "шт", 2, 5},
    };

    int size = 5;

    cout << "Исходная таблица:\n";
    printTable(table, size);

    mergeSort(table, 0, size - 1);
    cout << "Таблица после сортировки:\n";
    printTable(table, size);

    char searchCode[STRING_SIZE] = "003";
    int index = binarySearch(table, size, searchCode);
    if (index != -1)
        cout << "Деталь с кодом " << searchCode << " найдена: "
             << table[index].materialCode << ", " << table[index].unit << endl;
    else
        cout << "Деталь с кодом " << searchCode << " не найдена.\n";

    char deleteCode[STRING_SIZE] = "002";
    markDeleting(table, size, deleteCode);
    cout << "Таблица после удаления детали с кодом " << deleteCode << ":\n";
    printTable(table, size);

    return 0;
}
```

