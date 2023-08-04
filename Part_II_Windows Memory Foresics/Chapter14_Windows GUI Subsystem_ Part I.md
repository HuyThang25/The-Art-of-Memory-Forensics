Windows graphical user interface (GUI) subsystem có trách nhiệm quản lý đầu vào của người dùng, chẳng hạn như chuyển động chuột và phím nhấn. Ngoài ra, nó còn vẽ bề mặt hiển thị; hiển thị cửa sổ, nút và menu; và cung cấp sự cô lập cần thiết để hỗ trợ nhiều người dùng đồng thời đăng nhập qua bảng điều khiển, RDP và FastUser Switching. Hệ thống con GUI đóng một vai trò quan trọng trong việc sử dụng máy tính hàng ngày và không thể tránh khỏi việc mã độc và kẻ tấn công sửa đổi bộ nhớ GUI trong quá trình hoạt động của họ. Thật không may, có rất ít công cụ, đặc biệt là các công cụ pháp y tế, có khả năng phân tích và báo cáo về những hiện vật được tạo ra và duy trì bởi hệ thống con này.

Hai chương tiếp theo giới thiệu một bộ sưu tập các cấu trúc dữ liệu, lớp, thuật toán, API và plugin để trích xuất dữ liệu liên quan đến GUI từ bộ nhớ vật lý (RAM) của Windows 32 và 64 bit như XP, Server 2003, Vista, Server 2008 và Windows 7. Chúng tôi sẽ thảo luận về các ví dụ cụ thể khác nhau về cách phát hiện mã độc trong bộ nhớ và cách bạn có thể áp dụng kiến thức về bên trong GUI vào các cuộc điều tra pháp y.

## The GUI Landscape

Hệ thống con GUI được tạo thành từ các đối tượng khác nhau, tất cả cùng nhau tạo nên một trải nghiệm người dùng cao cấp. Mối quan hệ của các thành phần này được tóm tắt trong Hình 14-1. Biểu đồ không thể hiện tất cả các bên trong GUI - chỉ những thành phần quan trọng nhất cho pháp y tế và điều tra mã độc.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234858.png)

Như thể hiện trong hình vẽ, một phiên là bao bọc bên ngoài - nó đại diện cho môi trường đăng nhập của người dùng. Phiên có một ID phiên duy nhất và chúng được tạo khi người dùng đăng nhập qua Fast-User Switching, RDP hoặc Terminal Services (TS). Do đó, một phiên được liên kết với một người dùng cụ thể và các tài nguyên liên quan đến phiên có thể được ghi nhận lại những hành động mà người dùng đã thực hiện. Những tài nguyên này bao gồm bảng nguyên tử (một nhóm chuỗi được chia sẻ toàn cầu giữa các ứng dụng trong phiên), một hoặc nhiều trạm cửa sổ (định danh bảo mật được đặt tên), và một bảng xử lý cho các đối tượng USER (tương tự như các đối tượng hạt nhân như luồng và tiến trình, nhưng được quản lý bởi hệ thống con GUI).

Các ứng dụng cần yêu cầu đầu vào từ người dùng chạy trong một trạm cửa sổ tương tác (ví dụ: WinSta0). Các dịch vụ chạy nền sử dụng các trạm cửa sổ không tương tác. Mỗi trạm cửa sổ có bảng nguyên tử riêng của nó, clipboard và một hoặc nhiều desktop. Một desktop chứa các đối tượng giao diện người dùng như cửa sổ, menu, hook và một heap để cấp phát và lưu trữ các đối tượng như vậy. Các cửa sổ có thể hiển thị hoặc ẩn; chúng có một tập hợp tọa độ màn hình, một thủ tục cửa sổ (hàm thực thi khi nhận các tin nhắn cửa sổ), một tiêu đề tùy chọn hoặc tiêu đề, và một lớp kèm theo. Bằng cách phân tích cửa sổ, bạn có thể xác định người tấn công hoặc người dùng nạn nhân đang xem gì tại thời điểm bộ nhớ bị trôi qua, hoặc các ứng dụng GUI nào đã chạy trong quá khứ.

Hầu hết các API liên quan đến GUI được tiết lộ cho chế độ người dùng được thực hiện trong user32.dll và gdi32.dll. Chúng tương đương với ntdll.dll cho các API native - chúng chứa các đoạn mã đệm mà định tuyến người gọi thông qua Bảng giải pháp Dịch vụ Hệ thống (SSDT) và vào hạt nhân. Tuy nhiên, trong trường hợp này, SSDT định tuyến các cuộc gọi GUI vào mô-đun nhân win32k.sys thay vì mô-đun điều hành NT. Mối quan hệ này được hiển thị trong Hình 14-2.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234909.png)

>**Ghi chú:**<br> Hệ thống con GUI có kích thước lớn như thế nào? Có khoảng ba lần số lượng API (667 so với 284 trên máy 32 bit XP SP3) được tiết lộ thông qua các bảng gọi hệ thống để thực hiện tất cả các nhiệm vụ GUI, so với những API cần thiết để thực hiện các hành động native (tạo tiến trình, xử lý tệp, xác thực, registry, vv.).
> ![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234923.png)

## GUI Memory Forensics

Khung làm việc Volatility cung cấp một số plugin được thiết kế để khám phá và trích xuất dữ liệu từ hệ thống con GUI của Windows. Việc phát triển khả năng này không hề dễ dàng. Trước tiên và quan trọng nhất, lượng tài liệu công khai có sẵn rất ít. Ngoài ra, Microsoft không tiết lộ các tệp ký hiệu gỡ lỗi (PDB) cho thành phần chế độ kernel chính (win32k.sys) cho đến khi Windows 7 xuất hiện, và Microsoft lại loại bỏ chúng bắt đầu từ Windows 8. Do đó, để hỗ trợ tất cả các hệ điều hành khác ngoài Windows 7 yêu cầu công việc đáng kể để xác định các thay đổi về tên cấu trúc, kích thước và vị trí thành viên giữa các phiên bản hệ điều hành khác nhau.

Kết quả cuối cùng là một bộ plugin rất mạnh mẽ cho phân tích bộ nhớ, được tóm tắt trong Bảng 14-2.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234935.png)

## The Session Space

Phiên là bao bì bên ngoài cho cảnh quan GUI, như được thể hiện trong Hình 14-1. Khi người dùng đăng nhập vào hệ thống qua console, RDP hoặc Fast-User Switching, hạt nhân tạo ra một phiên mới, đó là một bao gồm tiến trình và các đối tượng (chẳng hạn như trạm cửa sổ và bàn làm việc, sẽ được thảo luận tiếp theo) thuộc về phiên đó. Một bản sao RAM sẽ chứa thông tin về các phiên đăng nhập đang hoạt động và, trong một số trường hợp, phiên đã kết thúc, bao gồm các tiến trình liên quan, các mô-đun kernel, dải địa chỉ cấp phát trong bộ nhớ pool và các bảng trang.

**Mục tiêu**

- Liên kết các tiến trình với người dùng RDP: Nếu bạn thấy một tiến trình đang chạy và muốn biết liệu một người dùng đã khởi chạy nó qua RDP hay không, bạn có thể sử dụng thông tin trong phần này để tìm hiểu.
- Phát hiện các tiến trình ẩn: Mỗi cấu trúc phiên chứa một danh sách liên kết các tiến trình cho phiên đó. Nếu malware tách tiến trình ra khỏi PsActiveProcessHead (xem Chương 6), bạn có thể sử dụng danh sách tiến trình thay thế này như một cách để xác định tiến trình ẩn.
- Xác định các trình điều khiển kernel: Mỗi cấu trúc phiên chứa một danh sách các trình điều khiển được ánh xạ vào phiên. Bạn có thể sử dụng điều này để phân biệt phiên RDP và phiên console.

**Cấu trúc dữ liệu**

Cấu trúc chính cho một phiên là _MM_SESSION_SPACE. Đây là một cấu trúc lớn, vì vậy chỉ một phần nhỏ của nó được hiển thị trong đoạn mã sau từ Windows 7 x64. Bằng cách xác định tất cả các cấu trúc _EPROCESS (thông qua việc duyệt danh sách hoặc quét pool) và xem các con trỏ _EPROCESS.Session duy nhất, bạn có thể thu thập một danh sách đầy đủ các cấu trúc _MM_SESSION_SPACE. Mỗi phiên có riêng danh sách bộ nhớ làm việc, danh sách look-aside, paged pool, bảng chỉ mục trang và bảng theo dõi big page pool.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20235003.png)

**Các điểm chính**

- SessionId: Định danh duy nhất cho phiên. Trên Windows XP và Server 2003, một phiên đơn (phiên 0) được chia sẻ bởi các dịch vụ hệ thống và các ứng dụng người dùng. Bắt đầu từ Vista, Microsoft đã giới thiệu "cô lập phiên 0" để ngăn chặn các cuộc tấn công shatter (xem thông tin tham khảo trong phần "Lạm dụng Cửa sổ độc hại"), kể từ đó phiên 0 chỉ dành cho các dịch vụ hệ thống.

- ProcessList: Bạn có thể sử dụng thành viên này làm danh sách tiến trình thay thế cho các tiến trình ẩn đi thông qua DKOM thông thường (gỡ bỏ khỏi PsActiveProcessHead). Mỗi tiến trình thuộc về đúng một phiên, ngoại trừ tiến trình Hệ thống và smss.exe. Trong quá trình khởi tạo tiến trình, thành viên _EPROCESS.Session được cập nhật để trỏ đến _MM_SESSION_SPACE. Tương tự, ProcessList được cập nhật với liên kết tới _EPROCESS mới. Một tiến trình ở trong danh sách này cho đến khi nó kết thúc.

- ImageList: Một danh sách các cấu trúc _IMAGE_ENTRY_IN_SESSION - một cho mỗi trình điều khiển thiết bị được ánh xạ vào không gian phiên. Nếu bạn có hai phiên, bạn sẽ có hai bản sao của win32k.sys. Do đó, các công cụ pháp y tế số muốn phân tích mã hoặc biến trong trình điều khiển win32k.sys phải trích xuất nó một lần cho mỗi phiên.

### Detecting Remote Logged-in Users over RDP

Như được hiển thị trong mã sau cho hệ thống Windows 2003 x86, bạn có thể xác định người dùng đăng nhập qua RDP bởi vì trình điều khiển RDPDD.dll và tiến trình rdpclip.exe đang chạy trong phiên. RDPDD.dll là trình điều khiển hiển thị RDP, và rdpclip.exe là tiến trình xử lý các hoạt động cắt và dán từ xa. Bây giờ, thông qua việc liên kết, bạn biết rằng mbamgui.exe, cmd.exe và notepad.exe (trong số các tiến trình khác) được mở/hiển thị qua RDP thay vì trên console máy tính. Điều này rất hữu ích khi tái tạo hành động của kẻ tấn công từ xa.

>**LƯU Ý**<br>
Tiện ích này tách các tiến trình vào phiên của họ, nhưng nó không tự động liên kết phiên với người dùng. Nó không nói rằng phiên 1 thuộc về Rob và phiên 2 thuộc về Jack chẳng hạn. Để thực hiện liên kết đó, hãy chọn một hoặc nhiều tiến trình từ phiên và sử dụng kỹ thuật liên quan đến getsids, như đã mô tả trong Chương 6.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20235019.png)

Một mẹo bạn nên nhớ là Volatility cũng có thể trích xuất lịch sử lệnh và toàn bộ bộ đệm màn hình từ RAM (để biết thêm thông tin, xem Chương 17). Từ đầu ra trước đó, bạn đã biết rằng cmd.exe đã được gọi qua RDP, nhưng để làm gì? Mặc dù giao thức RDP cho phép chuyển tệp, nhưng không phải lúc nào cũng được kích hoạt, vì vậy thông thường bạn sẽ thấy kẻ tấn công đang gửi/nhận các tệp qua FTP từ/đến trang web của họ. Ví dụ được hiển thị trong mã sau (để bảo vệ danh tính của nạn nhân, nhiều trường đã được che dấu):

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20235040.png)
![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20235105.png)
![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20235119.png)

Như bạn có thể thấy, chỉ với một vài lệnh, bạn có thể nhận ra rằng một người dùng đã đăng nhập vào hệ thống nạn nhân qua RDP. Sau đó, anh ta đã sử dụng cmd.exe với ID quy trình 5544 để tìm kiếm các mục nhập nhật ký IIS cụ thể và sao chép nhật ký vào trang web FTP của mình. Bạn có thể thấy địa chỉ máy chủ FTP, tên người dùng và mật khẩu của kẻ tấn công và các tệp chính xác mà anh ta quan tâm.



