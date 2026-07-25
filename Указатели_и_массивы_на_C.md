# Указатели и массивы на C

Краткий разбор квестов **T05D08** (Room 1) и **T06D09** (Room 2).  
Стандарт **C11**, `gcc` с `-Wall -Werror -Wextra`, стиль **Google** (`IndentWidth: 4`, `ColumnLimit: 110`).  
После каждого квеста: commit + push исходников из `src/` в ветку **`develop`**.

**Общие запреты:** динамическая память (`malloc`/`free` и т.п.), `system()` и аналоги.  
**С Quest 5:** нельзя подключать `stdlib.h`; массив в функцию — **только по указателю**.  
Ошибка ввода → ровно **`n/a`**.

Связанные: [C_СПРАВОЧНИК](C_СПРАВОЧНИК.md), [Циклы_рекурсия_и_функции_на_C](Циклы_рекурсия_и_функции_на_C.md).

---

## 0. Теория: указатели и массивы

### Указатель

**Указатель** — переменная, в которой хранится **адрес** другой переменной.

```c
int x = 10;
int *p = &x;   /* p хранит адрес x */
*p = 20;       /* записали 20 туда, куда указывает p → x стал 20 */
```

| Операция | Смысл |
|----------|--------|
| `&x` | взять адрес переменной `x` |
| `*p` | разыменование — значение по адресу в `p` |
| `int *p` | объявление: «указатель на int» |

Зачем: функция в C по умолчанию получает **копии** аргументов. Чтобы изменить переменную снаружи, передают **адрес**:

```c
void set_max(int a, int b, int *out) {
    *out = (a > b) ? a : b;   /* пишем результат по адресу out */
}
```

Без `&` / `*` как раз и ловят **Segmentation fault** в `maxmin`: туда передавали сами числа вместо адресов.

### Массив — это (почти) указатель

```c
int a[5] = {1, 2, 3, 4, 5};
```

Имя `a` в выражениях превращается в **указатель на первый элемент**. Поэтому:

```c
a[i]     ==  *(a + i)
*(a + i) ==  *(i + a)
i[a]     ==  a[i]      /* экзотика, но правда */
```

Передача в функцию:

```c
void f(int *arr, int n);   /* так и надо по условию дня */
f(a, 5);                   /* передаётся адрес начала, не «копия всего массива» */
```

Внутри `f` изменение `arr[i]` меняет исходный массив в `main`.

### Арифметика указателей

```c
int *p = a;     /* указывает на a[0] */
p++;            /* сдвиг на следующий int → a[1] */
p - a;          /* сколько элементов между p и началом */
```

Цикл по массиву двумя равносильными способами:

```c
/* индексы */
for (int i = 0; i < n; i++) {
    printf("%d", a[i]);
}

/* указатели */
for (int *p = a; p - a < n; p++) {
    printf("%d", *p);
}
```

### Типичные баги дня (из‑за них Segfault)

1. `scanf("%d", n)` вместо `scanf("%d", &n)` — пишем по адресу «число n», а не в переменную.
2. `maxmin(..., max, min)` вместо `&max, &min`.
3. В прототипе `int *max, int min` — `min` не указатель, запись в него снаружи не видна.
4. Сравнение `if (prob2 > max)` когда `max` — указатель: сравнивают число с адресом.

### Статистика для Quest 3–4

Дискретное **равномерное** распределение по `n` числам:

- **мат. ожидание (mean):** \(\bar{x} = \dfrac{1}{n}\sum x_i\)
- **дисперсия (variance):** \(\sigma^2 = \dfrac{1}{n}\sum (x_i - \bar{x})^2\)  ← делим на **n**, не на n−1

Пример `1 2 3 4`: mean = `2.500000`, variance = `1.250000`.

**Правило трёх сигм** в Quest 4 (как в комментарии к `search.c`):

- чётное, `!= 0`
- `>= mean`
- `<= mean + 3 * sqrt(variance)`

Берём **первое** подходящее; если нет — `0`.

### Стиль и Werther

```bash
# скопировать .clang-format в src/
clang-format -n src/имя_файла.c
gcc -std=c11 -Wall -Werror -Wextra src/файл.c -o prog
# если есть math.h (search) — добавить -lm:
gcc -std=c11 -Wall -Werror -Wextra src/search.c -o search -lm
```

В git — только `.c` (и данные, если есть). Не пушь бинарники.

В решениях ниже — **два варианта одинаковой сложности**:  
**A** — через арифметику указателей, **B** — через индексы. Сдавать можно любой один.

---

## Решения задач

Команды — из папки **`src/`** соответствующего репозитория (`T05D08` или `T06D09`).

```bash
git checkout -b develop   # если ещё нет
cd src
```

---

### Quest 1 — `maxmin.c` (T05D08)

**Суть:** починить модуль: max и min из трёх int. Структуру не ломать. Ошибка → `n/a`.

Баги в заготовке: нет `&` у `scanf`, в `maxmin` второй результат не указатель, сравнения идут с указателем как с числом.

#### Вариант A — указатели в `maxmin`, ранний выход

```c
#include <stdio.h>

void maxmin(int prob1, int prob2, int prob3, int *max, int *min);

int main(void) {
    int x, y, z;
    int max, min;

    if (scanf("%d%d%d", &x, &y, &z) != 3) {
        printf("n/a");
        return 0;
    }

    maxmin(x, y, z, &max, &min);
    printf("%d %d", max, min);
    return 0;
}

/* сигнатуру держим: оба результата — через указатели */
void maxmin(int prob1, int prob2, int prob3, int *max, int *min) {
    *max = *min = prob1;

    if (prob2 > *max) {
        *max = prob2;
    }
    if (prob2 < *min) {
        *min = prob2;
    }
    if (prob3 > *max) {
        *max = prob3;
    }
    if (prob3 < *min) {
        *min = prob3;
    }
}
```

**Как работает:** читаем три числа (проверка `scanf == 3`). В `maxmin` кладём в `*max` и `*min` сначала первое, потом обновляем по второму и третьему. Адреса `&max` / `&min` позволяют записать ответ в переменные `main`.

#### Вариант B — та же структура, чуть другая запись сравнений

```c
#include <stdio.h>

void maxmin(int prob1, int prob2, int prob3, int *max, int *min);

int main(void) {
    int x, y, z;
    int max, min;

    if (scanf("%d %d %d", &x, &y, &z) != 3) {
        printf("n/a");
        return 0;
    }

    maxmin(x, y, z, &max, &min);
    printf("%d %d", max, min);
    return 0;
}

void maxmin(int prob1, int prob2, int prob3, int *max, int *min) {
    *max = prob1;
    *min = prob1;

    if (prob2 > *max) *max = prob2;
    if (prob2 < *min) *min = prob2;
    if (prob3 > *max) *max = prob3;
    if (prob3 < *min) *min = prob3;
}
```

**Как работает:** то же самое. Разница только в оформлении присваиваний — для автотеста безразлично.

```bash
gcc -std=c11 -Wall -Werror -Wextra maxmin.c -o maxmin
echo "1 2 3" | ./maxmin
# 3 1

git add maxmin.c
git commit -m "Quest 1: fix maxmin"
git push origin develop
```

---

### Quest 2 — `squaring.c` (T05D08)

**Суть:** `n`, затем `n` целых → возвести в квадрат → вывести. `NMAX = 10`. Функции `input` / `squaring` / `output` не убирать.

#### Вариант A — циклы указателями

```c
#include <stdio.h>

#define NMAX 10

int input(int *a, int *n);
void output(int *a, int n);
void squaring(int *a, int n);

int main(void) {
    int n, data[NMAX];

    if (input(data, &n) != 0) {
        return 0;
    }
    squaring(data, n);
    output(data, n);
    return 0;
}

int input(int *a, int *n) {
    if (scanf("%d", n) != 1 || *n <= 0 || *n > NMAX) {
        printf("n/a");
        return 1;
    }
    for (int *p = a; p - a < *n; p++) {
        if (scanf("%d", p) != 1) {
            printf("n/a");
            return 1;
        }
    }
    return 0;
}

void squaring(int *a, int n) {
    for (int *p = a; p - a < n; p++) {
        *p = (*p) * (*p);
    }
}

void output(int *a, int n) {
    for (int *p = a; p - a < n; p++) {
        if (p != a) {
            printf(" ");
        }
        printf("%d", *p);
    }
}
```

**Как работает:** `input` читает длину и элементы в массив через указатель `p`. `squaring` на месте заменяет каждый элемент на квадрат. `output` печатает через пробел без лишнего пробела в конце.

#### Вариант B — циклы индексами

```c
#include <stdio.h>

#define NMAX 10

int input(int *a, int *n);
void output(int *a, int n);
void squaring(int *a, int n);

int main(void) {
    int n, data[NMAX];

    if (input(data, &n) != 0) {
        return 0;
    }
    squaring(data, n);
    output(data, n);
    return 0;
}

int input(int *a, int *n) {
    if (scanf("%d", n) != 1 || *n <= 0 || *n > NMAX) {
        printf("n/a");
        return 1;
    }
    for (int i = 0; i < *n; i++) {
        if (scanf("%d", &a[i]) != 1) {
            printf("n/a");
            return 1;
        }
    }
    return 0;
}

void squaring(int *a, int n) {
    for (int i = 0; i < n; i++) {
        a[i] *= a[i];
    }
}

void output(int *a, int n) {
    for (int i = 0; i < n; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", a[i]);
    }
}
```

**Как работает:** та же схема, доступ к элементам через `a[i]`. `&a[i]` — адрес i‑го элемента, то же что `a + i`.

```bash
gcc -std=c11 -Wall -Werror -Wextra squaring.c -o squaring
printf "3\n1 2 3\n" | ./squaring
# 1 4 9

git add squaring.c
git commit -m "Quest 2: fix squaring"
git push origin develop
```

---

### Quest 3 — `stat.c` (T05D08)

**Суть:** вывести массив, затем строку: `max min mean variance` (float с 6 знаками). Сигнатуры функций сохранить / можно добавлять.

#### Вариант A — указатели

```c
#include <stdio.h>

#define NMAX 10

int input(int *a, int *n);
void output(int *a, int n);
int max(int *a, int n);
int min(int *a, int n);
double mean(int *a, int n);
double variance(int *a, int n);
void output_result(int max_v, int min_v, double mean_v, double variance_v);

int main(void) {
    int n, data[NMAX];

    if (input(data, &n) != 0) {
        return 0;
    }
    output(data, n);
    printf("\n");
    output_result(max(data, n), min(data, n), mean(data, n), variance(data, n));
    return 0;
}

int input(int *a, int *n) {
    if (scanf("%d", n) != 1 || *n <= 0 || *n > NMAX) {
        printf("n/a");
        return 1;
    }
    for (int *p = a; p - a < *n; p++) {
        if (scanf("%d", p) != 1) {
            printf("n/a");
            return 1;
        }
    }
    return 0;
}

void output(int *a, int n) {
    for (int *p = a; p - a < n; p++) {
        if (p != a) {
            printf(" ");
        }
        printf("%d", *p);
    }
}

int max(int *a, int n) {
    int m = *a;
    for (int *p = a; p - a < n; p++) {
        if (*p > m) {
            m = *p;
        }
    }
    return m;
}

int min(int *a, int n) {
    int m = *a;
    for (int *p = a; p - a < n; p++) {
        if (*p < m) {
            m = *p;
        }
    }
    return m;
}

double mean(int *a, int n) {
    double sum = 0.0;
    for (int *p = a; p - a < n; p++) {
        sum += *p;
    }
    return sum / n;
}

double variance(int *a, int n) {
    double m = mean(a, n);
    double sum = 0.0;
    for (int *p = a; p - a < n; p++) {
        double d = *p - m;
        sum += d * d;
    }
    return sum / n;
}

void output_result(int max_v, int min_v, double mean_v, double variance_v) {
    printf("%d %d %.6f %.6f", max_v, min_v, mean_v, variance_v);
}
```

**Как работает:** после вывода массива считаем экстремумы, среднее и дисперсию (среднее квадратов отклонений от mean). `%.6f` даёт ровно 6 знаков после запятой.

#### Вариант B — индексы

```c
#include <stdio.h>

#define NMAX 10

int input(int *a, int *n);
void output(int *a, int n);
int max(int *a, int n);
int min(int *a, int n);
double mean(int *a, int n);
double variance(int *a, int n);
void output_result(int max_v, int min_v, double mean_v, double variance_v);

int main(void) {
    int n, data[NMAX];

    if (input(data, &n) != 0) {
        return 0;
    }
    output(data, n);
    printf("\n");
    output_result(max(data, n), min(data, n), mean(data, n), variance(data, n));
    return 0;
}

int input(int *a, int *n) {
    if (scanf("%d", n) != 1 || *n <= 0 || *n > NMAX) {
        printf("n/a");
        return 1;
    }
    for (int i = 0; i < *n; i++) {
        if (scanf("%d", &a[i]) != 1) {
            printf("n/a");
            return 1;
        }
    }
    return 0;
}

void output(int *a, int n) {
    for (int i = 0; i < n; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", a[i]);
    }
}

int max(int *a, int n) {
    int m = a[0];
    for (int i = 1; i < n; i++) {
        if (a[i] > m) {
            m = a[i];
        }
    }
    return m;
}

int min(int *a, int n) {
    int m = a[0];
    for (int i = 1; i < n; i++) {
        if (a[i] < m) {
            m = a[i];
        }
    }
    return m;
}

double mean(int *a, int n) {
    double sum = 0.0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
    }
    return sum / n;
}

double variance(int *a, int n) {
    double m = mean(a, n);
    double sum = 0.0;
    for (int i = 0; i < n; i++) {
        double d = a[i] - m;
        sum += d * d;
    }
    return sum / n;
}

void output_result(int max_v, int min_v, double mean_v, double variance_v) {
    printf("%d %d %.6f %.6f", max_v, min_v, mean_v, variance_v);
}
```

**Как работает:** те же формулы, доступ через индексы. Дисперсия снова `/ n`.

```bash
gcc -std=c11 -Wall -Werror -Wextra stat.c -o stat
printf "4\n1 2 3 4\n" | ./stat
# 1 2 3 4
# 4 1 2.500000 1.250000

git add stat.c
git commit -m "Quest 3: implement stat"
git push origin develop
```

---

### Quest 4 — `search.c` (T05D08)

**Суть:** до 30 чисел; найти **первое** чётное `!= 0`, `>= mean`, `<= mean + 3*sqrt(variance)`. Нет → `0`. Нужен `<math.h>` и линковка `-lm`.

#### Вариант A — указатели

```c
#include <math.h>
#include <stdio.h>

#define NMAX 30

int input(int *a, int *n);
double mean(int *a, int n);
double variance(int *a, int n);
int find_number(int *a, int n);

int main(void) {
    int n, data[NMAX];

    if (input(data, &n) != 0) {
        return 0;
    }
    printf("%d", find_number(data, n));
    return 0;
}

int input(int *a, int *n) {
    if (scanf("%d", n) != 1 || *n <= 0 || *n > NMAX) {
        printf("n/a");
        return 1;
    }
    for (int *p = a; p - a < *n; p++) {
        if (scanf("%d", p) != 1) {
            printf("n/a");
            return 1;
        }
    }
    return 0;
}

double mean(int *a, int n) {
    double sum = 0.0;
    for (int *p = a; p - a < n; p++) {
        sum += *p;
    }
    return sum / n;
}

double variance(int *a, int n) {
    double m = mean(a, n);
    double sum = 0.0;
    for (int *p = a; p - a < n; p++) {
        double d = *p - m;
        sum += d * d;
    }
    return sum / n;
}

int find_number(int *a, int n) {
    double m = mean(a, n);
    double limit = m + 3.0 * sqrt(variance(a, n));

    for (int *p = a; p - a < n; p++) {
        if (*p != 0 && *p % 2 == 0 && *p >= m && *p <= limit) {
            return *p;
        }
    }
    return 0;
}
```

**Как работает:** считаем mean и верхнюю границу трёх сигм, идём слева направо и возвращаем первое подходящее число. Если цикл кончился — `0`.

#### Вариант B — индексы

```c
#include <math.h>
#include <stdio.h>

#define NMAX 30

int input(int *a, int *n);
double mean(int *a, int n);
double variance(int *a, int n);
int find_number(int *a, int n);

int main(void) {
    int n, data[NMAX];

    if (input(data, &n) != 0) {
        return 0;
    }
    printf("%d", find_number(data, n));
    return 0;
}

int input(int *a, int *n) {
    if (scanf("%d", n) != 1 || *n <= 0 || *n > NMAX) {
        printf("n/a");
        return 1;
    }
    for (int i = 0; i < *n; i++) {
        if (scanf("%d", &a[i]) != 1) {
            printf("n/a");
            return 1;
        }
    }
    return 0;
}

double mean(int *a, int n) {
    double sum = 0.0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
    }
    return sum / n;
}

double variance(int *a, int n) {
    double m = mean(a, n);
    double sum = 0.0;
    for (int i = 0; i < n; i++) {
        double d = a[i] - m;
        sum += d * d;
    }
    return sum / n;
}

int find_number(int *a, int n) {
    double m = mean(a, n);
    double limit = m + 3.0 * sqrt(variance(a, n));

    for (int i = 0; i < n; i++) {
        if (a[i] != 0 && a[i] % 2 == 0 && a[i] >= m && a[i] <= limit) {
            return a[i];
        }
    }
    return 0;
}
```

**Как работает:** идентичная логика с индексами. Для `1 2 3 4`: mean `2.5`, подходит `4`.

```bash
gcc -std=c11 -Wall -Werror -Wextra search.c -o search -lm
printf "4\n1 2 3 4\n" | ./search
# 4

git add search.c
git commit -m "Quest 4: implement search"
git push origin develop
```

---

### Quest 5 — `sort.c` (T06D09)

**Суть:** ровно **10** целых → сортировка по возрастанию. Отдельные `input` / `sort` / `output`. Без `stdlib.h`. Массив — по указателю.

#### Вариант A — bubble sort + указатели

```c
#include <stdio.h>

#define N 10

int input(int *a);
void output(int *a);
void sort(int *a);
void swap(int *x, int *y);

int main(void) {
    int data[N];

    if (input(data) != 0) {
        printf("n/a");
        return 0;
    }
    sort(data);
    output(data);
    return 0;
}

int input(int *a) {
    for (int *p = a; p - a < N; p++) {
        if (scanf("%d", p) != 1) {
            return 1;
        }
    }
    return 0;
}

void swap(int *x, int *y) {
    int t = *x;
    *x = *y;
    *y = t;
}

void sort(int *a) {
    for (int i = 0; i < N - 1; i++) {
        for (int *p = a; p - a < N - 1 - i; p++) {
            if (*p > *(p + 1)) {
                swap(p, p + 1);
            }
        }
    }
}

void output(int *a) {
    for (int *p = a; p - a < N; p++) {
        if (p != a) {
            printf(" ");
        }
        printf("%d", *p);
    }
}
```

**Как работает:** пузырёк — соседние элементы меняются местами, пока массив не упорядочен. `swap` меняет значения по двум адресам.

#### Вариант B — selection sort + индексы

```c
#include <stdio.h>

#define N 10

int input(int *a);
void output(int *a);
void sort(int *a);
void swap(int *x, int *y);

int main(void) {
    int data[N];

    if (input(data) != 0) {
        printf("n/a");
        return 0;
    }
    sort(data);
    output(data);
    return 0;
}

int input(int *a) {
    for (int i = 0; i < N; i++) {
        if (scanf("%d", &a[i]) != 1) {
            return 1;
        }
    }
    return 0;
}

void swap(int *x, int *y) {
    int t = *x;
    *x = *y;
    *y = t;
}

void sort(int *a) {
    for (int i = 0; i < N - 1; i++) {
        int best = i;
        for (int j = i + 1; j < N; j++) {
            if (a[j] < a[best]) {
                best = j;
            }
        }
        if (best != i) {
            swap(&a[i], &a[best]);
        }
    }
}

void output(int *a) {
    for (int i = 0; i < N; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", a[i]);
    }
}
```

**Как работает:** на каждом шаге ищем минимум в хвосте и ставим его на позицию `i`. Сложность та же порядка \(O(n^2)\), для 10 элементов без разницы.

```bash
gcc -std=c11 -Wall -Werror -Wextra sort.c -o sort
echo "4 3 9 0 1 2 100 2 7 -1" | ./sort
# -1 0 1 2 2 3 4 7 9 100

git add sort.c
git commit -m "Quest 5: sort"
git push origin develop
```

---

### Quest 6 — `key9part1.c` (T06D09)

**Суть:** длина ≤ 10 и массив → сумма **чётных** (ноль **нечётный!**) → новая строка: элементы, на которые эта сумма делится нацело. Нет чётных / ошибка → `n/a`. Без `stdlib.h`.

#### Вариант A — указатели

```c
#include <stdio.h>

#define NMAX 10

int input(int *buffer, int *length);
void output(int *buffer, int length);
int sum_numbers(int *buffer, int length);
int find_numbers(int *buffer, int length, int number, int *numbers);
int has_even(int *buffer, int length);

int main(void) {
    int length, buffer[NMAX], numbers[NMAX];
    int sum, count;

    if (input(buffer, &length) != 0 || !has_even(buffer, length)) {
        printf("n/a");
        return 0;
    }

    sum = sum_numbers(buffer, length);
    count = find_numbers(buffer, length, sum, numbers);
    printf("%d\n", sum);
    output(numbers, count);
    return 0;
}

int input(int *buffer, int *length) {
    if (scanf("%d", length) != 1 || *length <= 0 || *length > NMAX) {
        return 1;
    }
    for (int *p = buffer; p - buffer < *length; p++) {
        if (scanf("%d", p) != 1) {
            return 1;
        }
    }
    return 0;
}

void output(int *buffer, int length) {
    for (int *p = buffer; p - buffer < length; p++) {
        if (p != buffer) {
            printf(" ");
        }
        printf("%d", *p);
    }
}

/* 0 по условию нечётный — в сумму не входит */
int has_even(int *buffer, int length) {
    for (int *p = buffer; p - buffer < length; p++) {
        if (*p != 0 && *p % 2 == 0) {
            return 1;
        }
    }
    return 0;
}

int sum_numbers(int *buffer, int length) {
    int sum = 0;
    for (int *p = buffer; p - buffer < length; p++) {
        if (*p != 0 && *p % 2 == 0) {
            sum += *p;
        }
    }
    return sum;
}

int find_numbers(int *buffer, int length, int number, int *numbers) {
    int count = 0;
    for (int *p = buffer; p - buffer < length; p++) {
        if (*p != 0 && number % *p == 0) {
            numbers[count++] = *p;
        }
    }
    return count;
}
```

**Как работает:** `has_even` ловит случай «чётных нет» (отдельно от суммы 0 из `2 + (-2)`). Суммируем ненулевые чётные, затем собираем делители этой суммы. Ноль в делители не берём.

#### Вариант B — индексы

```c
#include <stdio.h>

#define NMAX 10

int input(int *buffer, int *length);
void output(int *buffer, int length);
int sum_numbers(int *buffer, int length);
int find_numbers(int *buffer, int length, int number, int *numbers);
int has_even(int *buffer, int length);

int main(void) {
    int length, buffer[NMAX], numbers[NMAX];
    int sum, count;

    if (input(buffer, &length) != 0 || !has_even(buffer, length)) {
        printf("n/a");
        return 0;
    }

    sum = sum_numbers(buffer, length);
    count = find_numbers(buffer, length, sum, numbers);
    printf("%d\n", sum);
    output(numbers, count);
    return 0;
}

int input(int *buffer, int *length) {
    if (scanf("%d", length) != 1 || *length <= 0 || *length > NMAX) {
        return 1;
    }
    for (int i = 0; i < *length; i++) {
        if (scanf("%d", &buffer[i]) != 1) {
            return 1;
        }
    }
    return 0;
}

void output(int *buffer, int length) {
    for (int i = 0; i < length; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", buffer[i]);
    }
}

int has_even(int *buffer, int length) {
    for (int i = 0; i < length; i++) {
        if (buffer[i] != 0 && buffer[i] % 2 == 0) {
            return 1;
        }
    }
    return 0;
}

int sum_numbers(int *buffer, int length) {
    int sum = 0;
    for (int i = 0; i < length; i++) {
        if (buffer[i] != 0 && buffer[i] % 2 == 0) {
            sum += buffer[i];
        }
    }
    return sum;
}

int find_numbers(int *buffer, int length, int number, int *numbers) {
    int count = 0;
    for (int i = 0; i < length; i++) {
        if (buffer[i] != 0 && number % buffer[i] == 0) {
            numbers[count++] = buffer[i];
        }
    }
    return count;
}
```

**Как работает:** для `4 3 9 0 1 2 0 2 7 -1` чётные `4+2+2=8` (нули отброшены). Делители 8: `4, 1, 2, 2, -1`.

```bash
gcc -std=c11 -Wall -Werror -Wextra key9part1.c -o key9part1
printf "10\n4 3 9 0 1 2 0 2 7 -1\n" | ./key9part1
# 8
# 4 1 2 2 -1

git add key9part1.c
git commit -m "Quest 6: key9part1"
git push origin develop
```

---

### Quest 7 — `cycle_shift.c` (T06D09)

**Суть:** `n`, массив из `n`, сдвиг `c`. `c > 0` — влево, `c < 0` — вправо. Без `stdlib.h`. Max n = 10.

#### Вариант A — через временный буфер и указатели

```c
#include <stdio.h>

#define NMAX 10

int input(int *a, int *n, int *c);
void output(int *a, int n);
void cycle_shift(int *a, int n, int c);

int main(void) {
    int n, c, data[NMAX];

    if (input(data, &n, &c) != 0) {
        printf("n/a");
        return 0;
    }
    cycle_shift(data, n, c);
    output(data, n);
    return 0;
}

int input(int *a, int *n, int *c) {
    if (scanf("%d", n) != 1 || *n <= 0 || *n > NMAX) {
        return 1;
    }
    for (int *p = a; p - a < *n; p++) {
        if (scanf("%d", p) != 1) {
            return 1;
        }
    }
    if (scanf("%d", c) != 1) {
        return 1;
    }
    return 0;
}

void output(int *a, int n) {
    for (int *p = a; p - a < n; p++) {
        if (p != a) {
            printf(" ");
        }
        printf("%d", *p);
    }
}

void cycle_shift(int *a, int n, int c) {
    int tmp[NMAX];
    int shift;

    shift = c % n;
    if (shift < 0) {
        shift += n;
    }

    for (int i = 0; i < n; i++) {
        tmp[i] = a[(i + shift) % n];
    }
    for (int *p = a; p - a < n; p++) {
        *p = tmp[p - a];
    }
}
```

**Как работает:** нормализуем `c` в диапазон `[0; n)` (`%` + поправка для отрицательных). Новый массив: элемент `i` берётся из старого `(i + shift) % n` — это сдвиг влево. Копируем обратно.

#### Вариант B — сдвиг на один шаг в цикле (индексы)

```c
#include <stdio.h>

#define NMAX 10

int input(int *a, int *n, int *c);
void output(int *a, int n);
void cycle_shift(int *a, int n, int c);
void shift_left_once(int *a, int n);

int main(void) {
    int n, c, data[NMAX];

    if (input(data, &n, &c) != 0) {
        printf("n/a");
        return 0;
    }
    cycle_shift(data, n, c);
    output(data, n);
    return 0;
}

int input(int *a, int *n, int *c) {
    if (scanf("%d", n) != 1 || *n <= 0 || *n > NMAX) {
        return 1;
    }
    for (int i = 0; i < *n; i++) {
        if (scanf("%d", &a[i]) != 1) {
            return 1;
        }
    }
    if (scanf("%d", c) != 1) {
        return 1;
    }
    return 0;
}

void output(int *a, int n) {
    for (int i = 0; i < n; i++) {
        if (i > 0) {
            printf(" ");
        }
        printf("%d", a[i]);
    }
}

void shift_left_once(int *a, int n) {
    int first = a[0];
    for (int i = 0; i < n - 1; i++) {
        a[i] = a[i + 1];
    }
    a[n - 1] = first;
}

void cycle_shift(int *a, int n, int c) {
    int shift = c % n;
    if (shift < 0) {
        shift += n;
    }
    for (int k = 0; k < shift; k++) {
        shift_left_once(a, n);
    }
}
```

**Как работает:** один левый сдвиг — первый элемент уходит в конец. Повторяем `shift` раз. Для примера `c = 2`: `4 3 9 ...` → `9 0 1 2 0 2 7 -1 4 3`.

```bash
gcc -std=c11 -Wall -Werror -Wextra cycle_shift.c -o cycle_shift
printf "10\n4 3 9 0 1 2 0 2 7 -1\n2\n" | ./cycle_shift
# 9 0 1 2 0 2 7 -1 4 3

git add cycle_shift.c
git commit -m "Quest 7: cycle_shift"
git push origin develop
```

---

### Quest 8* — `key9part2.c` (T06D09, бонус)

**Суть:** два больших числа как массивы цифр (каждая 0…9, длина ≤ 100) → сумма и разность. Если вычитаемое больше уменьшаемого — во второй строке `n/a`. Без `stdlib.h`. Ошибка ввода → `n/a`.

#### Вариант A — сложение/вычитание «в столбик»

```c
#include <stdio.h>

#define LEN 100

int input(int *num, int *len);
void output(int *num, int len);
void sum(int *a, int n1, int *b, int n2, int *res, int *nres);
void sub(int *a, int n1, int *b, int n2, int *res, int *nres);
int compare(int *a, int n1, int *b, int n2);

int main(void) {
    int a[LEN], b[LEN], res[LEN + 1];
    int n1, n2, nres;

    if (input(a, &n1) != 0 || input(b, &n2) != 0) {
        printf("n/a");
        return 0;
    }

    sum(a, n1, b, n2, res, &nres);
    output(res, nres);
    printf("\n");

    if (compare(a, n1, b, n2) < 0) {
        printf("n/a");
    } else {
        sub(a, n1, b, n2, res, &nres);
        output(res, nres);
    }
    return 0;
}

int input(int *num, int *len) {
    char ch;
    *len = 0;
    while (1) {
        int d;
        if (scanf("%d%c", &d, &ch) != 2) {
            return 1;
        }
        if (d < 0 || d > 9 || *len >= LEN) {
            return 1;
        }
        num[(*len)++] = d;
        if (ch == '\n') {
            break;
        }
        if (ch != ' ') {
            return 1;
        }
    }
    return (*len == 0);
}

void output(int *num, int len) {
    int start = 0;
    while (start < len - 1 && num[start] == 0) {
        start++;
    }
    for (int i = start; i < len; i++) {
        if (i > start) {
            printf(" ");
        }
        printf("%d", num[i]);
    }
}

int compare(int *a, int n1, int *b, int n2) {
    int i = 0, j = 0;
    while (i < n1 - 1 && a[i] == 0) {
        i++;
    }
    while (j < n2 - 1 && b[j] == 0) {
        j++;
    }
    if (n1 - i != n2 - j) {
        return (n1 - i > n2 - j) ? 1 : -1;
    }
    for (; i < n1; i++, j++) {
        if (a[i] != b[j]) {
            return (a[i] > b[j]) ? 1 : -1;
        }
    }
    return 0;
}

void sum(int *a, int n1, int *b, int n2, int *res, int *nres) {
    int tmp[LEN + 1];
    int i = n1 - 1, j = n2 - 1, k = 0, carry = 0;

    while (i >= 0 || j >= 0 || carry) {
        int s = carry;
        if (i >= 0) {
            s += a[i--];
        }
        if (j >= 0) {
            s += b[j--];
        }
        tmp[k++] = s % 10;
        carry = s / 10;
    }
    *nres = k;
    for (int t = 0; t < k; t++) {
        res[t] = tmp[k - 1 - t];
    }
}

void sub(int *a, int n1, int *b, int n2, int *res, int *nres) {
    int tmp[LEN];
    int i = n1 - 1, j = n2 - 1, k = 0, borrow = 0;

    while (i >= 0) {
        int d = a[i] - borrow;
        if (j >= 0) {
            d -= b[j--];
        }
        if (d < 0) {
            d += 10;
            borrow = 1;
        } else {
            borrow = 0;
        }
        tmp[k++] = d;
        i--;
    }
    *nres = k;
    for (int t = 0; t < k; t++) {
        res[t] = tmp[k - 1 - t];
    }
}
```

**Как работает:** цифры храним слева направо (старший разряд в `[0]`). Сложение/вычитание идём с конца, как в столбик; перенос/`borrow` — обычная школьная арифметика. `compare` решает, можно ли вычитать. `output` режет ведущие нули.

#### Вариант B — тот же алгоритм, без постфиксных `--` в выражениях

```c
#include <stdio.h>

#define LEN 100

int input(int *num, int *len);
void output(int *num, int len);
void sum(int *a, int n1, int *b, int n2, int *res, int *nres);
void sub(int *a, int n1, int *b, int n2, int *res, int *nres);
int compare(int *a, int n1, int *b, int n2);

int main(void) {
    int a[LEN], b[LEN], res[LEN + 1];
    int n1, n2, nres;

    if (input(a, &n1) != 0 || input(b, &n2) != 0) {
        printf("n/a");
        return 0;
    }

    sum(a, n1, b, n2, res, &nres);
    output(res, nres);
    printf("\n");

    if (compare(a, n1, b, n2) < 0) {
        printf("n/a");
    } else {
        sub(a, n1, b, n2, res, &nres);
        output(res, nres);
    }
    return 0;
}

int input(int *num, int *len) {
    char ch;
    *len = 0;
    for (;;) {
        int d;
        if (scanf("%d%c", &d, &ch) != 2) {
            return 1;
        }
        if (d < 0 || d > 9 || *len >= LEN) {
            return 1;
        }
        num[*len] = d;
        (*len)++;
        if (ch == '\n') {
            break;
        }
        if (ch != ' ') {
            return 1;
        }
    }
    return (*len == 0);
}

void output(int *num, int len) {
    int start = 0;
    while (start < len - 1 && num[start] == 0) {
        start++;
    }
    for (int i = start; i < len; i++) {
        if (i > start) {
            printf(" ");
        }
        printf("%d", num[i]);
    }
}

int compare(int *a, int n1, int *b, int n2) {
    int ia = 0, ib = 0;
    while (ia < n1 - 1 && a[ia] == 0) {
        ia++;
    }
    while (ib < n2 - 1 && b[ib] == 0) {
        ib++;
    }
    if ((n1 - ia) != (n2 - ib)) {
        return ((n1 - ia) > (n2 - ib)) ? 1 : -1;
    }
    while (ia < n1) {
        if (a[ia] != b[ib]) {
            return (a[ia] > b[ib]) ? 1 : -1;
        }
        ia++;
        ib++;
    }
    return 0;
}

void sum(int *a, int n1, int *b, int n2, int *res, int *nres) {
    int tmp[LEN + 1];
    int i = n1 - 1, j = n2 - 1, k = 0, carry = 0;

    while (i >= 0 || j >= 0 || carry != 0) {
        int s = carry;
        if (i >= 0) {
            s += a[i];
            i--;
        }
        if (j >= 0) {
            s += b[j];
            j--;
        }
        tmp[k] = s % 10;
        carry = s / 10;
        k++;
    }
    *nres = k;
    for (int t = 0; t < k; t++) {
        res[t] = tmp[k - 1 - t];
    }
}

void sub(int *a, int n1, int *b, int n2, int *res, int *nres) {
    int tmp[LEN];
    int i = n1 - 1, j = n2 - 1, k = 0, borrow = 0;

    while (i >= 0) {
        int d = a[i] - borrow;
        if (j >= 0) {
            d -= b[j];
            j--;
        }
        if (d < 0) {
            d += 10;
            borrow = 1;
        } else {
            borrow = 0;
        }
        tmp[k] = d;
        k++;
        i--;
    }
    *nres = k;
    for (int t = 0; t < k; t++) {
        res[t] = tmp[k - 1 - t];
    }
}
```

**Как работает:** то же столбиковое сложение/вычитание. `output` проще: находим первый ненулевой индекс `start` и печатаем с пробелами от него.

```bash
gcc -std=c11 -Wall -Werror -Wextra key9part2.c -o key9part2
printf "1 9 4 4 6 7 4 4 0 7 3 7 0 9 5 5 1 6 1\n2 9\n" | ./key9part2
# 1 9 4 4 6 7 4 4 0 7 3 7 0 9 5 5 1 9 0
# 1 9 4 4 6 7 4 4 0 7 3 7 0 9 5 5 1 3 2

printf "0 1 0\n0 0 1\n" | ./key9part2
# 1 1
# 9

git add key9part2.c
git commit -m "Quest 8: key9part2"
git push origin develop
```

---

## Шпаргалка по файлам

| Квест | Репозиторий | Файл | Важно |
|-------|-------------|------|--------|
| 1 | T05D08 | `maxmin.c` | починить указатели, структуру не ломать |
| 2 | T05D08 | `squaring.c` | `&n`, квадрат на месте, n ≤ 10 |
| 3 | T05D08 | `stat.c` | mean/var с `/n`, `%.6f` |
| 4 | T05D08 | `search.c` | первое подходящее, `-lm`, n ≤ 30 |
| 5 | T06D09 | `sort.c` | ровно 10 чисел, без `stdlib.h` |
| 6 | T06D09 | `key9part1.c` | 0 — нечётный; нет чётных → `n/a` |
| 7 | T06D09 | `cycle_shift.c` | `c%n`, отрицательный = вправо |
| 8* | T06D09 | `key9part2.c` | длинная арифметика цифрами |

Перед сдачей:

```bash
cp ../materials/linters/.clang-format .
clang-format -n *.c
```
