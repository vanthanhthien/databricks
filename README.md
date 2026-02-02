# 🛒 FMCG Sales Data Pipeline | End-to-End Data Engineering

![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![Apache Spark](https://img.shields.io/badge/Apache%20Spark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![AWS S3](https://img.shields.io/badge/AWS%20S3-569A31?style=for-the-badge&logo=amazon-s3&logoColor=white)

## 📖 Project Overview
Dự án xây dựng một hệ thống xử lý dữ liệu (ETL Pipeline) hiện đại cho ngành hàng tiêu dùng nhanh (FMCG). Hệ thống tiếp nhận dữ liệu giao dịch thô, tự động làm sạch, chuẩn hóa và tổng hợp theo mô hình **Star Schema** trên nền tảng **Databricks** và **Delta Lake**.

Mục tiêu chính là chuyển đổi dữ liệu giao dịch chi tiết (Daily Grain) thành dữ liệu báo cáo tổng hợp theo tháng (Monthly Grain) để phục vụ Dashboard phân tích doanh số.

## 🏗 Architecture & Data Flow (Medallion Architecture)

Dữ liệu di chuyển qua 3 tầng (Layers) chuẩn công nghiệp:

1.  **Bronze Layer (Raw Ingestion):**
    * Đọc dữ liệu CSV từ **AWS S3**.
    * Lưu trữ nguyên trạng vào Delta Table.
    * Thêm metadata kỹ thuật (`read_timestamp`, `source_file`).
    * Tự động di chuyển file đã xử lý sang thư mục `processed/`.

2.  **Silver Layer (Cleansed & Enriched):**
    * **Data Quality:** Sửa lỗi chính tả (e.g., `Hyderabadd` -> `Hyderabad`), xử lý `NULL`, chuẩn hóa định dạng ngày tháng (xử lý hỗn hợp `yyyy/MM/dd`, `dd-MM-yyyy`...).
    * **Normalization:** Tách thông tin biến thể sản phẩm (Variant) từ tên sản phẩm bằng Regex.
    * **Hashing:** Tạo `product_code` bằng thuật toán **SHA2** để tạo khóa định danh bền vững.
    * **Pricing Logic:** Áp dụng Window Functions để lấy giá sản phẩm mới nhất theo từng năm.

3.  **Gold Layer (Aggregated for Business):**
    * **Star Schema:** Xây dựng bảng Fact (`fact_orders`) và các bảng Dimension (`dim_products`, `dim_customers`, `dim_date`).
    * **Aggregation:** Tổng hợp dữ liệu từ cấp độ ngày (Daily) lên cấp độ tháng (Monthly) để tối ưu hiệu năng truy vấn báo cáo.

## 🚀 Key Technical Features

### 1. Incremental Loading Strategy (Tải dữ liệu gia tăng)
Thay vì tải lại toàn bộ dữ liệu (Full Load) gây tốn kém tài nguyên, hệ thống sử dụng cơ chế **Staging & Merge**:
* Sử dụng bảng trung gian `staging_orders` chỉ chứa dữ liệu mới về.
* Sử dụng câu lệnh `MERGE INTO` (Upsert) để cập nhật dữ liệu vào bảng Gold.
* Tự động tính toán lại các chỉ số tổng hợp (Aggregates) cho các tháng bị ảnh hưởng bởi dữ liệu mới.

### 2. Advanced Data Transformations
* **Dynamic Date Dimension:** Tự động sinh bảng thời gian bằng hàm `sequence()` và `explode()` của Spark thay vì dùng file tĩnh.
* **Regex Extraction:** Trích xuất thông tin trọng lượng/quy cách đóng gói (e.g., "30 Sachets", "60g") từ chuỗi văn bản phi cấu trúc.
* **Window Functions:** Xử lý logic thay đổi giá theo thời gian (SCD Type 1 logic for pricing).

## 📂 Project Structure

```text
project-de-fmcg-atlikon/
├── 0_data/                          # Sample Raw Data
├── 1_codes/
│   ├── 1_setup/
│   │   ├── dim_date_table_creation.ipynb  # Sinh bảng Dim Date tự động
│   │   ├── setup_catalog.ipynb            # Cấu hình Unity Catalog & Schema
│   │   └── utilities.ipynb                # Các biến/hàm dùng chung
│   ├── 2_dimension_data_processing/       # Pipeline xử lý Dimension (SCD Type 1)
│   │   ├── 1_customers_data_processing.ipynb
│   │   ├── 2_products_data_processing.ipynb
│   │   └── 3_pricing_data_processing.ipynb
│   └── 3_fact_data_processing/            # Pipeline xử lý Fact
│       ├── 1_full_load_fact.ipynb         # Tải lại toàn bộ lịch sử
│       └── 2_incremental_load_fact.ipynb  # Tải dữ liệu mới & Upsert
├── 2_dashboarding/
│   ├── denormalise_table_query_fmcg.txt   # SQL Query phục vụ BI Tool
│   └── fmcg_dashboard.pdf                 # Kết quả báo cáo mẫu
└── resources/
🛠 Tech Stack
Platform: Databricks (Community/Standard Edition)

Compute Engine: Apache Spark (PySpark & Spark SQL)

Storage: Delta Lake (ACID Transactions support)

Orchestration: Databricks Notebook Workflows

Language: Python, SQL

📝 Usage Guide
Setup Environment:

Mount S3 bucket hoặc upload dữ liệu vào DBFS.

Chạy 1_codes/1_setup/setup_catalog.ipynb để khởi tạo Database.

Run Dimensions:

Thực thi lần lượt các notebook trong 2_dimension_data_processing để chuẩn bị dữ liệu tham chiếu.

Run Fact Pipeline:

Lần đầu: Chạy 3_fact_data_processing/1_full_load_fact.ipynb.

Hàng ngày/Hàng tháng: Chạy 3_fact_data_processing/2_incremental_load_fact.ipynb để cập nhật dữ liệu mới nhất.

📊 Sample Insights
Dữ liệu sau khi xử lý cho phép trả lời các câu hỏi kinh doanh:

Doanh số bán hàng theo từng tháng của từng dòng sản phẩm (Energy Bars vs Protein Bars) là bao nhiêu?

Khách hàng nào tại khu vực Hyderabad có lượng mua hàng tăng trưởng cao nhất?

Author: [Van Thanh Thien] Aspiring Data Engineer | Spark | Cloud | Big Data
