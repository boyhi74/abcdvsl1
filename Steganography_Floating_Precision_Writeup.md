# 🕵️ Steganography Write-up  
## Challenge: Floating Precision (16-bit PNG)

### 📌 Thông tin chung
- **Thể loại**: Steganography  
- **File**: `challenge.png`  
- **Định dạng flag**: `vsl{...}`  

---

## 🔍 Bước 1: Nhận dạng file và đặc tính ảnh

Kiểm tra loại file và thông tin ảnh:

```bash
file challenge.png
identify -verbose challenge.png | head -n 20
```

**Phân tích**
- Ảnh là **PNG 16-bit grayscale**
- Mỗi pixel có giá trị từ `0` đến `65535`
- PNG là định dạng **lossless**, phù hợp cho việc giấu dữ liệu ở các bit thấp (LSB)

---

## 🔬 Bước 2: Trích xuất dữ liệu pixel thô

Xuất dữ liệu raw của ảnh:

```bash
magick challenge.png -depth 16 gray:- | xxd -p -c2 | head
```

**Giải thích**
- Mỗi dòng hex đại diện cho **1 pixel (16-bit = 2 byte)**
- Ở bước này, **endianness (thứ tự byte)** chưa rõ

---

## 🧠 Bước 3: Nghi ngờ stego theo modulo 4

Trong các bài stego 16-bit, kỹ thuật phổ biến là:
- Tạo nền là **bội số của 4**
- Mã hóa bit bằng cách:
  - `+1` → bit `0`
  - `+3` → bit `1`
- Khi đó, đọc dữ liệu bằng **`pixel % 4`**

---

## ⚠️ Bước 4: Phát hiện bẫy Endianness

Khi tính `mod 4` trực tiếp, phân bố giá trị không hợp lý.  
Nguyên nhân là **PNG 16-bit lưu dữ liệu theo big-endian**, nên khi đọc raw cần **đảo byte**:

- `abcd` → `cdab`

Kiểm tra lại phân bố `mod 4` sau khi swap endian:

```bash
magick challenge.png -depth 16 gray:- | xxd -p -c2 \
| gawk '{v=strtonum("0x" substr($0,3,2) substr($0,1,2)); c[v%4]++}
        END{for(i=0;i<4;i++) print i, c[i]+0}'
```

**Kết quả**
```
0 65296
1 109
2 0
3 131
```

**Phân tích**
- `mod4 == 2` không xuất hiện → đúng kiểu nền bội số của 4
- `109 + 131 = 240` bit = `30 byte` → đúng độ dài flag

---

## 🔓 Bước 5: Giải mã bit → ASCII (không dùng script Python)

### Quy ước giải mã
| pixel % 4 | Bit |
|----------|-----|
| 1 | 0 |
| 3 | 1 |
| 0 | Bỏ qua |

### Lệnh giải hoàn chỉnh

```bash
magick challenge.png -depth 16 gray:- \
| xxd -p -c2 \
| gawk '
function bin2dec(s, i,v){
  v=0; for(i=1;i<=length(s);i++) v=v*2+substr(s,i,1);
  return v
}
BEGIN{ bits="" }
{
  v = strtonum("0x" substr($0,3,2) substr($0,1,2));  # swap endian
  r = v % 4

  if (r==1) b="0"
  else if (r==3) b="1"
  else next

  bits = bits b
  if (length(bits)==8){
    c = sprintf("%c", bin2dec(bits))
    printf "%s", c
    bits=""
    if (c=="}"){ print ""; exit }
  }
}
'
```

---

## ✅ Kết quả

```
vsl{float_precision_is_a_trap}
```

---

## 🧩 Kết luận

- Bài toán sử dụng **fixed-point steganography** trên ảnh PNG 16-bit
- Dữ liệu được giấu trong **2 bit thấp (modulo 4)**
- Có **bẫy endianness**, yêu cầu hiểu rõ cách lưu trữ dữ liệu 16-bit
- Bài giải hoàn toàn bằng **command-line**, không cần script Python

👉 Đây là dạng bài stego yêu cầu **phân tích dữ liệu và tư duy hệ thống**, không phụ thuộc vào tool tự động.
