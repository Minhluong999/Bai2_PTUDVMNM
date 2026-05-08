# BÀI TẬP 02

# Môn: Phát triển ứng dụng với mã nguồn mở - TEE0421

## Đề tài: Sử dụng Django để tạo web quản lý tiệm cầm đồ

---

# Thông tin sinh viên

* Họ và tên: .Lăng Nguyễn Minh Lượng
* Lớp: 58KTPM
* Mã sinh viên: K225480106044

---

# 1. Mô tả bài toán

Trong bài tập này em sử dụng Django kết hợp Docker để xây dựng hệ thống quản lý tiệm cầm đồ cơ bản.

Hệ thống cho phép:
- Quản lý khách hàng
- Quản lý tài sản cầm cố
- Quản lý phiếu cầm
- Quản lý thanh toán
- Theo dõi các khoản nợ đến hạn chưa thanh toán

Ngoài ra em còn sử dụng Cloudflare Tunnel để public website ra internet.

# 2. Công nghệ sử dụng

| Công nghệ | Mục đích |
|---|---|
| Ubuntu Server | Môi trường chạy project |
| Docker | Chạy container |
| Docker Compose | Quản lý nhiều service |
| Django | Framework backend |
| MariaDB | Hệ quản trị cơ sở dữ liệu |
| phpMyAdmin | Kiểm tra cơ sở dữ liệu |
| Cloudflare Tunnel | Public website ra internet |

# 3. Thiết kế cơ sở dữ liệu

## Các bảng chính

* KhachHang
* TaiSan
* PhieuCam
* ThanhToan

## Quan hệ

* Một khách hàng có nhiều phiếu cầm
* Một tài sản thuộc một phiếu cầm
* Một phiếu cầm có nhiều lần thanh toán

## Ảnh sơ đồ CSDL viết tay

> Chèn ảnh tại đây

---

# 4. Cấu trúc thư mục project

## Lệnh kiểm tra

```bash
tree ~/pawnshop
```

## Kết quả

<img width="1356" height="768" alt="image" src="https://github.com/user-attachments/assets/793341b6-214d-4ace-a806-51e3f6ca14d8" />

---

# 5. Tạo Dockerfile

## Nội dung Dockerfile

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN apt update && apt install -y gcc default-libmysqlclient-dev pkg-config

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

## Giải thích

| Lệnh                  | Ý nghĩa               |
| --------------------- | --------------------- |
| FROM python:3.12      | Sử dụng image Python  |
| WORKDIR /app          | Thư mục làm việc      |
| COPY requirements.txt | Copy file thư viện    |
| RUN apt install       | Cài thư viện hệ thống |
| RUN pip install       | Cài thư viện Python   |
| COPY . .              | Copy source code      |
| CMD                   | Chạy Django           |

---

# 6. Tạo requirements.txt

## Nội dung

```txt
Django
mysqlclient
```

## Giải thích

| Thư viện    | Công dụng       |
| ----------- | --------------- |
| Django      | Framework web   |
| mysqlclient | Kết nối MariaDB |

---

# 7. Tạo docker-compose.yml

## Nội dung

```yaml
services:

  mariadb:
    image: mariadb:11
    container_name: mariadb_pawn
    restart: always

    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: pawnshop_db

    ports:
      - "3307:3306"

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin_pawn
    restart: always

    environment:
      PMA_HOST: mariadb
      MYSQL_ROOT_PASSWORD: 123456

    ports:
      - "8080:80"

  django:
    build: ./django
    container_name: django_pawn
    restart: always

    volumes:
      - ./django/app:/app

    ports:
      - "8000:8000"

    depends_on:
      - mariadb
```

## Giải thích

| Service    | Chức năng          |
| ---------- | ------------------ |
| mariadb    | Chứa cơ sở dữ liệu |
| phpmyadmin | Xem cơ sở dữ liệu  |
| django     | Chạy web Django    |

---

# 8. Khởi động hệ thống Docker

## Lệnh

```bash
sudo docker compose up -d --build
```

## Giải thích

| Thành phần | Ý nghĩa             |
| ---------- | ------------------- |
| up         | Khởi động container |
| -d         | Chạy nền            |
| --build    | Build lại image     |

## Kết quả

<img width="1106" height="634" alt="image" src="https://github.com/user-attachments/assets/86be6f98-9c5a-4574-9a2e-d495167e27cd" />

---

# 9. Kiểm tra container Docker

## Lệnh

```bash
sudo docker ps
```

## Giải thích

Hiển thị danh sách container đang chạy.

## Kết quả

<img width="1107" height="273" alt="image" src="https://github.com/user-attachments/assets/314133b9-ec28-4549-bcca-5aa2f2ff9019" />

---

# 10. Cấu hình Django kết nối MariaDB

## File settings.py

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'pawnshop_db',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': 'mariadb',
        'PORT': '3306',
    }
}
```

## Giải thích

Django sử dụng MariaDB làm hệ quản trị cơ sở dữ liệu.

---

# 11. Tạo app Django

## Lệnh

```bash
python manage.py startapp core
```

## Giải thích

Tạo app tên `core` để quản lý nghiệp vụ.

---

# 12. Khai báo models.py

## Nội dung

```python
from django.db import models

class KhachHang(models.Model):
    ho_ten = models.CharField(max_length=100)
    so_dien_thoai = models.CharField(max_length=20)
    dia_chi = models.TextField()

class TaiSan(models.Model):
    ten_tai_san = models.CharField(max_length=100)
    mo_ta = models.TextField()

class PhieuCam(models.Model):
    khach_hang = models.ForeignKey(KhachHang, on_delete=models.CASCADE)
    tai_san = models.ForeignKey(TaiSan, on_delete=models.CASCADE)
    so_tien_vay = models.IntegerField()
    lai_suat = models.FloatField()
    ngay_cam = models.DateField()
    ngay_den_han = models.DateField()
    da_tra = models.BooleanField(default=False)

class ThanhToan(models.Model):
    phieu_cam = models.ForeignKey(PhieuCam, on_delete=models.CASCADE)
    so_tien = models.IntegerField()
    ngay_thanh_toan = models.DateField()
```

## Giải thích

* ForeignKey dùng để tạo khóa ngoại
* Django sẽ tự sinh bảng trong MariaDB

---

# 13. Tạo migration

## Lệnh

```bash
python manage.py makemigrations
```

## Giải thích

Tạo file migration từ models.py.

## Kết quả

<img width="1095" height="210" alt="image" src="https://github.com/user-attachments/assets/b7c75669-f93d-488f-a1ba-b96010fa7f2c" />

---

# 14. Tạo bảng trong MariaDB

## Lệnh

```bash
python manage.py migrate
```

## Giải thích

Django tạo các bảng trong MariaDB.

## Kết quả

<img width="762" height="113" alt="image" src="https://github.com/user-attachments/assets/3b9a25e8-ee34-428b-9158-d3df33a2203c" />


---

# 15. Tạo tài khoản admin

## Lệnh

```bash
python manage.py createsuperuser
```

## Giải thích

Tạo tài khoản quản trị Django.

## Kết quả

<img width="574" height="43" alt="image" src="https://github.com/user-attachments/assets/22cc6179-62c7-41e1-a045-a0221da941f5" />

---

# 16. Đăng ký models vào Django Admin

## File admin.py

```python
from django.contrib import admin

from .models import *

admin.site.register(KhachHang)
admin.site.register(TaiSan)
admin.site.register(PhieuCam)
admin.site.register(ThanhToan)
```

## Giải thích

Cho phép quản lý dữ liệu trên trang admin.

---

# 17. Giao diện Django Admin

## Link

```text
/admin
```

## Kết quả

<img width="1360" height="755" alt="image" src="https://github.com/user-attachments/assets/f59c147f-ab7a-4f4d-9ebf-1d02fe914b52" />

---

# 18. Kiểm tra khóa ngoại bằng phpMyAdmin

## Link

```text
http://http://192.168.1.16:8080
```

## Giải thích

Kiểm tra:

* khach_hang_id
* tai_san_id

được lưu bằng ID trong MariaDB.

## Kết quả

<img width="1350" height="718" alt="image" src="https://github.com/user-attachments/assets/a5f7abf1-d66d-4230-b339-15a8c44a8d0a" />

<img width="1363" height="763" alt="image" src="https://github.com/user-attachments/assets/3b6348ed-9e9c-458e-8f86-92866cd0fdfe" />


---

# 19. Tạo giao diện template HTML

## File home.html

Sử dụng:

* HTML
* CSS
* Jinja2 template

để hiển thị danh sách con nợ.

## Kết quả

<img width="1359" height="763" alt="image" src="https://github.com/user-attachments/assets/920895ff-bad3-47ec-91b6-c319dafcb180" />

---

# 20. Tạo view Django

## File views.py

```python
from django.shortcuts import render
from datetime import date

from .models import PhieuCam


def home_page(request):

    debtors = PhieuCam.objects.filter(
        ngay_den_han__lte=date.today(),
        da_tra=False
    )

    context = {
        'debtors': debtors
    }

    return render(request, 'home.html', context)
```

## Giải thích

* Lấy danh sách phiếu cầm đến hạn
* Truyền dữ liệu sang template HTML

---

# 21. Cấu hình URL

## File urls.py

```python
from django.contrib import admin
from django.urls import path

from core.views import home_page

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', home_page),
]
```

## Giải thích

* `/admin` → trang quản trị
* `/` → trang danh sách con nợ

---

# 22. Public website bằng Cloudflare Tunnel

## Đăng nhập Cloudflare

```bash
cloudflared tunnel login
```

## Tạo tunnel

```bash
cloudflared tunnel create pawnshop
```

## Chạy tunnel

```bash
cloudflared tunnel run pawnshop
```

## Kiểm tra tunnel

```bash
sudo systemctl status cloudflared
```

## Giải thích

Cloudflare Tunnel giúp public website ra internet mà không cần mở port modem.


---

# 23. Giao diện website public

## Link

```text
https://pawn.minhluong204.id.vn
```

## Kết quả

<img width="1359" height="763" alt="image" src="https://github.com/user-attachments/assets/920895ff-bad3-47ec-91b6-c319dafcb180" />

---



# 24. Kết luận

Sau khi hoàn thành bài tập, em đã xây dựng được website quản lý tiệm cầm đồ bằng Django kết hợp Docker và MariaDB. 

Hệ thống đã có:
- Django Admin để quản lý dữ liệu
- phpMyAdmin để kiểm tra cơ sở dữ liệu
- Giao diện hiển thị danh sách con nợ
- Cloudflare Tunnel để public website ra internet

Qua bài tập này em hiểu rõ hơn về cách triển khai Django bằng Docker, kết nối cơ sở dữ liệu MariaDB và xây dựng một hệ thống web quản lý cơ bản.
