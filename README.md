# Tối ưu hóa truy xuất trong hệ thống RAG thông qua kỹ thuật bổ sung ngữ cảnh chủ động và làm phẳng dữ liệu bán cấu trúc

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Docker](https://img.shields.io/badge/Docker-Supported-blue.svg)](https://www.docker.com/)

> **Trình bày tại:** HỘI THẢO KHOA HỌC QUỐC GIA VỀ CÔNG NGHỆ THÔNG TIN VÀ TRUYỀN THÔNG (ICT) 2026
> **Địa điểm:** Đồng Tháp
> **Ngày:** 22/5/2026

## 📋 Tổng quan

Nghiên cứu này đề xuất một quy trình tiền xử lý dữ liệu ở tầng biểu diễn (Data Representation Layer) nhằm cải thiện độ chính xác truy xuất cho hệ thống RAG khi xử lý dữ liệu bán cấu trúc (JSON) hoặc danh sách phân cấp dài. Phương pháp kết hợp hai kỹ thuật: làm phẳng dữ liệu sang ngôn ngữ tự nhiên (Natural Language Flattening) và bổ sung ngữ cảnh chủ động (Context Injection) để đảm bảo tính độc lập ngữ nghĩa cho từng phân đoạn văn bản (chunks).

### 🎯 Vấn đề giải quyết
- **Đứt gãy cấu trúc logic:** Các thuật toán phân mảnh truyền thống (Fixed-size/Recursive) cắt ngang các khối dữ liệu JSON phân cấp sâu.
- **Mất ngữ cảnh định danh (Context Loss):** Các phân đoạn phía dưới bị cô lập khỏi siêu dữ liệu định danh cấp cao (top-level metadata), gây ra hiện tượng ảo giác (hallucination) ở mô hình ngôn ngữ lớn (LLM).

### 💡 Giải pháp đề xuất
Quy trình tiền xử lý kiểm soát phân mảnh chủ động (Controlled Chunking):
- **Làm phẳng dữ liệu:** Chuyển đổi các cặp key-value lồng nhau thành các mệnh đề ngôn ngữ tự nhiên mạch lạc.
- **Bổ sung ngữ cảnh chủ động:** Lặp lại các chuỗi siêu dữ liệu định danh cố định vào đầu mỗi phân đoạn logic trước khi thực hiện quá trình nhúng (embedding).

## 🏗️ Kiến trúc hệ thống


```

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Dữ liệu thô    │    │ Làm phẳng & Gắn │    │  Mã hóa Vector  │
│     (JSON)      │───▶│ Ngữ cảnh Cố định│───▶│   & Lưu trữ     │
│                 │    │  (Plain Text)   │    │  (Chunknig 1K)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
│
▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Đánh giá tự động│    │ Mô hình Suy luận│    │ Hệ thống RAG    │
│  Framework RAGAS│◀───│ Llama-3.1-8b API│◀───│ AnythingLLM     │
│ (GPT-4o-mini J.)│    │ (Local ready)   │    │ (Docker/Ubuntu) │
└─────────────────┘    └─────────────────┘    └─────────────────┘

```

### Luồng xử lý chính:
1. **Tiền xử lý**: Trích xuất dữ liệu JSON → Làm phẳng sang văn bản tự nhiên → Gắn siêu dữ liệu định danh.
2. **Lưu trữ & Truy xuất**: Nạp vào AnythingLLM → Nhúng vector với Gemini Embedding → Kiểm soát phân mảnh theo khối logic.
3. **Tạo sinh & Đánh giá**: Truy vấn qua API Llama-3.1-8b → Chấm điểm tự động bằng mô hình giám khảo GPT-4o-mini (RAGAS).

## 📊 Kết quả thực nghiệm

Hiệu năng hệ thống được đánh giá tự động dựa trên bộ dữ liệu kiểm chuẩn (Ground Truth) gồm 50 câu hỏi truy vấn về thông tin tuyển sinh của Trường Đại học Nha Trang.

### Thống kê thang đo RAGAS (Trung bình trên 50 truy vấn)
| Thang đo (Metrics) | Baseline 1 (JSON thô) | Baseline 2 (Markdown thô) | Phương pháp đề xuất |
|--------------------|-----------------------|---------------------------|---------------------|
| Context Precision  | 0.7933                | 0.7817                    | **0.8556 (+6.23%)** |
| Faithfulness       | 0.5484                | 0.6371                    | **0.7990 (+16.19%)**|
| Answer Correctness | 0.5524                | 0.6181                    | **0.7017 (+8.36%)** |

### Đánh giá sự đánh đổi (Trade-offs):
- Kỹ thuật lặp lại siêu dữ liệu làm gia tăng **18%** tổng số lượng token của cơ sở dữ liệu vector.
- Độ trễ hệ thống tăng thêm trung bình **45ms** ở pha nhúng và **120ms** ở pha tạo sinh, đổi lại sự gia tăng vượt trội về độ chính xác và giảm thiểu tối đa hiện tượng ảo giác.

## 🛠️ Công nghệ sử dụng

- **Hạ tầng & Điều phối**: AnythingLLM (Dockerized), Linux (Ubuntu 24.04)
- **Mô hình nhúng (Embedding)**: Google Gemini Embedding 001 API
- **Mô hình ngôn ngữ (LLM)**: Llama-3.1-8b-instant (Cloud API Gateway / Hỗ trợ Local Deployment)
- **Mô hình giám khảo (Judge)**: OpenAI GPT-4o-mini (Tham số temperature = 0)
- **Thư viện đánh giá**: Framework RAGAS (Đã tinh chỉnh và Việt hóa bộ hệ thống prompt nội bộ)
- **Tham số phân mảnh**: Chunk Size = 1000 tokens, Chunk Overlap = 200 tokens

## 📝 Dữ liệu thực nghiệm

- **Cơ sở tri thức**: Đề án tuyển sinh Đại học chính quy năm 2025 và 2026 của Trường Đại học Nha Trang.
- **Độ phức tạp**: Bao gồm điểm chuẩn của 50 mã ngành đào tạo, tổ hợp môn xét tuyển, và các quy định điều kiện phụ đi kèm.
- **Bộ dữ liệu kiểm thử**: 50 câu hỏi (15 câu trích xuất đơn lẻ, 15 câu so sánh đa ngành, 20 câu tổng hợp logic) được thẩm định bởi chuyên gia (Human-in-the-loop).

## 🔮 Hướng phát triển

- [ ] **Mô hình nội bộ (Green AI)**: Triển khai toàn bộ hệ thống trên các mô hình ngôn ngữ nhỏ (Small/Local Models) được tinh chỉnh riêng cho tiếng Việt.
- [ ] **Thuật toán nâng cao**: Tích hợp thử nghiệm cùng các chiến lược phân định ranh giới ngữ cảnh như Parent Document Retrieval (PDR).
- [ ] **Mở rộng miền dữ liệu**: Thử nghiệm và kiểm chứng khả năng khái quát hóa của phương pháp trên các định dạng bán cấu trúc phức tạp khác ngoài giáo dục.

## 👥 Tác giả

- **Ngô Nguyễn Tường Nghi** - Khoa Công nghệ thông tin, Trường Đại học Nha Trang
  - Email: nghinnt@ntu.edu.vn
  
- **Lê Thị Bích Hằng** - Khoa Công nghệ thông tin, Trường Đại học Nha Trang
  - Email: hangltb@ntu.edu.vn

- **Nguyễn Đình Hưng** - Khoa Công nghệ thông tin, Trường Đại học Nha Trang
  - Email: hungnd@ntu.edu.vn

## 📄 Trích dẫn

Nếu bạn sử dụng nghiên cứu này, vui lòng trích dẫn:

```bibtex
@inproceedings{ngo2026rag,
  title={Tối ưu hóa truy xuất trong hệ thống RAG thông qua kỹ thuật bổ sung ngữ cảnh chủ động và làm phẳng dữ liệu bán cấu trúc},
  author={Nghi, Ngô Nguyễn Tường and Hằng, Lê Thị Bích và Hưng, Nguyễn Đình},
  booktitle={Hội thảo khoa học Quốc gia về Công nghệ thông tin và Truyền thông (ICT)},
  year={2026},
  address={Đồng Tháp, Việt Nam}
}

```

## 📜 License

Dự án này được phát hành dưới [MIT License](https://www.google.com/search?q=LICENSE).
