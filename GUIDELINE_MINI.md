# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe chở người cỡ nhỏ: sedan, hatchback, SUV, crossover, taxi, xe bán tải (pickup) | Thân thấp, nắp capo rõ. Pickup có thùng hở phía sau vẫn là `car`, không phải `truck`. Không gán mô tô/xe máy |
| `truck`  | Xe tải chở hàng: xe tải thùng, xe ben, xe bồn, xe đầu kéo (có/không rơ-moóc) | Cabin tách biệt với thùng hàng lớn hoặc khung kéo, bánh to, thường nhiều trục. Cabin nhỏ/pickup → `car`; van chở hàng liền khối → `van` |
| `bus`  | Xe chở khách nhiều chỗ: xe buýt thành phố, xe khách, minibus chở khách | Thân dài, cao, nhiều cửa sổ hai bên, cửa lên xuống lớn. Van chở người cỡ nhỏ (kiểu Ford Transit/Hiace) → `van` |
| `van`  | Xe van, minivan chở hàng hoặc chở người | Thân hộp liền khối (cabin và khoang sau chung một khối), cao hơn `car` nhưng ngắn hơn `bus`. Không có thùng hàng tách rời như `truck`. Ca mơ hồ van/bus/car → không tự chọn, ghi case ID và hỏi giảng viên |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: bốn cạnh box chạm đúng điểm ngoài cùng của phần thân xe nhìn thấy được (kể cả gương chiếu hậu, bánh xe); không lấy thêm mặt đường, bóng đổ, trời hay xe bên cạnh. Mỗi xe một box riêng, không gộp nhiều xe vào một box; hai xe chồng nhau thì vẽ hai box trên phần nhìn thấy của từng xe.
- Vật thể bị che khuất: vẫn gán box quanh **phần nhìn thấy được** nếu phần đó đủ để nhận diện chắc chắn lớp; không đoán/kéo box ra phần bị che. Nếu bị che đến mức không xác định được lớp thì không gán và ghi lại vào bảng ca mơ hồ bên dưới.
- Vật thể bị cắt bởi rìa ảnh: cạnh box chạm đúng rìa ảnh (tọa độ 0 hoặc 1), không đoán phần nằm ngoài khung; vẫn gán nếu phần còn lại nhận diện được lớp.
- Vật thể rất nhỏ/mơ hồ: chỉ gán khi xác định chắc là xe thuộc bốn lớp schema và nhận ra được lớp; nếu chỉ là mảnh rất nhỏ hoặc không chắc lớp thì **không tự đoán**

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_014.jpg` | Xe kéo cứu hộ | dán nhãn là truck| xác định là có khung kéo, cabin nhỏ|
| `drive_015.jpg` | Không xác định được xe car hay van do chỉ thấy ảnh mặt trước và mờ | không gán nhãn| vật thể mơ hồ không chắc chắn |
| `drive_008.jpg` | vật thể ở rìa | không gán nhãn | mặc dù có thể đoán là car nhưng không đủ chi tiết như mô tả schema |