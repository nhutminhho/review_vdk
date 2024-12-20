# Bài Tập 4: Đo Nhiệt Độ Sử dụng ADC và Hiển Thị Trên LED Đơn

## 1. Mô Tả Bài Tập
Sử dụng bộ chuyển đổi ADC của vi điều khiển PIC16F877A để đo nhiệt độ từ cảm biến LM35 và hiển thị giá trị nhiệt độ trên 8 LED đơn dưới dạng nhị phân.

## 2. Yêu Cầu
- Kết nối cảm biến LM35 vào chân AN0 của vi điều khiển PIC16F877A
- Sử dụng ADC để đọc giá trị nhiệt độ và chuyển đổi sang độ C
- Hiển thị giá trị nhiệt độ lên 8 LED đơn (D1-D8) dưới dạng nhị phân (0-255°C)
- Mỗi lần giá trị nhiệt độ thay đổi, cập nhật LED để hiển thị giá trị mới

## 3. Phần Cứng Cần Chuẩn Bị
- Vi điều khiển PIC16F877A
- Cảm biến nhiệt độ LM35
- 8 LED đơn (D1-D8) và điện trở giới hạn (220Ω mỗi LED)
- Dây nối và mạch breadboard

## 4. Sơ Đồ Mạch
```
        LM35
        +-----+
        |     |
      Vcc     GND
        |      |
        +------ OUT ---- AN0 (Chân A0 của PIC16F877A)

      PORTC
      | | | | | | | |
     D1 D2 D3 D4 D5 D6 D7 D8
      | | | | | | | |
   [220Ω][220Ω][220Ω][220Ω][220Ω][220Ω][220Ω][220Ω]
      | | | | | | | |
     GND GND GND GND GND GND GND GND
```

### Giải Thích Sơ Đồ Mạch

#### Cảm Biến Nhiệt Độ (LM35)
- Vcc: Kết nối với nguồn 5V
- GND: Kết nối với GND
- OUT: Kết nối với chân AN0 (A0) của PIC16F877A để đọc giá trị analog

#### LED Đơn (D1-D8)
- Mỗi LED được nối với một chân của PORTC (C0-C7) thông qua điện trở giới hạn 220Ω
- Chân cathode của mỗi LED nối với GND

## 5. Chương Trình Mẫu

```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT, NODEBUG, NOLVP
#use delay(clock=8M)

void main() {
    // Cấu hình PORTC là OUTPUT cho 8 LED
    set_tris_c(0x00);
    // Cấu hình PORTA là INPUT cho ADC
    set_tris_a(0xFF);
    // Tắt tất cả LED ban đầu
    output_c(0x00);
    
    // Cấu hình ADC
    setup_adc_ports(AN0);          // Sử dụng chân AN0
    setup_adc(ADC_CLOCK_DIV_32);   // Thiết lập tốc độ ADC
    set_adc_channel(0);            // Chọn kênh AN0
    delay_us(20);                  // Đợi ổn định
    
    while(TRUE) {
        // Đọc giá trị ADC
        int16 adc_value = read_adc();
        
        // Chuyển đổi giá trị ADC thành nhiệt độ
        // Công thức: T (°C) = (ADC_value * 5V / 1023) / 0.01V/°C
        float temperature = (adc_value * 5.0 / 1023.0) / 0.01;
        
        // Giới hạn nhiệt độ trong khoảng 0-255°C
        if(temperature > 255.0) {
            temperature = 255.0;
        }
        else if(temperature < 0.0) {
            temperature = 0.0;
        }
        
        // Chuyển đổi nhiệt độ thành giá trị nhị phân
        unsigned int8 binary_value = (unsigned int8)temperature;
        
        // Hiển thị giá trị nhị phân lên LED đơn
        output_c(binary_value);
        
        delay_ms(500); // Độ trễ trước khi đọc giá trị mới
    }
}
```

## 6. Giải Thích Chi Tiết Chương Trình

### Khởi Tạo
- `set_tris_c(0x00);`: Thiết lập PORTC0-PORTC7 là OUTPUT để kết nối với LED D1-D8
- `set_tris_a(0xFF);`: Thiết lập PORTA là INPUT để đọc giá trị từ LM35
- `output_c(0x00);`: Tắt tất cả LED trên PORTC ban đầu

### Cấu Hình ADC
- `setup_adc_ports(AN0);`: Chỉ sử dụng chân AN0 cho ADC
- `setup_adc(ADC_CLOCK_DIV_32);`: Thiết lập tốc độ ADC phù hợp
- `set_adc_channel(0);`: Chọn kênh AN0 để đọc dữ liệu
- `delay_us(20);`: Đợi 20 microseconds để ADC ổn định

### Vòng Lặp Chính (while(TRUE))

#### Đọc Giá Trị ADC
- `adc_value = read_adc();`: Đọc giá trị ADC từ chân AN0
- Giá trị này nằm trong khoảng 0-1023 (10-bit ADC)

#### Chuyển Đổi Giá Trị ADC thành Nhiệt Độ
- Công thức tính dựa trên độ nhạy của LM35 (10mV/°C)
- `temperature = (adc_value * 5.0 / 1023.0) / 0.01;`

#### Giới Hạn Nhiệt Độ
- Đảm bảo giá trị nhiệt độ nằm trong khoảng 0-255°C
- Xử lý các trường hợp vượt ngưỡng

#### Hiển Thị Nhiệt Độ
- `binary_value = (unsigned int8)temperature;`: Chuyển đổi nhiệt độ thành giá trị nhị phân
- `output_c(binary_value);`: Gửi giá trị nhị phân lên PORTC

#### Độ Trễ
- `delay_ms(500);`: Đợi 500ms trước khi đọc giá trị mới
- Giảm tải cho hệ thống và tránh cập nhật quá nhanh

## 7. Thang Điểm
- Kết nối và cấu hình ADC với LM35: 3 điểm
- Đọc và chuyển đổi giá trị nhiệt độ: 4 điểm
- Hiển thị giá trị nhiệt độ trên LED đơn dưới dạng nhị phân: 3 điểm

## 8. Lưu Ý
- Kiểm tra điện áp nguồn cấp cho LM35 phải ổn định
- Đảm bảo độ chính xác của ADC bằng cách cấu hình đúng tần số lấy mẫu
- Xử lý nhiễu và lọc tín hiệu nếu cần thiết
- Kiểm tra sự hoạt động của các LED để đảm bảo hiển thị chính xác