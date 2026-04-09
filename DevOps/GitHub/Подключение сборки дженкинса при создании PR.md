# Установка mvn внутрь контейнера с Jenkins

### 1. Зайти в контейнер
```bash
docker exec -it -u root <имя контейнера> /bin/bash
```

### 2. Установка Maven внутри Jenkins контейнера:

* Если на основе Debian/Ubuntu
```bash
apt-get update
apt-get install -y maven
```

* Для Alpine версии Jenkins (мне как раз помогло)

```bash
apk add maven
```

* Проверка
```bash
mvn --version
```
