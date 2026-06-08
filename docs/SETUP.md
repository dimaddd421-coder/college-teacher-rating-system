# 🔧 Руководство по настройке Supabase

## Шаг 1: Создание проекта Supabase

1. Перейдите на [supabase.com](https://supabase.com/)
2. Нажмите "Start your project"
3. Войдите или создайте аккаунт
4. Создайте новый проект:
   - Organization: выберите или создайте
   - Project name: `college-teacher-rating`
   - Database password: придумайте безопасный пароль
   - Region: выберите ближайший регион

## Шаг 2: Копирование ключей доступа

1. После создания проекта откройте Settings (шестеренка)
2. Перейдите в API
3. Скопируйте:
   - Project URL (это `SUPABASE_URL`)
   - anon public (это `SUPABASE_ANON_KEY`)

## Шаг 3: Обновление конфигурации

Откройте `config.js` и обновите:

```javascript
const SUPABASE_URL = 'ваша_URL_здесь';
const SUPABASE_ANON_KEY = 'ваш_ключ_здесь';
```

## Шаг 4: Создание таблиц БД

1. В консоли Supabase откройте SQL Editor
2. Выполните все запросы из `docs/database.sql`

Или скопируйте этот код:

```sql
-- Таблица пользователей
CREATE TABLE users (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица преподавателей
CREATE TABLE teachers (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  department TEXT NOT NULL,
  bio TEXT,
  rating FLOAT DEFAULT 0,
  review_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица оценок
CREATE TABLE reviews (
  id SERIAL PRIMARY KEY,
  teacher_id INT REFERENCES teachers(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  rating INT NOT NULL CHECK (rating >= 1 AND rating <= 5),
  comment TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP,
  UNIQUE(teacher_id, user_id)
);

-- Таблица избранных
CREATE TABLE favorites (
  id SERIAL PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  teacher_id INT REFERENCES teachers(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, teacher_id)
);

-- Таблица администраторов
CREATE TABLE admin_users (
  id SERIAL PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Создание индексов для оптимизации
CREATE INDEX idx_reviews_teacher ON reviews(teacher_id);
CREATE INDEX idx_reviews_user ON reviews(user_id);
CREATE INDEX idx_favorites_user ON favorites(user_id);
CREATE INDEX idx_teachers_rating ON teachers(rating DESC);
```

## Шаг 5: Настройка RLS политик (опционально)

Для дополнительной безопасности включите Row Level Security:

```sql
-- Включить RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE teachers ENABLE ROW LEVEL SECURITY;
ALTER TABLE reviews ENABLE ROW LEVEL SECURITY;
ALTER TABLE favorites ENABLE ROW LEVEL SECURITY;

-- Политики для таблицы users
CREATE POLICY "Users can read all users" ON users FOR SELECT USING (true);
CREATE POLICY "Users can update own profile" ON users FOR UPDATE 
  USING (auth.uid() = id);

-- Политики для таблицы teachers
CREATE POLICY "Anyone can read teachers" ON teachers FOR SELECT USING (true);
CREATE POLICY "Only admins can insert teachers" ON teachers FOR INSERT 
  WITH CHECK (EXISTS (SELECT 1 FROM admin_users WHERE user_id = auth.uid()));

-- Политики для таблицы reviews
CREATE POLICY "Anyone can read reviews" ON reviews FOR SELECT USING (true);
CREATE POLICY "Users can insert own reviews" ON reviews FOR INSERT 
  WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Users can update own reviews" ON reviews FOR UPDATE 
  USING (auth.uid() = user_id);
CREATE POLICY "Users can delete own reviews" ON reviews FOR DELETE 
  USING (auth.uid() = user_id);

-- Политики для таблицы favorites
CREATE POLICY "Users can manage own favorites" ON favorites FOR ALL 
  USING (auth.uid() = user_id);
```

## Шаг 6: Добавление первого администратора

1. Зарегистрируйтесь в приложении
2. В консоли Supabase откройте таблицу `users` и скопируйте ваш `id`
3. В таблице `admin_users` добавьте новую запись с вашим `user_id`

SQL запрос:
```sql
INSERT INTO admin_users (user_id) VALUES ('ваш_uuid_здесь');
```

## Шаг 7: Настройка Authentication

1. В консоли Supabase откройте Authentication > Providers
2. Убедитесь, что Email включен
3. (Опционально) включите OAuth провайдеры (Google, GitHub и т.д.)

## Готово! ✅

Ваше приложение готово к использованию:

1. Откройте `index.html` в браузере
2. Зарегистрируйтесь
3. Начните оценивать преподавателей

## Решение проблем

### Ошибка: "Supabase not defined"
- Убедитесь, что в `index.html` подключена библиотека Supabase
- Проверьте консоль браузера на ошибки

### Ошибка: "Invalid API key"
- Проверьте, что ключи в `config.js` скопированы правильно
- Убедитесь, что используете `anon public` ключ

### Таблицы не создаются
- Убедитесь, что вы в правильном проекте
- Проверьте SQL синтаксис
- Посмотрите логи ошибок в консоли

## Дополнительные ресурсы

- [Документация Supabase](https://supabase.com/docs)
- [Supabase Auth](https://supabase.com/docs/guides/auth)
- [Supabase JavaScript](https://supabase.com/docs/reference/javascript)
