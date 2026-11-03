# Session16 – Ex03: Sửa lỗi chồng lấn giao diện bằng DevTools & CSS Box Model

> **Sinh viên:** [Họ và tên] – **Lớp:** [Tên Lớp] – **Môn:** [Môn Học]

> **Lưu ý về số liệu:** đề bài không cho sẵn mã nguồn nên các thông số trong bài (kích thước, `-30px`, `24px`...) là **số liệu giả định để minh họa**. Khi làm thật, hãy mở F12 trên trang được giao và **thay bằng số bạn đọc được** ở tab Computed.

## 1. Bối cảnh

Giao diện "Theo dõi lộ trình xe" bị vỡ: khung **"Thông tin tài xế"** đè lên khung **"Danh sách trạm"**. Khi soi bằng F12, các thông số kích thước trông lộn xộn.

**Ràng buộc:** tuyệt đối **không sửa file mã nguồn gốc**. Mọi thay đổi chỉ được **thử và đề xuất** qua tab **Styles** trong DevTools (thay đổi chỉ tồn tại tạm thời, tải lại trang là mất).

Cấu trúc HTML giả định của trang:

```html
<div class="tracking-page">
  <div id="driver-info" class="panel">  Thông tin tài xế  </div>
  <div id="stop-list"   class="panel">  Danh sách trạm    </div>
</div>
```

---

## 2. Chức năng 1 – Định vị phần tử lỗi bằng Inspect Element

1. Mở trang "Theo dõi lộ trình xe" trên Chrome/Edge.
2. Nhấn **F12** (hoặc chuột phải vào khung lỗi → **Inspect**).
3. Nhấn biểu tượng **Select an element** (mũi tên trong ô vuông, phím tắt `Ctrl + Shift + C`).
4. Di chuột vào khung **"Thông tin tài xế"** rồi bấm chọn. Tab **Elements** sẽ tự nhảy tới đúng thẻ trong **DOM Tree**.
5. Ghi lại thẻ tìm được: `<div id="driver-info" class="panel">`.
6. Rê chuột lên thẻ trong tab Elements để trình duyệt tô màu từng lớp Box Model trên trang (xanh dương = content, xanh lá = padding, vàng = border, cam = margin). Vùng cam lấn sang khung "Danh sách trạm" chính là nơi gây lỗi.

| Nội dung | Ảnh |
|---|---|
| Inspect: thẻ `#driver-info` được chọn trong DOM Tree | ![Inspect](images/01-inspect-element.png) |

---

## 3. Phần 1 – Phân tích: Box Model hiện tại (tab Computed)

### 3.1. Cách đọc

F12 → chọn thẻ `#driver-info` → tab **Computed**: ở đầu tab có sơ đồ Box Model gồm 4 lớp lồng nhau.

| Lớp | Vai trò | Giá trị giả định |
|---|---|---|
| **Content** | Vùng chứa nội dung thật (chữ, ảnh) | 320 × 110 |
| **Padding** | Đệm bên trong, giữa content và border | 20px (4 phía) |
| **Border** | Đường viền quanh padding | 2px (4 phía) |
| **Margin** | Khoảng cách **bên ngoài**, đẩy các phần tử khác ra xa | top 0 / right 0 / **bottom -30px** / left 0 |

### 3.2. Sơ đồ Box Model hiện tại

![Box Model hiện tại](images/04-box-model-before.png)

Kích thước hiển thị (không tính margin):

- Chiều rộng = 320 + 20×2 + 2×2 = **364px**
- Chiều cao = 110 + 20×2 + 2×2 = **154px**

### 3.3. Nguyên nhân gây chồng lấn

`margin-bottom` của `#driver-info` là **`-30px` (số âm)**. Margin âm làm phần tử phía dưới bị **kéo ngược lên** thay vì bị đẩy xuống. Khung "Danh sách trạm" bắt đầu **sớm hơn 30px**, nên đè lên 30px cuối của khung "Thông tin tài xế".

![Chồng lấn](images/06-layout-before-after.png)

> **Kết luận:** lỗi nằm ở lớp **Margin** của `#driver-info`, không phải ở `#stop-list`.

---

## 4. Phần 2 – Triển khai: thông số CSS đề xuất

### 4.1. Thao tác trong tab Styles (không sửa file gốc)

1. Chọn thẻ `#driver-info` trong tab Elements.
2. Mở tab **Styles**, tìm quy tắc có `margin-bottom: -30px`.
3. Bấm vào giá trị `-30px`, đổi thành **`24px`**, nhấn Enter.
4. Quan sát trang: hai khung tách ra ngay lập tức.
5. (Tùy chọn) Mở **Changes** (phím `Ctrl + Shift + P` → gõ "Show Changes") để xem và chụp lại phần khác biệt dùng làm bằng chứng. Vì không sửa file gốc nên mọi thay đổi chỉ nằm trong DevTools.

### 4.2. Đề xuất CSS mới (chỉ ghi trong báo cáo, không áp vào file gốc)

```css
/* Trước (đang lỗi) */
#driver-info {
  margin-bottom: -30px;
}

/* Sau (đề xuất) */
#driver-info {
  box-sizing: border-box;   /* width/height đã gồm padding + border, dễ tính toán */
  margin-bottom: 24px;      /* tạo khoảng cách 24px giữa hai khung */
}
```

Nếu khung có chiều cao cố định làm nội dung tràn ra ngoài, đề xuất thêm:

```css
#driver-info {
  height: auto;             /* để khung tự co giãn theo nội dung */
  overflow: visible;
}
```

### 4.3. Box Model sau khi sửa

![Box Model sau khi sửa](images/05-box-model-after.png)

| Thông số | Trước | Sau |
|---|---|---|
| margin-bottom | **-30px** | **24px** |
| Vị trí khung "Danh sách trạm" | Đè 30px lên khung tài xế | Cách 24px bên dưới |
| Content / Padding / Border | 320×110 / 20 / 2 | Không đổi |

Khoảng cách thực tế giữa hai khung thay đổi: từ −30px (chồng lấn) thành +24px (tách rời), tức nhích khung "Danh sách trạm" xuống 54px.

| Nội dung | Ảnh |
|---|---|
| Tab Styles: đổi `margin-bottom` từ -30px thành 24px | ![Styles](images/02-styles-fix.png) |

---

## 5. Phần 3 – Kiểm thử: đổi Margin nhưng phần tử vẫn không di chuyển

Giả sử đã đổi margin nhưng khung vẫn đứng yên, cần kiểm tra xem có thuộc tính nào đang **ghi đè** hoặc **vô hiệu hóa** margin không, kể cả ở tầng **Browser Engine** (CSS mặc định của trình duyệt).

![Bảng kiểm tra](images/07-kiem-tra-margin.png)

### 5.1. Quy trình kiểm tra trong DevTools

**Bước 1. Xem Styles: dòng margin có bị gạch ngang không?**

- Dòng bị **gạch ngang** nghĩa là có quy tắc khác đang thắng.
- Các nguyên nhân thường gặp: quy tắc có **độ ưu tiên (specificity) cao hơn**, khai báo **`!important`**, hoặc thuộc tính **`style="..."` viết trực tiếp (inline)** trên thẻ.

**Bước 2. Xem Computed: giá trị cuối cùng là gì?**

- Tab Computed hiển thị giá trị **thực sự được áp dụng**.
- Bấm mũi tên **▸** cạnh thuộc tính `margin-bottom` để biết **rule nào, ở file nào, dòng nào** đang thắng.

**Bước 3. Kiểm tra tầng Browser Engine (User Agent Stylesheet)**

- Trình duyệt có sẵn một bảng CSS mặc định (hiển thị trong tab Styles với nhãn **`user agent stylesheet`**, chữ màu xám). Ví dụ `body { margin: 8px }`, `h1 { margin-block: 0.67em }`, `p { margin-block: 1em }`.
- CSS của trang (author CSS) thường **thắng** user agent stylesheet. Nhưng nếu trang **không khai báo lại** margin hoặc khai báo ở selector yếu hơn thì giá trị mặc định của trình duyệt vẫn được dùng.
- Cách xử lý: khai báo rõ `margin` cho phần tử trong CSS của trang.

**Bước 4. Các thuộc tính khiến margin "không có tác dụng"**

| Thuộc tính | Hiện tượng | Cách xử lý |
|---|---|---|
| `display: inline` | Margin **trên/dưới** không có tác dụng | Đổi sang `block` hoặc `inline-block` |
| `position: absolute / fixed` | Phần tử rời khỏi luồng, margin không đẩy các phần tử khác | Chỉnh `top / bottom / inset` |
| Cha là `display: flex / grid` | `gap`, `align-*`, `order` quyết định vị trí | Kiểm tra `display` của phần tử cha, dùng `gap` |
| **Margin collapsing** | Hai margin dọc liền kề gộp lại, lấy giá trị lớn nhất | Dùng `padding`, hoặc `display: flow-root` ở phần tử cha |
| `transform: translate()` | Dịch chuyển vị trí hiển thị nhưng không đổi chỗ trong luồng | Xem thuộc tính `transform` |
| Chiều cao cố định + nội dung tràn | Nội dung vượt khung, nhìn như bị đè | Dùng `height: auto` |

### 5.2. Kết luận kiểm thử

Nếu thay đổi margin không có tác dụng, thứ tự kiểm tra là: **(1)** dòng margin có bị gạch ngang không → **(2)** giá trị trong Computed và rule đang thắng → **(3)** nhãn `user agent stylesheet` → **(4)** `display`, `position`, kiểu bố cục của phần tử cha, margin collapsing. Khi tìm ra thuộc tính gây cản trở, sửa lại trong tab Styles và quan sát trang cập nhật ngay.

| Nội dung | Ảnh |
|---|---|
| Computed: bấm ▸ xem rule đang thắng; Styles: dòng bị gạch ngang / user agent stylesheet | ![Kiểm tra ghi đè](images/03-computed-override.png) |

---

## 6. Kết luận

| Nội dung | Kết quả |
|---|---|
| Phần tử lỗi | `<div id="driver-info">` (định vị bằng Inspect Element) |
| Nguyên nhân | `margin-bottom: -30px` kéo khung "Danh sách trạm" lên đè |
| Cách sửa (đề xuất qua tab Styles) | `margin-bottom: 24px` (kèm `box-sizing: border-box`, `height: auto` nếu cần) |
| Margin không có tác dụng | Kiểm tra dòng bị gạch ngang, rule ghi đè, user agent stylesheet, `display`, `position`, margin collapsing |
| Ràng buộc | Không sửa file gốc, mọi thay đổi chỉ qua DevTools |
