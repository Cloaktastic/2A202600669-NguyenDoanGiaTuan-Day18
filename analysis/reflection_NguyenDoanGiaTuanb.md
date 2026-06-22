# Individual Reflection — Lab 18

**Tên:** Nguyễn Doàn Gia Tuấn  

---

## 1. Đóng góp kỹ thuật

- **Module đã implement:** Hoàn thành toàn bộ 5 modules của Production RAG Pipeline (Chunking, Hybrid Search, Reranking, Evaluation, Enrichment).
- **Các hàm/class chính đã viết:**
  - `m1_chunking`: `chunk_semantic()`, `chunk_hierarchical()`, `chunk_structure_aware()`.
  - `m2_search`: `segment_vietnamese()`, `BM25Search`, `DenseSearch`, `reciprocal_rank_fusion()`.
  - `m3_rerank`: `CrossEncoderReranker`.
  - `m4_eval`: `evaluate_ragas()`, `failure_analysis()`.
  - `m5_enrichment`: `summarize_chunk()`, `generate_hypothesis_questions()`, `contextual_prepend()`, `extract_metadata()`, `_enrich_single_call()`.
- **Số tests pass:** 37 / 37 tests.

---

## 2. Kiến thức học được

- **Khái niệm mới nhất:** 
  - Kỹ thuật **Chunk Enrichment** (làm giàu dữ liệu chunk trước khi embed bằng LLM: summarize, sinh câu hỏi giả định HyQA, prepend ngữ cảnh tài liệu cha).
  - Thuật toán **Reciprocal Rank Fusion (RRF)** kết hợp thứ hạng tìm kiếm từ khóa truyền thống (BM25) và vector tương đồng (Dense Search).
- **Điều bất ngờ nhất:** 
  - Mô hình Cross-Encoder (`bge-reranker-v2-m3`) mặc dù tốn tài nguyên tính toán hơn nhiều so với Bi-Encoder thông thường nhưng lại cải thiện độ chính xác tìm kiếm vượt trội nhờ phân tích tương tác chéo giữa câu hỏi và tài liệu.
  - Có thể chạy đánh giá RAGAS hoàn toàn miễn phí/offline về phần embeddings bằng cách tích hợp LangChain `HuggingFaceEmbeddings` cục bộ thay vì gọi API trả phí của OpenAI.
- **Kết nối với bài giảng (slide nào):**
  - Advanced Chunking & Parent-Child Retrieval: Slide 15-18.
  - Hybrid Search & Fusion (RRF): Slide 22-25.
  - Reranking Models & Latency Tradeoffs: Slide 28-30.
  - RAGAS Metrics Framework: Slide 32-35.

---

## 3. Khó khăn & Cách giải quyết

- **Khó khăn lớn nhất:**
  - Lỗi sập tiến trình Python (Segmentation Fault) trên Windows chạy Python 3.13 khi import NumPy. Do NumPy 1.26.x không có wheel prebuilt cho Python 3.13 trên Windows, dẫn đến việc pip tự build từ source qua MinGW-w64 tạo ra binary lỗi.
  - Lỗi Rate Limit (429) và cạn credit (402) khi dùng OpenRouter API ở tài khoản miễn phí khi thực hiện hàng trăm cuộc gọi liên tục trong bước làm giàu chunk.
- **Cách giải quyết:**
  - Hạ cấp môi trường ảo xuống Python 3.12 để tải trực tiếp bản NumPy precompiled chính thức (MSVC) từ PyPI, giải quyết triệt để lỗi crash.
  - Cấu hình OpenRouter sang model `openrouter/free` và triển khai cơ chế retry với exponential backoff (ngủ tăng dần khi gặp lỗi 429) giúp pipeline hoàn thành trơn tru.
- **Thời gian debug:** ~1.5 giờ.

---

## 4. Nếu làm lại

- **Sẽ làm khác điều gì:** 
  - Lưu cache vector embeddings của các chunks đã tạo để tránh việc mã hóa lại toàn bộ khi chạy lại pipeline, giúp tiết kiệm chi phí API và rút ngắn thời gian chạy.
  - Tích hợp thêm bộ lọc metadata filter ở mức truy vấn Qdrant để lọc chính xác phiên bản chính sách năm v2024 trước khi thực hiện tìm kiếm hybrid.
- **Module nào muốn thử tiếp:** 
  - Muốn thử nghiệm mở rộng M5 bằng các kỹ thuật GraphRAG hoặc áp dụng Agentic Chunking để tăng tính cấu trúc cho tài liệu phức tạp.

---

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) |
|----------|---------------|
| Hiểu bài giảng | 5/5 |
| Code quality | 5/5 |
| Teamwork | 5/5 |
| Problem solving | 5/5 |
