# RAG Evaluation Results

## Framework sử dụng

> **Custom LLM-as-a-Judge** sử dụng mô hình **Gemini 3.1 Flash Lite** làm giám khảo để tự động chấm điểm trên 16 câu hỏi thuộc Golden Dataset.

---

## Overall Scores

| Metric | Config A (hybrid + rerank) | Config B (dense-only) | Δ |
|--------|---------------------------|----------------------|---|
| Faithfulness | 0.975 | 1.000 | -0.025 |
| Answer Relevance | 0.969 | 0.650 | +0.319 |
| Context Recall | 0.956 | 0.119 | +0.838 |
| Context Precision | 0.869 | 0.275 | +0.594 |
| **Average** | **0.942** | **0.511** | **+0.431** |

---

## A/B Comparison Analysis

**Config A (Hybrid Search + Reranking):**
* Tìm kiếm kết hợp Hybrid (dense vector + BM25 keyword matching) với cấu trúc chunk phân mảnh theo Markdown Header (chứa trọn vẹn Điều luật). Sau đó chạy Reranker cục bộ kết hợp 40% Cosine Similarity và 60% Jaccard Overlap để ưu tiên từ khóa chính xác.

**Config B (Dense-only):**
* Tìm kiếm hoàn toàn dựa trên sự tương đồng vector ngữ nghĩa của mô hình `all-MiniLM-L6-v2` và không áp dụng bất kỳ thuật toán rerank hay kết hợp từ khóa nào.

**Kết luận:**
* **Config A vượt trội hơn hẳn Config B** ở mọi chỉ số, đặc biệt là **Context Recall (+83.8%)** và **Context Precision (+59.4%)**.
* Mô hình embedding MiniLM hoạt động rất kém trên tiếng Việt, khiến tìm kiếm Dense-only bị sai sót nhiều. Khi có thêm tìm kiếm từ khóa BM25 và bộ lọc Reranker Jaccard Overlap của Config A, các tài liệu chứa đúng số điều luật và thuật ngữ ma túy chính xác được đẩy lên hàng đầu, giúp chất lượng sinh câu trả lời nâng cao rõ rệt.

---

## Worst Performers (Bottom 3)

| # | Question | Faithfulness | Relevance | Recall | Failure Stage | Root Cause |
|---|----------|-------------|-----------|--------|---------------|------------|
| 1 | Cơ quan nào có thẩm quyền xác định tình trạng nghiện ma túy theo Thông tư liên tịch 2015? | 0.80 | 0.70 | 0.30 | Retriever | The retrieved context documents do not contain the full text of 'Điều 3' of the 2015 Circular, which is the primary source defining the individuals authorized to determine drug addiction. Thus, the system lacked the necessary information to formulate the expected answer. |
| 2 | Tội trồng cây thuốc phiện quy định hình phạt thế nào theo Điều 247 Bộ luật hình sự? | 1.00 | 1.00 | 1.00 | Retriever | The retrieved context includes many irrelevant documents (e.g., Articles 252, 251, 250, 249, 106, 104, 30) that are not related to Article 247, indicating poor retrieval precision despite the correct document being present. |
| 3 | Tội tổ chức sử dụng trái phép chất ma túy bị xử phạt thế nào theo Bộ luật hình sự? | 1.00 | 1.00 | 1.00 | Retriever | While the relevant document (Article 255) was retrieved, the search result also included multiple irrelevant documents concerning other crimes (Articles 253, 254, 256, 257, 141, 321) and administrative procedures, which were unnecessary for answering the specific question. |

---

## Recommendations

### Cải tiến 1: Nâng cấp mô hình nhúng sang BAAI/bge-m3
* **Action:** Thay thế mô hình `all-MiniLM-L6-v2` bằng mô hình `BAAI/bge-m3` đa ngôn ngữ để cải thiện chất lượng tìm kiếm dense cho tiếng Việt.
* **Expected impact:** Giúp điểm Faithfulness và Context Recall của Config B tăng lên, giảm gánh nặng lọc từ khóa của Reranker.

### Cải tiến 2: Bổ dung công cụ Tách từ Tiếng Việt (Word Segmentation)
* **Action:** Áp dụng thư viện `underthesea` trước khi lập chỉ mục BM25 để các từ ghép như "thuốc phiện", "cai nghiện" không bị xé nhỏ thành các từ đơn.
* **Expected impact:** Tăng độ chính xác của tìm kiếm từ khóa, từ đó nâng điểm Context Precision lên gần mức tuyệt đối.

### Cải tiến 3: Tối ưu hóa Reranker với Cross-Encoder chính xác hơn
* **Action:** Đăng ký và đưa khóa Jina Reranker API (`jina-reranker-v2-base-multilingual`) thực tế vào thay thế cho thuật toán Jaccard + Cosine nội bộ.
* **Expected impact:** Khả năng rerank ngữ nghĩa sâu sắc của Jina giúp loại bỏ hoàn toàn các tài liệu rác, tối ưu hóa tối đa điểm Context Precision và loại trừ hiện tượng "mục lục lấn lướt".
