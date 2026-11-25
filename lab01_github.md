# Лабораторная работа №1

## Указатели

**Язык программирования:** C (стандарт C11)

Отчёт включает 10 задач. Для задачи №3 приведены два способа выделения памяти. Все программы вынесены в отдельные файлы папки [`src`](src), а ниже даны условия, списки идентификаторов, исходный код и примеры результатов работы.

### Содержание

1. [Задача 1. Тройной указатель](#задача-1-тройной-указатель)
2. [Задача 2. Динамический одномерный массив](#задача-2-динамический-одномерный-массив)
3. [Задача 3. Двумерный динамический массив](#задача-3-двумерный-динамический-массив)
4. [Задача 4. Обмен значениями через указатели](#задача-4-обмен-значениями-через-указатели)
5. [Задача 5. Указатель на максимальный элемент](#задача-5-указатель-на-максимальный-элемент)
6. [Задача 6. Переворот массива](#задача-6-переворот-массива)
7. [Задача 7. Поиск подстроки](#задача-7-поиск-подстроки)
8. [Задача 8. Копирование строки](#задача-8-копирование-строки)
9. [Задача 9. Подсчёт пробелов](#задача-9-подсчёт-пробелов)
10. [Задача 10. Сортировка через указатель на функцию](#задача-10-сортировка-через-указатель-на-функцию)

---
## Задача 1. Тройной указатель

**Условие:** Внутри функции `int main(void)` определите указатель `double ***pointer = NULL;`. Инициализируйте его адресом другого указателя типа `double **`, который через следующий указатель ссылается на переменную типа `double`. Используйте `pointer` для записи и чтения значения `2`. Выделите память через функцию семейства `*alloc`, а затем освободите её с помощью `free()`. **Не используйте никаких идентификаторов переменных, кроме `pointer`.**

### Список идентификаторов 1

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `pointer` | `double ***` | Тройной указатель на динамически выделенные указатели и значение типа `double`. |

**Код программы:** [`src/task01.c`](src/task01.c)

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    double ***pointer = NULL;

    // Память для указателя double **.
    pointer = malloc(sizeof *pointer);
    if (pointer == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return 1;
    }

    // Память для указателя double *.
    *pointer = malloc(sizeof **pointer);
    if (*pointer == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        free(pointer);
        return 1;
    }

    // Память для значения double.
    **pointer = malloc(sizeof ***pointer);
    if (**pointer == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        free(*pointer);
        free(pointer);
        return 1;
    }

    ***pointer = 2.0;
    printf("Value: %.0f\n", ***pointer);

    free(**pointer);
    free(*pointer);
    free(pointer);
    return 0;
}
```

**Результат работы программы:**

```text
Value: 2
```

---

## Задача 2. Динамический одномерный массив

**Условие:** Напишите программу, которая создаёт одномерный динамический массив чисел типа `double`, заполняет его с клавиатуры и выводит все элементы. Для доступа к элементам используйте арифметику указателей (`+`), а не оператор `[]`.

### Список идентификаторов 2

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `n` | `int` | Количество элементов массива. |
| `array` | `double *` | Указатель на динамически выделенный массив. |
| `i` | `int` | Индекс элемента в циклах ввода и вывода. |

**Код программы:** [`src/task02.c`](src/task02.c)

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int n;
    printf("Enter the array size: ");
    if (scanf("%d", &n) != 1 || n <= 0) {
        fprintf(stderr, "Incorrect input\n");
        return 1;
    }

    double *array = calloc((size_t)n, sizeof *array);
    if (array == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return 1;
    }

    printf("Enter the array: ");
    for (int i = 0; i < n; ++i) {
        if (scanf("%lf", array + i) != 1) {
            fprintf(stderr, "Incorrect input\n");
            free(array);
            return 1;
        }
    }

    printf("Array elements: ");
    for (int i = 0; i < n; ++i) {
        printf("%g ", *(array + i));
    }
    printf("\n");

    free(array);
    return 0;
}
```

**Результат работы программы:**

```text
Enter the array size: 3
Enter the array: 1.5 -2 0.25
Array elements: 1.5 -2 0.25
```

---

## Задача 3. Двумерный динамический массив

**Условие:** Выделите память под двумерный динамический массив двумя способами: классическим (отдельные блоки для строк) и непрерывным (единый блок для элементов). После выполнения программы корректно освободите всю выделенную память с помощью `free()`.

### Классический способ

**Список идентификаторов 3 (классический способ):**

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `rows` | `int` | Количество строк матрицы. |
| `cols` | `int` | Количество столбцов матрицы. |
| `matrix` | `double **` | Указатель на массив указателей на строки. |
| `i`, `j` | `int` | Счётчики циклов по строкам и столбцам. |
| `k` | `int` | Счётчик при освобождении памяти после ошибки. |

**Код программы:** [`src/task03_classic.c`](src/task03_classic.c)

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int rows, cols;
    printf("Enter rows and cols: ");
    if (scanf("%d %d", &rows, &cols) != 2 || rows <= 0 || cols <= 0) {
        fprintf(stderr, "Incorrect input\n");
        return 1;
    }

    // Отдельный массив указателей на строки.
    double **matrix = calloc((size_t)rows, sizeof *matrix);
    if (matrix == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return 1;
    }

    // Каждая строка занимает собственный блок памяти.
    for (int i = 0; i < rows; ++i) {
        *(matrix + i) = calloc((size_t)cols, sizeof **matrix);
        if (*(matrix + i) == NULL) {
            fprintf(stderr, "Memory allocation failed\n");
            for (int k = 0; k < i; ++k) {
                free(*(matrix + k));
            }
            free(matrix);
            return 1;
        }
    }

    printf("Enter the matrix: ");
    for (int i = 0; i < rows; ++i) {
        for (int j = 0; j < cols; ++j) {
            if (scanf("%lf", *(matrix + i) + j) != 1) {
                fprintf(stderr, "Incorrect input\n");
                for (int k = 0; k < rows; ++k) {
                    free(*(matrix + k));
                }
                free(matrix);
                return 1;
            }
        }
    }

    printf("Matrix elements:\n");
    for (int i = 0; i < rows; ++i) {
        for (int j = 0; j < cols; ++j) {
            printf("%g ", *(*(matrix + i) + j));
        }
        printf("\n");
    }

    for (int i = 0; i < rows; ++i) {
        free(*(matrix + i));
    }
    free(matrix);
    return 0;
}
```

**Результат работы программы (классический способ):**

```text
Enter rows and cols: 2 3
Enter the matrix: 1 2 3 4 5 6
Matrix elements:
1 2 3
4 5 6
```

### Непрерывный способ

**Список идентификаторов 3 (непрерывный способ):**

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `rows` | `int` | Количество строк матрицы. |
| `cols` | `int` | Количество столбцов матрицы. |
| `data` | `double *` | Начало единого непрерывного блока элементов матрицы. |
| `matrix` | `double **` | Массив указателей на начала строк в блоке `data`. |
| `i`, `j` | `int` | Счётчики циклов. |

**Код программы:** [`src/task03_contiguous.c`](src/task03_contiguous.c)

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int rows, cols;
    printf("Enter rows and cols: ");
    if (scanf("%d %d", &rows, &cols) != 2 || rows <= 0 || cols <= 0) {
        fprintf(stderr, "Incorrect input\n");
        return 1;
    }

    // Не допускаем переполнения размера выделяемой памяти.
    if ((size_t)cols > SIZE_MAX / sizeof(double) ||
        (size_t)rows > SIZE_MAX / ((size_t)cols * sizeof(double))) {
        fprintf(stderr, "Matrix is too large\n");
        return 1;
    }

    // Все значения матрицы хранятся одним непрерывным блоком.
    double *data = calloc((size_t)rows * (size_t)cols, sizeof *data);
    if (data == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return 1;
    }

    double **matrix = calloc((size_t)rows, sizeof *matrix);
    if (matrix == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        free(data);
        return 1;
    }

    for (int i = 0; i < rows; ++i) {
        *(matrix + i) = data + (size_t)i * (size_t)cols;
    }

    printf("Enter the matrix: ");
    for (int i = 0; i < rows; ++i) {
        for (int j = 0; j < cols; ++j) {
            if (scanf("%lf", *(matrix + i) + j) != 1) {
                fprintf(stderr, "Incorrect input\n");
                free(matrix);
                free(data);
                return 1;
            }
        }
    }

    printf("Matrix elements:\n");
    for (int i = 0; i < rows; ++i) {
        for (int j = 0; j < cols; ++j) {
            printf("%g ", *(*(matrix + i) + j));
        }
        printf("\n");
    }

    free(matrix);
    free(data);
    return 0;
}
```

**Результат работы программы (непрерывный способ):**

```text
Enter rows and cols: 2 3
Enter the matrix: 1 2 3 4 5 6
Matrix elements:
1 2 3
4 5 6
```

---

## Задача 4. Обмен значениями через указатели

**Условие:** Напишите программу, которая использует функцию для обмена значениями двух переменных через указатели.

### Список идентификаторов 4

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `x, y (main)` | `int` | Две целочисленные переменные, значения которых меняются местами. |
| `x, y (swap)` | `int *` | Указатели на целочисленные переменные. |
| `temp` | `int` | Временное значение при обмене. |

**Код программы:** [`src/task04.c`](src/task04.c)

```c
#include <stdio.h>

void swap(int *x, int *y) {
    int temp = *x;
    *x = *y;
    *y = temp;
}

int main(void) {
    int x = 5, y = 10;
    printf("Before swap: x = %d, y = %d\n", x, y);
    swap(&x, &y);
    printf("After swap: x = %d, y = %d\n", x, y);
    return 0;
}
```

**Результат работы программы:**

```text
Before swap: x = 5, y = 10
After swap: x = 10, y = 5
```

---

## Задача 5. Указатель на максимальный элемент

**Условие:** Создайте функцию, которая принимает массив целых чисел и его размер, а затем возвращает указатель на максимальный элемент массива.

### Список идентификаторов 5

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `arr` | `int * / int[]` | Указатель на начало массива (параметр функции) или сам массив в `main`. |
| `size` | `int` | Количество элементов массива. |
| `max_pos` | `int *` | Указатель на текущий максимальный элемент. |
| `i` | `int` | Счётчик цикла сравнения. |
| `max_el` | `int *` | Указатель на найденный максимальный элемент. |

**Код программы:** [`src/task05.c`](src/task05.c)

```c
#include <stdio.h>

int *find_max(int *arr, int size) {
    if (arr == NULL || size <= 0) {
        return NULL;
    }

    int *max_pos = arr;
    for (int i = 1; i < size; ++i) {
        if (*(arr + i) > *max_pos) {
            max_pos = arr + i;
        }
    }
    return max_pos;
}

int main(void) {
    int arr[] = {1, 100, 3, 4, 5, 6, 7};
    int size = (int)(sizeof arr / sizeof *arr);
    int *max_el = find_max(arr, size);
    if (max_el == NULL) {
        return 1;
    }
    printf("Max element: %d\n", *max_el);
    return 0;
}
```

**Результат работы программы:**

```text
Max element: 100
```

---

## Задача 6. Переворот массива

**Условие:** Реализуйте функцию, которая переворачивает массив целых чисел, используя указатели для перемещения по элементам.

### Список идентификаторов 6

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `arr` | `int * / int[]` | Начало массива в функции и исходный массив в `main`. |
| `size` | `int` | Количество элементов массива. |
| `left` | `int *` | Указатель, перемещающийся от начала массива. |
| `right` | `int *` | Указатель, перемещающийся от конца массива. |
| `temp` | `int` | Временное значение для обмена элементов. |
| `i` | `int` | Счётчик при выводе массива. |

**Код программы:** [`src/task06.c`](src/task06.c)

```c
#include <stdio.h>

void reverse_arr(int *arr, int size) {
    if (arr == NULL || size <= 1) {
        return;
    }

    int *left = arr;
    int *right = arr + size - 1;
    while (left < right) {
        int temp = *left;
        *left = *right;
        *right = temp;
        ++left;
        --right;
    }
}

int main(void) {
    int arr[] = {1, 2, 3, 4, 5, 6, 7};
    int size = (int)(sizeof arr / sizeof *arr);

    printf("Before reverse: ");
    for (int i = 0; i < size; ++i) {
        printf("%d ", *(arr + i));
    }
    printf("\n");

    reverse_arr(arr, size);

    printf("After reverse: ");
    for (int i = 0; i < size; ++i) {
        printf("%d ", *(arr + i));
    }
    printf("\n");
    return 0;
}
```

**Результат работы программы:**

```text
Before reverse: 1 2 3 4 5 6 7
After reverse: 7 6 5 4 3 2 1
```

---

## Задача 7. Поиск подстроки

**Условие:** Реализуйте функцию, которая находит первое вхождение одной строки в другую, используя указатели вместо индексов для доступа к символам строки.

### Список идентификаторов 7

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `str` | `char * / char[]` | Строка, в которой выполняется поиск. |
| `substr` | `const char * / char[]` | Искомая подстрока. |
| `start` | `char *` | Указатель на начало текущей попытки поиска. |
| `p1` | `char *` | Текущая позиция в основной строке. |
| `p2` | `const char *` | Текущая позиция в подстроке. |
| `res` | `char *` | Результат поиска: адрес найденной подстроки либо `NULL`. |

**Код программы:** [`src/task07.c`](src/task07.c)

```c
#include <stddef.h>
#include <stdio.h>

char *find_substr(char *str, const char *substr) {
    // Пустая подстрока начинается с первого символа.
    if (*substr == '\0') {
        return str;
    }

    for (char *start = str; *start != '\0'; ++start) {
        char *p1 = start;
        const char *p2 = substr;
        while (*p1 != '\0' && *p2 != '\0' && *p1 == *p2) {
            ++p1;
            ++p2;
        }
        if (*p2 == '\0') {
            return start;
        }
    }
    return NULL;
}

int main(void) {
    char str[] = "hello world";
    char substr[] = "world";
    char *res = find_substr(str, substr);

    if (res != NULL) {
        printf("Substring position: %td\n", res - str);
        printf("Substr: %s\n", res);
    } else {
        printf("Substring not found\n");
    }
    return 0;
}
```

**Результат работы программы:**

```text
Substring position: 6
Substr: world
```

---

## Задача 8. Копирование строки

**Условие:** Напишите функцию, которая копирует одну строку в другую, используя указатели вместо встроенной функции `strcpy()`.

### Список идентификаторов 8

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `src` | `const char *` | Указатель на исходную строку. |
| `dest` | `char *` | Указатель на строку назначения. |
| `source` | `char[]` | Исходная строка в `main`. |
| `destination` | `char[]` | Буфер для копии строки. |

**Код программы:** [`src/task08.c`](src/task08.c)

```c
#include <stdio.h>

// Размер dest должен позволять сохранить src вместе с '\0'.
void copy_str(const char *src, char *dest) {
    while (*src != '\0') {
        *dest = *src;
        ++src;
        ++dest;
    }
    *dest = '\0';
}

int main(void) {
    char source[] = "Hello world";
    char destination[50];
    copy_str(source, destination);

    printf("String: %s\n", source);
    printf("Copy: %s\n", destination);
    return 0;
}
```

**Результат работы программы:**

```text
String: Hello world
Copy: Hello world
```

---

## Задача 9. Подсчёт пробелов

**Условие:** Напишите функцию, которая подсчитывает количество пробелов в строке, используя указатели для перемещения по символам строки.

### Список идентификаторов 9

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `string` | `const char *` | Указатель на проверяемую строку. |
| `count` | `int` | Количество найденных пробелов. |
| `text` | `char[]` | Исходная строка в `main`. |
| `spaces` | `int` | Результат работы функции. |

**Код программы:** [`src/task09.c`](src/task09.c)

```c
#include <stdio.h>

int count_spaces(const char *string) {
    int count = 0;
    while (*string != '\0') {
        if (*string == ' ') {
            ++count;
        }
        ++string;
    }
    return count;
}

int main(void) {
    char text[] = "Hello world, chicken, cat, cucumber.";
    int spaces = count_spaces(text);
    printf("Text: %s\n", text);
    printf("Count spaces: %d\n", spaces);
    return 0;
}
```

**Результат работы программы:**

```text
Text: Hello world, chicken, cat, cucumber.
Count spaces: 4
```

---

## Задача 10. Сортировка через указатель на функцию

**Условие:** Реализуйте сортировку одномерного массива, принимающую указатель на функцию сравнения. В зависимости от переданной функции массив сортируется по возрастанию или убыванию. Обмен элементов выполняйте через функцию `swap`, работающую с указателями.

### Список идентификаторов 10

| Имя переменной | Тип данных | Смысловое обозначение |
|---|---|---|
| `a, b (swap)` | `int *` | Указатели на элементы для обмена. |
| `temp` | `int` | Временное значение при обмене. |
| `array` | `int *` | Указатель на сортируемый массив. |
| `size` | `int` | Количество элементов. |
| `compare` | `int (*)(int, int)` | Указатель на функцию сравнения. |
| `i, j` | `int` | Счётчики циклов сортировки. |
| `a, b (compare_*)` | `int` | Значения для сравнения. |
| `arr` | `int[]` | Исходный массив в `main`. |
| `choice` | `int` | Выбор порядка: `1` — по возрастанию, `2` — по убыванию. |

**Код программы:** [`src/task10.c`](src/task10.c)

```c
#include <stdio.h>

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// compare возвращает ненулевое значение, если элементы надо поменять.
void sort(int *array, int size, int (*compare)(int, int)) {
    for (int i = 0; i < size - 1; ++i) {
        for (int j = 0; j < size - i - 1; ++j) {
            if (compare(*(array + j), *(array + j + 1))) {
                swap(array + j, array + j + 1);
            }
        }
    }
}

int compare_ascending(int a, int b) {
    return a > b;
}

int compare_descending(int a, int b) {
    return a < b;
}

int main(void) {
    int arr[] = {52, 689, 1, 4, 40, 23};
    int size = (int)(sizeof arr / sizeof *arr);
    int choice;

    printf("Start array: ");
    for (int i = 0; i < size; ++i) {
        printf("%d ", *(arr + i));
    }
    printf("\n");

    printf("Choose sorting order:\n");
    printf("1 - in ascending order\n");
    printf("2 - in descending order\n");
    printf("Your choice: ");
    if (scanf("%d", &choice) != 1 || (choice != 1 && choice != 2)) {
        fprintf(stderr, "Incorrect input\n");
        return 1;
    }

    if (choice == 1) {
        sort(arr, size, compare_ascending);
        printf("Final array (ascending): ");
    } else {
        sort(arr, size, compare_descending);
        printf("Final array (descending): ");
    }

    for (int i = 0; i < size; ++i) {
        printf("%d ", *(arr + i));
    }
    printf("\n");
    return 0;
}
```

**Результат работы программы:**

```text
Вариант 1 — по возрастанию:
Start array: 52 689 1 4 40 23
Choose sorting order:
1 - in ascending order
2 - in descending order
Your choice: 1
Final array (ascending): 1 4 23 40 52 689

Вариант 2 — по убыванию:
Start array: 52 689 1 4 40 23
Choose sorting order:
1 - in ascending order
2 - in descending order
Your choice: 2
Final array (descending): 689 52 40 23 4 1
```

---

## Как запустить программы

**Вариант 1 — по одной программе:**

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic src/task01.c -o task01
./task01
```

Чтобы запустить другую задачу, замените `task01` на `task02`, `task04` и т. д. Для задачи №3 используйте `task03_classic` или `task03_contiguous`.

**Вариант 2 — собрать все сразу (Linux, macOS, WSL):**

```bash
make
./build/task01
./build/task03_classic
```

Для очистки папки с собранными программами: `make clean`.

## Проверка

Программы успешно собраны компилятором GCC с флагами `-std=c11 -Wall -Wextra -Wpedantic`. Проверены основные примеры, оба направления сортировки и реакции на некорректный пользовательский ввод. Для операций динамического выделения памяти предусмотрены проверки ошибок и освобождение памяти.

Примечание: в исходном образце результаты показаны скриншотами `task*.png`. Поскольку сами изображения не были приложены, здесь приведены текстовые примеры запуска, которые GitHub отображает без дополнительных файлов.

---

**Выполнил: Сверчков Георгий**
