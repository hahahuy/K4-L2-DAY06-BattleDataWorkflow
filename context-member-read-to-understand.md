# Bối Cảnh Cho Thành Viên Nhóm Đọc Trước

## 1. Bài Toán Của Nhóm Là Gì?

Vinmec muốn dùng AI để **sắp xếp ưu tiên hàng đợi đọc ảnh X-quang ngực**. Ảnh có dấu hiệu nghi ngờ bất thường sẽ được đưa lên đầu hàng đợi để bác sĩ đọc sớm hơn. Ảnh còn lại vẫn được đọc theo thứ tự thông thường.

Đây là bài toán **triage / screening**, không phải bài toán chẩn đoán:

- AI không kết luận bệnh gì.
- AI không thay thế bác sĩ chẩn đoán hình ảnh.
- AI không được bỏ qua hay ẩn bất kỳ ảnh nào.
- Cuối cùng, bác sĩ vẫn đọc tất cả các ca.

Objective mà cả nhóm cần nhớ:

> Từ một ảnh X-quang ngực thẳng đã được ẩn danh, quyết định đưa ảnh lên đầu hàng đợi bác sĩ đọc hay giữ thứ tự thông thường, để các ca nghi ngờ bất thường được đọc sớm hơn.

## 2. Quy Tắc An Toàn Quan Trọng Nhất

**False negative đắt hơn false positive.**

False negative ở đây là ảnh có dấu hiệu bất thường nhưng lại bị gán vào hàng đợi thường. Điều này có thể làm chậm việc bác sĩ phát hiện ca nặng.

False positive là ảnh không đáng lo nhưng vẫn bị đẩy lên hàng đợi ưu tiên. Điều này làm tăng tải cho bác sĩ, nhưng ít nguy hiểm hơn bỏ sót ca nghi ngờ.

Vì vậy, workflow của nhóm phải ưu tiên độ nhạy cao cho nhãn `suspected abnormality`, chấp nhận có thêm cảnh báo sai, và kiểm tra rất kỹ lỗi `suspected abnormality -> routine`.

## 3. Dữ Liệu Được Dùng Và Không Được Dùng

Chỉ dùng ảnh y tế công khai đã ẩn danh, có giấy phép / data-use agreement rõ ràng. Không dùng ảnh bệnh nhân thật của Vinmec hay bất kỳ dữ liệu chưa ẩn danh nào.

Ví dụ hợp lý: MIMIC-CXR là bộ dữ liệu X-quang ngực công khai đã ẩn danh, có ảnh DICOM, báo cáo, `patient_id` và `study_id`. Tuy nhiên, nó vẫn là dữ liệu truy cập có kiểm soát và phải tuân thủ data-use agreement.

Trước khi annotator xem ảnh:

- Kiểm tra metadata DICOM đã ẩn danh.
- Kiểm tra chữ chèn trên ảnh (burned-in text) có thể lộ thông tin cá nhân.
- Ảnh hỏng, sai định dạng, hoặc nghi lộ PHI phải vào **quarantine**, không được xóa im lặng.
- Mỗi ảnh phải có dòng ledger ghi nguồn gốc và lịch sử xử lý.

## 4. Đơn Vị Dữ Liệu, Group Key Và Leakage

- **Đơn vị dữ liệu:** một ảnh X-quang ngực thẳng (AP hoặc PA) của một lần chụp.
- **Group key:** `patient_id`.

Một bệnh nhân có thể chụp nhiều lần. Nếu ảnh lần 1 nằm ở train, ảnh lần 2 nằm ở test, model đã thấy gần như cùng người và kết quả test sẽ đẹp giả. Đây gọi là **data leakage**.

Do đó, tất cả ảnh của một `patient_id` chỉ được nằm trong một split duy nhất. Nhóm đề xuất khóa split theo tỷ lệ `70% train / 10% validation / 20% test` trước khi gán nhãn hàng loạt. Gate bắt buộc: không có `patient_id` nào xuất hiện ở hơn một split.

## 5. Nhãn Là Gì?

| Nhãn | Nghĩa đúng | Hành động |
|---|---|---|
| `Suspected abnormality` | Bác sĩ thấy dấu hiệu trên ảnh có thể cần đọc sớm hơn, dù chưa chắc chắn chẩn đoán cụ thể. | Đưa vào hàng đợi ưu tiên |
| `Routine: no suspected abnormality` | Không có dấu hiệu trên ảnh cần đọc sớm theo quy tắc đã thống nhất. Không đồng nghĩa với "người bệnh hoàn toàn khỏe". | Giữ hàng đợi thường |
| `Unreadable / insufficient quality` | Ảnh quá mờ, sai tư thế, thiếu vùng giải phẫu, phơi sáng tối không phù hợp, hoặc artifact làm bác sĩ không thể triage tin cậy. | Chuyển kiểm tra kỹ thuật / thủ công |
| `Abstain / escalate` | Bác sĩ annotator không thể chọn nhãn an toàn theo guideline. | Chuyển senior radiologist phân xử |

Quy tắc quan trọng: **không chắc chắn không được đoán bừa.** Ca mơ hồ có thể là `suspected abnormality` hoặc `abstain`, tùy guideline; không được tự động gán `routine` chỉ vì không chắc chắn.

## 6. Vì Sao Guideline Phải Cụ Thể?

Guideline phải mô tả dấu hiệu quan sát được, không viết chung chung như "nếu ảnh xấu thì gán bất thường".

Ca khó cần đưa vào guideline:

- Mờ opacity nhẹ / không chắc chắn: ưu tiên `suspected abnormality`, không gọi là routine do phân vân.
- Bất thường mạn tính hay cấp tính: không tự suy đoán bệnh sử. Nếu từ ảnh mà cần bác sĩ đọc sớm thì gán nghi ngờ.
- Ảnh bị motion, quay lệch, thiếu đỉnh phổi/góc sườn-hoành, exposure kém: nếu không triage tin cậy được thì `unreadable`.
- Ống, dây dẫn, thiết bị: chỉ ưu tiên nếu có dấu hiệu quan sát được theo quy tắc; không suy đoán tình trạng lâm sàng.

Guideline có version. Mỗi lần sửa quy tắc phải ghi vào decision log, và biết batch nào đã dùng guideline version nào.

## 7. Tại Sao Pilot Quan Trọng?

Sinh viên không đủ thẩm quyền gán nhãn lâm sàng. Annotator bắt buộc phải là bác sĩ chẩn đoán hình ảnh, nhưng bác sĩ rất ít thời gian: giả định chỉ có 2 giờ mỗi tuần.

Trước khi gán nhãn nhiều, chạy pilot 30 ảnh có ca dễ, khó, nghi ngờ bất thường, khác nguồn, khác projection và chất lượng kém.

- Hai bác sĩ gán độc lập và không thấy đáp án của nhau.
- Không cho thấy AI pre-label, candidate label từ report, hay nhãn của người kia trước khi lưu quyết định đầu tiên. Đây là cách chống **anchoring**.
- Đo weighted Cohen's kappa.
- Xem từng ca bất đồng, nhất là trường hợp một người gán `routine` còn người kia gán `suspected abnormality`.
- Senior radiologist hoặc hội đồng đã được chỉ định phân xử.
- Bất đồng lặp lại phải biến thành quy tắc rõ hơn trong guideline v2.

Gate để qua pilot: weighted kappa `>= 0.80`, không còn nhóm bất đồng lặp lại chưa được giải quyết, và tất cả false-routine disagreement đã được xem lại.

## 8. Gán Nhãn Hàng Loạt Như Thế Nào Khi Bác Sĩ Ít Thời Gian?

Không thể bắt hai bác sĩ gán lại toàn bộ dữ liệu. Cách thực chiến hơn:

- Một radiologist annotator gán nhãn độc lập cho ca thường.
- Bắt buộc có người thứ hai review cho: `suspected abnormality`, `unreadable`, `abstain`, nguồn/scanner lạ, và ca nằm trong risk queue.
- AI hoặc báo cáo cũ chỉ được dùng để tạo risk queue, không phải ground truth.
- Hệ thống ẩn AI/report suggestion cho tới khi bác sĩ đã lưu nhãn đầu tiên.
- Reviewer không được là người vừa gán nhãn ảnh đó.
- Senior radiologist giải quyết abstain và disagreement; phê duyệt thay đổi guideline.

## 9. QC: Không Được Chỉ Báo Cáo Accuracy Trung Bình

Lớp `routine` có thể chiếm đa số. Nếu chỉ báo cáo accuracy tổng, hệ thống có thể nhìn rất tốt nhưng vẫn bỏ sót nhiều ca nghi ngờ bất thường.

Nhóm tách ba dòng kiểm tra:

| Dòng QC | Mục đích | Không được dùng để |
|---|---|---|
| Random audit | Ước lượng chất lượng của cả batch. Lấy mẫu ngẫu nhiên có seed, `max(10% batch, 100 ảnh)`. | Săn lỗi hiếm hiệu quả |
| Safety audit | Tìm false-routine nguy hiểm. Review độc lập ít nhất 100 ảnh đã gán routine. | Ước lượng error rate chung của batch |
| Risk queue | Xem tất cả abstain, disagreement, low-confidence, nguồn mới, ảnh chất lượng kém. | Báo cáo như tỷ lệ lỗi của cả batch |

Metric cần nhớ:

```text
False-routine rate
= số ảnh được adjudicate là suspected abnormality nhưng bị gán routine
  / tổng số ảnh suspected abnormality đã được audit
```

Báo cáo thêm macro agreement, worst-class error rate, confusion matrix, và tách kết quả theo nguồn, projection, chất lượng, guideline version. Mọi tỷ lệ phải có tử số và mẫu số.

Gate để release ban đầu:

- Macro agreement trên random audit `>= 0.90`.
- False-routine rate trên safety audit `<= 2%`.
- Không còn systematic error cluster chưa xử lý.

Con số `2%` là mục tiêu thiết kế cho bài tập, không phải tuyên bố đã được chứng minh an toàn lâm sàng. Khi triển khai thật, ngưỡng phải do clinical governance của Vinmec phê duyệt.

## 10. Release Và Monitoring

Dataset chỉ được `RELEASE v1.0` khi có đủ bằng chứng:

1. Ảnh và nhãn cuối cùng.
2. Patient-level split đã khóa.
3. Guideline version và decision log.
4. Lịch sử annotator, reviewer, adjudication.
5. QC report: sample, seed, tử số/mẫu số, confusion matrix, corrective action.
6. Dataset card: dùng cho triage gì, không được dùng cho gì, nguồn/license, khoảng trống coverage, lỗi đã biết, privacy control.
7. Chữ ký của data owner cụ thể.

Nếu thiếu bất kỳ bằng chứng nào: `HOLD`, ghi rõ thiếu gì, ai bổ sung, khi nào xong.

Sau release, workflow vẫn tiếp tục. Theo dõi hàng tuần:

- Nguồn, scanner, AP/PA mix, chất lượng ảnh có thay đổi không.
- Tỷ lệ ảnh bị đẩy lên priority có quá cao và gây alert fatigue không.
- Priority có được đọc sớm hơn routine không.
- Các ca routine có bị bỏ sót bất thường không.
- Lỗi tập trung ở scanner, source, projection, chất lượng hay loại bất thường nào.

Mỗi phát hiện phải quay về một bước cụ thể trong lifecycle và có người chịu trách nhiệm xử lý.

## 11. Sáu Câu Hỏi Có Thể Bị Hỏi

1. **Dataset phục vụ quyết định gì?** Ưu tiên ảnh X-quang nghi ngờ bất thường để bác sĩ đọc sớm hơn; không chẩn đoán và không bỏ qua ảnh nào.
2. **Lỗi nào đắt hơn?** False negative. Vì vậy ưu tiên sensitivity, có abstain, safety audit false-routine, chấp nhận false positive.
3. **Ca khó và cách xử lý?** Mờ opacity nhẹ/không chắc chắn: không tự động gán routine; gán suspected abnormality hoặc abstain theo guideline.
4. **Group key là gì?** `patient_id`; khóa split patient-level và kiểm tra không trùng patient giữa train/val/test.
5. **Đo chất lượng bằng gì?** Random audit + safety audit, macro/worst-class error, false-routine rate có tử số/mẫu số; reviewer độc lập, senior radiologist adjudicate.
6. **Cần bao nhiêu người và dễ vỡ ở đâu?** Data steward, radiologist annotator, reviewer, senior adjudicator, data owner. Dễ vỡ nhất là thiếu thời gian bác sĩ; xử lý bằng pilot, risk-based double review, audit và relabel có mục tiêu.
