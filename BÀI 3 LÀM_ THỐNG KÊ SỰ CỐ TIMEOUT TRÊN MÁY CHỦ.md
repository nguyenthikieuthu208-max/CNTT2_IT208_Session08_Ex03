***BÀI 3 LÀM: THỐNG KÊ SỰ CỐ TIMEOUT TRÊN MÁY CHỦ***

Phần 1: Phân tích và Sơ đồ luồng dữ liệu (Pipeline)

Cơ chế Pipeline (|) của PowerShell hoạt động theo nguyên lý truyền nhận Đối tượng (Object) có cấu trúc. Mỗi câu lệnh phía sau sẽ nhận đầu ra của câu lệnh phía trước làm đầu vào để xử lý tiếp.

Sơ đồ luồng dữ liệu (File Tree dạng luồng Pipeline):

text  
\[Tệp Log gốc\]   
      │  
      ▼ (Bước 1: Đọc tệp \- Trả về mảng các chuỗi dòng log)  
  Get-Content server.log  
      │  
      ▼ \[Mảng chuỗi\] qua Pipeline (|)  
  Select-String \-Pattern "TIMEOUT" (Bước 2A: Lọc lấy các dòng có chữ TIMEOUT)  
      │  
      ▼ \[Mảng đối tượng MatchInfo\] qua Pipeline (|)  
  Where-Object { \$\_ \-notmatch "NOT TIMEOUT" } (Bước 2B: Loại trừ các dòng nhiễu "NOT TIMEOUT")  
      │  
      ▼ \[Mảng đối tượng đã làm sạch\] bọc trong dấu ngoặc (...)  
( ... ).Count (Bước 3: Đếm kết quả thông qua thuộc tính .Count của Object)  
      │  
      ▼  
\[Một con số tổng duy nhất\]

Phần 2: Triển khai dòng lệnh PowerShell hoàn chỉnh

Để giải quyết bài toán và xử lý triệt để bẫy dữ liệu (loại trừ các dòng chữ NOT TIMEOUT), câu lệnh một dòng duy nhất trên PowerShell được viết chuẩn xác như sau:

powershell  
(Get-Content server.log | Select-String \-Pattern "TIMEOUT" | Where-Object { \$\_ \-notmatch "NOT TIMEOUT" }).Count

Giải thích chi tiết cơ chế hoạt động:

Get-Content server.log: Đọc toàn bộ nội dung tệp nhật ký và đẩy dữ liệu sang bước tiếp theo.

Select-String \-Pattern "TIMEOUT": Nhận dữ liệu và chỉ giữ lại những dòng có chứa từ khóa "TIMEOUT". Đầu ra lúc này là các đối tượng dạng MatchInfo.

Where-Object { \$\_ \-notmatch "NOT TIMEOUT" }: Thực hiện bộ lọc lần hai (xử lý bẫy). Lệnh này sẽ kiểm tra từng dòng dữ liệu hiện tại (\$\_) và loại bỏ hoàn toàn những dòng chứa chuỗi "NOT TIMEOUT", bảo vệ tính chính xác của báo cáo.

(...).Count: Bao bọc toàn bộ chuỗi đường ống trong dấu ngoặc đơn để ép hệ thống xử lý xong xuôi dữ liệu, sau đó chấm gọi thuộc tính .Count để in ra một con số tổng kết duy nhất, không hiển thị văn bản thừa.

