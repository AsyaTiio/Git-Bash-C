# Прокачиваем знания о C

Краткий курс языка **C** на примерах квестов **Chapter III / Level 1 / Room 3** (T03D03).  
Стандарт **C11**, компилятор **gcc**, флаги `-Wall -Werror -Wextra`, стиль Google.  
После **каждого** квеста — commit + push только `.c` (и данные при необходимости) в `src/` ветки **`develop`**.  
Запрет: `system()` и любые системные вызовы. Часто вывод **без** `\n` в конце.

Связанные: [C_СПРАВОЧНИК](C_СПРАВОЧНИК.md), [T03D03_РАЗБОР_ЗАДАНИЯ](T03D03_РАЗБОР_ЗАДАНИЯ.md), [Циклы_рекурсия_и_функции_на_C](Циклы_рекурсия_и_функции_на_C.md) (Room 4).

---

## 0. Что такое C в двух словах

**C** — компилируемый язык: пишешь `.c` → `gcc` собирает исполняемый файл → запускаешь.

```bash
gcc -std=c11 -Wall -Werror -Wextra src/hello.c -o hello
./hello
```

**Флаги `-Wall -Werror -Wextra`** — это настройки `gcc`, которые включают строгую проверку кода. В условии их требуют, чтобы ловить ошибки раньше автотестов.

| Флаг | Что делает | Зачем |
|------|------------|--------|
| `-Wall` | Включает «все» основные предупреждения (unused variable, сравнение signed/unsigned и т.п.) | Показывает подозрительные места в коде |
| `-Wextra` | Дополнительные предупреждения сверх `-Wall` (например, неиспользуемые параметры) | Ещё жёстче ловит мелочи |
| `-Werror` | Любое предупреждение считается **ошибкой** — сборка не проходит | Нельзя «игнорировать» warning и сдать сомнительный код |

Без них программа может скомпилироваться, но с багом; с ними `gcc` просто не даст собрать «грязный» код.

### Как самой тестировать (от нуля)

`.c` файл **не запускается** двойным кликом как скрипт. Сначала компиляция → потом запуск получившейся программы.

**1.** Открой терминал в корне задания (там, где лежит папка `src/`):

```bash
cd путь/к/T03D03
```

**2.** Скомпилируй нужный файл (пример — Quest 1):

```bash
gcc -std=c11 -Wall -Werror -Wextra src/hello.c -o hello
```

- если ошибки — `gcc` напишет, в какой строке; исправь и повтори команду;
- если всё ок — появится исполняемый файл `hello` (в Windows часто `hello.exe`).

**3.** Запусти программу:

```bash
./hello          # Linux / macOS / Git Bash
hello.exe        # PowerShell / cmd на Windows
```

Для Quest 1 сразу увидишь: `Hello, AI!` — сравни с таблицей из условия.

**4.** Если программа ждёт ввод (Quest 2+), после запуска курсор мигает — набери данные и Enter:

```text
$ ./named_hello
123          ← ты вводишь
Hello, 123!  ← программа печатает
```

Или одной строкой (ввод «подсовывается» программе):

```bash
echo 123 | ./named_hello
echo "8 2" | ./arithmetic
echo "1.5 1.5" | ./crack
```

**5.** С `math.h` (Quest 5) не забудь `-lm` в конце:

```bash
gcc -std=c11 -Wall -Werror -Wextra src/important_function.c -o important -lm
echo 1 | ./important
```

**6.** Исполняемый файл (`hello`, `a.out`, `*.exe`) в git **не пушь** — только `src/*.c`.

| Идея | Смысл |
|------|--------|
| Исходник `.c` | Текст программы |
| Заголовок `.h` | Объявления функций (`stdio.h`, `math.h`) |
| `main` | Точка входа |
| `return 0` | Успешное завершение |

Минимальный каркас (Quest 1):

```c
#include <stdio.h>

int main(void) {
  printf("Hello, AI!");
  return 0;
}
```

---

## 1. Вывод на экран — `printf` (Quest 1)

`printf` печатает текст и подставляет значения по спецификаторам (`%d`, `%f`, …).

```c
printf("Hello, AI!");   // строка как есть
printf("Hello, %d!", name);  // подставить int
```

В этом квесте вывод **не** заканчивается переносом строки — не ставь `\n`, если тесты этого не требуют.

| Ввод | Вывод |
|------|-------|
| _(нет)_ | `Hello, AI!` |

---

## 2. Ввод — `scanf` и типы (Quest 2)

Переменная — именованная ячейка памяти. Для целого имени ИИ нужен тип `int`.

```c
int name = 0;
scanf("%d", &name);   // & — адрес: куда положить прочитанное
printf("Hello, %d!", name);
```

| Спецификатор | Тип | Пример |
|--------------|-----|--------|
| `%d` | `int` | `123` |
| `%lf` | `double` | `1.5` |

| Ввод | Вывод |
|------|-------|
| `123` | `Hello, 123!` |

---

## 3. Арифметика и ветвления (Quest 3)

Операторы: `+` `-` `*` `/`. Для `int` деление целочисленное: `8 / 2 → 4`, `3 / 2 → 1`.

**Проверка ввода:** `scanf` возвращает, сколько значений успешно прочитал.

```c
if (scanf("%d %d", &a, &b) != 2) {
  printf("n/a");
  return 0;
}
```

**Деление на ноль** — нельзя. Отдельная ветка `if (b == 0)`: частное печатаем как `n/a`, остальное считаем (`1 1 0 n/a`).

Порядок вывода: сумма, разность, произведение, частное — через пробел, **без** пробела в конце.

| Ввод | Вывод |
|------|-------|
| `8 2` | `10 6 16 4` |
| `1 0` | `1 1 0 n/a` |
| `3 2` | `5 1 6 1` |

---

## 4. Функции (Quest 4)

Функция — именованный кусок кода с входом и выходом. Максимум выносят отдельно.

```c
int max2(int a, int b);   // прототип (объявление)

int max2(int a, int b) {  // определение
  if (a >= b) {
    return a;
  }
  return b;
}
```

`12.3` в `%d` — не целое → `scanf` ≠ 2 → `n/a`. Равные числа → вывести это число.

| Ввод | Вывод |
|------|-------|
| `3 2` | `3` |
| `5 5` | `5` |
| `12.3 10` | `n/a` |

---

## 5. Вещественные числа и `math.h` (Quest 5)

`double` — числа с плавающей точкой. Нужны `pow`, иногда `isnan` / `isinf` → `#include <math.h>` и линковка `-lm`.

```bash
gcc -std=c11 -Wall -Werror -Wextra src/important_function.c -o prog -lm
```

Формула:

\[
y = 7\cdot10^{-3}\,x^{4} + \frac{(22.8\cdot x^{1/3}-10^{3})\cdot x + 3}{x^{2}/2} - x\cdot(10+x)^{2/x} - 1.01
\]

Вывод с одним знаком: `printf("%.1f", y);`  
При `x == 0` (или `x <= 0`) знаменатель и степень `2/x` ломаются → `n/a`. После счёта можно проверить `isnan` / `isinf`.

| Ввод | Вывод |
|------|-------|
| `1` | `-2070.4` |
| `0` | `n/a` |

---

## 6. Сравнение float через epsilon (Quest 6)

После вычислений результат почти никогда не равен `0.0` буквально. Сравнивают с порогом:

```c
#define EPSILON 1e-6

if (fabs(res) < EPSILON)
  printf("OK!");
```

Не читерить (не заменять всё на `printf("OK!")` в обход логики). Функцию `fun()` обычно не трогают.

---

## 7. Геометрия без `math.h` (Quest 7)

Окружность: \(x^{2} + y^{2} = 25\) (радиус 5). Точка **строго внутри** (`<`, не `<=`):

```c
if (x * x + y * y < 25.0)
  printf("GOTCHA");
else
  printf("MISS");
```

Достаточно `<stdio.h>`: квадрат через умножение, без `sqrt`. На границе (`5 0` → сумма квадратов `25`) — `MISS`. Плохой ввод → `n/a`.

| Ввод | Вывод |
|------|-------|
| `1.5 1.5` | `GOTCHA` |
| `5 0` | `MISS` |
| `a 1` | `n/a` |

---

## Шпаргалка по квестам

| # | Файл | Что учим |
|---|------|----------|
| 1 | `hello.c` | `#include`, `main`, `printf` |
| 2 | `named_hello.c` | `int`, `scanf`, `%d` |
| 3 | `arithmetic.c` | операторы, `if`, `n/a`, деление на 0 |
| 4 | `max.c` | своя функция, `if` |
| 5 | `important_function.c` | `double`, `math.h`, `pow`, `%.1f` |
| 6 | `float_compare.c` | epsilon, `fabs` |
| 7 | `crack.c` | условие внутри круга |

---

## Решения задач

Команды ниже — из папки **`src/`** (сначала `cd src`).  
После каждого квеста: проверить → `git add` → `commit` → `push` в **`develop`**.  
В git только `.c` — не добавляй `hello`, `a.out`, `*.exe`, `*.o`.

### Перед первым `hello.c`

Сначала не код, а репозиторий:

1. Клонировать проект с GitLab  
2. Создать ветку `develop` и работать только в ней  
3. Писать файлы в `src/`  
4. Потом уже создать `src/hello.c`

```bash
git clone <url-репозитория>
cd <имя-репозитория>
git checkout -b develop
cd src
# здесь создаёшь hello.c
```

Пушить только исходники (`.c`), не бинарники. После каждого квеста: `add` → `commit` → `push origin develop`.

---

### Quest 1 — `hello.c`

```c
#include <stdio.h>

int main(void) {
  printf("Hello, AI!");
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra hello.c -o hello
./hello
# ожидается: Hello, AI!

git add hello.c
git commit -m "Quest 1: hello.c"
git push origin develop
```

---

### Quest 2 — `named_hello.c`

```c
#include <stdio.h>

int main(void) {
  int name = 0;
  if (scanf("%d", &name) != 1) {
    return 0;
  }
  printf("Hello, %d!", name);
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra named_hello.c -o named_hello
echo 123 | ./named_hello
# ожидается: Hello, 123!

git add named_hello.c
git commit -m "Quest 2: named_hello.c"
git push origin develop
```

---

### Quest 3 — `arithmetic.c`

```c
#include <stdio.h>

int main(void) {
  int a = 0, b = 0;
  if (scanf("%d %d", &a, &b) != 2) {
    printf("n/a");
    return 0;
  }
  int sum = a + b;
  int diff = a - b;
  int prod = a * b;
  if (b == 0) {
    printf("%d %d %d n/a", sum, diff, prod);
    return 0;
  }
  printf("%d %d %d %d", sum, diff, prod, a / b);
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra arithmetic.c -o arithmetic
echo "8 2" | ./arithmetic
# ожидается: 10 6 16 4
echo "1 0" | ./arithmetic
# ожидается: 1 1 0 n/a
echo "3 2" | ./arithmetic
# ожидается: 5 1 6 1

git add arithmetic.c
git commit -m "Quest 3: arithmetic.c"
git push origin develop
```

---

### Quest 4 — `max.c`

```c
#include <stdio.h>

int max2(int a, int b);

int main(void) {
  int a = 0, b = 0;
  if (scanf("%d %d", &a, &b) != 2) {
    printf("n/a");
    return 0;
  }
  printf("%d", max2(a, b));
  return 0;
}

int max2(int a, int b) {
  if (a >= b) {
    return a;
  }
  return b;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra max.c -o max
echo "3 2" | ./max
# ожидается: 3
echo "5 5" | ./max
# ожидается: 5
echo "12.3 10" | ./max
# ожидается: n/a

git add max.c
git commit -m "Quest 4: max.c"
git push origin develop
```

---

### Quest 5 — `important_function.c`

```c
#include <math.h>
#include <stdio.h>

int main(void) {
  double x = 0.0;
  if (scanf("%lf", &x) != 1) {
    printf("n/a");
    return 0;
  }
  if (x <= 0.0) {
    printf("n/a");
    return 0;
  }
  double y = 7e-3 * pow(x, 4)
      + ((22.8 * pow(x, 1.0 / 3.0) - 1e3) * x + 3) / (x * x / 2.0)
      - x * pow(10.0 + x, 2.0 / x) - 1.01;
  if (isnan(y) || isinf(y)) {
    printf("n/a");
    return 0;
  }
  printf("%.1f", y);
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra important_function.c -o important -lm
echo 1 | ./important
# ожидается: -2070.4
echo 0 | ./important
# ожидается: n/a

git add important_function.c
git commit -m "Quest 5: important_function.c"
git push origin develop
```

---

### Quest 6 — `float_compare.c`

Файл уже есть в репозитории. Меняешь **только** `if` (комментарий `CHANGE THIS IF`). Функцию `fun()` не трогай.

Исходный `if (res == 0)` не сработает: `fun()` возвращает ~`2.2e-11` (почти ноль, но не ровно). Нужно сравнение с эпсилоном ~`1e-6`:

```c
#include <stdio.h>
#include <math.h>

double fun();

int main() {
  double res = fun();
  // CHANGE THIS IF - AI
  if ((res > -0.000001) && (res < 0.000001))
    printf("OK!");
  else
    printf("%lf", res);
  return 0;
}

// DO NOT TOUCH THIS FUNCTION - AI
double fun() {
  return (1.0 / 13) * (pow(((2 - 1.0) / (2 + 1.0)), 20));
}
```

Эквивалент: `if (fabs(res) < 1e-6)`.

```bash
gcc -std=c11 -Wall -Werror -Wextra float_compare.c -o float_compare -lm
./float_compare
# ожидается: OK!

git add float_compare.c
git commit -m "Quest 6: float_compare.c"
git push origin develop
```

---

### Quest 7 — `crack.c`

Строго внутри: `< 25`. На окружности (`5 0`) — `MISS`.

```c
#include <stdio.h>

int main(void) {
  double x = 0.0, y = 0.0;
  if (scanf("%lf %lf", &x, &y) != 2) {
    printf("n/a");
    return 0;
  }
  if (x * x + y * y < 25.0)
    printf("GOTCHA");
  else
    printf("MISS");
  return 0;
}
```

```bash
gcc -std=c11 -Wall -Werror -Wextra crack.c -o crack
echo "1.5 1.5" | ./crack
# ожидается: GOTCHA
echo "5 0" | ./crack
# ожидается: MISS

git add crack.c
git commit -m "Quest 7: crack.c"
git push origin develop
```
