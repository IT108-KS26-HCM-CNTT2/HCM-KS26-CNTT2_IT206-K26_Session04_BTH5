
## 1. Sơ đồ mô tả chuỗi Prompt (Prompt Chaining Flow)

Để tránh tình trạng AI viết bài quảng cáo bị rời rạc, không trúng tâm lý sinh viên năm nhất, em chia quy trình thành **4 Prompt chạy nối tiếp nhau**. Dữ liệu đầu ra của bước trước được đóng gói bằng các thẻ định danh (`[THONG_TIN_GOC]`, `[CHAN_DUNG_KH]`, `[THONG_DIEP_CHINH]`, `[KE_HOACH_7_NGAY]`) và truyền bắt buộc sang bước sau để khóa chặt thông tin khóa học.

```
[THÔNG TIN KHÓA HỌC GỐC]
(SV năm nhất, 18-20 tuổi, 8 tuần, 100 lượt tư vấn, FB & TikTok)
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│ PROMPT 1: Phân tích chân dung khách hàng mục tiêu (SV Y1) │
└──────────────────────────┬───────────────────────────────┘
                           │ Xuất: [CHAN_DUNG_KH]
                           ▼
┌──────────────────────────────────────────────────────────┐
│ PROMPT 2: Chốt 1 Thông điệp chính & Lên lịch 7 ngày      │
└──────────────────────────┬───────────────────────────────┘
                           │ Xuất: [THONG_DIEP_CHINH] + [LICH_7_NGAY]
                           ▼
┌──────────────────────────────────────────────────────────┐
│ PROMPT 3: Viết 2 bài Facebook & 2 kịch bản TikTok (30-45s)│
└──────────────────────────┬───────────────────────────────┘
                           │ Xuất: [NOI_DUNG_CHI_TIET]
                           ▼
┌──────────────────────────────────────────────────────────┐
│ PROMPT 4: Kiểm tra tính nhất quán & Rà soát sai lệch     │
└──────────────────────────────────────────────────────────┘
```

## 2. Hệ thống 4 Prompt liên tiếp

### Prompt 1: Phân tích chân dung khách hàng mục tiêu
```
Bạn là chuyên viên nghiên cứu thị trường giáo dục. Hãy phân tích chân dung khách hàng mục tiêu cho khóa học lập trình dựa trên dữ liệu gốc dưới đây.

[THONG_TIN_GOC]:
- Khóa học: Python cơ bản dành cho người mới bắt đầu.
- Đối tượng: Sinh viên năm nhất (độ tuổi 18–20).
- Thời lượng: 8 tuần.
- Mục tiêu chiến dịch: Thu hút 100 lượt đăng ký tư vấn.
- Kênh triển khai: Facebook và TikTok.

Yêu cầu:
1. Tuyệt đối KHÔNG tự ý sửa đổi bất kỳ thông số nào trong [THONG_TIN_GOC].
2. Phân tích ngắn gọn, thực tế về: Đặc điểm tâm lý, 3 nỗi sợ lớn nhất khi học code của sinh viên năm nhất, 3 mong muốn chính, và thói quen dùng Facebook/TikTok của các bạn.

Định dạng đầu ra (chỉ xuất nội dung trong thẻ này để truyền sang bước sau):
[CHAN_DUNG_KH]
...
```

### Prompt 2: Xác định thông điệp chính & Lập kế hoạch nội dung 7 ngày
```
Bạn là trưởng nhóm lập kế hoạch truyền thông (Content Planner). Dựa vào thông tin khóa học gốc và chân dung khách hàng đã phân tích ở bước trước, hãy xác định thông điệp và lập lịch đăng bài trong 7 ngày.

Dữ liệu đầu vào bắt buộc:
[THONG_TIN_GOC]: Khóa Python cơ bản cho người mới bắt đầu | SV năm nhất (18–20 tuổi) | 8 tuần | Mục tiêu 100 lượt đăng ký tư vấn | Kênh: Facebook & TikTok.
{{OUTPUT_PROMPT_1}}

Yêu cầu:
1. Đề xuất đúng 01 Thông điệp truyền thông chính (Key Message) ngắn gọn, đánh trúng tâm lý sinh viên năm nhất.
2. Lập bảng kế hoạch nội dung trong 7 ngày (Ngày 1 đến Ngày 7) phân bổ đều cho 2 kênh Facebook và TikTok, bám sát thông điệp chính và dẫn dắt người xem đến hành động "đăng ký tư vấn".

Định dạng đầu ra bắt buộc:
[THONG_DIEP_CHINH]: "..."
[LICH_7_NGAY]: (Bảng Markdown gồm các cột: Ngày | Kênh | Chủ đề nội dung | Mục tiêu nhỏ)
```

### Prompt 3: Viết 2 bài đăng Facebook & 2 kịch bản video TikTok
```
Bạn là chuyên viên sáng tạo nội dung (Copywriter) chuyên viết cho gen Z và sinh viên. Dựa trên dữ liệu từ các bước trước, hãy sản xuất nội dung chi tiết cho chiến dịch.

Dữ liệu đầu vào bắt buộc:
[THONG_TIN_GOC]: Khóa Python cơ bản cho người mới bắt đầu | SV năm nhất (18–20 tuổi) | 8 tuần | Mục tiêu 100 lượt đăng ký tư vấn | Kênh: Facebook & TikTok.
{{OUTPUT_PROMPT_1}}
{{OUTPUT_PROMPT_2}}

Yêu cầu sản xuất:
1. Viết 02 bài đăng Facebook (Bài 1: Đánh vào nỗi đau sợ rớt môn/mất gốc code của SV năm nhất; Bài 2: Giới thiệu lộ trình 8 tuần thực chiến và kêu gọi đăng ký tư vấn).
2. Viết 02 kịch bản video TikTok có thời lượng chuẩn 30–45 giây (chia rõ cột Thời gian | Hình ảnh/Cảnh quay | Lời thoại/Text màn hình).
3. Ràng buộc: Mọi bài viết đều phải nhắc đúng thời lượng "8 tuần", đúng đối tượng "sinh viên năm nhất" và gắn chặt với [THONG_DIEP_CHINH].

Định dạng đầu ra:
[NOI_DUNG_CHI_TIET]
...
```

### Prompt 4: Kiểm tra tính nhất quán của toàn bộ chiến dịch
```
Bạn là Quản lý chất lượng chiến dịch Marketing (QA Lead). Hãy đối chiếu toàn bộ kết quả từ Bước 1, Bước 2 và Bước 3 so với [THONG_TIN_GOC] ban đầu để lập báo cáo kiểm tra tính nhất quán.

Dữ liệu kiểm tra:
- Thông tin gốc: Khóa Python cơ bản | SV năm nhất (18-20 tuổi) | 8 tuần | Mục tiêu 100 lượt đăng ký tư vấn | Kênh: Facebook, TikTok.
{{OUTPUT_PROMPT_1}}
{{OUTPUT_PROMPT_2}}
{{OUTPUT_PROMPT_3}}

Yêu cầu kiểm tra 4 tiêu chí:
1. Thông số khóa học (Có bị sai lệch độ tuổi, thời lượng 8 tuần hay kênh triển khai không?).
2. Sự liên kết giữa Chân dung khách hàng -> Thông điệp chính -> Nội dung bài viết.
3. Mức độ đáp ứng mục tiêu kêu gọi hành động (CTA hướng tới 100 lượt đăng ký tư vấn).
4. Đề xuất chỉnh sửa nhanh (nếu phát hiện điểm chưa tối ưu).
```

---

## 3. Kết quả chạy thử thực tế toàn bộ chuỗi Prompt

### 3.1. Chân dung khách hàng mục tiêu (Output Prompt 1)
`[CHAN_DUNG_KH]`
* **Thông tin nhân khẩu học:** Sinh viên năm nhất (18–20 tuổi), mới bước chân vào đại học, đang làm quen với các môn đại cương hoặc nhập môn lập trình.
* **Nỗi sợ lớn nhất:**
  1. *Sợ "mất gốc" ngay từ đầu:* Chưa từng viết dòng code nào ở cấp 3, lên đại học giảng viên dạy nhanh nên không theo kịp, sợ rớt môn.
  2. *Sợ lý thuyết khô khan:* Học toàn cú pháp trên giấy/slide, không biết viết ra ứng dụng gì thực tế.
  3. *Sợ tốn thời gian & chi phí:* Lịch học trên trường dày đặc, ngân sách sinh viên hạn hẹp nên ngại các khóa học kéo dài lê thê.
* **Mong muốn:** Có một lộ trình cầm tay chỉ việc ngắn gọn, dễ hiểu cho người chưa biết gì; tự tay làm được sản phẩm nhỏ sau khóa học để tự tin học tiếp các môn chuyên ngành.
* **Thói quen dùng mạng xã hội:** Lướt TikTok giải trí nhanh vào buổi tối (thích video ngắn 30–45s hài hước, bắt trend, đánh trúng nỗi khổ sinh viên); dùng Facebook để theo dõi các hội nhóm trường đại học, xem review khóa học và nhắn tin inbox xin tư vấn.

### 3.2. Thông điệp truyền thông chính & Lịch nội dung 7 ngày (Output Prompt 2)

`[THONG_DIEP_CHINH]`: **"8 tuần xóa mù Python từ con số 0 – Tự tin làm chủ lập trình ngay từ năm nhất!"**

`[LICH_7_NGAY]`:

| Ngày | Kênh | Chủ đề nội dung | Mục tiêu nhỏ |
| :--- | :--- | :--- | :--- |
| **Ngày 1** | TikTok | Video ngắn: "Cảm giác của SV năm nhất khi nhìn màn hình đen ngòm toàn lỗi đỏ" | Thu hút sự đồng cảm, tăng lượt xem và bình luận |
| **Ngày 2** | Facebook | Bài viết phân tích 3 sai lầm khiến tân sinh viên dễ "gãy" môn lập trình cơ bản | Giữ chân người đọc, định vị khóa học Python cho người mới |
| **Ngày 3** | TikTok | Video 35s: "Sau 8 tuần học Python từ con số 0, sinh viên năm nhất làm được gì?" | Chứng minh hiệu quả thực tế của lộ trình 8 tuần |
| **Ngày 4** | Facebook | Giới thiệu chi tiết lộ trình 8 tuần Python cơ bản + Ưu đãi học phí sinh viên | Thúc đẩy hành động inbox / điền form đăng ký tư vấn |
| **Ngày 5** | TikTok | Video giải đáp nhanh: "Không giỏi Toán, chưa từng học Tin cấp 3 có học Python được không?" | Xóa bỏ rào cản tâm lý cho nhóm 18–20 tuổi |
| **Ngày 6** | Facebook | Chia sẻ dự án mini của học viên năm nhất sau 8 tuần học + Lịch khai giảng | Tạo niềm tin xã hội (Social Proof), gom lượt đăng ký tư vấn |
| **Ngày 7** | Cả 2 kênh | Bài đăng & Video nhắc hạn đăng ký tư vấn đợt này (Chốt suất ưu đãi cuối tuần) | Đẩy mạnh chuyển đổi để cán mốc 100 lượt tư vấn |

### 3.3. Hai bài đăng Facebook (Output Prompt 3 - Phần 1)

#### Bài đăng Facebook 1 (Đánh vào nỗi đau mất gốc của SV năm nhất)
> **SINH VIÊN NĂM NHẤT "SANG CHẤN TÂM LÝ" VÌ MÔN LẬP TRÌNH ĐẦU TIÊN?** 😭💻
> 
> Mới chân ướt chân ráo vào giảng đường ở tuổi 18–20, nhiều bạn tân sinh viên đã phải đối mặt với "cơn ác mộng" mang tên: Nhìn dòng code như nhìn bức vách, giảng viên lướt slide vèo vèo còn mình thì chưa kịp hiểu biến với vòng lặp là gì!
> 
> Đừng để nỗi sợ mất gốc ngay từ năm nhất khiến bạn chán nản hay đối mặt nguy cơ học lại. Thực tế, lập trình không hề đáng sợ nếu bạn chọn đúng ngôn ngữ khởi đầu dễ tiếp cận nhất: **Python**.
> 
> Với thông điệp **"8 tuần xóa mù Python từ con số 0 – Tự tin làm chủ lập trình ngay từ năm nhất"**, khóa học Python cơ bản tại Trung tâm được thiết kế riêng cho người mới bắt đầu hoàn toàn:
> ✅ Không yêu cầu nền tảng code trước đó.
> ✅ Học đi đôi với thực hành, sửa lỗi trực tiếp cùng giảng viên.
> ✅ Thời lượng gọn nhẹ chỉ **8 tuần**, vừa vặn với lịch học trên trường của sinh viên.
> 
> 👉 Bạn đang loay hoay chưa biết bắt đầu học code từ đâu? **Comment "TƯ VẤN"** hoặc **nhắn tin trực tiếp cho Fanpage** ngay hôm nay để được đội ngũ chuyên môn kiểm tra năng lực và gửi lộ trình học chi tiết nhé!

#### Bài đăng Facebook 2 (Giới thiệu lộ trình 8 tuần & Kêu gọi đăng ký tư vấn)
> **CHỈ 8 TUẦN – TỪ NGƯỜI MỚI BẮT ĐẦU ĐẾN TỰ TAY VIẾT ỨNG DỤNG PYTHON ĐẦU TIÊN!** 🚀
> 
> Dành riêng cho các bạn **sinh viên năm nhất (18–20 tuổi)** muốn xây nền tảng lập trình vững chắc ngay từ học kỳ đầu tiên mà không tốn quá nhiều thời gian!
> 
> Khóa học **Python Cơ Bản (8 tuần)** cam kết giúp bạn **"8 tuần xóa mù Python từ con số 0 – Tự tin làm chủ lập trình ngay từ năm nhất"** thông qua lộ trình tinh gọn:
> 🔹 **Tuần 1 – 2:** Làm quen tư duy lập trình, cài đặt môi trường và nắm chắc cú pháp Python căn bản.
> 🔹 **Tuần 3 – 5:** Làm chủ cấu trúc điều kiện, vòng lặp, hàm và xử lý dữ liệu thực tế.
> 🔹 **Tuần 6 – 8:** Thực hành xây dựng dự án mini hoàn chỉnh (Tool tự động hóa / Game đơn giản) để đưa vào CV sinh viên.
> 
> 🎯 Lớp học giới hạn sĩ số để giảng viên kèm cặp từng bạn, đảm bảo không ai bị bỏ lại phía sau.
> 📩 **Đăng ký nhận tư vấn lộ trình 1-1 miễn phí ngay tại link dưới bình luận** hoặc **Inbox Fanpage** để giữ suất ưu đãi học phí dành riêng cho tân sinh viên trong tuần này!

### 3.4. Hai kịch bản video TikTok 30–45 giây (Output Prompt 3 - Phần 2)

#### Kịch bản TikTok 1: Nỗi khổ sinh viên năm nhất học code (Thời lượng: 35 giây)
* **Thông điệp gắn kết:** 8 tuần xóa mù Python từ con số 0.

| Thời gian | Hình ảnh / Cảnh quay | Lời thoại / Text trên màn hình |
| :--- | :--- | :--- |
| **00s – 05s** | Quay cận mặt một bạn SV năm nhất (18-20 tuổi) nhìn màn hình laptop đầy chữ báo lỗi màu đỏ, mặt ngơ ngác. | **Text to:** "Tân sinh viên buổi đầu học code và cái kết..."<br>**Thoại:** "Ủa alo, thầy mới giảng tới đâu rồi mà màn hình em đỏ lòm như sớ táo quân vậy?" |
| **05s – 18s** | Chuyển cảnh nhanh: Bạn SV gõ phím lạch cạch nhưng vẫn lỗi, gục đầu xuống bàn thở dài. | **Thoại:** "Cấp 3 chưa từng viết một dòng code nào, lên đại học học nhanh quá nên mất gốc luôn từ tuần thứ 2. Bạn nào năm nhất đang bị tình trạng này giơ tay điểm danh cái!" |
| **18s – 30s** | Bạn SV mở trang khóa học Python cơ bản 8 tuần, giao diện code chạy mượt mà màu xanh. | **Text:** "8 tuần xóa mù Python từ con số 0"<br>**Thoại:** "Muốn hết cảnh mù mờ thì bắt đầu ngay với Python cơ bản nha. Lộ trình chỉ 8 tuần thiết kế riêng cho người mới bắt đầu, cầm tay chỉ việc từng dòng lệnh một." |
| **30s – 35s** | Chỉ tay xuống nút tin nhắn / link bio của kênh TikTok. | **Text:** "Nhắn tin nhận tư vấn lộ trình 8 tuần!"<br>**Thoại:** "Bấm ngay vào link đầu trang để nhận tư vấn lộ trình miễn phí cho sinh viên năm nhất nhé!" |

#### Kịch bản TikTok 2: Giải đáp thắc mắc & Kêu gọi tư vấn (Thời lượng: 40 giây)
* **Thông điệp gắn kết:** Tự tin làm chủ lập trình ngay từ năm nhất.

| Thời gian | Hình ảnh / Cảnh quay | Lời thoại / Text trên màn hình |
| :--- | :--- | :--- |
| **00s – 06s** | Người dẫn đứng trước bảng trắng hoặc laptop, cầm tờ giấy ghi chữ "MẤT GỐC TIN HỌC". | **Text to:** "Chưa biết gì về lập trình có học Python được không?"<br>**Thoại:** "Nhiều bạn sinh viên năm nhất nhắn hỏi mình: Em chưa từng học lập trình bao giờ thì có theo nổi Python không?" |
| **06s – 20s** | Chuyển cảnh quay màn hình code Python đơn giản dễ đọc như tiếng Anh, kèm hình ảnh học viên đang thực hành. | **Thoại:** "Câu trả lời là cực kỳ hợp luôn nha! Python có cú pháp ngắn gọn, dễ hiểu nhất cho người mới bắt đầu. Tại trung tâm, chỉ trong 8 tuần bạn sẽ đi từ con số 0 đến tự tay làm được dự án nhỏ của riêng mình." |
| **20s – 32s** | Hiện bảng tóm tắt: Đối tượng SV năm nhất (18-20t) - Thời lượng 8 tuần - Kèm cặp 1-1. | **Text:** "8 tuần xóa mù Python từ con số 0"<br>**Thoại:** "Lịch học linh hoạt không trùng giờ trên trường, giảng viên sửa lỗi tận tay nên không sợ bị bỏ lại phía sau." |
| **32s – 40s** | Cười tươi, vẫy tay chào và chỉ vào phần bình luận / link bio. | **Text:** "Đăng ký tư vấn ngay hôm nay!"<br>**Thoại:** "Nhanh tay đăng ký tư vấn ở link đầu trang để giữ suất ưu đãi cho tân sinh viên đợt này nha!" |

### 3.5. Báo cáo kiểm tra tính nhất quán (Output Prompt 4)

1. **Kiểm tra thông số khóa học:**
   * **Đạt chuẩn 100%:** Toàn bộ 4 bước đều giữ nguyên thông số gốc: Khóa học Python cơ bản cho người mới bắt đầu, đối tượng sinh viên năm nhất (18–20 tuổi), thời lượng 8 tuần, triển khai trên 2 kênh Facebook và TikTok. Không có thông tin nào bị AI tự ý bịa thêm hay sửa đổi.
2. **Kiểm tra sự liên kết chuỗi (Context Consistency):**
   * **Chặt chẽ:** 3 nỗi sợ ở Bước 1 (sợ mất gốc, sợ lý thuyết khô khan, sợ tốn thời gian) đã được chuyển hóa trực tiếp thành Thông điệp chính ở Bước 2 (*"8 tuần xóa mù Python từ con số 0..."*) và xuất hiện xuyên suốt trong cả 7 ngày lên lịch cũng như 4 sản phẩm nội dung ở Bước 3.
3. **Kiểm tra mục tiêu CTA (100 lượt đăng ký tư vấn):**
   * **Đạt yêu cầu:** Cả 2 bài Facebook và 2 kịch bản TikTok đều có lời kêu gọi hành động (CTA) rõ ràng hướng người xem nhắn tin hoặc điền link đăng ký tư vấn thay vì chỉ câu like chung chung.
4. **Đề xuất chỉnh sửa nhỏ để tối ưu thêm:**
   * Ở **Bài đăng Facebook 2** và **Kịch bản TikTok 2**, phần lời kêu gọi cuối bài có nhắc tới *"ưu đãi học phí cho tân sinh viên"* — nếu trung tâm thực tế chưa có chương trình giảm giá thì nên sửa lại thành *"tặng bộ tài liệu tự học Python cho người mới"* để tránh sai lệch chính sách khi tư vấn viên chốt đơn.
