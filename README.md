# Практическая работа №8  
## Настройка CI/CD-пайплайна для Go-сервиса с использованием Docker  

**Выполнил:** студент группы ИВБО-01-25 Сотников М.Е.  

---

## Цель работы  
Освоить базовые принципы непрерывной интеграции (CI) на примере GitHub Actions:  
- автоматический запуск тестов и сборки Go-приложения при push/pull request;  
- контейнеризация сервиса (Docker) в рамках пайплайна;  
- разделение этапов тестирования и сборки Docker-образа.  

---

## Используемые технологии  
- Go 1.23  
- Docker, Docker Compose  
- GitHub Actions  
 
---

## Описание CI/CD пайплайна  

Файл `.github/workflows/ci.yaml` содержит два задания (`jobs`):

1. **test-and-build** – тестирование и сборка Go-кода:
   - установка Go 1.23;
   - загрузка зависимостей (`go mod tidy`);
   - запуск тестов (`go test ./...`);
   - проверка компиляции (`go build ./...`).

2. **docker-build** – сборка Docker-образа (запускается только после успешного выполнения `test-and-build`):
   - настройка Docker Buildx;
   - сборка образа с тегом по SHA коммита: `techip-tasks:${{ github.sha }}`.

Пайплайн запускается автоматически при push в ветки `main`/`master` и при создании pull request.

Ниже приведён полный листинг `ci.yaml`:

```yaml
name: CI Pipeline

on:
  push:
    branches: [ "main", "master" ]
  pull_request:
    branches: [ "main", "master" ]

jobs:
  test-and-build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.23'

      - name: Show Go version
        run: go version

      - name: Download dependencies
        run: go mod tidy
        working-directory: ./services/tasks

      - name: Run tests
        run: go test ./...
        working-directory: ./services/tasks

      - name: Build application
        run: go build ./...
        working-directory: ./services/tasks

  docker-build:
    runs-on: ubuntu-latest
    needs: test-and-build
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build Docker image
        run: docker build -t techip-tasks:${{ github.sha }} .
        working-directory: ./services/tasks
```

<img width="1318" height="256" alt="actions" src="https://github.com/user-attachments/assets/69575c46-a168-4ae1-afa0-2a00adf02aa5" />

Пайплайн успешно отработал после пуша в репозиторий gh_actions.

Job test-and-build завершился за ~22 секунды, все тесты пройдены, сборка успешна.

Job docker-build выполнился без ошибок, Docker-образ собран с тегом, соответствующим хешу коммита.

<img width="1532" height="376" alt="pipe" src="https://github.com/user-attachments/assets/dca60004-67a4-4200-8c71-7a1045d1d4fe" />

<img width="1509" height="503" alt="image" src="https://github.com/user-attachments/assets/28304743-cf54-4cbf-a62c-d0510f95e22b" />

Автоматизация проверок исключает отправку неработающего кода в основную ветку.

Многостадийная сборка Docker-образа в CI гарантирует, что образ будет создан только после прохождения тестов.

Тегирование образа хешем коммита позволяет однозначно связать артефакт с исходным кодом.

Разделение заданий через needs делает пайплайн прозрачным и упрощает диагностику ошибок.

Настроенный CI готов к расширению: добавление публикации образа в регистр или деплоя на сервер.

