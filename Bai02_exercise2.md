# Bài Tập 2: Điều Khiển LED 7 Đoạn và LED Đơn Bằng Nút Nhấn

## 1. Mô Tả Bài Tập
Sử dụng vi điều khiển PIC16F877A để điều khiển một LED 7 đoạn và một LED đơn thông qua nút nhấn. Mỗi lần nhấn nút, LED 7 đoạn sẽ hiển thị một số tăng dần từ 0 đến 9, đồng thời LED đơn sẽ bật/tắt theo số lần nhấn.

## 2. Yêu Cầu
- Một nút nhấn (SW) kết nối với PIN_B0
- Một LED 7 đoạn (D1) kết nối với PORTD0-PORTD6 (a-g)
- Một LED đơn (D2) kết nối với PIN_C0
- Mỗi lần nhấn nút:
  - Số hiển thị trên LED 7 đoạn tăng lên 1 (từ 0 đến 9, sau đó quay lại 0)
  - LED đơn bật nếu số hiển thị là số lẻ, tắt nếu số chẵn
- Khi số hiển thị đạt 9 và nhấn tiếp sẽ quay lại 0

## 3. Phần Cứng Cần Chuẩn Bị
- Vi điều khiển PIC16F877A
- Một nút nhấn (SW) và điện trở kéo lên (10kΩ)
- Một LED 7 đoạn (Common Cathode hoặc Common Anode) và điện trở giới hạn (220Ω)
- Một LED đơn (D2) và điện trở giới hạn (220Ω)
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
         D2
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

#### LED Đơn (D2)
- Chân dương của LED đơn được nối với chân C0 của PORTC thông qua điện trở 220Ω
- Chân cathode nối với GND

## 5. Chương Trình Mẫu

```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT, NODEBUG, NOLVP
#use delay(clock=8M)

// Định nghĩa chân nút nhấn và LED đơn
#define SW PIN_B0
#define LED_D2 PIN_C0

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

void main() {
    // Cấu hình PORTD là OUTPUT cho LED 7-segment
    set_tris_d(0x00);
    // Cấu hình PORTC là OUTPUT cho LED đơn
    set_tris_c(0x00);
    // Cấu hình PORTB là INPUT cho nút nhấn
    set_tris_b(0xFF);
    // Tắt tất cả LED ban đầu
    output_d(0x00);
    output_low(LED_D2);
    
    unsigned int8 number = 0;
    unsigned int16 debounce_time = 50; // Thời gian debounce (ms)
    
    while(TRUE) {
        // Kiểm tra nút nhấn
        if(!input(SW)) { // Nếu nút nhấn được nhấn
            delay_ms(debounce_time); // Debounce
            if(!input(SW)) { // Xác nhận nút nhấn vẫn được nhấn
                number++;
                if(number > 9) {
                    number = 0;
                }
                output_d(LED7S[number]); // Hiển thị số trên LED 7 đoạn
                
                // Điều khiển LED đơn
                if(number % 2 != 0) { // Số lẻ
                    output_high(LED_D2);
                }
                else { // Số chẵn
                    output_low(LED_D2);
                }
                
                while(!input(SW)); // Chờ nút nhả
                delay_ms(debounce_time);
            }
        }
    }
}
```

## 6. Giải Thích Chi Tiết Chương Trình

### Khởi Tạo
- `set_tris_d(0x00);`: Thiết lập PORTD0-PORTD6 là OUTPUT cho LED 7 đoạn
- `set_tris_c(0x00);`: Thiết lập chân C0 của PORTC là OUTPUT cho LED đơn D2
- `set_tris_b(0xFF);`: Thiết lập chân B0 của PORTB là INPUT cho nút nhấn SW
- `output_d(0x00);` và `output_low(LED_D2);`: Tắt tất cả LED ban đầu

### Mảng LED7S
- Mảng chứa mã nhị phân để hiển thị các chữ số từ 0 đến 9 trên LED 7 đoạn
- Mỗi giá trị đại diện cho trạng thái bật/tắt của các chân a-g

### Vòng Lặp Chính (while(TRUE))

#### Kiểm Tra Nút Nhấn
- `if(!input(SW))`: Kiểm tra nếu nút nhấn được nhấn
- `delay_ms(debounce_time);`: Chờ thời gian debounce
- `if(!input(SW))`: Xác nhận nút nhấn vẫn được nhấn

#### Tăng Số Hiển Thị
- `number++;`: Tăng biến number lên 1
- `if(number > 9) { number = 0; }`: Reset về 0 nếu vượt quá 9
- `output_d(LED7S[number]);`: Hiển thị số number lên LED 7 đoạn

#### Điều Khiển LED Đơn D2
- `if(number % 2 != 0)`: Nếu number là số lẻ, bật LED D2
- `else`: Nếu number là số chẵn, tắt LED D2

#### Chờ Nút Nhả
- `while(!input(SW));`: Chờ cho đến khi nút nhả
- `delay_ms(debounce_time);`: Thêm thời gian debounce sau khi nút nhả

## 7. Thang Điểm
- Hiểu và cấu hình các chân I/O: 2 điểm
- Viết hàm kiểm tra nút nhấn và debounce: 3 điểm
- Điều khiển LED 7 đoạn và LED đơn theo yêu cầu: 4 điểm
- Tối ưu và xử lý lỗi: 1 điểm