# Полное руководство: удаление Java 17 и установка Java 21 на Ubuntu

## 📋 Содержание
1. [Проверка текущей версии Java](#1-проверка-текущей-версии-java)
2. [Удаление Java 17](#2-удаление-java-17)
3. [Очистка системы](#3-очистка-системы)
4. [Установка Java 21](#4-установка-java-21)
5. [Настройка переменных окружения](#5-настройка-переменных-окружения)
6. [Проверка и верификация](#6-проверка-и-верификация)
7. [Устранение проблем](#7-устранение-проблем)

---

## 1. Проверка текущей версии Java

Прежде чем удалять, убедимся, какая версия Java установлена:

```bash
# Проверка версии Java
java -version

# Проверка компилятора Java
javac -version

# Где установлена Java
which java

# Все установленные пакеты Java
dpkg --list | grep -i java
```

**Пример вывода (если установлена Java 17):**
```
openjdk version "17.0.10" 2026-01-20
OpenJDK Runtime Environment (build 17.0.10+7-Ubuntu-124.04)
OpenJDK 64-Bit Server VM (build 17.0.10+7-Ubuntu-124.04, mixed mode, sharing)
```

---

## 2. Удаление Java 17

### Способ 1: Удаление через apt (рекомендуемый)

```bash
# Находим точное имя пакета Java 17
dpkg --list | grep openjdk-17

# Удаляем все пакеты Java 17
sudo apt remove --purge openjdk-17-jdk openjdk-17-jre openjdk-17-jre-headless

# Если есть дополнительные пакеты
sudo apt remove --purge openjdk-17-*
```

### Способ 2: Удаление конкретных пакетов (более точный)

```bash
# Удаляем JDK 17
sudo apt remove --purge openjdk-17-jdk

# Удаляем JRE 17
sudo apt remove --purge openjdk-17-jre

# Удаляем JRE headless
sudo apt remove --purge openjdk-17-jre-headless

# Удаляем документацию и исходники (если есть)
sudo apt remove --purge openjdk-17-doc openjdk-17-source
```

### Способ 3: Если Java установлена из другого источника

```bash
# Если Java установлена через SDKMAN
sdk uninstall java 17.0.10-open

# Если установлена вручную из tar.gz
# Удаляем директорию Java 17
sudo rm -rf /usr/lib/jvm/java-17-openjdk-amd64
```

---

## 3. Очистка системы

После удаления необходимо очистить систему от остаточных файлов:

```bash
# Автоматическое удаление ненужных пакетов
sudo apt autoremove --purge

# Очистка кэша apt
sudo apt autoclean
sudo apt clean

# Удаление конфигурационных файлов (если есть)
sudo apt purge $(dpkg -l | grep '^rc' | awk '{print $2}')

# Удаление JAVA_HOME из переменных окружения
sudo sed -i '/JAVA_HOME/d' /etc/environment
sudo sed -i '/JAVA_HOME/d' /etc/profile.d/java.sh 2>/dev/null

# Удаление альтернатив Java
sudo update-alternatives --remove-all java
sudo update-alternatives --remove-all javac
sudo update-alternatives --remove-all javadoc
sudo update-alternatives --remove-all javap
```

### Проверка, что Java 17 полностью удалена:

```bash
# Должно вернуть "command not found"
java -version

# Проверка остаточных файлов
find /usr/lib/jvm -name "*17*" 2>/dev/null
find /usr/bin -name "*java*" | grep -v "java-21" 2>/dev/null

# Проверка пакетов (должно быть пусто)
dpkg --list | grep -i "openjdk-17"
```

---

## 4. Установка Java 21

### Способ 1: Установка из официального репозитория Ubuntu (рекомендуемый)

```bash
# Обновление списка пакетов
sudo apt update

# Установка Java 21 JDK (полная версия)
sudo apt install -y openjdk-21-jdk

# Или только JRE (если не нужна разработка)
# sudo apt install -y openjdk-21-jre
```

### Способ 2: Установка с дополнительными компонентами

```bash
# Установка полного набора для разработки
sudo apt install -y openjdk-21-jdk openjdk-21-doc openjdk-21-source

# Установка инструментов разработки
sudo apt install -y openjdk-21-demo openjdk-21-dbg
```

### Способ 3: Установка через SDKMAN (альтернативный, для разработчиков)

```bash
# Если SDKMAN не установлен
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# Установка Java 21
sdk install java 21.0.5-tem

# Установка как версии по умолчанию
sdk default java 21.0.5-tem
```

### Способ 4: Ручная установка из официального архива (для полного контроля)

```bash
# Скачивание Java 21 (например, от Eclipse Temurin)
cd /tmp
wget https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21.0.5%2B11/OpenJDK21U-jdk_x64_linux_hotspot_21.0.5_11.tar.gz

# Распаковка
sudo tar -xzf OpenJDK21U-jdk_x64_linux_hotspot_21.0.5_11.tar.gz -C /usr/lib/jvm/

# Переименование для удобства
sudo mv /usr/lib/jvm/jdk-21.0.5+11 /usr/lib/jvm/java-21-openjdk-amd64

# Настройка альтернатив
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-21-openjdk-amd64/bin/java 1
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-21-openjdk-amd64/bin/javac 1
```

---

## 5. Настройка переменных окружения

### Настройка JAVA_HOME для Java 21:

```bash
# Находим путь к Java 21
JAVA21_PATH=$(readlink -f $(which java) | sed 's:/bin/java::')
echo "Java 21 находится по пути: $JAVA21_PATH"

# Добавляем JAVA_HOME в /etc/environment (для всех пользователей)
echo "JAVA_HOME=$JAVA21_PATH" | sudo tee -a /etc/environment

# Создаём отдельный скрипт для профиля (рекомендуемый способ)
sudo bash -c "cat > /etc/profile.d/java.sh << EOF
export JAVA_HOME=$JAVA21_PATH
export PATH=\$JAVA_HOME/bin:\$PATH
EOF"

# Делаем скрипт исполняемым
sudo chmod +x /etc/profile.d/java.sh

# Применяем изменения для текущей сессии
source /etc/profile.d/java.sh
source /etc/environment
```

### Настройка для конкретного пользователя (если нужна своя версия):

```bash
# Добавляем в ~/.bashrc
echo "export JAVA_HOME=$JAVA21_PATH" >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc

# Применяем
source ~/.bashrc
```

### Настройка Maven для работы с Java 21:

Если у вас установлен Maven, обновите его конфигурацию:

```bash
# Для глобальной установки Maven
sudo nano /etc/maven/mavenrc

# Добавьте строку:
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
```

---

## 6. Проверка и верификация

### Базовая проверка:

```bash
# Проверка версии Java
java -version
```

**Ожидаемый вывод:**
```
openjdk version "21.0.5" 2026-01-20
OpenJDK Runtime Environment (build 21.0.5+11-Ubuntu-124.04)
OpenJDK 64-Bit Server VM (build 21.0.5+11-Ubuntu-124.04, mixed mode, sharing)
```

```bash
# Проверка компилятора
javac -version
# Вывод: javac 21.0.5
```

### Проверка переменных окружения:

```bash
# Проверка JAVA_HOME
echo $JAVA_HOME
# Вывод: /usr/lib/jvm/java-21-openjdk-amd64

# Проверка PATH
echo $PATH | grep java

# Проверка, что используется правильная Java
which java
# Вывод: /usr/bin/java или /usr/lib/jvm/java-21-openjdk-amd64/bin/java
```

### Проверка альтернатив Java:

```bash
# Просмотр всех установленных версий Java
sudo update-alternatives --config java

# Если Java 21 не выбрана автоматически, выберите её
# Введите номер строки с Java 21 и нажмите Enter
```

### Тестовый запуск Java-программы:

```bash
# Создаём тестовую программу
cat > TestJava.java << 'EOF'
public class TestJava {
    public static void main(String[] args) {
        System.out.println("Java version: " + System.getProperty("java.version"));
        System.out.println("Java vendor: " + System.getProperty("java.vendor"));
        System.out.println("OS: " + System.getProperty("os.name"));
    }
}
EOF

# Компилируем
javac TestJava.java

# Запускаем
java TestJava
```

**Ожидаемый вывод:**
```
Java version: 21.0.5
Java vendor: Ubuntu
OS: Linux
```

### Проверка для Jenkins:

```bash
# Останавливаем Jenkins (если запущен)
sudo systemctl stop jenkins

# Проверяем, какую Java видит Jenkins
sudo -u jenkins java -version

# Запускаем Jenkins снова
sudo systemctl start jenkins

# Проверяем статус
sudo systemctl status jenkins
```

---

## 7. Устранение проблем

### Проблема 1: Java 17 не удаляется полностью

```bash
# Принудительное удаление всех пакетов с Java 17
sudo dpkg --remove --force-remove-reinstreq $(dpkg -l | grep openjdk-17 | awk '{print $2}')
sudo apt-get clean
sudo apt-get autoclean
sudo apt-get --purge autoremove
```

### Проблема 2: После установки Java 21 всё ещё видна Java 17

```bash
# Проверка символических ссылок
ls -la /usr/bin/java

# Принудительное обновление альтернатив
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-21-openjdk-amd64/bin/java 1
sudo update-alternatives --set java /usr/lib/jvm/java-21-openjdk-amd64/bin/java

# Перезагрузка терминала
exec $SHELL
```

### Проблема 3: Jenkins не видит Java 21

```bash
# Проверка конфигурации Jenkins
sudo nano /etc/default/jenkins

# Найдите строку JAVA_HOME и измените её:
JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64

# Или добавьте, если строки нет

# Перезапуск Jenkins
sudo systemctl daemon-reload
sudo systemctl restart jenkins

# Проверка логов
sudo journalctl -u jenkins -n 50
```

### Проблема 4: Maven не видит Java 21

```bash
# Обновление Maven конфигурации
sudo nano /etc/maven/mavenrc

# Добавьте:
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64

# Проверка
mvn -version
```

### Проблема 5: Ошибка "Unable to locate package openjdk-21-jdk"

```bash
# Обновление списка пакетов
sudo apt update

# Проверка доступных версий
apt-cache search openjdk | grep 21

# Если Ubuntu версии ниже 22.04, Java 21 может отсутствовать
# Проверка версии Ubuntu
lsb_release -a

# Для старых версий Ubuntu добавьте PPA
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt update
sudo apt install openjdk-21-jdk
```

### Проблема 6: Конфликт с другими версиями Java

```bash
# Полный сброс всех альтернатив Java
sudo update-alternatives --remove-all java
sudo update-alternatives --remove-all javac

# Чистая переустановка Java 21
sudo apt install --reinstall openjdk-21-jdk

# Настройка заново
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-21-openjdk-amd64/bin/java 1
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-21-openjdk-amd64/bin/javac 1
```

---

## 📊 Финальный чек-лист

После выполнения всех шагов проверьте:

| ✅ | Проверка | Команда | Ожидаемый результат |
|----|----------|---------|---------------------|
| ☐ | Java 17 удалена | `dpkg -l \| grep openjdk-17` | Пусто |
| ☐ | Java 21 установлена | `java -version` | 21.x.x |
| ☐ | JAVA_HOME настроен | `echo $JAVA_HOME` | /usr/lib/jvm/java-21-* |
| ☐ | Компилятор работает | `javac -version` | javac 21.x.x |
| ☐ | Альтернативы настроены | `update-alternatives --list java` | Показывает Java 21 |
| ☐ | Jenkins работает | `sudo systemctl status jenkins` | active (running) |
| ☐ | Maven работает | `mvn -version` | Java version: 21.x.x |

---

## 🎯 Заключение

Вы успешно удалили Java 17 и установили Java 21 на ваш Ubuntu сервер. Теперь:

- **Jenkins** будет использовать Java 21 (обязательно для новых версий)
- **Maven** будет компилировать проекты с Java 21
- Все переменные окружения настроены корректно

Если вы используете Jenkins, не забудьте также обновить настройки Java в Jenkins UI:
**Manage Jenkins** → **Tools** → **JDK** → обновите путь на `/usr/lib/jvm/java-21-openjdk-amd64`

При возникновении любых проблем — проверьте логи или обратитесь за помощью!