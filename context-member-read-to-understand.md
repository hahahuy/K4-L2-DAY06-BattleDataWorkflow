# Boi Canh Cho Thanh Vien Nhom Doc Truoc

## 1. Bai Toan Cua Nhom La Gi?

Vinmec muon dung AI de **sap xep uu tien hang doi doc anh X-quang nguc**. Anh co dau hieu nghi ngo bat thuong se duoc dua len dau hang doi de bac si doc som hon. Anh con lai van duoc doc theo thu tu thong thuong.

Day la bai toan **triage / screening**, khong phai bai toan chan doan:

- AI khong ket luan benh gi.
- AI khong thay the bac si chan doan hinh anh.
- AI khong duoc bo qua hay an bat ky anh nao.
- Cuoi cung, bac si van doc tat ca cac ca.

Objective ma ca nhom can nho:

> Tu mot anh X-quang nguc thang da duoc an danh, quyet dinh dua anh len dau hang doi bac si doc hay giu thu tu thong thuong, de cac ca nghi ngo bat thuong duoc doc som hon.

## 2. Quy Tac An Toan Quan Trong Nhat

**False negative dat hon false positive.**

False negative o day la anh co dau hieu bat thuong nhung lai bi gan vao hang doi thuong. Dieu nay co the lam cham viec bac si phat hien ca nang.

False positive la anh khong dang lo nhung van bi day len hang doi uu tien. Dieu nay lam tang tai cho bac si, nhung it nguy hiem hon bo sot ca nghi ngo.

Vi vay, workflow cua nhom phai uu tien do nhay cao cho nhan `suspected abnormality`, chap nhan co them canh bao sai, va kiem tra rat ky loi `suspected abnormality -> routine`.

## 3. Du Lieu Duoc Dung Va Khong Duoc Dung

Chi dung anh y te cong khai da an danh, co giay phep / data-use agreement ro rang. Khong dung anh benh nhan that cua Vinmec hay bat ky du lieu chua an danh nao.

Vi du hop ly: MIMIC-CXR la bo du lieu X-quang nguc cong khai da an danh, co anh DICOM, bao cao, `patient_id` va `study_id`. Tuy nhien, no van la du lieu truy cap co kiem soat va phai tuan thu data-use agreement.

Truoc khi annotator xem anh:

- Kiem tra metadata DICOM da an danh.
- Kiem tra chu chen tren anh (burned-in text) co the lo thong tin ca nhan.
- Anh hong, sai dinh dang, hoac nghi lo PHI phai vao **quarantine**, khong duoc xoa im lang.
- Moi anh phai co dong ledger ghi nguon goc va lich su xu ly.

## 4. Don Vi Du Lieu, Group Key Va Leakage

- **Don vi du lieu:** mot anh X-quang nguc thang (AP hoac PA) cua mot lan chup.
- **Group key:** `patient_id`.

Mot benh nhan co the chup nhieu lan. Neu anh lan 1 nam o train, anh lan 2 nam o test, model da thay gan nhu cung nguoi va ket qua test se dep gia. Day goi la **data leakage**.

Do do, tat ca anh cua mot `patient_id` chi duoc nam trong mot split duy nhat. Nhom de xuat khoa split theo ty le `70% train / 10% validation / 20% test` truoc khi gan nhan hang loat. Gate bat buoc: khong co `patient_id` nao xuat hien o hon mot split.

## 5. Nhan La Gi?

| Nhan | Nghia dung | Hanh dong |
|---|---|---|
| `Suspected abnormality` | Bac si thay dau hieu tren anh co the can doc som hon, du chua chac chan chan doan cu the. | Dua vao hang doi uu tien |
| `Routine: no suspected abnormality` | Khong co dau hieu tren anh can doc som theo quy tac da thong nhat. Khong dong nghia voi "nguoi benh hoan toan khoe". | Giu hang doi thuong |
| `Unreadable / insufficient quality` | Anh qua mo, sai tu the, thieu vung giai phau, phoi sang toi khong phu hop, hoac artifact lam bac si khong the triage tin cay. | Chuyen kiem tra ky thuat / thu cong |
| `Abstain / escalate` | Bac si annotator khong the chon nhan an toan theo guideline. | Chuyen senior radiologist phan xu |

Quy tac quan trong: **khong chac chan khong duoc doan bay.** Ca mo ho co the la `suspected abnormality` hoac `abstain`, tuy guideline; khong duoc tu dong gan `routine` chi vi khong chac chan.

## 6. Vi Sao Guideline Phai Cu The?

Guideline phai mo ta dau hieu quan sat duoc, khong viet chung chung nhu "neu anh xau thi gan bat thuong".

Ca kho can dua vao guideline:

- Mo opacities nhe / khong chac chan: uu tien `suspected abnormality`, khong goi la routine do phan van.
- Bat thuong man tinh hay cap tinh: khong tu suy doan benh su. Neu tu anh ma can bac si doc som thi gan nghi ngo.
- Anh bi motion, quay lech, thieu dinh phoi/goc suon-hoanh, exposure kem: neu khong triage tin cay duoc thi `unreadable`.
- Ong, day dan, thiet bi: chi uu tien neu co dau hieu quan sat duoc theo quy tac; khong suy doan tinh trang lam sang.

Guideline co version. Moi lan sua quy tac phai ghi vao decision log, va biet batch nao da dung guideline version nao.

## 7. Tai Sao Pilot Quan Trong?

Sinh vien khong du tham quyen gan nhan lam sang. Annotator bat buoc phai la bac si chan doan hinh anh, nhung bac si rat it thoi gian: gia dinh chi co 2 gio moi tuan.

Truoc khi gan nhan nhieu, chay pilot 30 anh co ca de, kho, nghi ngo bat thuong, khac nguon, khac projection va chat luong kem.

- Hai bac si gan doc lap va khong thay dap an cua nhau.
- Khong cho thay AI pre-label, candidate label tu report, hay nhan cua nguoi kia truoc khi luu quyet dinh dau tien. Day la cach chong **anchoring**.
- Do weighted Cohen's kappa.
- Xem tung ca bat dong, nhat la truong hop mot nguoi gan `routine` con nguoi kia gan `suspected abnormality`.
- Senior radiologist hoac hoi dong da duoc chi dinh phan xu.
- Bat dong lap lai phai bien thanh quy tac ro hon trong guideline v2.

Gate de qua pilot: weighted kappa `>= 0.80`, khong con nhom bat dong lap lai chua duoc giai quyet, va tat ca false-routine disagreement da duoc xem lai.

## 8. Gan Nhan Hang Loat Nhu The Nao Khi Bac Si It Thoi Gian?

Khong the bat hai bac si gan lai toan bo du lieu. Cach thuc chien hon:

- Mot radiologist annotator gan nhan doc lap cho ca thuong.
- Bat buoc co nguoi thu hai review cho: `suspected abnormality`, `unreadable`, `abstain`, nguon/scanner la, va ca nam trong risk queue.
- AI hoac bao cao cu chi duoc dung de tao risk queue, khong phai ground truth.
- He thong an AI/report suggestion cho toi khi bac si da luu nhan dau tien.
- Reviewer khong duoc la nguoi vua gan nhan anh do.
- Senior radiologist giai quyet abstain va disagreement; phe duyet thay doi guideline.

## 9. QC: Khong Duoc Chi Bao Cao Accuracy Trung Binh

Lop `routine` co the chieu da so. Neu chi bao cao accuracy tong, he thong co the nhin rat tot nhung van bo sot nhieu ca nghi ngo bat thuong.

Nhom tach ba dong kiem tra:

| Dong QC | Muc dich | Khong duoc dung de |
|---|---|---|
| Random audit | Uoc luong chat luong cua ca batch. Lay mau ngau nhien co seed, `max(10% batch, 100 anh)`. | San loi hiem hieu qua |
| Safety audit | Tim false-routine nguy hiem. Review doc lap it nhat 100 anh da gan routine. | Uoc luong error rate chung cua batch |
| Risk queue | Xem tat ca abstain, disagreement, low-confidence, nguon moi, anh chat luong kem. | Bao cao nhu ti le loi cua ca batch |

Metric can nho:

```text
False-routine rate
= so anh duoc adjudicate la suspected abnormality nhung bi gan routine
  / tong so anh suspected abnormality da duoc audit
```

Bao cao them macro agreement, worst-class error rate, confusion matrix, va tach ket qua theo nguon, projection, chat luong, guideline version. Moi ty le phai co tu so va mau so.

Gate de release ban dau:

- Macro agreement tren random audit `>= 0.90`.
- False-routine rate tren safety audit `<= 2%`.
- Khong con systematic error cluster chua xu ly.

Con so `2%` la muc tieu thiet ke cho bai tap, khong phai tuyen bo da duoc chung minh an toan lam sang. Khi trien khai that, nguong phai do clinical governance cua Vinmec phe duyet.

## 10. Release Va Monitoring

Dataset chi duoc `RELEASE v1.0` khi co du bang chung:

1. Anh va nhan cuoi cung.
2. Patient-level split da khoa.
3. Guideline version va decision log.
4. Lich su annotator, reviewer, adjudication.
5. QC report: sample, seed, tu so/mau so, confusion matrix, corrective action.
6. Dataset card: dung cho triage gi, khong duoc dung cho gi, nguon/license, khoang trong coverage, loi da biet, privacy control.
7. Chu ky cua data owner cu the.

Neu thieu bat ky bang chung nao: `HOLD`, ghi ro thieu gi, ai bo sung, khi nao xong.

Sau release, workflow van tiep tuc. Theo doi hang tuan:

- Nguon, scanner, AP/PA mix, chat luong anh co thay doi khong.
- Ty le anh bi day len priority co qua cao va gay alert fatigue khong.
- Priority co duoc doc som hon routine khong.
- Cac ca routine co bi bo sot bat thuong khong.
- Loi tap trung o scanner, source, projection, chat luong hay loai bat thuong nao.

Moi phat hien phai quay ve mot buoc cu the trong lifecycle va co nguoi chiu trach nhiem xu ly.

## 11. Sau Cau Hoi Co The Bi Hoi

1. **Dataset phuc vu quyet dinh gi?** Uu tien anh X-quang nghi ngo bat thuong de bac si doc som hon; khong chan doan va khong bo qua anh nao.
2. **Loi nao dat hon?** False negative. Vi vay uu tien sensitivity, co abstain, safety audit false-routine, chap nhan false positive.
3. **Ca kho va cach xu ly?** Mo opacity nhe/khong chac chan: khong tu dong gan routine; gan suspected abnormality hoac abstain theo guideline.
4. **Group key la gi?** `patient_id`; khoa split patient-level va kiem tra khong trung patient giua train/val/test.
5. **Do chat luong bang gi?** Random audit + safety audit, macro/worst-class error, false-routine rate co tu so/mau so; reviewer doc lap, senior radiologist adjudicate.
6. **Can bao nhieu nguoi va de vo o dau?** Data steward, radiologist annotator, reviewer, senior adjudicator, data owner. De vo nhat la thieu thoi gian bac si; xu ly bang pilot, risk-based double review, audit va relabel co muc tieu.
