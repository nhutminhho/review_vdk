# Bài Tập 3: Hiển Thị Thông Báo Trên LCD Khi Nhấn Nút

## 1. Mô Tả Bài Tập
Sử dụng vi điều khiển PIC16F877A để hiển thị thông báo trên màn hình LCD khi nhấn nút. Mỗi lần nhấn nút, LCD sẽ hiển thị một thông điệp khác nhau theo thứ tự.

## 2. Yêu Cầu
- Một nút nhấn (SW) kết nối với PIN_B0
- Một màn hình LCD 16x2 kết nối với vi điều khiển
- Mỗi lần nhấn nút, hiển thị một thông điệp khác nhau trên LCD:
  - Lần nhấn 1: "Chao Mung!"
  - Lần nhấn 2: "Hoc Vi Du!"
  - Lần nhấn 3: "Ve Nha!"
  - Lần nhấn 4: Quay lại thông điệp đầu tiên
- Khi không có nút nhấn nào được nhấn, giữ thông điệp hiện tại trên LCD

## 3. Phần Cứng Cần Chuẩn Bị
- Vi điều khiển PIC16F877A
- Một nút nhấn (SW) và điện trở kéo lên (10kΩ)
- Màn hình LCD 16x2 (Common Cathode)
- Điện trở giới hạn cho các chân I/O (220Ω)
- Potentiometer 10kΩ cho độ tương phản của LCD
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

      PIC16F877A
      +--------+
      |        |
  PORTC0-PORTC2 --- RS, RW, E của LCD
  PORTD4-PORTD7 --- D4-D7 của LCD
      |
    Potentiometer 10kΩ --- VO của LCD
      |
      Vcc và GND của LCD
```

### Giải Thích Sơ Đồ Mạch

#### Nút Nhấn (SW)
- Một chân của nút nhấn được nối với nguồn 5V thông qua điện trở kéo lên 10kΩ
- Chân còn lại nối với GND và PIN_B0 của PORTB
- Khi nút được nhấn, PIN_B0 sẽ được kéo xuống mức thấp (GND)

#### Màn Hình LCD 16x2
- Chân RS, RW, E:
  - RS (Register Select) nối với chân C0 của PORTC
  - RW (Read/Write) nối với chân C1 của PORTC
  - E (Enable) nối với chân C2 của PORTC
- Chân D4-D7: Nối với PORTD4-PORTD7 của PIC16F877A
- Chân VO: Nối với trung tâm của Potentiometer 10kΩ để điều chỉnh độ tương phản
- Chân Vcc và GND: Nối với nguồn 5V và GND

## 5. Chương Trình Mẫu

```c
#include <16F877A.h>
#fuses NOWDT, HS, NOPUT, NOPROTECT, NODEBUG, NOLVP
#use delay(clock=8M)
#include <lcd.c>

    // Các thông điệp
    const char *messages[] = {"Chao Mung!", "Hoc Vi Du!", "Ve Nha!"};
    int8 msg_index = 0;
    int8 total_msgs = 3;
    unsigned int16 debounce_time = 50; // Thời gian debounce
    
    // Hiển thị thông điệp đầu tiên
    lcd_gotoxy(1,1);
    printf(lcd_putc, messages[msg_index]);
    
    while(TRUE) {
        // Kiểm tra nút nhấn
        if(!input(SW)) { // Nếu nút nhấn được nhấn
            delay_ms(debounce_time); // Debounce
            if(!input(SW)) { // Xác nhận nút nhấn vẫn được nhấn
                // Chuyển sang thông điệp tiếp theo
                msg_index++;
                if(msg_index >= total_msgs) {
                    msg_index = 0; // Quay lại thông điệp đầu tiên
                }
                // Hiển thị thông điệp mới trên LCD
                lcd_gotoxy(1,1);
                printf(lcd_putc, "                "); // Xóa dòng hiện tại
                lcd_gotoxy(1,1);
                printf(lcd_putc, messages[msg_index]);
                
                while(!input(SW)); // Chờ nút nhả
                delay_ms(debounce_time);
            }
        }
    }
}
```

## 6. Giải Thích Chi Tiết Chương Trình

### Khởi Tạo
- `set_tris_c(0xF8);`: Thiết lập chân C0-C2 của PORTC là OUTPUT để kết nối với RS, RW, E của LCD
- `set_tris_d(0x00);`: Thiết lập PORTD4-PORTD7 là OUTPUT để kết nối với D4-D7 của LCD
- `set_tris_b(0xFF);`: Thiết lập PORTB là INPUT để đọc trạng thái của nút nhấn SW
- `lcd_init();`: Khởi tạo màn hình LCD

### Các Thông Điệp
- Mảng `messages` chứa các thông điệp sẽ hiển thị trên LCD
- Biến `msg_index` để theo dõi thông điệp hiện tại
- Biến `total_msgs` chứa tổng số thông điệp
- Biến `debounce_time` để xử lý debounce cho nút nhấn

### Vòng Lặp Chính (while(TRUE))

#### Kiểm Tra Nút Nhấn
- `if(!input(SW))`: Kiểm tra nếu nút nhấn được nhấn
- `delay_ms(debounce_time);`: Chờ thời gian debounce
- `if(!input(SW))`: Kiểm tra lại nút nhấn

#### Chuyển Đổi Thông Điệp
- `msg_index++;`: Tăng biến msg_index lên 1
- `if(msg_index >= total_msgs) { msg_index = 0; }`: Reset về 0 nếu đã hiển thị hết thông điệp

#### Hiển Thị Thông Điệp
- `lcd_gotoxy(1,1);`: Đặt con trỏ LCD về vị trí (1,1)
- `printf(lcd_putc, " ");`: Xóa dòng hiện tại
- `printf(lcd_putc, messages[msg_index]);`: Hiển thị thông điệp mới

#### Chờ Nút Nhả
- `while(!input(SW));`: Chờ cho đến khi nút nhả
- `delay_ms(debounce_time);`: Thêm thời gian debounce sau khi nút nhả

## 7. Thang Điểm
- Hiểu và cấu hình các chân I/O: 2 điểm
- Viết hàm kiểm tra nút nhấn và debounce: 3 điểm
- Viết hàm hiển thị thông điệp trên LCD: 4 điểm
- Tối ưu và xử lý lỗi: 1 điểm

## 8. Lưu Ý
- Đảm bảo chọn đúng chế độ giao tiếp với LCD (4-bit hoặc 8-bit)
- Kiểm tra điện áp nguồn cấp cho LCD và độ tương phản
- Xử lý debounce cho nút nhấn để tránh đọc sai tín hiệu
- Xóa màn hình trước khi hiển thị thông điệp mới để tránh hiển thị chồng lấp Định nghĩa chân nút nhấn
#define SW PIN_B0

void main() {
    // Cấu hình PORTC0-PORTC2 là OUTPUT cho RS, RW, E của LCD
    set_tris_c(0xF8); // Chân C0-C2 là OUTPUT, C3-C7 là INPUT
    // Cấu hình PORTD4-PORTD7 là OUTPUT cho D4-D7 của LCD
    set_tris_d(0x00);
    // Cấu hình PORTB là INPUT cho nút nhấn
    set_tris_b(0xFF);
    
    // Khởi tạo LCD
    lcd_init();
    
    //