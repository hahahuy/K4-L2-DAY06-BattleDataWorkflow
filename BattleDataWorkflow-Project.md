VINUNIVERSITY

AICB-P2T4 · DATA TRACK

⚔Battle Data Workflow

[Tổng quan][1][9 bước lifecycle][2][Đề tài][3][Chấm điểm][4][Thể lệ battle][5][Phân nhóm][6][⏱ Trình chiếu][7]

# Bốn đề tài

Bốn bài toán có thật trong hệ sinh thái Vin. Mỗi lớp dùng đúng bốn đề tài này, mỗi đề tài có đúng hai nhóm — nhóm trưởng
lên bốc thăm ở đầu buổi. Hai nhóm cùng đề tài sẽ battle với nhau ở cuối buổi.

Bối cảnh là gợi ý — objective là của nhóm

Phần “quyết định gợi ý” bên dưới là một cách viết objective đúng chuẩn bước 1, không phải đáp án bắt buộc. Hai nhóm cùng
đề tài hoàn toàn có thể phục vụ hai quyết định khác nhau từ cùng một bài toán — và đó thường là chỗ thú vị nhất khi
battle.

Đ1

### Phát hiện nguy cơ va chạm cho xe tự lái

VinFast

VinFast phát triển hệ thống hỗ trợ lái nâng cao. Camera trước xe ghi hình liên tục trong mỗi chuyến đi. Cần bộ dữ liệu
có nhãn để huấn luyện module quyết định phanh khẩn cấp.

Quyết định gợi ý — viết theo chuẩn bước 1

“Quyết định kích hoạt phanh khẩn cấp / phát cảnh báo cho tài xế / không làm gì, trên xe VinFast đang chạy, từ ảnh camera
trước, trong vòng dưới 200 mili giây.”

Đơn vị dữ liệu

một vật thể trong một khung hình camera trước

Lớp gợi ý

người đi bộxe máyô tôchướng ngại vật tĩnh

Group key gợi ý

chuyến đi (cùng một phiên ghi hình)

Lớp hiếm cần để ý

người băng qua đường ban đêm, xe cắt đầu đột ngột

Dữ liệu thực tế đến từ đâu

Video dashcam giao thông Việt Nam, bộ dữ liệu lái xe công khai.

Khó nhất ở chỗ

Chi phí lỗi không quy ra tiền được — bỏ sót là tính mạng.

Thách thức phải giải trong thiết kế

  * 1.Bỏ sót một người đi bộ không phải mất doanh thu mà là mất mạng. Ngưỡng QC của đề tài này phải đặt khác hẳn ba đề tài còn lại, và phải đặt trên lớp nguy hiểm nhất chứ không phải trên trung bình.
  * 2.Các khung hình liền nhau trong một chuyến gần như trùng hoàn toàn. Chia ngẫu nhiên từng khung là rò rỉ chắc chắn — điểm test sẽ đẹp giả.
  * 3.Đêm, mưa, ngược sáng chiếm tỉ lệ nhỏ trong dữ liệu nhưng lại đúng là lúc hệ thống buộc phải đúng.
  * 4.Vật bị che một phần: người đứng sau cột đèn, xe máy khuất sau ô tô. Che bao nhiêu thì vẫn gán?
  * 5.Biển số và mặt người đi đường lọt vào khung hình — phải che trước khi đưa cho người gán.

Đ2

### Nhận diện hành vi trong lớp học AI thực chiến

VinUni

Lớp thực hành muốn tự động điểm danh và nắm được mức độ tham gia của sinh viên. Camera đặt cuối phòng, ghi hình suốt
buổi học.

Quyết định gợi ý — viết theo chuẩn bước 1

“Quyết định ghi nhận một sinh viên là có mặt hay vắng, và có báo cho giảng viên khi lớp mất tập trung hay không, tại
phòng D303, từ camera cuối lớp.”

Đơn vị dữ liệu

một người trong một khung hình, hoặc một đoạn clip ngắn của một người

Lớp gợi ý

đang nghe giảngdùng điện thoạingủ gậttrao đổi nhómrời chỗ

Group key gợi ý

buổi học, và người (cùng một người xuất hiện suốt buổi)

Lớp hiếm cần để ý

ngủ gật, rời chỗ giữa giờ

Dữ liệu thực tế đến từ đâu

Tự quay lớp mình (phải xin phép cả lớp trước), video lớp học công khai.

Khó nhất ở chỗ

Dữ liệu là mặt sinh viên — và ranh giới hành vi thì cực mờ.

Thách thức phải giải trong thiết kế

  * 1.Đây là dữ liệu định danh của chính bạn bè trong lớp. Ai được xem, lưu bao lâu, sinh viên có được biết và đồng ý không? Phần quyền dùng dữ liệu của đề tài này nặng nhất.
  * 2.“Cúi xuống ghi chép” và “cúi xuống dùng điện thoại” nhìn gần giống hệt nhau. Guideline phải mô tả bằng dấu hiệu quan sát được, không bằng suy đoán ý định.
  * 3.Hành vi là chuỗi thời gian, không nằm gọn trong một khung hình. Gán một khung hay gán một đoạn — quyết định này đổi toàn bộ cách chia batch và cách đo.
  * 4.Người ngồi bàn sau bị người bàn trước che mất nửa người.
  * 5.Group key phải theo người chứ không theo khung hình: cùng một sinh viên mà rơi vào hai bên split là rò rỉ.

Đ3

### Sàng lọc bất thường trên ảnh X-quang ngực

Vinmec

Vinmec muốn có công cụ sàng lọc sơ bộ: gắn cờ những ảnh nghi ngờ để bác sĩ đọc trước, rút ngắn thời gian chờ cho ca
nặng.

Quyết định gợi ý — viết theo chuẩn bước 1

“Quyết định đưa một ảnh X-quang lên đầu hàng đợi đọc của bác sĩ hay để theo thứ tự thường, tại Vinmec, từ ảnh chụp thẳng
ngực.”

Đơn vị dữ liệu

một ảnh X-quang của một lượt chụp

Lớp gợi ý

bình thườngnghi ngờ bất thườngkhông đọc được (lỗi kỹ thuật)

Group key gợi ý

bệnh nhân — một người có thể chụp nhiều lần

Lớp hiếm cần để ý

các bất thường ít gặp

Dữ liệu thực tế đến từ đâu

CHỈ dùng bộ ảnh y tế công khai đã ẩn danh. Tuyệt đối không dùng ảnh bệnh nhân thật.

Khó nhất ở chỗ

Ai đủ thẩm quyền gán nhãn? Sinh viên thì không.

Thách thức phải giải trong thiết kế

  * 1.Annotator bắt buộc phải là bác sĩ — đắt, hiếm, lịch kín. Nếu bác sĩ chỉ có hai giờ mỗi tuần thì pilot, QC và vòng gán lại phải thiết kế thế nào để vẫn chạy được? Đây là câu hỏi trung tâm của đề tài này.
  * 2.Hai bác sĩ gán khác nhau chưa chắc là ai sai — có thể đó là ca thật sự khó. Ai là người phân xử, và phân xử dựa vào gì?
  * 3.Dữ liệu bệnh nhân: phải ẩn danh trước khi đưa cho bất kỳ ai, và không được mang ra ngoài bệnh viện. Ràng buộc pháp lý nặng nhất trong bốn đề tài.
  * 4.Group key là bệnh nhân chứ không phải ảnh. Cùng một người chụp ba lần mà chia hai bên split là rò rỉ nghiêm trọng.
  * 5.Lớp “bình thường” chiếm đa số áp đảo, nên chỉ số trung bình gần như vô nghĩa — phải nhìn lớp yếu nhất.
  * 6.Không có chuyên môn thì phải abstain chứ không đoán bừa. Quy tắc abstain của đề tài này là bắt buộc, không phải tuỳ chọn.

Đ4

### Giám sát trạm sạc xe điện

V-Green · VinFast

V-Green vận hành mạng trạm sạc trên toàn quốc. Camera đặt tại trạm. Đội vận hành cần biết trạm nào đang bị chiếm chỗ,
đang có sự cố, hay đang có hàng chờ.

Quyết định gợi ý — viết theo chuẩn bước 1

“Quyết định gửi cảnh báo cho đội vận hành / nhắc tài xế rời chỗ / không làm gì, tại một trạm sạc, từ camera giám sát
trạm.”

Đơn vị dữ liệu

một ô sạc trong một khung hình

Lớp gợi ý

đang sạcđỗ chiếm chỗ không sạcô trốngsự cố cáp

Group key gợi ý

trạm + phiên ghi hình

Lớp hiếm cần để ý

sự cố cáp, trạm mới lắp kiểu trụ khác

Dữ liệu thực tế đến từ đâu

Ảnh tự chụp tại trạm sạc, ảnh trạm sạc công khai.

Khó nhất ở chỗ

Hai lớp quan trọng nhất nhìn gần như giống hệt nhau.

Thách thức phải giải trong thiết kế

  * 1.“Đang sạc” và “đỗ chiếm chỗ” trông giống nhau: đều là một chiếc xe đứng yên ở ô sạc. Phân biệt bằng dấu hiệu quan sát được nào — cáp có cắm không, đèn trạng thái màu gì? Đây là bài toán viết guideline khó nhất trong bốn đề tài.
  * 2.Camera ngoài trời: đêm, mưa, nắng gắt, đèn pha xe khác quét qua.
  * 3.Một xe đỗ hai tiếng sinh ra hàng trăm khung hình gần như trùng nhau.
  * 4.Biển số lọt vào khung hình — phải che trước khi đưa cho người gán.
  * 5.Trạm mới lắp dùng kiểu trụ khác: nguồn mới mà model chưa từng thấy. Bước 9 phải phát hiện được điều này.

## Bốn đề tài, bốn kiểu khó khác nhau

Bốc trúng đề tài nào cũng gặp đủ chín bước, nhưng chỗ gãy của mỗi đề tài nằm ở một bước khác nhau. Biết mình khó ở đâu
thì dồn công sức vào đúng chỗ đó.

| Đề tài                                          | Khó nhất ở                                       | Bước chịu áp lực nhất               |
|-------------------------------------------------|--------------------------------------------------|-------------------------------------|
| Đ1Phát hiện nguy cơ va chạm cho xe tự lái       | Chi phí lỗi là tính mạng, không quy ra tiền được | Bước 1 và 7 — đặt ngưỡng            |
| Đ2Nhận diện hành vi trong lớp học AI thực chiến | Dữ liệu định danh + ranh giới hành vi mờ         | Bước 2 và 4 — quyền dùng và         |
| guideline                                       |
| Đ3Sàng lọc bất thường trên ảnh X-quang ngực     | Người gán phải là chuyên gia, rất hiếm giờ       | Bước 5 và 6 — pilot và phân vai     |
| Đ4Giám sát trạm sạc xe điện                     | Hai lớp quan trọng nhìn giống hệt nhau           | Bước 4 — viết quy tắc quan sát được |


## Luật chung cho mọi đề tài

### Được làm

  * ✓Đổi objective so với gợi ý, miễn là giải thích được vì sao.
  * ✓Thêm hoặc bớt lớp, miễn là bám theo quyết định nhóm đặt ra ở bước 1.
  * ✓Đề xuất dùng model AI gợi ý nhãn ban đầu — nhưng phải nói rõ cách chống anchoring.
  * ✓Nói thẳng chỗ nào trong quy trình của mình còn yếu.

### Không được làm

  * ✕Dùng ảnh bệnh nhân thật, hay bất kỳ dữ liệu y tế chưa ẩn danh nào (đề tài Đ3).
  * ✕Quay hoặc dùng hình ảnh bạn học mà chưa xin phép (đề tài Đ2).
  * ✕Bỏ qua phần che biển số và mặt người khi dữ liệu có lọt vào.
  * ✕Thiết kế quy trình giả định annotator không bao giờ sai và không ai bỏ việc giữa chừng.

**VinUniversity** · AICB-P2T4 · Ngày 06 — Annotation Workflow & Quality ControlTrang dành cho trợ giảng trình chiếu tại
lớp.

   [1]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/

   [2]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/lifecycle

   [3]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/de-tai

   [4]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/cham-diem

   [5]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/battle

   [6]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/phan-nhom

   [7]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/trinh-chieu

