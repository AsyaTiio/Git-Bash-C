# Bonus Quest 10. Gitlab

Результат: `src/gitlab_manual.md` — 4 пункта, каждый = заголовок + скриншот. Push в `develop`.

Скриншоты снять в отдельном репозитории GitLab, мануал сдать в репозитории задания.

---

## 1. Репозиторий + `.gitignore` + `README.md`

1. GitLab → **New project** → **Create blank project**.
2. Включить **Initialize repository with a README**.
3. Добавить `.gitignore` (шаблон при создании или **Files → New file** → `.gitignore`).
4. Скриншот: страница проекта с README и `.gitignore`.

---

## 2. Ветки `master`, `develop`, `feature` от `develop`

1. **Repository → Branches**.
2. Есть `master` (если `main` — создать `master`).
3. **New branch**: `develop` from `master`.
4. **New branch**: `feature/manual` from `develop`.
5. Скриншот: список веток `master`, `develop`, `feature/manual`.

```bash
git checkout master
git checkout -b develop && git push -u origin develop
git checkout -b feature/manual && git push -u origin feature/manual
```

---

## 3. Merge request в `develop`

1. В `feature/manual` — коммит (любое изменение) → push.
2. **Merge requests → New merge request**.
3. Source: `feature/manual` → Target: `develop` → **Create**.
4. Скриншот: страница MR.

---

## 4. Issue + комментарий

1. **Issues → New issue** → title про создание мануала → **Create**.
2. Добавить комментарий → **Comment**.
3. Скриншот: issue с комментарием.

---

## 5. Файл в репозитории задания

```bash
git checkout develop
mkdir -p src/img
# положить скриншоты в src/img/
```

`src/gitlab_manual.md`:

```markdown
# GitLab Manual

## 1. Создание личного репозитория с .gitignore и README.md

1. New project → Create blank project.
2. Указать имя проекта.
3. Включить Initialize repository with a README.
4. Выбрать шаблон .gitignore (или создать файл `.gitignore` после).
5. Create project.

![Создание репозитория](img/01_new_repo.png)

## 2. Создание веток master, develop и feature от develop

1. Repository → Branches.
2. Убедиться, что есть ветка `master`.
3. New branch → имя `develop`, Create from: `master`.
4. New branch → имя `feature/manual`, Create from: `develop`.

![Ветки](img/02_branches.png)

## 3. Создание merge request в develop

1. В ветке `feature/manual` сделать коммит и push.
2. Merge requests → New merge request.
3. Source branch: `feature/manual`.
4. Target branch: `develop`.
5. Create merge request.

![Merge request](img/03_merge_request.png)

## 4. Создание issue на мануал и комментария к issue

1. Issues → New issue.
2. Title: например, «Create gitlab_manual.md».
3. Create issue.
4. Внизу страницы написать комментарий → Comment.

![Issue и комментарий](img/04_issue.png)
```

```bash
git add src/gitlab_manual.md src/img/
git commit -m "Quest 10: gitlab manual"
git push origin develop
```

