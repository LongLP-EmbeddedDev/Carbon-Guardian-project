# Carbon Guardian

Hệ thống giám sát chất lượng không khí sử dụng ESP32, cảm biến MQ-7 (CO) và MQ-2 (gas/LPG), hiển thị dữ liệu real-time trên màn OLED SSD1306.

Đồ án phục vụ mục đích học tập và demo. Cảm biến dòng MQ phù hợp cho giám sát tương đối, không thay thế thiết bị đo khí đã hiệu chuẩn công nghiệp.

## Tính năng

- Đọc dữ liệu từ 2 cảm biến khí (MQ-7, MQ-2)
- Hiển thị trên OLED: giá trị ppm và mức cảnh báo (SAFE / WARN / DANGER) cho từng kênh, kèm mức tổng hợp (OVERALL)
- Tự động calibrate khi khởi động
- Lọc nhiễu bằng EMA (Exponential Moving Average)
- Chống dao động trạng thái bằng hysteresis
- Cảnh báo bằng LED và buzzer khi OVERALL = DANGER

## Phần cứng

- ESP32
- Cảm biến MQ-7 và MQ-2
- Màn OLED SSD1306 I2C 128x64 (địa chỉ 0x3C)
- LED và trở hạn dòng
- Buzzer loại active
- Dây cắm, breadboard, nguồn cấp ổn định

Lưu ý: nguồn cấp không ổn định là một trong những nguyên nhân phổ biến gây sai lệch số liệu đọc từ cảm biến.

## Sơ đồ chân

| Linh kiện | Chân ESP32 |
|---|---|
| MQ-7 (AO) | GPIO 35 |
| MQ-2 (AO) | GPIO 34 |
| LED | GPIO 25 |
| Buzzer | GPIO 26 |
| OLED SDA | GPIO 21 |
| OLED SCL | GPIO 22 |

GPIO 34 và 35 là chân chỉ đọc (input-only), phù hợp cho việc đọc ADC.

## Thư viện

Cài đặt qua Library Manager trong Arduino IDE:
- Adafruit GFX Library
- Adafruit SSD1306
- Wire (có sẵn trong core)

## Nguyên lý hoạt động

**1. Calibrate khi khởi động**
Hệ thống lấy 80 mẫu (khoảng 16 giây, chu kỳ 200ms) để tính giá trị điện trở tham chiếu R0 cho từng cảm biến.

**2. Đo và xử lý dữ liệu**
Đọc giá trị ADC từ hai cảm biến, tính điện trở Rs, quy đổi sang nồng độ ppm theo đường cong cảm biến, sau đó làm mượt bằng bộ lọc EMA (alpha = 0.15).

**3. Phân loại mức cảnh báo**
So sánh giá trị đã lọc với ngưỡng WARN/DANGER. Áp dụng hysteresis để tránh dao động trạng thái khi giá trị nằm gần ngưỡng.

**4. Cảnh báo**
Mức tổng hợp OVERALL được tính bằng max(mức CO, mức gas). Hệ thống chỉ kích hoạt cảnh báo khi OVERALL = DANGER.

## Ngưỡng cảnh báo

| | WARN | DANGER |
|---|---:|---:|
| CO | 20 ppm | 80 ppm |
| Gas | 800 ppm | 2500 ppm |

## Cài đặt và nạp chương trình

1. Mở file `.ino` bằng Arduino IDE
2. Chọn đúng board ESP32 và cổng COM tương ứng
3. Cài đặt các thư viện được liệt kê ở trên
4. Nạp chương trình (Upload)
5. Mở Serial Monitor với baudrate 115200 để theo dõi log

Ví dụ log đầu ra:
```
[DATA] CO: 12.3 (SAFE) | GAS: 950.1 (WARN) | OVERALL: WARN | raw7=1234 raw2=1456
```

## Các thông số có thể tùy chỉnh

| Nhóm | Biến |
|---|---|
| Calibrate | `INIT_SAMPLES` |
| Lọc nhiễu | `ALPHA` |
| Ngưỡng cảnh báo | `CO_WARN`, `CO_DANGER`, `GAS_WARN`, `GAS_DANGER` |
| Hysteresis | `HYS_CO`, `HYS_GAS` |
| Đường cong cảm biến | `MQ7_CO_A`, `MQ7_CO_B`, `MQ2_LPG_A`, `MQ2_LPG_B` |

## Giới hạn kỹ thuật

- Cảm biến dòng MQ phù hợp cho giám sát tương đối và cảnh báo xu hướng, không phải thiết bị đo khí đã hiệu chuẩn công nghiệp.
- Độ chính xác chịu ảnh hưởng bởi chất lượng nguồn cấp, chất lượng module, nhiệt độ, độ ẩm, và bố trí mạch.

## Hướng phát triển

- Gửi dữ liệu lên nền tảng IoT (MQTT/Blynk/ThingsBoard)
- Lưu trữ lịch sử đo (SD card hoặc cloud)
- Cảnh báo nhiều cấp độ (WARN: beep chậm, DANGER: beep nhanh)
- Bổ sung cảm biến nhiệt độ/độ ẩm để bù trừ sai số
- Nút recalibrate trực tiếp trên mạch

