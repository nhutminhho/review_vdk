# Hướng Dẫn Lập Trình Vi Điều Khiển PIC16F877A

## Bài Tập 1: Điều Khiển 8 LED Đơn Bằng Nút Nhấn

### 1. Mô Tả Bài Tập
Sử dụng vi điều khiển PIC16F877A để điều khiển 8 LED đơn thông qua một nút nhấn. Mỗi lần nhấn nút, một LED sẽ bật lên theo thứ tự từ D1 đến D8. Khi tất cả LED đã được bật, tiếp tục nhấn nút sẽ tắt LED từ D1 đến D8 theo cùng thứ tự.

### 2. Yêu Cầu
- Một nút nhấn (SW) kết nối với PIN_B0
- 8 LED đơn (D1-D8) kết nối với PORTC0-PORTC7
- Mỗi lần nhấn nút, bật LED tiếp theo trong chuỗi từ D1 đến D8
- Khi tất cả LED đã được bật, tiếp tục nhấn nút sẽ tắt LED từ D1 đến D8 theo thứ tự
- Khi tất cả LED đã được tắt, tiếp tục nhấn nút sẽ bắt đầu quá trình từ đầu

### 3. Phần Cứng Cần Chuẩn Bị
- Vi điều khiển PIC16F877A
- Một nút nhấn (SW) và điện trở kéo lên (10kΩ)
- 8 LED đơn (D1-D8) và điện trở giới hạn (220Ω mỗi LED)
- Dây nối và mạch breadboard

### 4. Sơ Đồ Mạch
```
          +5V
           |
         [10kΩ]
           |
         SW -|-- GND
           |
         PIN_B0

        PORTC
        | | | | | | | |
       D1 D2 D3 D4 D5 D6 D7 D8
        | | | | | | | |
     [220Ω][220Ω][220Ω][220Ω][220Ω][220Ω][220Ω][220Ω]
        | | | | | | | |
       GND GND GND GND GND GND GND GND
```

#### Giải Thích Sơ Đồ Mạch

##### Nút Nhấn (SW)
- Một chân của nút nhấn được nối với nguồn 5V thông qua điện trở kéo lên 10kΩ
- Chân còn lại nối với GND và PIN_B0 của PORTB
- Khi nút được nhấn, PIN_B0 sẽ được kéo xuống mức thấp (GND)

##### LED Đơn (D1-D8)
- Mỗi LED được kết nối với một chân của PORTC (C0-C7) thông qua điện trở giới hạn 220Ω 
- Chân cathode của mỗi LED nối với GND

### 5. Chương Trình Mẫu

```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT, NODEBUG, NOLVP
#use delay(clock=8M)

// Định nghĩa chân nút nhấn
#define SW PIN_B0

void main() {
    // Cấu hình PORTC là OUTPUT cho 8 LED
    set_tris_c(0x00);
    // Cấu hình PORTB là INPUT cho nút nhấn
    set_tris_b(0xFF);
    // Tắt tất cả LED ban đầu
    output_c(0x00);
    
    unsigned int8 state = 0; // 0: Bật, 1: Tắt
    unsigned int8 current_led = 0;
    unsigned int16 debounce_time = 50; // Thời gian debounce (ms)
    
    while(TRUE) {
        // Kiểm tra nút nhấn
        if(!input(SW)) { // Nếu nút nhấn được nhấn
            delay_ms(debounce_time); // Debounce
            if(!input(SW)) { // Xác nhận nút nhấn vẫn được nhấn
                if(state == 0) { // Chế độ bật LED
                    if(current_led < 8) {
                        output_high(PIN_C0 + current_led);
                        current_led++;
                    }
                    else {
                        state = 1; // Chuyển sang chế độ tắt LED
                        current_led = 7;
                    }
                }
                else { // Chế độ tắt LED
                    if(current_led < 8) {
                        output_low(PIN_C0 + current_led);
                        current_led--;
                    }
                    else {
                        state = 0; // Chuyển sang chế độ bật LED
                        current_led = 0;
                    }
                }
                while(!input(SW)); // Chờ nút nhả
                delay_ms(debounce_time);
            }
        }
    }
}
```

### 6. Giải Thích Chi Tiết Chương Trình

#### Khởi Tạo
- `set_tris_c(0x00);`: Thiết lập tất cả các chân của PORTC (C0-C7) là OUTPUT để kết nối với LED D1-D8
- `set_tris_b(0xFF);`: Thiết lập chân B0 của PORTB là INPUT để đọc trạng thái của nút nhấn SW
- `output_c(0x00);`: Tắt tất cả LED trên PORTC ban đầu

#### Biến
- `state`: Biến trạng thái hiện tại, 0 là chế độ bật LED, 1 là chế độ tắt LED
- `current_led`: Chỉ số LED hiện tại đang được bật/tắt (0-7 tương ứng với D1-D8)
- `debounce_time`: Thời gian chờ (ms) để xử lý hiện tượng nhấp nháy khi nhấn nút (debounce)

#### Vòng Lặp Chính (while(TRUE))

##### Kiểm Tra Nút Nhấn
- `if(!input(SW))`: Nếu nút nhấn được nhấn (PIN_B0 kéo xuống thấp)
- `delay_ms(debounce_time);`: Chờ debounce để tránh đọc sai tín hiệu
- `if(!input(SW))`: Kiểm tra lại nút nhấn

##### Chế Độ Bật LED (state == 0)
- Nếu `current_led < 8` (chưa bật hết LED):
  - `output_high(PIN_C0 + current_led);`: Bật LED hiện tại
  - `current_led++;`: Tăng chỉ số LED hiện tại
- Nếu đã bật hết 8 LED:
  - `state = 1;`: Chuyển sang chế độ tắt LED
  - `current_led = 7;`: Đặt lại chỉ số LED hiện tại

##### Chế Độ Tắt LED (state == 1)
- Nếu `current_led < 8` (đang tắt LED):
  - `output_low(PIN_C0 + current_led);`: Tắt LED hiện tại
  - `current_led--;`: Giảm chỉ số LED hiện tại
- Nếu đã tắt hết LED:
  - `state = 0;`: Chuyển sang chế độ bật LED
  - `current_led = 0;`: Đặt lại chỉ số LED hiện tại

##### Chờ Nút Nhả
- `while(!input(SW));`: Chờ cho đến khi nút nhả
- `delay_ms(debounce_time);`: Thêm thời gian debounce sau khi nút nhả

### 7. Thang Điểm
- Hiểu và cấu hình các chân I/O: 2 điểm
- Viết hàm kiểm tra nút nhấn và debounce: 3 điểm
- Điều khiển 8 LED theo chế độ bật/tắt: 4 điểm
- Tối ưu và xử lý lỗi: 1 điểm

[Tiếp tục với các bài tập còn lại...]