# Git — справочник

Обобщённые команды. Пополняем по мере встречи в работе и заданиях.

---

## Как пополнять

1. Найди раздел или создай новый.
2. Добавь строку: **команда → зачем → пример**.
3. Запиши в **Журнал** внизу, что добавил.

---

## Начало работы

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git clone <url>` | Склонировать репозиторий | `git clone https://github.com/user/repo.git` |
| `git init` | Создать новый репозиторий в текущей папке | `git init` |

---

## Состояние и история

| Команда                           | Зачем                                  | Пример                            |
| --------------------------------- | -------------------------------------- | --------------------------------- |
| `git status`                      | Изменённые файлы, stage, текущая ветка | `git status`                      |
| `git log`                         | История коммитов                       | `git log`                         |
| `git log --oneline`               | Короткая история                       | `git log --oneline`               |
| `git log --oneline --graph --all` | Дерево веток                           | `git log --oneline --graph --all` |
| `git diff`                        | Изменения vs последний коммит          | `git diff`                        |
| `git diff --staged`               | Что в stage перед коммитом             | `git diff --staged`               |
| `git show <commit>`               | Детали коммита                         | `git show a1b2c3d`                |

---

## Staging и коммит

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git add <файл>` | Добавить файл в stage | `git add README.md` |
| `git add <папка>/` | Добавить папку | `git add src/` |
| `git add .` | Все изменения в текущей директории и ниже | `git add .` |
| `git commit -m "..."` | Коммит staged-изменений | `git commit -m "fix handler"` |
| `git commit -am "..."` | Коммит только **уже отслеживаемых** файлов | `git commit -am "typo"` |

---

## Ветки

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git branch` | Список веток | `git branch` |
| `git branch <имя>` | Создать ветку | `git branch feature/login` |
| `git checkout <ветка>` | Переключиться | `git checkout main` |
| `git checkout -b <ветка>` | Создать и переключиться | `git checkout -b feature/login` |
| `git switch <ветка>` | Переключиться (новый синтаксис) | `git switch main` |
| `git switch -c <ветка>` | Создать и переключиться | `git switch -c feature/login` |
| `git branch -d <ветка>` | Удалить влитую ветку | `git branch -d feature/login` |
| `git branch -D <ветка>` | Принудительно удалить ветку | `git branch -D old-branch` |

---

## Слияние

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git merge <ветка>` | Влить ветку в **текущую** | `git merge feature/login` |

```bash
git checkout main
git merge feature/login
```

---

## Удалённый репозиторий

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git remote -v` | Список remote и URL | `git remote -v` |
| `git remote add <имя> <url>` | Добавить remote | `git remote add origin https://...` |
| `git fetch` | Скачать изменения без merge | `git fetch origin` |
| `git pull` | fetch + merge текущей ветки | `git pull origin main` |
| `git push <remote> <ветка>` | Отправить коммиты | `git push origin main` |
| `git push -u <remote> <ветка>` | Push + привязать upstream | `git push -u origin main` |

---

## Отмена и откат

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git restore <файл>` | Отменить изменения в файле | `git restore file.txt` |
| `git restore --staged <файл>` | Убрать файл из stage | `git restore --staged file.txt` |
| `git reset HEAD~1` | Отменить последний коммит, изменения остаются | `git reset HEAD~1` |
| `git reset --hard HEAD~1` | Отменить коммит и **стереть** изменения | `git reset --hard HEAD~1` |
| `git revert <commit>` | Новый коммит, отменяющий старый | `git revert a1b2c3d` |

---

## Stash (временно спрятать изменения)

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git stash` | Спрятать незакоммиченное | `git stash` |
| `git stash list` | Список stash | `git stash list` |
| `git stash pop` | Вернуть и удалить из списка | `git stash pop` |
| `git stash apply` | Вернуть, stash оставить | `git stash apply` |

---

## Теги

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git tag` | Список тегов | `git tag` |
| `git tag <имя>` | Лёгкий тег на текущем коммите | `git tag v1.0.0` |
| `git push origin <тег>` | Отправить тег на сервер | `git push origin v1.0.0` |
| `git push origin --tags` | Отправить все теги | `git push origin --tags` |

---

## Rebase

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git rebase <ветка>` | Перебазировать текущую ветку | `git rebase main` |
| `git rebase --continue` | Продолжить после разрешения конфликта | `git rebase --continue` |
| `git rebase --abort` | Отменить rebase | `git rebase --abort` |

---

## Конфликты merge / rebase

| Команда | Зачем | Пример |
|---------|-------|--------|
| `git status` | Найти файлы с `both modified` | `git status` |
| `git diff` | Посмотреть конфликтные участки | `git diff` |
| `git add <файл>` | Отметить конфликт как решённый | `git add src/file.txt` |
| `git merge --abort` | Отменить merge целиком | `git merge --abort` |
| `git rebase --continue` | Продолжить после fix при rebase | `git rebase --continue` |

Маркеры в файле:

```text
<<<<<<< HEAD
ваш код (текущая ветка)
=======
чужой код (вливаемая ветка)
>>>>>>> branch-name
```

Порядок действий:

1. Открыть файл в vim/nano.
2. Удалить маркеры `<<<<<<<`, `=======`, `>>>>>>>`.
3. Оставить нужный текст (или объединить оба варианта).
4. `git add <файл>`
5. `git commit` (merge) или `git rebase --continue` (rebase)
6. `git push origin <ветка>`

Проверка, что маркеров не осталось:

```bash
grep -r '<<<<<<<' .
```

---

## .gitignore

Файл `.gitignore` в корне — шаблоны файлов, которые git не отслеживает:

```gitignore
*.log
.env
node_modules/
```

### C / бассейн — не коммитить

```gitignore
*.o
*.out
a.out
hello
```

| Не пушить | Почему |
|-----------|--------|
| `*.o` | объектные файлы после компиляции |
| `a.out`, бинарники | исполняемые файлы |
| только `.c`, данные | правило T03D03 |

Проверка перед commit: `git status` — не должно быть `.o` / `a.out` в stage.

---

## Термины

| Термин | Смысл |
|--------|--------|
| Репозиторий | Проект + папка `.git` с историей |
| Коммит | Зафиксированный снимок изменений |
| Stage (index) | Область перед `git commit` |
| Ветка | Отдельная линия истории |
| merge | Слияние веток |
| origin | Имя remote по умолчанию |
| HEAD | Текущая позиция в истории; при конфликте — блок `<<<<<<< HEAD` |
| upstream | Связь локальной ветки с remote |
| merge conflict | Конфликт при слиянии двух версий файла |

---

## Журнал пополнений

| Дата | Что добавили |
|------|--------------|
| 2026-07-13 | Базовая структура справочника |
| 2026-07-13 | Конфликты merge: маркеры, `merge --abort`, grep-проверка, push после fix |
| 2026-07-13 | .gitignore для C: не пушить *.o, a.out, бинарники |
