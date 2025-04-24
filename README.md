# ESP32-S3 Eye Person Detection (Custom Model)

This project demonstrates how to flash a **custom person detection model** with 3 categories (`ON`, `OFF`, `NEUTRAL`) onto an **ESP32-S3 Eye** using the [esp-tflite-micro](https://github.com/espressif/esp-tflite-micro) repository.

---

## 🚀 Getting Started

### ✅ Prerequisites
- ESP32-S3 Eye dev board
- VSCode with ESP-IDF Extension
- ESP-IDF v5.4.1
- Python 3.11 (installed by ESP-IDF)
- USB-C cable

### 📦 Clone the Repo
```bash
git clone https://github.com/espressif/esp-tflite-micro.git
cd esp-tflite-micro
```

### 📂 Select the Example
```bash
cd person_detection
```

---

## 🔧 Modify for ESP32-S3 Eye and Custom Model

### 📁 Replace Model Files
Replace the default model with your own quantized `.h` and `.cpp` model files in the project directory.

### ✍️ Update `model_settings.cc`
```cpp
constexpr int kCategoryCount = 3;
const char* kCategoryLabels[kCategoryCount] = {"ON", "OFF", "NEUTRAL"};
```

### ✍️ Update Output Parsing in `main_functions.cc`
Print percentages:
```cpp
float score = (output->data.uint8[i] - output->params.zero_point) * output->params.scale;
```
Clamp values with:
```cpp
int to_percent(float score) {
    int pct = static_cast<int>(score * 100.0f + 0.5f);
    if (pct < 0) return 0;
    if (pct > 100) return 100;
    return pct;
}
```

---

## 🛠️ Fixes for Common Issues

### ❌ `#endif without #if`
**Fix:** Remove leftover conditional compilation blocks from original person detection code.

### ❌ `kPersonIndex` not found
**Fix:** Replace with index-based access and custom label logic:
```cpp
output->data.uint8[0] // ON
output->data.uint8[1] // OFF
output->data.uint8[2] // NEUTRAL
```

### ❌ "Didn't find op for builtin opcode 'FULLY_CONNECTED'"
**Fix:** Add the missing op to `MicroMutableOpResolver`:
```cpp
micro_op_resolver.AddFullyConnected();
```
Update count from `<5>` to `<6>` if needed.

### ❌ Output greater than 100%
**Fix:** Use the `to_percent()` function above to clamp values between 0 and 100.

---

## ⚙️ Set Target & Build

```bash
idf.py set-target esp32s3
idf.py menuconfig  # Optional
idf.py fullclean
idf.py build
```

---

## 🔥 Flash to Device

```bash
idf.py -p COM4 flash monitor
```
> Replace `COM4` with the correct port for your ESP32-S3 Eye

---

## ✅ Example Output
```text
ON = 83%, OFF = 5%, NEUTRAL = 12%
Image Captured
```

---

## 🧠 Notes
- You must run `idf.py set-target esp32s3` to match your chip.
- The ESP32-S3 Eye uses USB CDC — ensure drivers are installed.
- If flashing fails, try pressing the reset button while connecting or reducing baud rate.

---

## 🙌 Acknowledgments
Thanks to the [Espressif](https://github.com/espressif) team and TensorFlow Lite Micro for enabling embedded ML!

---

## 🧪 To Do / Next Steps
- Add GPIO output or LED trigger on ON detection
- Display results on an OLED screen
- Capture images to SD card for training feedback

---

