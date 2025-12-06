# libman - Library Managing Web

Ứng dụng web Java cơ bản để quản lý thư viện.

## Cấu trúc Project

```
libman/
├── pom.xml                          # File cấu hình Maven
├── src/
│   └── main/
│       ├── java/
│       │   └── com/libman/
│       │       └── servlet/
│       │           └── HomeServlet.java    # Servlet cơ bản
│       └── webapp/
│           ├── WEB-INF/
│           │   └── web.xml                 # Cấu hình web application
│           ├── index.html                  # Trang HTML chính
│           └── index.jsp                   # Trang JSP chính
└── README.md
```

## Yêu cầu

- Java 25 (JDK 25)
- Maven 3.6+ 
- **Tomcat 11** (hỗ trợ Jakarta EE 10)

## Cài đặt và Chạy

### 1. Build project

```bash
mvn clean package
```

File WAR sẽ được tạo trong thư mục `target/libman.war`

### 2. Chạy với Maven Tomcat Plugin

```bash
mvn tomcat7:run
```

Ứng dụng sẽ chạy tại: `http://localhost:8080/libman`

### 3. Deploy lên Tomcat 11

#### Bước 1: Download và cài đặt Tomcat 11

1. Truy cập: https://tomcat.apache.org/download-11.cgi
2. Download file **zip** (Windows) hoặc **tar.gz** (Linux/Mac) của Tomcat 11
3. Giải nén vào thư mục bạn muốn, ví dụ: `C:\apache-tomcat-11.0.0` hoặc `/opt/apache-tomcat-11.0.0`

#### Bước 2: Cấu hình biến môi trường (Tùy chọn)

**Windows:**
```cmd
set CATALINA_HOME=C:\apache-tomcat-11.0.0
set JAVA_HOME=C:\Program Files\Java\jdk-25
```

**Linux/Mac:**
```bash
export CATALINA_HOME=/opt/apache-tomcat-11.0.0
export JAVA_HOME=/usr/lib/jvm/jdk-25
```

#### Bước 3: Build project và tạo file WAR

```bash
mvn clean package
```

File `libman.war` sẽ được tạo trong thư mục `target/`

#### Bước 4: Copy file WAR vào Tomcat

**Windows:**
```cmd
copy target\libman.war C:\apache-tomcat-11.0.0\webapps\
```

**Linux/Mac:**
```bash
cp target/libman.war /opt/apache-tomcat-11.0.0/webapps/
```

**Hoặc:** Copy file `target/libman.war` vào thư mục `webapps` của Tomcat bằng File Explorer/Finder

#### Bước 5: Khởi động Tomcat

**Windows:**
```cmd
cd C:\apache-tomcat-11.0.0\bin
startup.bat
```

**Linux/Mac:**
```bash
cd /opt/apache-tomcat-11.0.0/bin
./startup.sh
```

**Hoặc:** Chạy file `startup.bat` (Windows) hoặc `startup.sh` (Linux/Mac) trong thư mục `bin` của Tomcat

#### Bước 6: Kiểm tra và truy cập ứng dụng

1. Đợi vài giây để Tomcat khởi động và deploy ứng dụng
2. Mở trình duyệt và truy cập:
   - Trang chủ: `http://localhost:8080/libman/`
   - Trang HTML: `http://localhost:8080/libman/index.html`
   - Trang JSP: `http://localhost:8080/libman/index.jsp`
   - Servlet: `http://localhost:8080/libman/home`

3. Kiểm tra log trong thư mục `logs/` nếu có lỗi:
   - `catalina.out` (Linux/Mac)
   - `catalina.log` (Windows)

#### Bước 7: Dừng Tomcat

**Windows:**
```cmd
cd C:\apache-tomcat-11.0.0\bin
shutdown.bat
```

**Linux/Mac:**
```bash
cd /opt/apache-tomcat-11.0.0/bin
./shutdown.sh
```

#### Lưu ý quan trọng:

- **Port 8080**: Nếu port 8080 đã được sử dụng, bạn có thể thay đổi trong file `conf/server.xml` (tìm `port="8080"`)
- **Tự động deploy**: Tomcat sẽ tự động deploy file WAR khi khởi động. Nếu bạn thay đổi code, chỉ cần:
  1. Build lại: `mvn clean package`
  2. Xóa thư mục `webapps/libman` (nếu có)
  3. Copy lại file `libman.war` vào `webapps/`
  4. Tomcat sẽ tự động deploy lại
- **Hot deploy**: Để hot deploy, bạn có thể copy file WAR trực tiếp vào `webapps/` khi Tomcat đang chạy, Tomcat sẽ tự động phát hiện và deploy lại
- **Kiểm tra log**: Nếu có lỗi, kiểm tra file log trong thư mục `logs/` của Tomcat

## Các trang có sẵn

- `/` hoặc `/index.html` - Trang HTML chính
- `/index.jsp` - Trang JSP với thông tin server
- `/home` - Servlet HomeServlet

## Công nghệ sử dụng

- **Jakarta EE 10** - Enterprise Java platform
- **Jakarta Servlet API 6.0** - Xử lý HTTP requests
- **Jakarta JSP 3.1** - JavaServer Pages cho dynamic content
- **Jakarta JSTL 3.0** - Jakarta Standard Tag Library
- **Tomcat 11** - Servlet container
- **Maven** - Quản lý dependencies và build

## Phát triển tiếp

Bạn có thể mở rộng project bằng cách:
- Thêm các servlet mới trong package `com.libman.servlet`
- Tạo các JSP pages trong `src/main/webapp`
- Thêm các dependencies mới vào `pom.xml`
- Tạo các class model trong package `com.libman.model`
- Tạo các class DAO trong package `com.libman.dao`
