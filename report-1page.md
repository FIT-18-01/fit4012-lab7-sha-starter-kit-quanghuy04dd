# FIT4012 Lab 7 - Báo cáo 1 trang: SHA-256

## 1. Mục tiêu / Objective

Bài thực hành nhằm tìm hiểu và triển khai thuật toán SHA-256 trong các ứng dụng thực tiễn:
- Phân tích cấu trúc và quy trình thực hiện SHA-256 (padding, chia block, message schedule, 64 vòng nén)
- Kiểm tra tính toàn vẹn của tệp tin bằng hash SHA-256
- Mô phỏng hệ thống đăng ký/xác thực mật khẩu dùng hash
- Cải tiến bảo mật mật khẩu bằng cơ chế salt

## 2. Cách làm / Approach

**Quy trình SHA-256:**
1. **Padding**: Dữ liệu đầu vào được thêm byte `0x80` (bit '1' theo sau bởi các bit '0'), rồi thêm các byte `0x00` cho đến khi độ dài dữ liệu đạt `56 mod 64` byte, cuối cùng thêm độ dài ban đầu dưới dạng 64-bit Big Endian. Điều này đảm bảo dữ liệu luôn có thể chia thành các khối 512 bit (64 byte).

2. **Chia block**: Dữ liệu sau padding được chia thành các khối 512 bit.

3. **Message Schedule W[0..63]**: Với mỗi block:
   - W[0..15] được tạo từ 16 word đầu tiên của block (32 bit/word)
   - W[16..63] được tính từ công thức: `σ1(W[t-2]) + W[t-7] + σ0(W[t-15]) + W[t-16]`

4. **64 vòng nén**: Khởi tạo a-h từ trạng thái hash H[0..7]. Trong mỗi vòng t (t=0..63):
   - `T1 = h + Σ1(e) + Ch(e,f,g) + K256[t] + W[t]`
   - `T2 = Σ0(a) + Maj(a,b,c)`
   - Cập nhật: `e ← d+T1`, `a ← T1+T2`, và dịch các giá trị khác

5. **Cập nhật state**: H[0..7] được cộng với a-h cuối cùng.

6. **Digest**: Ghép H[0..7] thành 256 bit (64 ký tự hex).

**Ứng dụng:**
- **Q1**: Chạy self-test và băm các chuỗi khác nhau để quan sát tính nhạy cảm
- **Q2**: Tính hash file, sau đó phát hiện file bị sửa bằng cách so sánh hash
- **Q3**: Lưu trữ hash SHA-256 của mật khẩu thay vì plaintext, kiểm tra mật khẩu bằng cách so sánh hash
- **Q4**: Thêm salt (chuỗi ngẫu nhiên 128 bit) vào trước mật khẩu rồi hash, để tránh hai người có cùng mật khẩu tạo ra cùng hash

## 3. Kết quả / Result

**Q1 - Phân tích SHA-256:**
- Self-test: **PASS** (3/3 test vector - chuỗi rỗng, "abc", "hello FIT4012 SHA")
- Hash của `abc`: `ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad`
- Hash của `FIT4012`: `04980625134a14f1db10c9bb1d80fe75d7baf070d41476db9df568012b30a12d`
- Hash của `FIT4012!`: `2707f292d79553c8f1bf07af888c146a42a66850dff66d7cf793ecb577ddaaa8`
- **Quan sát**: Chỉ thay đổi một ký tự ("FIT4012" → "FIT4012!"), hash thay đổi hoàn toàn (không một bit nào trùng), chứng minh tính nhạy cảm của SHA-256.

**Q2 - Kiểm tra toàn vẹn file:**
- Hash file ban đầu: `0de77ca6b0acc905412bfcde0f16f42bffb943e6f94baf0aba2ced1bf82c579a`
- Trước khi sửa file: `[PASS] File integrity OK`
- Sau khi thêm "tampered" vào file:
  - Hash mới: `a58a615368eb8e1dc451193eeef3e2aede66aecb86088c018e1cdb2cfbb0a439`
  - Kết quả: `[FAIL] File was changed or expected hash is incorrect`

**Q3 - Băm và kiểm tra mật khẩu:**
- Đăng ký mật khẩu "SecurePass123": `[PASS] Password hash saved`
- Đăng nhập với đúng mật khẩu: `[PASS] Login success`
- Đăng nhập với mật khẩu sai: `[FAIL] Login failed: wrong password`

**Q4 - Salt trong hash mật khẩu:**
- Đăng ký mật khẩu "MyPassword" hai lần tạo ra hai bản ghi hoàn toàn khác:
  - Record 1: `56efab6c280a27c37d474d07c85391c0:0fa3941e32f6de5ab589ac3a6ba36f9d4fdce15d0630337f12e1a1997290a18e`
  - Record 2: `cf7455b3d039c450db5bafd763ec8f32:2db7633571f6a5415e967d9781a0733e4b612dfedf3fe6779a05e38d72ea1937`
- Đăng nhập với "MyPassword" vào cả hai: **Cả hai [PASS]**
- Đăng nhập với mật khẩu sai: **[FAIL]**

## 4. Kết luận / Conclusion

**SHA-256 và kiểm tra toàn vẹn:**
- SHA-256 là hàm băm một chiều: từ bất kỳ dữ liệu đầu vào nào, nó luôn tạo ra digest 256 bit nhất định
- Dữ liệu giống → hash giống; dữ liệu khác → hash khác (với xác suất va chạm cực kỳ thấp)
- Để kiểm tra toàn vẹn: tính hash file, lưu trữ an toàn, sau đó so sánh với hash hiện tại. Nếu khác → file bị thay đổi

**Salt và bảo mật mật khẩu:**
- Khi lưu mật khẩu, nếu chỉ dùng hash đơn (không salt), hai người có cùng mật khẩu sẽ có cùng hash. Kẻ tấn công dùng "rainbow table" (bảng hash sẵn) có thể nhanh chóng tìm ra mật khẩu
- Salt là số ngẫu nhiên được thêm vào mật khẩu trước khi hash. Cùng mật khẩu nhưng salt khác → hash khác
- Salt không cần giữ bí mật (thường lưu cùng hash), nhưng phải khác nhau giữa các bản ghi
- Khi xác thực: đọc salt từ bản ghi, tính `hash(salt + password)`, so sánh với hash lưu trữ

**Hạn chế của demo này:**
- SHA-256 tính rất nhanh (~billions hash/giây với GPU), nên không thích hợp cho mật khẩu
- Hệ thống thực tế cần thuật toán làm chậm (bcrypt, scrypt, Argon2id) với tham số cost cao
- Lab này chỉ phục vụ mục đích học tập minh họa, **không nên dùng cho hệ thống sản xuất**

**Minh chứng:** Xem file log `logs/lab7-results.log`
