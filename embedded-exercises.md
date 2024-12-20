# Bài Tập Lập Trình Vi Điều Khiển PIC16F877A

## Bài Tập 1: Điều Khiển 8 LED Đơn Bằng Nút Nhấn

### Mô Tả
Sử dụng vi điều khiển PIC16F877A để điều khiển 8 LED đơn thông qua một nút nhấn. Mỗi lần nhấn nút, một LED sẽ bật lên theo thứ tự từ D1 đến D8. Khi tất cả LED đã được bật, tiếp tục nhấn nút sẽ tắt LED từ D1 đến D8 theo cùng thứ tự.

### Yêu Cầu
1. Có một nút nhấn (SW) kết nối với PIN_B0
2. Có 8 LED đơn (D1-D8) kết nối với PORTC0-PORTC7
3. Mỗi lần nhấn nút, bật LED tiếp theo trong chuỗi từ D1 đến D8
4. Khi tất cả LED đã được bật, tiếp tục nhấn nút sẽ tắt LED từ D1 đến D8 theo thứ tự
5. Khi tất cả LED đã được tắt, tiếp tục nhấn nút sẽ bắt đầu quá trình từ đầu

## Bài Tập 2: Điều Khiển LED 7 Đoạn và LED Đơn Bằng Nút Nhấn

### Mô Tả
Sử dụng vi điều khiển PIC16F877A để điều khiển một LED 7 đoạn và một LED đơn thông qua nút nhấn. Mỗi lần nhấn nút, LED 7 đoạn sẽ hiển thị một số tăng dần từ 0 đến 9, đồng thời LED đơn sẽ bật/tắt theo số lần nhấn.

### Yêu Cầu
1. Có một nút nhấn (SW) kết nối với PIN_B0
2. Có một LED 7 đoạn (D1) kết nối với PORTD0-PORTD6 (a-g)
3. Có một LED đơn (D2) kết nối với PIN_C0
4. Mỗi lần nhấn nút:
   - Số hiển thị trên LED 7 đoạn tăng lên 1 (từ 0 đến 9, sau đó quay lại 0)
   - LED đơn bật nếu số hiển thị là số lẻ, tắt nếu số chẵn
5. Khi số hiển thị đạt 9 và nhấn tiếp sẽ quay lại 0

## Bài Tập 3: Hiển Thị Thông Báo Trên LCD Khi Nhấn Nút

### Mô Tả
Sử dụng vi điều khiển PIC16F877A để hiển thị thông báo trên màn hình LCD khi nhấn nút. Mỗi lần nhấn nút, LCD sẽ hiển thị một thông điệp khác nhau theo thứ tự.

### Yêu Cầu
1. Có một nút nhấn (SW) kết nối với PIN_B0
2. Có một màn hình LCD 16x2 kết nối với vi điều khiển
3. Mỗi lần nhấn nút, hiển thị một thông điệp khác nhau trên LCD:
   - Lần nhấn 1: "Chao Mung!"
   - Lần nhấn 2: "Hoc Vi Du!"
   - Lần nhấn 3: "Ve Nha!"
   - Lần nhấn 4: Quay lại thông điệp đầu tiên
4. Khi không có nút nhấn nào được nhấn, giữ thông điệp hiện tại trên LCD

## Bài Tập 4: Đo Nhiệt Độ Sử dụng ADC và Hiển Thị Trên LED Đơn

### Mô Tả
Sử dụng bộ chuyển đổi ADC của vi điều khiển PIC16F877A để đo nhiệt độ từ cảm biến LM35 và hiển thị giá trị nhiệt độ trên 8 LED đơn dưới dạng nhị phân.

### Yêu Cầu
1. Kết nối cảm biến LM35 vào chân AN0 của vi điều khiển PIC16F877A
2. Sử dụng ADC để đọc giá trị nhiệt độ và chuyển đổi sang độ C
3. Hiển thị giá trị nhiệt độ lên 8 LED đơn (D1-D8) dưới dạng nhị phân (0-255°C)
4. Mỗi lần giá trị nhiệt độ thay đổi, cập nhật LED để hiển thị giá trị mới

## Bài Tập 5: Tạo Đồng Hồ Đếm Giây Sử Dụng Timer và Ngắt

### Mô Tả
Thiết kế một đồng hồ đếm giây sử dụng Timer0 và ngắt trên vi điều khiển PIC16F877A. Hiển thị số giây đếm được lên LED 7 đoạn và LED đơn.

### Yêu Cầu
1. Sử dụng Timer0 để tạo ngắt định kỳ mỗi 1 giây
2. Đếm từ 00 đến 59 giây và sau đó quay lại 00
3. Hiển thị số giây lên LED 7 đoạn và LED đơn:
   - LED 7 đoạn hiển thị số giây (từ 00 đến 59)
   - LED đơn (D1) sẽ bật khi giây là số chẵn và tắt khi giây là số lẻ
4. Sử dụng một nút nhấn (SW) để tạm dừng và tiếp tục đồng hồ