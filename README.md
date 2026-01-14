
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

<img width="625" height="426" alt="image" src="https://github.com/user-attachments/assets/b6fe6f91-d1a1-40fa-aa4e-d8c8e3071ea8" />

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

<img width="724" height="440" alt="image" src="https://github.com/user-attachments/assets/3fb51a5d-85bf-4d79-aca0-1f04b3ccb376" />

## Bỏ “LV4APP15|” để lấy blob thật
Cắt bỏ 9 byte đầu:
dd if=app15_payload.bin of=blob.bin bs=1 skip=9 status=none

<img width="688" height="347" alt="image" src="https://github.com/user-attachments/assets/3811c64f-3eb5-48cf-be69-cd72e4dcf59b" />

## 🔐 3. AES decrypt
openssl enc -d -aes-256-cbc -pbkdf2 -in blob.bin -out recovered.zip -k h31lh7_l1i3s

## 📦 4. ZIP
unzip recovered.zip

## 🔎 5. Flag
echo "$(cat p1.txt)$(cat p2.txt)" | xxd -r -p

<img width="460" height="306" alt="image" src="https://github.com/user-attachments/assets/b567cb81-e794-4d28-a1a2-f17660475099" />

`vsl{sof0_height_override}`
