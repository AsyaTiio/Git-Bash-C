# Циклы, рекурсия и функции на C

Краткий разбор квестов **Chapter III / Level 1 / Room 4**.  
Стандарт **C11**, `gcc` с флагами `-Wall -Werror -Wextra`, стиль Google.  
После каждого квеста: commit + push исходников из `src/` в ветку **`develop`**.

**Общие запреты дня:** массивы, динамическая память, `system()` и аналоги.  
**Разрешено:** `<stdio.h>`, `<math.h>`.

Связанные: [C_СПРАВОЧНИК](C_СПРАВОЧНИК.md), [Прокачиваем_знания_о_C](Прокачиваем_знания_о_C.md).

---

## 0. Правила дня в двух словах

| Правило | Зачем |
|---------|--------|
| Нет массивов `int a[10]` и `malloc` | Учимся обходиться циклами и пересчётом на лету |
| Нет `/` и `%` в Quest 1 | Деление только через вычитание (как на «Baby» 1948) |
| Ошибка ввода → `n/a` | Автотесты ждут ровно эту строку |
| Отдельные функции | Требование условия + принцип «один вход — один выход» |

```bash
cd путь/к/репозиторию
git checkout -b develop   # если ещё нет
cd src
```

Компиляция:

```bash
gcc -std=c11 -Wall -Werror -Wextra file.c -o prog
# если есть math.h — в конце добавить -lm
./prog
```

---

## 1. Деление через вычитание (Quest 1)

Операторы `/` и `%` запрещены. Идея:

- **частное** — сколько раз удалось вычесть делитель из делимого;
- **остаток** — то, что осталось после этих вычитаний.

```c
/* 17 / 5 → 3,  17 % 5 → 2 */
int divide(int a, int b) {   /* a >= 0, b > 0 */
  int q = 0;
  while (a >= b) {
    a -= b;
    q++;
  }
  return q;
}

int modulo(int a, int b) {
  while (a >= b) {
    a -= b;
  }
  return a;
}
```

**Наибольший простой делитель** = наибольший простой множитель.

Алгоритм (без массивов):

1. Взять модуль числа (`-4` → `4`).
2. Для `n ≤ 1` — ошибка (`n/a`).
3. Перебирать делитель `d = 2, 3, 4, …` пока `(long long)d * d ≤ n` (чтобы не переполнить `int`).
4. Пока `n` делится на `d` — запомнить `d` и заменить `n` на `n / d`.
5. Если в конце осталось `n > 1` — это тоже простой множитель (и он больше всех предыдущих).

| Ввод | Вывод | Почему |
|------|-------|--------|
| `100` | `5` | \(100 = 2^2 \cdot 5^2\) |
| `-4` | `2` | \(4 = 2^2\) |
| `7` | `7` | само число простое |
| `1` / `0` / мусор | `n/a` | нет простого делителя / плохой ввод |

---

## 2. Аргументы командной строки и ASCII (Quest 2)

`main` может принимать аргументы запуска:

```c
int main(int argc, char *argv[])
```

| Что | Смысл |
|-----|--------|
| `argc` | сколько аргументов (включая имя программы) |
| `argv[0]` | имя программы |
| `argv[1]` | первый параметр — у нас `"0"` или `"1"` |

Запуск:

```bash
./char_decode 0    # кодирование
./char_decode 1    # декодирование
```

**ASCII:** символ ↔ число. `'A' = 65 = 0x41`, `'W' = 87 = 0x57`.

Шестнадцатеричная цифра без массива `"0123456789ABCDEF"`:

```c
char to_hex(int n) {
  if (n < 10) return '0' + n;
  return 'A' + n - 10;
}
```

Байт в две hex-цифры — сдвигами (без `/` и `%`, хотя здесь они уже разрешены):

```c
putchar(to_hex(u >> 4));   /* старшие 4 бита */
putchar(to_hex(u & 15));   /* младшие 4 бита */
```

**Формат ввода жёсткий:** символы/пары через пробел.  
`WORLD` без пробелов → `n/a`. `48454C4C4F` слитно → `n/a`.

Важно: сначала проверить, что после символа идёт пробел или конец строки, и **только потом** печатать — иначе при ошибке получится `57n/a`.

| Режим | Ввод | Вывод |
|-------|------|-------|
| `0` | `W O R L D` | `57 4F 52 4C 44` |
| `0` | `WORLD` | `n/a` |
| `1` | `48 45 4C 4C 4F` | `H E L L O` |
| `1` | `48454C4C4F` | `n/a` |

Расшифровка сюжета: `46 49 42 4F 4E 41 43 43 49 32 31` → `F I B O N A C C I 2 1`.

---

## 3. Рекурсия и Фибоначчи (Quest 3)

**Рекурсия** — функция вызывает сама себя с более простой задачей.

```text
fib(5) = fib(4) + fib(3)
       = (fib(3)+fib(2)) + (fib(2)+fib(1))
       = …
```

База: `fib(0) = 0`, `fib(1) = 1` (или `fib(1)=fib(2)=1` — для 21-го числа ответ тот же).

| Ввод | Вывод |
|------|-------|
| `21` | `10946` |

Отрицательное / не целое → `n/a`.  
На больших `n` наивная рекурсия тормозит — для 21 ещё нормально.

---

## 4. Таблица функций без массивов (Quest 4)

Интервал \([-\pi, \pi]\), ровно **42** точки → **41** шаг:

\[
\text{step} = \frac{2\pi}{41},\quad x_i = -\pi + i \cdot \text{step},\quad i = 0..41
\]

\(\pi\) — в `#define` или переменную, не копипастить по коду:

```c
#define PI 3.14159265358979323846
```

Формулы (единичные параметры):

| Функция | Формула | Когда `-` |
|---------|---------|-----------|
| Верзьера Аньези | \(y = \dfrac{1}{1 + x^2}\) | всегда определена |
| Лемниската Бернулли | \(y = \sqrt{\sqrt{1 + 4x^2} - x^2 - 1}\) | если под корнем \(< 0\) |
| Квадратичная гипербола | \(y = \dfrac{1}{x^2}\) | при \(x = 0\) |

Формат строки:

```text
x | y1 | y2 | y3
```

Все числа — `double`, `%.7f`, разделитель ` | ` (пробел-палка-пробел).

Массивы нельзя → в цикле считаешь точку и сразу `printf`, ничего не сохраняешь.

Перенаправление в файл (запись из программы не нужна):

```bash
mkdir -p data
./door_functions > data/door_data.txt
```

Первые строки должны совпасть с примером:

```text
-3.1415927 | 0.0919997 | - | 0.1013212
-2.9883442 | 0.1007029 | - | 0.1119796
```

---

## 5. График в терминале (Quest 5)

Тот же код + отрисовка `*` в сетке **42 × 21** (по оси X — 42 отсечки, по Y — 21 уровень).  
Три графика подряд, один под другим. Оси как угодно — требований к оформлению нет.

Без массива: для каждой строки `row` и столбца `col` заново считаешь \(y(x_{col})\) и смотришь, попадает ли точка в эту строку.

Идея масштаба по Y: найти максимум функции на 42 точках, затем

```c
y_row = (int)(y / ymax * 20.0 + 0.5);  /* 0..20 */
```

Файл: `src/door_functions_print.c` (таблица + графики).

---

## Шпаргалка

| # | Файл | Что учим |
|---|------|----------|
| 1 | `1948.c` | циклы, деление вычитанием, простые множители |
| 2 | `char_decode.c` | `argc`/`argv`, ASCII/hex, `getchar` |
| 3 | `quest3.c` | рекурсия, Фибоначчи |
| 4 | `door_functions.c` + `data/door_data.txt` | `double`, формулы, 42 точки |
| 5* | `door_functions_print.c` | ASCII-график `*` |

---

## Решения задач

Команды — из папки **`src/`**.  
В git только исходники (и `data/door_data.txt` для Q4). Не пушь бинарники.

```bash
git clone <url>
cd <репозиторий>
git checkout -b develop
cd src
```

---

### Quest 1 — `1948.c`

Функции пишем **до** `main` — тогда прототипы не нужны.  
Комментарии ниже — для разбора; перед сдачей можно оставить или укоротить.

```c
#include <stdio.h>

/* Целочисленное деление a / b через вычитание.
 * Пример: 17 и 5 → вычитаем 5 три раза → частное 3.
 * Условие: a >= 0, b > 0. Операторы / и % запрещены. */
int divide(int a, int b) {
  int q = 0;              /* счётчик: сколько раз удалось вычесть */
  while (a >= b) {        /* пока делимое ещё не меньше делителя */
    a -= b;               /* вычитаем b из a */
    q++;                  /* +1 к частному */
  }
  return q;
}

/* Остаток a % b через вычитание.
 * Пример: 17 и 5 → после трёх вычитаний остаётся 2. */
int modulo(int a, int b) {
  while (a >= b) {
    a -= b;
  }
  return a;               /* то, что не делится нацело */
}

/* Наибольший простой делитель (он же наибольший простой множитель).
 * Возвращает -1, если ответа нет (0, ±1 и т.п.). */
int largest_prime_divisor(int n) {
  int largest = 1;        /* пока лучший найденный множитель */
  int d = 2;              /* текущий кандидат в делители */

  /* INT_MIN нельзя безопасно обратить в int: |-2^31| = 2^31.
   * Единственный простой множитель — двойка. */
  if (n == -2147483647 - 1) {
    return 2;
  }
  if (n < 0) {
    n = -n;               /* для -4 ищем делители числа 4 */
  }
  if (n <= 1) {
    return -1;            /* у 0 и 1 нет простого делителя */
  }

  /* Достаточно d до sqrt(n). Считаем d*d в long long,
   * иначе на больших n умножение int переполнится. */
  while ((long long)d * d <= n) {
    /* пока n делится на d — d простой множитель (идём по возрастанию) */
    while (modulo(n, d) == 0) {
      largest = d;        /* запоминаем: дальше d только растёт */
      n = divide(n, d);   /* «вычёркиваем» множитель: 100/2 → 50/2 → 25 */
    }
    d++;                  /* следующий кандидат: 3, 4, 5, ... */
  }

  /* Если после разложения осталось n > 1 — это последний простой множитель
   * (и он больше всех предыдущих). Пример: 100 → остаётся 5. */
  if (n > 1) {
    largest = n;
  }
  return largest;
}

int main(void) {
  int a = 0;
  int result = 0;

  /* Только проверка scanf: лишний getchar ломает тесты с пробелом/без \n */
  if (scanf("%d", &a) != 1) {
    printf("n/a");
    return 0;
  }

  result = largest_prime_divisor(a);
  if (result < 0) {
    printf("n/a");
  } else {
    printf("%d", result); /* без \n — как в примерах условия */
  }
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra 1948.c -o 1948
echo 100 | ./1948
# 5
echo -4 | ./1948
# 2

git add 1948.c
git commit -m "Quest 1: 1948.c"
git push origin develop
```

---

### Quest 2 — `char_decode.c`

```c
#include <stdio.h>

/* Цифра 0..15 → символ '0'..'9' или 'A'..'F' (без массива строк). */
char to_hex(int n) {
  if (n < 10) {
    return (char)('0' + n);       /* 0→'0', 9→'9' */
  }
  return (char)('A' + n - 10);    /* 10→'A', 15→'F' */
}

/* Символ hex → число 0..15; мусор → -1. */
int from_hex(char c) {
  if (c >= '0' && c <= '9') {
    return c - '0';
  }
  if (c >= 'A' && c <= 'F') {
    return c - 'A' + 10;
  }
  if (c >= 'a' && c <= 'f') {
    return c - 'a' + 10;          /* на всякий случай принимаем и строчные */
  }
  return -1;
}

/* Один символ → две hex-цифры. 'W'=87=0x57 → печатаем "57".
 * >> 4 — старшие 4 бита, & 15 — младшие (это 0xF). */
void encode_char(char c) {
  unsigned char u = (unsigned char)c; /* чтобы отрицательный char не мешал */
  putchar(to_hex(u >> 4));
  putchar(to_hex(u & 15));
}

/* Режим 0: "W O R L D" → "57 4F 52 4C 44"
 * Возврат 0 — ок, 1 — ошибка формата (потом main напечатает n/a).
 *
 * Важно: сначала смотрим следующий символ, и только потом печатаем.
 * Иначе на "WORLD" успели бы вывести "57" и получили бы "57n/a". */
int do_encode(void) {
  int first = 1;                  /* ещё ничего не печатали — пробел не нужен */
  int c = getchar();
  if (c == '\n' || c == EOF) {
    return 1;                     /* пустой ввод */
  }

  while (1) {
    if (c == ' ') {
      return 1;                   /* два пробела подряд / пробел вместо буквы */
    }

    int next = getchar();
    /* После буквы обязан быть пробел или конец строки.
     * Если сразу другая буква ("WORLD") — ошибка. */
    if (next != ' ' && next != '\n' && next != EOF) {
      return 1;
    }

    if (!first) {
      putchar(' ');               /* пробел между парами, не в начале */
    }
    encode_char((char)c);
    first = 0;

    if (next == '\n' || next == EOF) {
      return 0;                   /* нормальный конец: ... D\n */
    }

    /* next был пробел — читаем следующий символ */
    c = getchar();
    if (c == '\n' || c == EOF) {
      return 1;                   /* висячий пробел в конце */
    }
  }
}

/* Режим 1: "48 45 4C 4C 4F" → "H E L L O"
 * Каждая «ячейка» — ровно две hex-цифры, между ними пробел. */
int do_decode(void) {
  int first = 1;
  int h1 = getchar();
  if (h1 == '\n' || h1 == EOF) {
    return 1;
  }

  while (1) {
    int h2 = getchar();
    /* Вторая цифра обязана быть; если пробел/конец — пара обрезана */
    if (h2 == ' ' || h2 == '\n' || h2 == EOF) {
      return 1;
    }
    if (from_hex((char)h1) < 0 || from_hex((char)h2) < 0) {
      return 1;                   /* не hex-символ */
    }

    int next = getchar();
    /* После пары — пробел или конец. Слипшиеся "4845..." → ошибка. */
    if (next != ' ' && next != '\n' && next != EOF) {
      return 1;
    }

    if (!first) {
      putchar(' ');
    }
    /* Собираем байт: старшая тетрада << 4, младшая через |.
     * '4' и '8' → 4*16+8 = 72 = 'H' */
    putchar((char)((from_hex((char)h1) << 4) | from_hex((char)h2)));
    first = 0;

    if (next == '\n' || next == EOF) {
      return 0;
    }

    h1 = getchar();               /* начало следующей пары */
    if (h1 == '\n' || h1 == EOF) {
      return 1;                   /* пробел в конце без пары */
    }
  }
}

int main(int argc, char *argv[]) {
  int err = 0;

  /* Нужен ровно один аргумент: ./char_decode 0  или  ./char_decode 1 */
  if (argc != 2) {
    printf("n/a");
    return 0;
  }

  /* argv[1] — строка; проверяем, что это ровно "0" или ровно "1" */
  if (argv[1][0] == '0' && argv[1][1] == '\0') {
    err = do_encode();
  } else if (argv[1][0] == '1' && argv[1][1] == '\0') {
    err = do_decode();
  } else {
    err = 1;
  }

  if (err) {
    printf("n/a");
  }
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra char_decode.c -o char_decode
echo "W O R L D" | ./char_decode 0
# 57 4F 52 4C 44
echo "WORLD" | ./char_decode 0
# n/a
echo "48 45 4C 4C 4F" | ./char_decode 1
# H E L L O
echo "48454C4C4F" | ./char_decode 1
# n/a

# сюжетное послание:
echo "46 49 42 4F 4E 41 43 43 49 32 31" | ./char_decode 1
# F I B O N A C C I 2 1

git add char_decode.c
git commit -m "Quest 2: char_decode.c"
git push origin develop
```

---

### Quest 3 — `quest3.c`

```c
#include <stdio.h>

/* n-е число Фибоначчи рекурсией.
 * База: F(0)=0, F(1)=1.
 * Шаг: F(n) = F(n-1) + F(n-2).
 *
 * Пример разворачивания:
 *   fib(4) = fib(3) + fib(2)
 *          = (fib(2)+fib(1)) + (fib(1)+fib(0))
 *          = (1+1) + (1+0) = 3
 *
 * Для n=21 ответ 10946. Наивная рекурсия медленная на больших n —
 * для 21 ещё хватает. */
int fibonacci(int n) {
  if (n == 0) {
    return 0;
  }
  if (n == 1) {
    return 1;
  }
  return fibonacci(n - 1) + fibonacci(n - 2);
}

int main(void) {
  int n = 0;

  if (scanf("%d", &n) != 1) {
    printf("n/a");
    return 0;
  }
  if (n < 0) {
    printf("n/a");              /* отрицательный индекс не определён */
    return 0;
  }

  printf("%d", fibonacci(n));
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra quest3.c -o quest3
echo 21 | ./quest3
# 10946

git add quest3.c
git commit -m "Quest 3: quest3.c"
git push origin develop
```

---

### Quest 4 — `door_functions.c`

```c
#include <math.h>
#include <stdio.h>

/* π один раз — чтобы не копировать по коду (требование условия).
 * 20 знаков достаточно. */
#define PI 3.14159265358979323846

/* Верзьера Аньези с единичным диаметром: y = 1 / (1 + x²).
 * Определена при любом x. */
double agnesi(double x) { return 1.0 / (1.0 + x * x); }

/* Лемниската Бернулли (положительная полуплоскость):
 *   y = sqrt( sqrt(1 + 4x²) - x² - 1 )
 * ok=0 → в этой точке функция не определена (под корнем < 0) → в таблице "-".
 * Указатель ok — «вернуть наружу» флаг, без второго return-значения. */
double bernoulli(double x, int *ok) {
  double inner = sqrt(1.0 + 4.0 * x * x) - x * x - 1.0;
  if (inner < 0.0) {
    *ok = 0;
    return 0.0;                 /* само число не важно — смотрим на *ok */
  }
  *ok = 1;
  return sqrt(inner);
}

/* Квадратичная гипербола: y = 1 / x². При x≈0 — не определена. */
double hyperbola(double x, int *ok) {
  if (fabs(x) < 1e-12) {        /* сравнение с нулём через эпсилон */
    *ok = 0;
    return 0.0;
  }
  *ok = 1;
  return 1.0 / (x * x);
}

/* Печать числа с 7 знаками или "-" если функция не определена. */
void print_value(double y, int ok) {
  if (ok) {
    printf("%.7f", y);
  } else {
    printf("-");
  }
}

int main(void) {
  int i;
  /* 42 точки на отрезке [-π; π] включительно → 41 промежуток.
   * step = 2π / 41. Массивы запрещены — считаем и сразу печатаем. */
  double step = (2.0 * PI) / 41.0;

  for (i = 0; i < 42; i++) {
    double x = -PI + step * (double)i; /* i=0 → -π, i=41 → +π */

    int ok_b = 0;
    int ok_h = 0;
    double y1 = agnesi(x);             /* всегда ok */
    double y2 = bernoulli(x, &ok_b);   /* &ok_b — передаём адрес флага */
    double y3 = hyperbola(x, &ok_h);

    /* Формат: "x | y1 | y2 | y3" с пробелами вокруг | */
    printf("%.7f | ", x);
    print_value(y1, 1);
    printf(" | ");
    print_value(y2, ok_b);
    printf(" | ");
    print_value(y3, ok_h);
    putchar('\n');
  }
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra door_functions.c -o door_functions -lm
mkdir -p data
./door_functions > data/door_data.txt
head -n 2 data/door_data.txt
# -3.1415927 | 0.0919997 | - | 0.1013212
# -2.9883442 | 0.1007029 | - | 0.1119796
wc -l data/door_data.txt
# 42

git add door_functions.c data/door_data.txt
git commit -m "Quest 4: door_functions.c and door_data.txt"
git push origin develop
```

---

### Quest 5 — `door_functions_print.c`

Таблица как в Q4, плюс три графика `*` (42×21) друг под другом.

```c
#include <math.h>
#include <stdio.h>

#define PI 3.14159265358979323846

double agnesi(double x) { return 1.0 / (1.0 + x * x); }

double bernoulli(double x, int *ok) {
  double inner = sqrt(1.0 + 4.0 * x * x) - x * x - 1.0;
  if (inner < 0.0) {
    *ok = 0;
    return 0.0;
  }
  *ok = 1;
  return sqrt(inner);
}

double hyperbola(double x, int *ok) {
  if (fabs(x) < 1e-12) {
    *ok = 0;
    return 0.0;
  }
  *ok = 1;
  return 1.0 / (x * x);
}

void print_value(double y, int ok) {
  if (ok) {
    printf("%.7f", y);
  } else {
    printf("-");
  }
}

/* Та же таблица, что в Quest 4 — вынесена в функцию, чтобы main был короче. */
void print_table(void) {
  int i;
  double step = (2.0 * PI) / 41.0;
  for (i = 0; i < 42; i++) {
    double x = -PI + step * (double)i;
    int ok_b = 0;
    int ok_h = 0;
    double y1 = agnesi(x);
    double y2 = bernoulli(x, &ok_b);
    double y3 = hyperbola(x, &ok_h);
    printf("%.7f | ", x);
    print_value(y1, 1);
    printf(" | ");
    print_value(y2, ok_b);
    printf(" | ");
    print_value(y3, ok_h);
    putchar('\n');
  }
}

/* which: 0 — Аньези, 1 — Бернулли, 2 — гипербола.
 * Удобно, чтобы не копировать три почти одинаковых цикла рисования. */
double value_at(int which, double x, int *ok) {
  *ok = 1;
  if (which == 0) {
    return agnesi(x);
  }
  if (which == 1) {
    return bernoulli(x, ok);
  }
  return hyperbola(x, ok);
}

/* Максимум функции на тех же 42 точках — чтобы масштабировать ось Y.
 * Массивов нет: просто ещё один проход с пересчётом. */
double max_of(int which) {
  int i;
  double step = (2.0 * PI) / 41.0;
  double mx = 0.0;
  for (i = 0; i < 42; i++) {
    int ok = 0;
    double x = -PI + step * (double)i;
    double y = value_at(which, x, &ok);
    if (ok && y > mx) {
      mx = y;
    }
  }
  return mx;
}

/* Рисуем сетку 42 столбца × 21 строка символами '*'.
 * Строки идут сверху вниз (row 20 → 0): больший y выше на экране.
 * Без массива: для каждой клетки (row, col) заново считаем y(x_col)
 * и смотрим, попадает ли точка в эту строку. */
void draw_graph(int which) {
  int row;
  int col;
  double step = (2.0 * PI) / 41.0;
  double mx = max_of(which);
  if (mx <= 0.0) {
    mx = 1.0;                   /* защита от деления на ноль */
  }

  for (row = 20; row >= 0; row--) {     /* 21 уровень: 20..0 */
    for (col = 0; col < 42; col++) {    /* 42 отсечки по X */
      int ok = 0;
      double x = -PI + step * (double)col;
      double y = value_at(which, x, &ok);
      if (ok) {
        /* Нормируем y в диапазон 0..20 и округляем */
        int y_row = (int)(y / mx * 20.0 + 0.5);
        if (y_row == row) {
          putchar('*');         /* точка графика в этой клетке */
        } else {
          putchar(' ');
        }
      } else {
        putchar(' ');           /* функция не определена — пусто */
      }
    }
    putchar('\n');              /* конец строки сетки */
  }
}

int main(void) {
  print_table();                /* сначала числа */
  putchar('\n');
  draw_graph(0);                /* Аньези */
  putchar('\n');
  draw_graph(1);                /* Бернулли */
  putchar('\n');
  draw_graph(2);                /* гипербола */
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra door_functions_print.c -o door_print -lm
./door_print

git add door_functions_print.c
git commit -m "Quest 5: door_functions_print.c"
git push origin develop
```

---

## Чеклист перед сдачей

1. В коде нет `/` и `%` в `1948.c`, нет массивов нигде.  
2. Есть отдельные функции там, где просят.  
3. Сборка с `-Wall -Werror -Wextra` без предупреждений.  
4. Сверка с таблицами из условия.  
5. Стиль (отступы 2 пробела, имена) — локальные тесты из `materials/`, если есть.  
6. В репозитории только `.c` (+ `door_data.txt`), не `a.out` / `*.exe`.
