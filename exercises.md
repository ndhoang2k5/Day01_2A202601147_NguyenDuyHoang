# K3 — Ngày 1: Bài Tập & Phản Ánh
## Khám Phá LLM API | Phiếu Thực Hành

**Thời lượng:** 9h00–13h00
**Cách làm:** Trả lời từng câu ngay sau khi hoàn thành block tương ứng —
đừng để dồn hết về cuối buổi. Thay placeholder câu trả lời bằng nội dung
thật (chấm tự động sẽ đếm số câu đã trả lời).

---

## Block 1 — API Cơ Bản (trả lời sau Checkpoint 1)

### Câu 1.1 — Độ nhạy của temperature
Gọi `call_openai` với temperature 0.0, 0.5, 1.0 và 1.5 dùng prompt
**"Hãy kể cho tôi một sự thật thú vị về Việt Nam."**

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> Temperature càng thấp (0.0–0.5) thì câu trả lời càng ổn định, ít sáng tạo, thường lặp lại cùng kiểu sự thật. Khi tăng lên 1.0–1.5, model đa dạng hơn về chủ đề và cách diễn đạt, đôi khi dài và “tự do” hơn nhưng cũng dễ lệch hoặc kém chính xác.

### Câu 1.2 — Chọn temperature cho sản phẩm
**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> Tôi sẽ đặt khoảng 0.2–0.4. Chatbot hỗ trợ cần trả lời chính xác, nhất quán theo chính sách/FAQ, tránh bịa thông tin; temperature thấp giúp giảm biến thiên và dễ kiểm soát chất lượng hơn.

### Câu 1.3 — Đánh đổi chi phí
Kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người gọi API 3 lần,
mỗi lần trung bình ~350 token đầu ra.

**Ước tính GPT-4o đắt hơn GPT-4o-mini bao nhiêu lần cho workload này? Nêu một
trường hợp GPT-4o xứng đáng với chi phí và một trường hợp nên dùng mini:**
> Tổng output = 10.000 × 3 × 350 = 10.500.000 token/ngày. Theo bảng giá lab (USD/1K token output): gpt-4o = 0.010, gpt-4o-mini = 0.0006 → tỷ lệ ≈ 0.010/0.0006 = **16,7 lần**. Nên dùng mini cho FAQ đơn giản, tóm tắt ngắn, phân loại/ý định người dùng — lưu lượng lớn mà yêu cầu chất lượng vừa phải.

---

## Block 2 — System Prompt & Token (trả lời sau Checkpoint 2)

### Câu 2.1 — Sức mạnh của persona
Gọi `chat_with_system_prompt` hai lần với cùng câu hỏi
**"Giải thích blockchain là gì?"** nhưng hai system prompt khác nhau:
- "Bạn là giáo viên tiểu học, giải thích thật đơn giản cho trẻ 8 tuổi."
- "Bạn là chuyên gia tài chính, trả lời chuyên sâu bằng thuật ngữ kỹ thuật."

**Hai phản hồi khác nhau như thế nào (độ dài, từ vựng, ví dụ)? System prompt
ảnh hưởng đến hành vi model ra sao?** (3–4 câu)
> Persona giáo viên tiểu học thường trả lời ngắn–vừa, từ ngữ đời thường, dùng ví dụ gần gũi. Persona chuyên gia tài chính dài hơn, dùng thuật ngữ như ledger, consensus, hash, smart contract và giải thích cơ chế kỹ thuật. System prompt đóng vai “đạo diễn”: cùng một user prompt nhưng model đổi giọng văn, độ sâu và kiểu ví dụ theo chỉ dẫn. Điều này cho thấy persona là cách rẻ và hiệu quả để định hình hành vi sản phẩm mà không cần fine-tune.

### Câu 2.2 — tiktoken vs đếm từ
Chọn một đoạn văn tiếng Việt ~100 từ. So sánh số token theo `count_tokens`
(tiktoken) với ước lượng `số từ / 0.75` mà Part 1 đã dùng.

**Hai con số chênh nhau bao nhiêu phần trăm? Vì sao tiếng Việt thường tốn
nhiều token hơn tiếng Anh cùng độ dài?**
> Đoạn ~104 từ: ước lượng `số từ / 0.75` ≈ 139 token, `count_tokens` (tiktoken) ≈ 133 token → chênh khoảng **4%** trên mẫu này. Tiếng Việt thường tốn nhiều token hơn tiếng Anh cùng độ dài vì bộ mã hóa BPE của OpenAI được tối ưu nhiều cho tiếng Anh; dấu thanh và âm tiết tiếng Việt hay bị tách thành nhiều subword, nên cùng số ký tự/từ vẫn có thể tốn token hơn.

---

## Block 3 — Streaming & Độ Bền (trả lời sau Checkpoint 3)

### Câu 3.1 — Trải nghiệm người dùng với streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì
non-streaming lại phù hợp hơn?** (1 đoạn văn)
> Streaming quan trọng nhất ở chatbot/trợ lý tương tác: người dùng thấy chữ hiện dần nên cảm giác phản hồi nhanh, giảm cảm giác chờ dù tổng thời gian sinh vẫn tương đương. Non-streaming phù hợp hơn khi cần câu trả lời trọn vẹn trước khi xử lý tiếp (ví dụ parse JSON, chấm điểm, sinh code rồi chạy), hoặc khi phản hồi rất ngắn và chi phí/độ phức tạp của stream không đáng.

### Câu 3.2 — Vì sao backoff theo cấp số nhân?
**So với delay cố định (ví dụ luôn chờ 1 giây), exponential backoff có lợi
thế gì khi API bị quá tải? Điều gì xảy ra nếu hàng nghìn client cùng retry
với delay cố định giống nhau?**
> Exponential backoff tăng thời gian chờ sau mỗi lần fail (0.1 → 0.2 → 0.4…) nên giảm áp lực lên server đang nghẽn và tăng cơ hội thành công ở lần thử sau. Nếu hàng nghìn client cùng retry với delay cố định giống nhau, chúng sẽ đồng loạt đánh lại API cùng một lúc (thundering herd), làm quá tải kéo dài hoặc tái diễn lỗi 429.

---

## Block 4 — Mini-Project (trả lời sau Checkpoint 4)

### Câu 4.1 — Thiết kế persona
**Bạn chọn persona gì cho trợ lý của mình? Viết lại system prompt đó và giải
thích 1–2 lựa chọn từ ngữ quan trọng trong prompt (ví dụ: vì sao yêu cầu
"trả lời ngắn gọn", vì sao chỉ định ngôn ngữ...):**
> Persona: trợ giảng khóa AI thân thiện. System prompt: “Bạn là trợ giảng thân thiện của khóa AI Practical Competency. Trả lời ngắn gọn bằng tiếng Việt, ưu tiên ví dụ thực tế, nếu không chắc thì nói rõ giới hạn.” “Trả lời ngắn gọn” giúp giảm token/chi phí và phù hợp CLI. “Bằng tiếng Việt” cố định ngôn ngữ đầu ra để UX nhất quán với học viên Việt Nam.

### Câu 4.2 — Hạn chế & cải thiện
**Trợ lý của bạn hiện có hạn chế lớn nhất là gì (ví dụ: history chỉ 3 lượt,
không có bộ nhớ dài hạn, không kiểm duyệt nội dung...)? Đề xuất một cải
thiện cụ thể và mô tả ngắn cách triển khai:**
> Hạn chế lớn nhất: history chỉ giữ 3 lượt cuối nên ngữ cảnh dài bị mất. Cải thiện: thêm tóm tắt hội thoại cũ — khi cắt history, gọi model một lần để tóm tắt các message bị bỏ thành một message system/assistant ngắn, rồi luôn prepend bản tóm tắt đó vào messages cùng 3 lượt gần nhất.

---

## Danh Sách Kiểm Tra Nộp Bài

- [ ] `python grade.py` — xem điểm tự động, mục tiêu ≥ 75/100
- [ ] Cả 4 checkpoint pytest đều pass
- [ ] Tất cả 9 câu trong file này đã được trả lời
- [ ] Đã copy bài làm vào folder `solution/` và zip theo hướng dẫn README
