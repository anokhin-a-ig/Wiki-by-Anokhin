## Установка Java (JDK) и Maven на Ubuntu

### 📦 Быстрая установка (рекомендуемый способ)

Самый простой и надёжный способ — использовать штатный менеджер пакетов APT:

### 1. Обновление списка пакетов
```bash
sudo apt update
````
### 2. Установка JDK (выберите нужную версию)
#### Вариант А: Java 17 (LTS — рекомендуется для новых проектов)
```bash
sudo apt install openjdk-17-jdk -y
```

#### Вариант Б: Java 11 (LTS — всё ещё очень популярна)
#### sudo apt install openjdk-11-jdk -y

### 3. Установка Maven
```bash
sudo apt install maven -y
```


### ✅ Проверка установки

После установки выполните команды для проверки:

# Проверка Java
```bash
java -version
```
```bash
javac -version
```
# Проверка Maven
```bash
mvn -version
```

Если всё установлено правильно, вы увидите информацию о версиях .

---

### ⚙️ Настройка переменных окружения

Maven и многие другие инструменты требуют корректно настроенную переменную `JAVA_HOME`.

**Шаг 1: Найдите путь к установленной Java**
# Узнаем фактический путь к JDK
```bash
readlink -f $(which java) | sed 's:/bin/java::'
```

Обычно путь выглядит так:
- Для Java 17: `/usr/lib/jvm/java-17-openjdk-amd64`
- Для Java 11: `/usr/lib/jvm/java-11-openjdk-amd64`

**Шаг 2: Настройте переменные (рекомендуемый способ для всех пользователей)**

Создайте файл в каталоге `/etc/profile.d/`:
```bash
sudo nano /etc/profile.d/java.sh
```

Добавьте в него следующие строки (подставьте ваш реальный путь):
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export M2_HOME=/usr/share/maven
export MAVEN_HOME=/usr/share/maven
```

Сохраните файл и сделайте его исполняемым:
```bash
sudo chmod +x /etc/profile.d/java.sh
```

**Шаг 3: Примените изменения**
```bash
source /etc/profile.d/java.sh
```

**Шаг 4: Проверьте переменную**
```bash
echo $JAVA_HOME
```

Должен отобразиться правильный путь к JDK .

---

### 🚀 Создание первого Maven-проекта (проверка работы)

```bash
# Создание проекта
mvn archetype:generate -DgroupId=com.example \
    -DartifactId=my-app \
    -DarchetypeArtifactId=maven-archetype-quickstart \
    -DinteractiveMode=false

# Переход в папку проекта
cd my-app

# Сборка проекта
mvn package

# Запуск программы
java -cp target/my-app-1.0-SNAPSHOT.jar com.example.App
```

Если вы видите `Hello World!` — всё работает отлично .

---

### 🔄 Управление несколькими версиями Java

Если вам нужно переключаться между разными версиями Java:

```bash
# Посмотреть доступные версии
sudo update-alternatives --config java

# Выберите номер нужной версии и нажмите Enter
```

Или для более гибкого управления используйте **SDKMAN!**:
```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java 17.0.9-tem
sdk install java 11.0.21-tem
sdk use java 17.0.9-tem
```

---

### 📋 Краткий чек-лист

- [x] Установлен JDK (`openjdk-17-jdk` или `openjdk-11-jdk`)
- [x] Установлен Maven (`maven`)
- [x] Настроена переменная `JAVA_HOME` в `/etc/profile.d/java.sh`
- [x] Проверено: `java -version`, `javac -version`, `mvn -version`
- [x] Создан и собран тестовый Maven-проект

Этих настроек достаточно для начала разработки Java-приложений на вашем сервере Ubuntu. Если понадобится настроить Gradle или установить конкретную версию Maven вручную (не из репозитория), дайте знать!