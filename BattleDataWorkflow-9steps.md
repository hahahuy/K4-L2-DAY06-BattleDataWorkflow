VINUNIVERSITY

AICB-P2T4 · DATA TRACK

⚔Battle Data Workflow

[Tổng quan][1][9 bước lifecycle][2][Đề tài][3][Chấm điểm][4][Thể lệ battle][5][Phân nhóm][6][⏱ Trình chiếu][7]

# Chín bước của Data Annotation Lifecycle

Gán nhãn không phải một việc làm xong là hết. Nó là một vòng đời chín bước và nó lặp lại — mỗi vòng cho ra một dataset
version mới. Mỗi bước kết thúc bằng một artifact và một cổng; bước nào không nêu được artifact của mình thì bước đó chưa
thật sự hoàn thành.

Cách hiểu sai và cách hiểu đúng

**Sai:** gán xong là hết việc, dataset là sản phẩm dùng một lần — thu ảnh, gán nhãn, giao file, đóng dự án.

**Đúng:** dataset là sản phẩm có phiên bản. v1.0 phát hành, được theo dõi, phát hiện lỗi, và sinh ra v1.1 với rule đã
sửa. Mỗi release phải để lại đủ dấu vết để vòng sau biết cần thu thêm ảnh nào và sửa rule nào.

1

### Mục tiêu & Phạm vi

Define Data Objective & Annotation Scope

Xác định dataset này phục vụ quyết định nào, và phạm vi gán nhãn đến đâu.

Nhận gì

Bài toán nghiệp vụ, người dùng cuối, ràng buộc của dự án.

Ai làm

Người phụ trách dữ liệu (data owner), có người duyệt.

Làm gì

  * →Viết objective có đơn vị dữ liệu, nơi dùng và hành động cụ thể.
  * →Chốt danh sách class, và liệt kê rõ cái gì nằm ngoài phạm vi.
  * →Quyết định false positive hay false negative đắt hơn — quyết ngay, không để sau.
  * →Tự đặt trước ngưỡng cho pilot và cho QC, khi chưa có con số nào trong tay.

Giao ra artifact

Tài liệu Objective & Scope, có người duyệt.

Cổng

Đọc objective mà suy ra được cần mấy class và độ chính xác nào là đủ.

Bẫy thường gặp “Xây dataset để train model nhận diện trái cây.” Đó là tên công việc, không phải objective — không suy ra
được điều gì.

2

### Thu thập & Chọn lọc

Data Collection / Data Selection

Thu và chọn dữ liệu theo kế hoạch, kèm provenance đầy đủ.

Nhận gì

Tài liệu Objective & Scope; nguồn dữ liệu và quyền truy cập.

Ai làm

Người thu thập, cùng người chịu trách nhiệm pháp lý về nguồn.

Làm gì

  * →Chia hạn ngạch theo từng nguồn và từng lớp, nhất là lớp hiếm.
  * →Mỗi file một dòng ledger: ID, group, nguồn, thời điểm, trạng thái.
  * →File hỏng hoặc thiếu thông tin thì đưa vào quarantine, không xoá im lặng.
  * →Ghi lại quyền sử dụng của từng nguồn, và xử lý thông tin cá nhân trước khi đưa cho người gán.

Giao ra artifact

Bộ file thô + data ledger có provenance đầy đủ.

Cổng

Các con số cộng đúng, mỗi file có đủ provenance, mọi nguồn đều có ghi nhận quyền sử dụng.

Bẫy thường gặp 5.000 ảnh cùng một khay, cùng một giờ — nhìn thì nhiều, thực chất vẫn chỉ là một khay.

3

### Chuẩn bị & Tiền xử lý

Data Preparation & Preprocessing

Hiểu dữ liệu, dọn dữ liệu, rồi mới chia train / validation / test.

Nhận gì

Bộ file thô + data ledger từ bước 2.

Ai làm

Người phân tích dữ liệu.

Làm gì

  * →EDA: kiểm schema, xem phân bố theo lớp và theo nguồn, mở tận mắt các ảnh khó.
  * →Tìm trùng lặp và gần trùng — điểm tương đồng cao là tín hiệu để xem, không phải lệnh xoá.
  * →Chuẩn hoá định dạng, kích thước, tên file; che thông tin nhạy cảm; ghi mọi biến đổi vào ledger.
  * →Chọn group key rồi chia split theo group, không chia ngẫu nhiên từng ảnh.

Giao ra artifact

Bộ eligible đã chuẩn hoá, báo cáo EDA, danh sách trùng lặp, split cố định kèm group key.

Cổng

Không có ảnh nào cùng group nằm ở hai bên split; mọi biến đổi đều có dòng ghi trong ledger.

Bẫy thường gặp Chia ngẫu nhiên từng ảnh. Test đã “nhìn thấy” cùng một khay qua các ảnh gần trùng, và điểm số cao hơn
thực tế.

4

### Viết Guideline

Annotation Guideline

Biến định nghĩa trong đầu người quản lý thành quy tắc mà ai đọc cũng gán ra cùng một nhãn.

Nhận gì

Objective & Scope; bộ eligible đã chia split; nhận xét từ EDA về các trường hợp khó.

Ai làm

Người phụ trách chuyên môn, viết cho người chưa biết gì về dự án.

Làm gì

  * →Định nghĩa từng class bằng dấu hiệu quan sát được, không bằng kết luận.
  * →Đưa ví dụ đúng, ví dụ sai, và các trường hợp rìa.
  * →Quy định cách vẽ, khi nào được abstain, khi nào phải escalate.
  * →Khai báo schema trong công cụ khớp từng chữ với guideline, kể cả cách viết hoa.

Giao ra artifact

Guideline có version + schema đã khai báo trong công cụ + decision log sẵn sàng ghi.

Cổng

Người chưa tham gia dự án đọc guideline và gán được một ảnh mẫu ra đúng nhãn mong đợi.

Bẫy thường gặp “Gán là dập nếu quả trông bị hỏng.” Mỗi người hiểu “trông bị hỏng” một kiểu.

5

### Pilot

Pilot Annotation

Chạy thử trên mẫu nhỏ để phát hiện guideline hỏng trước khi tốn tiền gán hàng loạt.

Nhận gì

Guideline bản nháp; schema trong công cụ; 20–30 ảnh rút từ bộ eligible.

Ai làm

Đúng những người sẽ gán batch thật, không phải người quản lý gán cho nhanh.

Làm gì

  * →Hai người gán độc lập, không nhìn kết quả của nhau.
  * →Mẫu phải có cả ảnh dễ lẫn ảnh khó, đủ các nguồn — không chỉ lấy ảnh đẹp.
  * →Đo agreement, nhóm các ca bất đồng lại xem chúng dồn vào loại tình huống nào.
  * →Sửa guideline thành version mới, và dựng bộ reference cho QC về sau.

Giao ra artifact

Kết quả pilot (agreement, danh sách ca bất đồng), guideline version tiếp theo, bộ reference.

Cổng

Agreement đạt ngưỡng đặt trước, và mọi loại bất đồng lặp lại đều đã thành quy tắc.

Bẫy thường gặp Pilot toàn ảnh dễ — agreement 98%, rồi batch thật rơi xuống 70%.

6

### Gán nhãn hàng loạt

Annotation

Gán nhãn hàng loạt: phân vai rõ, làm độc lập, tự kiểm trước khi bàn giao.

Nhận gì

Guideline đã qua pilot; schema trong công cụ; batch ảnh chia theo group; phân vai rõ.

Ai làm

Annotator · Reviewer · Adjudicator · Data owner — bốn vai, bốn trách nhiệm.

Làm gì

  * →Chia batch theo group để còn truy được lỗi theo lô.
  * →Gán độc lập; người gán một batch không được là người review chính batch đó.
  * →Nếu dùng AI prelabel: người xem từng đề xuất, người sửa, người ký — và phải cảnh giác với anchoring.
  * →Chạy checklist tự kiểm trước khi bấm nộp.

Giao ra artifact

Batch candidate label đã tự kiểm, danh sách ảnh escalate, log ai gán ảnh nào theo rule version nào.

Cổng

Batch nộp đủ và đúng schema; reviewer có quyền từ chối nhận nếu thiếu field hoặc sai version.

Bẫy thường gặp Cả hai người cùng nhận prelabel từ AI và cùng chấp nhận một lỗi giống nhau. Agreement 100%, nhãn sai
100%.

7

### QA / QC

Quality Assurance / Quality Control

Đo chất lượng trên mẫu có mẫu số rõ ràng, rồi sửa nguyên nhân chứ không chỉ sửa từng ảnh.

Nhận gì

Batch candidate label; reference từ bước 5; ngưỡng gate đặt trước.

Ai làm

Reviewer, không phải người vừa gán batch đó.

Làm gì

  * →Rút mẫu audit ngẫu nhiên có seed — để mẫu số tái hiện được.
  * →Tách random audit khỏi risk queue: một cái ước lượng cho cả batch, một cái để tìm lỗi.
  * →Mỗi loại lỗi một mẫu số riêng. Không cộng ba tỉ lệ khác mẫu số vào nhau.
  * →Báo cáo cả micro, macro và worst class; đặt gate trên macro hoặc worst.
  * →Với mẫu lỗi lặp lại: tìm nguyên nhân, sửa rule, gán lại đúng nhóm ảnh đó, rồi kiểm chứng.

Giao ra artifact

QC report: mẫu audit và mẫu số, tỉ lệ lỗi theo từng loại, confusion matrix, issue đã phân xử, corrective action.

Cổng

Batch đạt ngưỡng đã đặt trước, hoặc bị trả lại kèm phạm vi gán lại được xác định rõ.

Bẫy thường gặp Lấy tỉ lệ lỗi trên risk queue rồi báo cáo như tỉ lệ lỗi của cả batch.

8

### Phát hành

Release

Đóng gói dataset thành một phiên bản tái hiện được, kèm đủ bằng chứng để người khác tin được.

Nhận gì

Batch đã qua QC; guideline version; split; kết quả đo và danh sách hạn chế.

Ai làm

Data owner — một người cụ thể ký tên, không phải “cả nhóm”.

Làm gì

  * →Đóng gói đủ năm thành phần: dữ liệu + nhãn, split, guideline, QC report, dataset card.
  * →Viết dataset card một trang, trong đó phần hạn chế đã biết là phần quan trọng nhất.
  * →Trả lời sáu câu hỏi trước khi ký — bằng tài liệu trong packet, không bằng trí nhớ.
  * →Nếu thiếu bằng chứng thì HOLD, kèm: thiếu gì, ai bổ sung, mất bao lâu.

Giao ra artifact

Release packet v1.0 có version và chữ ký.

Cổng

Sáu câu hỏi đều trả lời được bằng tài liệu trong packet, và có người ký tên.

Bẫy thường gặp “Đã trễ hạn rồi, cứ phát hành đi, phần thiếu bổ sung sau.” Phần thiếu gần như không bao giờ được bổ sung.

9

### Theo dõi & Phản hồi

Monitoring, Error Analysis & Feedback

Theo dõi sau khi phát hành, phân tích lỗi, rồi đưa bài học quay lại bước 1.

Nhận gì

Release packet đã phát hành; dữ liệu và kết quả từ vận hành thực tế.

Ai làm

Người vận hành cùng data owner.

Làm gì

  * →Theo dõi dữ liệu vào: ảnh chạy thật có còn giống ảnh trong dataset không?
  * →Theo dõi kết quả: model sai nhiều hơn không, sai tập trung ở lớp nào, nguồn nào?
  * →Phân tích lỗi để biết lỗi thuộc về model, guideline, thiếu dữ liệu hay sai objective.
  * →Mỗi phát hiện chỉ về một bước xác định, kèm người chịu trách nhiệm xử lý.

Giao ra artifact

Báo cáo theo dõi định kỳ, kết quả error analysis, danh sách việc cho vòng sau.

Cổng

Mỗi phát hiện đều chỉ về một bước xác định trong lifecycle, kèm người chịu trách nhiệm.

Bẫy thường gặp Chỉ nhìn tỉ lệ lỗi. Biết “chất lượng đang tụt” mà không biết vì sao — cột nguồn mới là thứ chỉ ra nguyên
nhân.

## Bốn lỗi im lặng

Cả bốn lỗi này đều im lặng: không có thông báo lỗi, không có màn hình đỏ. Chúng chỉ lộ ra khi đã tốn nhiều tiền. Cách
duy nhất phát hiện sớm là mỗi bước phải có cổng, và cổng phải được kiểm bằng bằng chứng chứ không bằng cảm nhận.

| Lỗi                         | Biểu hiện                 | Bước chặn được |
|-----------------------------|---------------------------|----------------|
| Không ai viết objective     | tranh cãi vô tận về class | Bước 1         |
| Mất liên kết group          | điểm test cao giả         | Bước 2 và 3    |
| Guideline nói bằng kết luận | hai người gán khác nhau   | Bước 4 và 5    |
| Tỉ lệ không có mẫu số       | báo cáo đẹp, dữ liệu xấu  | Bước 7         |


## Mười thuật ngữ

Objective — quyết định mà dataset này phục vụ

Scope — phạm vi: gán cái gì, không gán cái gì

Provenance — nguồn gốc: thu ở đâu, khi nào, qua tay ai

Ledger — sổ ghi từng record và lịch sử của nó

Guideline — tài liệu quy định nhãn nào là đúng

Gate — cổng pass/fail có ngưỡng và người quyết

Split — cách chia train / validation / test

Agreement — mức đồng thuận giữa hai người gán

Audit — kiểm tra trên một mẫu rút ngẫu nhiên

Release — dataset đã đóng gói và đánh version

Năm ý phải nhớ

  1. Annotation là một vòng đời chín bước và nó lặp lại: mỗi vòng cho ra một dataset version mới.
  2. Mỗi bước kết thúc bằng một artifact và một cổng — nếu không, bước sau buộc phải nhận mọi thứ.
  3. Objective và guideline đi trước dữ liệu — và guideline phải qua pilot trước khi gán hàng loạt.
  4. Agreement không phải là truth — mọi tỉ lệ phải kèm mẫu số.
  5. AI đề xuất, con người quyết định — chữ ký trên release vẫn là của người.

**VinUniversity** · AICB-P2T4 · Ngày 06 — Annotation Workflow & Quality ControlTrang dành cho trợ giảng trình chiếu tại
lớp.

   [1]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/

   [2]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/lifecycle

   [3]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/de-tai

   [4]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/cham-diem

   [5]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/battle

   [6]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/phan-nhom

   [7]: https://grocery-computed-endorsement-paste.trycloudflare.com/#/trinh-chieu

