# CI/CD Guide для Saleor SvelteKit Storefront

Этот проект настроен с современным CI/CD pipeline, включающим автоматизированное тестирование, проверки безопасности и сборку.

## 📋 Обзор Pipeline

### 🔄 Основной Workflow (.github/workflows/main.yml)

Запускается при:

- Push в ветки `main` и `develop`
- Pull Request в ветки `main` и `develop`
- Ручной запуск через `workflow_dispatch`

#### Этапы:

1. **Lint and Type Check** 📝
   - Установка зависимостей с PNPM
   - Генерация GraphQL типов
   - Проверка типов TypeScript

2. **Build Application** 🔨
   - Сборка SvelteKit приложения
   - Сохранение артефактов сборки
   - Проверка корректности production build

3. **E2E Tests** 🧪
   - Playwright тесты с Chromium
   - Использование собранного приложения
   - Сохранение отчетов о тестах

4. **Docker Build Test** 🐳
   - Тест сборки Docker образа (только для main ветки)
   - Кеширование слоев для оптимизации

### 🛡️ Security Workflow (.github/workflows/security.yml)

Запускается при:

- Push/PR в `main`/`develop`
- Еженедельно по расписанию

#### Включает:

- **Security Audit**: Проверка уязвимостей зависимостей
- **Dependency Review**: Анализ новых зависимостей в PR
- **CodeQL Analysis**: Статический анализ безопасности кода

## 🧪 Тестирование

### Локальное выполнение тестов

```bash
# Установка зависимостей и браузеров
pnpm install
pnpm exec playwright install chromium --with-deps

# Запуск всех тестов
pnpm test

# Запуск с UI режимом для отладки
pnpm exec playwright test --ui

# Запуск определенного теста
pnpm exec playwright test STF_01.spec.ts
```

### Структура тестов

- `__tests__/STF_*.spec.ts` - Основные функциональные тесты
- `__tests__/lazy-loading.spec.ts` - Тесты ленивой загрузки
- `__tests__/utils.ts` - Утилиты для тестов

### Test Coverage

Тесты покрывают:

- ✅ Добавление товаров в корзину
- ✅ Удаление товаров из корзины
- ✅ Расчет цен и итоговых сумм
- ✅ Оформление заказа
- ✅ Ленивая загрузка продуктов
- ✅ Навигация и UI взаимодействия

## 🐳 Docker интеграция

### Доступные конфигурации

1. **Development** (порт 3000)

   ```bash
   ./dev.sh docker development
   ```

2. **Production** (порт 3001)

   ```bash
   ./dev.sh docker production
   ```

3. **Test** (порт 3002)
   ```bash
   ./dev.sh docker test
   ```

### Docker файлы

- `Dockerfile` - Production образ с Node.js адаптером
- `Dockerfile.test` - Специализированный образ для тестирования
- `docker-compose.*.yml` - Конфигурации для разных сред

## 🔧 Настройка для вашего проекта

### 1. Environment Variables

Скопируйте и настройте:

```bash
cp .env.example .env
```

Основные переменные:

- `PUBLIC_SALEOR_API_URL` - URL вашего Saleor API
- `PUBLIC_STOREFRONT_URL` - URL витрины
- `PORT` - Порт для разработки

### 2. GitHub Actions Secrets

Настройте в GitHub Settings > Secrets:

- Добавьте необходимые API ключи
- Настройте переменные для deployment

### 3. Saleor API Configuration

Убедитесь что ваш Saleor API:

- Доступен публично или имеет правильные CORS настройки
- Поддерживает GraphQL introspection (для разработки)
- Имеет правильную схему данных

## 📊 Мониторинг и отчеты

### Playwright Reports

- Автоматически генерируются HTML отчеты
- Доступны как артефакты в GitHub Actions
- Включают скриншоты при сбоях
- Трейсы выполнения для отладки

### Проверить последний отчет:

```bash
pnpm exec playwright show-report
```

## ⚡ Оптимизация производительности

### CI/CD улучшения

1. **Параллельные джобы** - Lint, Build и Test выполняются параллельно где возможно
2. **Кеширование** - PNPM cache, Playwright browsers, Docker layers
3. **Условное выполнение** - Docker build только для main ветки
4. **Быстрые фейлы** - Остановка при первой ошибке в критических джобах

### Local Development

- Используйте `pnpm run dev` для hot reload
- Playwright UI mode для интерактивной отладки тестов
- Docker для изоляции окружений

## 🔍 Troubleshooting

### Распространенные проблемы

#### GraphQL типы не генерируются

```bash
pnpm run generate
# или полная перегенерация
rm -rf src/gql && pnpm run generate
```

#### Тесты падают с timeout

- Увеличьте timeout в `playwright.config.ts`
- Проверьте доступность Saleor API
- Убедитесь что порт не занят

#### Docker сборка не работает

- Проверьте доступность Docker daemon
- Убедитесь в правильности environment variables
- Очистите Docker cache: `docker system prune`

#### CI/CD не запускается

- Проверьте syntax YAML файлов
- Убедитесь в настройке GitHub Actions permissions
- Проверьте наличие secrets в репозитории

### Получение логов

```bash
# CI/CD логи
git log --oneline -10

# Docker логи
docker logs saleor-storefront-test

# Playwright отчеты
pnpm exec playwright show-report
```

## 🚀 Deployment

Проект готов к deploy на:

- **Vercel** (рекомендуется для SvelteKit)
- **Docker** контейнеры (Kubernetes, Docker Swarm)
- **Node.js** хостинг (адаптер включен)

См. [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) для подробных инструкций.

---

**✨ Готов к продакшену!** Все тесты проходят, CI/CD настроен, безопасность проверена.
