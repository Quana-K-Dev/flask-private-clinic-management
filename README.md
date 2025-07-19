
# 🏥 Clinic Management System – Flask Private Clinic Management

> **Quản lý Phòng Mạch Tư** – Đồ án tốt nghiệp Đại học Mở TP. Hồ Chí Minh  
> **Nhóm thực hiện:**  
> • 2251050031 – Nguyễn Thanh Hoàng  
> • 2251050002 – Nguyễn Đức An  
> • 2251050061 – Nguyễn Trung Quân  
> **Năm:** 2024  

---

## 📖 Mục lục

1. [Giới thiệu](#giới-thiệu)
2. [Tính năng](#tính-năng)
3. [Kiến trúc hệ thống](#kiến-trúc-hệ-thống)
4. [Phân tích yêu cầu & Use‑Case](#phân-tích-yêu-cầu--use‑case)
5. [Thiết kế cơ sở dữ liệu](#thiết-kế-cơ-sở-dữ-liệu)
6. [Thiết kế giao diện](#thiết-kế-giao-diện)
7. [Cài đặt & Khởi chạy](#cài-đặt--khởi-chạy)
8. [Cấu trúc thư mục dự án](#cấu-trúc-thư-mục-dự-án)
9. [Lộ trình phát triển](#lộ-trình-phát-triển)
10. [Đóng góp](#đóng-góp)
11. [Giấy phép](#giấy-phép)
12. [Liên hệ](#liên-hệ)

---

## Giới thiệu

**Clinic Management System** (CMS) là phần mềm web mã nguồn mở được xây dựng bằng **Flask** nhằm số hoá toàn bộ quy trình quản lý hoạt động của **phòng mạch tư**:

* **Quản lý bệnh nhân** – lưu trữ hồ sơ, lịch sử khám chữa.
* **Quản lý lịch hẹn** – đặt, sửa, hủy và nhắc lịch.
* **Lập phiếu khám & kê đơn** – hỗ trợ bác sĩ khám, chẩn đoán, tra cứu thuốc và in phiếu.
* **Quản lý thuốc & kho** – thêm, sửa, xoá thuốc, đơn vị tính, tồn kho.
* **Thống kê – báo cáo** – theo dõi doanh thu, lượng bệnh, tồn kho, xuất báo cáo PDF/Excel.
* **Hệ thống phân quyền** – Bệnh nhân · Nhân viên Hành chính · Bác sĩ · Quản trị viên.

Dự án hướng tới **tăng hiệu quả chuyên môn**, **giảm sai sót giấy tờ**, đồng thời mang tới **trải nghiệm số hoá** cho cả bệnh nhân lẫn nhân viên y tế.

---

## Tính năng

| Nhóm chức năng | Mô tả chi tiết |
|----------------|---------------|
| **Đăng ký khám** (UC001) | Nhân viên hành chính đặt lịch. Kiểm tra lịch bác sĩ, ghi nhận thông tin bệnh nhân mới, xuất phiếu hẹn. |
| **Tra cứu thuốc** (UC002) | Bác sĩ tìm thuốc theo tên/ký tự gợi ý, xem chi tiết thành phần, đơn vị tính, giá, tồn kho. |
| **Thống kê doanh thu** (UC003) | Quản trị viên lọc doanh thu theo ngày/tháng/năm, hiển thị bảng & biểu đồ ChartJS, xuất file. |
| **Lập phiếu khám** (UC004) | Bác sĩ tra cứu lịch sử bệnh, ghi chẩn đoán, chỉ định thuốc (nhiều đơn vị), in & lưu hồ sơ. |
| **Thêm thuốc** (UC005) | Quản trị viên thêm thuốc, đơn vị tính, giá nhập/xuất, tự động tạo nhiều Unit, chỉnh sửa, huỷ. |
| **Quản trị hệ thống** | Quản lý User, Policy (quy định), Role & Permission, Backup DB, nhật ký hoạt động. |
| **Báo cáo & Biểu đồ** | Báo cáo MedicalVisit, doanh thu, kho thuốc. Biểu đồ thời gian thực. |
| **Bảo mật** | Xác thực JWT + Flask‑Login, CSRF Protect, Bcrypt password‑hash, phân quyền RBAC. |

---

## Kiến trúc hệ thống

```
┌─────────────────┐
│   Presentation  │  Flask Jinja2  →  Templates (HTML, CSS, JS)
└─────────────────┘
          │
          ▼
┌─────────────────┐
│  Application    │  Flask Blueprint  →  auth, admin, doctor, staff, patient
└─────────────────┘
          │
          ▼
┌─────────────────┐
│    Service      │  Business Logic, Validation, Scheduler
└─────────────────┘
          │
          ▼
┌─────────────────┐
│  Persistence    │  SQLAlchemy ORM  →  SQLite / MySQL
└─────────────────┘
```

* **MVC** thuần Flask (Blueprint = Controller, Jinja2 = View).  
* **Responsive UI** với Bootstrap 5 + Vanilla JS.  
* **Background job**: APScheduler gửi email nhắc lịch.  
* **DevOps**: Dockerfile & docker‑compose cho prod; GitHub Actions CI test + lint.

---

## Phân tích yêu cầu & Use‑Case

### Lược đồ Use‑Case Tổng quát

> `docs/diagrams/usecase_overview.png`

### Bảng đặc tả Use‑Case

| ID | Tên | Actor chính | Mục tiêu | Tiền điều kiện | Hậu điều kiện |
|----|-----|-------------|----------|----------------|---------------|
| UC001 | Đăng ký khám | Staff | Tạo lịch hẹn cho bệnh nhân | Staff đăng nhập | Lịch hẹn lưu DB |
| UC002 | Tra cứu thuốc | Doctor | Xem chi tiết thuốc | Doctor đăng nhập | Hiển thị thông tin |
| UC003 | Thống kê doanh thu | Admin | Thống kê thu chi | Admin đăng nhập | Hiển thị báo cáo |
| UC004 | Lập phiếu khám | Doctor | Kê đơn & lưu hồ sơ | Doctor đăng nhập | Phiếu khám được lưu |
| UC005 | Thêm thuốc | Admin | Cập nhật kho thuốc | Admin đăng nhập | Thuốc mới trong DB |

> Xem thêm file [`docs/usecase_spec.md`](docs/usecase_spec.md) để biết **Main Flow / Alternative Flow / Exception Flow** chi tiết.

---

## Thiết kế cơ sở dữ liệu

![ERD](docs/diagrams/erd.png)

### Mô hình dữ liệu chính

| Bảng | Vai trò | Quan hệ |
|------|---------|---------|
| **user** | Lưu thông tin đăng nhập | 1‑1 với *patient*, *doctor*, *staff*, *admin* |
| **appointment** | Lịch hẹn | N‑1 *patient*, N‑1 *doctor*, N‑1 *staff* |
| **medical_visit** | Phiếu khám | 1‑1 *appointment*, N‑1 *doctor*, N‑1 *patient* |
| **medicine** | Thuốc | N‑M *medical_visit* thông qua *medicalvisit_medicine* |
| **receipt** | Hóa đơn | N‑1 *medical_visit*, N‑1 *patient*, N‑1 *staff* |
| **policy** | Quy định | 1‑N *admin* |
| *(…xem chi tiết trong `docs/database.md`)* |

---

## Thiết kế giao diện

> Folder `screenshots/` chứa hình chụp thực tế (xem file báo cáo gốc).  
> Một số màn hình tiêu biểu:

| Màn hình | Mô tả |
|----------|-------|
| **Đăng ký khám** | Chọn ngày, bác sĩ; kiểm tra lịch trống; tạo appointment. |
| **Phiếu khám** | Xem lịch sử bệnh, nhập chẩn đoán, chọn thuốc, in PDF. |
| **Thêm thuốc** | Form động thêm nhiều đơn vị (Unit, Price, Quantity). |
| **Thống kê doanh thu** | Chọn mốc thời gian, hiển thị bảng + Chart JS. |

---

## Cài đặt & Khởi chạy

### 1. Yêu cầu hệ thống

| Phần mềm | Phiên bản |
|----------|-----------|
| Python | 3.10+ |
| Node (tùy chọn để build assets) | 18+ |
| SQLite | 3.35+ (hoặc MySQL 8) |

### 2. Clone & tạo môi trường ảo

```bash
git clone https://github.com/<YOUR_USERNAME>/flask-private-clinic-management.git
cd flask-private-clinic-management
python -m venv venv
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Cấu hình biến môi trường (file `.env`)

```ini
FLASK_APP=app
FLASK_ENV=development
SECRET_KEY=your_secret_key
DATABASE_URL=sqlite:///instance/clinic.db
JWT_SECRET=jwt_secret_key
```

### 4. Khởi tạo DB & chạy

```bash
flask db upgrade       # Nếu dùng Flask-Migrate
flask seed             # Script seed dữ liệu mẫu (tuỳ chọn)
flask run
```

Truy cập **http://127.0.0.1:5000**. Tài khoản mẫu:

| Quyền | User | Pass |
|-------|------|------|
| Admin | admin | 123456 |
| Doctor | dr.house | 123456 |
| Staff | staff1 | 123456 |

> Sửa đổi trong `seed.py` tuỳ ý.

---

## Cấu trúc thư mục dự án

```
.
├── app/
│   ├── __init__.py
│   ├── models/
│   ├── services/
│   ├── blueprints/
│   │   ├── auth/
│   │   ├── admin/
│   │   ├── doctor/
│   │   └── staff/
│   ├── templates/
│   └── static/
├── docs/
│   ├── diagrams/
│   ├── usecase_spec.md
│   └── database.md
├── screenshots/
├── tests/
├── migrations/
├── requirements.txt
└── README.md
```

---

## Lộ trình phát triển

- [x] Hoàn thiện CRUD cơ bản & phân quyền
- [x] Báo cáo doanh thu + biểu đồ
- [ ] Module đặt lịch **online** cho bệnh nhân (RESTful API + React)
- [ ] Tích hợp gửi **email/SMS** nhắc lịch
- [ ] Triển khai **Docker + CI/CD** trên GitHub Actions & Render
- [ ] Mã hoá dữ liệu nhạy cảm (Fernet)
- [ ] Đa ngôn ngữ (VN/EN)

---

## Đóng góp

Chúng tôi hoan nghênh mọi **pull request** và **issue**!

1. Fork & tạo nhánh feature: `git checkout -b feature/your-awesome-feature`
2. Commit có message rõ ràng
3. Tạo pull request mô tả chi tiết

Hãy đảm bảo **pytest** pass toàn bộ (`pytest -q`) và **pre‑commit** không lỗi.

---

## Giấy phép

Dự án được phát hành theo giấy phép **MIT**. Xem file [`LICENSE`](LICENSE) để biết thêm chi tiết.

---

## Liên hệ

| Tên | Email |
|-----|-------|
| Nguyễn Thanh Hoàng | hoang.nt@example.com |
| Nguyễn Đức An | an.nd@example.com |
| Nguyễn Trung Quân | quan.nt@example.com |

Nếu dự án hữu ích, hãy ⭐ **star** repo để ủng hộ!

---
