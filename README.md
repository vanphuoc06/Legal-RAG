# Knowledge Graph-Based RAG System

## Project Overview

Đây là một framework RAG dựa trên Đồ thị Tri thức (Knowledge Graph) gọn nhẹ và hiệu quả. Hệ thống áp dụng kiến trúc hai lớp (dual-layer) để quản lý đồng thời Đồ thị tri thức và Vector embeddings, giúp kết nối và thu hẹp khoảng cách giữa phương pháp RAG dựa trên vector truyền thống và RAG dựa trên đồ thị. 

Mục tiêu của hệ thống là cung cấp khả năng tìm kiếm và truy xuất mạnh mẽ, dễ dàng mở rộng và tinh chỉnh cho nhiều bài toán đặc thù.

## Architecture Overview

Kiến trúc hệ thống được thiết kế tối ưu, có khả năng mở rộng cao và tích hợp mượt mà với các mô hình cũng như cơ sở dữ liệu lưu trữ chạy cục bộ (local).

```mermaid
graph TB
    classDef userApp fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef module fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef storage fill:#fff3e0,stroke:#f57c00,stroke-width:2px;
    classDef external fill:#f1f8e9,stroke:#689f38,stroke-width:2px;

    User(["Ứng dụng Người dùng / WebUI"]):::userApp --> |REST API / SDK| API["API Server"]:::module

    subgraph Core[Core Framework]
        direction TB
        Ingest["Luồng Xử lý Tài liệu"]:::module
        Query["Luồng Truy vấn & Tìm kiếm"]:::module

        Ingest --> Parse["Phân tích & Cắt Text (Chunking)"]:::module
        Parse --> Extract["Trích xuất Thực thể & Mối quan hệ"]:::module
        Extract --> Embed["Tạo Vector Embedding"]:::module

        Query --> Retrieve["Truy xuất Hai lớp (Dual-Level)"]:::module
        Retrieve --> Gen["Tạo câu trả lời (LLM Generation)"]:::module
    end
    
    API --> Core

    subgraph StorageLayer[Lưu trữ Cục bộ (Local Storage)]
        direction LR
        KV[("KV Storage")]:::storage
        Vector[("Vector Storage")]:::storage
        Graph[("Graph Storage")]:::storage
        Doc[("Trạng thái Tài liệu")]:::storage
    end

    subgraph Models[Mô hình Cục bộ (Local Models)]
        direction LR
        LLM["Large Language Models"]:::external
        EmbedMod["Embedding Models"]:::external
        VLM["Vision Language Models"]:::external
    end

    Extract --> Models
    Embed --> EmbedMod
    Gen --> LLM
    Parse -.-> VLM

    Extract --> KV
    Extract --> Graph
    Embed --> Vector
    Ingest --> Doc

    Retrieve --> KV
    Retrieve --> Vector
    Retrieve --> Graph
```

## Running Locally

Làm theo các bước sau để khởi chạy Server API và WebUI trực tiếp trên máy của bạn.

### 1. Prerequisites

Khuyến nghị sử dụng [uv](https://docs.astral.sh/uv/) để quản lý các gói Python một cách nhanh chóng và ổn định:
- **Windows**: `powershell -c "irm https://astral.sh/uv/install.ps1 | iex"`
- **Unix/macOS**: `curl -LsSf https://astral.sh/uv/install.sh | sh`

Đảm bảo bạn đã cài đặt [Bun](https://bun.sh/) để có thể build giao diện WebUI.

### 2. Install and Setup

Clone mã nguồn dự án và thiết lập môi trường phát triển:

```bash
git clone <URL_REPO_CỦA_BẠN>
cd <TÊN_THƯ_MỤC_REPO>

# Thiết lập môi trường phát triển (Cài đặt các thư viện cần thiết & build frontend)
make dev

# Kích hoạt môi trường ảo (virtual environment)
# Trên Windows:
.venv\Scripts\activate
# Trên Linux/macOS:
# source .venv/bin/activate  
```

### 3. Environment Configuration

Khởi tạo file cấu hình biến môi trường gốc:

```bash
make env-base  # Lệnh này sẽ tạo ra file .env
```
Sau đó, hãy mở và cập nhật file `.env` với các cấu hình LLM và embedding cục bộ của bạn (ví dụ: cấu hình để gọi API từ **Ollama** hoặc các nền tảng tự host khác).

### 4. Start Server

Khởi chạy API server và giao diện WebUI bằng lệnh CLI của dự án (ví dụ):

```bash
# Lệnh khởi chạy server (Tùy thuộc vào thiết lập package của bạn)
# Ví dụ:
uvicorn api.server:app --reload 
# Hoặc sử dụng CLI được cung cấp sẵn của dự án
```

Sau khi khởi chạy thành công, Server API và giao diện web sẽ có sẵn ở môi trường cục bộ để bạn bắt đầu thao tác (ví dụ như upload tài liệu và thực hiện tìm kiếm).

## Key Configurations (Local)

Khi chạy cục bộ, bạn có thể điều chỉnh các biến môi trường sau trong file `.env` để kiểm soát hành vi của hệ thống:

- **Local LLM Providers**: Cấu hình `LLM_BINDING` và `LLM_MODEL` trỏ tới các mô hình AI cục bộ của bạn.
- **Embedding Models**: Thiết lập `EMBEDDING_BINDING` và `EMBEDDING_MODEL`. Nên sử dụng các mô hình nhỏ, tốc độ cao (ví dụ: `BAAI/bge-m3`).
- **Query Mode (Chế độ Truy vấn)**: Hệ thống hỗ trợ 5 chế độ truy vấn (`local`, `global`, `hybrid`, `naive`, `mix`). Chế độ mặc định `mix` mang lại kết quả toàn diện nhất.
- **Storage Backends (Lưu trữ Backend)**: Mặc định, hệ thống sử dụng lưu trữ dưới dạng file cục bộ (file-persisted). Đối với các hệ thống dữ liệu lớn, bạn có thể cấu hình để hệ thống kết nối với cơ sở dữ liệu như PostgreSQL, Milvus, hoặc Neo4j cục bộ thông qua biến môi trường.
