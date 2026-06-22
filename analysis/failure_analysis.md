# Failure Analysis — Lab 18: Production RAG

**Nhóm:** Nhóm 1  
**Thành viên:** Nguyễn Doàn Gia Tuấn (M1-M5)

---

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.7417 | 0.7867 | +0.0450 |
| Answer Relevancy | 0.1495 | 0.1626 | +0.0131 |
| Context Precision | 0.9167 | 0.9083 | -0.0083 |
| Context Recall | 0.7667 | 0.8417 | +0.0750 |

---

## Bottom-5 Failures

### #1
- **Question:** Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào?
- **Expected:** Theo chính sách v2024: 15 ngày cơ bản + 3 ngày thâm niên (9÷3=3) = 18 ngày phép. Lương Senior (P3-P4): 20-35 triệu VNĐ/tháng.
- **Got:** 18 ngày phép nhưng lương của Senior lại được ước lượng sai khoảng do thông tin mập mờ giữa các phiên bản hoặc mô tả lương bị nhầm với cấp bậc khác.
- **Worst metric:** faithfulness (0.0)
- **Error Tree:** Output sai (Lương Senior bị tính nhầm) -> Context đúng (Cả hai văn bản lương và phép đều được retrieve đúng) -> Query OK (Query rõ ràng)
- **Root cause:** Context chứa cả chính sách cũ (v2023) và mới (v2024) khiến LLM bị bối rối và tính toán nhầm số ngày phép hoặc khoảng lương.
- **Suggested fix:** Cần thêm bộ lọc metadata để chỉ truy cập tài liệu v2024 hoặc tinh chỉnh prompt để yêu cầu LLM bỏ qua các chính sách cũ đã hết hiệu lực.

### #2
- **Question:** Nhân viên thử việc có được hưởng bảo hiểm sức khỏe PVI không?
- **Expected:** KHÔNG. Nhân viên thử việc chưa được hưởng gói bảo hiểm sức khỏe PVI. Chỉ được tham gia bảo hiểm xã hội bắt buộc.
- **Got:** LLM trả lời là "Không tìm thấy thông tin" hoặc nhầm lẫn rằng nhân viên thử việc được hưởng bảo hiểm sức khỏe PVI do văn bản nói chung về quyền lợi bảo hiểm.
- **Worst metric:** faithfulness (0.0)
- **Error Tree:** Output sai -> Context thiếu (Chunk nói về thử việc và bảo hiểm sức khỏe không được liên kết trực tiếp) -> Query OK
- **Root cause:** Thiếu liên kết ngữ cảnh giữa chế độ bảo hiểm và đối tượng áp dụng (thử việc vs chính thức).
- **Suggested fix:** Sử dụng contextual prepend (làm giàu chunk) để đưa thông tin "áp dụng cho nhân viên chính thức" vào đầu tất cả các chunks về bảo hiểm PVI.

### #3
- **Question:** Nhân viên được nghỉ bao nhiêu ngày khi kết hôn?
- **Expected:** Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn, không trừ vào phép năm.
- **Got:** LLM đưa ra câu trả lời sai lệch về số ngày (ví dụ 1 ngày hoặc 5 ngày) do nhầm sang trường hợp kết hôn của con hoặc người thân khác.
- **Worst metric:** faithfulness (0.0)
- **Error Tree:** Output sai -> Context đúng -> Query OK
- **Root cause:** LLM đọc lướt qua bảng/danh sách các loại nghỉ phép và nhầm lẫn dòng "bản thân kết hôn" (3 ngày) với "con kết hôn" (1 ngày).
- **Suggested fix:** Tối ưu hóa prompt bằng kỹ thuật CoT (Chain of Thought) yêu cầu LLM xác định chính xác chủ thể của hành động trước khi tra cứu số liệu.

### #4
- **Question:** Nhân viên được tài trợ khóa học 25 triệu, nghỉ việc sau 8 tháng hoàn thành khóa học. Phải hoàn trả bao nhiêu?
- **Expected:** Nhân viên phải cam kết làm việc ít nhất 1 năm sau khi hoàn thành khóa học. Nghỉ sau 8 tháng là trước hạn cam kết, phải hoàn trả 100% chi phí tức 25.000.000 VNĐ.
- **Got:** LLM trả lời chung chung về quy định bồi hoàn (phải làm việc 1 năm) mà không trả lời trực tiếp con số cụ thể 25 triệu đồng.
- **Worst metric:** answer_relevancy (0.0278)
- **Error Tree:** Output không liên quan trực tiếp -> Context đúng -> Query OK
- **Root cause:** Prompt template chưa bắt buộc LLM phải trả lời trực diện vào câu hỏi số lượng (bao nhiêu) trước khi giải thích lý do.
- **Suggested fix:** Cải thiện Prompt Template để yêu cầu LLM luôn trả lời trực tiếp câu hỏi (ví dụ: "Số tiền phải hoàn trả là X") rồi mới diễn giải.

### #5
- **Question:** Nhân viên tạm ứng 15 triệu, sau 20 ngày mới thanh toán. Bị phạt bao nhiêu?
- **Expected:** Thời hạn thanh toán là 15 ngày. Quá hạn 5 ngày, bị tính phí 2%/tháng trên 15.000.000 VNĐ = 300.000 VNĐ/tháng (tính pro-rata khoảng 50.000 VNĐ cho 5 ngày).
- **Got:** LLM chỉ liệt kê quy chế tạm ứng và hạn thanh toán là 15 ngày mà không thực hiện phép tính phạt 2%/tháng cho phần quá hạn.
- **Worst metric:** answer_relevancy (0.0495)
- **Error Tree:** Output thiếu tính toán -> Context đúng -> Query OK
- **Root cause:** LLM không tự thực hiện tính toán số học trên các con số trong context trừ khi được yêu cầu rõ ràng.
- **Suggested fix:** Cấu hình prompt kích hoạt khả năng tính toán số học từng bước (Chain of Thought Math) đối với các câu hỏi liên quan đến số tiền phạt hoặc chi phí.

---

## Case Study (cho presentation)

**Question chọn phân tích:**
"Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào?"

**Error Tree walkthrough:**
1. **Output đúng?** -> Sai. Số ngày phép bị tính nhầm hoặc khoảng lương bị lấy từ chính sách cũ v2023.
2. **Context đúng?** -> Đúng. Cả hai tài liệu về chính sách nghỉ phép và lương đều được retrieve về trong top-k.
3. **Query rewrite OK?** -> Đúng. Câu hỏi gốc rõ ràng và không cần viết lại.
4. **Fix ở bước:** Tách biệt tài liệu chính sách cũ và mới bằng metadata filter để loại bỏ hoàn toàn các thông tin v2023 gây nhiễu cho LLM.

**Nếu có thêm 1 giờ, sẽ optimize:**
- Tích hợp bộ lọc metadata thông minh (ví dụ: chỉ lọc tài liệu có tag `current` hoặc `v2024`).
- Tinh chỉnh Prompt Template để tăng cường khả năng suy luận logic và tính toán số học của LLM đối với các câu hỏi bồi hoàn và thâm niên nghỉ phép.
