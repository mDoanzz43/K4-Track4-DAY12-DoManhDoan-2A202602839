# REPORT - Lab Day 12

| | |
|---|---|
| Họ và tên | Đỗ Mạnh Đoan |
| MSSV | 202602839 |
| GitHub | https://github.com/mDoanzz43/K4-Track4-DAY12-DoManhDoan-2A202602839 |
| Seed chia dữ liệu / seed mô hình | 6347 / 661 |
| Số seed huấn luyện | 5 seed cho mô hình gốc; 3 seed cho mô hình sau sửa |

## Kết quả cuối

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---:|---:|---|
| constant-velocity | 1.252 | 19.60 | BLOCK |
| IDM | 1.533 | 0.07 | baseline |
| EgoMLP | 0.762 +/- 0.002 | 13.84-30.94 | BLOCK |
| PercMLP | 0.622 +/- 0.001 | 0.59-1.21 | BLOCK |
| PercMLP+ | 0.814 +/- 0.019 | 0.28-0.31 | RELEASE |

## Câu 1 - Dữ liệu

`data_audit.json` có 992,679 dòng, 9,993 quỹ đạo thô; 383 bản trùng hoàn toàn bị loại, còn 9,610 quỹ đạo. Cột `Acc_LV` gần như không tương quan với đạo hàm tốc độ (`r=0.045`), trong khi `Acc_FAV` có `r=0.929`, nên tôi suy gia tốc xe dẫn từ `Speed_LV` đã làm mượt. Nếu chia theo frame, các frame kề nhau của cùng một quỹ đạo, hoặc hai quỹ đạo trùng, có thể nằm ở cả train và test và gây rò rỉ. Vì vậy dữ liệu được khử trùng trước và chia theo toàn bộ quỹ đạo.

## Câu 2 - Phát lại

Hai lần chạy cùng code khớp từng bit trên 600/600 đoạn. Refactor float32 khác bit ở 600 đoạn, nhưng 0 đoạn vượt `atol=rtol=1e-3`; sai khác lớn nhất chỉ `1.22e-4`, nên so sánh dung sai mới phân biệt nhiễu số với đổi hành vi. AEB làm lệch 7 đoạn. Ở đoạn 55, chỉ frame 98-101 lệch: TTC `1.431-1.490 s` nhỏ hơn ngưỡng 1.5 s, nên lệnh IDM khoảng `-5.44..-5.91` đổi thành `-6.0 m/s^2`. Fastest replay báo 0 trễ/rớt, còn realtime có 586 frame trễ, 581 frame rớt và 381/600 đoạn thay đổi, vì fastest không mô phỏng deadline và việc giữ lệnh cũ.

## Câu 3 - Kịch bản

Lưới dùng vận tốc ego `12 m/s`, gap `2.76-59.31 m`, closing speed `0-12 m/s`, tổng 2,700 ô. Vùng bất khả thi tăng từ `2.22%` khi trễ 0 s lên `7.44%` khi trễ 0.6 s; full-brake khớp biên vật lý với 0 ô lệch xa frontier. IDM va chạm đúng các vùng này, nên attributable area là 0% ở cả hai mức trễ. Log có 614 ca xe dẫn phanh; giới hạn phanh người lái p99.9 chỉ `3.625 m/s^2`, trong khi `8.56%` lưới cần phanh mạnh hơn, cho thấy log không phủ đầy đủ vùng khó.

## Câu 4 - Vòng hở và vòng kín

Thứ hạng L2 là PercMLP (0.622), EgoMLP (0.762), constant-velocity (1.252), IDM (1.533); thứ hạng va chạm vòng kín lại là IDM (0.07%), PercMLP (trung bình 1.00%), constant-velocity (19.60%), EgoMLP (23.45%). Bốn cặp đảo thứ hạng là const-vel/IDM, const-vel/EgoMLP, IDM/EgoMLP và IDM/PercMLP. IDM không bắt chước chính xác quỹ đạo người lái nên L2 cao, nhưng dùng gap và relative speed để phản hồi ổn định. EgoMLP chỉ thấy lịch sử tốc độ ego: nó dự đoán trung bình khá tốt nhưng không thấy xe dẫn phanh/cut-in, nên sai số nhỏ tích lũy trong vòng kín và gây va chạm.

## Câu 5 - Một lần chạy chưa đủ

EgoMLP dao động rất lớn qua 5 seed: tỷ lệ va chạm `13.84-30.94%`, chênh `17.10` điểm phần trăm. Ở worst seed, 892/2,883 quỹ đạo va chạm (`30.94%`) và Wilson 95% là `[29.28%, 32.65%]`; đây là bất định huấn luyện, không phải chỉ nhiễu lấy mẫu. Nếu chỉ chạy seed tốt nhất, có thể báo nhầm 13.84% và che hơn một nửa rủi ro thấy ở seed xấu. PercMLP ổn định hơn (`0.59-1.21%`) nhưng worst-seed CI vẫn `[0.87%, 1.68%]`, đủ để rớt R2 so với IDM 0.07% + 1 điểm.

## Câu 6 - Tác nhân phản ứng

Người lái có headway trung vị `1.856 s`; IDM rear agent được hiệu chỉnh tại `T=1.4 s` với W1=0.351, tốt hơn tác nhân không phản ứng W1=0.890. Tại ego delay 0.6 s, rear replay/reactive cho IDM là `75.74%/7.44%`, PercMLP `77.81%/24.96%`, và PercMLP+ `75.30%/7.81%`. Rear replay tạo `68.30%` rear collision cho IDM vì xe sau giữ quỹ đạo cũ và không biết ego đã phanh; rear reactive đưa rear collision về 0%. Tuy vậy PercMLP vẫn cao hơn IDM khi thế giới phản ứng, nên việc sửa tác nhân loại được va chạm giả nhưng không xóa lỗi controller.

## Câu 7 - Cổng phát hành và sửa

PercMLP bị chặn bởi R2 (worst `1.21%`, upper 95% `1.68%`), R3 (braking slice `4.88%`, upper `9.33%`), R4 (avoidable cut-in `17.52%`) và R6 (reactive-world `24.96%`). Nguyên nhân là log thiếu cut-in khó và imitation error đưa model đến trạng thái chưa thấy. Tôi thêm 196,870 window từ 20,000 cut-in và hai vòng DAgger gán nhãn IDM tại trạng thái model tự ghé thăm. PercMLP+ còn `0.28-0.31%` va chạm, `0.37%` attributable grid, qua cả 7 rule; ở 20 m/s ngoài miền train nó còn `0.67-0.89%`, so với PercMLP `21.44-30.30%`. Tuy nhiên L2 tăng `0.622 -> 0.814 m` và bằng chứng chỉ cho thấy không kém IDM theo margin của gate, không chứng minh tốt hơn IDM; nên chỉ phát hành giới hạn sau thêm test thực địa, ODD và shadow mode.

## Câu 8 - Bài lab này không chứng minh được gì

Thứ nhất, mô phỏng chỉ có chuyển động dọc 1-D, không có lane, cua xe, người đi bộ hay động lực học đầy đủ, nên không đại diện cho giao thông thật. Thứ hai, lead trong log bị replay và cảm biến được mô phỏng đơn giản; kết quả không bao phủ latency, occlusion, packet loss và lỗi nhận thức thực. Thứ ba, 9,610 quỹ đạo Argoverse là một phân bố hạn chế và còn thiếu vùng phanh/cut-in mạnh, nên không chứng minh được safety ngoài ODD. Cuối cùng, gate dùng các margin quy ước và số lần thử hữu hạn, nên RELEASE không phải chứng nhận an toàn. Bước tiếp theo là scenario coverage có hệ thống, sensor/HIL, nhiều tác nhân 2-D, dữ liệu fleet độc lập, shadow deployment và giám sát failure rate với khoảng tin cậy.