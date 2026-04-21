Подключение к базе данных H2 зависит от того, в каком режиме она используется (встроенный, серверный, в памяти) и через какой инструмент/код.

Ниже основные способы.

## 1. Подключение через URL JDBC

Общий формат:
```
jdbc:h2:{режим}:{путь/параметры}
```

### Режимы подключения

| Режим | URL пример | Описание |
|-------|-----------|----------|
| **В памяти** | `jdbc:h2:mem:test` | Существует только пока приложение запущено |
| **Встроенный (файл)** | `jdbc:h2:~/db/mydb` | Файл БД в домашней директории |
| **Встроенный (относительный путь)** | `jdbc:h2:./data/myapp` | Папка относительно запуска |
| **TCP сервер** | `jdbc:h2:tcp://localhost/~/test` | Подключение к серверному режиму |
| **TCP с файлом** | `jdbc:h2:tcp://localhost/./data/db` | Серверный доступ к относительному пути |

### Параметры в URL
```
jdbc:h2:~/test;MODE=MySQL;DB_CLOSE_DELAY=-1;AUTO_SERVER=TRUE
```

- `MODE=MySQL` – совместимость с MySQL
- `DB_CLOSE_DELAY=-1` – не закрывать БД при последнем подключении
- `AUTO_SERVER=TRUE` – автоматический серверный режим (для многопроцессного доступа к файлу)
- `USER=sa` / `PASSWORD=`

## 2. Подключение из Java кода

```java
import java.sql.*;

public class H2Example {
    public static void main(String[] args) throws SQLException {
        // Загружаем драйвер (необязательно для новых JDBC)
        try {
            Class.forName("org.h2.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        // Подключаемся
        String url = "jdbc:h2:~/test";
        String user = "sa";
        String password = "";

        try (Connection conn = DriverManager.getConnection(url, user, password)) {
            System.out.println("Подключено к H2!");

            // Создаём таблицу
            Statement stmt = conn.createStatement();
            stmt.execute("CREATE TABLE IF NOT EXISTS users (id INT PRIMARY KEY, name VARCHAR(255))");
            
            // Вставляем данные
            stmt.execute("INSERT INTO users VALUES (1, 'Alice')");
            
            // Читаем
            ResultSet rs = stmt.executeQuery("SELECT * FROM users");
            while (rs.next()) {
                System.out.println(rs.getInt("id") + " " + rs.getString("name"));
            }
        }
    }
}
```

## 3. Подключение из Spring Boot (application.properties)

```properties
# Встроенная H2 в памяти
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# Консоль H2 (веб-интерфейс)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Настройка для сохранения данных между перезапусками
# spring.datasource.url=jdbc:h2:file:./data/mydb;DB_CLOSE_DELAY=-1
```

## 4. Подключение через H2 Console (веб-интерфейс)

Запустите H2 Console:

```bash
java -jar h2*.jar
```

Обычно доступно по адресу: `http://localhost:8082`

Параметры подключения в веб-интерфейсе:
- **Driver Class**: `org.h2.Driver`
- **JDBC URL**: `jdbc:h2:~/test` (или ваш URL)
- **User Name**: `sa`
- **Password**: (оставить пустым)

## 5. Maven / Gradle зависимость

**Maven:**
```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.224</version>
    <scope>runtime</scope>
</dependency>
```

**Gradle:**
```gradle
runtimeOnly 'com.h2database:h2:2.2.224'
```

## 6. Особые сценарии подключения

### Для одновременного доступа нескольких процессов к файлу
```properties
jdbc:h2:~/mydb;AUTO_SERVER=TRUE
```

### Подключение к удалённой H2 (TCP)
```java
String url = "jdbc:h2:tcp://192.168.1.100:9092/~/remoteDB";
```

### База данных только для чтения
```properties
jdbc:h2:~/mydb;ACCESS_MODE_DATA=r
```

### Шифрование базы данных
```properties
jdbc:h2:~/secure;CIPHER=AES
```
При этом пароль указывается так: `user` / `filePassword userPassword`

---

Если уточните, какой именно сценарий вас интересует (Java, Spring, CLI, консоль, встраиваемая БД), могу дать более точный пример.