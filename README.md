# Sửa Lỗi Nhập Liệu Tiếng Việt cho Claude Code CLI

[![Version](https://img.shields.io/github/v/release/hangocduong/sua-loi-nhap-lieu-tieng-viet-claude-code-cli?label=version)](https://github.com/hangocduong/sua-loi-nhap-lieu-tieng-viet-claude-code-cli/releases)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)

> Sửa lỗi nhập liệu tiếng Việt (OpenKey, EVKey, Unikey, PHTV) cho terminal Claude Code.

---

## Cài Đặt

### macOS / Linux

```bash
curl -fsSL https://raw.githubusercontent.com/hangocduong/sua-loi-nhap-lieu-tieng-viet-claude-code-cli/main/install.sh | bash
```

### Windows (PowerShell)

```powershell
irm https://raw.githubusercontent.com/hangocduong/sua-loi-nhap-lieu-tieng-viet-claude-code-cli/main/install.ps1 | iex
```

### ⚠️ Quan trọng

**Sau khi cài đặt, thoát và khởi động lại Claude Code để bản vá có hiệu lực!**

```bash
# Nhấn Ctrl+C để thoát phiên hiện tại, sau đó:
claude
```

---

## Sử Dụng

| Lệnh | Mô tả |
|------|-------|
| `claude-vn-patch` | Áp dụng bản vá |
| `claude-vn-patch status` | Kiểm tra trạng thái |
| `claude-vn-patch restore` | Khôi phục file gốc |
| `claude-update` | Cập nhật Claude + tự động vá |

**Sau khi Claude cập nhật:** Chạy `claude-vn-patch` hoặc `claude-update`, rồi **restart Claude Code**.

---

## Vấn Đề & Giải Pháp

### Vấn đề

Bộ gõ tiếng Việt sử dụng kỹ thuật "backspace-rồi-thay-thế" để chuyển đổi ký tự (`a` → `á`). Claude Code xử lý phím backspace nhưng không hiển thị các ký tự thay thế, gây mất chữ.

```text
Gõ: "cộng hòa xã hội" → Kết quả: "ộng hòa ã hội" ❌
```

### Giải pháp (v1.4.0+)

Bản vá dùng stack-based algorithm để xử lý đúng thứ tự ký tự, kể cả khi gõ nhanh.

```text
Gõ: "cộng hòa xã hội" → Kết quả: "cộng hòa xã hội" ✓
```

---

## Yêu Cầu

- Python 3.6+
- Claude Code qua npm: `npm install -g @anthropic-ai/claude-code`
- Windows, macOS, hoặc Linux

---

## Phiên Bản Đã Kiểm Tra

- Claude Code v2.1.12 (Tháng 1/2026)
- macOS, Windows (npm)

---

## Xử Lý Sự Cố

| Lỗi | Giải pháp |
|-----|-----------|
| Gõ tiếng Việt vẫn lỗi | Đã restart Claude Code chưa? Nhấn `Ctrl+C`, chạy `claude` |
| "Could not find Claude Code cli.js" | Cài Claude qua npm: `npm install -g @anthropic-ai/claude-code` |
| "Could not extract variables" | Mở issue với `claude --version` |
| "Patch already applied" | Bản vá đã hoạt động, kiểm tra: `claude-vn-patch status` |

---

## Chi Tiết Kỹ Thuật

<details>
<summary>Xem cách hoạt động</summary>

### Vấn đề gốc

Code gốc của Claude làm backspace **TRƯỚC** khi chèn ký tự mới:

```text
State: "c" | Input: "o[DEL]ộ" (gõ "cộ" nhanh)
Code gốc: backspace trên "c" → "" (xóa nhầm "c"!)
Đúng ra: chèn "o"→"co", backspace→"c", chèn "ộ"→"cộ"
```

### Giải pháp: Stack-based Algorithm

Dùng stack để tính số DEL thực sự ảnh hưởng state gốc:

- Mỗi ký tự non-DEL: push stack
- Mỗi DEL: pop stack (hoặc xóa từ original nếu stack rỗng)
- Khôi phục ký tự bị xóa nhầm, rồi chèn ký tự thay thế

### Code được chèn (v1.6.0+)

```javascript
// Stack-based: xử lý từng ký tự, chỉ dùng biến global (S, l, Q, T)
let _ns = S, _sk = [];  // _ns: new state, _sk: stack

for(const c of l) {
  if(c === "\x7f") {  // DEL char
    if(_sk.length > 0) _sk.pop();     // DEL tiêu thụ ký tự pending
    else _ns = _ns.backspace();        // DEL ảnh hưởng state gốc
  } else {
    _sk.push(c);                       // Ký tự thường: push stack
  }
}

// Chèn các ký tự còn lại trong stack
for(const c of _sk) _ns = _ns.insert(c);

// Cập nhật UI nếu có thay đổi
if(!S.equals(_ns)) {
  if(S.text !== _ns.text) Q(_ns.text);
  T(_ns.offset);
}
```

</details>

---

## Cấu Trúc Dự Án

```text
sua-loi-nhap-lieu-tieng-viet-claude-code-cli/
├── install.sh                           # Installer (macOS/Linux)
├── install.ps1                          # Installer (Windows)
└── scripts/
    ├── vietnamese-ime-patch.sh          # Wrapper (Bash)
    ├── vietnamese-ime-patch.ps1         # Wrapper (PowerShell)
    ├── vietnamese-ime-patch-core.py     # Logic chính
    ├── claude-update-wrapper.sh         # Update (Bash)
    └── claude-update-wrapper.ps1        # Update (PowerShell)
```

---

## Changelog

### v1.6.0

- Viết lại hoàn toàn thuật toán xử lý IME với proper JavaScript scoping
- Sửa lỗi mất ký tự khi gõ nhanh lần đầu tiên

### v1.5.0

- Đổi tên dự án thành "Sửa Lỗi Nhập Liệu Tiếng Việt cho Claude Code CLI"
- Cập nhật tất cả URL

### v1.4.1

- Thêm thông báo restart sau khi cài đặt/cập nhật

### v1.4.0

- Sửa lỗi mất chữ đầu từ khi gõ nhanh (stack-based algorithm)

### v1.2.0

- Thêm hỗ trợ Windows (PowerShell)

### v1.1.0

- Cài đặt một dòng lệnh qua curl/irm

### v1.0.0

- Phiên bản đầu tiên

---

## Ghi Công

- Ý tưởng ban đầu: [manhit96/claude-code-vietnamese-fix](https://github.com/manhit96/claude-code-vietnamese-fix)
- Trích xuất động & stack-based algorithm: Dự án này

---

## Giấy Phép

MIT License - Xem [LICENSE](LICENSE) để biết thêm chi tiết.
