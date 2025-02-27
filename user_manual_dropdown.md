# User Manual: Hướng dẫn sử dụng và duplicate Widget BloX để lọc Dashboard

## 1. Tổng quan
Widget này dùng trong Sisense BloX để lọc dữ liệu dashboard dựa trên một field (ví dụ: `Project_Type` hoặc `Region`). Nó có hai nút:
- **Filter!**: Áp dụng bộ lọc từ giá trị chọn trong dropdown.
- **Clear**: Reset bộ lọc về tất cả dữ liệu và hiển thị "Select All" trong dropdown.

## 2. Cách hoạt động
- **Dropdown**: Hiển thị danh sách giá trị từ field được chọn (như `Project_Type`).
- **Filter!**: Lọc dashboard theo giá trị chọn (ví dụ: "Type A").
- **Clear**: Xóa bộ lọc, trả dashboard về trạng thái tất cả dữ liệu, và reset dropdown về "Select All".

## 3. Code cơ bản của Widget
Dưới đây là code mẫu cho widget lọc field `Project_Type`, với chú thích giải thích.

### JSON (Editor):
```json
{
    "style": "",
    "script": "",
    "title": "",
    "showCarousel": true,
    "carouselAnimation": {
        "showButtons": false
    },
    "body": [
        {
            "type": "Container",
            "style": {
                "justify-items": "center",
                "align-items": "center"
            },
            "items": [
                {
                    "type": "TextBlock",
                    "spacing": "default",
                    "size": "small",
                    "text": "Loại công trình",
                    "style": {
                        "text-align": "center",
                        "font-weight": "600"
                    }
                },
                {
                    "type": "Container",
                    "spacing": "small",
                    "items": [
                        {
                            "type": "Input.ChoiceSet",
                            "id": "data.filters[0].filterJaql.members[0]",
                            "class": "Project_Type", // Class để tìm dropdown, phải khác nhau nếu duplicate
                            "displayType": "compact",
                            "style": {
                                "align-self": "center"
                            },
                            "choices": "{choices:Project_Type}" // Nguồn dữ liệu cho dropdown, phải khác nhau nếu duplicate
                        }
                    ]
                }
            ]
        }
    ],
    "actions": [
        {
            "type": "Filters",
            "title": "Filter!",
            "data": {
                "filters": [
                    {
                        "panelName": "Project_Type",
                        "filterJaql": {
                            "explicit": true,
                            "members": [
                                ""
                            ]
                        }
                    }
                ]
            }
        },
        {
            "type": "clear-choice",
            "title": "Clear",
            "data": {
                "FilterFields": [
                    "[Project_SaleIn.Project_Type]" // Field cần reset (dimension thật), phải khác nhau nếu duplicate 
                ],
                "dropdownClass": "Project_Type" // Class của dropdown để reset giao diện, phải khác nhau nếu duplicate
            }
        }
    ]
}
```
### Edit Script (Scripts):
// Vào BloX → bấm icon ⋮ (Option) → Chọn Edit Script
Copy đoạn code sau 
```javascript
widget.on('ready', (w, args) => {
    var tempFilter = 'Select All'; // Giá trị mặc định của dropdown
    // Kiểm tra xem dashboard có bộ lọc nào không
    if (prism.activeDashboard.filters.$$items.length > 0 && 
        prism.activeDashboard.filters.$$items[0].jaql.filter.members) {
        tempFilter = prism.activeDashboard.filters.$$items[0].jaql.filter.members.toString();
    }
    // Thêm "Select All" hoặc giá trị hiện tại vào dropdown khi widget load
    $('select.Project_Type').prepend('<option value="" disabled selected>' + tempFilter + '</option>'); // phải đúng class của dropdown định nghĩa trong Json Editor
});
```
### Custom Action "clear-choice":
// Vào BloX Editor → "Create Action" → đặt tên `clear-choice`
```javascript
const filterDims = payload.data.FilterFields; // Lấy danh sách field cần reset từ JSON
const dropdownClass = payload.data.dropdownClass; // Lấy class của dropdown từ JSON
const dash = payload.widget.dashboard; // Dashboard object để cập nhật bộ lọc

let newFilter = {
    jaql: {
        dim: "", // Dimension sẽ được cập nhật trong vòng lặp
        filter: {
            all: true // Reset field về tất cả giá trị
        }
    }
};

// Lặp qua từng field trong FilterFields để reset
filterDims.forEach(function(dim) {
    newFilter.jaql.dim = dim; // Gán dimension cần reset
    dash.filters.update(newFilter, { refresh: true, save: true }); // Cập nhật bộ lọc dashboard
});

// Reset giao diện dropdown
const dropdown = $(`select.${dropdownClass}`); // Tìm dropdown bằng class
if (dropdown.length > 0) {
    dropdown.find('option').removeAttr('selected'); // Xóa lựa chọn cũ
    dropdown.find('option[value=""]').remove(); // Xóa placeholder cũ nếu có
    dropdown.prepend('<option value="" disabled selected>Select All</option>'); // Thêm "Select All"
    dropdown.val(''); // Reset giá trị dropdown về rỗng
    console.log('Dropdown reset thành công cho class:', dropdownClass); // Log để kiểm tra
} else {
    console.log('Không tìm thấy dropdown với class:', dropdownClass); // Log nếu lỗi
}
```
## 4. Hướng dẫn thêm Widget mới
Nếu muốn tạo mới widget Blox Dropdown Filter, làm theo các bước sau:
### Bước 1: Tạo file Template
Mở 1 trình biên dịch text, Copy toàn bộ nội dung trong JSON (Editor) phía trên, lưu file với định dạng json. Ví dụ Dropdown.json
### Bước 2: Import Template
Mở Blox → Design → bấm icon ⋮  → chọn Import Template
Kéo file json vừa tạo vào vùng tải lên hoặc bấm browse điều hướng đến tệp và đặt tên cho template. Sau khi tải lên, bạn có thể tìm thấy danh sách mẫu của mình.
### Bước 3: Chọn Template
Blox → Design → Templates → All Template hoặc Recent Template, chọn Template mà bạn vừa đặt tên

Sửa các giá trị như hướng dẫn, tạo **Custom Action** và **edit script** như trên 

## 5. Hướng dẫn Duplicate widget để tạo thêm Filter từ field mới
Nếu muốn tạo widget mới để lọc field khác (ví dụ: `Region`), làm theo các bước sau:

### Bước 1: Duplicate Widget
- Sao chép toàn bộ JSON của widget cũ.

### Bước 2: Thay đổi thông tin field mới
Trong JSON mới, thay các giá trị liên quan đến field cũ bằng field mới:
- **Text hiển thị**: `"text": "Đối tượng"` → `"text": "Khu vực"`.
- **Field trong dropdown**: `"choices": "{choices:Project_Type}"` → `"choices": "{choices:Region}"`.
- **Filter! action**:
  - `"members": "{panel:Project_Type}"` → `"members": "{panel:Region}"`.
  - `"title": "Project_Type"` → `"title": "Region"`.
- **Clear action**: `"FilterFields": ["[Project_SaleIn.Project_Type]"]` → `"FilterFields": ["[Project_SaleIn.Region]"]`.

### Bước 3: Đổi Class để độc lập
- Đổi `class` của dropdown: `"class": "Project_Type"` → `"class": "Region"`.
- Đổi `dropdownClass` trong "Clear": `"dropdownClass": "Project_Type"` → `"dropdownClass": "Region"`.

### Bước 4: Cập nhật Script
- Tạo script riêng cho widget mới:
```javascript
widget.on('ready', (w, args) => {
    var tempFilter = 'Select All'; // Giá trị mặc định
    // Kiểm tra bộ lọc hiện tại
    if (prism.activeDashboard.filters.$$items.length > 0 && 
        prism.activeDashboard.filters.$$items[0].jaql.filter.members) {
        tempFilter = prism.activeDashboard.filters.$$items[0].jaql.filter.members.toString();
    }
    // Khởi tạo dropdown với class mới
    $('select.Region').prepend('<option value="" disabled selected>' + tempFilter + '</option>');
});
```
### Bước 5: Kiểm tra Dimension thật
- Mở F12 → Network → bấm "Filter!" → tìm JAQL request.
- Xác nhận `[Region]` trong `"FilterFields"` khớp với `dim` thật (ví dụ: `[Sales.Region]`).

## 5. Lưu ý khi thêm Widget mới
- **Class duy nhất**: Mỗi widget cần class riêng (như `Project_Type`, `Region`, ...).
- **Custom Action chung**: `clear-choice` dùng được cho tất cả widget, chỉ cần `dropdownClass` đúng.
- **Script riêng**: Mỗi widget cần script riêng với class tương ứng.
- **Dimension chính xác**: Kiểm tra `dim` trong JAQL để `FilterFields` đúng.

## 6. Cách kiểm tra
1. Thêm widget vào dashboard.
2. Chọn giá trị → "Filter!" → xem dashboard lọc đúng.
3. Bấm "Clear" → dashboard về "all", dropdown về "Select All".
4. Nếu lỗi, mở F12 → Console, gửi log cho kỹ thuật viên.
