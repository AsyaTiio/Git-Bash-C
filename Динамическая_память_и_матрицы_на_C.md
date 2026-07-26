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

В этом квесте в вариантах ниже специально нет вызова `free`, чтобы показать утечку, которую устраняет Quest 2. Для проверки вывода оба варианта подходят.

#### Вариант A. `calloc` и пузырьковая сортировка без `free`

```c
#include <stdio.h>
#include <stdlib.h>

int read_n(int *n);
int read_array(int *a, int n);
void sort_array(int *a, int n);
void print_array(int *a, int n);

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
        error = read_array(data, n);
    }
    if (error == 0) {
        sort_array(data, n);
        print_array(data, n);
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

/* читает n целых в уже выделенный массив */
int read_array(int *a, int n) {
    int i;
    int error;

    error = 0;
    i = 0;
    while (i < n && error == 0) {
        if (scanf("%d", &a[i]) != 1) {
            error = 1;
        }
        i++;
    }
    return error;
}

/* пузырьковая сортировка по возрастанию */
void sort_array(int *a, int n) {
    int i;
    int j;
    int tmp;

    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - 1 - i; j++) {
            if (a[j] > a[j + 1]) {
                tmp = a[j];
                a[j] = a[j + 1];
                a[j + 1] = tmp;
            }
        }
    }
}

/* печать через пробел без хвостового пробела и без \\n */
void print_array(int *a, int n) {
    int i;

    for (i = 0; i < n; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", a[i]);
    }
}
```

**Как работает.** Программа читает длину массива `n` и выделяет ровно `n` элементов типа `int` через `calloc`, поэтому выделенная область сразу заполнена нулями. Затем элементы считываются во выделенный массив. Пузырьковая сортировка многократно сравнивает соседние элементы и при необходимости меняет их местами, пока массив не станет упорядоченным по возрастанию. После этого массив печатается через пробел. Вызова `free` здесь нет, поэтому выделенная память остаётся занятой до конца работы программы. Именно такую утечку нужно убрать в Quest 2.

#### Вариант B. `malloc` и сортировка выбором без `free`

```c
#include <stdio.h>
#include <stdlib.h>

int read_n(int *n);
int read_array(int *a, int n);
void sort_array(int *a, int n);
void print_array(int *a, int n);

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
        error = read_array(data, n);
    }
    if (error == 0) {
        sort_array(data, n);
        print_array(data, n);
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

int read_array(int *a, int n) {
    int i;
    int error;

    error = 0;
    for (i = 0; i < n; i++) {
        if (scanf("%d", a + i) != 1) {
            error = 1;
        }
    }
    return error;
}

/* сортировка выбором: на место i ставим минимум хвоста */
void sort_array(int *a, int n) {
    int i;
    int j;
    int min_i;
    int tmp;

    for (i = 0; i < n - 1; i++) {
        min_i = i;
        for (j = i + 1; j < n; j++) {
            if (a[j] < a[min_i]) {
                min_i = j;
            }
        }
        tmp = a[i];
        a[i] = a[min_i];
        a[min_i] = tmp;
    }
}

void print_array(int *a, int n) {
    int i;

    for (i = 0; i < n; i++) {
        if (i != 0) {
            printf(" ");
        }
        printf("%d", a[i]);
    }
}
```

**Как работает.** Логика программы та же, что в варианте A. Память выделяется через `malloc`, поэтому содержимое блока заранее не обнуляется. Это допустимо, потому что каждый элемент всё равно будет перезаписан при вводе. Сортировка выбором на каждом шаге находит минимум в ещё не упорядоченной части массива и ставит его на текущую позицию. Вызова `free` снова нет.

```bash
gcc -std=c11 -Wall -Werror -Wextra sort.c -o sort
printf "10\n4 3 9 0 1 2 100 2 7 -1\n" | ./sort
# -1 0 1 2 2 3 4 7 9 100
```

---

### Quest 2 — `sort_no_leak.c` (T07D10)

**Суть задания.** Нужна та же сортировка, что в Quest 1, но уже без утечки памяти. После использования массива вызывается `free(data)`. Если в Quest 1 вызов `free` уже был, файл можно просто скопировать в `sort_no_leak.c`.

#### Вариант A. Пузырьковая сортировка и `free` на всех путях

```c
#include <stdio.h>
#include <stdlib.h>

int read_n(int *n);
int read_array(int *a, int n);
void sort_array(int *a, int n);
void print_array(int *a, int n);

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
        error = read_array(data, n);
    }
    if (error == 0) {
        sort_array(data, n);
        print_array(data, n);
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

int read_array(int *a, int n) {
    int i;
    int error;

    error = 0;
    i = 0;
    while (i < n && error == 0) {
        if (scanf("%d", &a[i]) != 1) {
            error = 1;
        }
        i++;
    }
    return error;
}

void sort_array(int *a, int n) {
    int i;
    int j;
    int tmp;

    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - 1 - i; j++) {
            if (a[j] > a[j + 1]) {
                tmp = a[j];
                a[j] = a[j + 1];
                a[j + 1] = tmp;
            }
        }
    }
}

void print_array(int *a, int n) {
    int i;

    for (i = 0; i < n; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", a[i]);
    }
}
```

**Как работает.** Алгоритм совпадает с Quest 1. Отличие в том, что в конце `main` вызывается `free`, если указатель уже не равен `NULL`. Память освобождается и при успешном завершении, и при ошибке чтения после того, как выделение уже произошло.

#### Вариант B. Сортировка выбором и `free`

```c
#include <stdio.h>
#include <stdlib.h>

int read_n(int *n);
int read_array(int *a, int n);
void sort_array(int *a, int n);
void print_array(int *a, int n);

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
        error = read_array(data, n);
    }
    if (error == 0) {
        sort_array(data, n);
        print_array(data, n);
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

int read_array(int *a, int n) {
    int i;
    int error;

    error = 0;
    for (i = 0; i < n; i++) {
        if (scanf("%d", a + i) != 1) {
            error = 1;
        }
    }
    return error;
}

void sort_array(int *a, int n) {
    int i;
    int j;
    int min_i;
    int tmp;

    for (i = 0; i < n - 1; i++) {
        min_i = i;
        for (j = i + 1; j < n; j++) {
            if (a[j] < a[min_i]) {
                min_i = j;
            }
        }
        tmp = a[i];
        a[i] = a[min_i];
        a[min_i] = tmp;
    }
}

void print_array(int *a, int n) {
    int i;

    for (i = 0; i < n; i++) {
        if (i != 0) {
            printf(" ");
        }
        printf("%d", a[i]);
    }
}
```

**Как работает.** В языке C вызов `free(NULL)` безопасен и ничего не делает. Поэтому в конце можно писать просто `free(data)` даже тогда, когда выделение памяти не выполнялось и указатель остался равным `NULL`.

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

**Суть задания.** Нужно собрать в терминале картину со стены комнаты, используя уже заданные в коде массивы и матрицы. Статические массивы и матрицы изменять нельзя. Обычно достаточно дописать функцию `make_picture` и вывод результата. Размер картины равен 15 строкам на 13 столбцов.

#### Вариант A. Порядок отрисовки: рамка, ствол, крона, солнце

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

/* превращает плоский/2D буфер в массив указателей на строки */
void transform(int *buf, int **matr, int n, int m) {
    int i;

    for (i = 0; i < n; i++) {
        matr[i] = buf + i * m;
    }
}

/* собирает картину из заготовок */
void make_picture(int **picture, int n, int m) {
    int frame_w[] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
    int frame_h[] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
    int tree_trunk[] = {7, 7, 7, 7};
    int tree_foliage[] = {3, 3, 3, 3};
    int sun_data[6][5] = {{0, 6, 6, 6, 6}, {0, 0, 6, 6, 6}, {0, 0, 6, 6, 6},
                          {0, 6, 0, 0, 6}, {0, 0, 0, 0, 0}, {0, 0, 0, 0, 0}};
    int i;
    int j;
    int length_frame_w;
    int length_frame_h;
    int length_tree_trunk;
    int length_tree_foliage;
    int trunk_row;

    (void)n;
    (void)m;
    reset_picture(picture, N, M);

    length_frame_w = (int)(sizeof(frame_w) / sizeof(frame_w[0]));
    for (i = 0; i < length_frame_w; i++) {
        picture[0][i] = frame_w[i];
        picture[N / 2][i] = frame_w[i];
        picture[N - 1][i] = frame_w[i];
    }

    length_frame_h = (int)(sizeof(frame_h) / sizeof(frame_h[0]));
    for (i = 0; i < length_frame_h; i++) {
        picture[i][0] = frame_h[i];
        picture[i][M / 2] = frame_h[i];
        picture[i][M - 1] = frame_h[i];
    }

    length_tree_trunk = (int)(sizeof(tree_trunk) / sizeof(tree_trunk[0]));
    for (i = 0; i < length_tree_trunk; i++) {
        trunk_row = (7 - i == 7) ? (6 + i) : (7 + i);
        picture[trunk_row][3] = tree_trunk[i];
        picture[trunk_row][4] = tree_trunk[i];
        picture[10][2 + i] = tree_trunk[i];
    }

    length_tree_foliage = (int)(sizeof(tree_foliage) / sizeof(tree_foliage[0]));
    for (i = 0; i < length_tree_foliage; i++) {
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

/* заливает картину нулями */
void reset_picture(int **picture, int n, int m) {
    int i;
    int j;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            picture[i][j] = 0;
        }
    }
}

/* печать матрицы картины */
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

**Как работает.** Сначала матрица обнуляется. Затем по краям и по центральным линиям копируются единицы из массивов рамки. Ствол из семёрок записывается так, чтобы средняя горизонтальная линия рамки в строке 7 осталась без изменений. Крона заполняется тройками, справа копируется заготовка солнца из шестёрок. Исходные статические массивы при этом не меняются: их значения только копируются в итоговую матрицу `picture`.

#### Вариант B. Тот же рисунок, строки ствола заданы явным списком

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

void transform(int *buf, int **matr, int n, int m) {
    int i;

    for (i = 0; i < n; i++) {
        matr[i] = buf + i * m;
    }
}

void make_picture(int **picture, int n, int m) {
    int frame_w[] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
    int frame_h[] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
    int tree_trunk[] = {7, 7, 7, 7};
    int tree_foliage[] = {3, 3, 3, 3};
    int sun_data[6][5] = {{0, 6, 6, 6, 6}, {0, 0, 6, 6, 6}, {0, 0, 6, 6, 6},
                          {0, 6, 0, 0, 6}, {0, 0, 0, 0, 0}, {0, 0, 0, 0, 0}};
    int trunk_rows[] = {6, 8, 9, 10};
    int i;
    int j;

    (void)n;
    (void)m;
    reset_picture(picture, N, M);

    for (i = 0; i < M; i++) {
        picture[0][i] = frame_w[i];
        picture[7][i] = frame_w[i];
        picture[14][i] = frame_w[i];
    }
    for (i = 0; i < N; i++) {
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

**Как работает.** Здесь номера строк ствола заданы явным массивом `{6, 8, 9, 10}`. Условная формула для вычисления номера строки не используется. Заготовки `tree_trunk`, `frame_w`, `frame_h` и остальные статические данные остаются прежними, поэтому условие о запрете их изменения соблюдено.

```bash
gcc -std=c11 -Wall -Werror -Wextra picture.c -o picture
./picture
```

---

### Quest 6 — `matrix_arithmetic.c` (T07D10)

**Суть задания.** Сначала читается код операции: `1` означает сложение, `2` означает умножение, `3` означает транспонирование. Затем читаются размеры и сами матрицы. Если операцию выполнить нельзя, программа выводит `n/a`.

#### Вариант A. Каждая матрица выделяется одним сплошным блоком

```c
#include <stdio.h>
#include <stdlib.h>

int **create_matrix(int rows, int cols);
void free_matrix(int **m);
int read_matrix(int **m, int rows, int cols);
void print_matrix(int **m, int rows, int cols);
void add_matrix(int **a, int **b, int **c, int rows, int cols);
void mul_matrix(int **a, int **b, int **c, int n, int k, int m);
void transpose_matrix(int **a, int **c, int rows, int cols);

int main(void) {
    int op;
    int n1;
    int m1;
    int n2;
    int m2;
    int **a;
    int **b;
    int **c;
    int error;

    error = 0;
    a = NULL;
    b = NULL;
    c = NULL;
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
        }
    }
    if (error == 0) {
        error = read_matrix(a, n1, m1);
    }
    if (error == 0 && op == 1) {
        if (scanf("%d%d", &n2, &m2) != 2 || n2 <= 0 || m2 <= 0 || n1 != n2 || m1 != m2) {
            error = 1;
        } else {
            b = create_matrix(n2, m2);
            c = create_matrix(n1, m1);
            if (b == NULL || c == NULL) {
                error = 1;
            } else {
                error = read_matrix(b, n2, m2);
                if (error == 0) {
                    add_matrix(a, b, c, n1, m1);
                    print_matrix(c, n1, m1);
                }
            }
        }
    } else if (error == 0 && op == 2) {
        if (scanf("%d%d", &n2, &m2) != 2 || n2 <= 0 || m2 <= 0 || m1 != n2) {
            error = 1;
        } else {
            b = create_matrix(n2, m2);
            c = create_matrix(n1, m2);
            if (b == NULL || c == NULL) {
                error = 1;
            } else {
                error = read_matrix(b, n2, m2);
                if (error == 0) {
                    mul_matrix(a, b, c, n1, m1, m2);
                    print_matrix(c, n1, m2);
                }
            }
        }
    } else if (error == 0 && op == 3) {
        c = create_matrix(m1, n1);
        if (c == NULL) {
            error = 1;
        } else {
            transpose_matrix(a, c, n1, m1);
            print_matrix(c, m1, n1);
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

/* выделяет матрицу одним блоком */
int **create_matrix(int rows, int cols) {
    int **m;
    int *data;
    int i;

    m = (int **)malloc((size_t)rows * sizeof(int *) + (size_t)rows * (size_t)cols * sizeof(int));
    if (m != NULL) {
        data = (int *)(m + rows);
        for (i = 0; i < rows; i++) {
            m[i] = data + i * cols;
        }
    }
    return m;
}

void free_matrix(int **m) {
    free(m);
}

int read_matrix(int **m, int rows, int cols) {
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

void add_matrix(int **a, int **b, int **c, int rows, int cols) {
    int i;
    int j;

    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            c[i][j] = a[i][j] + b[i][j];
        }
    }
}

void mul_matrix(int **a, int **b, int **c, int n, int k, int m) {
    int i;
    int j;
    int t;
    int sum;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            sum = 0;
            for (t = 0; t < k; t++) {
                sum += a[i][t] * b[t][j];
            }
            c[i][j] = sum;
        }
    }
}

void transpose_matrix(int **a, int **c, int rows, int cols) {
    int i;
    int j;

    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            c[j][i] = a[i][j];
        }
    }
}
```

**Как работает.** По коду операции программа проверяет, совместимы ли размеры матриц. Затем результат записывается в матрицу `c` и печатается. В конце `main` освобождаются все три указателя, которые могли быть выделены в ходе работы.

#### Вариант B. Каждая строка выделяется отдельным `malloc`

```c
#include <stdio.h>
#include <stdlib.h>

int **create_matrix(int rows, int cols);
void free_matrix(int **m, int rows);
int read_matrix(int **m, int rows, int cols);
void print_matrix(int **m, int rows, int cols);
void add_matrix(int **a, int **b, int **c, int rows, int cols);
void mul_matrix(int **a, int **b, int **c, int n, int k, int m);
void transpose_matrix(int **a, int **c, int rows, int cols);

int main(void) {
    int op;
    int n1;
    int m1;
    int n2;
    int m2;
    int **a;
    int **b;
    int **c;
    int error;
    int rows_c;
    int cols_c;

    error = 0;
    a = NULL;
    b = NULL;
    c = NULL;
    rows_c = 0;
    n1 = 0;
    n2 = 0;
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
            error = read_matrix(a, n1, m1);
        }
    }
    if (error == 0 && op == 1) {
        if (scanf("%d%d", &n2, &m2) != 2 || n2 != n1 || m2 != m1) {
            error = 1;
        } else {
            b = create_matrix(n2, m2);
            c = create_matrix(n1, m1);
            rows_c = n1;
            cols_c = m1;
            if (b == NULL || c == NULL) {
                error = 1;
            } else {
                error = read_matrix(b, n2, m2);
                if (error == 0) {
                    add_matrix(a, b, c, n1, m1);
                }
            }
        }
    }
    if (error == 0 && op == 2) {
        if (scanf("%d%d", &n2, &m2) != 2 || n2 != m1 || m2 <= 0) {
            error = 1;
        } else {
            b = create_matrix(n2, m2);
            c = create_matrix(n1, m2);
            rows_c = n1;
            cols_c = m2;
            if (b == NULL || c == NULL) {
                error = 1;
            } else {
                error = read_matrix(b, n2, m2);
                if (error == 0) {
                    mul_matrix(a, b, c, n1, m1, m2);
                }
            }
        }
    }
    if (error == 0 && op == 3) {
        c = create_matrix(m1, n1);
        rows_c = m1;
        cols_c = n1;
        if (c == NULL) {
            error = 1;
        } else {
            transpose_matrix(a, c, n1, m1);
        }
    }
    if (error == 0) {
        print_matrix(c, rows_c, cols_c);
    } else {
        printf("n/a");
    }
    free_matrix(a, n1);
    free_matrix(b, n2);
    free_matrix(c, rows_c);
    return 0;
}

int **create_matrix(int rows, int cols) {
    int **m;
    int i;
    int failed;

    failed = 0;
    m = (int **)malloc((size_t)rows * sizeof(int *));
    if (m != NULL) {
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
    return m;
}

void free_matrix(int **m, int rows) {
    int i;

    if (m != NULL) {
        for (i = 0; i < rows; i++) {
            free(m[i]);
        }
        free(m);
    }
}

int read_matrix(int **m, int rows, int cols) {
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

void add_matrix(int **a, int **b, int **c, int rows, int cols) {
    int i;
    int j;

    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            c[i][j] = a[i][j] + b[i][j];
        }
    }
}

void mul_matrix(int **a, int **b, int **c, int n, int k, int m) {
    int i;
    int j;
    int t;

    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++) {
            c[i][j] = 0;
            for (t = 0; t < k; t++) {
                c[i][j] += a[i][t] * b[t][j];
            }
        }
    }
}

void transpose_matrix(int **a, int **c, int rows, int cols) {
    int i;
    int j;

    for (i = 0; i < rows; i++) {
        for (j = 0; j < cols; j++) {
            c[j][i] = a[i][j];
        }
    }
}
```

**Как работает.** Требования к вводу и выводу те же, что в варианте A. Отличие только в организации памяти: каждая строка матрицы лежит в своём блоке кучи. Поэтому при освобождении нужно пройти по всем строкам и вызвать `free` для каждой из них, а затем освободить массив указателей.

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

**Суть задания.** На вход подаётся квадратная матрица вещественных чисел. Нужно вычислить её определитель и вывести его с точностью шесть знаков после запятой. Если матрица не квадратная или ввод некорректен, выводится `n/a`.

#### Вариант A. Разложение Лапласа по первой строке

```c
#include <stdio.h>
#include <stdlib.h>

double **create_matrix(int n);
void free_matrix(double **m);
int read_matrix(double **m, int n);
void minor_matrix(double **m, double **dst, int n, int skip_col);
double determinant(double **m, int n);

int main(void) {
    int rows;
    int cols;
    double **matrix;
    int error;
    double det;

    error = 0;
    matrix = NULL;
    det = 0.0;
    if (scanf("%d%d", &rows, &cols) != 2 || rows <= 0 || cols <= 0 || rows != cols) {
        error = 1;
    }
    if (error == 0) {
        matrix = create_matrix(rows);
        if (matrix == NULL) {
            error = 1;
        }
    }
    if (error == 0) {
        error = read_matrix(matrix, rows);
    }
    if (error == 0) {
        det = determinant(matrix, rows);
        printf("%.6f", det);
    } else {
        printf("n/a");
    }
    free_matrix(matrix);
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

/* минор: удалили строку 0 и столбец skip_col */
void minor_matrix(double **m, double **dst, int n, int skip_col) {
    int i;
    int j;
    int rj;
    int ci;

    rj = 0;
    for (i = 1; i < n; i++) {
        ci = 0;
        for (j = 0; j < n; j++) {
            if (j != skip_col) {
                dst[rj][ci] = m[i][j];
                ci++;
            }
        }
        rj++;
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
            minor_matrix(m, tmp, n, j);
            det += (double)sign * m[0][j] * determinant(tmp, n - 1);
            sign = -sign;
        }
        free_matrix(tmp);
    }
    return det;
}
```

**Как работает.** Если размер матрицы равен 1, определитель совпадает с единственным элементом. При большем размере используется разложение по первой строке: каждый элемент умножается на соответствующий минор и на знак \((-1)^{0+j}\). Функция вызывает себя рекурсивно для миноров меньшего размера, пока не дойдёт до базового случая. Для матрицы из чисел от 1 до 9, записанных по строкам, получается `0.000000`.

#### Вариант B. Для размера 2 на 2 используется явная формула, дальше Лаплас

```c
#include <stdio.h>
#include <stdlib.h>

double **create_matrix(int n);
void free_matrix(double **m, int n);
int read_matrix(double **m, int n);
void minor_matrix(double **m, double **dst, int n, int skip_col);
double determinant(double **m, int n);

int main(void) {
    int rows;
    int cols;
    double **matrix;
    int error;
    double det;

    error = 0;
    matrix = NULL;
    rows = 0;
    if (scanf("%d%d", &rows, &cols) != 2 || rows < 1 || cols < 1 || rows != cols) {
        error = 1;
    }
    if (error == 0) {
        matrix = create_matrix(rows);
        if (matrix == NULL) {
            error = 1;
        } else {
            error = read_matrix(matrix, rows);
        }
    }
    if (error == 0) {
        det = determinant(matrix, rows);
        printf("%.6f", det);
    } else {
        printf("n/a");
    }
    free_matrix(matrix, rows);
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

void minor_matrix(double **m, double **dst, int n, int skip_col) {
    int i;
    int j;
    int ci;

    for (i = 1; i < n; i++) {
        ci = 0;
        for (j = 0; j < n; j++) {
            if (j != skip_col) {
                dst[i - 1][ci] = m[i][j];
                ci++;
            }
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
            minor_matrix(m, tmp, n, j);
            det += (double)sign * m[0][j] * determinant(tmp, n - 1);
            sign = -sign;
        }
        free_matrix(tmp, n - 1);
    }
    return det;
}
```

**Как работает.** Для матрицы размера 2 на 2 определитель считается по формуле \(ad - bc\). Для матриц большего размера снова применяется разложение Лапласа. За счёт отдельной обработки случая 2 на 2 рекурсия становится короче на мелких матрицах.

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
