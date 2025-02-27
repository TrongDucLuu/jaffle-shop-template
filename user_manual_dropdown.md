User Manual: Hướng dẫn sử dụng và duplicate Widget BloX để lọc Dashboard
1. Tổng quan
Widget này dùng trong Sisense BloX để lọc dữ liệu dashboard dựa trên một field (ví dụ: Project_Customer_Type hoặc Region). Nó có hai nút:
Filter!: Áp dụng bộ lọc từ giá trị chọn trong dropdown.

Clear: Reset bộ lọc về tất cả dữ liệu và hiển thị "Select All" trong dropdown.

2. Cách hoạt động
Dropdown: Hiển thị danh sách giá trị từ field được chọn (như Project_Customer_Type).

Filter!: Lọc dashboard theo giá trị chọn (ví dụ: "Type A").

Clear: Xóa bộ lọc, trả dashboard về trạng thái tất cả dữ liệu, và reset dropdown về "Select All".

3. Code cơ bản của Widget
Dưới đây là code mẫu cho widget lọc field Project_Customer_Type, với comment giải thích.
JSON (Editor):
json

{
    "showCarousel": false, // Tắt carousel để hiển thị tĩnh
    "body": [
        {
            "type": "Container", // Container để nhóm các phần tử
            "style": {
                "justify-items": "center", // Căn giữa ngang
                "align-items": "center" // Căn giữa dọc
            },
            "items": [
                {
                    "type": "TextBlock", // Văn bản tiêu đề
                    "text": "Đối tượng", // Tên hiển thị trên widget
                    "style": {
                        "text-align": "center", // Căn giữa chữ
                        "font-weight": "600" // In đậm chữ
                    }
                },
                {
                    "type": "Input.ChoiceSet", // Dropdown để chọn giá trị
                    "id": "customerTypeFilter", // ID duy nhất của dropdown
                    "class": "addPlaceholder1", // Class để tìm dropdown, phải khác nhau nếu duplicate
                    "displayType": "compact", // Kiểu hiển thị gọn
                    "style": {
                        "align-self": "center" // Căn giữa dropdown
                    },
                    "choices": "{choices:Project_Customer_Type}" // Nguồn dữ liệu cho dropdown
                }
            ]
        }
    ],
    "actions": [
        {
            "type": "Filters", // Action để lọc dashboard
            "title": "Filter!", // Tên nút Filter!
            "data": {
                "filters": [
                    {
                        "filterJaql": {
                            "explicit": true, // Lọc chính xác giá trị chọn
                            "members": "{panel:Project_Customer_Type}" // Giá trị từ dropdown
                        },
                        "dim": {
                            "title": "Project_Customer_Type" // Tên field trong dashboard
                        }
                    }
                ]
            }
        },
        {
            "type": "clear-choice", // Action tự định nghĩa để reset
            "title": "Clear", // Tên nút Clear
            "data": {
                "FilterFields": ["[Project_Customer_Type]"], // Field cần reset (dimension thật)
                "dropdownClass": "addPlaceholder1" // Class của dropdown để reset giao diện
            }
        }
    ]
}

Script (Scripts):
javascript

widget.on('ready', (w, args) => {
    var tempFilter = 'Select All'; // Giá trị mặc định của dropdown
    // Kiểm tra xem dashboard có bộ lọc nào không
    if (prism.activeDashboard.filters.$$items.length > 0 && 
        prism.activeDashboard.filters.$$items[0].jaql.filter.members) {
        tempFilter = prism.activeDashboard.filters.$$items[0].jaql.filter.members.toString(); // Lấy giá trị bộ lọc hiện tại
    }
    // Thêm "Select All" hoặc giá trị hiện tại vào dropdown khi widget load
    $('select.addPlaceholder1').prepend('<option value="" disabled selected>' + tempFilter + '</option>');
});

Custom Action "clear-choice":
Vào BloX Editor → "Create Action" → đặt tên clear-choice.

Code:
javascript

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

4. Hướng dẫn Duplicate để lọc field mới
Nếu muốn tạo widget mới để lọc field khác (ví dụ: Region), làm theo các bước sau:
Bước 1: Duplicate Widget
Sao chép toàn bộ JSON của widget cũ.

Bước 2: Thay đổi thông tin field mới
Trong JSON mới, thay các giá trị liên quan đến field cũ bằng field mới:
Text hiển thị:
"text": "Đối tượng" → "text": "Khu vực".

Field trong dropdown:
"choices": "{choices:Project_Customer_Type}" → "choices": "{choices:Region}".

Filter! action:
"members": "{panel:Project_Customer_Type}" → "members": "{panel:Region}".

"title": "Project_Customer_Type" → "title": "Region".

Clear action:
"FilterFields": ["[Project_Customer_Type]"] → "FilterFields": ["[Region]"].

Bước 3: Đổi Class để độc lập
Đổi class của dropdown:
"class": "addPlaceholder1" → "class": "addPlaceholder2".

Đổi dropdownClass trong "Clear":
"dropdownClass": "addPlaceholder1" → "dropdownClass": "addPlaceholder2".

Bước 4: Cập nhật Script
Tạo script riêng cho widget mới:
javascript

widget.on('ready', (w, args) => {
    var tempFilter = 'Select All'; // Giá trị mặc định
    // Kiểm tra bộ lọc hiện tại
    if (prism.activeDashboard.filters.$$items.length > 0 && 
        prism.activeDashboard.filters.$$items[0].jaql.filter.members) {
        tempFilter = prism.activeDashboard.filters.$$items[0].jaql.filter.members.toString();
    }
    // Khởi tạo dropdown với class mới
    $('select.addPlaceholder2').prepend('<option value="" disabled selected>' + tempFilter + '</option>');
});

Bước 5: Kiểm tra Dimension thật
Mở F12 → Network → bấm "Filter!" → tìm JAQL request.

Xác nhận [Region] trong "FilterFields" khớp với dim thật (ví dụ: [Sales.Region]).

5. Lưu ý khi thêm Widget mới
Class duy nhất: Mỗi widget cần class riêng (như addPlaceholder1, addPlaceholder2, ...).

Custom Action chung: clear-choice dùng được cho tất cả widget, chỉ cần dropdownClass đúng.

Script riêng: Mỗi widget cần script riêng với class tương ứng.

Dimension chính xác: Kiểm tra dim trong JAQL để FilterFields đúng.

6. Cách kiểm tra
Thêm widget vào dashboard.

Chọn giá trị → "Filter!" → xem dashboard lọc đúng.

Bấm "Clear" → dashboard về "all", dropdown về "Select All".

Nếu lỗi, mở F12 → Console, gửi log cho kỹ thuật viên.

