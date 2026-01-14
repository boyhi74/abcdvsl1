
**File:** `final.jpg`  

---

## 📌 Mô tả
Ta được cung cấp một file ảnh JPEG. Ảnh hiển thị bình thường nhưng thực chất chứa:
- dữ liệu bị che bởi height override,
- một payload nhúng trong APP15,
- payload đó được mã hóa AES và chứa ZIP.

Mục tiêu: trích xuất dữ liệu ẩn và tìm flag.

## 🔎 1. Phát hiện height override
Kiểm tra thông tin ảnh:
identify final.jpg
SOF0:
`FF C0 .... 01 2C`
<img width="642" height="449" alt="image" src="https://github.com/user-attachments/assets/9924826f-de7a-424b-b386-087143d7bdd1" />

sử dụng hexedit để chỉnh sửa
hexedit final.jpg
Đổi thành:
`01 A1`
`| Dữ liệu ảnh bên dưới vẫn tồn tại nhưng bị che bởi SOF0 height override`
PASS_B64 lộ ra:
`aDMxbGg3X2wxaTNz`
Decode:
echo aDMxbGg3X2wxaTNz | base64 -d

→ `h31lh7_l1i3s`

## 🔍 2. Tìm APP15
Tìm marker:
grep -aobU $'\xFF\xEF' final.jpg
Xem header:
xxd -l 40 final.jpg
📌 Salted__ là signature của openssl enc
Payload bắt đầu sau 4 byte.
dd if=final.jpg of=app15_payload.bin bs=1 skip=24 count=393 status=none
xxd -l 32 app15_payload.bin

## 🔐 3. AES decrypt
openssl enc -d -aes-256-cbc -pbkdf2 -in blob.bin -out recovered.zip -k h31lh7_l1i3s
## 📦 4. ZIP
unzip recovered.zip
## 🔎 5. Flag

echo "$(cat p1.txt)$(cat p2.txt)" | xxd -r -p

`vsl{sof0_height_override}`
