# Динамическая память и матрицы на C

Краткий разбор квестов **T07D10** (Room 3) и **T08D11** (Room 4, бонус).
Стандарт **C11**, компилятор `gcc` с флагами `-Wall -Werror -Wextra`, стиль **Google** (`IndentWidth: 4`, `ColumnLimit: 110`).
После каждого квеста нужно сделать commit и push исходников из `src/` в ветку **`develop`**.

**Общие правила Werther и чеклиста.**
При ошибке ввода или выделения памяти выводится ровно **`n/a`**.
В конце всего вывода символа перевода строки быть не должно. Если выводится несколько строк, после последней строки перевода строки тоже быть не должно.
В конце каждой строки не должно быть лишнего пробела.
Всю динамическую память нужно освобождать через **`free`**, иначе проверки на утечки не пройдут.
Функция **`system()`** и аналогичные вызовы запрещены.
Код лежит в `src/`, разработка ведётся в ветке `develop`, бинарники пушить нельзя.

Связанные материалы: [Указатели_и_массивы_на_C](Указатели_и_массивы_на_C.md), [C_СПРАВОЧНИК](C_СПРАВОЧНИК.md).

В решениях приведены **два варианта одинаковой сложности**. Вариант A чаще использует один сплошной блок памяти или пузырьковую сортировку. Вариант B чаще выделяет каждую строку отдельно или применяет сортировку выбором. Сдавать можно любой один вариант.

**Структурное правило дня.** У каждой функции должен быть один вход и один выход: один `return` в конце тела, а ошибки обрабатываются через флаги, без ранних `return` посередине.

---

## 0. Теория

### Куча и стек

Обычные локальные массивы живут в **стеке**. Их размер обычно известен при компиляции, а память освобождается сама при выходе из блока. Динамическая память берётся из **кучи** функциями `malloc` и `calloc`. Размер задаётся во время выполнения программы, а освобождать такой блок нужно вручную через `free`.

| | Стек (обычные массивы) | Куча (`malloc` / `calloc`) |
|--|------------------------|----------------------------|
| Размер | известен на этапе компиляции (или VLA) | любой, задаётся в runtime |
| Живучесть | до конца блока `{}` | до вызова `free` |
| Освобождение | автоматическое | вручную |

### `malloc`, `calloc`, `free`

```c
#include <stdlib.h>

int *p = (int *)malloc(n * sizeof(int));   /* содержимое не инициализировано */
int *q = (int *)calloc(n, sizeof(int));    /* все байты равны нулю */
if (p == NULL) { /* память выделить не удалось */ }
free(p);   /* вернуть блок системе */
p = NULL;  /* после free указатель лучше обнулить */
```

Функция `malloc(n)` выделяет `n` байт без инициализации. Функция `calloc(count, size)` выделяет место под `count` объектов размера `size` и заполняет его нулями. Обе функции возвращают указатель типа `void *`, а при неудаче возвращают `NULL`. Функции `free` можно передавать только адрес, полученный от `malloc`, `calloc` или `realloc`. Повторный `free` одного и того же адреса приводит к ошибке.

### Утечка памяти

Если память выделена, а `free` не вызван, блок остаётся занятым до завершения программы. На коротких тестах это может остаться незамеченным, но `valgrind` и проверки на утечки такую ситуацию находят. Поэтому на каждом пути выполнения после успешного `malloc` память нужно освобождать: и при успешном завершении, и при ошибке последующего ввода.

### Матрица как массив массивов

Элемент `a[i][j]` лежит в строке с номером `i` и столбце с номером `j`.

В Quest 3 и Quest 4 нужно реализовать четыре способа выделения памяти и выбирать их пунктами меню от 1 до 4.

1. **Статический способ.** Объявляется массив вида `int a[100][100]`. По условию максимальный размер не превышает 100 на 100.
2. **Один сплошной блок.** Одним вызовом `malloc` выделяются и массив указателей на строки, и сами элементы подряд. Тогда достаточно одного `free`.
3. **Массив указателей и отдельный `malloc` на каждую строку.** Сначала выделяется массив указателей, затем для каждой строки вызывается свой `malloc`. При очистке сначала освобождается каждая строка, затем массив указателей.
4. **Массив указателей и один массив данных.** Делаются два вызова `malloc`: для `int **rows` и для `int *data`. Далее выполняется присваивание `rows[i] = data + i * cols`.

Функции ввода, вывода и обработки нужно писать через `int **`, чтобы они работали одинаково при любом способе выделения памяти. Тогда логику не придётся копировать четыре раза.

### Арифметика матриц

При **сложении** размеры матриц должны совпадать, а каждый элемент результата равен сумме соответствующих элементов: `c[i][j] = a[i][j] + b[i][j]`.

При **умножении** матрица `a` размера `n` на `k` умножается на матрицу `b` размера `k` на `m`. Результат имеет размер `n` на `m`, а элемент считается так: `c[i][j] = Σ a[i][t] * b[t][j]`.

При **транспонировании** элемент `a[i][j]` переходит в позицию `b[j][i]`, поэтому число строк и столбцов меняются местами.

### Определитель и обратная матрица

Определитель определён только для **квадратной** матрицы. Если матрица не квадратная или ввод некорректен, программа должна вывести `n/a`. Определитель удобно считать разложением по строке (метод Лапласа) с рекурсией до матрицы 1 на 1.

Обратная матрица задаётся формулой \(A^{-1} = \frac{1}{\det(A)} \cdot \mathrm{adj}(A)\), где `adj(A)` получается транспонированием матрицы алгебраических дополнений. Если определитель близок к нулю, обратной матрицы не существует и нужно вывести `n/a`. Числа вещественные, вывод выполняется форматом `%.6f`.

### Стиль и проверка

```bash
clang-format -n src/имя.c
gcc -std=c11 -Wall -Werror -Wextra src/файл.c -o prog
# утечки (если есть valgrind):
valgrind --leak-check=full ./prog
```

Пункт меню от 1 до 4 читается обычным числом из stdin. Текст меню печатать не нужно: автотесты ожидают только матрицу либо строку `n/a`.

---

## Решения задач

Команды ниже запускаются из папки `src/` репозитория `T07D10` или `T08D11`.

```bash
git checkout -b develop   # если ещё нет
cd src
```

---

### Quest 1 — `sort.c` (T07D10)

**Суть задания.** Сначала из stdin читается целое `n`, затем ровно `n` целых чисел. Числа нужно отсортировать по возрастанию и вывести. Память под массив выделяется динамически через `malloc` или `calloc`. При любой ошибке выводится `n/a`. В конце вывода символа перевода строки быть не должно.

В прошлой комнате массив был статическим (`int data[10]`) и всегда содержал 10 чисел. Здесь размер задаётся вводом, а память берётся из кучи. Функции `input`, `output`, `sort`, `swap` и проверка разделителей через `getchar` остаются как в прошлом дне — меняется только выделение памяти и работа с `n`.

В этом квесте в вариантах ниже специально нет вызова `free`, чтобы показать утечку, которую устраняет Quest 2. Для проверки вывода оба варианта подходят.

#### Вариант A. `calloc`, пузырёк через указатели, без `free`

```c
#include <stdio.h>
#include <stdlib.h>

int read_n(int *n);
int input(int *a, int n);
void output(int *a, int n);
void sort(int *a, int n);
void swap(int *x, int *y);

int main(void) {
    int n;
    int *data;
    int error;

    error = 0;
    data = NULL;
    error = read_n(&n);
    if (error == 0) {
        data = (int *)calloc((size_t)n, sizeof(int));
        if (data == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = input(data, n);
    }
    if (error == 0) {
        sort(data, n);
        output(data, n);
    } else {
        printf("n/a");
    }
    return 0;
}

/* читает длину массива; 0 — ок, 1 — ошибка */
int read_n(int *n) {
    int error;

    error = 0;
    if (scanf("%d", n) != 1 || *n <= 0) {
        error = 1;
    }
    return error;
}

/* считывает n целых и валидирует разделители */
int input(int *a, int n) {
    int error;
    char next_char;
    int *p;

    error = 0;
    next_char = ' ';
    p = a;
    while (p - a < n && !error) {
        if (scanf("%d", p) != 1) {
            error = 1;
        } else {
            if (p - a < n - 1) {
                next_char = getchar();
                if (next_char != ' ' && next_char != '\t' && next_char != '\n') {
                    error = 1;
                }
            }
            p++;
        }
    }
    if (!error) {
        next_char = getchar();
        while (next_char != '\n' && next_char != EOF && !error) {
            if (next_char != ' ' && next_char != '\t') {
                error = 1;
            } else {
                next_char = getchar();
            }
        }
    }
    return error;
}

/* меняет местами значения двух элементов по их указателям */
void swap(int *x, int *y) {
    int t;

    t = *x;
    *x = *y;
    *y = t;
}

/* сортирует массив пузырьком по возрастанию, используя указатели */
void sort(int *a, int n) {
    int i;
    int *p;

    for (i = 0; i < n - 1; i++) {
        p = a;
        while (p - a < n - 1 - i) {
            if (*p > *(p + 1)) {
                swap(p, p + 1);
            }
            p++;
        }
    }
}

/* выводит элементы массива через пробел без \\n в конце */
void output(int *a, int n) {
    int *p;

    p = a;
    while (p - a < n) {
        if (p != a) {
            printf(" ");
        }
        printf("%d", *p);
        p++;
    }
}
```

**Как работает.** Сначала читается `n`, затем через `calloc` выделяется ровно `n` элементов. Функция `input` перенесена из прошлого дня: она проверяет не только `scanf`, но и символы между числами через `getchar`. Сортировка — тот же пузырёк с `swap` и указателями, только границы зависят от `n`, а не от фиксированного 10. Вызова `free` нет — память «течёт» до конца программы.

#### Вариант B. `malloc` и тот же алгоритм без `free`

```c
#include <stdio.h>
#include <stdlib.h>

int read_n(int *n);
int input(int *a, int n);
void output(int *a, int n);
void sort(int *a, int n);
void swap(int *x, int *y);

int main(void) {
    int n;
    int *data;
    int error;

    error = 0;
    data = NULL;
    error = read_n(&n);
    if (error == 0) {
        data = (int *)malloc((size_t)n * sizeof(int));
        if (data == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = input(data, n);
    }
    if (error == 0) {
        sort(data, n);
        output(data, n);
    } else {
        printf("n/a");
    }
    return 0;
}

int read_n(int *n) {
    int error;

    error = 0;
    if (scanf("%d", n) != 1 || *n <= 0) {
        error = 1;
    }
    return error;
}

int input(int *a, int n) {
    int error;
    char next_char;
    int *p;

    error = 0;
    next_char = ' ';
    p = a;
    while (p - a < n && !error) {
        if (scanf("%d", p) != 1) {
            error = 1;
        } else {
            if (p - a < n - 1) {
                next_char = getchar();
                if (next_char != ' ' && next_char != '\t' && next_char != '\n') {
                    error = 1;
                }
            }
            p++;
        }
    }
    if (!error) {
        next_char = getchar();
        while (next_char != '\n' && next_char != EOF && !error) {
            if (next_char != ' ' && next_char != '\t') {
                error = 1;
            } else {
                next_char = getchar();
            }
        }
    }
    return error;
}

void swap(int *x, int *y) {
    int t;

    t = *x;
    *x = *y;
    *y = t;
}

void sort(int *a, int n) {
    int i;
    int *p;

    for (i = 0; i < n - 1; i++) {
        p = a;
        while (p - a < n - 1 - i) {
            if (*p > *(p + 1)) {
                swap(p, p + 1);
            }
            p++;
        }
    }
}

void output(int *a, int n) {
    int *p;

    p = a;
    while (p - a < n) {
        if (p != a) {
            printf(" ");
        }
        printf("%d", *p);
        p++;
    }
}
```

**Как работает.** Отличие от варианта A только в `malloc` вместо `calloc`: блок не обнуляется, но все ячейки всё равно перезаписываются при вводе. Остальная логика совпадает с прошлым днём.

```bash
gcc -std=c11 -Wall -Werror -Wextra sort.c -o sort
printf "10\n4 3 9 0 1 2 100 2 7 -1\n" | ./sort
# -1 0 1 2 2 3 4 7 9 100
```

---

### Quest 2 — `sort_no_leak.c` (T07D10)

**Суть задания.** Нужна та же сортировка, что в Quest 1, но уже без утечки памяти. После использования массива вызывается `free(data)`. Если в Quest 1 вызов `free` уже был, файл можно просто скопировать в `sort_no_leak.c`.

#### Вариант A. `calloc` и `free`, если указатель не `NULL`

```c
#include <stdio.h>
#include <stdlib.h>

int read_n(int *n);
int input(int *a, int n);
void output(int *a, int n);
void sort(int *a, int n);
void swap(int *x, int *y);

int main(void) {
    int n;
    int *data;
    int error;

    error = 0;
    data = NULL;
    error = read_n(&n);
    if (error == 0) {
        data = (int *)calloc((size_t)n, sizeof(int));
        if (data == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = input(data, n);
    }
    if (error == 0) {
        sort(data, n);
        output(data, n);
    } else {
        printf("n/a");
    }
    if (data != NULL) {
        free(data);
    }
    return 0;
}

int read_n(int *n) {
    int error;

    error = 0;
    if (scanf("%d", n) != 1 || *n <= 0) {
        error = 1;
    }
    return error;
}

int input(int *a, int n) {
    int error;
    char next_char;
    int *p;

    error = 0;
    next_char = ' ';
    p = a;
    while (p - a < n && !error) {
        if (scanf("%d", p) != 1) {
            error = 1;
        } else {
            if (p - a < n - 1) {
                next_char = getchar();
                if (next_char != ' ' && next_char != '\t' && next_char != '\n') {
                    error = 1;
                }
            }
            p++;
        }
    }
    if (!error) {
        next_char = getchar();
        while (next_char != '\n' && next_char != EOF && !error) {
            if (next_char != ' ' && next_char != '\t') {
                error = 1;
            } else {
                next_char = getchar();
            }
        }
    }
    return error;
}

void swap(int *x, int *y) {
    int t;

    t = *x;
    *x = *y;
    *y = t;
}

void sort(int *a, int n) {
    int i;
    int *p;

    for (i = 0; i < n - 1; i++) {
        p = a;
        while (p - a < n - 1 - i) {
            if (*p > *(p + 1)) {
                swap(p, p + 1);
            }
            p++;
        }
    }
}

void output(int *a, int n) {
    int *p;

    p = a;
    while (p - a < n) {
        if (p != a) {
            printf(" ");
        }
        printf("%d", *p);
        p++;
    }
}
```

**Как работает.** Код совпадает с Quest 1, вариант A. В конце `main` вызывается `free(data)`, если память была выделена — это убирает утечку при успехе и при ошибке чтения после `calloc`.

#### Вариант B. `malloc` и `free(data)` всегда

```c
#include <stdio.h>
#include <stdlib.h>

int read_n(int *n);
int input(int *a, int n);
void output(int *a, int n);
void sort(int *a, int n);
void swap(int *x, int *y);

int main(void) {
    int n;
    int *data;
    int error;

    error = 0;
    data = NULL;
    error = read_n(&n);
    if (error == 0) {
        data = (int *)malloc((size_t)n * sizeof(int));
        if (data == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = input(data, n);
    }
    if (error == 0) {
        sort(data, n);
        output(data, n);
    } else {
        printf("n/a");
    }
    free(data);
    return 0;
}

int read_n(int *n) {
    int error;

    error = 0;
    if (scanf("%d", n) != 1 || *n <= 0) {
        error = 1;
    }
    return error;
}

int input(int *a, int n) {
    int error;
    char next_char;
    int *p;

    error = 0;
    next_char = ' ';
    p = a;
    while (p - a < n && !error) {
        if (scanf("%d", p) != 1) {
            error = 1;
        } else {
            if (p - a < n - 1) {
                next_char = getchar();
                if (next_char != ' ' && next_char != '\t' && next_char != '\n') {
                    error = 1;
                }
            }
            p++;
        }
    }
    if (!error) {
        next_char = getchar();
        while (next_char != '\n' && next_char != EOF && !error) {
            if (next_char != ' ' && next_char != '\t') {
                error = 1;
            } else {
                next_char = getchar();
            }
        }
    }
    return error;
}

void swap(int *x, int *y) {
    int t;

    t = *x;
    *x = *y;
    *y = t;
}

void sort(int *a, int n) {
    int i;
    int *p;

    for (i = 0; i < n - 1; i++) {
        p = a;
        while (p - a < n - 1 - i) {
            if (*p > *(p + 1)) {
                swap(p, p + 1);
            }
            p++;
        }
    }
}

void output(int *a, int n) {
    int *p;

    p = a;
    while (p - a < n) {
        if (p != a) {
            printf(" ");
        }
        printf("%d", *p);
        p++;
    }
}
```

**Как работает.** В языке C вызов `free(NULL)` безопасен и ничего не делает. Поэтому в конце можно писать просто `free(data)` — память освободится, если она была выделена, и ничего не произойдёт, если `read_n` завершился с ошибкой.

```bash
gcc -std=c11 -Wall -Werror -Wextra sort_no_leak.c -o sort_no_leak
printf "10\n4 3 9 0 1 2 100 2 7 -1\n" | ./sort_no_leak
# -1 0 1 2 2 3 4 7 9 100
```

---

### Quest 3 — `matrix.c` (T07D10)

**Суть задания.** Сначала из stdin читается номер способа выделения памяти от 1 до 4. Текст меню печатать не нужно. Затем читаются размеры `rows` и `cols`, после них элементы матрицы. Ввод и вывод выполняются через `int **`. Для статического способа размер не превышает 100 на 100. Всю динамическую память нужно освободить. В конце строки не должно быть лишнего пробела. После последней строки матрицы не должно быть перевода строки.

| Код | Способ |
|-----|--------|
| 1 | статический `int[100][100]` |
| 2 | один большой блок (указатели + данные) |
| 3 | `malloc` на каждую строку |
| 4 | массив указателей + один массив данных |

#### Вариант A. Режим 2 выделяет один сплошной блок через `malloc`

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

int read_choice(int *choice);
int read_size(int *rows, int *cols, int choice);
int **alloc_matrix(int choice, int rows, int cols, int static_buf[MAX_SIZE][MAX_SIZE]);
int fill_matrix(int **m, int rows, int cols);
void print_matrix(int **m, int rows, int cols);
void free_matrix(int **m, int choice, int rows);

int main(void) {
    int choice;
    int rows;
    int cols;
    int static_buf[MAX_SIZE][MAX_SIZE];
    int **matrix;
    int error;

    error = 0;
    matrix = NULL;
    error = read_choice(&choice);
    if (error == 0) {
        error = read_size(&rows, &cols, choice);
    }
    if (error == 0) {
        matrix = alloc_matrix(choice, rows, cols, static_buf);
        if (matrix == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = fill_matrix(matrix, rows, cols);
    }
    if (error == 0) {
        print_matrix(matrix, rows, cols);
    } else {
        printf("n/a");
    }
    free_matrix(matrix, choice, rows);
    return 0;
}

/* читает пункт меню 1..4 */
int read_choice(int *choice) {
    int error;

    error = 0;
    if (scanf("%d", choice) != 1 || *choice < 1 || *choice > 4) {
        error = 1;
    }
    return error;
}

/* читает размеры; для статики — не больше MAX_SIZE */
int read_size(int *rows, int *cols, int choice) {
    int error;

    error = 0;
    if (scanf("%d%d", rows, cols) != 2 || *rows <= 0 || *cols <= 0) {
        error = 1;
    }
    if (error == 0 && choice == 1 && (*rows > MAX_SIZE || *cols > MAX_SIZE)) {
        error = 1;
    }
    if (error == 0 && (*rows > MAX_SIZE || *cols > MAX_SIZE)) {
        error = 1;
    }
    return error;
}

/* выделяет матрицу выбранным способом; для 1 — оборачивает static_buf */
int **alloc_matrix(int choice, int rows, int cols, int static_buf[MAX_SIZE][MAX_SIZE]) {
    int **m;
    int i;
    int *data;

    m = NULL;
    if (choice == 1) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        if (m != NULL) {
            for (i = 0; i < rows; i++) {
                m[i] = static_buf[i];
            }
        }
    } else if (choice == 2) {
        m = (int **)malloc((size_t)rows * sizeof(int *) + (size_t)rows * (size_t)cols * sizeof(int));
        if (m != NULL) {
            data = (int *)(m + rows);
            for (i = 0; i < rows; i++) {
                m[i] = data + i * cols;
            }
        }
    } else if (choice == 3) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        if (m != NULL) {
            i = 0;
            while (i < rows) {
                m[i] = (int *)malloc((size_t)cols * sizeof(int));
                if (m[i] == NULL) {
                    while (i > 0) {
                        i--;
                        free(m[i]);
                    }
                    free(m);
                    m = NULL;
                    break;
                }
                i++;
            }
        }
    } else if (choice == 4) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        data = (int *)malloc((size_t)rows * (size_t)cols * sizeof(int));
        if (m == NULL || data == NULL) {
            free(m);
            free(data);
            m = NULL;
        } else {
            for (i = 0; i < rows; i++) {
                m[i] = data + i * cols;
            }
        }
    }
    return m;
}

/* заполняет матрицу из stdin */
int fill_matrix(int **m, int rows, int cols) {
    int i;
    int j;
    int error;

    error = 0;
    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            if (scanf("%d", &m[i][j]) != 1) {
                error = 1;
            }
        }
    }
    return error;
}

/* печатает матрицу без хвостовых пробелов и без \\n после последней строки */
void print_matrix(int **m, int rows, int cols) {
    int i;
    int j;

    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            if (j > 0) {
                printf(" ");
            }
            printf("%d", m[i][j]);
        }
        if (i < rows - 1) {
            printf("\n");
        }
    }
}

/* освобождает только динамические режимы 2–4 */
void free_matrix(int **m, int choice, int rows) {
    int i;

    if (m != NULL) {
        if (choice == 3) {
            for (i = 0; i < rows; i++) {
                free(m[i]);
            }
            free(m);
        } else if (choice == 4) {
            free(m[0]);
            free(m);
        } else if (choice == 1 || choice == 2) {
            free(m);
        }
    }
}
```

**Как работает.** Число `choice` выбирает способ размещения матрицы в памяти. После выделения все остальные функции работают с одним и тем же типом `int **`, поэтому ввод, вывод и дальнейшая обработка не зависят от выбранного режима. Печать построена так, чтобы удовлетворять требованиям автотестов Werther. В режиме 1 сами числа лежат в статическом буфере `static_buf`, а через `malloc` создаётся только массив указателей на строки. Этот массив указателей тоже нужно освободить через `free`.

#### Вариант B. Режим 2 использует два отдельных `malloc`, очистка без раннего выхода

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

int read_choice(int *choice);
int read_size(int *rows, int *cols);
int **alloc_matrix(int choice, int rows, int cols, int static_buf[MAX_SIZE][MAX_SIZE]);
int fill_matrix(int **m, int rows, int cols);
void print_matrix(int **m, int rows, int cols);
void free_matrix(int **m, int choice, int rows);

int main(void) {
    int choice;
    int rows;
    int cols;
    int static_buf[MAX_SIZE][MAX_SIZE];
    int **matrix;
    int error;

    error = 0;
    matrix = NULL;
    rows = 0;
    error = read_choice(&choice);
    if (error == 0) {
        error = read_size(&rows, &cols);
    }
    if (error == 0 && choice == 1 && (rows > MAX_SIZE || cols > MAX_SIZE)) {
        error = 1;
    }
    if (error == 0) {
        matrix = alloc_matrix(choice, rows, cols, static_buf);
        if (matrix == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = fill_matrix(matrix, rows, cols);
    }
    if (error == 0) {
        print_matrix(matrix, rows, cols);
    } else {
        printf("n/a");
    }
    free_matrix(matrix, choice, rows);
    return 0;
}

int read_choice(int *choice) {
    int error;

    error = 0;
    if (scanf("%d", choice) != 1 || *choice < 1 || *choice > 4) {
        error = 1;
    }
    return error;
}

int read_size(int *rows, int *cols) {
    int error;

    error = 0;
    if (scanf("%d%d", rows, cols) != 2 || *rows <= 0 || *cols <= 0 || *rows > MAX_SIZE ||
        *cols > MAX_SIZE) {
        error = 1;
    }
    return error;
}

int **alloc_matrix(int choice, int rows, int cols, int static_buf[MAX_SIZE][MAX_SIZE]) {
    int **m;
    int *data;
    int i;
    int failed;

    m = NULL;
    data = NULL;
    failed = 0;
    if (choice == 1) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        if (m != NULL) {
            for (i = 0; i < rows; i++) {
                m[i] = static_buf[i];
            }
        }
    } else if (choice == 2 || choice == 4) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        data = (int *)malloc((size_t)rows * (size_t)cols * sizeof(int));
        if (m == NULL || data == NULL) {
            free(m);
            free(data);
            m = NULL;
        } else {
            for (i = 0; i < rows; i++) {
                m[i] = data + i * cols;
            }
        }
    } else if (choice == 3) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        if (m != NULL) {
            for (i = 0; i < rows; i++) {
                m[i] = NULL;
            }
            for (i = 0; i < rows; i++) {
                m[i] = (int *)malloc((size_t)cols * sizeof(int));
                if (m[i] == NULL) {
                    failed = 1;
                }
            }
            if (failed != 0) {
                for (i = 0; i < rows; i++) {
                    free(m[i]);
                }
                free(m);
                m = NULL;
            }
        }
    }
    return m;
}

int fill_matrix(int **m, int rows, int cols) {
    int i;
    int j;
    int error;

    error = 0;
    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            if (scanf("%d", &m[i][j]) != 1) {
                error = 1;
            }
        }
    }
    return error;
}

void print_matrix(int **m, int rows, int cols) {
    int i;
    int j;

    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            if (j > 0) {
                printf(" ");
            }
            printf("%d", m[i][j]);
        }
        if (i + 1 < rows) {
            printf("\n");
        }
    }
}

void free_matrix(int **m, int choice, int rows) {
    int i;

    if (m != NULL) {
        if (choice == 3) {
            for (i = 0; i < rows; i++) {
                free(m[i]);
            }
            free(m);
        } else if (choice == 2 || choice == 4) {
            free(m[0]);
            free(m);
        } else if (choice == 1) {
            free(m);
        }
    }
}
```

**Как работает.** В этом варианте режимы 2 и 4 устроены одинаково: сначала выделяется массив указателей, затем один общий массив данных. Для автотеста это допустимо, потому что оба режима остаются динамическими. Они отличаются от статического режима 1 и от режима 3 с отдельным выделением каждой строки. При очистке сначала освобождается блок данных через `m[0]`, затем освобождается массив указателей.

```bash
gcc -std=c11 -Wall -Werror -Wextra matrix.c -o matrix
printf "2\n2 2\n4 3\n9 0\n" | ./matrix
# 4 3
# 9 0
```

---

### Quest 4 — `matrix_extended.c` (T07D10)

**Суть задания.** Программа расширяет `matrix.c`. После печати самой матрицы нужно дополнительно вывести максимумы по каждой строке и минимумы по каждому столбцу. Между этими блоками ставится перевод строки. После последней строки с минимумами по столбцам перевода строки быть не должно.

#### Вариант A

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

int read_choice(int *choice);
int read_size(int *rows, int *cols);
int **alloc_matrix(int choice, int rows, int cols, int static_buf[MAX_SIZE][MAX_SIZE]);
int fill_matrix(int **m, int rows, int cols);
void print_matrix(int **m, int rows, int cols);
void print_vector(int *v, int n);
void row_maxes(int **m, int rows, int cols, int *out);
void col_mins(int **m, int rows, int cols, int *out);
void free_matrix(int **m, int choice, int rows);

int main(void) {
    int choice;
    int rows;
    int cols;
    int static_buf[MAX_SIZE][MAX_SIZE];
    int **matrix;
    int row_max[MAX_SIZE];
    int col_min[MAX_SIZE];
    int error;

    error = 0;
    matrix = NULL;
    rows = 0;
    error = read_choice(&choice);
    if (error == 0) {
        error = read_size(&rows, &cols);
    }
    if (error == 0 && choice == 1 && (rows > MAX_SIZE || cols > MAX_SIZE)) {
        error = 1;
    }
    if (error == 0) {
        matrix = alloc_matrix(choice, rows, cols, static_buf);
        if (matrix == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = fill_matrix(matrix, rows, cols);
    }
    if (error == 0) {
        print_matrix(matrix, rows, cols);
        row_maxes(matrix, rows, cols, row_max);
        col_mins(matrix, rows, cols, col_min);
        printf("\n");
        print_vector(row_max, rows);
        printf("\n");
        print_vector(col_min, cols);
    } else {
        printf("n/a");
    }
    free_matrix(matrix, choice, rows);
    return 0;
}

int read_choice(int *choice) {
    int error;

    error = 0;
    if (scanf("%d", choice) != 1 || *choice < 1 || *choice > 4) {
        error = 1;
    }
    return error;
}

int read_size(int *rows, int *cols) {
    int error;

    error = 0;
    if (scanf("%d%d", rows, cols) != 2 || *rows <= 0 || *cols <= 0 || *rows > MAX_SIZE ||
        *cols > MAX_SIZE) {
        error = 1;
    }
    return error;
}

int **alloc_matrix(int choice, int rows, int cols, int static_buf[MAX_SIZE][MAX_SIZE]) {
    int **m;
    int *data;
    int i;

    m = NULL;
    if (choice == 1) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        if (m != NULL) {
            for (i = 0; i < rows; i++) {
                m[i] = static_buf[i];
            }
        }
    } else if (choice == 2) {
        m = (int **)malloc((size_t)rows * sizeof(int *) + (size_t)rows * (size_t)cols * sizeof(int));
        if (m != NULL) {
            data = (int *)(m + rows);
            for (i = 0; i < rows; i++) {
                m[i] = data + i * cols;
            }
        }
    } else if (choice == 3) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        if (m != NULL) {
            for (i = 0; i < rows; i++) {
                m[i] = (int *)malloc((size_t)cols * sizeof(int));
            }
        }
    } else if (choice == 4) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        data = (int *)malloc((size_t)rows * (size_t)cols * sizeof(int));
        if (m == NULL || data == NULL) {
            free(m);
            free(data);
            m = NULL;
        } else {
            for (i = 0; i < rows; i++) {
                m[i] = data + i * cols;
            }
        }
    }
    return m;
}

int fill_matrix(int **m, int rows, int cols) {
    int i;
    int j;
    int error;

    error = 0;
    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            if (scanf("%d", &m[i][j]) != 1) {
                error = 1;
            }
        }
    }
    return error;
}

void print_matrix(int **m, int rows, int cols) {
    int i;
    int j;

    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            if (j > 0) {
                printf(" ");
            }
            printf("%d", m[i][j]);
        }
        if (i + 1 < rows) {
            printf("\n");
        }
    }
}

void print_vector(int *v, int n) {
    int i;

    for (i = 0; i < n; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", v[i]);
    }
}

void row_maxes(int **m, int rows, int cols, int *out) {
    int i;
    int j;
    int mx;

    for (i = 0; i < rows; i++) {
        mx = m[i][0];
        for (j = 1; j < cols; j++) {
            if (m[i][j] > mx) {
                mx = m[i][j];
            }
        }
        out[i] = mx;
    }
}

void col_mins(int **m, int rows, int cols, int *out) {
    int i;
    int j;
    int mn;

    for (j = 0; j < cols; j++) {
        mn = m[0][j];
        for (i = 1; i < rows; i++) {
            if (m[i][j] < mn) {
                mn = m[i][j];
            }
        }
        out[j] = mn;
    }
}

void free_matrix(int **m, int choice, int rows) {
    int i;

    if (m != NULL) {
        if (choice == 3) {
            for (i = 0; i < rows; i++) {
                free(m[i]);
            }
            free(m);
        } else if (choice == 4) {
            free(m[0]);
            free(m);
        } else if (choice == 1 || choice == 2) {
            free(m);
        }
    }
}
```

**Как работает.** После печати матрицы программа проходит по каждой строке и находит в ней наибольший элемент. Затем она проходит по каждому столбцу и находит в нём наименьший элемент. Для примера из условия максимумы строк равны `4 55 111`. Минимумы столбцов равны `-4 0 1`.

#### Вариант B. Те же четыре режима, поиск через указатель на строку

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

int read_choice(int *choice);
int read_size(int *rows, int *cols);
int **alloc_matrix(int choice, int rows, int cols, int static_buf[MAX_SIZE][MAX_SIZE]);
int fill_matrix(int **m, int rows, int cols);
void print_matrix(int **m, int rows, int cols);
void print_vector(int *v, int n);
void row_maxes(int **m, int rows, int cols, int *out);
void col_mins(int **m, int rows, int cols, int *out);
void free_matrix(int **m, int choice, int rows);

int main(void) {
    int choice;
    int rows;
    int cols;
    int static_buf[MAX_SIZE][MAX_SIZE];
    int **matrix;
    int row_max[MAX_SIZE];
    int col_min[MAX_SIZE];
    int error;

    error = 0;
    matrix = NULL;
    rows = 0;
    error = read_choice(&choice);
    if (error == 0) {
        error = read_size(&rows, &cols);
    }
    if (error == 0) {
        matrix = alloc_matrix(choice, rows, cols, static_buf);
        if (matrix == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = fill_matrix(matrix, rows, cols);
    }
    if (error == 0) {
        print_matrix(matrix, rows, cols);
        row_maxes(matrix, rows, cols, row_max);
        col_mins(matrix, rows, cols, col_min);
        printf("\n");
        print_vector(row_max, rows);
        printf("\n");
        print_vector(col_min, cols);
    } else {
        printf("n/a");
    }
    free_matrix(matrix, choice, rows);
    return 0;
}

int read_choice(int *choice) {
    int error;

    error = 0;
    if (scanf("%d", choice) != 1 || *choice < 1 || *choice > 4) {
        error = 1;
    }
    return error;
}

int read_size(int *rows, int *cols) {
    int error;

    error = 0;
    if (scanf("%d%d", rows, cols) != 2 || *rows <= 0 || *cols <= 0 || *rows > MAX_SIZE ||
        *cols > MAX_SIZE) {
        error = 1;
    }
    return error;
}

int **alloc_matrix(int choice, int rows, int cols, int static_buf[MAX_SIZE][MAX_SIZE]) {
    int **m;
    int *data;
    int i;

    m = NULL;
    data = NULL;
    if (choice == 1) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        if (m != NULL) {
            for (i = 0; i < rows; i++) {
                m[i] = static_buf[i];
            }
        }
    } else if (choice == 2 || choice == 4) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        data = (int *)malloc((size_t)rows * (size_t)cols * sizeof(int));
        if (m != NULL && data != NULL) {
            for (i = 0; i < rows; i++) {
                m[i] = data + i * cols;
            }
        } else {
            free(m);
            free(data);
            m = NULL;
        }
    } else if (choice == 3) {
        m = (int **)malloc((size_t)rows * sizeof(int *));
        if (m != NULL) {
            for (i = 0; i < rows; i++) {
                m[i] = (int *)malloc((size_t)cols * sizeof(int));
            }
        }
    }
    return m;
}

int fill_matrix(int **m, int rows, int cols) {
    int i;
    int j;
    int error;

    error = 0;
    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            if (scanf("%d", *(m + i) + j) != 1) {
                error = 1;
            }
        }
    }
    return error;
}

void print_matrix(int **m, int rows, int cols) {
    int i;
    int j;

    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            if (j > 0) {
                printf(" ");
            }
            printf("%d", *(*(m + i) + j));
        }
        if (i + 1 < rows) {
            printf("\n");
        }
    }
}

void print_vector(int *v, int n) {
    int i;

    for (i = 0; i < n; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", *(v + i));
    }
}

void row_maxes(int **m, int rows, int cols, int *out) {
    int i;
    int j;
    int *row;
    int mx;

    for (i = 0; i < rows; i++) {
        row = m[i];
        mx = row[0];
        for (j = 1; j < cols; j++) {
            if (row[j] > mx) {
                mx = row[j];
            }
        }
        out[i] = mx;
    }
}

void col_mins(int **m, int rows, int cols, int *out) {
    int i;
    int j;
    int mn;

    for (j = 0; j < cols; j++) {
        mn = m[0][j];
        for (i = 1; i < rows; i++) {
            if (m[i][j] < mn) {
                mn = m[i][j];
            }
        }
        out[j] = mn;
    }
}

void free_matrix(int **m, int choice, int rows) {
    int i;

    if (m != NULL) {
        if (choice == 3) {
            for (i = 0; i < rows; i++) {
                free(m[i]);
            }
            free(m);
        } else if (choice == 2 || choice == 4) {
            free(m[0]);
            free(m);
        } else if (choice == 1) {
            free(m);
        }
    }
}
```

**Как работает.** Запись `*(*(m + i) + j)` даёт то же значение, что и запись `m[i][j]`. Отличие только в синтаксисе: используется арифметика указателей.

```bash
gcc -std=c11 -Wall -Werror -Wextra matrix_extended.c -o matrix_extended
printf "2\n3 3\n4 3 1\n9 0 55\n-4 7 111\n" | ./matrix_extended
# 4 3 1
# 9 0 55
# -4 7 111
# 4 55 111
# -4 0 1
```

---

### Quest 5 — `picture.c` (T07D10)

**Суть задания.** Нужно собрать в терминале картину со стены комнаты, используя уже заданные в коде массивы и матрицы. Статические массивы и матрицы в `make_picture` изменять нельзя — только копировать их значения в `picture`. В репозитории обычно уже есть заготовка с `transform`, пустым `make_picture` и `reset_picture`. Дописать нужно отрисовку, вывод и `main`. Размер картины: **15** строк (`N`) на **13** столбцов (`M`).

**Типичные ошибки в заготовке:**
- `void main()` → нужно `int main(void)`
- `reset_picture`: перепутаны `n` и `m` в циклах (`i < n`, `j < m`)
- `transform(picture_data, ...)` → `transform((int *)picture_data, picture, N, M)`
- в `make_picture` нарисована только одна линия рамки — нужны рамка, крона, ствол и солнце

| Символ | Что на картине |
|--------|----------------|
| `1` | рамка «окна» |
| `3` | крона дерева |
| `7` | ствол |
| `6` | солнце |
| `0` | фон |

#### Вариант A. Дописанная заготовка: `trunk_rows` и `sizeof` для рамки

```c
#include <stdio.h>

#define N 15
#define M 13

void transform(int *buf, int **matr, int n, int m);
void make_picture(int **picture, int n, int m);
void reset_picture(int **picture, int n, int m);
void print_picture(int **picture, int n, int m);

int main(void) {
    int picture_data[N][M];
    int *picture[N];

    transform((int *)picture_data, picture, N, M);
    make_picture(picture, N, M);
    print_picture(picture, N, M);

    return 0;
}

void make_picture(int **picture, int n, int m) {
    int frame_w[] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
    int frame_h[] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
    int tree_trunk[] = {7, 7, 7, 7};
    int tree_foliage[] = {3, 3, 3, 3};
    int sun_data[6][5] = {
        {0, 6, 6, 6, 6},
        {0, 0, 6, 6, 6},
        {0, 0, 6, 6, 6},
        {0, 6, 0, 0, 6},
        {0, 0, 0, 0, 0},
        {0, 0, 0, 0, 0}
    };
    int trunk_rows[] = {6, 8, 9, 10};
    int length_frame_w;
    int length_frame_h;
    int i;
    int j;

    (void)n;
    (void)m;
    reset_picture(picture, N, M);

    length_frame_w = (int)(sizeof(frame_w) / sizeof(frame_w[0]));
    for (i = 0; i < length_frame_w; i++) {
        picture[0][i] = frame_w[i];
        picture[7][i] = frame_w[i];
        picture[14][i] = frame_w[i];
    }

    length_frame_h = (int)(sizeof(frame_h) / sizeof(frame_h[0]));
    for (i = 0; i < length_frame_h; i++) {
        picture[i][0] = frame_h[i];
        picture[i][6] = frame_h[i];
        picture[i][12] = frame_h[i];
    }

    for (i = 0; i < 4; i++) {
        picture[trunk_rows[i]][3] = tree_trunk[i];
        picture[trunk_rows[i]][4] = tree_trunk[i];
        picture[10][2 + i] = tree_trunk[i];
    }

    for (i = 0; i < 4; i++) {
        picture[2 + i][3] = tree_foliage[i];
        picture[2 + i][4] = tree_foliage[i];
        picture[3][2 + i] = tree_foliage[i];
        picture[4][2 + i] = tree_foliage[i];
    }

    for (i = 0; i < 6; i++) {
        for (j = 0; j < 5; j++) {
            picture[1 + i][7 + j] = sun_data[i][j];
        }
    }
}

void reset_picture(int **picture, int n, int m) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            picture[i][j] = 0;
        }
    }
}

void transform(int *buf, int **matr, int n, int m) {
    int i;

    for (i = 0; i < n; i++) {
        matr[i] = buf + i * m;
    }
}

void print_picture(int **picture, int n, int m) {
    int row;
    int col;

    for (row = 0; row < n; row++) {
        for (col = 0; col < m; col++) {
            if (col > 0) {
                printf(" ");
            }
            printf("%d", picture[row][col]);
        }
        if (row + 1 < n) {
            printf("\n");
        }
    }
}
```

**Как работает.** `reset_picture` заливает матрицу нулями. Рамка: горизонтальные линии в строках 0, 7 и 14, вертикальные в столбцах 0, 6 и 12. Ствол (`7`) — в строках 6, 8, 9, 10 и ветка в строке 10. Крона (`3`) — в верхней части дерева. Солнце (`6`) — из `sun_data` начиная с позиции `[1][7]`. Заготовки `frame_w`, `frame_h`, `tree_trunk`, `tree_foliage`, `sun_data` не меняются.

#### Вариант B. Те же рисунки, индексы рамки через `N` и `M`

```c
#include <stdio.h>

#define N 15
#define M 13

void transform(int *buf, int **matr, int n, int m);
void make_picture(int **picture, int n, int m);
void reset_picture(int **picture, int n, int m);
void print_picture(int **picture, int n, int m);

int main(void) {
    int picture_data[N][M];
    int *picture[N];

    transform(&picture_data[0][0], picture, N, M);
    make_picture(picture, N, M);
    print_picture(picture, N, M);

    return 0;
}

void make_picture(int **picture, int n, int m) {
    int frame_w[] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
    int frame_h[] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
    int tree_trunk[] = {7, 7, 7, 7};
    int tree_foliage[] = {3, 3, 3, 3};
    int sun_data[6][5] = {
        {0, 6, 6, 6, 6},
        {0, 0, 6, 6, 6},
        {0, 0, 6, 6, 6},
        {0, 6, 0, 0, 6},
        {0, 0, 0, 0, 0},
        {0, 0, 0, 0, 0}
    };
    int trunk_rows[] = {6, 8, 9, 10};
    int i;
    int j;

    (void)n;
    (void)m;
    reset_picture(picture, N, M);

    for (i = 0; i < M; i++) {
        picture[0][i] = frame_w[i];
        picture[N / 2][i] = frame_w[i];
        picture[N - 1][i] = frame_w[i];
    }
    for (i = 0; i < N; i++) {
        picture[i][0] = frame_h[i];
        picture[i][M / 2] = frame_h[i];
        picture[i][M - 1] = frame_h[i];
    }
    for (i = 0; i < 4; i++) {
        picture[trunk_rows[i]][3] = tree_trunk[i];
        picture[trunk_rows[i]][4] = tree_trunk[i];
        picture[10][2 + i] = tree_trunk[i];
    }
    for (i = 0; i < 4; i++) {
        picture[2 + i][3] = tree_foliage[i];
        picture[2 + i][4] = tree_foliage[i];
        picture[3][2 + i] = tree_foliage[i];
        picture[4][2 + i] = tree_foliage[i];
    }
    for (i = 0; i < 6; i++) {
        for (j = 0; j < 5; j++) {
            picture[1 + i][7 + j] = sun_data[i][j];
        }
    }
}

void reset_picture(int **picture, int n, int m) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            picture[i][j] = 0;
        }
    }
}

void transform(int *buf, int **matr, int n, int m) {
    int i;

    for (i = 0; i < n; i++) {
        matr[i] = buf + i * m;
    }
}

void print_picture(int **picture, int n, int m) {
    int row;
    int col;

    for (row = 0; row < n; row++) {
        for (col = 0; col < m; col++) {
            if (col > 0) {
                printf(" ");
            }
            printf("%d", picture[row][col]);
        }
        if (row + 1 < n) {
            printf("\n");
        }
    }
}
```

**Как работает.** Логика отрисовки совпадает с вариантом A. Отличие: для рамки используются `N / 2`, `N - 1`, `M / 2`, `M - 1` вместо явных чисел 7, 14, 6, 12; `transform` вызывается через `&picture_data[0][0]`.

```bash
gcc -std=c11 -Wall -Werror -Wextra picture.c -o picture
./picture
```

---

### Quest 6 — `matrix_arithmetic.c` (T07D10)

**Суть задания.** Сначала читается код операции: `1` — сложение, `2` — умножение, `3` — транспонирование. Затем читаются размеры и матрицы. Если операцию выполнить нельзя, выводится `n/a`.

В репозитории обычно уже есть заготовка с прототипами `input`, `output`, `sum`, `mul`, `transpose` и пустым `main`. Нужно дописать вспомогательные функции выделения памяти, реализации и `main`.

**Формат ввода**

| Операция | Что читать после кода |
|----------|------------------------|
| `1` сумма | `n m` + матрица A, `n m` + матрица B (размеры совпадают) |
| `2` умножение | `n k` + A, `k m` + B (`m` столбцов A = `n` строк B) |
| `3` транспонирование | `n m` + одна матрица |

**Типичные ошибки в заготовке:** `int main()` → `int main(void)`; пустой `main`; для `transpose` нужен указатель на матрицу результата (добавьте 4-й параметр, как в `sum`/`mul`).

**Как работает `input`.** Сначала в `main` читаются `n` и `m`, выделяется матрица, затем `input` считывает только элементы в уже выделённый `int **`.

#### Вариант A. Заготовка репозитория, один блок памяти на матрицу

```c
#include <stdio.h>
#include <stdlib.h>

int **create_matrix(int rows, int cols);
void free_matrix(int **matrix);

int input(int **matrix, int *n, int *m);
void output(int **matrix, int n, int m);
int sum(int **matrix_first, int n_first, int m_first, int **matrix_second,
        int n_second, int m_second, int **matrix_result, int *n_result, int *m_result);
int transpose(int **matrix, int n, int m, int **matrix_result);
int mul(int **matrix_first, int n_first, int m_first, int **matrix_second,
        int n_second, int m_second, int **matrix_result, int *n_result, int *m_result);

int main(void) {
    int op;
    int n1;
    int m1;
    int n2;
    int m2;
    int n_res;
    int m_res;
    int **a;
    int **b;
    int **c;
    int error;

    error = 0;
    a = NULL;
    b = NULL;
    c = NULL;
  /* код операции */
    if (scanf("%d", &op) != 1 || op < 1 || op > 3) {
        error = 1;
    }
  /* первая матрица */
    if (error == 0) {
        if (scanf("%d%d", &n1, &m1) != 2 || n1 <= 0 || m1 <= 0) {
            error = 1;
        }
    }
    if (error == 0) {
        a = create_matrix(n1, m1);
        if (a == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = input(a, &n1, &m1);
    }
  /* операция 1 — сумма */
    if (error == 0 && op == 1) {
        if (scanf("%d%d", &n2, &m2) != 2 || n2 <= 0 || m2 <= 0) {
            error = 1;
        }
        if (error == 0 && (n1 != n2 || m1 != m2)) {
            error = 1;
        }
        if (error == 0) {
            b = create_matrix(n2, m2);
            c = create_matrix(n1, m1);
            if (b == NULL || c == NULL) {
                error = 1;
            }
        }
        if (error == 0) {
            error = input(b, &n2, &m2);
        }
        if (error == 0) {
            error = sum(a, n1, m1, b, n2, m2, c, &n_res, &m_res);
        }
        if (error == 0) {
            output(c, n_res, m_res);
        }
    }
  /* операция 2 — умножение */
    if (error == 0 && op == 2) {
        if (scanf("%d%d", &n2, &m2) != 2 || n2 <= 0 || m2 <= 0) {
            error = 1;
        }
        if (error == 0 && m1 != n2) {
            error = 1;
        }
        if (error == 0) {
            b = create_matrix(n2, m2);
            c = create_matrix(n1, m2);
            if (b == NULL || c == NULL) {
                error = 1;
            }
        }
        if (error == 0) {
            error = input(b, &n2, &m2);
        }
        if (error == 0) {
            error = mul(a, n1, m1, b, n2, m2, c, &n_res, &m_res);
        }
        if (error == 0) {
            output(c, n_res, m_res);
        }
    }
  /* операция 3 — транспонирование */
    if (error == 0 && op == 3) {
        c = create_matrix(m1, n1);
        if (c == NULL) {
            error = 1;
        }
        if (error == 0) {
            error = transpose(a, n1, m1, c);
        }
        if (error == 0) {
            output(c, m1, n1);
        }
    }
    if (error != 0) {
        printf("n/a");
    }
    free_matrix(a);
    free_matrix(b);
    free_matrix(c);
    return 0;
}

int **create_matrix(int rows, int cols) {
    int **matrix;
    int *data;
    int i;

    matrix = (int **)malloc((size_t)rows * sizeof(int *) +
                            (size_t)rows * (size_t)cols * sizeof(int));
    if (matrix != NULL) {
        data = (int *)(matrix + rows);
        for (i = 0; i < rows; i++) {
            matrix[i] = data + i * cols;
        }
    }
    return matrix;
}

void free_matrix(int **matrix) {
    free(matrix);
}

/* читает элементы в уже выделённую матрицу rows x cols; 0 — ок, 1 — ошибка */
int input(int **matrix, int *n, int *m) {
    int i;
    int j;

    (void)n;
    (void)m;
    for (i = 0; i < *n; i++) {
        for (j = 0; j < *m; j++) {
            if (scanf("%d", &matrix[i][j]) != 1) {
                return 1;
            }
        }
    }
    return 0;
}

void output(int **matrix, int n, int m) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            if (j > 0) {
                printf(" ");
            }
            printf("%d", matrix[i][j]);
        }
        if (i + 1 < n) {
            printf("\n");
        }
    }
}

int sum(int **matrix_first, int n_first, int m_first, int **matrix_second,
        int n_second, int m_second, int **matrix_result, int *n_result, int *m_result) {
    int i;
    int j;

    if (n_first != n_second || m_first != m_second) {
        return 1;
    }
    *n_result = n_first;
    *m_result = m_first;
    for (i = 0; i < n_first; i++) {
        for (j = 0; j < m_first; j++) {
            matrix_result[i][j] = matrix_first[i][j] + matrix_second[i][j];
        }
    }
    return 0;
}

int mul(int **matrix_first, int n_first, int m_first, int **matrix_second,
        int n_second, int m_second, int **matrix_result, int *n_result, int *m_result) {
    int i;
    int j;
    int t;
    int sum_val;

    if (m_first != n_second) {
        return 1;
    }
    *n_result = n_first;
    *m_result = m_second;
    for (i = 0; i < n_first; i++) {
        for (j = 0; j < m_second; j++) {
            sum_val = 0;
            for (t = 0; t < m_first; t++) {
                sum_val += matrix_first[i][t] * matrix_second[t][j];
            }
            matrix_result[i][j] = sum_val;
        }
    }
    return 0;
}

int transpose(int **matrix, int n, int m, int **matrix_result) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            matrix_result[j][i] = matrix[i][j];
        }
    }
    return 0;
}
```

**Как работает.** `create_matrix` выделяет указатели на строки и данные одним `malloc`. `main` читает код операции, размеры первой матрицы, выделяет `a` и вызывает `input`. Для суммы и умножения читается вторая матрица `b`, результат пишется в `c`. Для транспонирования результат — матрица `m × n`. При любой ошибке печатается `n/a`, память освобождается через `free_matrix`.

#### Вариант B. Те же функции, каждая строка — отдельный `malloc`

```c
#include <stdio.h>
#include <stdlib.h>

int **create_matrix(int rows, int cols);
void free_matrix(int **matrix, int rows);

int input(int **matrix, int *n, int *m);
void output(int **matrix, int n, int m);
int sum(int **matrix_first, int n_first, int m_first, int **matrix_second,
        int n_second, int m_second, int **matrix_result, int *n_result, int *m_result);
int transpose(int **matrix, int n, int m, int **matrix_result);
int mul(int **matrix_first, int n_first, int m_first, int **matrix_second,
        int n_second, int m_second, int **matrix_result, int *n_result, int *m_result);

int main(void) {
    int op;
    int n1;
    int m1;
    int n2;
    int m2;
    int n_res;
    int m_res;
    int rows_free_a;
    int rows_free_b;
    int rows_free_c;
    int **a;
    int **b;
    int **c;
    int error;

    error = 0;
    a = NULL;
    b = NULL;
    c = NULL;
    rows_free_a = 0;
    rows_free_b = 0;
    rows_free_c = 0;
    if (scanf("%d", &op) != 1 || op < 1 || op > 3) {
        error = 1;
    }
    if (error == 0) {
        if (scanf("%d%d", &n1, &m1) != 2 || n1 <= 0 || m1 <= 0) {
            error = 1;
        }
    }
    if (error == 0) {
        a = create_matrix(n1, m1);
        if (a == NULL) {
            error = 1;
        } else {
            rows_free_a = n1;
        }
    }
    if (error == 0) {
        error = input(a, &n1, &m1);
    }
    if (error == 0 && op == 1) {
        if (scanf("%d%d", &n2, &m2) != 2 || n2 <= 0 || m2 <= 0) {
            error = 1;
        }
        if (error == 0 && (n1 != n2 || m1 != m2)) {
            error = 1;
        }
        if (error == 0) {
            b = create_matrix(n2, m2);
            c = create_matrix(n1, m1);
            if (b == NULL || c == NULL) {
                error = 1;
            } else {
                rows_free_b = n2;
                rows_free_c = n1;
            }
        }
        if (error == 0) {
            error = input(b, &n2, &m2);
        }
        if (error == 0) {
            error = sum(a, n1, m1, b, n2, m2, c, &n_res, &m_res);
        }
        if (error == 0) {
            output(c, n_res, m_res);
        }
    }
    if (error == 0 && op == 2) {
        if (scanf("%d%d", &n2, &m2) != 2 || n2 <= 0 || m2 <= 0) {
            error = 1;
        }
        if (error == 0 && m1 != n2) {
            error = 1;
        }
        if (error == 0) {
            b = create_matrix(n2, m2);
            c = create_matrix(n1, m2);
            if (b == NULL || c == NULL) {
                error = 1;
            } else {
                rows_free_b = n2;
                rows_free_c = n1;
            }
        }
        if (error == 0) {
            error = input(b, &n2, &m2);
        }
        if (error == 0) {
            error = mul(a, n1, m1, b, n2, m2, c, &n_res, &m_res);
        }
        if (error == 0) {
            output(c, n_res, m_res);
        }
    }
    if (error == 0 && op == 3) {
        c = create_matrix(m1, n1);
        if (c == NULL) {
            error = 1;
        } else {
            rows_free_c = m1;
        }
        if (error == 0) {
            error = transpose(a, n1, m1, c);
        }
        if (error == 0) {
            output(c, m1, n1);
        }
    }
    if (error != 0) {
        printf("n/a");
    }
    free_matrix(a, rows_free_a);
    free_matrix(b, rows_free_b);
    free_matrix(c, rows_free_c);
    return 0;
}

int **create_matrix(int rows, int cols) {
    int **matrix;
    int i;
    int failed;

    matrix = (int **)malloc((size_t)rows * sizeof(int *));
    failed = 0;
    if (matrix != NULL) {
        for (i = 0; i < rows; i++) {
            matrix[i] = (int *)malloc((size_t)cols * sizeof(int));
            if (matrix[i] == NULL) {
                failed = 1;
            }
        }
        if (failed != 0) {
            for (i = 0; i < rows; i++) {
                free(matrix[i]);
            }
            free(matrix);
            matrix = NULL;
        }
    }
    return matrix;
}

void free_matrix(int **matrix, int rows) {
    int i;

    if (matrix != NULL) {
        for (i = 0; i < rows; i++) {
            free(matrix[i]);
        }
        free(matrix);
    }
}

int input(int **matrix, int *n, int *m) {
    int i;
    int j;

    (void)n;
    (void)m;
    for (i = 0; i < *n; i++) {
        for (j = 0; j < *m; j++) {
            if (scanf("%d", &matrix[i][j]) != 1) {
                return 1;
            }
        }
    }
    return 0;
}

void output(int **matrix, int n, int m) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            if (j > 0) {
                printf(" ");
            }
            printf("%d", matrix[i][j]);
        }
        if (i + 1 < n) {
            printf("\n");
        }
    }
}

int sum(int **matrix_first, int n_first, int m_first, int **matrix_second,
        int n_second, int m_second, int **matrix_result, int *n_result, int *m_result) {
    int i;
    int j;

    if (n_first != n_second || m_first != m_second) {
        return 1;
    }
    *n_result = n_first;
    *m_result = m_first;
    for (i = 0; i < n_first; i++) {
        for (j = 0; j < m_first; j++) {
            matrix_result[i][j] = matrix_first[i][j] + matrix_second[i][j];
        }
    }
    return 0;
}

int mul(int **matrix_first, int n_first, int m_first, int **matrix_second,
        int n_second, int m_second, int **matrix_result, int *n_result, int *m_result) {
    int i;
    int j;
    int t;
    int sum_val;

    if (m_first != n_second) {
        return 1;
    }
    *n_result = n_first;
    *m_result = m_second;
    for (i = 0; i < n_first; i++) {
        for (j = 0; j < m_second; j++) {
            sum_val = 0;
            for (t = 0; t < m_first; t++) {
                sum_val += matrix_first[i][t] * matrix_second[t][j];
            }
            matrix_result[i][j] = sum_val;
        }
    }
    return 0;
}

int transpose(int **matrix, int n, int m, int **matrix_result) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            matrix_result[j][i] = matrix[i][j];
        }
    }
    return 0;
}
```

**Как работает.** Логика `main` и арифметических функций совпадает с вариантом A. Отличие — каждая строка матрицы в отдельном `malloc`, `free_matrix` освобождает строки по одной.

```bash
gcc -std=c11 -Wall -Werror -Wextra matrix_arithmetic.c -o matrix_arithmetic
printf "1\n2 2\n4 3\n9 0\n2 2\n1 1\n2 2\n" | ./matrix_arithmetic
# 5 4
# 11 2
printf "2\n2 3\n4 3 1\n9 0 2\n3 1\n1\n2\n3\n" | ./matrix_arithmetic
# 13
# 15
printf "3\n2 2\n4 3\n9 0\n" | ./matrix_arithmetic
# 4 9
# 3 0
```

---

### Quest 7 — `key10.txt` (T07D10)

На экране показана такая запись:

```text
1 T       87  46  57  29
2    *   129 156 122 141
3        143 127 107 116
4         69  78 112 101
```

Числа справа образуют матрицу \(M\) размера 4 на 4. Числа слева образуют вектор номеров строк \(v = (1, 2, 3, 4)^T\). Символы **T** и **\*** указывают порядок действий: сначала транспонирование матрицы, затем умножение транспонированной матрицы на вектор \(v\).

\[
M^{T} \cdot v = (1050,\ 1051,\ 1070,\ 1063)
\]

Тот же результат можно проверить поэлементно: \(j\)-й элемент ответа равен сумме \(\sum_i M_{i j}\cdot (i+1)\).

В файл `src/key10.txt` нужно записать одну строку:

```text
1050 1051 1070 1063
```

```bash
# содержимое одной строкой, как выше
git add key10.txt
git commit -m "Quest 7: key10"
git push origin develop
```

---

### Quest 8 — `det.c` (T08D11, бонус)

**Суть задания.** На вход подаётся квадратная матрица вещественных чисел. Нужно вычислить определитель и вывести его с шестью знаками после запятой (`%.6f`). Если матрица не квадратная или ввод некорректен — `n/a`.

В репозитории обычно есть заготовка с прототипами `det`, `input`, `output` и пустым `main`. Нужно дописать выделение памяти, рекурсивный расчёт определителя (Лаплас) и `main`.

**Типичные ошибки в заготовке:** `void main()` → `int main(void)`; `void input` лучше заменить на `int input` для обработки ошибок чтения; `input` считывает элементы в уже выделённую матрицу (размеры читаются в `main`).

#### Вариант A. Заготовка репозитория, Лаплас, один блок памяти

```c
#include <stdio.h>
#include <stdlib.h>

double **create_matrix(int n);
void free_matrix(double **matrix);

int input(double **matrix, int *n, int *m);
void output(double value);
double det(double **matrix, int n, int m);
void minor_matrix(double **matrix, double **dst, int size, int skip_col);

int main(void) {
    int n;
    int m;
    double **matrix;
    double value;
    int error;

    error = 0;
    matrix = NULL;
    value = 0.0;
    if (scanf("%d%d", &n, &m) != 2 || n <= 0 || m <= 0 || n != m) {
        error = 1;
    }
    if (error == 0) {
        matrix = create_matrix(n);
        if (matrix == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = input(matrix, &n, &m);
    }
    if (error == 0) {
        value = det(matrix, n, m);
        output(value);
    } else {
        printf("n/a");
    }
    free_matrix(matrix);
    return 0;
}

double **create_matrix(int n) {
    double **matrix;
    double *data;
    int i;

    matrix = (double **)malloc((size_t)n * sizeof(double *) +
                               (size_t)n * (size_t)n * sizeof(double));
    if (matrix != NULL) {
        data = (double *)(matrix + n);
        for (i = 0; i < n; i++) {
            matrix[i] = data + i * n;
        }
    }
    return matrix;
}

void free_matrix(double **matrix) {
    free(matrix);
}

/* читает n*n элементов в уже выделённую матрицу; 0 — ок, 1 — ошибка */
int input(double **matrix, int *n, int *m) {
    int i;
    int j;

    (void)m;
    for (i = 0; i < *n; i++) {
        for (j = 0; j < *n; j++) {
            if (scanf("%lf", &matrix[i][j]) != 1) {
                return 1;
            }
        }
    }
    return 0;
}

void output(double value) {
    printf("%.6f", value);
}

/* минор: удалили строку 0 и столбец skip_col */
void minor_matrix(double **matrix, double **dst, int size, int skip_col) {
    int i;
    int j;
    int ci;

    for (i = 1; i < size; i++) {
        ci = 0;
        for (j = 0; j < size; j++) {
            if (j != skip_col) {
                dst[i - 1][ci] = matrix[i][j];
                ci++;
            }
        }
    }
}

double det(double **matrix, int n, int m) {
    double result;
    double **tmp;
    int j;
    int sign;

    (void)m;
    result = 0.0;
    if (n == 1) {
        result = matrix[0][0];
    } else if (n == 2) {
        result = matrix[0][0] * matrix[1][1] - matrix[0][1] * matrix[1][0];
    } else {
        tmp = create_matrix(n - 1);
        sign = 1;
        for (j = 0; j < n; j++) {
            minor_matrix(matrix, tmp, n, j);
            result += (double)sign * matrix[0][j] * det(tmp, n - 1, n - 1);
            sign = -sign;
        }
        free_matrix(tmp);
    }
    return result;
}
```

**Как работает.** `main` читает размеры, проверяет `n == m`, выделяет матрицу и вызывает `input`. `det` рекурсивно раскладывает по первой строке (Лаплас); для 1×1 и 2×2 — базовые случаи. `output` печатает одно число с шестью знаками после запятой.

#### Вариант B. Тот же API, каждая строка — отдельный `malloc`

```c
#include <stdio.h>
#include <stdlib.h>

double **create_matrix(int n);
void free_matrix(double **matrix, int n);

int input(double **matrix, int *n, int *m);
void output(double value);
double det(double **matrix, int n, int m);
void minor_matrix(double **matrix, double **dst, int size, int skip_col);

int main(void) {
    int n;
    int m;
    double **matrix;
    double value;
    int error;

    error = 0;
    matrix = NULL;
    value = 0.0;
    if (scanf("%d%d", &n, &m) != 2 || n <= 0 || m <= 0 || n != m) {
        error = 1;
    }
    if (error == 0) {
        matrix = create_matrix(n);
        if (matrix == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = input(matrix, &n, &m);
    }
    if (error == 0) {
        value = det(matrix, n, m);
        output(value);
    } else {
        printf("n/a");
    }
    free_matrix(matrix, n);
    return 0;
}

double **create_matrix(int n) {
    double **matrix;
    int i;
    int failed;

    matrix = (double **)malloc((size_t)n * sizeof(double *));
    failed = 0;
    if (matrix != NULL) {
        for (i = 0; i < n; i++) {
            matrix[i] = (double *)malloc((size_t)n * sizeof(double));
            if (matrix[i] == NULL) {
                failed = 1;
            }
        }
        if (failed != 0) {
            for (i = 0; i < n; i++) {
                free(matrix[i]);
            }
            free(matrix);
            matrix = NULL;
        }
    }
    return matrix;
}

void free_matrix(double **matrix, int n) {
    int i;

    if (matrix != NULL) {
        for (i = 0; i < n; i++) {
            free(matrix[i]);
        }
        free(matrix);
    }
}

int input(double **matrix, int *n, int *m) {
    int i;
    int j;

    (void)m;
    for (i = 0; i < *n; i++) {
        for (j = 0; j < *n; j++) {
            if (scanf("%lf", &matrix[i][j]) != 1) {
                return 1;
            }
        }
    }
    return 0;
}

void output(double value) {
    printf("%.6f", value);
}

void minor_matrix(double **matrix, double **dst, int size, int skip_col) {
    int i;
    int j;
    int ci;

    for (i = 1; i < size; i++) {
        ci = 0;
        for (j = 0; j < size; j++) {
            if (j != skip_col) {
                dst[i - 1][ci] = matrix[i][j];
                ci++;
            }
        }
    }
}

double det(double **matrix, int n, int m) {
    double result;
    double **tmp;
    int j;
    int sign;

    (void)m;
    result = 0.0;
    if (n == 1) {
        result = matrix[0][0];
    } else if (n == 2) {
        result = matrix[0][0] * matrix[1][1] - matrix[0][1] * matrix[1][0];
    } else {
        tmp = create_matrix(n - 1);
        sign = 1;
        for (j = 0; j < n; j++) {
            minor_matrix(matrix, tmp, n, j);
            result += (double)sign * matrix[0][j] * det(tmp, n - 1, n - 1);
            sign = -sign;
        }
        free_matrix(tmp, n - 1);
    }
    return result;
}
```

**Как работает.** Логика совпадает с вариантом A. Память под строки выделяется отдельно; `free_matrix` освобождает каждую строку.

```bash
gcc -std=c11 -Wall -Werror -Wextra det.c -o det
printf "3 3\n1 2 3\n4 5 6\n7 8 9\n" | ./det
# 0.000000
```

---

### Quest 9 — `invert.c` (T08D11, бонус)

**Суть задания.** Нужно вычислить обратную матрицу для квадратной матрицы вещественных чисел и вывести её форматом `%.6f`. В конце строк пробелов быть не должно, после последней строки перевода строки тоже быть не должно. При ошибке или нулевом определителе выводится `n/a`.

В сюжете ИИ просит дополнительно умножить результат на \(-1\). Для автотеста это действие выполнять не нужно: сдаётся обычная обратная матрица \(A^{-1}\).

#### Вариант A. Обратная матрица через алгебраические дополнения

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#define EPS 1e-9

double **create_matrix(int n);
void free_matrix(double **m);
int read_matrix(double **m, int n);
void print_matrix(double **m, int n);
void minor_matrix(double **m, double **dst, int n, int skip_row, int skip_col);
double determinant(double **m, int n);
int inverse_matrix(double **m, double **out, int n);

int main(void) {
    int rows;
    int cols;
    double **matrix;
    double **inv;
    int error;

    error = 0;
    matrix = NULL;
    inv = NULL;
    if (scanf("%d%d", &rows, &cols) != 2 || rows < 1 || cols < 1 || rows != cols) {
        error = 1;
    }
    if (error == 0) {
        matrix = create_matrix(rows);
        inv = create_matrix(rows);
        if (matrix == NULL || inv == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = read_matrix(matrix, rows);
    }
    if (error == 0) {
        error = inverse_matrix(matrix, inv, rows);
    }
    if (error == 0) {
        print_matrix(inv, rows);
    } else {
        printf("n/a");
    }
    free_matrix(matrix);
    free_matrix(inv);
    return 0;
}

double **create_matrix(int n) {
    double **m;
    double *data;
    int i;

    m = (double **)malloc((size_t)n * sizeof(double *) + (size_t)n * (size_t)n * sizeof(double));
    if (m != NULL) {
        data = (double *)(m + n);
        for (i = 0; i < n; i++) {
            m[i] = data + i * n;
        }
    }
    return m;
}

void free_matrix(double **m) {
    free(m);
}

int read_matrix(double **m, int n) {
    int i;
    int j;
    int error;

    error = 0;
    for (i = 0; i < n; i++) {
        for (j = 0; j < n; j++) {
            if (scanf("%lf", &m[i][j]) != 1) {
                error = 1;
            }
        }
    }
    return error;
}

void print_matrix(double **m, int n) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < n; j++) {
            if (j > 0) {
                printf(" ");
            }
            printf("%.6f", m[i][j]);
        }
        if (i + 1 < n) {
            printf("\n");
        }
    }
}

void minor_matrix(double **m, double **dst, int n, int skip_row, int skip_col) {
    int i;
    int j;
    int ri;
    int ci;

    ri = 0;
    for (i = 0; i < n; i++) {
        if (i != skip_row) {
            ci = 0;
            for (j = 0; j < n; j++) {
                if (j != skip_col) {
                    dst[ri][ci] = m[i][j];
                    ci++;
                }
            }
            ri++;
        }
    }
}

double determinant(double **m, int n) {
    double det;
    double **tmp;
    int j;
    int sign;

    det = 0.0;
    if (n == 1) {
        det = m[0][0];
    } else {
        tmp = create_matrix(n - 1);
        sign = 1;
        for (j = 0; j < n; j++) {
            minor_matrix(m, tmp, n, 0, j);
            det += (double)sign * m[0][j] * determinant(tmp, n - 1);
            sign = -sign;
        }
        free_matrix(tmp);
    }
    return det;
}

/* out = adj(m)^T / det = C^T / det */
int inverse_matrix(double **m, double **out, int n) {
    double det;
    double **tmp;
    int i;
    int j;
    int sign;
    int error;

    error = 0;
    det = determinant(m, n);
    if (fabs(det) < EPS) {
        error = 1;
    } else {
        tmp = create_matrix(n - 1);
        for (i = 0; i < n; i++) {
            for (j = 0; j < n; j++) {
                minor_matrix(m, tmp, n, i, j);
                sign = ((i + j) % 2 == 0) ? 1 : -1;
                out[j][i] = (double)sign * determinant(tmp, n - 1) / det;
            }
        }
        free_matrix(tmp);
    }
    return error;
}
```

**Как работает.** Сначала вычисляется определитель. Если его модуль меньше заданной точности `EPS`, считается, что обратной матрицы не существует, и возвращается ошибка. При ненулевом определителе для каждой позиции строится минор. Он умножается на знак \((-1)^{i+j}\) и делится на определитель. Результат записывается в позицию `out[j][i]`, то есть сразу в транспонированном виде, как требует формула через присоединённую матрицу. При сборке программы с `fabs` нужно добавить флаг `-lm`.

#### Вариант B. Та же формула, строки выделяются отдельными `malloc`

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#define EPS 1e-9

double **create_matrix(int n);
void free_matrix(double **m, int n);
int read_matrix(double **m, int n);
void print_matrix(double **m, int n);
void minor_matrix(double **m, double **dst, int n, int skip_row, int skip_col);
double determinant(double **m, int n);
int inverse_matrix(double **m, double **out, int n);

int main(void) {
    int rows;
    int cols;
    double **matrix;
    double **inv;
    int error;

    error = 0;
    matrix = NULL;
    inv = NULL;
    rows = 0;
    if (scanf("%d%d", &rows, &cols) != 2 || rows < 1 || cols < 1 || rows != cols) {
        error = 1;
    }
    if (error == 0) {
        matrix = create_matrix(rows);
        inv = create_matrix(rows);
        if (matrix == NULL || inv == NULL) {
            error = 1;
        } else {
            error = read_matrix(matrix, rows);
        }
    }
    if (error == 0) {
        error = inverse_matrix(matrix, inv, rows);
    }
    if (error == 0) {
        print_matrix(inv, rows);
    } else {
        printf("n/a");
    }
    free_matrix(matrix, rows);
    free_matrix(inv, rows);
    return 0;
}

double **create_matrix(int n) {
    double **m;
    int i;
    int failed;

    failed = 0;
    m = (double **)malloc((size_t)n * sizeof(double *));
    if (m != NULL) {
        for (i = 0; i < n; i++) {
            m[i] = (double *)malloc((size_t)n * sizeof(double));
            if (m[i] == NULL) {
                failed = 1;
            }
        }
        if (failed != 0) {
            for (i = 0; i < n; i++) {
                free(m[i]);
            }
            free(m);
            m = NULL;
        }
    }
    return m;
}

void free_matrix(double **m, int n) {
    int i;

    if (m != NULL) {
        for (i = 0; i < n; i++) {
            free(m[i]);
        }
        free(m);
    }
}

int read_matrix(double **m, int n) {
    int i;
    int j;
    int error;

    error = 0;
    for (i = 0; i < n; i++) {
        for (j = 0; j < n; j++) {
            if (scanf("%lf", &m[i][j]) != 1) {
                error = 1;
            }
        }
    }
    return error;
}

void print_matrix(double **m, int n) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < n; j++) {
            if (j > 0) {
                printf(" ");
            }
            printf("%.6f", m[i][j]);
        }
        if (i + 1 < n) {
            printf("\n");
        }
    }
}

void minor_matrix(double **m, double **dst, int n, int skip_row, int skip_col) {
    int i;
    int j;
    int ri;
    int ci;

    ri = 0;
    for (i = 0; i < n; i++) {
        if (i != skip_row) {
            ci = 0;
            for (j = 0; j < n; j++) {
                if (j != skip_col) {
                    dst[ri][ci] = m[i][j];
                    ci++;
                }
            }
            ri++;
        }
    }
}

double determinant(double **m, int n) {
    double det;
    double **tmp;
    int j;
    int sign;

    det = 0.0;
    if (n == 1) {
        det = m[0][0];
    } else if (n == 2) {
        det = m[0][0] * m[1][1] - m[0][1] * m[1][0];
    } else {
        tmp = create_matrix(n - 1);
        sign = 1;
        for (j = 0; j < n; j++) {
            minor_matrix(m, tmp, n, 0, j);
            det += (double)sign * m[0][j] * determinant(tmp, n - 1);
            sign = -sign;
        }
        free_matrix(tmp, n - 1);
    }
    return det;
}

int inverse_matrix(double **m, double **out, int n) {
    double det;
    double **tmp;
    int i;
    int j;
    int sign;
    int error;

    error = 0;
    det = determinant(m, n);
    if (fabs(det) < EPS) {
        error = 1;
    } else if (n == 1) {
        out[0][0] = 1.0 / m[0][0];
    } else {
        tmp = create_matrix(n - 1);
        for (i = 0; i < n; i++) {
            for (j = 0; j < n; j++) {
                minor_matrix(m, tmp, n, i, j);
                sign = ((i + j) % 2 == 0) ? 1 : -1;
                out[j][i] = (double)sign * determinant(tmp, n - 1) / det;
            }
        }
        free_matrix(tmp, n - 1);
    }
    return error;
}
```

**Как работает.** Алгоритм совпадает с вариантом A. Дополнительно явно обрабатывается случай матрицы размера 1 на 1: обратный элемент равен `1.0 / m[0][0]`. При компиляции снова нужен флаг `-lm`:

```bash
gcc -std=c11 -Wall -Werror -Wextra invert.c -o invert -lm
printf "3 3\n1 0.5 1\n4 1 2\n3 2 2\n" | ./invert
# -1.000000 0.500000 0.000000
# -1.000000 -0.500000 1.000000
# 2.500000 -0.250000 -0.500000
```

---

## Чеклист перед пушем

1. Работа ведётся в ветке `develop`, исходники лежат в `src/`.
2. В репозиторий не попадают бинарники, объектные файлы и `a.out`.
3. Код проходит проверку стиля `clang-format` и оформлен по Google Style.
4. При ошибке печатается ровно `n/a`, без лишних пробелов и без запрещённых переводов строк.
5. Вся динамическая память освобождается. В режиме 3 освобождается каждая строка отдельно.
6. Для меню матриц в stdin передаётся только число способа выделения. Текст интерфейса не печатается.
7. В `key10.txt` записано: `1050 1051 1070 1063`.
