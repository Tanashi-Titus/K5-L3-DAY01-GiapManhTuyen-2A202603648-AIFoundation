# Báo cáo cá nhân — Day 01

**MSSV:** 2A202603648  
**Họ tên:** Giáp Mạnh Tuyên  
**SEED:** 6217

## 1. Bằng chứng

- [x] `day01_lab.ipynb`
- [x] `lab1_report.json`
- [x] `hand_calc.jpg`

Đã hoàn thành các bài thực hành, kiểm tra tự động và các thí nghiệm chính.

## 2. Kết quả chính

| Mô hình | F1 Test | F1 SUB | p95 |
|---|---:|---:|---:|
| BoW + LogReg | 0.897 | 0.889 | 0.064 ms |
| BoW + MLP | 0.940 | 0.947 | 0.333 ms |
| MiniLM + kNN | 0.900 | 1.000 | 20.843 ms |
| MiniLM + LogReg | 0.950 | 0.974 | 6.842 ms |
| MiniLM + MLP | **0.955** | 0.974 | 10.414 ms |
| GPT-OSS 20B Zero-shot | — | 0.864 | 2351.650 ms |
| GPT-OSS 120B Zero-shot | — | 0.889 | 2319.914 ms |
| GPT-OSS 20B Few-shot | — | 0.905 | 2431.338 ms |
MiniLM + MLP đạt F1 cao nhất trên tập test với 0.955. MiniLM + kNN đạt F1 = 1.0 trên SUB nhưng F1 test chỉ đạt 0.900.
Các mô hình BoW và MiniLM đều đạt SLA dưới 50 ms, trong khi LLM có độ trễ cao hơn nhiều.
Chi phí API Groq chưa được ghi nhận chính xác trong kết quả đo.

## 3. Dự đoán và thực tế
**Thí nghiệm 2.B — Bỏ ReLU**
Ban đầu em dự đoán F1 sẽ giảm khi bỏ ReLU vì mạng mất khả năng biểu diễn phi tuyến. Tuy nhiên, F1 vẫn đạt 0.940, bằng với MLP có ReLU. Điều này cho thấy ReLU chưa tạo ra khác biệt rõ rệt trên dữ liệu hiện tại
**Thí nghiệm 3.B — Tin nhắn ngụy trang**
Em dự đoán MiniLM + LogReg sẽ phân loại đúng nhiều hơn BoW nhờ embedding ngữ nghĩa. Kết quả cả hai đều đúng 5/6 tin, cùng bỏ sót một tin spam. Như vậy, embedding không phải lúc nào cũng xử lý tốt hơn BoW trong từng trường hợp cụ thể
**Stress Test — 50 case khó**
Em dự đoán các mô hình MiniLM sẽ xử lý tốt phần lớn case khó. Tuy nhiên, MiniLM + LogReg và MLP chỉ đạt Accuracy khoảng 58%, thấp hơn dự đoán. Ngược lại, LLM xử lý các trường hợp spam ngụy trang tốt hơn
Điều này cho thấy chất lượng trên tập test thông thường chưa phản ánh đầy đủ khả năng tổng quát hóa

## 4. Phân tích 50 case khó
BoW + LogReg và BoW + MLP có mức giảm F1 lớn nhất, đều khoảng 0.519 so với tập test thông thường.
| Mô hình | F1 Test | F1 Hard |
|---|---:|---:|
| BoW + LogReg | 0.897 | 0.378 |
| BoW + MLP | 0.940 | 0.421 |
| MiniLM + kNN | 0.900 | 0.764 |
| MiniLM + LogReg | 0.950 | 0.553 |
| MiniLM + MLP | 0.955 | 0.553 |

BoW gặp khó khăn với những tin nhắn né từ khóa và spam giọng tự nhiên. MiniLM có khả năng biểu diễn ngữ nghĩa nhưng vẫn dễ nhầm các tin quảng cáo được viết giống hội thoại bình thường.

**Case #2 — False Positive**
"FREE pizza in the office kitchen right now, come quick before it's gone"
Đây là tin nhắn đồng nghiệp thông báo đồ ăn miễn phí, nhưng mô hình nhận nhầm thành spam vì chứa các từ FREE và lời thúc giục

**Case #47 — False Positive**
Tin nhắn kể lại việc nhận được thông báo trúng thưởng và số điện thoại lạ
Người gửi đang nói về spam chứ không thực sự quảng cáo, nhưng mô hình vẫn phân loại nhầm do nội dung chứa nhiều từ khóa spam
Những lỗi này cho thấy cần quan tâm đến False Positive và ngữ cảnh của tin nhắn khi triển khai thực tế

## 5. Thử thách cá nhân
**Đề #2 — Thay đổi kích thước Vocabulary**
Thực hiện huấn luyện Logistic Regression với ba kích thước vocabulary là 100, 2.000 và 5.000 từ.
Giữ nguyên tập train/test, learning rate = 0.5 và 300 epochs.
| Vocabulary | Precision | Recall | F1 |
|---|---:|---:|---:|
| 100 | 0.962 | 0.676 | 0.794 |
| 2.000 | 0.989 | 0.820 | 0.897 |
| 5.000 | 0.989 | 0.820 | 0.897 |

Với 100 từ, mô hình thiếu nhiều đặc trưng nên Recall thấp, kéo theo F1 chỉ đạt 0.794.
Tăng lên 2.000 từ giúp mô hình nhận diện thêm những từ quan trọng, cải thiện F1 lên 0.897. Tuy nhiên, tăng lên 5.000 từ không mang lại cải thiện rõ rệt.
Em chọn vocabulary 2.000 từ vì đạt kết quả tương đương 5.000 từ nhưng sử dụng ít đặc trưng hơn

## 6. Phương án sản phẩm thật
**Lựa chọn mô hình**
Em chọn MiniLM + MLP vì đạt F1 test cao nhất là 0.955 và p95 = 10.414 ms, đáp ứng yêu cầu SLA dưới 50 ms
Với 10 triệu SMS/ngày, hệ thống cần xử lý trung bình khoảng 116 tin/giây. Vì vậy, ngoài F1 cần quan tâm đến tốc độ, chi phí và khả năng chịu tải thực tế
LLM có thể phân loại tốt những trường hợp khó nhưng độ trễ khoảng 2.3–2.4 giây, không phù hợp để xử lý đồng bộ toàn bộ tin nhắn

**Thiết kế Router**
Em đề xuất kết hợp ba phương pháp:
1. BoW + MLP xử lý nhanh các tin có độ tin cậy cao.
2. MiniLM + MLP kiểm tra những tin chưa chắc chắn.
3. LLM hỗ trợ phân loại các trường hợp khó hoặc khi hai mô hình bất đồng.
Ngưỡng chuyển tiếp cần được điều chỉnh trên tập validation
Cách kết hợp này giúp tận dụng tốc độ của BoW, chất lượng embedding và khả năng hiểu ngữ cảnh của LLM, đồng thời hạn chế chi phí API

## 7. AI log và điều đã học
### AI log
| Phần | Nội dung hỗ trợ AI | Kiểm chứng |
|---|---|---|
| Phần 0–1 | Giải thích Data Leakage, BoW, Logistic Regression và gợi ý code | Chạy kiểm tra, đối chiếu các chỉ số |
| Phần 2 | Hỗ trợ code PyTorch, Gradient Descent và Overfitting | Kiểm tra mô hình, phân tích Train/Val Loss |
| Phần 3 | Gợi ý triển khai Cosine, kNN và giải thích embedding | Chạy kiểm tra, so sánh F1 |
| Phần 4 | Hỗ trợ API, Prompt Engineering và xử lý phản hồi | Chạy mock test, sửa lỗi và kiểm tra kết quả |
| Phần 5 | Phân tích Pareto, Stress Test và các trường hợp sai | Đối chiếu kết quả thực nghiệm |
| Thử thách cá nhân | Tham khảo code thay đổi vocabulary | Thực nghiệm với ba kích thước từ điển |
| Tính tay | Tham khảo công thức và cách tính | Kiểm tra đáp án và trình bày lại trên giấy |

### Điều đã học
Qua bài Lab, em hiểu rõ hơn sự khác nhau giữa ML, DL, Embedding và LLM, đặc biệt là cách tạo đặc trưng và khả năng học của từng phương pháp
Em nhận ra F1 cao trên tập test chưa chắc phản ánh đúng chất lượng khi gặp dữ liệu khó. Khi xây dựng sản phẩm AI, cần cân bằng giữa độ chính xác, tốc độ, chi phí và khả năng tổng quát hóa
Ngoài ra, các dự đoán ban đầu có thể khác kết quả thực tế, nên việc chạy thí nghiệm và phân tích lỗi rất quan trọng