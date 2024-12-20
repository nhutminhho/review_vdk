# Bài Tập 5: Tạo Đồng Hồ Đếm Giây Sử Dụng Timer và Ngắt

## 1. Mô Tả Bài Tập
Thiết kế một đồng hồ đếm giây sử dụng Timer0 và ngắt trên vi điều khiển PIC16F877A. Hiển thị số giây đếm được lên LED 7 đoạn và LED đơn. Sử dụng một nút nhấn (SW) để tạm dừng và tiếp tục đồng hồ.

## 2. Yêu Cầu
- Sử dụng Timer0 để tạo ngắt định kỳ mỗi 1 giây
- Đếm từ 00 đến 59 giây và sau đó quay lại 00
- Hiển thị số giây lên LED 7 đoạn và LED đơn:
  - LED 7 đoạn hiển thị số giây (từ 00 đến 59)
  - LED đơn (D1) sẽ bật khi giây là số chẵn và tắt khi giây là số lẻ
- Sử dụng một nút nhấn (SW) để tạm dừng và tiếp tục đồng hồ

## 3. Phần Cứng Cần Chuẩn Bị
- Vi điều khiển PIC16F877A
- Timer0 của PIC16F877A
- Một nút nhấn (SW) và điện trở kéo lên (10kΩ)
- LED 7 đoạn (Common Cathode hoặc Common Anode) và điện trở giới hạn (220Ω)
- Một LED đơn (D1) và điện trở giới hạn (220Ω)
- Dây nối và mạch breadboard

## 4. Sơ Đồ Mạch
```
          +5V
           |
         [10kΩ]
           |
         SW -|-- GND
           |
         PIN_B0

        PORTD
        | | | | | | |
        a b c d e f g
        | | | | | | |
     [220Ω][220Ω][220Ω][220Ω][220Ω][220Ω][220Ω]
        | | | | | | |
       GND GND GND GND GND GND GND

        PORTC
          |
         D1
          |
        [220Ω]
          |
         GND
```

### Giải Thích Sơ Đồ Mạch

#### Nút Nhấn (SW)
- Một chân của nút nhấn được nối với nguồn 5V thông qua điện trở kéo lên 10kΩ
- Chân còn lại nối với GND và PIN_B0 của PORTB
- Khi nút được nhấn, PIN_B0 sẽ được kéo xuống mức thấp (GND)

#### LED 7 Đoạn (D1)
- Các chân a-g của LED 7 đoạn (Common Cathode) được kết nối với các chân PORTD0-PORTD6 thông qua điện trở giới hạn 220Ω
- Chân cathode của LED 7 đoạn nối với GND

#### LED Đơn (D1)
- Chân dương của LED đơn được nối với chân C0 của PORTC thông qua điện trở 220Ω
- Chân cathode nối với GND

## 5. Chương Trình Mẫu

```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT, NODEBUG, NOLVP
#use delay(clock=8M)
#include <lcd.c>

// Định nghĩa chân nút nhấn và LED đơn
#define SW PIN_B0
#define LED_D1 PIN_C0

// Mã 7-segment cho các chữ số 0-9 (Common Cathode)
const int8 LED7S[10] = {
    0x3F, // 0: a b c d e f
    0x06, // 1: b c
    0x5B, // 2: a b d e g
    0x4F, // 3: a b c d g
    0x66, // 4: b c f g
    0x6D, // 5: a c d f g
    0x7D, // 6: a c d e f g
    0x07, // 7: a b c
    0x7F, // 8: a b c d e f g
    0x6F  // 9: a b c d f g
};

volatile int16 seconds = 0;
volatile int8 pause = 0;

// Khai báo ngắt Timer0
#INT_TIMER0
void timer0_isr() {
    if(!pause) {
        seconds++;
        if(seconds >= 60) {
            seconds = 0;
        }
    }
    set_timer0(6); // Đặt lại Timer0 để ngắt tiếp theo
}

void main() {
    // Cấu hình PORTD là OUTPUT cho LED 7-segment
    set_tris_d(0x00);
    // Cấu hình PORTC là OUTPUT cho LED đơn
    set_tris_c(0x00);
    // Cấu hình PORTB là INPUT cho nút nhấn
    set_tris_b(0xFF);
    // Tắt tất cả LED ban đầu
    output_d(0x00);
    output_low(LED_D1);
    
    // Cấu hình Timer0
    setup_timer_0(RTCC_INTERNAL | RTCC_DIV_256);
    set_timer0(6); // Giá trị khởi tạo để tạo ngắt mỗi ~1 giây
    enable_interrupts(INT_TIMER0);
    enable_interrupts(GLOBAL);
    
    while(TRUE) {
        // Kiểm tra nút nhấn để tạm dừng/tiếp tục đồng hồ
        if(!input(SW)) { // Nếu nút nhấn được nhấn
            delay_ms(50); // Debounce
            if(!input(SW)) { // Xác nhận nút nhấn vẫn được nhấn
                pause = !pause; // Thay đổi trạng thái pause
                while(!input(SW)); // Chờ nút nhả
                delay_ms(50);
            }
        }
        
        // Tách giây thành chữ số chục và đơn vị
        int8 tens = seconds / 10;
        int8 units = seconds % 10;
        
        // Hiển thị chữ số chục trên PORTD
        output_d(LED7S[tens]);
        delay_ms(5);
        // Hiển thị chữ số đơn vị trên PORTD
        output_d(LED7S[units]);
        delay_ms(5);
        
        // Điều khiển LED đơn D1 dựa trên số giây
        if(seconds % 2 == 0) { // Số giây là số chẵn
            output_high(LED_D1);
        }
        else { // Số giây là số lẻ
            output_low(LED_D1);
        }
    }
}
```

## 6. Giải Thích Chi Tiết Chương Trình

### Khởi Tạo
- `set_tris_d(0x00);`: Thiết lập PORTD0-PORTD6 là OUTPUT cho LED 7 đoạn
- `set_tris_c(0x00);`: Thiết lập chân C0 của PORTC là OUTPUT cho LED đơn D1
- `set_tris_b(0xFF);`: Thiết lập PORTB là INPUT cho nút nhấn SW
- `output_d(0x00);` và `output_low(LED_D1);`: Tắt tất cả LED ban đầu

### Timer0 và Ngắt
- `setup_timer_0(RTCC_INTERNAL | RTCC_DIV_256);`: Cấu hình Timer0 sử dụng nguồn xung nội bộ và chia tần số 256
- `set_timer0(6);`: Đặt giá trị khởi tạo cho Timer0
- `enable_interrupts(INT_TIMER0);` và `enable_interrupts(GLOBAL);`: Kích hoạt ngắt Timer0 và ngắt toàn cục

### Hàm Ngắt timer0_isr()
- Kiểm tra trạng thái pause
- Nếu không ở trạng thái tạm dừng, tăng biến seconds và xử lý reset
- Đặt lại Timer0 cho ngắt tiếp theo

### Vòng Lặp Chính (while(TRUE))

#### Kiểm Tra Nút Nhấn
- Xử lý debounce khi nút được nhấn
- Đảo trạng thái pause khi nút được nhấn và nhả ra

#### Hiển Thị Số Giây
- Tách số giây thành chữ số chục và đơn vị
- Hiển thị lần lượt trên LED 7 đoạn với độ trễ nhỏ

#### Điều Khiển LED Đơn D1
- Bật LED khi số giây là số chẵn
- Tắt LED khi số giây là số lẻ

## 7. Thang Điểm
- Cấu hình Timer0 và ngắt: 3 điểm
- Viết hàm đếm giây và cập nhật hiển thị: 5 điểm
- Xử lý tạm dừng đồng hồ bằng nút nhấn: 2 điểm

## 8. Lưu Ý
- Đảm bảo cấu hình Timer0 chính xác để tạo ngắt mỗi 1 giây
- Xử lý debounce cho nút nhấn để tránh đọc sai tín hiệu
- Kiểm tra độ chính xác của thời gian đếm
- Tối ưu hóa code để giảm thời gian xử lý trong ngắt