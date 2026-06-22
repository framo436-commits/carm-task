# carm task

Менеджер задач (liquid-glass), один самодостаточный файл `index.html`.
RU/EN, отчёты + диаграмма Ганта, общая облачная синхронизация через Supabase.

---

## 1. Создать базу в Supabase (общие данные для теста департамента)

1. Зайди на https://supabase.com → **New project** (бесплатный тариф). Запомни пароль БД.
2. Когда проект создан: **SQL Editor → New query**, вставь и выполни:

```sql
create table if not exists app_state (
  id text primary key,
  data jsonb,
  client text,
  updated_at timestamptz default now()
);

alter table app_state enable row level security;

-- Для внутреннего теста: разрешаем чтение/запись анонимным клиентам.
create policy "test_all_access" on app_state
  for all to anon using (true) with check (true);

-- Включить realtime (живое обновление у всех)
alter publication supabase_realtime add table app_state;
```

3. **Settings → API**, скопируй два значения:
   - **Project URL** (вид `https://xxxx.supabase.co`)
   - **anon public** key (длинный ключ, начинается с `eyJ...`)

> anon-ключ публичный — его безопасно держать во фронтенде. Доступ ограничивается правилами RLS выше (сейчас открыт для теста; для боевого режима их нужно ужесточить).

---

## 2. Вписать ключи в приложение

В `index.html` найди блок `const CLOUD = {` (раздел *Cloud sync*) и заполни:

```js
const CLOUD = {
  url: 'https://xxxx.supabase.co',
  key: 'eyJ...твой anon key...',
};
```

Сохрани файл. Пустые значения = приложение работает локально (у каждого своя копия);
заполненные = все видят общие данные и правки синхронизируются в реальном времени.

---

## 3. Выложить на GitHub Pages

```bash
# в этой папке уже инициализирован git и сделан первый коммит
# 1) создай ПУСТОЙ репозиторий на github.com (например carm-task), без README

# 2) привяжи и запушь:
git remote add origin https://github.com/<твой-логин>/carm-task.git
git branch -M main
git push -u origin main

# 3) включи Pages:
#    Repo → Settings → Pages → Source: "Deploy from a branch"
#    Branch: main, папка: / (root) → Save
```

Через ~1 минуту сайт будет доступен по адресу:
`https://<твой-логин>.github.io/carm-task/`

Эту ссылку отдаёшь техническому департаменту на тест.

---

## Заметки

- Данные общие на всех (один документ `app_state/main`). Модель простая:
  при одновременном редактировании двумя людьми действует «последний сохранил — победил».
- Локальный режим (без Supabase) полностью рабочий — удобно для разработки.
- Запуск локально: открыть `index.html` в браузере, либо `python -m http.server`.
