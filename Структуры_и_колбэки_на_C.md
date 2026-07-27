# Структуры и колбэки на C

Краткий разбор квестов **D10T07** (Level 3: Room 3 + Room 4).
Стандарт **C11**, компилятор `gcc` с флагами `-Wall -Werror -Wextra`, стиль **Google** (`IndentWidth: 4`, `ColumnLimit: 110`).
После каждого квеста нужно сделать commit и push исходников из `src/` в ветку **`develop`**.

**Общие правила Werther и чеклиста.**
Функция **`system()`** и аналогичные вызовы к ядру **запрещены** во всех заданиях.
Код лежит в `src/`, разработка ведётся в ветке `develop`, бинарники / `.o` пушить нельзя.
Всю динамическую память нужно освобождать через **`free`**, иначе проверки на утечки не пройдут.
Список и стек дополнительно гоняются через **cppcheck**.
Сборка — через **Makefile**, исполняемые файлы — в `build/` в корне репозитория (`Quest_1` … `Quest_8`).

Связанные материалы: [Многофайловые_проекты_символы_и_строки_на_C](Многофайловые_проекты_символы_и_строки_на_C.md), [Динамическая_память_и_матрицы_на_C](Динамическая_память_и_матрицы_на_C.md), [C_СПРАВОЧНИК](C_СПРАВОЧНИК.md).

В решениях приведены **два варианта одинаковой сложности**. Вариант A чаще использует пузырьковую сортировку / итеративный обход. Вариант B — сортировку выбором / рекурсию там, где это уместно. Сдавать можно любой один вариант.

**Структурное правило дня.** У каждой функции должен быть один вход и один выход: один `return` в конце тела, ошибки — через флаги, без ранних `return` посередине.

**Важно про стартовые файлы.** Функцию `initialize_doors` **нельзя менять**. Если в твоём репозитории сигнатуры списка принимают `struct door *` (указатель), а не `struct door` (значение) — подстрой реализацию под свой `.h`, смысл тот же.

---

## 0. Теория

### Структура (`struct`)

Структура — пользовательский тип, который хранит несколько полей разных типов вместе.

```c
struct door {
    int id;
    int status;  /* 0 — закрыта, 1 — открыта */
};

struct door d;
d.id = 3;
d.status = 0;
```

Поля лежат в памяти в порядке объявления. Размер структуры ≈ сумма размеров полей (плюс выравнивание). Через точку обращаются к полю объекта (`d.id`), через `->` — к полю по указателю (`p->id` то же самое, что `(*p).id`).

Структуры удобно передавать в функции и класть в динамические узлы списка/дерева: интерфейс функции не раздувается десятками отдельных параметров.

### Объединение (`union`) и «метка»

В `union` все поля делят **одну** область памяти. Размер объединения = размер самого большого поля. В каждый момент «живёт» только одно значение — какое именно, программа должна помнить сама.

Частый приём — **объединение с меткой** (tagged union): структура с полем-типом и `union` внутри.

```c
struct value {
    int tag;           /* 0 — int, 1 — double */
    union {
        int i;
        double d;
    } data;
};
```

### Односвязный список

Узел хранит данные и указатель на следующий узел. Последний узел хранит `NULL`.

```
[door0 | *] --> [door1 | *] --> [door2 | NULL]
```

Плюсы: длина не фиксирована, вставка/удаление после известного узла дёшевы. Минусы: нет быстрого доступа по индексу, нужна динамическая память, больше накладных расходов, чем у массива.

Типичные операции: `init` (создать первый узел), `add` (вставить после узла), `find`, `remove`, `destroy` (освободить всю цепочку).

### Стек

Стек — структура **LIFO** (последним положил — первым забрал). Базовые операции: `push` (положить сверху), `pop` (снять сверху). Динамический стек целых чисел удобно делать как односвязный список, где «голова» — вершина стека.

### Функции обратного вызова (callbacks)

Колбэк — указатель на функцию, который передают как параметр. Вызывающий модуль не знает деталей реализации: он просто вызывает переданную функцию.

```c
void print_log(char (*print)(char), char *message);
/* print — указатель на функцию: принимает char, возвращает char */

int cmp(int a, int b);
void bstree_insert(t_btree *root, int item, int (*cmpf)(int, int));
```

Так ядро системы может подставлять разный вывод, разную валидацию или разный компаратор без переписывания модуля.

### Переменное число аргументов (`stdarg.h`)

Если функция принимает `...`, нужен `<stdarg.h>`:

```c
#include <stdarg.h>

va_list args;
va_start(args, last_named_param);
/* va_arg(args, type) — очередной аргумент */
va_end(args);
```

`va_start` / `va_arg` / `va_end` обязательны парой: после обхода всегда вызывай `va_end`.

### Бинарное дерево поиска (BST)

Узел:

```c
typedef struct s_btree {
    struct s_btree *left;
    struct s_btree *right;
    int item;
} t_btree;
```

Правило BST: слева — меньшие значения, справа — большие (сравнение через колбэк `cmpf`).

Обходы:

| Имя | Порядок | Для BST |
|-----|---------|---------|
| infix (inorder) | лево → корень → право | по возрастанию |
| prefix (preorder) | корень → лево → право | «как лежит» |
| postfix (postorder) | лево → право → корень | классический постфикс |

(В тексте задания у postfix написано «по убыванию», но в скобках указан порядок LRN — его и реализуем.)

### Makefile и стадии

Типичные цели в `src/Makefile`:

| Стадия | Бинарник |
|--------|----------|
| `door_struct` | `../build/Quest_1` |
| `list_test` | `../build/Quest_2` |
| `stack_test` | `../build/Quest_3` |
| `print_module` | `../build/Quest_4` |
| `documentation_module` | `../build/Quest_5` |
| `bst_create_test` | `../build/Quest_6` |
| `bst_insert_test` | `../build/Quest_7` |
| `bst_traverse_test` | `../build/Quest_8` |

Перед сборкой создай папку `build` в корне репозитория (`mkdir -p ../build`).

### Стиль и проверка

```bash
clang-format -n *.c *.h
cppcheck --enable=all --suppress=missingIncludeSystem *.c
# утечки (Linux):
valgrind --leak-check=full ../build/Quest_2
```

---

## Решения задач

Команды ниже — из папки `src/` репозитория проекта (после `git checkout -b develop`).

```bash
git checkout -b develop   # если ещё нет
mkdir -p ../build
cd src
```

---

### Quest 1 — The Doors (`door_struct.h` + `dmanager_module.c`)

**Суть.** Описать `struct door` (`id`, `status`). После `initialize_doors` закрыть все двери (`status = 0`), отсортировать массив по возрастанию `id` и вывести строки вида `id, status`. Функцию инициализации не трогать. Стадия Makefile: `door_struct` → `build/Quest_1`.

Ожидаемый вывод (все статусы 0, id от 0 до 14):

```
0, 0
1, 0
2, 0
...
14, 0
```

#### Вариант A. Пузырьковая сортировка

`src/door_struct.h`:

```c
#ifndef DOOR_STRUCT_H
#define DOOR_STRUCT_H

struct door {
    int id;
    int status;
};

#endif
```

`src/dmanager_module.c` (фрагмент после инициализации — полный файл):

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#include "door_struct.h"

#define DOORS_COUNT 15
#define MAX_ID_SEED 10000

void initialize_doors(struct door *doors);
void close_all_doors(struct door *doors, int n);
void sort_doors(struct door *doors, int n);
void print_doors(struct door *doors, int n);

int main(void) {
    struct door doors[DOORS_COUNT];

    initialize_doors(doors);
    close_all_doors(doors, DOORS_COUNT);
    sort_doors(doors, DOORS_COUNT);
    print_doors(doors, DOORS_COUNT);
    return 0;
}

/* закрывает все двери: status = 0 */
void close_all_doors(struct door *doors, int n) {
    int i;

    for (i = 0; i < n; i++) {
        doors[i].status = 0;
    }
}

/* пузырьковая сортировка дверей по возрастанию id */
void sort_doors(struct door *doors, int n) {
    int i;
    int j;
    struct door tmp;

    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - 1 - i; j++) {
            if (doors[j].id > doors[j + 1].id) {
                tmp = doors[j];
                doors[j] = doors[j + 1];
                doors[j + 1] = tmp;
            }
        }
    }
}

/* печатает id и status каждой двери */
void print_doors(struct door *doors, int n) {
    int i;

    for (i = 0; i < n; i++) {
        printf("%d, %d\n", doors[i].id, doors[i].status);
    }
}

/* Doors initialization function */
/* ATTENTION!!! DO NOT CHANGE! */
void initialize_doors(struct door *doors) {
    srand(time(0));

    int seed = rand() % MAX_ID_SEED;
    for (int i = 0; i < DOORS_COUNT; i++) {
        doors[i].id = (i + seed) % DOORS_COUNT;
        doors[i].status = rand() % 2;
    }
}
```

**Как работает.** `initialize_doors` случайно расставляет id и статусы — её не трогаем. Затем все статусы принудительно обнуляются. Пузырьковая сортировка сравнивает соседние двери и меняет их местами, пока id не выстроятся по возрастанию (0…14). Печать идёт в формате `id, status` с переводом строки после каждой двери.

#### Вариант B. Сортировка выбором

Отличие только в `sort_doors` (остальное как в A):

```c
/* сортировка выбором: на место i ставим дверь с минимальным id */
void sort_doors(struct door *doors, int n) {
    int i;
    int j;
    int min_i;
    struct door tmp;

    for (i = 0; i < n - 1; i++) {
        min_i = i;
        for (j = i + 1; j < n; j++) {
            if (doors[j].id < doors[min_i].id) {
                min_i = j;
            }
        }
        tmp = doors[i];
        doors[i] = doors[min_i];
        doors[min_i] = tmp;
    }
}
```

**Как работает.** На каждом шаге в хвосте массива ищется дверь с наименьшим `id`, и она ставится на текущую позицию. Результат тот же: id по возрастанию, все двери закрыты.

```bash
# в Makefile должна быть цель door_struct
make door_struct
../build/Quest_1
# ожидается 15 строк: 0, 0 … 14, 0
```

Пример куска Makefile:

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c11
BUILD = ../build

door_struct: dmanager_module.c door_struct.h
	mkdir -p $(BUILD)
	$(CC) $(CFLAGS) dmanager_module.c -o $(BUILD)/Quest_1
```

---

### Quest 2 — Linked List (`list.h`, `list.c`, `list_test.c`)

**Суть.** Односвязный список дверей: `init`, `add_door`, `find_door`, `remove_door`, `destroy`. Модульные тесты `add_door` и `remove_door` возвращают `SUCCESS` / `FAIL`. Стадия: `list_test` → `Quest_2`. cppcheck обязателен.

#### Общий заголовок `src/list.h`

```c
#ifndef LIST_H
#define LIST_H

#include "door_struct.h"

struct node {
    struct door door;
    struct node *next;
};

struct node *init(struct door door);
struct node *add_door(struct node *elem, struct door door);
struct node *find_door(int door_id, struct node *root);
struct node *remove_door(struct node *elem, struct node *root);
void destroy(struct node *root);

#endif
```

#### Вариант A. Итеративные операции

`src/list.c`:

```c
#include <stdlib.h>

#include "list.h"

/* создаёт первый узел списка из двери */
struct node *init(struct door door) {
    struct node *node;

    node = (struct node *)malloc(sizeof(struct node));
    if (node != NULL) {
        node->door = door;
        node->next = NULL;
    }
    return node;
}

/* вставляет новую дверь сразу после узла elem */
struct node *add_door(struct node *elem, struct door door) {
    struct node *node;

    node = NULL;
    if (elem != NULL) {
        node = init(door);
        if (node != NULL) {
            node->next = elem->next;
            elem->next = node;
        }
    }
    return node;
}

/* ищет узел с заданным id, иначе NULL */
struct node *find_door(int door_id, struct node *root) {
    struct node *cur;

    cur = root;
    while (cur != NULL && cur->door.id != door_id) {
        cur = cur->next;
    }
    return cur;
}

/* удаляет узел elem, возвращает (новый) корень списка */
struct node *remove_door(struct node *elem, struct node *root) {
    struct node *cur;
    struct node *result;

    result = root;
    if (elem != NULL && root != NULL) {
        if (root == elem) {
            result = root->next;
            free(elem);
        } else {
            cur = root;
            while (cur != NULL && cur->next != elem) {
                cur = cur->next;
            }
            if (cur != NULL) {
                cur->next = elem->next;
                free(elem);
            }
        }
    }
    return result;
}

/* освобождает всю цепочку узлов */
void destroy(struct node *root) {
    struct node *cur;
    struct node *next;

    cur = root;
    while (cur != NULL) {
        next = cur->next;
        free(cur);
        cur = next;
    }
}
```

`src/list_test.c`:

```c
#include <stdio.h>

#include "list.h"

#define SUCCESS "SUCCESS"
#define FAIL "FAIL"

char *add_door_test(void);
char *remove_door_test(void);

int main(void) {
    printf("%s\n", add_door_test());
    printf("%s\n", remove_door_test());
    return 0;
}

/* проверяет вставку узла после головы */
char *add_door_test(void) {
    struct door d0;
    struct door d1;
    struct door d2;
    struct node *root;
    struct node *added;
    char *result;

    d0.id = 0;
    d0.status = 0;
    d1.id = 1;
    d1.status = 0;
    d2.id = 2;
    d2.status = 0;
    root = init(d0);
    root->next = init(d1);
    added = add_door(root, d2);
    if (added != NULL && root->next == added && added->door.id == 2 && added->next->door.id == 1) {
        result = SUCCESS;
    } else {
        result = FAIL;
    }
    destroy(root);
    return result;
}

/* проверяет удаление среднего узла */
char *remove_door_test(void) {
    struct door d0;
    struct door d1;
    struct door d2;
    struct node *root;
    struct node *mid;
    struct node *tail;
    char *result;

    d0.id = 0;
    d0.status = 0;
    d1.id = 1;
    d1.status = 0;
    d2.id = 2;
    d2.status = 0;
    root = init(d0);
    mid = add_door(root, d1);
    add_door(mid, d2);
    tail = find_door(2, root);
    root = remove_door(mid, root);
    if (root != NULL && root->next == tail && find_door(1, root) == NULL) {
        result = SUCCESS;
    } else {
        result = FAIL;
    }
    destroy(root);
    return result;
}
```

**Как работает.** `init` выделяет узел и копирует в него структуру двери. `add_door` создаёт новый узел и вшивает его между `elem` и бывшим `elem->next`. `find_door` линейно идёт по цепочке. `remove_door` отдельно обрабатывает удаление головы и удаление из середины/хвоста, возвращая актуальный корень. `destroy` проходит список и освобождает каждый узел. Тесты собирают маленький список, проверяют вставку и удаление, печатают `SUCCESS`/`FAIL` и чистят память.

#### Вариант B. Те же операции, поиск через индексный стиль указателей

Отличие в `find_door` / чуть другой записи циклов — интерфейс тот же:

```c
/* ищет дверь, двигая указатель-корень */
struct node *find_door(int door_id, struct node *root) {
    while (root != NULL && root->door.id != door_id) {
        root = root->next;
    }
    return root;
}
```

Остальные функции — как в варианте A. Логика та же: локальный параметр `root` — копия указателя, исходный корень снаружи не портится.

```bash
make list_test
../build/Quest_2
# SUCCESS
# SUCCESS
cppcheck --enable=all --suppress=missingIncludeSystem list.c list_test.c
```

```makefile
list_test: list.c list_test.c list.h door_struct.h
	mkdir -p $(BUILD)
	$(CC) $(CFLAGS) list.c list_test.c -o $(BUILD)/Quest_2
```

---

### Quest 3 — Stack for key (`stack.h`, `stack.c`, `stack_test.c`)

**Суть.** Динамический стек целых чисел: `init`, `push`, `pop`, `destroy`. Тесты `push` и `pop` → `SUCCESS`/`FAIL`. Стадия: `stack_test` → `Quest_3`. Автотестов Werther нет, но cppcheck и стиль есть.

#### Вариант A. Стек как список, вершина — голова

`src/stack.h`:

```c
#ifndef STACK_H
#define STACK_H

struct stack {
    int data;
    struct stack *next;
};

struct stack *init(int data);
struct stack *push(struct stack *root, int data);
int pop(struct stack **root);
void destroy(struct stack *root);

#endif
```

`src/stack.c`:

```c
#include <stdlib.h>

#include "stack.h"

/* создаёт стек из одного элемента */
struct stack *init(int data) {
    struct stack *node;

    node = (struct stack *)malloc(sizeof(struct stack));
    if (node != NULL) {
        node->data = data;
        node->next = NULL;
    }
    return node;
}

/* кладёт значение на вершину, возвращает новую вершину */
struct stack *push(struct stack *root, int data) {
    struct stack *node;

    node = init(data);
    if (node != NULL) {
        node->next = root;
    }
    return node;
}

/* снимает вершину, пишет новое значение корня через указатель */
int pop(struct stack **root) {
    struct stack *top;
    int value;

    value = 0;
    if (root != NULL && *root != NULL) {
        top = *root;
        value = top->data;
        *root = top->next;
        free(top);
    }
    return value;
}

/* освобождает весь стек */
void destroy(struct stack *root) {
    struct stack *cur;
    struct stack *next;

    cur = root;
    while (cur != NULL) {
        next = cur->next;
        free(cur);
        cur = next;
    }
}
```

`src/stack_test.c`:

```c
#include <stdio.h>

#include "stack.h"

#define SUCCESS "SUCCESS"
#define FAIL "FAIL"

char *push_test(void);
char *pop_test(void);

int main(void) {
    printf("%s\n", push_test());
    printf("%s\n", pop_test());
    return 0;
}

/* проверяет, что push кладёт элемент на вершину */
char *push_test(void) {
    struct stack *root;
    char *result;

    root = init(10);
    root = push(root, 20);
    if (root != NULL && root->data == 20 && root->next != NULL && root->next->data == 10) {
        result = SUCCESS;
    } else {
        result = FAIL;
    }
    destroy(root);
    return result;
}

/* проверяет, что pop возвращает верхнее и обновляет вершину */
char *pop_test(void) {
    struct stack *root;
    int value;
    char *result;

    root = init(1);
    root = push(root, 2);
    root = push(root, 3);
    value = pop(&root);
    if (value == 3 && root != NULL && root->data == 2) {
        result = SUCCESS;
    } else {
        result = FAIL;
    }
    destroy(root);
    return result;
}
```

**Как работает.** Вершина стека — голова списка. `push` создаёт новый узел и ставит его перед старой головой. `pop` читает данные головы, сдвигает корень на `next` и освобождает старую голову (поэтому нужен `struct stack **`). `destroy` чистит остаток цепочки. Тесты проверяют порядок LIFO.

#### Вариант B. Обёртка с полем `top`

`src/stack.h`:

```c
#ifndef STACK_H
#define STACK_H

struct stack_node {
    int data;
    struct stack_node *next;
};

struct stack {
    struct stack_node *top;
};

struct stack *init(void);
int push(struct stack *s, int data);
int pop(struct stack *s);
void destroy(struct stack *s);

#endif
```

`src/stack.c` (ключевые функции):

```c
#include <stdlib.h>

#include "stack.h"

/* создаёт пустой стек */
struct stack *init(void) {
    struct stack *s;

    s = (struct stack *)malloc(sizeof(struct stack));
    if (s != NULL) {
        s->top = NULL;
    }
    return s;
}

/* кладёт data на вершину; 0 — ок, 1 — ошибка */
int push(struct stack *s, int data) {
    struct stack_node *node;
    int error;

    error = 1;
    if (s != NULL) {
        node = (struct stack_node *)malloc(sizeof(struct stack_node));
        if (node != NULL) {
            node->data = data;
            node->next = s->top;
            s->top = node;
            error = 0;
        }
    }
    return error;
}

/* снимает вершину; возвращает значение (при пустом стеке — 0) */
int pop(struct stack *s) {
    struct stack_node *node;
    int value;

    value = 0;
    if (s != NULL && s->top != NULL) {
        node = s->top;
        value = node->data;
        s->top = node->next;
        free(node);
    }
    return value;
}

/* освобождает узлы и сам стек */
void destroy(struct stack *s) {
    struct stack_node *cur;
    struct stack_node *next;

    if (s != NULL) {
        cur = s->top;
        while (cur != NULL) {
            next = cur->next;
            free(cur);
            cur = next;
        }
        free(s);
    }
}
```

Тесты адаптируй под `init(void)` / коды возврата `push`. Идея та же: LIFO, без утечек.

```bash
make stack_test
../build/Quest_3
# SUCCESS
# SUCCESS
```

```makefile
stack_test: stack.c stack_test.c stack.h
	mkdir -p $(BUILD)
	$(CC) $(CFLAGS) stack.c stack_test.c -o $(BUILD)/Quest_3
```

---

### Quest 4 — Print Module (`print_module.c`, при необходимости `main_module_entry_point.c`)

**Суть.** Дописать `print_log`: через колбэк печати посимвольно вывести `[LOG] ЧЧ:ММ:СС сообщение`. Время — текущее локальное. Стадия: `print_module` → `Quest_4`.

Типичные заготовки в репозитории:

```c
/* print_module.h */
#define Module_load_success_message "Output stream module load: success\n"
#define Log_prefix "[LOG]"
void print_log(char (*print)(char), char *message);
char print_char(char ch);
```

#### Вариант A. Собрать строку через `strftime`, затем отдать колбэку

```c
#include <stdio.h>
#include <time.h>

#include "print_module.h"

/* печатает один символ через putchar */
char print_char(char ch) {
    putchar(ch);
    return ch;
}

/* печатает строку посимвольно колбэком print */
void print_string(char (*print)(char), const char *s) {
    int i;

    i = 0;
    while (s[i] != '\0') {
        print(s[i]);
        i++;
    }
}

/* [LOG] HH:MM:SS message через колбэк */
void print_log(char (*print)(char), char *message) {
    time_t now;
    struct tm *t;
    char time_buf[16];

    print_string(print, Log_prefix);
    print(' ');
    now = time(NULL);
    t = localtime(&now);
    strftime(time_buf, sizeof(time_buf), "%H:%M:%S", t);
    print_string(print, time_buf);
    print(' ');
    if (message != NULL) {
        print_string(print, message);
    }
}
```

`main_module_entry_point.c` (квест 4):

```c
#include "print_module.h"

int main(void) {
    print_log(print_char, Module_load_success_message);
    return 0;
}
```

**Как работает.** `print_log` не вызывает `printf` для всего сообщения целиком: она пользуется переданным колбэком. Сначала печатается префикс `[LOG]`, пробел, время `ЧЧ:ММ:СС` (из `localtime`/`strftime`), ещё пробел и текст сообщения. Так ядро может подменить способ вывода, не меняя модуль лога.

#### Вариант B. Печать времени по полям `tm` без `strftime`

```c
/* печатает двузначное число колбэком */
void print_two_digits(char (*print)(char), int value) {
    print((char)('0' + value / 10));
    print((char)('0' + value % 10));
}

void print_log(char (*print)(char), char *message) {
    time_t now;
    struct tm *t;
    int i;

    i = 0;
    while (Log_prefix[i] != '\0') {
        print(Log_prefix[i]);
        i++;
    }
    print(' ');
    now = time(NULL);
    t = localtime(&now);
    print_two_digits(print, t->tm_hour);
    print(':');
    print_two_digits(print, t->tm_min);
    print(':');
    print_two_digits(print, t->tm_sec);
    print(' ');
    i = 0;
    if (message != NULL) {
        while (message[i] != '\0') {
            print(message[i]);
            i++;
        }
    }
}
```

**Как работает.** То же сообщение, но часы/минуты/секунды печатаются как цифры из `tm_hour` / `tm_min` / `tm_sec`. Удобно, если не хочется буфера `strftime`.

```bash
make print_module
../build/Quest_4
# [LOG] 21:45:03 Output stream module load: success
```

```makefile
print_module: print_module.c main_module_entry_point.c print_module.h
	mkdir -p $(BUILD)
	$(CC) $(CFLAGS) print_module.c main_module_entry_point.c -o $(BUILD)/Quest_4
```

---

### Quest 5 — Checking Module (`documentation_module.c` + вывод в `main`)

**Суть.** `check_available_documentation_module` принимает колбэк `validate`, число документов и `...` имён. Для каждого имени вызывает `validate`, пишет `0/1` в динамический массив. В `main` — человекочитаемый вывод с шириной поля **15**. Стадия: `documentation_module` → `Quest_5`.

В `.h` обычно нужно дописать:

```c
#define Documents_count 4
```

(макрос `Documents` уже разворачивается в четыре строки: `"Linked lists", "Queues", "Maps", "Binary Trees"`; доступен только `"Binary Trees"`).

#### Вариант A. Классический `va_list`

`src/documentation_module.c`:

```c
#include <stdarg.h>
#include <stdlib.h>
#include <string.h>

#include "documentation_module.h"

/* 1 если документ доступен, иначе 0 */
int validate(char *data) {
    int result;

    result = 0;
    if (data != NULL && strcmp(data, Available_document) == 0) {
        result = 1;
    }
    return result;
}

/* применяет validate ко всем документам из ... */
int *check_available_documentation_module(int (*validate_cb)(char *), int document_count,
                                          ...) {
    va_list args;
    int *mask;
    int i;
    char *doc;

    mask = NULL;
    if (document_count > 0 && validate_cb != NULL) {
        mask = (int *)malloc(sizeof(int) * (size_t)document_count);
        if (mask != NULL) {
            va_start(args, document_count);
            for (i = 0; i < document_count; i++) {
                doc = va_arg(args, char *);
                mask[i] = validate_cb(doc);
            }
            va_end(args);
        }
    }
    return mask;
}
```

Фрагмент `main_module_entry_point.c` для Quest 5:

```c
#include <stdio.h>
#include <stdlib.h>

#include "documentation_module.h"
#include "print_module.h"

int main(void) {
    int *availability_mask;
    char *docs[] = {Documents};
    int i;
    const char *status;

    print_log(print_char, Module_load_success_message);

    availability_mask = check_available_documentation_module(validate, Documents_count, Documents);
    if (availability_mask != NULL) {
        for (i = 0; i < Documents_count; i++) {
            if (availability_mask[i]) {
                status = "available";
            } else {
                status = "unavailable";
            }
            printf("[%-15s : %s]\n", docs[i], status);
        }
        free(availability_mask);
    }
    return 0;
}
```

**Как работает.** `va_start` открывает доступ к безымянным аргументам после `document_count`. В цикле `va_arg` достаёт очередное имя (`char *`), колбэк `validate` сравнивает его с `"Binary Trees"`. Результат складывается в массив на куче. `main` печатает имя в поле шириной 15 (`%-15s`) и статус, затем освобождает массив.

#### Вариант B. Тот же `va_list`, маска через `calloc`

```c
int *check_available_documentation_module(int (*validate_cb)(char *), int document_count,
                                          ...) {
    va_list args;
    int *mask;
    int i;
    char *doc;

    mask = NULL;
    if (document_count > 0 && validate_cb != NULL) {
        mask = (int *)calloc((size_t)document_count, sizeof(int));
        if (mask != NULL) {
            va_start(args, document_count);
            i = 0;
            while (i < document_count) {
                doc = va_arg(args, char *);
                mask[i] = validate_cb(doc);
                i++;
            }
            va_end(args);
        }
    }
    return mask;
}
```

**Как работает.** Логика идентична варианту A. `calloc` сразу обнуляет маску; цикл записан через `while`. Для автотестов разницы нет.

Ожидаемый вид вывода документов:

```
[Linked lists   : unavailable]
[Queues         : unavailable]
[Maps           : unavailable]
[Binary Trees   : available]
```

```bash
make documentation_module
../build/Quest_5
```

```makefile
documentation_module: documentation_module.c print_module.c main_module_entry_point.c
	mkdir -p $(BUILD)
	$(CC) $(CFLAGS) documentation_module.c print_module.c main_module_entry_point.c -o $(BUILD)/Quest_5
```

---

### Bonus Quest 6 — BST create (`bst.h`, `bst.c`, `bst_create_test.c`)

**Суть.** Тип `t_btree` и `bstree_create_node(int item)`: выделить узел, записать `item`, обнулить `left`/`right`. Тест — минимум два значения с понятным выводом. Стадия: `bst_create_test` → `Quest_6`.

#### Вариант A

`src/bst.h`:

```c
#ifndef BST_H
#define BST_H

typedef struct s_btree {
    struct s_btree *left;
    struct s_btree *right;
    int item;
} t_btree;

t_btree *bstree_create_node(int item);

#endif
```

`src/bst.c`:

```c
#include <stdlib.h>

#include "bst.h"

/* создаёт лист дерева со значением item */
t_btree *bstree_create_node(int item) {
    t_btree *node;

    node = (t_btree *)malloc(sizeof(t_btree));
    if (node != NULL) {
        node->item = item;
        node->left = NULL;
        node->right = NULL;
    }
    return node;
}
```

`src/bst_create_test.c`:

```c
#include <stdio.h>
#include <stdlib.h>

#include "bst.h"

int main(void) {
    t_btree *a;
    t_btree *b;

    a = bstree_create_node(4);
    b = bstree_create_node(7);
    if (a != NULL) {
        printf("created node item=%d left=%p right=%p\n", a->item, (void *)a->left, (void *)a->right);
    }
    if (b != NULL) {
        printf("created node item=%d left=%p right=%p\n", b->item, (void *)b->left, (void *)b->right);
    }
    free(a);
    free(b);
    return 0;
}
```

**Как работает.** Узел — три поля: значение и два пустых ребёнка. Пока вставки нет, это просто изолированный лист. Тест создаёт два узла (например, 4 — id модуля документации — и 7), показывает, что дети `NULL`, и освобождает память.

#### Вариант B. То же с явной проверкой успеха в тесте

```c
int main(void) {
    t_btree *n1;
    t_btree *n2;
    int ok;

    ok = 1;
    n1 = bstree_create_node(0);
    n2 = bstree_create_node(10);
    if (n1 == NULL || n1->item != 0 || n1->left != NULL || n1->right != NULL) {
        ok = 0;
    }
    if (n2 == NULL || n2->item != 10 || n2->left != NULL || n2->right != NULL) {
        ok = 0;
    }
    printf("node 0: item=%d\n", n1 ? n1->item : -1);
    printf("node 10: item=%d\n", n2 ? n2->item : -1);
    printf("%s\n", ok ? "SUCCESS" : "FAIL");
    free(n1);
    free(n2);
    return 0;
}
```

```bash
make bst_create_test
../build/Quest_6
```

---

### Bonus Quest 7 — Growing tree (`bstree_insert` + компаратор)

**Суть.** `bstree_insert(root, item, cmpf)` вставляет новый узел через колбэк сравнения. Тест — ≥2 набора с выводом, куда вставился лист. Стадия: `bst_insert_test` → `Quest_7`.

Добавь в `bst.h`:

```c
void bstree_insert(t_btree *root, int item, int (*cmpf)(int, int));
int int_cmp(int a, int b);
```

#### Вариант A. Итеративная вставка

```c
#include <stdio.h>
#include <stdlib.h>

#include "bst.h"

t_btree *bstree_create_node(int item) {
    t_btree *node;

    node = (t_btree *)malloc(sizeof(t_btree));
    if (node != NULL) {
        node->item = item;
        node->left = NULL;
        node->right = NULL;
    }
    return node;
}

/* компаратор: <0 если a < b, >0 если a > b, 0 если равны */
int int_cmp(int a, int b) {
    return a - b;
}

/* вставляет item в BST относительно root */
void bstree_insert(t_btree *root, int item, int (*cmpf)(int, int)) {
    t_btree *cur;
    t_btree *parent;
    int side;

    if (root != NULL && cmpf != NULL) {
        cur = root;
        parent = NULL;
        side = 0;
        while (cur != NULL) {
            parent = cur;
            if (cmpf(item, cur->item) < 0) {
                side = -1;
                cur = cur->left;
            } else {
                side = 1;
                cur = cur->right;
            }
        }
        if (parent != NULL) {
            if (side < 0) {
                parent->left = bstree_create_node(item);
            } else {
                parent->right = bstree_create_node(item);
            }
        }
    }
}
```

`src/bst_insert_test.c` (идея):

```c
#include <stdio.h>
#include <stdlib.h>

#include "bst.h"

void free_tree(t_btree *root);

int main(void) {
    t_btree *root;

    /* набор 1: 4 — корень, 2 влево, 6 вправо */
    root = bstree_create_node(4);
    printf("insert 2 -> left of %d\n", root->item);
    bstree_insert(root, 2, int_cmp);
    printf("insert 6 -> right of %d\n", root->item);
    bstree_insert(root, 6, int_cmp);
    free_tree(root);

    /* набор 2: 5, затем 3, 7, 1 */
    root = bstree_create_node(5);
    printf("insert 3 -> left of %d\n", root->item);
    bstree_insert(root, 3, int_cmp);
    printf("insert 7 -> right of %d\n", root->item);
    bstree_insert(root, 7, int_cmp);
    printf("insert 1 -> left of %d\n", root->left->item);
    bstree_insert(root, 1, int_cmp);
    free_tree(root);
    return 0;
}

/* рекурсивно освобождает дерево */
void free_tree(t_btree *root) {
    if (root != NULL) {
        free_tree(root->left);
        free_tree(root->right);
        free(root);
    }
}
```

**Как работает.** От корня спускаемся: если `cmpf(item, cur) < 0` — идём влево, иначе вправо, пока не найдём пустое место. Туда вешаем новый лист. Компаратор — обычное вычитание целых; его можно заменить другой функцией без правки `bstree_insert`.

#### Вариант B. Рекурсивная вставка

```c
void bstree_insert(t_btree *root, int item, int (*cmpf)(int, int)) {
    if (root != NULL && cmpf != NULL) {
        if (cmpf(item, root->item) < 0) {
            if (root->left == NULL) {
                root->left = bstree_create_node(item);
            } else {
                bstree_insert(root->left, item, cmpf);
            }
        } else {
            if (root->right == NULL) {
                root->right = bstree_create_node(item);
            } else {
                bstree_insert(root->right, item, cmpf);
            }
        }
    }
}
```

**Как работает.** Та же логика BST, но шаг вниз оформлен рекурсией: функция вызывает себя для левого или правого ребёнка, пока не дойдёт до `NULL`-ссылки у родителя.

```bash
make bst_insert_test
../build/Quest_7
```

---

### Bonus Quest 8 — Three styles of traversing

**Суть.** Три обхода с колбэком `applyf`:

- `bstree_apply_infix` — лево, корень, право;
- `bstree_apply_prefix` — корень, лево, право;
- `bstree_apply_postfix` — лево, право, корень.

`applyf` в тесте печатает значение узла. Наборы данных — из Quest 7. Стадия: `bst_traverse_test` → `Quest_8`.

В `bst.h`:

```c
void bstree_apply_infix(t_btree *root, void (*applyf)(int));
void bstree_apply_prefix(t_btree *root, void (*applyf)(int));
void bstree_apply_postfix(t_btree *root, void (*applyf)(int));
void apply_print(int item);
```

#### Вариант A. Классическая рекурсия

```c
#include <stdio.h>

#include "bst.h"

/* печатает значение узла */
void apply_print(int item) {
    printf("%d ", item);
}

/* infix: лево → корень → право (возрастание) */
void bstree_apply_infix(t_btree *root, void (*applyf)(int)) {
    if (root != NULL && applyf != NULL) {
        bstree_apply_infix(root->left, applyf);
        applyf(root->item);
        bstree_apply_infix(root->right, applyf);
    }
}

/* prefix: корень → лево → право */
void bstree_apply_prefix(t_btree *root, void (*applyf)(int)) {
    if (root != NULL && applyf != NULL) {
        applyf(root->item);
        bstree_apply_prefix(root->left, applyf);
        bstree_apply_prefix(root->right, applyf);
    }
}

/* postfix: лево → право → корень */
void bstree_apply_postfix(t_btree *root, void (*applyf)(int)) {
    if (root != NULL && applyf != NULL) {
        bstree_apply_postfix(root->left, applyf);
        bstree_apply_postfix(root->right, applyf);
        applyf(root->item);
    }
}
```

`src/bst_traverse_test.c`:

```c
#include <stdio.h>
#include <stdlib.h>

#include "bst.h"

void free_tree(t_btree *root);

int main(void) {
    t_btree *root;

    root = bstree_create_node(4);
    bstree_insert(root, 2, int_cmp);
    bstree_insert(root, 6, int_cmp);
    bstree_insert(root, 1, int_cmp);
    bstree_insert(root, 3, int_cmp);
    bstree_insert(root, 5, int_cmp);
    bstree_insert(root, 7, int_cmp);

    printf("infix:   ");
    bstree_apply_infix(root, apply_print);
    printf("\n");

    printf("prefix:  ");
    bstree_apply_prefix(root, apply_print);
    printf("\n");

    printf("postfix: ");
    bstree_apply_postfix(root, apply_print);
    printf("\n");

    free_tree(root);

    /* второй набор */
    root = bstree_create_node(5);
    bstree_insert(root, 3, int_cmp);
    bstree_insert(root, 7, int_cmp);
    bstree_insert(root, 1, int_cmp);
    printf("set2 infix:   ");
    bstree_apply_infix(root, apply_print);
    printf("\n");
    printf("set2 prefix:  ");
    bstree_apply_prefix(root, apply_print);
    printf("\n");
    printf("set2 postfix: ");
    bstree_apply_postfix(root, apply_print);
    printf("\n");
    free_tree(root);
    return 0;
}

void free_tree(t_btree *root) {
    if (root != NULL) {
        free_tree(root->left);
        free_tree(root->right);
        free(root);
    }
}
```

Для дерева `4` с детьми `2/6` и внуками `1,3 / 5,7` ожидается примерно:

```
infix:   1 2 3 4 5 6 7
prefix:  4 2 1 3 6 5 7
postfix: 1 3 2 5 7 6 4
```

**Как работает.** Каждый обход — рекурсия по одному и тому же дереву, меняется только момент вызова `applyf`. Infix для BST даёт отсортированный порядок — удобно «выстроить модули по id». Prefix показывает структуру «сверху вниз». Postfix обрабатывает детей раньше родителя (полезно, когда родителя можно трогать только после детей).

#### Вариант B. Обходы с дополнительной обёрткой `applyf == NULL`

Поведение то же; акцент на защитных проверках уже внутри каждой функции (как в A). Можно вынести общую проверку:

```c
void bstree_apply_infix(t_btree *root, void (*applyf)(int)) {
    if (applyf != NULL) {
        if (root != NULL) {
            bstree_apply_infix(root->left, applyf);
            applyf(root->item);
            bstree_apply_infix(root->right, applyf);
        }
    }
}
```

Смысл и вывод совпадают с вариантом A.

```bash
make bst_traverse_test
../build/Quest_8
```

```makefile
bst_create_test: bst.c bst_create_test.c bst.h
	mkdir -p $(BUILD)
	$(CC) $(CFLAGS) bst.c bst_create_test.c -o $(BUILD)/Quest_6

bst_insert_test: bst.c bst_insert_test.c bst.h
	mkdir -p $(BUILD)
	$(CC) $(CFLAGS) bst.c bst_insert_test.c -o $(BUILD)/Quest_7

bst_traverse_test: bst.c bst_traverse_test.c bst.h
	mkdir -p $(BUILD)
	$(CC) $(CFLAGS) bst.c bst_traverse_test.c -o $(BUILD)/Quest_8
```

---

## Чеклист перед сдачей

1. Ветка `develop`, файлы только в `src/` (+ Makefile).
2. `initialize_doors` не изменена.
3. Нет `system()` и аналогов.
4. Все `malloc`/`calloc` парные с `free` (списки, стек, маска документов, узлы дерева).
5. Тесты печатают `SUCCESS`/`FAIL` (квесты 2–3; для BST — понятный вывод создания/вставки/обхода).
6. Форматы вывода: двери `id, status`; лог `[LOG] HH:MM:SS ...`; документы `[%-15s : available|unavailable]`.
7. `clang-format`, для списка/стека — `cppcheck`, при возможности — `valgrind`.
8. Бинарники из `build/` в git не пушить.

---

## Карта файлов проекта

```
project/
├── build/
│   ├── Quest_1 … Quest_8
├── src/
│   ├── Makefile
│   ├── door_struct.h
│   ├── dmanager_module.c
│   ├── list.h / list.c / list_test.c
│   ├── stack.h / stack.c / stack_test.c
│   ├── print_module.h / print_module.c
│   ├── documentation_module.h / documentation_module.c
│   ├── main_module_entry_point.c
│   ├── bst.h / bst.c
│   ├── bst_create_test.c
│   ├── bst_insert_test.c
│   └── bst_traverse_test.c
└── materials/
```
