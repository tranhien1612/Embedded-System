# Embedded-System

```
// Typical embedded system memory layout
High Address
    ┌─────────────────┐
    │     Stack       │ ← Grows downward
    ├─────────────────┤
    │     Heap        │ ← Grows upward
    ├─────────────────┤
    │     .bss        │ ← Uninitialized data
    ├─────────────────┤
    │     .data       │ ← Initialized data
    ├─────────────────┤
    │     .text       │ ← Code
    └─────────────────┘
Low Address

Stack: Automatic allocation/deallocation, LIFO, limited size
Heap: Manual allocation/deallocation, flexible size, potential fragmentation
```

| Section | Lưu gì         | Nằm ở đâu | Khi nào cấp phát |
| ------- | -------------- | --------- | ---------------- |
| `.text` | Code           | Flash     | Compile          |
| `.data` | Global có init | RAM       | Boot             |
| `.bss`  | Global = 0     | RAM       | Boot             |
| Heap    | malloc         | RAM       | Runtime          |
| Stack   | local var      | RAM       | Runtime          |


### .text (Code section)
👉 Đây là nơi chứa:
- Code C/C++ sau khi compile
- Instruction machine code
- Hằng số read-only (có thể nằm ở đây)

📌 Đặc điểm:
- Nằm trong Flash (ROM)
- Chỉ đọc (read-only)
- Không thay đổi khi chạy

### .data (Initialized Data)
👉 Đây là nơi chứa:
- Biến global/static có giá trị khởi tạo khác 0

📌 Cơ chế đặc biệt:
- Giá trị ban đầu nằm trong Flash
- Khi boot → được copy vào RAM

### .bss (Uninitialized Data)
👉 Chứa:
- Biến global/static không khởi tạo
- Hoặc = 0

📌 Đặc điểm:
- Nằm trong RAM
- Khi boot → được set = 0

### Heap (Dynamic Memory)
👉 Chứa:
- Bộ nhớ cấp phát động

📌 Đặc điểm:
- Nằm trong RAM
- Tăng dần lên trên (↑)
- Quản lý bởi runtime (malloc/free)

### Stack (Call Stack)
👉 Chứa:
- Biến local
- Tham số hàm
- Return address

📌 Đặc điểm:
- Nằm trong RAM
- Tăng xuống dưới (↓)
- Quản lý tự động

## Flow khi MCU khởi động
1. CPU start từ .text
2. Copy .data từ Flash → RAM
3. Set .bss = 0
4. Setup stack pointer
5. Gọi main()

<img width="451" height="709" alt="image" src="https://github.com/user-attachments/assets/b2d78e22-335f-48ca-8961-10fd0ad64d97" />

## Linker Script (.ld)
👉 File .ld dùng để:
- Quy định memory layout
- Ánh xạ các section (.text, .data, .bss…) vào Flash / RAM

## SECTIONS – mapping quan trọng nhất
```
SECTIONS
{
  .text :
  {
    *(.text)
    *(.rodata)
  } > FLASH

  .data :
  {
    *(.data)
  } > RAM AT > FLASH

  .bss :
  {
    *(.bss)
    *(COMMON)
  } > RAM
}
```

Phân tích:

```
.text > FLASH
👉 Chứa:
- Code
- Hằng số (const)
👉 CPU:
- Fetch instruction trực tiếp từ Flash

.data > RAM AT > FLASH
👉 Nghĩa là:
- Khi chạy → .data nằm ở RAM
- Khi lưu firmware → nằm trong Flash

.bss > RAM
👉 Không có trong Flash
👉 Chỉ cần:
- cấp phát RAM
- set = 0 khi startup

```
Quá trình boot:
```
Bước 1: Reset xảy ra
- CPU nhảy vào: Reset_Handler (trong .text)
Bước 2: Copy .data từ Flash → RAM
Bước 3: Zero .bss
Bước 4: Setup stack
- Load SP từ vector table
- Stack bắt đầu từ top RAM
Bước 5: Gọi main()


FLASH:
[ .text ]
[ .rodata ]
[ .data (initial values) ]  ← LMA (Flash)

RAM:
[ .data (runtime) ]        ← VMA (RAM)
[ .bss ]
[ heap ↑ ]
[      ↓ stack ]

```
