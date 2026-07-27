# Многофайловые проекты, символы и строки на C

Краткий разбор квестов **D09T06** (Level 3: Room 1 + Room 2).
Стандарт **C11**, компилятор `gcc` с флагами `-Wall -Werror -Wextra`, стиль **Google** (`IndentWidth: 4`, `ColumnLimit: 110`).
После каждого квеста нужно сделать commit и push исходников из `src/` в ветку **`develop`**.

**Общие правила Werther и чеклиста.**
Функция **`system()`** и аналогичные вызовы к ядру **запрещены** во всех заданиях.
Код лежит в `src/`, разработка ведётся в ветке `develop`, бинарники / `.o` / `.a` / `.so` пушить нельзя.
Вещественные числа, если не сказано иначе, печатать с **двумя** знаками после запятой через пробел (`%.2lf`).
Разрешённые стандартные библиотеки для строковых квестов: только `stdio.h` и `stdlib.h` (без `string.h`).
Для модулей Room 1 обычно нужны ещё `math.h` (и при динамической библиотеке — `dlfcn.h`).

Связанные материалы: [C_СПРАВОЧНИК](C_СПРАВОЧНИК.md), [Динамическая_память_и_матрицы_на_C](Динамическая_память_и_матрицы_на_C.md).

В решениях приведены **два варианта одинаковой сложности**. Вариант A чаще идёт через индексы массива. Вариант B чаще через указательную арифметику. Сдавать можно любой один вариант.

**Структурное правило.** У каждой функции должен быть один вход и один выход: один `return` в конце тела, ошибки — через флаги/`ok`, без ранних `return` посередине.

**Про `.h` файлы.** Да, заголовочные файлы в этом проекте **есть и нужны**. Часть уже лежит в репозитории (`data_io.h`, `data_stat.h`, `data_process.h`, `decision.h` и т. д.) — их нужно подключать через `#include`. Для Room 2 ты **сама создаёшь** `s21_string.h`. Без `.h` многофайловый проект по задумке архитектора не собрать.

---

## 0. Теория

### Многофайловый проект

Большую программу делят на **модули**: каждый `.c` — реализация, каждый `.h` — «витрина» (прототипы, константы, макросы). Компилятор сначала делает из каждого `.c` объектный файл `.o`, затем **линковщик** склеивает их в один исполняемый файл.

```
data_io.c  ──► data_io.o  ──┐
data_stat.c ─► data_stat.o ─┼──► Quest_3
data_process.c ► … .o ──────┤
main_….c ────► main.o ──────┘
```

### Заголовочные файлы и include guard

`#include "file.h"` вставляет содержимое заголовка в место директивы. Чтобы один и тот же `.h` не попал в единицу трансляции дважды, пишут **include guard**:

```c
#ifndef DATA_STAT_H
#define DATA_STAT_H

double max(double *data, int n);
/* ... */

#endif
```

Кавычки `"..."` — искать рядом / по путям проекта. Угловые `<...>` — системные заголовки.

### Препроцессор

До компиляции препроцессор обрабатывает `#include`, `#define`, `#ifdef` / `#ifndef` / `#endif`. Макросы вроде `USE_DYNAMIC` позволяют одной и той же программе собираться по-разному (обычная линковка / `dlopen`).

### Make и Makefile

**Make** читает `Makefile` и собирает цели. Стандартный минимум:

| Цель | Смысл |
|------|--------|
| `all` | собрать основной бинарник |
| `clean` | удалить `.o`, библиотеки, бинарники |
| `rebuild` | `clean` + `all` |

Пути в Makefile — **относительно каталога, где лежит Makefile** (у тебя это часто `src/main_executable_module/`).

Типичная схема:

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c11
BUILD_DIR = ../../build

all: $(BUILD_DIR)/Quest_3

$(BUILD_DIR)/Quest_3: ...
	$(CC) $(CFLAGS) ... -o $@ -lm

clean:
	rm -f *.o ../../build/Quest_3

rebuild: clean all
```

### Статическая библиотека `.a`

Архив объектных файлов. Код **вшивается** в бинарник на этапе линковки.

```bash
gcc -c data_stat.c -o data_stat.o
ar rcs data_stat.a data_stat.o
gcc main.o ... data_stat.a -o Quest_4 -lm
```

### Динамическая библиотека `.so`

Подгружается при запуске (или через `dlopen`). Код **не копируется** внутрь бинарника целиком.

```bash
gcc -fPIC -c data_process.c -o data_process.o
gcc -shared -o ../../build/data_process.so data_process.o
```

В коде при макросе `USE_DYNAMIC`:

```c
void *handle = dlopen("./data_process.so", RTLD_LAZY);
/* dlsym → указатель на normalization */
/* ... работа ... */
dlclose(handle);
```

### Символы и строки в C

Строка — массив `char`, заканчивающийся `'\0'`.

| Понятие | Пример |
|---------|--------|
| Символ | `'A'`, `'\\n'` — тип `char` / `int` в функциях |
| Строковый литерал | `"hello"` — массив из 6 байт (`h e l l o \\0`) |
| Длина | число символов **без** `\\0` |

Без `string.h` все операции делаешь сама: цикл до `\\0`, копирование побайтно, сравнение код за кодом.

### Свои аналоги string.h

| Функция | Назначение |
|---------|------------|
| `s21_strlen` | длина строки |
| `s21_strcmp` | сравнение: `<0`, `0`, `>0` |
| `s21_strcpy` | копирование `src → dest` |
| `s21_strcat` | дописать `src` в конец `dest` |
| `s21_strchr` | первое вхождение символа |
| `s21_strstr` | первое вхождение подстроки |

### Модульные тесты

Тест — отдельная функция `имя_test`, которая гоняет **не меньше 3** наборов: норма, край, «странное». В stdout по каждому кейсу: вход, выход, `SUCCESS`/`FAIL`.

### Стиль и проверка

```bash
# скопируй materials/linters/.clang-format в src/
clang-format -n src/path/to/file.c

gcc -std=c11 -Wall -Werror -Wextra ... -o prog -lm
# Linux:
valgrind --tool=memcheck --leak-check=yes ./prog
# macOS:
leaks -atExit -- ./prog | grep LEAK:
```

---

## Структура репозитория (ожидаемая)

```
D09T06/   (или T09D06 / как назван репозиторий)
├── build/                          ← бинарники сюда, в git НЕ пушить
├── materials/
├── src/
│   ├── data_libs/
│   │   ├── data_io.c / .h
│   │   ├── data_io_macro.h
│   │   └── data_stat.c / .h
│   ├── data_module/
│   │   ├── data_module_entry.c
│   │   ├── data_process.c / .h
│   ├── yet_another_decision_module/
│   │   ├── yet_another_decision_module_entry.c
│   │   ├── decision.c / .h
│   ├── main_executable_module/
│   │   ├── main_executable_module.c
│   │   └── Makefile
│   └── s21_string/
│       ├── s21_string.h / .c
│       ├── s21_string_test.c
│       ├── text_processor.c
│       └── Makefile
└── README.md
```

Файлы с кодом из условия уже лежат в репозитории, но **модули «сломанные» / недоделанные**. Разбивку на файлы **не менять**. Нужно дописать `#include`, реализации, константы и Makefile.

```bash
git checkout -b develop   # если ещё нет
cd src
```

---

## Решения задач — Room 1

### Quest 1 — Modules (`data_module`)

**Суть.** Собрать рабочий конвейер: ввод массива → нормализация в `[0; 1]` → вывод с `%.2lf`. Переиспользовать `data_libs` (`input`/`output`, `max`/`min`).

**Вход / выход (пример):**

| Вход | Выход |
|------|-------|
| `5` затем `1 2 3 4 5` | `0.00 0.25 0.50 0.75 1.00` |

Нормализация: `x' = (x - min) / (max - min)`. Если `max ≈ min` — ошибка (`ERROR`).

#### Вариант A. Индексы + явные проверки

`src/data_libs/data_io.h`

```c
#ifndef DATA_IO_H
#define DATA_IO_H

void input(double **data, int *n);
void output(double *data, int n);

#endif
```

`src/data_libs/data_io.c`

```c
#include "data_io.h"

#include <stdio.h>
#include <stdlib.h>

/* читает n и массив double в динамическую память */
void input(double **data, int *n) {
    int i;
    int ok;

    ok = 1;
    *data = NULL;
    *n = 0;
    if (scanf("%d", n) != 1 || *n <= 0) {
        ok = 0;
        *n = 0;
    }
    if (ok == 1) {
        *data = (double *)malloc((size_t)(*n) * sizeof(double));
        if (*data == NULL) {
            ok = 0;
            *n = 0;
        }
    }
    i = 0;
    while (ok == 1 && i < *n) {
        if (scanf("%lf", &(*data)[i]) != 1) {
            free(*data);
            *data = NULL;
            *n = 0;
            ok = 0;
        }
        i++;
    }
}

/* печатает массив через пробел с двумя знаками */
void output(double *data, int n) {
    int i;

    if (data != NULL && n > 0) {
        for (i = 0; i < n; i++) {
            if (i > 0) {
                printf(" ");
            }
            printf("%.2lf", data[i]);
        }
    }
}
```

`src/data_libs/data_stat.h`

```c
#ifndef DATA_STAT_H
#define DATA_STAT_H

double max(double *data, int n);
double min(double *data, int n);
double mean(double *data, int n);
double variance(double *data, int n);
void sort(double *data, int n);

#endif
```

`src/data_libs/data_stat.c`

```c
#include "data_stat.h"

#include <math.h>

/* максимум массива */
double max(double *data, int n) {
    int i;
    double res;

    res = data[0];
    for (i = 1; i < n; i++) {
        if (data[i] > res) {
            res = data[i];
        }
    }
    return res;
}

/* минимум массива */
double min(double *data, int n) {
    int i;
    double res;

    res = data[0];
    for (i = 1; i < n; i++) {
        if (data[i] < res) {
            res = data[i];
        }
    }
    return res;
}

/* среднее арифметическое */
double mean(double *data, int n) {
    int i;
    double sum;

    sum = 0.0;
    for (i = 0; i < n; i++) {
        sum += data[i];
    }
    return sum / (double)n;
}

/* дисперсия (деление на n) */
double variance(double *data, int n) {
    int i;
    double m;
    double sum;

    m = mean(data, n);
    sum = 0.0;
    for (i = 0; i < n; i++) {
        sum += (data[i] - m) * (data[i] - m);
    }
    return sum / (double)n;
}

/* пузырьковая сортировка по возрастанию */
void sort(double *data, int n) {
    int i;
    int j;
    double tmp;

    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - 1 - i; j++) {
            if (data[j] > data[j + 1]) {
                tmp = data[j];
                data[j] = data[j + 1];
                data[j + 1] = tmp;
            }
        }
    }
}
```

`src/data_module/data_process.h`

```c
#ifndef DATA_PROCESS_H
#define DATA_PROCESS_H

#define EPS 1E-6

int normalization(double *data, int n);

#endif
```

`src/data_module/data_process.c`

```c
#include "data_process.h"

#include <math.h>

#include "../data_libs/data_stat.h"

/* нормирует данные в [0; 1]; 1 — успех, 0 — ошибка */
int normalization(double *data, int n) {
    int i;
    int ok;
    double max_value;
    double min_value;
    double range;

    ok = 1;
    if (data == NULL || n <= 0) {
        ok = 0;
    }
    if (ok == 1) {
        max_value = max(data, n);
        min_value = min(data, n);
        range = max_value - min_value;
        if (fabs(range) <= EPS) {
            ok = 0;
        }
    }
    if (ok == 1) {
        for (i = 0; i < n; i++) {
            data[i] = (data[i] - min_value) / range;
        }
    }
    return ok;
}
```

`src/data_module/data_module_entry.c`

```c
#include <stdio.h>
#include <stdlib.h>

#include "../data_libs/data_io.h"
#include "data_process.h"

#define ERROR_MSG "ERROR"

int main(void) {
    double *data;
    int n;
    int ok;

    data = NULL;
    n = 0;
    ok = 1;
    input(&data, &n);
    if (data == NULL || n <= 0) {
        ok = 0;
    }
    if (ok == 1) {
        ok = normalization(data, n);
    }
    if (ok == 1) {
        output(data, n);
        printf("\n");
    } else {
        printf("%s\n", ERROR_MSG);
    }
    if (data != NULL) {
        free(data);
    }
    return 0;
}
```

**Что писать в консоли**

```bash
mkdir -p ../../build
cd src/data_module
gcc -std=c11 -Wall -Werror -Wextra \
  data_module_entry.c data_process.c \
  ../data_libs/data_io.c ../data_libs/data_stat.c \
  -o ../../build/Quest_1 -lm
../../build/Quest_1
# ввод:
# 5
# 1 2 3 4 5
# ожидание: 0.00 0.25 0.50 0.75 1.00
```

**Как работает.** `input` читает размер, выделяет массив и заполняет его числами. `normalization` находит min/max и растягивает значения на отрезок от 0 до 1. Если все числа одинаковые, делить не на что — печатается `ERROR`. `output` печатает результат с двумя знаками. Память в конце освобождается через `free`.

#### Вариант B. Указатели + тот же контракт

Отличие только в стиле обхода массивов (через указатели). Заголовки те же.

Фрагмент `normalization` (остальные файлы как в A, с заменой циклов на указатели где удобно):

```c
/* нормирует данные в [0; 1]; 1 — успех, 0 — ошибка */
int normalization(double *data, int n) {
    double *p;
    int ok;
    double max_value;
    double min_value;
    double range;

    ok = 1;
    if (data == NULL || n <= 0) {
        ok = 0;
    }
    if (ok == 1) {
        max_value = max(data, n);
        min_value = min(data, n);
        range = max_value - min_value;
        if (fabs(range) <= EPS) {
            ok = 0;
        }
    }
    if (ok == 1) {
        for (p = data; p - data < n; p++) {
            *p = (*p - min_value) / range;
        }
    }
    return ok;
}
```

В `mean` / `variance` то же самое: `for (p = data; p - data < n; p++)`.

**Консоль** — та же команда `gcc`, что в варианте A.

**Как работает.** Логика идентична варианту A. Разница только в том, как двигаешься по массиву: не `data[i]`, а указатель `p`, который сдвигается на следующий `double`.

---

### Quest 2 — Modules II (`yet_another_decision_module`)

**Суть.** Модуль принимает массив и отвечает `YES`/`NO`.  
`make_decision` возвращает 1, если:
1. данные удовлетворяют **правилу трёх сигм** (уже есть идея в `decision.c`: все точки в `[mean ± 3σ]`);
2. среднее **не меньше** обратного золотого сечения ≈ **0.618**.

Важно: в заготовке часто стоит `#define GOLDEN_RATIO 0.666` — это **неверно** для условия. Нужно **~0.618**.

**Пример:** вход `4` / `1 2 3 4` → `YES`.

#### Вариант A

`src/yet_another_decision_module/decision.h`

```c
#ifndef DECISION_H
#define DECISION_H

#define GOLDEN_RATIO 0.618

int make_decision(double *data, int n);

#endif
```

`src/yet_another_decision_module/decision.c`

```c
#include "decision.h"

#include <math.h>

#include "../data_libs/data_stat.h"

/* 1 — данные «нормальны» по 3σ и mean >= 0.618 */
int make_decision(double *data, int n) {
    int ok;
    double m;
    double sigma;
    double max_value;
    double min_value;

    ok = 1;
    if (data == NULL || n <= 0) {
        ok = 0;
    }
    if (ok == 1) {
        m = mean(data, n);
        if (m < GOLDEN_RATIO) {
            ok = 0;
        }
    }
    if (ok == 1) {
        sigma = sqrt(variance(data, n));
        max_value = max(data, n);
        min_value = min(data, n);
        if (max_value > m + 3.0 * sigma || min_value < m - 3.0 * sigma) {
            ok = 0;
        }
    }
    return ok;
}
```

`src/yet_another_decision_module/yet_another_decision_module_entry.c`

```c
#include <stdio.h>
#include <stdlib.h>

#include "../data_libs/data_io.h"
#include "decision.h"

int main(void) {
    double *data;
    int n;
    int decision;

    data = NULL;
    n = 0;
    input(&data, &n);
    if (data == NULL || n <= 0) {
        printf("NO\n");
    } else {
        decision = make_decision(data, n);
        printf("%s\n", decision ? "YES" : "NO");
        free(data);
    }
    return 0;
}
```

**Что писать в консоли**

```bash
cd src/yet_another_decision_module
gcc -std=c11 -Wall -Werror -Wextra \
  yet_another_decision_module_entry.c decision.c \
  ../data_libs/data_io.c ../data_libs/data_stat.c \
  -o ../../build/Quest_2 -lm
../../build/Quest_2
# 4
# 1 2 3 4
# YES
```

**Как работает.** После ввода считается среднее. Если оно меньше 0.618 — сразу `NO`. Иначе считается σ из дисперсии и проверяется, что минимум и максимум лежат в интервале трёх сигм. Для `1 2 3 4` среднее 2.5, разброс небольшой — ответ `YES`.

#### Вариант B

Тот же контракт; в `make_decision` сначала собираешь все статистики, потом одним условием выставляешь флаг:

```c
/* 1 — данные «нормальны» по 3σ и mean >= 0.618 */
int make_decision(double *data, int n) {
    int ok;
    double m;
    double sigma;
    double max_value;
    double min_value;
    double upper;
    double lower;

    ok = 0;
    if (data != NULL && n > 0) {
        m = mean(data, n);
        sigma = sqrt(variance(data, n));
        max_value = max(data, n);
        min_value = min(data, n);
        upper = m + 3.0 * sigma;
        lower = m - 3.0 * sigma;
        if (m >= GOLDEN_RATIO && max_value <= upper && min_value >= lower) {
            ok = 1;
        }
    }
    return ok;
}
```

**Консоль** — как в варианте A.

**Как работает.** Все числа считаются заранее, решение принимается одним составным условием. Результат тот же, читать чуть проще.

---

### Quest 3 — Makefile (`main_executable_module`)

**Суть.** Доработать `main_executable_module.c` и написать `src/main_executable_module/Makefile` с целями `all`, `clean`, `rebuild`. Бинарник: **`../../build/Quest_3`** (т. е. `build/Quest_3` в корне репо).

Программа печатает этапы: сырые данные → нормализация → сортировка → решение `YES`/`NO`.

#### Вариант A. Классический Makefile через `.o`

`src/main_executable_module/main_executable_module.c`

```c
#include <stdio.h>
#include <stdlib.h>

#ifdef USE_MACRO_IO
#include "../data_libs/data_io_macro.h"
#else
#include "../data_libs/data_io.h"
#endif

#include "../data_libs/data_stat.h"
#ifndef USE_DYNAMIC
#include "../data_module/data_process.h"
#endif
#include "../yet_another_decision_module/decision.h"

#ifdef USE_DYNAMIC
#include <dlfcn.h>
typedef int (*normalization_fn)(double *, int);
#define LIB_PATH "./data_process.so"
#endif

#define ERROR_MSG "ERROR"

int main(void) {
    double *data;
    int n;
    int ok;
#ifdef USE_DYNAMIC
    void *handle;
    normalization_fn normalization;
#endif

    data = NULL;
    n = 0;
    ok = 1;
#ifdef USE_DYNAMIC
    handle = NULL;
    normalization = NULL;
    handle = dlopen(LIB_PATH, RTLD_LAZY);
    if (handle == NULL) {
        ok = 0;
    }
    if (ok == 1) {
        normalization = (normalization_fn)dlsym(handle, "normalization");
        if (normalization == NULL) {
            ok = 0;
        }
    }
#endif

    if (ok == 1) {
        printf("LOAD DATA...\n");
        input(&data, &n);
        if (data == NULL || n <= 0) {
            ok = 0;
        }
    }
    if (ok == 1) {
        printf("RAW DATA:\n\t");
        output(data, n);
        printf("\n");
        printf("NORMALIZED DATA:\n\t");
        if (normalization(data, n) == 0) {
            ok = 0;
        }
    }
    if (ok == 1) {
        output(data, n);
        printf("\n");
        printf("SORTED NORMALIZED DATA:\n\t");
        sort(data, n);
        output(data, n);
        printf("\n");
        printf("FINAL DECISION:\n\t");
        printf("%s\n", make_decision(data, n) ? "YES" : "NO");
    } else {
        printf("%s\n", ERROR_MSG);
    }
    if (data != NULL) {
        free(data);
    }
#ifdef USE_DYNAMIC
    if (handle != NULL) {
        dlclose(handle);
    }
#endif
    return 0;
}
```

`src/main_executable_module/Makefile` (вариант A)

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c11
BUILD_DIR = ../../build
TARGET = $(BUILD_DIR)/Quest_3

IO_SRC = ../data_libs/data_io.c
STAT_SRC = ../data_libs/data_stat.c
PROC_SRC = ../data_module/data_process.c
DEC_SRC = ../yet_another_decision_module/decision.c
MAIN_SRC = main_executable_module.c

SRCS = $(MAIN_SRC) $(IO_SRC) $(STAT_SRC) $(PROC_SRC) $(DEC_SRC)
OBJS = $(SRCS:.c=.o)

.PHONY: all clean rebuild

all: $(TARGET)

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

$(TARGET): $(BUILD_DIR) $(OBJS)
	$(CC) $(CFLAGS) $(OBJS) -o $(TARGET) -lm

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS)
	rm -f $(TARGET)
	rm -f data_stat.a data_stat.o
	rm -f $(BUILD_DIR)/Quest_4 $(BUILD_DIR)/Quest_5
	rm -f $(BUILD_DIR)/data_process.so

rebuild: clean all
```

**Что писать в консоли**

```bash
cd src/main_executable_module
make all
# или: make rebuild
../../build/Quest_3
# пример ввода:
# 5
# 1 2 3 4 5
make clean
```

**Как работает.** Make компилирует каждый `.c` в `.o`, затем линкует их в `build/Quest_3` с библиотекой математики `-lm`. Цель `clean` удаляет артефакты, `rebuild` делает полную пересборку. Сама программа показывает пайплайн обработки данных по шагам.

#### Вариант B. Сборка одним вызовом gcc в рецепте

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c11
BUILD_DIR = ../../build
TARGET = $(BUILD_DIR)/Quest_3

SRCS = main_executable_module.c \
       ../data_libs/data_io.c \
       ../data_libs/data_stat.c \
       ../data_module/data_process.c \
       ../yet_another_decision_module/decision.c

.PHONY: all clean rebuild

all: $(TARGET)

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

$(TARGET): $(BUILD_DIR) $(SRCS)
	$(CC) $(CFLAGS) $(SRCS) -o $(TARGET) -lm

clean:
	rm -f $(TARGET) $(BUILD_DIR)/Quest_4 $(BUILD_DIR)/Quest_5
	rm -f $(BUILD_DIR)/data_process.so data_stat.a *.o
	rm -f ../data_libs/*.o ../data_module/*.o ../yet_another_decision_module/*.o

rebuild: clean all
```

**Консоль** — `make all` / `make rebuild` / `make clean`, как в A.

**Как работает.** Без промежуточных `.o` в явном виде: gcc получает сразу весь список исходников. Для автотестов этого достаточно; вариант A ближе к «учебному» Make.

---

### Bonus Quest 4 — Static Lib

**Суть.** В тот же Makefile добавить:
- цель `data_stat.a` — статическая библиотека из `data_stat`;
- цель `build_with_static` — сборка main с этой библиотекой → **`build/Quest_4`**.

`data_stat.a` в git **не** класть.

#### Вариант A. `ar rcs` + линковка `.a`

Дописать в Makefile:

```makefile
STAT_OBJ = data_stat.o
STAT_LIB = data_stat.a
TARGET4 = $(BUILD_DIR)/Quest_4

.PHONY: build_with_static

$(STAT_OBJ): $(STAT_SRC)
	$(CC) $(CFLAGS) -c $(STAT_SRC) -o $(STAT_OBJ)

$(STAT_LIB): $(STAT_OBJ)
	ar rcs $(STAT_LIB) $(STAT_OBJ)

build_with_static: $(BUILD_DIR) $(STAT_LIB)
	$(CC) $(CFLAGS) $(MAIN_SRC) $(IO_SRC) $(PROC_SRC) $(DEC_SRC) \
		$(STAT_LIB) -o $(TARGET4) -lm
```

**Что писать в консоли**

```bash
cd src/main_executable_module
make build_with_static
../../build/Quest_4
```

**Как работает.** `data_stat.c` компилируется в объектник, `ar` упаковывает его в архив `.a`. При линковке нужные символы (`max`, `mean`, …) достаются из архива и попадают внутрь `Quest_4`.

#### Вариант B. Цель с именем файла библиотеки как у условия

```makefile
TARGET4 = $(BUILD_DIR)/Quest_4

.PHONY: build_with_static

data_stat.a: ../data_libs/data_stat.c
	$(CC) $(CFLAGS) -c ../data_libs/data_stat.c -o data_stat.o
	ar rcs data_stat.a data_stat.o
	rm -f data_stat.o

build_with_static: $(BUILD_DIR) data_stat.a
	$(CC) $(CFLAGS) main_executable_module.c \
		../data_libs/data_io.c \
		../data_module/data_process.c \
		../yet_another_decision_module/decision.c \
		data_stat.a -o $(TARGET4) -lm
```

**Консоль** — `make data_stat.a` и/или `make build_with_static`.

**Как работает.** То же самое, только цель библиотеки называется ровно `data_stat.a`, как в формулировке квеста — удобно для чеклиста.

---

### Bonus Quest 5 — Dynamic Lib

**Суть.**
- цель `data_process.so` → положить `.so` в **`build/`**;
- цель `build_with_dynamic` → бинарник **`build/Quest_5`**;
- в `main` использование `.so` через макрос `USE_DYNAMIC` + `dlopen`/`dlsym`/`dlclose`.

Код `main` уже показан в Quest 3 (ветки `#ifdef USE_DYNAMIC`).

#### Вариант A

```makefile
TARGET5 = $(BUILD_DIR)/Quest_5
PROCESS_SO = $(BUILD_DIR)/data_process.so

.PHONY: build_with_dynamic

data_process.so: $(PROCESS_SO)

$(PROCESS_SO): $(PROC_SRC) ../data_module/data_process.h
	$(CC) $(CFLAGS) -fPIC -c $(PROC_SRC) -o data_process.o
	$(CC) -shared -o $(PROCESS_SO) data_process.o -lm
	rm -f data_process.o

build_with_dynamic: $(BUILD_DIR) $(PROCESS_SO)
	$(CC) $(CFLAGS) -DUSE_DYNAMIC $(MAIN_SRC) $(IO_SRC) $(STAT_SRC) $(DEC_SRC) \
		-o $(TARGET5) -lm -ldl
```

Запуск из каталога `build`, чтобы `./data_process.so` находился:

```bash
cd src/main_executable_module
make build_with_dynamic
cd ../../build
./Quest_5
```

**Как работает.** `-fPIC` и `-shared` собирают разделяемую библиотеку. Бинарник собирается **без** `data_process.c`, зато с `-ldl`. В runtime `dlopen` открывает `.so`, `dlsym` находит `normalization`, после работы `dlclose` закрывает библиотеку.

#### Вариант B. `LD_LIBRARY_PATH` и имя цели как в условии

Если путь в коде сделать `"data_process.so"` и запускать так:

```bash
cd build
LD_LIBRARY_PATH=. ./Quest_5
```

или оставить `LIB_PATH "./data_process.so"` и всегда стартовать из `build/`.

Makefile — как в A; можно назвать phony-цель так:

```makefile
.PHONY: data_process.so build_with_dynamic
```

и в рецепте копировать/писать `.so` сразу в `$(BUILD_DIR)`.

**Как работает.** Идея та же: код нормализации живёт снаружи бинарника и подключается при запуске.

---

## Решения задач — Room 2 (`s21_string`)

Общий заголовок накапливается от квеста к квесту.

### Quest 6 — Strlen

Создать `s21_string.h`, `s21_string.c`, `s21_string_test.c`, Makefile.  
Цель сборки: **`strlen_tests`** → `build/Quest_6`.  
Только `stdio.h` / `stdlib.h`. Без `string.h`.

#### Вариант A. Индексный обход

`string.h` запрещён, поэтому свой тип длины объявляем typedef'ом (без `<stddef.h>`, если materials этого не разрешают явно).

`src/s21_string/s21_string.h` (накопительный вид к концу всех квестов)

```c
#ifndef S21_STRING_H
#define S21_STRING_H

typedef unsigned long s21_size_t;

s21_size_t s21_strlen(const char *str);
int s21_strcmp(const char *s1, const char *s2);
char *s21_strcpy(char *dest, const char *src);
char *s21_strcat(char *dest, const char *src);
char *s21_strchr(const char *str, int c);
char *s21_strstr(const char *haystack, const char *needle);

#endif
```

На Quest 6 в заголовке достаточно только `s21_strlen`; остальные прототипы добавляй по мере квестов.

`src/s21_string/s21_string.c` (Quest 6)

```c
#include "s21_string.h"

/* считает число символов до '\\0' */
s21_size_t s21_strlen(const char *str) {
    s21_size_t len;

    len = 0;
    if (str != 0) {
        while (str[len] != '\0') {
            len++;
        }
    }
    return len;
}
```

`src/s21_string/s21_string_test.c` (фрагмент Quest 6)

```c
#include <stdio.h>

#include "s21_string.h"

/* гоняет s21_strlen на нескольких наборах */
void s21_strlen_test(void) {
    const char *inputs[3];
    s21_size_t expected[3];
    s21_size_t got;
    int i;

    inputs[0] = "hello";
    expected[0] = 5;
    inputs[1] = "";
    expected[1] = 0;
    inputs[2] = "a";
    expected[2] = 1;

    for (i = 0; i < 3; i++) {
        got = s21_strlen(inputs[i]);
        printf("input: \"%s\" output: %lu ", inputs[i], (unsigned long)got);
        if (got == expected[i]) {
            printf("SUCCESS\n");
        } else {
            printf("FAIL\n");
        }
    }
}

int main(void) {
    s21_strlen_test();
    return 0;
}
```

`src/s21_string/Makefile` (старт)

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c11
BUILD_DIR = ../../build

.PHONY: strlen_tests clean

strlen_tests: $(BUILD_DIR)
	$(CC) $(CFLAGS) s21_string.c s21_string_test.c -o $(BUILD_DIR)/Quest_6

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

clean:
	rm -f $(BUILD_DIR)/Quest_6 $(BUILD_DIR)/Quest_7 $(BUILD_DIR)/Quest_8 \
		$(BUILD_DIR)/Quest_9 $(BUILD_DIR)/Quest_10 $(BUILD_DIR)/Quest_11 \
		$(BUILD_DIR)/Quest_12 $(BUILD_DIR)/Quest_13
```

**Что писать в консоли**

```bash
cd src/s21_string
make strlen_tests
../../build/Quest_6
```

**Как работает.** Функция идёт по символам, пока не встретит нулевой терминатор, и считает шаги. Пустая строка `""` сразу даёт длину 0. Тест печатает вход, полученную длину и вердикт.

#### Вариант B. Движение указателя

```c
/* считает число символов до '\\0' */
s21_size_t s21_strlen(const char *str) {
    const char *p;
    s21_size_t len;

    len = 0;
    if (str != 0) {
        p = str;
        while (*p != '\0') {
            p++;
        }
        len = (s21_size_t)(p - str);
    }
    return len;
}
```

**Консоль** — та же `make strlen_tests`.

**Как работает.** Указатель бежит к концу строки; разность адресов и есть длина.

---

### Quest 7 — Strcmp

Добавить `s21_strcmp` + `s21_strcmp_test`. Цель: **`strcmp_tests`** → `Quest_7`.

#### Вариант A

В `.h`:

```c
int s21_strcmp(const char *s1, const char *s2);
```

В `.c`:

```c
/* сравнивает строки: <0, 0, >0 как в классическом strcmp */
int s21_strcmp(const char *s1, const char *s2) {
    int i;
    int result;

    i = 0;
    result = 0;
    if (s1 != 0 && s2 != 0) {
        while (s1[i] != '\0' && s2[i] != '\0' && s1[i] == s2[i]) {
            i++;
        }
        result = (int)((unsigned char)s1[i] - (unsigned char)s2[i]);
    }
    return result;
}
```

Тест (идея): равные строки → 0; `"abc"` vs `"abd"` → отрицательное; `""` vs `"x"` → отрицательное. В stdout: вход, выход, SUCCESS/FAIL. В `main` вызвать и `s21_strlen_test`, и `s21_strcmp_test` (или только новые — смотри materials; обычно оставляют оба).

В Makefile:

```makefile
strcmp_tests: $(BUILD_DIR)
	$(CC) $(CFLAGS) s21_string.c s21_string_test.c -o $(BUILD_DIR)/Quest_7
```

**Консоль**

```bash
make strcmp_tests
../../build/Quest_7
```

**Как работает.** Идём по обеим строкам, пока символы совпадают. На первом отличии (или на `\\0`) возвращаем разницу кодов. Ноль значит «строки равны».

#### Вариант B

```c
/* сравнивает строки: <0, 0, >0 как в классическом strcmp */
int s21_strcmp(const char *s1, const char *s2) {
    int result;

    result = 0;
    if (s1 != 0 && s2 != 0) {
        while (*s1 != '\0' && *s1 == *s2) {
            s1++;
            s2++;
        }
        result = (int)((unsigned char)*s1 - (unsigned char)*s2);
    }
    return result;
}
```

**Как работает.** То же сравнение, но через продвижение указателей.

---

### Quest 8 — Strcpy

Только реализация `s21_strcpy` (тесты — **не** обязательны, пока не попросят). Цель Makefile: **`strcpy`** → `Quest_8`.

#### Вариант A

```c
/* копирует строку src в dest, включая '\\0' */
char *s21_strcpy(char *dest, const char *src) {
    int i;

    i = 0;
    if (dest != 0 && src != 0) {
        while (src[i] != '\0') {
            dest[i] = src[i];
            i++;
        }
        dest[i] = '\0';
    }
    return dest;
}
```

```makefile
strcpy: $(BUILD_DIR)
	$(CC) $(CFLAGS) s21_string.c s21_string_test.c -o $(BUILD_DIR)/Quest_8
```

**Консоль:** `make strcpy` && `../../build/Quest_8`

**Как работает.** Байт за байтом копирует символы в буфер назначения и в конце ставит `\\0`. Возвращает `dest` (как стандартный `strcpy`).

#### Вариант B

```c
/* копирует строку src в dest, включая '\\0' */
char *s21_strcpy(char *dest, const char *src) {
    char *start;

    start = dest;
    if (dest != 0 && src != 0) {
        while (*src != '\0') {
            *dest = *src;
            dest++;
            src++;
        }
        *dest = '\0';
    }
    return start;
}
```

**Как работает.** Сохраняем начало `dest`, копируем через указатели, возвращаем исходный адрес буфера.

---

### Quest 9 — Strcat

Цель: **`strcat`** → `Quest_9`.

#### Вариант A

```c
/* дописывает src в конец dest */
char *s21_strcat(char *dest, const char *src) {
    int i;
    int j;

    i = 0;
    j = 0;
    if (dest != 0 && src != 0) {
        while (dest[i] != '\0') {
            i++;
        }
        while (src[j] != '\0') {
            dest[i] = src[j];
            i++;
            j++;
        }
        dest[i] = '\0';
    }
    return dest;
}
```

**Консоль:** `make strcat` → `../../build/Quest_9`

**Как работает.** Сначала находим конец `dest` (его `\\0`), затем с этого места копируем `src` и ставим новый терминатор.

#### Вариант B

```c
/* дописывает src в конец dest */
char *s21_strcat(char *dest, const char *src) {
    char *start;

    start = dest;
    if (dest != 0 && src != 0) {
        while (*dest != '\0') {
            dest++;
        }
        while (*src != '\0') {
            *dest = *src;
            dest++;
            src++;
        }
        *dest = '\0';
    }
    return start;
}
```

**Как работает.** Указатель на `dest` сдвигается к концу, туда дописывается `src`.

---

### Quest 10 — Strchr

Цель: **`strchr`** → `Quest_10`.

#### Вариант A

```c
/* первое вхождение символа c; если c == '\\0' — указатель на терминатор */
char *s21_strchr(const char *str, int c) {
    char *res;
    char ch;
    int i;
    int found;

    res = 0;
    ch = (char)c;
    i = 0;
    found = 0;
    if (str != 0) {
        while (found == 0) {
            if (str[i] == ch) {
                res = (char *)(str + i);
                found = 1;
            } else if (str[i] == '\0') {
                found = 1;
            } else {
                i++;
            }
        }
    }
    return res;
}
```

**Консоль:** `make strchr` → `../../build/Quest_10`

**Как работает.** Ищем символ слева направо. Если нашли — возвращаем адрес. Если дошли до `\\0` и искали не его — `NULL`. Поиск `\\0` должен вернуть указатель на терминатор.

#### Вариант B

```c
/* первое вхождение символа c */
char *s21_strchr(const char *str, int c) {
    char *res;
    char ch;

    res = 0;
    ch = (char)c;
    if (str != 0) {
        while (*str != '\0' && *str != ch) {
            str++;
        }
        if (*str == ch) {
            res = (char *)str;
        }
    }
    return res;
}
```

**Как работает.** Указатель бежит, пока не встретит нужный символ или конец. Условие `*str == ch` покрывает и случай поиска `\\0`.

---

### Bonus Quest 11 — Strstr

Цель: **`strstr`** → `Quest_11`.

#### Вариант A. Вложенные индексы

```c
/* первое вхождение подстроки needle в haystack */
char *s21_strstr(const char *haystack, const char *needle) {
    char *res;
    int i;
    int j;
    int ok_match;

    res = 0;
    if (haystack != 0 && needle != 0) {
        if (needle[0] == '\0') {
            res = (char *)haystack;
        } else {
            i = 0;
            while (haystack[i] != '\0' && res == 0) {
                j = 0;
                ok_match = 1;
                while (needle[j] != '\0' && ok_match == 1) {
                    if (haystack[i + j] != needle[j]) {
                        ok_match = 0;
                    }
                    j++;
                }
                if (ok_match == 1) {
                    res = (char *)(haystack + i);
                }
                i++;
            }
        }
    }
    return res;
}
```

**Консоль:** `make strstr` → `../../build/Quest_11`

**Как работает.** Для каждой позиции в `haystack` пытаемся сопоставить весь `needle`. Пустой `needle` по традиции даёт начало `haystack`. Если совпадений нет — `NULL`.

#### Вариант B. Указательный поиск

```c
/* первое вхождение подстроки needle в haystack */
char *s21_strstr(const char *haystack, const char *needle) {
    const char *h;
    const char *n;
    char *res;
    int done;

    res = 0;
    done = 0;
    if (haystack == 0 || needle == 0) {
        done = 1;
    }
    if (done == 0 && *needle == '\0') {
        res = (char *)haystack;
        done = 1;
    }
    while (done == 0 && *haystack != '\0') {
        h = haystack;
        n = needle;
        while (*h != '\0' && *n != '\0' && *h == *n) {
            h++;
            n++;
        }
        if (*n == '\0') {
            res = (char *)haystack;
            done = 1;
        } else {
            haystack++;
        }
    }
    return res;
}
```

**Как работает.** С каждой позиции запускается «параллельный» проход по `needle`. Если `needle` исчерпан — нашли вхождение.

---

### Bonus Quest 12 — Extended testing

Добавить тесты: `s21_strcpy_test`, `s21_strcat_test`, `s21_strchr_test`, `s21_strstr_test` (каждый ≥ 3 кейса). В `main` — запуск **всех** тестов. Цель: **`full_coverage_tests`** → `Quest_12`.

#### Вариант A. Отдельные функции-тесты с массивами кейсов

```c
#include <stdio.h>

#include "s21_string.h"

void s21_strlen_test(void);
void s21_strcmp_test(void);
void s21_strcpy_test(void);
void s21_strcat_test(void);
void s21_strchr_test(void);
void s21_strstr_test(void);

/* ... strlen/strcmp как раньше ... */

/* проверяет копирование строк */
void s21_strcpy_test(void) {
    char buf[64];
    const char *src;
    char *got;

    src = "hi";
    got = s21_strcpy(buf, src);
    printf("input: \"%s\" output: \"%s\" ", src, buf);
    printf("%s\n", (got == buf && s21_strcmp(buf, "hi") == 0) ? "SUCCESS" : "FAIL");

    src = "";
    got = s21_strcpy(buf, src);
    printf("input: \"%s\" output: \"%s\" ", src, buf);
    printf("%s\n", (s21_strcmp(buf, "") == 0) ? "SUCCESS" : "FAIL");

    src = "School21";
    got = s21_strcpy(buf, src);
    printf("input: \"%s\" output: \"%s\" ", src, buf);
    printf("%s\n", (s21_strcmp(buf, "School21") == 0) ? "SUCCESS" : "FAIL");
}

/* проверяет конкатенацию */
void s21_strcat_test(void) {
    char buf[64];

    s21_strcpy(buf, "ab");
    s21_strcat(buf, "cd");
    printf("input: \"ab\"+\"cd\" output: \"%s\" ", buf);
    printf("%s\n", (s21_strcmp(buf, "abcd") == 0) ? "SUCCESS" : "FAIL");

    s21_strcpy(buf, "");
    s21_strcat(buf, "x");
    printf("input: \"\"+\"x\" output: \"%s\" ", buf);
    printf("%s\n", (s21_strcmp(buf, "x") == 0) ? "SUCCESS" : "FAIL");

    s21_strcpy(buf, "a");
    s21_strcat(buf, "");
    printf("input: \"a\"+\"\" output: \"%s\" ", buf);
    printf("%s\n", (s21_strcmp(buf, "a") == 0) ? "SUCCESS" : "FAIL");
}

/* проверяет поиск символа */
void s21_strchr_test(void) {
    const char *s;
    char *got;

    s = "hello";
    got = s21_strchr(s, 'l');
    printf("input: \"%s\" 'l' output: \"%s\" ", s, got ? got : "NULL");
    printf("%s\n", (got == s + 2) ? "SUCCESS" : "FAIL");

    got = s21_strchr(s, 'z');
    printf("input: \"%s\" 'z' output: %s ", s, got ? got : "NULL");
    printf("%s\n", (got == 0) ? "SUCCESS" : "FAIL");

    got = s21_strchr(s, '\0');
    printf("input: \"%s\" '\\\\0' output: end ", s);
    printf("%s\n", (got == s + 5) ? "SUCCESS" : "FAIL");
}

/* проверяет поиск подстроки */
void s21_strstr_test(void) {
    const char *h;
    char *got;

    h = "hello world";
    got = s21_strstr(h, "world");
    printf("input: \"%s\" \"world\" output: \"%s\" ", h, got ? got : "NULL");
    printf("%s\n", (got == h + 6) ? "SUCCESS" : "FAIL");

    got = s21_strstr(h, "bye");
    printf("input: \"%s\" \"bye\" output: %s ", h, got ? got : "NULL");
    printf("%s\n", (got == 0) ? "SUCCESS" : "FAIL");

    got = s21_strstr(h, "");
    printf("input: \"%s\" \"\" output: \"%s\" ", h, got ? got : "NULL");
    printf("%s\n", (got == h) ? "SUCCESS" : "FAIL");
}

int main(void) {
    s21_strlen_test();
    s21_strcmp_test();
    s21_strcpy_test();
    s21_strcat_test();
    s21_strchr_test();
    s21_strstr_test();
    return 0;
}
```

```makefile
full_coverage_tests: $(BUILD_DIR)
	$(CC) $(CFLAGS) s21_string.c s21_string_test.c -o $(BUILD_DIR)/Quest_12
```

**Консоль:** `make full_coverage_tests` && `../../build/Quest_12`

**Как работает.** Каждая `*_test` готовит буферы/строки, вызывает целевую функцию и сравнивает результат с ожиданием. В консоль всегда пишутся вход, выход и SUCCESS/FAIL — это и есть формат, который ждут автотесты сюжета.

#### Вариант B

Те же кейсы, но вынести печать вердикта в маленькую функцию `print_verdict(int ok)` — без ранних `return` внутри тестов. Сложность та же.

---

### Bonus Quest 13 — Width (`text_processor.c`)

**Суть.** Запуск только с ключом `-w`. Иначе печать `n/a`.  
Со stdin: число (ширина) и текст ≤ 100 символов (до `\\n`).  
Правила:
- строки ровно/по смыслу ширины, **не** начинаются и **не** заканчиваются пробелом;
- последняя строка **без** завершающего `\\n`;
- слово режется через `-` **только** если целиком не помещается в ширину;
- иначе слово остаётся целым;
- слова в строке раскидывать **равномерно** пробелами.

Примеры из условия:

```text
-w
10
hello how are you
→
hello how
are you

-w
5
ab abcd ab abcd ab abcdefgh
→
ab
abcd
ab
abcd
ab a-
bcde-
fgh
```

Два варианта ниже одинаковы по сложности: A кладёт слова индексами, B — через указатели/смещения. Оба делают перенос **с учётом оставшегося места в текущей строке** (это важно для примера с `abcdefgh`).

#### Вариант A. Индексы + куски «на лету»

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_TEXT 101
#define MAX_LINE 64
#define MAX_WORD 101

int is_w_flag(int argc, char **argv);
int read_width_and_text(int *width, char *text);
int strlen_local(const char *s);
void strcpy_local(char *dst, const char *src);
void strncpy_local(char *dst, const char *src, int n);
void flush_line(char line[][MAX_WORD], int n, int width, int last);
void process_text(int width, const char *text);

int main(int argc, char **argv) {
    int width;
    char text[MAX_TEXT];
    int ok;

    ok = 1;
    if (is_w_flag(argc, argv) == 0) {
        printf("n/a");
        ok = 0;
    }
    if (ok == 1) {
        ok = read_width_and_text(&width, text);
        if (ok == 0) {
            printf("n/a");
        }
    }
    if (ok == 1) {
        process_text(width, text);
    }
    return 0;
}

/* проверка argv на -w */
int is_w_flag(int argc, char **argv) {
    int ok;

    ok = 0;
    if (argc == 2 && argv[1][0] == '-' && argv[1][1] == 'w' && argv[1][2] == '\0') {
        ok = 1;
    }
    return ok;
}

int strlen_local(const char *s) {
    int i;

    i = 0;
    while (s[i] != '\0') {
        i++;
    }
    return i;
}

void strcpy_local(char *dst, const char *src) {
    int i;

    i = 0;
    while (src[i] != '\0') {
        dst[i] = src[i];
        i++;
    }
    dst[i] = '\0';
}

void strncpy_local(char *dst, const char *src, int n) {
    int i;

    for (i = 0; i < n; i++) {
        dst[i] = src[i];
    }
    dst[n] = '\0';
}

/* ширина и текст до '\\n', текст не длиннее 100 */
int read_width_and_text(int *width, char *text) {
    int ok;
    int i;
    int ch;

    ok = 1;
    if (scanf("%d", width) != 1 || *width <= 0) {
        ok = 0;
    }
    if (ok == 1) {
        ch = getchar();
        while (ch == ' ' || ch == '\t') {
            ch = getchar();
        }
        i = 0;
        if (ch == '\n' || ch == EOF) {
            ch = getchar();
        }
        while (ch != '\n' && ch != EOF && i < 100) {
            text[i++] = (char)ch;
            ch = getchar();
        }
        text[i] = '\0';
        if (i == 0) {
            ok = 0;
        }
    }
    return ok;
}

/* равномерные пробелы; last=1 — без '\\n' в конце */
void flush_line(char line[][MAX_WORD], int n, int width, int last) {
    int letters;
    int i;
    int gaps;
    int spaces;
    int base;
    int extra;
    int g;
    int k;

    if (n == 1) {
        printf("%s", line[0]);
    } else if (n > 1) {
        letters = 0;
        for (i = 0; i < n; i++) {
            letters += strlen_local(line[i]);
        }
        gaps = n - 1;
        spaces = width - letters;
        if (spaces < gaps) {
            spaces = gaps;
        }
        base = spaces / gaps;
        extra = spaces % gaps;
        for (i = 0; i < n; i++) {
            printf("%s", line[i]);
            if (i < gaps) {
                g = base + (i < extra ? 1 : 0);
                for (k = 0; k < g; k++) {
                    putchar(' ');
                }
            }
        }
    }
    if (last == 0) {
        putchar('\n');
    }
}

/* укладка с переносом: '-' только если слово длиннее width */
void process_text(int width, const char *text) {
    char line[MAX_LINE][MAX_WORD];
    char word[MAX_WORD];
    char piece[MAX_WORD];
    int nline;
    int used;
    int i;
    int j;
    int wlen;
    int room;
    int take;
    int pos;

    nline = 0;
    used = 0;
    i = 0;
    while (text[i] != '\0') {
        while (text[i] == ' ') {
            i++;
        }
        if (text[i] != '\0') {
            j = 0;
            while (text[i] != '\0' && text[i] != ' ' && j < MAX_WORD - 1) {
                word[j++] = text[i++];
            }
            word[j] = '\0';
            wlen = strlen_local(word);
            pos = 0;
            while (pos < wlen) {
                room = (nline == 0) ? width : (width - used - 1);
                if (room <= 0) {
                    flush_line(line, nline, width, 0);
                    nline = 0;
                    used = 0;
                    room = width;
                }
                if (wlen - pos <= room) {
                    strcpy_local(piece, word + pos);
                    take = wlen - pos;
                    pos = wlen;
                } else if (wlen <= width) {
                    /* слово целиком короче ширины — перенос без дефиса */
                    flush_line(line, nline, width, 0);
                    nline = 0;
                    used = 0;
                    strcpy_local(piece, word + pos);
                    take = wlen - pos;
                    pos = wlen;
                } else {
                    /* слово длиннее ширины — режем с '-' */
                    if (room < 2) {
                        flush_line(line, nline, width, 0);
                        nline = 0;
                        used = 0;
                        room = width;
                    }
                    take = room - 1;
                    strncpy_local(piece, word + pos, take);
                    piece[take] = '-';
                    piece[take + 1] = '\0';
                    pos += take;
                    take = room;
                }
                strcpy_local(line[nline], piece);
                if (nline == 0) {
                    used = strlen_local(piece);
                } else {
                    used = used + 1 + strlen_local(piece);
                }
                nline++;
                if (used >= width) {
                    flush_line(line, nline, width, 0);
                    nline = 0;
                    used = 0;
                }
            }
        }
    }
    if (nline > 0) {
        flush_line(line, nline, width, 1);
    }
}
```

**Что писать в консоли**

```bash
cd src/s21_string
gcc -std=c11 -Wall -Werror -Wextra text_processor.c -o ../../build/Quest_13
# или: make text_processor
../../build/Quest_13 -w
# 10
# hello how are you

../../build/Quest_13 -w
# 5
# ab abcd ab abcd ab abcdefgh

../../build/Quest_13 -x
# n/a
```

**Как работает.** Текст режется на слова. Для каждого слова смотрим, сколько символов ещё влезает в текущую строку. Если слово короткое и не влезает — строка сбрасывается, слово переносится целиком. Если слово длиннее всей ширины — оно режется кусками с дефисом, причём размер куска считается от **оставшегося** места (отсюда `ab a-` / `bcde-` / `fgh` в примере). Пробелы между несколькими кусками в строке раскидываются равномерно.

#### Вариант B. То же правило, но `room` через указатель на хвост слова

Логика та же, что в A; отличается только обход остатка слова:

```c
/* фрагмент вместо pos/word[pos]: cur указывает на ещё не уложенный хвост */
void process_text_b(int width, const char *text) {
    char line[MAX_LINE][MAX_WORD];
    char word[MAX_WORD];
    char piece[MAX_WORD];
    const char *cur;
    int nline;
    int used;
    int i;
    int j;
    int left;
    int room;

    nline = 0;
    used = 0;
    i = 0;
    while (text[i] != '\0') {
        while (text[i] == ' ') {
            i++;
        }
        if (text[i] != '\0') {
            j = 0;
            while (text[i] != '\0' && text[i] != ' ' && j < MAX_WORD - 1) {
                word[j++] = text[i++];
            }
            word[j] = '\0';
            cur = word;
            left = strlen_local(cur);
            while (left > 0) {
                room = (nline == 0) ? width : (width - used - 1);
                if (room <= 0) {
                    flush_line(line, nline, width, 0);
                    nline = 0;
                    used = 0;
                    room = width;
                }
                if (left <= room) {
                    strcpy_local(piece, cur);
                    cur += left;
                    left = 0;
                } else if (strlen_local(word) <= width) {
                    flush_line(line, nline, width, 0);
                    nline = 0;
                    used = 0;
                    strcpy_local(piece, cur);
                    cur += left;
                    left = 0;
                } else {
                    if (room < 2) {
                        flush_line(line, nline, width, 0);
                        nline = 0;
                        used = 0;
                        room = width;
                    }
                    strncpy_local(piece, cur, room - 1);
                    piece[room - 1] = '-';
                    piece[room] = '\0';
                    cur += (room - 1);
                    left -= (room - 1);
                }
                strcpy_local(line[nline], piece);
                used = (nline == 0) ? strlen_local(piece) : used + 1 + strlen_local(piece);
                nline++;
                if (used >= width) {
                    flush_line(line, nline, width, 0);
                    nline = 0;
                    used = 0;
                }
            }
        }
    }
    if (nline > 0) {
        flush_line(line, nline, width, 1);
    }
}
```

Остальные функции (`main`, `flush_line`, чтение) — как в варианте A. В сдаваемый файл кладёшь одну функцию `process_text` (A или B), не обе.

```makefile
text_processor: $(BUILD_DIR)
	$(CC) $(CFLAGS) text_processor.c -o $(BUILD_DIR)/Quest_13
```

**Консоль** — как в варианте A.

**Как работает.** То же выравнивание и те же правила дефиса. Разница косметическая: хвост длинного слова двигаешь указателем `cur`, а не индексом `pos`.

---

## Полный Makefile для `src/s21_string` (все цели)

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c11
BUILD_DIR = ../../build
LIB = s21_string.c
TEST = s21_string_test.c

.PHONY: strlen_tests strcmp_tests strcpy strcat strchr strstr \
        full_coverage_tests text_processor clean rebuild

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

strlen_tests: $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LIB) $(TEST) -o $(BUILD_DIR)/Quest_6

strcmp_tests: $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LIB) $(TEST) -o $(BUILD_DIR)/Quest_7

strcpy: $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LIB) $(TEST) -o $(BUILD_DIR)/Quest_8

strcat: $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LIB) $(TEST) -o $(BUILD_DIR)/Quest_9

strchr: $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LIB) $(TEST) -o $(BUILD_DIR)/Quest_10

strstr: $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LIB) $(TEST) -o $(BUILD_DIR)/Quest_11

full_coverage_tests: $(BUILD_DIR)
	$(CC) $(CFLAGS) $(LIB) $(TEST) -o $(BUILD_DIR)/Quest_12

text_processor: $(BUILD_DIR)
	$(CC) $(CFLAGS) text_processor.c -o $(BUILD_DIR)/Quest_13

clean:
	rm -f $(BUILD_DIR)/Quest_6 $(BUILD_DIR)/Quest_7 $(BUILD_DIR)/Quest_8 \
		$(BUILD_DIR)/Quest_9 $(BUILD_DIR)/Quest_10 $(BUILD_DIR)/Quest_11 \
		$(BUILD_DIR)/Quest_12 $(BUILD_DIR)/Quest_13

rebuild: clean strlen_tests
```

---

## Итоговый чеклист перед push

1. Ветка **`develop`**, файлы только в **`src/`**.
2. **Нет** в коммите: `build/`, `*.o`, `*.a`, `*.so`, бинарников.
3. `clang-format -n` по своим `.c`/`.h` (рядом лежит скопированный `.clang-format`).
4. Нет `system()`.
5. Нет `#include <string.h>` в строковых квестах.
6. `GOLDEN_RATIO` ≈ **0.618**, не 0.666.
7. Makefile-пути относительные от `src/main_executable_module` и `src/s21_string`.
8. Имена бинарников строго: `Quest_3` … `Quest_13` как в условии.
9. У функций один `return` в конце (как просили в разборе).
10. После Quest 1–2 не ломай разбиение файлов — только доработки внутри согласованной структуры.

---

## Ответ на вопрос про `.h`

**Да, `.h` будут.**  
В Room 1 они уже приходят с проектом: ты их **подключаешь** и при необходимости чуть правишь (например, константу в `decision.h`).  
В Room 2 файл **`s21_string.h` ты создаёшь сама** — без него тесты и другие `.c` не увидят объявления функций. Это нормальная и обязательная часть многофайлового проекта.
