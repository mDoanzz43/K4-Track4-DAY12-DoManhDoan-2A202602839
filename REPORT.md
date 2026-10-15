# REPORT - Lab Day 12

| | |
|---|---|
| Họ và tên | Đỗ Mạnh Đoàn |
| MSSV | 202602839 |
| GitHub | mDoanzz43 |
| Seed chia du lieu / seed mo hinh | 6347 / 661 |
| Số seed huấn luyện | 5 seed cho mô hình gốc; 3 seed cho mô hình sau sửa |

## Ket qua cuoi

| Bo dieu khien | L2 vong ho (m) | Va cham vong kin tren log (%) | Quyet dinh cong |
|---|---:|---:|---|
| constant-velocity | 1.252 | 19.60 | BLOCK |
| IDM | 1.533 | 0.07 | baseline |
| EgoMLP | 0.762 +/- 0.002 | 13.84-30.94 | BLOCK |
| PercMLP | 0.622 +/- 0.001 | 0.59-1.21 | BLOCK |
| PercMLP+ | 0.814 +/- 0.019 | 0.28-0.31 | RELEASE |

## Câu 1 - Dữ liệu

`data_audit.json` co 992,679 dong, 9,993 quy dao tho; 383 ban trung hoan toan bi loai, con 9,610 quy dao. Cot `Acc_LV` gan nhu khong tuong quan voi dao ham toc do (`r=0.045`), trong khi `Acc_FAV` co `r=0.929`, nen toi suy gia toc xe dan tu `Speed_LV` da lam muot. Neu chia theo frame, cac frame ke nhau cua cung mot quy dao, hoac hai quy dao trung, co the nam o ca train va test va gay ro ri. Vi vay du lieu duoc khu trung truoc va chia theo toan bo quy dao.

## Câu 2 - Phát lại

Hai lan chay cung code khop tung bit tren 600/600 doan. Refactor float32 khac bit o 600 doan, nhung 0 doan vuot `atol=rtol=1e-3`; sai khac lon nhat chi `1.22e-4`, nen so sanh dung sai moi phan biet nhieu so voi doi hanh vi. AEB lam lech 7 doan. O doan 55, chi frame 98-101 lech: TTC `1.431-1.490 s` nho hon nguong 1.5 s, nen lenh IDM khoang `-5.44..-5.91` doi thanh `-6.0 m/s^2`. Fastest replay bao 0 tre/rot, con realtime co 586 frame tre, 581 frame rot va 381/600 doan thay doi, vi fastest khong mo phong deadline va viec giu lenh cu.

## Câu 3 - Kịch bản

Luoi dung van toc ego `12 m/s`, gap `2.76-59.31 m`, closing speed `0-12 m/s`, tong 2,700 o. Vung bat kha thi tang tu `2.22%` khi tre 0 s len `7.44%` khi tre 0.6 s; full-brake khop bien vat ly voi 0 o lech xa frontier. IDM va cham dung cac vung nay, nen attributable area la 0% o ca hai muc tre. Log co 614 ca xe dan phanh; gioi han phanh nguoi lai p99.9 chi `3.625 m/s^2`, trong khi `8.56%` luoi can phanh manh hon, cho thay log khong phu day du vung kho.

## Câu 4 - Vòng hở và vòng kín

Thu hang L2 la PercMLP (0.622), EgoMLP (0.762), constant-velocity (1.252), IDM (1.533); thu hang va cham vong kin lai la IDM (0.07%), PercMLP (trung binh 1.00%), constant-velocity (19.60%), EgoMLP (23.45%). Bon cap dao thu hang la const-vel/IDM, const-vel/EgoMLP, IDM/EgoMLP va IDM/PercMLP. IDM khong bat chuoc chinh xac quy dao nguoi lai nen L2 cao, nhung dung gap va relative speed de phan hoi on dinh. EgoMLP chi thay lich su toc do ego: no du doan trung binh kha tot nhung khong thay xe dan phanh/cut-in, nen sai so nho tich luy trong vong kin va gay va cham.

## Câu 5 - Một lần chạy chưa đủ

EgoMLP dao dong rat lon qua 5 seed: ty le va cham `13.84-30.94%`, chenh `17.10` diem phan tram. O worst seed, 892/2,883 quy dao va cham (`30.94%`) va Wilson 95% la `[29.28%, 32.65%]`; day la bat dinh huan luyen, khong phai chi nhieu lay mau. Neu chi chay seed tot nhat, co the bao nham 13.84% va che hon mot nua rui ro thay o seed xau. PercMLP on dinh hon (`0.59-1.21%`) nhung worst-seed CI van `[0.87%, 1.68%]`, du de rot R2 so voi IDM 0.07% + 1 diem.

## Câu 6 - Tác nhân phản ứng

Nguoi lai co headway trung vi `1.856 s`; IDM rear agent duoc hieu chinh tai `T=1.4 s` voi W1=0.351, tot hon tac nhan khong phan ung W1=0.890. Tai ego delay 0.6 s, rear replay/reactive cho IDM la `75.74%/7.44%`, PercMLP `77.81%/24.96%`, va PercMLP+ `75.30%/7.81%`. Rear replay tao `68.30%` rear collision cho IDM vi xe sau giu quy dao cu va khong biet ego da phanh; rear reactive dua rear collision ve 0%. Tuy vay PercMLP van cao hon IDM khi the gioi phan ung, nen viec sua tac nhan loai duoc va cham gia nhung khong xoa loi controller.

## Câu 7 - Cổng phát hành và sửa

PercMLP bi chan boi R2 (worst `1.21%`, upper 95% `1.68%`), R3 (braking slice `4.88%`, upper `9.33%`), R4 (avoidable cut-in `17.52%`) va R6 (reactive-world `24.96%`). Nguyen nhan la log thieu cut-in kho va imitation error dua model den trang thai chua thay. Toi them 196,870 window tu 20,000 cut-in va hai vong DAgger gan nhan IDM tai trang thai model tu ghe tham. PercMLP+ con `0.28-0.31%` va cham, `0.37%` attributable grid, qua ca 7 rule; o 20 m/s ngoai mien train no con `0.67-0.89%`, so voi PercMLP `21.44-30.30%`. Tuy nhien L2 tang `0.622 -> 0.814 m` va bang chung chi cho thay khong kem IDM theo margin cua gate, khong chung minh tot hon IDM; nen chi phat hanh gioi han sau them test thuc dia, ODD va shadow mode.

## Câu 8 - Bài lab này không chứng minh được gì

Thu nhat, mo phong chi co chuyen dong doc 1-D, khong co lane, cua xe, nguoi di bo hay dong luc hoc day du, nen khong dai dien cho giao thong that. Thu hai, lead trong log bi replay va cam bien duoc mo phong don gian; ket qua khong bao phu latency, occlusion, packet loss va loi nhan thuc thuc. Thu ba, 9,610 quy dao Argoverse la mot phan bo han che va con thieu vung phanh/cut-in manh, nen khong chung minh duoc safety ngoai ODD. Cuoi cung, gate dung cac margin quy uoc va so lan thu huu han, nen RELEASE khong phai chung nhan an toan. Buoc tiep theo la scenario coverage co he thong, sensor/HIL, nhieu tac nhan 2-D, du lieu fleet doc lap, shadow deployment va giam sat failure rate voi khoang tin cay.
