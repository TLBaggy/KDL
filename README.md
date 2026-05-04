# 📦 KDL – Kho Dữ Liệu Đơn Hàng (Data Warehouse)

Dự án xây dựng hệ thống **Kho Dữ Liệu (Data Warehouse)** cho dữ liệu đơn hàng bán lẻ toàn cầu, bao gồm toàn bộ quy trình từ thu thập, làm sạch, tích hợp dữ liệu (ETL) bằng SSIS, mô hình hoá đa chiều bằng SSAS, và trực quan hoá bằng Power BI.

---

## 🗂️ Cấu trúc dự án

```
KDL-main/
├── QuaTrinhETL_SSIS/        # Dự án SSIS – quy trình ETL
│   ├── Package.dtsx         # Gói ETL chính
│   ├── *.conmgr             # Connection managers (nguồn & đích)
│   └── QuaTrinhETL_SSIS.dtproj
├── SSAS/                    # Dự án SSAS – mô hình OLAP Cube
│   └── SSAS/
│       ├── DONHANG.cube     # Cube chính
│       ├── DONHANG.ds       # Data Source
│       ├── DONHANG.dsv      # Data Source View
│       ├── Dim *.dim        # Các bảng chiều (Dimensions)
│       └── bin/SSAS.asdatabase
├── Thi_Dashboard.pbix       # Dashboard Power BI
├── SQL.sql                  # Script reset dữ liệu
├── MDX.mdx                  # Các câu truy vấn MDX phân tích
└── README.md
```

---

## 🏗️ Kiến trúc hệ thống

```
[Nguồn dữ liệu thô]
        │
        ▼
[SSIS – ETL Pipeline]
  ├── Import Source (nhập dữ liệu thô)
  ├── Quá trình làm sạch (xử lý NULL, chuẩn hoá)
  ├── Reset Data (xoá dữ liệu cũ)
  ├── Create Dim (nạp các bảng chiều)
  └── Fact (nạp bảng sự kiện)
        │
        ▼
[SQL Server – Database DONHANG]
  ├── Dim_Category       ├── Dim_Customer
  ├── Dim_Date           ├── Dim_Department
  ├── Dim_Location       ├── Dim_Market
  ├── Dim_Product        ├── Dim_ShippingMode
  ├── Fact_Sales         └── Fact_Shipping
        │
        ▼
[SSAS – OLAP Cube DONHANG]
        │
        ▼
[Power BI – Dashboard phân tích]
```

---

## ⚙️ Quy trình ETL (SSIS)

Dự án SSIS (`QuaTrinhETL_SSIS`) thực hiện toàn bộ quy trình nạp dữ liệu vào Data Warehouse theo các bước:

| Bước | Tên Task | Mô tả |
|------|----------|-------|
| 1 | **Import Source** | Nhập dữ liệu thô từ nguồn vào bảng staging |
| 2 | **Quá trình làm sạch** | Xử lý giá trị NULL, chuẩn hoá định dạng |
| 3 | **Reset Data** | Xoá dữ liệu cũ trong các bảng Dim và Fact |
| 4 | **Create Dim** | Nạp dữ liệu vào các bảng chiều (Dim_*) |
| 5 | **Fact** | Nạp dữ liệu vào bảng sự kiện (Fact_Sales, Fact_Shipping) |

**Connection Managers:**
- `LAPTOP-6B6J647F\TLBAGGY_SQL.DONHANG` – Kết nối tới cơ sở dữ liệu đích
- `LAPTOP-6B6J647F\TLBAGGY_SQL.DuLieuLamSach` – Kết nối tới dữ liệu đã làm sạch

---

## 🧊 Mô hình OLAP Cube (SSAS)

Cube `DONHANG` được xây dựng trên SQL Server Analysis Services theo mô hình **Star Schema**.

### 📐 Các bảng chiều (Dimensions)

| Dimension | Thuộc tính chính |
|-----------|-----------------|
| **Dim Customer** | Customer Id, Customer Name, Customer Segment |
| **Dim Product** | Product Card Id, Product Name, Product Price |
| **Dim Category** | Category Id, Category Name |
| **Dim Department** | Department Id, Department Name |
| **Dim Location** | Location Id, City, State, Country, Region |
| **Dim Market** | Market |
| **Dim Shipping Mode** | Shipping Mode |
| **Dim Date** | Date Key, Day, Month, Quarter, Year |

### 📊 Các Measure Groups (Bảng sự kiện)

**Fact Sales:**
| Measure | Mô tả |
|---------|-------|
| Sales Amount | Tổng doanh thu |
| Quantity | Số lượng sản phẩm |
| Unit Price | Đơn giá |
| Discount Amount | Giá trị chiết khấu |
| Discount Rate | Tỷ lệ chiết khấu |
| Benefit Per Order | Lợi nhuận mỗi đơn |
| Sales Per Customer | Doanh thu trên mỗi khách hàng |
| Fact Sales Count | Số đơn hàng |

**Fact Shipping:**
| Measure | Mô tả |
|---------|-------|
| Days For Shipment Scheduled | Số ngày giao hàng dự kiến |
| Days For Shipping Real | Số ngày giao hàng thực tế |
| Late Delivery Risk | Rủi ro giao trễ |
| Fact Shipping Count | Số lần vận chuyển |

---

## 📝 Truy vấn MDX mẫu

File `MDX.mdx` bao gồm các phân tích điển hình trên Cube:

1. **Top 5 thành phố tại California** có tổng doanh thu cao nhất
2. **Số đơn giao trễ** phân theo phương thức vận chuyển (Standard / Express) theo năm
3. **Thống kê số đơn hàng** của các Department có doanh thu trên 1.000 USD qua các năm
4. **Doanh thu và lợi nhuận** của phân khúc Corporate theo năm
5. **Tổng số đơn hàng** của Department Golf phân theo thành phố
6. **Doanh thu và lợi nhuận** theo Market (APAC, EMEA, US) qua các năm
7. **Top 10 sản phẩm** bán chạy nhất theo Sales Amount

---

## 📊 Dashboard Power BI

File `Thi_Dashboard.pbix` kết nối trực tiếp với SSAS Cube để trực quan hoá các chỉ số kinh doanh, bao gồm doanh thu, lợi nhuận, hiệu suất vận chuyển và phân tích thị trường.

---

## 🚀 Hướng dẫn triển khai

### Yêu cầu hệ thống

- SQL Server 2016+ với SQL Server Integration Services (SSIS)
- SQL Server Analysis Services (SSAS) – Multidimensional Mode
- Power BI Desktop
- Visual Studio với SQL Server Data Tools (SSDT)

### Các bước thực hiện

1. **Tạo database** `DONHANG` và `DuLieuLamSach` trên SQL Server
2. **Chạy script** `SQL.sql` để reset/chuẩn bị các bảng
3. **Mở SSIS project** trong `QuaTrinhETL_SSIS/` bằng Visual Studio, cập nhật connection strings phù hợp với máy chủ cục bộ, sau đó deploy và chạy `Package.dtsx`
4. **Mở SSAS project** trong `SSAS/`, deploy Cube lên Analysis Services và tiến hành Process Cube
5. **Mở** `Thi_Dashboard.pbix` bằng Power BI Desktop, cập nhật data source trỏ tới SSAS Cube đã deploy

> **Lưu ý:** Connection string hiện tại trong dự án trỏ tới `LAPTOP-6B6J647F\TLBAGGY_SQL`. Cần đổi thành tên SQL Server instance của máy bạn trước khi chạy.

---

## 📁 Dữ liệu nguồn

Dữ liệu đơn hàng bao gồm thông tin về khách hàng, sản phẩm, danh mục, cửa hàng (department), địa điểm giao hàng, thị trường toàn cầu (APAC, EMEA, US, …) và phương thức vận chuyển.
