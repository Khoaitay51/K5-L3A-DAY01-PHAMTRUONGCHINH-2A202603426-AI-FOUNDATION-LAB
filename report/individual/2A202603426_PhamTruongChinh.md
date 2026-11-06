# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác bằng ML, DL, Embedding và LLM

> Đổi tên file này thành `MSSV_HoTen.md` của bạn, ví dụ `2A20260000_NguyenVanAn.md`, rồi điền từng mục. Xoá các dòng hướng dẫn in nghiêng khi xong.

- **MSSV:*2A202603426*
- **Họ và tên:*Phạm Trường Chinh*
- **SEED** (in ở ô đầu notebook):3110

## 1. Bằng chứng

| Tệp | Có? |
| --- | --- |
| [`day01_lab.ipynb`](day01_lab.ipynb) — notebook đã chạy, còn output | x |
| [`lab1_report.json`](lab1_report.json) — do ô "Báo cáo cuối" sinh ra | x |
| [`hand_calc.jpg`](hand_calc.jpg) — ảnh tính tay có các bước | o |

## 2. Kết quả chính

*Chép bảng so sánh cuối Phần 5 (F1 test, F1 SUB 40, p95 ms/tin, USD/1k tin) của chính bạn.*

                                     f1  f1_SUB
1. BoW + LogReg (tự code)        0.911   0.857
2. BoW + MLP (deep learning)     0.970   1.000
3a. Encoder (MiniLM) + kNN       0.917   1.000
3b. Encoder (MiniLM) + LogReg    0.971   1.000
3c. Encoder (MiniLM) + MLP head  0.971   1.000
4a. LLM nhỏ zero-shot            0.976   0.976
4b. LLM lớn zero-shot            1.000   1.000
4c. LLM nhỏ few-shot             0.976   0.976

## 3. Dự đoán và thực tế

*Chọn 3 dự đoán `predict(...)` lệch nhiều nhất so với kết quả. Mỗi cái: bạn đoán gì, ra gì, vì sao lệch.*
  tradeoff:
Đoán: LLM sẽ nằm gần vạch SLA 50 ms.
Ra: LLM chậm hơn rất nhiều (p95 từ vài trăm ms đến vài giây), nằm ngoài hoàn toàn ngưỡng SLA.
Vì sao lệch: Chưa tính tới độ trễ mạng khi gọi API và tốc độ sinh token của LLM chậm hơn nhiều so với tính ma trận cục bộ.
  llm_fewshot:
Đoán: Thêm ví dụ mẫu thì F1 sẽ tăng lên.
Ra: F1 giữ nguyên không đổi so với zero-shot, trong khi chi phí token tăng gần gấp 3.
Vì sao lệch: Bài toán phân loại spam/ham đơn giản, zero-shot đã đủ rõ ràng nên vài ví dụ không giúp mô hình học thêm được gì mới.
  hard_eval:
Đoán: Điểm các model sẽ tụt xuống dưới 60 và LLM dễ bị lừa bởi prompt injection.
Ra: LLM xử lý rất tốt và giữ F1 ở mức cao, trong khi BoW và MLP tụt rất sâu.
Vì sao lệch: Đánh giá thấp khả năng hiểu ngữ cảnh của LLM và đánh giá quá cao BoW (vốn chỉ đọc mặt chữ).

## 4. Phân tích 50 case khó

*Model nào tụt nhiều nhất, ở nhóm case nào? Nêu 2 case cụ thể và cơ chế khiến model sai.*
Model tụt nhiều nhất: BoW + Logistic Regression và MLP. 
Tụt nặng nhất ở nhóm ham bẫy (tin thường chứa từ giống spam) và spam thao túng tâm lý (né từ khóa quen thuộc).
2 case cụ thể:
Case ham trích dẫn tin trúng thưởng: Tin nhắn có chứa các từ "trúng thưởng", "link" nên BoW cộng dồn điểm spam và đoán nhầm thành spam.
Case spam giả người quen nhờ chuyển tiền: Toàn từ ngữ sinh hoạt thông thường, không chứa từ khóa trong từ điển BoW nên bị đoán nhầm thành ham.

## 5. Thử thách cá nhân

*Đề theo SEED của bạn, cách làm, kết quả, giải thích.*
Em chưa làm ạ :(

## 6. Phương án cho sản phẩm thật

*Với yêu cầu p95 < 50 ms/tin cho 10 triệu SMS/ngày: chọn phương án nào, dẫn số liệu nào, chấp nhận đánh đổi gì.*
Lựa chọn: Embedding (MiniLM) + Logistic Regression.
Lý do: Đạt độ trễ p95 an toàn dưới 50 ms, tốc độ xử lý hàng nghìn tin/giây đủ đáp ứng 10 triệu tin/ngày, F1 cao hơn BoW và không tốn chi phí gọi API như LLM.
Đánh đổi: Chấp nhận khả năng xử lý kém hơn LLM ở các case lừa đảo biến tướng tinh vi, và tốn thêm một ít RAM để nạp model so với BoW.


## 7. AI log và điều đã học

*Mỗi lần dùng AI: hỏi gì, nhận gì, bạn sửa gì. Sau đó 2–3 câu: điều bạn hiểu ra hôm nay mà trước đó chưa hiểu.*
Phần 1–3: Chỉ hỏi giải thích lý thuyết về đạo hàm sigmoid, vòng lặp train PyTorch và cosine similarity; tự kiểm chứng công thức rồi tự viết code.
Phần 4: Nhờ AI gợi ý format prompt và regex parse nhãn, tự viết thêm logic chuẩn hóa chuỗi để tránh lỗi định dạng.
Phần 5: Nhờ AI gợi ý ý tưởng các kịch bản lừa đảo tinh vi để tự viết vào tập test case cá nhân.
Điều hiểu ra: Trước đây em nghĩ LLM luôn là lựa chọn tốt nhất cho mọi bài toán. Hôm nay em hiểu rằng mô hình nhỏ chạy cục bộ mới là giải pháp tối ưu về độ trễ và chi phí trong hệ thống lớn. Thiết kế mô hình lai (dùng mô hình nhẹ lọc trước, chỉ đẩy ca khó cho LLM) là hướng đi hợp lý nhất cho bài toán thực tế.
