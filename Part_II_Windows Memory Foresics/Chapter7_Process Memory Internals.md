Ngay cả sau nhiều năm kinh nghiệm tìm kiếm thông tin trong RAM, lượng bằng chứng mà bạn có thể tìm thấy trong bộ nhớ tiến trình vẫn khiến chúng tôi ngạc nhiên. Mặc dù các dấu vết trong bộ nhớ kernel, như _EPROCESS, có thể cung cấp thông tin hữu ích, nhưng bộ nhớ thực sự mà tiến trình sử dụng chứa rất nhiều dữ liệu có thể tiết lộ thông tin quý giá về trạng thái hiện tại của tiến trình. Dữ liệu này bao gồm, nhưng không giới hạn, tất cả nội dung được xử lý bởi ứng dụng (nhận qua mạng hoặc nhập từ người dùng tương tác); các tệp đã được ánh xạ; thư viện chia sẻ; mật khẩu; giao dịch thẻ tín dụng (đối với hệ thống điểm bán [POS]); và các cấu trúc riêng tư cho email, tài liệu và nhật ký trò chuyện.

Chương này phân tích các giao diện lập trình ứng dụng (API) được sử dụng để cấp phát các loại dữ liệu khác nhau và xem xét cách liệt kê các vùng bộ nhớ của tiến trình thông qua phân tích kỹ thuật ghi chép bộ nhớ. Trong quá trình làm như vậy, bạn sẽ thấy cách tận dụng các đặc điểm (như quyền truy cập, cờ và kích thước) của các phạm vi bộ nhớ để suy luận về loại dữ liệu chúng chứa. Bạn cũng sẽ học cách sử dụng các công cụ và kỹ thuật để trích xuất toàn bộ bộ nhớ tiến trình (ít nhất là những gì có địa chỉ và nằm trong bộ nhớ) hoặc các phạm vi riêng lẻ vào một tệp, điều này cho phép bạn phân tích nó chi tiết hơn với các công cụ bên ngoài như máy quét virus, trình phân tích mã và vân vân. Cuối cùng, chúng tôi giới thiệu một số cách tìm kiếm các mẫu trong bộ nhớ tiến trình, bao gồm cả các API và chữ ký Yara của Volatility.

## What’s in Process Memory?

Hình 7-1 hiển thị một biểu đồ cấp cao về bộ nhớ tiến trình. Mỗi tiến trình có một khung nhìn riêng của bộ nhớ trong phạm vi này. Bởi vì giới hạn trên của phạm vi có thể thay đổi giữa các hệ điều hành, chúng tôi đơn giản hóa giá trị cao nhất bằng cách sử dụng biểu tượng MmHighestUserAddress. Đây là một biểu tượng trong mô-đun NT mà bạn có thể truy vấn bằng trình gỡ lỗi hoặc trong plugin volshell của Volatility. Nói chung, giá trị này sẽ là 0x7FFEFFFF trên các hệ thống 32-bit không sử dụng chuyển mạch /3GB khi khởi động, 0xBFFEFFFF trên các hệ thống 32-bit sử dụng chuyển mạch /3GB khi khởi động và 0x7FFFFFEFFFF trên các hệ thống 64-bit. Tất nhiên, không phải tiến trình nào (hoặc bất kỳ tiến trình nào cả) điền đầy toàn bộ không gian bộ nhớ này, nhưng ngay cả nếu một phần nhỏ của nó được sử dụng, điều đó vẫn là một lượng dữ liệu đáng kể.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20185810.png)

### Address Space Layout Details


Chúng tôi sẽ tóm tắt các phạm vi được đánh số trên hình ảnh, từ trên xuống dưới. Tuy nhiên, điều quan trọng cần lưu ý là vị trí của các phạm vi không cố định, đặc biệt là trên các hệ thống sử dụng ngẫu nhiên hóa bố trí không gian địa chỉ (ASLR). Nói cách khác, các ngăn xếp luồng có thể tồn tại phía dưới hoặc phía trên chương trình thực thi của tiến trình, hoặc các phạm vi chứa các tệp được ánh xạ có thể được xen kẽ trong toàn bộ không gian tiến trình, không được tập trung liền mạch như hình vẽ hiển thị. Bạn cũng sẽ tìm hiểu về từng thành phần này một cách chi tiết hơn trong suốt chương này và chương tiếp theo:

- Thư viện động (DLL): Khu vực này đại diện cho các thư viện động (DLL) đã được tải vào không gian địa chỉ của tiến trình, có thể thông qua việc tải cùng với tiến trình hoặc thông qua kỹ thuật tiêm thư viện.
- Biến môi trường: Phạm vi bộ nhớ này lưu trữ các biến môi trường của tiến trình, chẳng hạn như các đường dẫn thực thi, thư mục tạm thời, thư mục chính và các thông tin tương tự.
- Process Environment Block (PEB): Một cấu trúc vô cùng hữu ích cung cấp thông tin về các thành phần khác trong danh sách này, bao gồm các DLL, heaps và biến môi trường. Nó cũng chứa các đối số dòng lệnh của tiến trình, thư mục làm việc hiện tại và các xử lý thông thường.
- Heaps của tiến trình: Nơi mà bạn có thể tìm thấy hầu hết các đầu vào động mà tiến trình nhận được. Ví dụ, văn bản có độ dài thay đổi mà bạn nhập vào email hoặc tài liệu thường được đặt trên heap, cũng như dữ liệu được gửi hoặc nhận qua các socket mạng.
- Ngăn xếp luồng: Mỗi luồng có một phạm vi bộ nhớ riêng dành cho ngăn xếp thời gian chạy của nó. Đây là nơi bạn có thể tìm thấy các đối số hàm, địa chỉ trả về (giúp bạn xây dựng lại lịch sử gọi hàm) và các biến cục bộ.
- Các tệp đã ánh xạ và dữ liệu ứng dụng: Phần này được để lại mơ hồ một chút vì nội dung thực sự phụ thuộc vào tiến trình. Các tệp đã ánh xạ đại diện cho nội dung từ các tệp trên đĩa, có thể là dữ liệu cấu hình, tài liệu, v.v. Dữ liệu ứng dụng là bất kỳ thông tin nào mà tiến trình cần để thực hiện công việc của mình.
- Thực thi: Chương trình thực thi của tiến trình chứa mã chính và biến đọc/viết cho ứng dụng. Dữ liệu này có thể được nén hoặc mã hóa trên đĩa, nhưng sau khi được tải vào bộ nhớ, nó sẽ được giải nén, cho phép bạn ghi mã văn bản thuần khiết trở lại đĩa.

>**LƯU Ý:**<br>
Để truy vấn địa chỉ người dùng cao nhất bằng debugger Microsoft, bạn gõ lệnh sau:<br>
>>		kd> dq nt!MmHighestUserAddress L1 <br>
>>		fffff802`821da040 000007ff`fffeffff<br>
>Để lấy cùng một giá trị từ một bản sao bộ nhớ với Volatility, bạn có thể sử dụng các lệnh volshell sau:<br>
>> 		>>> kdbg = win32.tasks.get_kdbg(addrspace())
>> 		>>> addr = kdbg.MmHighestUserAddress.dereference_as("address")
>>		>>> hex(addr)
>>		'0x7fffffeffffL'
> địa chỉ cao nhất trên hệ thống 64-bit đó là 0x7FFFFFEFFFF, không phân biệt bạn xem nó từ trình gỡ lỗi (debugger) hay Volatility

### Memory Allocation APIs

Hình 7-2 thể hiện sơ đồ các API được sử dụng để cấp phát bộ nhớ cho quá trình và cách các chức năng liên quan đến nhau. Tương tự như nhiều thành phần khác trong hệ điều hành, nhiều giao diện trừu tượng cấp cao được đặt trên cùng của các API nguyên gốc. Mô hình này cung cấp khá nhiều tính linh hoạt cho các nhà lập trình, cùng với khả năng cho các phân hệ như bộ quản lý heap tồn tại ở giữa và giúp đảm bảo rằng việc cấp phát và giải phóng các khối bộ nhớ nhỏ diễn ra một cách hiệu quả. Các chức năng gọi lẫn nhau theo cách này được kết nối với nhau bằng các mũi tên.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20194235.png)

Chúng tôi cung cấp biểu đồ các API này để giúp hình dung cách tất cả các chức năng đều dẫn đến NtAllocateVirtualMemory cuối cùng. Sinh viên trong lớp học đào tạo của chúng tôi thường hỏi về sự khác biệt giữa chức năng new trong C++ và các API như VirtualAlloc. Như được hiển thị trong biểu đồ, cả hai đều dẫn đến một cuộc gọi đến NtAllocateVirtualMemory; tuy nhiên, new sẽ đi qua HeapAlloc trước tiên, điều này có nghĩa là bộ nhớ được yêu cầu được cấp phát từ một trong các heap hiện có của quá trình. Điều này hữu ích từ góc độ lý thuyết (giúp bạn hiểu rõ hơn về bộ nhớ nội tại) và cũng từ góc độ pháp y. Ví dụ, nếu bạn thấy mã độc sử dụng new và sau đó muốn tìm các nội dung của nó sau khi xảy ra sự kiện, bạn có thể tập trung vào các đoạn heap thay vì toàn bộ bộ nhớ của quá trình.

Nhìn chung, các API trong biểu đồ khác nhau như đã được thảo luận trong các phần sau.

#### Permissions

Chỉ có một số API cho phép người lập trình có đầy đủ quyền kiểm soát quyền truy cập vào bộ nhớ được cấp phát. Ví dụ, VirtualAlloc cho phép bạn chỉ định liệu bộ nhớ có nên không được truy cập, có thể đọc, có thể ghi, có thể thực thi, bảo vệ, hoặc kết hợp các quyền truy cập đó. Tuy nhiên, với các heap, nó luôn có thể đọc và có thể ghi, và không có cách nào để tắt điều đó. Tuy nhiên, bạn có thể kiểm soát việc dữ liệu trên heap có thể thực thi bằng cách cung cấp tham số HEAP_CREATE_ENABLE_EXECUTE cho HeapCreate. Tuy nhiên, đây là một tùy chọn nguy hiểm vì nó giúp thực hiện heap spray - một trong các kỹ thuật tấn công mà Data Execution Prevention (DEP) nhằm ngăn chặn.

#### Scope and Flexibility

VirtualAllocEx là duy nhất một hàm API trên Windows trong sơ đồ cho phép một tiến trình cấp phát bộ nhớ cho một tiến trình khác. Chữ "Ex" trong tên hàm đại diện cho "Extra" vì hàm này có một tham số bổ sung mà VirtualAlloc không có - một handle mở đến tiến trình đích. Thường thấy cách sử dụng này được sử dụng như một bước chuẩn bị trước khi tiến hành tiêm mã vào tiến trình mục tiêu.

Cả hai hàm cấp phát ảo này cũng là duy nhất cho phép người gọi đặt trước bộ nhớ (reserve memory), có nghĩa là dự trữ bộ nhớ trước khi cấp phát nó. Điều này cho phép ứng dụng "lưu giữ" một khu vực lớn của bộ nhớ ảo liên tục cho việc sử dụng sau này, mà không làm ô nhiễm các trang vật lý phía dưới trong thời gian chờ đợi. Để biết thêm thông tin, xem "Reserving and Committing Memory": http://msdn.microsoft.com/en-us/library/windows/desktop/aa366803(v=vs.85).aspx.

>**LƯU Ý:**<br> Sau khi các khoảng bộ nhớ được cấp phát với VirtualAlloc(Ex), các chương trình trên hệ thống đang chạy có thể liệt kê chúng bằng cách sử dụng VirtualQueryEx. Điều này là cách Dexter và BlackPOS (mã độc sử dụng trong cuộc tấn công Target về điểm bán hàng) xác định các khoảng bộ nhớ có sẵn trước khi quét tìm số thẻ tín dụng (xem http://volatility-labs.blogspot.com/2014/01/comparingdexter-and-blackpos-target.html).

## Enumerating Process Memory

Dưới đây là mô tả về một số nguồn dữ liệu bộ nhớ quy trình mà bạn có thể sử dụng để liệt kê các khu vực bộ nhớ bằng các công cụ pháp y.
- Bảng trang (Page tables): Chương 1 đã giới thiệu về bảng trang cho một số kiến trúc phổ biến nhất (x86, x64). Đây là các cấu trúc dữ liệu cụ thể cho từng CPU. Bạn có thể sử dụng bảng trang để ánh xạ địa chỉ ảo trong bộ nhớ quy trình thành vị trí vật lý trong RAM, xác định các trang đã bị ghi đè lên đĩa, và phân tích các quyền dựa trên phần cứng được áp dụng cho các trang đó.
- Bộ mô tả địa chỉ ảo (Virtual address descriptors - VADs): VADs là các cấu trúc được định nghĩa bởi Windows để theo dõi các bộ nhớ được dự trữ hoặc đã được cam kết, có các bộ trang ảo có liên tiếp. Ví dụ, nếu một trang có kích thước 4KB và một quy trình cam kết 10 trang cùng một lúc, một VAD sẽ được tạo ra trong bộ nhớ kernel để mô tả khoảng bộ nhớ 40KB đó. Nếu khu vực chứa một tệp được ánh xạ vào bộ nhớ, VAD cũng lưu trữ thông tin về đường dẫn của tệp đó.
- Danh sách bộ nhớ đang hoạt động (Working set list): Danh sách bộ nhớ đang hoạt động của một quy trình mô tả các trang gần đây được truy cập trong bộ nhớ ảo và hiện diện trong bộ nhớ vật lý (không bị ghi đè lên đĩa). Nó có thể hữu ích cho mục đích gỡ lỗi hoặc để so sánh chéo với các nguồn dữ liệu bộ nhớ quy trình khác, nhưng thường không được sử dụng trong pháp y. Không giống như bảng trang, danh sách bộ nhớ đang hoạt động không bao giờ chứa tham chiếu đến bộ nhớ không thể chuyển đổi hoặc các trang lớn, do đó bạn không thể phụ thuộc vào chúng để cung cấp danh sách toàn diện các trang có thể truy cập được của một quy trình. Ngoài ra, danh sách bộ nhớ đang hoạt động có thể được làm rỗng theo yêu cầu (xem API EmptyWorkingSet).
- Cơ sở dữ liệu PFN: Windows sử dụng cơ sở dữ liệu PFN để theo dõi trạng thái của từng trang trong bộ nhớ vật lý, điều này có thể cung cấp cho bạn cái nhìn riêng biệt về cách bộ nhớ đang được sử dụng vì bảng trang, VADs và danh sách bộ nhớ đang hoạt động tập trung vào bộ nhớ ảo. Cái gọi là cơ sở dữ liệu thực tế là một mảng của các cấu trúc _MMPFN mà bạn có thể truy cập từ khối dữ liệu debug kernel (_KDEBUGGER_DATA64.MmPfnDatabase). Để biết thêm thông tin, xem bài viết Mining the PFN Database for Malware Artifacts của George Garner: http://volatility-labs.blogspot.com/2012/10/omfw-2012-mining-pfn-database-for.html.

Chúng ta sử dụng các nguồn thông tin khác nhau vì chúng có thể bổ sung lẫn mâu thuẫn lẫn nhau. Ví dụ, bảng trang lưu trữ các bảo vệ truy cập phần cứng chính xác, trong khi VADs chứa tên của các tệp được ánh xạ. Để có được nhiều thông tin nhất có thể, bạn phải truy vấn cả hai nguồn thông tin. Tuy nhiên, nếu malware gỡ bỏ một nút VAD khỏi cây để ẩn một phạm vi bộ nhớ, các mục nhập bảng trang tương ứng vẫn tồn tại - vì vậy bạn có thể phát hiện ra một "lỗ hổng" trong bộ nhớ quy trình. Tương tự như vậy, như bài thuyết trình của Garner đã chỉ ra, các khung cấp trên cùng trong các mục nhập cơ sở dữ liệu PFN chứa các tham chiếu ngược lại đến quy trình sở hữu. Do đó, bất kể _EPROCESS được ẩn đến đâu, bạn có thể tiềm năng tìm thấy nó, và bộ nhớ của nó, bằng cách xem trong cơ sở dữ liệu.

### Process Page Tables

Giữa Chương 1 và tóm tắt trước đó, bạn đã có một cái nhìn tổng quan về vai trò của các bảng trang trong quản lý bộ nhớ. Vì vậy, bạn có thể tiếp tục vào một số kịch bản phân tích để làm quen với việc sử dụng thực tế của các bảng trang trong công việc điều tra dữ liệu trong bộ nhớ.

#### Exploring Process Memory

Các plugin memmap và memdump cho phép bạn liệt kê và trích xuất tất cả các trang được truy cập bởi một quy trình. Trong trường hợp này, "được truy cập" bao gồm cả các địa chỉ chế độ kernel vì các luồng mà bắt đầu trong bộ nhớ người dùng sẽ chuyển sang bộ nhớ kernel khi gọi các API hệ thống. Do đó, dù bạn có thể không mong đợi thấy các địa chỉ kernel trong một bản đồ của bộ nhớ quy trình, đó không phải là một lỗi.

Hình 7-3 cho thấy phạm vi dữ liệu mà các plugin này báo cáo. Hãy chú ý đến những khoảng trắng lớn màu trắng, đại diện cho các vùng trống hoặc dự trữ. Bạn cũng có thể thấy các lỗ nhỏ trong các khu vực đã cam kết, cho thấy các trang không nằm trong bộ nhớ vì việc swap.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20195853.png)

Dưới đây là một ví dụ về cách đầu ra của lệnh memmap có thể hiển thị:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20200048.png)

Lưu ý các điểm sau về đầu ra của plugin này:
- Các địa chỉ ảo 0x1000 và 0x2000 tương ứng với các địa chỉ vật lý 0x151162000 và 0x158de3000. Điều này chứng tỏ rằng các trang ảo tương đồng không liên tiếp trong bộ nhớ vật lý.
- Địa chỉ ảo 0x7ffe0000 là vùng KUSER_SHARED_DATA - một khối bộ nhớ được tạo ra bởi kernel và chia sẻ với tất cả các quy trình. Giá trị của 0x7ffe0000 được cố định trong Windows, do đó không thay đổi trên mỗi hệ thống. Nếu bạn sử dụng memmap trên các quy trình khác, bạn sẽ thấy rằng tất cả đều có một bản đồ giống nhau. Điều này là một cách để liệt kê các trang được chia sẻ (xem phần "Phát hiện trang được chia sẻ" trong Chương 17 để xem một ví dụ khác).
- Có một khoảng trống lớn sau vùng KUSER_SHARED_DATA. Không có các trang khác khả dụng cho đến 0xff4c0000. Những địa chỉ ảo này có thể chưa được thực hiện hoặc chúng đã được đẩy ra.
- Mặc dù kích thước trang mặc định của hệ thống là 4KB (0x1000 byte hex), bạn có thể thấy một số trang có kích thước là 0x200000, tương đương 2MB. Đây là các trang PSE (Page Size Entry).

Một điểm quan trọng khác về đầu ra của memmap là cột DumpFileOffset ở bên phải. Giá trị này chỉ định vị trí offset của trang tương ứng trong tệp được tạo bởi plugin memdump. Vì không gian địa chỉ của quy trình là thưa thớt - tồn tại lỗ hoặc khoảng trống giữa các trang khả dụng - tệp đầu ra cũng sẽ thưa thớt. Ví dụ, dữ liệu tại địa chỉ ảo 0xff4c1000 được ánh xạ vào offset vật lý 0x151b77000 (5.2GB), nhưng tệp đầu ra của bạn cho quy trình này sẽ không lớn như vậy. Do đó, cột DumpFileOffset cho biết nơi để tìm nội dung của 0xff4c1000 trong tệp đầu ra của bạn (0x2a3000). Dưới đây là một ví dụ về việc sử dụng plugin memdump:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20212022.png)

Như bạn có thể thấy, tệp kết quả (864.dmp) có kích thước là 434MB. Nếu bạn sử dụng volshell để xem nội dung của địa chỉ ảo 0xff4c1000, bạn nên có thể khớp nó với dữ liệu tại offset 0x2a3000 trong tệp 864.dmp. Dưới đây là một kiểm tra nhanh để kiểm tra tính đúng đắn:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20212459.png)

Ở điểm này, bạn nên có thể xác định được các trang (pages) nào có thể truy cập được bởi một tiến trình và chúng được ánh xạ đến đâu trong bộ nhớ vật lý. Bạn đã biết cách dump bộ nhớ có sẵn vào một tệp duy nhất trên đĩa, sau đó bạn có thể xử lý nó với các công cụ bên ngoài như các chương trình quét vi-rút. Cái duy nhất mà bạn đang còn thiếu là khả năng kết hợp các địa chỉ trong bộ nhớ ảo hoặc vật lý với tên của các tệp được ánh xạ, DLL hoặc các chương trình thực thi chiếm giữ không gian đó. Ví dụ, nếu công cụ quét của bạn cho biết nó đã tìm thấy một chữ ký xấu đã biết tại vị trí offset 0x2a3030 trong tệp 864.dmp, bạn có thể gặp khó khăn khi cố gắng phân loại dữ liệu tiếp theo. Đây là lúc VADs rất hữu ích, chúng ta sẽ thảo luận về điều này tiếp theo.

### Virtual Address Descriptors

Cây VAD (Virtual Address Descriptor) của một tiến trình mô tả cấu trúc bộ nhớ của nó ở mức độ cao hơn so với các bảng trang. Các cấu trúc dữ liệu này được định nghĩa và duy trì bởi hệ điều hành, chứ không phải CPU. Do đó, hệ điều hành có thể điền thông tin vào các nút VAD về các dải bộ nhớ gốc mà CPU không cần thiết phải quan tâm. Ví dụ, VAD chứa tên của các tệp được ánh xạ vào bộ nhớ, tổng số trang trong vùng, quyền bảo vệ ban đầu (đọc, ghi, thực thi) và một số cờ khác có thể cho bạn biết nhiều về loại dữ liệu mà các vùng chứa.

Như được hiển thị trong Hình 7-4, VAD là một cây nhị phân tự cân bằng (xem The VAD Tree: A Process-Eye View of Physical Memory của Brendan Dolan-Gavitt tại http://www.dfrws.org/2007/proceedings/p62-dolan-gavitt.pdf). Mỗi nút trong cây đại diện cho một dải trong bộ nhớ ảo của tiến trình. Một nút mô tả một dải bộ nhớ thấp hơn nút cha xuất hiện ở bên trái, và một nút mô tả dải cao hơn xuất hiện ở bên phải. Cấu trúc này giúp làm cho việc tìm kiếm, thêm và xóa hiệu quả hơn so với việc sử dụng danh sách đơn hoặc danh sách liên kết kép.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230605.png)

Sơ đồ này được tạo ra bằng plugin vadtree của Volatility với tùy chọn --render=dot, sẽ được đề cập nhiều hơn trong vài trang tiếp theo. Bên cạnh việc vẽ các mối quan hệ giữa các nút VAD, nó cũng được mã màu theo nội dung của chúng. Ví dụ, các vùng nhớ của tiến trình được mã màu đỏ, các ngăn xếp của luồng được mã màu xanh lá cây, các tệp được ánh xạ được mã màu vàng và các DLL được mã màu xám, giúp bạn dễ dàng hình dung được loại dữ liệu được ánh xạ vào bộ nhớ của tiến trình.

#### VAD Structures

Đối với mỗi tiến trình, _EPROCESS.VadRoot trỏ tới gốc của cây VAD. Mặc dù tên thành viên này giữ nguyên qua tất cả các phiên bản Windows từ XP đến 8.1, kiểu dữ liệu của thành viên này đã thay đổi thường xuyên, cũng như tên của các nút VAD. Sự khác biệt được thể hiện trong Bảng 7-1. Ví dụ, trên Windows XP và 2003 Server, VadRoot trỏ tới một _MMVAD_SHORT, _MMVAD, hoặc _MMVAD_LONG. Hạt nhân quyết định chính xác cấu trúc nào sẽ được tạo ra dựa trên API được sử dụng để cấp phát và các tham số được truyền vào API. Cùng nhau, thông tin này đủ để suy ra mục đích dự kiến của vùng nhớ - liệu nó sẽ lưu trữ một tệp được ánh xạ, DLL hoặc chỉ là một phân bổ riêng tư. Mỗi nút có một LeftChild và RightChild, chúng chỉ định hướng bạn đi xuống các nhánh khác nhau trong cây.

Như bạn có thể thấy, bắt đầu từ Vista, VadRoot đã được thiết kế lại thành một _MM_AVL_TABLE, và nó đã thay đổi lại lần nữa trong Windows 8.1 thành một _RTL_AVL_TREE. Trong quá trình này, các nút cũng đã được thiết kế lại ba lần. Tuy nhiên, mặc dù có vẻ như có những thay đổi đáng kể, VAD luôn giữ nguyên một cây, và các thuật toán để liệt kê các nút cũng đã giữ nguyên tương đối giống nhau. Trên thực tế, như đã gợi ý trong Bảng 7-1, các tên nút khác nhau về cơ bản chỉ là các bí danh của các cấu trúc _MMVAD[_SHORT,_LONG] ban đầu từ XP. Khi bạn phân tích cây VAD bằng Volatility, những thay đổi này được xử lý một cách trong suốt.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230627.png)

Dưới đây là các cấu trúc liên quan từ một máy Windows 7 64-bit. Bạn có thể thấy rằng _MM_AVL_TABLE có một thành viên BalancedRoot, là một _MMADDRESS_NODE. Mỗi nút có một tập hợp con trỏ tới các nút con của nó và một StartingVpn và EndingVpn. Từ những số trang ảo (VPN) này, bạn có thể tạo ra địa chỉ của trang đầu tiên và trang cuối cùng trong bộ nhớ ảo của tiến trình mục tiêu. Chúng tôi nói "tạo ra" vì các VPN là các số trang, không phải địa chỉ. Để có được địa chỉ, bạn phải nhân số trang với kích thước của một trang.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230655.png)

Bảng 7-1 đặt _MMADDRESS_NODE là một bí danh vì nó thực sự là một trong các cấu trúc _MMVAD*. Hãy chú ý trong các danh sách sau đây rằng nút ngắn ( _MMVAD_SHORT) bắt đầu bằng các thành viên giống nhau như _MMADDRESS_NODE. Tương tự, nút thường ( _MMVAD) và nút dài ( _MMVAD_LONG) xây dựng lên các cấu trúc nhỏ hơn, nhưng bao gồm các thành viên bổ sung ở cuối. Đặc biệt, chúng thêm thành viên Subsection, mà hệ điều hành sử dụng để theo dõi thông tin về các tệp hoặc DLL được ánh xạ vào vùng này:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230726.png)

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230741.png)

Thảo luận này giúp bạn bắt đầu hiểu cách phân loại mục tiêu tiềm năng của một dãy bộ nhớ dựa trên loại nút. Ví dụ, một nút ngắn không có mục Subsection, vì vậy nó không thể lưu trữ một tệp được ánh xạ. Trong khi đó, mã shell được tiêm vào một quá trình không bao giờ cần tồn tại trên đĩa. Do đó, nó sẽ không được hỗ trợ bởi một tệp. Kết quả là, nếu bạn đang săn tìm mã tiêm, bạn có thể bỏ qua các nút thông thường và dài vì hệ điều hành thường không chọn một trong những cấu trúc lớn hơn nếu các thành viên bổ sung mà chúng cung cấp sẽ không bao giờ được sử dụng - điều này sẽ là một lãng phí.

#### VAD Tags

Sau khi đọc các thảo luận trước đó, bạn có thể tự hỏi Volatility làm thế nào để xác định cấu trúc _MMVAD* nào trong ba cấu trúc được alias bởi _MMADDRESS_NODE. Câu trả lời nằm ở thành viên Tag của các cấu trúc _MMVAD*. Chú ý rằng vị trí của thành viên này là -0xc (hoặc cách cấu trúc bắt đầu 12 byte) đối với các nền tảng 64-bit. Trong các ngôn ngữ lập trình thông thường, chẳng hạn như C và C++, bạn không bao giờ thấy các thành viên tại các offset âm. Trên thực tế, đây là một hack thuận tiện được cho phép khi định nghĩa các cấu trúc với Volatility. Nói cách khác, chúng ta đang truy cập vào thành viên PoolTag của _POOL_HEADER, nằm trực tiếp trước nút trong bộ nhớ. Nếu bạn nhớ từ Chương 5, Windows gán các thẻ cho các vùng nhớ để chỉ định loại dữ liệu chúng chứa. Bảng 7-2 liệt kê các mục liên quan.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230758.png)

Các vùng nhớ chứa mã shell được tiêm vào thường không được hỗ trợ bởi một tệp. Do đó, bạn sẽ tìm kiếm các nút có thẻ VadS hoặc VadF. Giả sử bạn có một phiên bản của cấu trúc nút (chúng tôi sẽ chỉ cho bạn cách liệt kê chúng trong thời gian ngắn) trong biến vad, bạn có thể dễ dàng in hoặc kiểm tra giá trị bằng cách trích dẫn vad.Tag trong mã Python của bạn.

#### VAD Flags

Mỗi nút có một hoặc nhiều tập hợp cờ chứa thông tin về phạm vi bộ nhớ. Các cờ này được đặt trong các liên minh nhúng có tên là u, u1, u2, u3, và cứ tiếp tục như vậy. Ví dụ, thành viên u của nút ngắn có kiểu __unnamed_15c2. Cách đặt tên này dựa trên các trường <unnamed-tag> được tìm thấy trong ký hiệu gỡ lỗi của Microsoft vì các liên minh không có kiểu tương ứng. Để cho phép bạn xác định chúng một cách duy nhất trong ngôn ngữ kiểu của Volatility, một số nguyên được đính kèm vào cuối (phần _15c2). Liên minh cụ thể này có dạng như sau:

```
>>> dt("__unnamed_15c2")
'__unnamed_15c2' (8 bytes)
0x0 : LongFlags         ['unsigned long long']
0x0 : VadFlags          ['_MMVAD_FLAGS']
```

Cả hai thành viên đều tồn tại tại vị trí offset 0, do đó chúng chiếm cùng một không gian. Bạn có thể tham chiếu đến cờ của một nút bằng cách sử dụng giá trị 8 byte đầy đủ (_MMVAD.u.LongFlags) hoặc từng cái một bằng cách truy cập vào các trường bit đã được định nghĩa trước của _MMVAD.u.VadFlags. Cấu trúc sau đây hiển thị các cờ có thể có. Ví dụ, các bit từ 0 đến 51 của giá trị 8 byte là cho CommitCharge, và các bit từ 56 đến 61 là cho Protection.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230820.png)

**Các phần dưới đây mô tả một số trường quan trọng hơn:**

##### CommitCharge

 CommitCharge là trường quy định số lượng trang đã được cấp phát (commit) trong vùng bộ nhớ mà nút VAD mô tả. Trường này tương tự như MemCommit, trường này cho biết liệu bộ nhớ đã được cấp phát khi API cấp phát bộ nhớ ảo (NtAllocateVirtualMemory) được gọi lần đầu. Một điều quan trọng cần lưu ý là khi mã độc tiêm mã vào quy trình mục tiêu, nó thường tiến hành cấp phát tất cả các trang bộ nhớ một lần duy nhất - tức là cấp phát và cấp phát luôn (commit) mà không cần dự trữ trước và sau đó mới cấp phát. Mặc dù điều này hoàn toàn có thể thay đổi tùy thuộc vào cách thức mã độc thực hiện tiêm mã. Tuy nhiên, thông thường, sẽ cấp phát và commit từ đầu. Vì vậy, thông tin về CommitCharge và MemCommit có thể hỗ trợ trong việc xác định các vùng bộ nhớ bị tiêm mã độc.

##### Protection

Trường này xác định loại quyền truy cập được phép cho vùng bộ nhớ. Giá trị này tương đối phù hợp với các hằng số bảo vệ bộ nhớ được truyền cho các API cấp phát bộ nhớ ảo. Những hằng số này (được hiển thị trong danh sách sau đây) khá dễ hiểu phần lớn, nhưng thực sự có một số thông tin tinh tế có thể giúp bạn xác định loại dữ liệu trong một vùng bộ nhớ không xác định. Ví dụ, bạn không thể sử dụng PAGE_EXECUTE để lưu trữ các tệp được ánh xạ, trong khi PAGE_EXECUTE_WRITECOPY chỉ hợp lệ đối với các tệp được ánh xạ - thường là các tệp DLL.

- PAGE_EXECUTE: Bộ nhớ có thể được thực thi, nhưng không thể ghi. Quyền bảo vệ này không thể được sử dụng cho các tệp được ánh xạ.
- PAGE_EXECUTE_READ: Bộ nhớ có thể được thực thi hoặc đọc, nhưng không thể ghi.
- PAGE_EXECUTE_READWRITE: Bộ nhớ có thể được thực thi, đọc hoặc ghi. Các vùng mã được chèn thông thường đều có quyền bảo vệ này.
- PAGE_EXECUTE_WRITECOPY: Cho phép thực thi, chỉ đọc hoặc ghi vào một tệp được ánh xạ và tạo bản sao khi ghi. Không thể thiết lập bằng cách gọi VirtualAlloc hoặc VirtualAllocEx. Các DLL hầu như luôn có quyền bảo vệ này.
- PAGE_NOACCESS: Vô hiệu hóa toàn bộ quyền truy cập vào bộ nhớ. Quyền bảo vệ này không thể được sử dụng cho các tệp được ánh xạ. Ứng dụng có thể ngăn không cho đọc/ghi dữ liệu một cách vô tình bằng cách thiết lập quyền bảo vệ này.
- PAGE_READONLY: Bộ nhớ có thể được đọc, nhưng không thể thực thi hoặc ghi.
- PAGE_READWRITE: Bộ nhớ có thể được đọc hoặc ghi, nhưng không thể thực thi.
- PAGE_WRITECOPY: Cho phép chỉ đọc hoặc ghi vào một tệp được ánh xạ và tạo bản sao khi ghi. Không thể thiết lập bằng cách gọi VirtualAlloc hoặc VirtualAllocEx.

Một trong những khía cạnh dễ gây nhầm lẫn và được tài liệu giải thích không rõ ràng nhất của trường Protection trong cờ VAD là rằng nó chỉ là bảo vệ ban đầu được chỉ định cho tất cả các trang trong phạm vi bộ nhớ khi chúng được dự trữ hoặc cam kết lần đầu tiên. Do đó, quyền hạn hiện tại có thể khác biệt một cách đáng kể. 

Ví dụ, bạn có thể gọi hàm VirtualAlloc và dự trữ mười trang với PAGE_NOACCESS. Sau đó, bạn có thể cam kết ba trang với quyền hạn PAGE_EXECUTE_READWRITE và bốn trang khác với quyền hạn PAGE_READONLY, nhưng trường Protection vẫn giữ giá trị PAGE_NOACCESS. Các phiên bản cũ hơn của mã độc Zeus từ năm 2006 đã sử dụng kỹ thuật này, dẫn đến việc các khu vực bộ nhớ bị chèn mã (injected memory regions) xuất hiện với quyền hạn PAGE_NOACCESS. Điều này có ý nghĩa vì chỉ có một giá trị Protection cho tất cả các trang trong phạm vi, trong khi quyền hạn có thể được áp dụng cho từng trang cụ thể. Do đó, không ngạc nhiên nếu bạn tìm thấy mã chèn, cần phải có quyền thực thi, ẩn trong một nút mà cho rằng tất cả các trang của nó không thể truy cập được hoặc chỉ có thể đọc. Để có cái nhìn đáng tin cậy về quyền hạn trang, bạn cần tham khảo các bit trong bảng trang (page table). Bảng trang lưu trữ các quyền hạn dựa trên phần cứng cho từng trang trong bộ nhớ và cung cấp các quyền hạn cho từng trang cụ thể. Tóm lại, trường Protection trong cờ VAD có thể hữu ích, nhưng chỉ khi bạn nhận thức về giới hạn đáng tin cậy của nó. Các công cụ chạy trên hệ thống thời gian thực và sử dụng VirtualQueryEx không gặp vấn đề này, vì API không trực tiếp truy vấn giá trị Protection. Thay vào đó, chúng truy vấn các mục nhập bảng trang thực tế để xác định quyền hạn cho từng trang.

>**LƯU Ý:**<br> Để có thêm thông tin chi tiết, bạn có thể tham khảo các Mã hằng bảo vệ bộ nhớ tại địa chỉ http://msdn.microsoft.com/en-us/library/windows/desktop/aa366786(v=vs.85).aspx. Ngoài ra, trường Protection không hoàn toàn giống với các hằng số này, mà thực tế là một chỉ mục vào nt!MmProtectToValue, đó là một bảng tra cứu (lookup table) các vị trí trong đó lưu trữ các giá trị hằng số. Mối quan hệ này được mô tả trong Recipe 16-4 của Malware Analyst's Cookbook.

##### Private Memory

Bộ nhớ riêng tư (private memory), trong ngữ cảnh này, đề cập đến các vùng đã được cam kết (committed) và thường không thể chia sẻ hoặc kế thừa bởi các quy trình khác. Các tệp được ánh xạ, bộ nhớ chia sẻ có tên và các DLL copy-on-write có thể được chia sẻ với các quy trình khác (mặc dù có thể không). Do đó, nếu cờ PrivateMemory được đặt cho một vùng bộ nhớ, nó không chứa một trong các loại dữ liệu đã nêu trên. Các heaps, stacks và các phạm vi được cấp phát bằng cách sử dụng VirtualAlloc hoặc VirtualAllocEx thường được đánh dấu là riêng tư. Như đã mô tả trước đây, vì VirtualAllocEx được sử dụng để cấp phát bộ nhớ trong một quy trình từ xa, thành viên PrivateMemory là một yếu tố khác bạn có thể xem xét khi tìm kiếm mã shell bị tiêm vào.

#### Volatility VAD Plugins

Volatility cung cấp các plugin sau để kiểm tra các VAD (Virtual Address Descriptor) như sau:

- vadinfo: Hiển thị đầu ra chi tiết nhất, bao gồm các địa chỉ bắt đầu và kết thúc, mức bảo vệ (protection level), cờ (flags) và đường dẫn đầy đủ đến các tệp đã được ánh xạ hoặc DLL.

- vadtree: Ở chế độ văn bản, plugin này in ra cây VAD, cho phép bạn xem các mối quan hệ cha con trên bảng điều khiển của bạn. Nó cũng hỗ trợ tạo đồ thị được mã màu như được hiển thị trong Hình 7-4.

- vaddump: Trích xuất khoảng bộ nhớ quá trình mà mỗi nút VAD miêu tả ra một tệp riêng trên đĩa. Khác với memmap (đã thảo luận trước đó), đầu ra của plugin này được lấp đầy bằng các số không nếu bất kỳ trang nào trong phạm vi được ghi nhớ trên đĩa để duy trì tính toàn vẹn không gian (offsets).

Dưới đây là một ví dụ về đầu ra của vadinfo cho một mẫu Windows 7 64-bit. Tất nhiên, mỗi quá trình chứa hàng trăm vùng, vì vậy chúng ta chỉ lựa chọn một số ví dụ để giới thiệu.

Đoạn đầu tiên cho thấy bạn có thể tìm thấy nút VAD tại địa chỉ 0xfffffa80012184a0 trong bộ nhớ kernel và thẻ là VadS, điều này có nghĩa là loại cấu trúc là _MMVAD_SHORT. Nút này miêu tả phạm vi 0x50000 - 0x51fff trong bộ nhớ quá trình. Dựa vào kích thước của phạm vi này (hai trang), và việc đặt MemCommit và CommitCharge là 2, bạn biết rằng cả hai trang đều được cam kết với lần gọi đầu tiên của API phân bổ ảo. Dựa vào bảo vệ, bộ nhớ này có thể đọc và ghi, nhưng không thể thực thi (không thể thực hiện mã).

```
$ python vol.py –f memory.dmp --profile=Win7SP1x64 vadinfo –p 1080
Volatility Foundation Volatility Framework 2.4
 
 [snip]

VAD node @ 0xfffffa80012184a0
Start 0x0000000000050000 End 0x0000000000051fff Tag VadS
Flags: CommitCharge: 2, MemCommit: 1, PrivateMemory: 1, Protection: 4
Protection: PAGE_READWRITE
Vad Type: VadNone
```

Mục tiếp theo tương tự như mục đầu tiên vì cũng là một nút ngắn. Nó miêu tả bộ nhớ trong phạm vi từ 0x7f0e0000 đến 0x7ffdffff, tương đương khoảng 15MB. Tuy nhiên, các cờ CommitCharge và MemCommit không xuất hiện, điều này có nghĩa là cả hai đều là số không. Do đó, phạm vi bộ nhớ này chỉ được đặt trước - hệ điều hành không kết hợp nó với bất kỳ trang vật lý nào:

```
VAD node @ 0xfffffa8000e17460
Start 0x000000007f0e0000 End 0x000000007ffdffff Tag VadS
Flags: PrivateMemory: 1, Protection: 1
Protection: PAGE_READONLY
Vad Type: VadNone
```

Các nút tiếp theo mô tả các phạm vi là 0x4200000 - 0x4207fff và 0x77070000 - 0x77218fff trong bộ nhớ của quy trình. Nhãn cho cả hai nút đều là Vad, có nghĩa là chúng đang sử dụng một trong các cấu trúc _MMVAD lớn hơn cho phép các tệp được ánh xạ vào vùng này và sau đó được chia sẻ với các tiến trình khác. Thực sự, đó là điều bạn thấy - index.dat và ntdll.dll tồn tại tại các vị trí này. Mặc dù tương tự nhau, chỉ có một trong hai ánh xạ (cho ntdll.dll) có thể được thực thi. Bạn có thể nhận biết điều này bằng cách kiểm tra bảo vệ là PAGE_EXECUTE_WRITECOPY, loại là VadImageMap và bit Image trong các cờ điều khiển được đặt:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230857.png)

Việc sử dụng plugin vadinfo là một cách tuyệt vời để truy vấn thông tin chi tiết về bộ nhớ của quy trình. Giả sử bạn đã tìm thấy một vùng dữ liệu mà bạn quan tâm, có thể do bạn nhận ra tệp được ánh xạ là một trong số các chỉ báo trên danh sách của bạn, hoặc do bạn tìm thấy một con trỏ hoặc hiện vật khác đưa bạn đến một địa chỉ trong phạm vi mà nút VAD mô tả. Ví dụ sau cho thấy cách sử dụng volshell để chéo tham chiếu những gì nút VAD nói cho bạn về vùng dữ liệu (nó nên chứa index.dat) với dữ liệu thực sự tồn tại ở đó. Như được thể hiện, tại địa chỉ 0x4200000 trong quy trình này, tiêu đề bộ nhớ cache lịch sử URL của IE thực sự có mặt:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230918.png)

Bạn có thể lưu toàn bộ vùng cụ thể này vào đĩa bằng cách sử dụng tùy chọn --base của vaddump, như được hiển thị dưới đây:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230936.png)

Để mặc định, nếu bạn không chỉ định --base, plugin sẽ lưu tất cả các vùng vào các tệp riêng biệt và sau đó đặt tên cho chúng dựa trên vị trí trong bộ nhớ tiến trình mà chúng được tìm thấy. Hình 7-5 hiển thị một sơ đồ về những gì plugin này trích xuất. Bạn có thể so sánh sơ đồ này với memmap trong Hình 7-3 để xem sự khác biệt. Đặc biệt, vaddump cung cấp một tệp được lấp đầy số không cho mỗi vùng của bộ nhớ tiến trình thay vì một tệp duy nhất được nén chứa tất cả các trang có thể truy cập qua bộ nhớ tiến trình và kernel.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20230954.png)

Chương 9 thảo luận về plugin evtlogs, sử dụng chức năng tương tự như vaddump, nhưng chỉ trích xuất các vùng chứa các tệp nhật ký sự kiện được ánh xạ vào bộ nhớ. Nếu bạn không lấp đầy số không cho các tệp đã được trích xuất, một trang bị ghi đè có thể làm cho tất cả các vị trí offset trong tệp dịch chuyển một cách không mong muốn. Do đó, bởi vì bước tiếp theo trong phân tích thường là phân tích các vùng đã trích xuất trong các công cụ bên ngoài (bộ phân tích nhật ký sự kiện, disassembler và v.v.), việc lấp đầy số không là rất cần thiết.

#### Traversing the VAD in Python

Nhiều plugin của Volatility sử dụng VAD tree để thu thập dữ liệu từ bộ nhớ quy trình. Ví dụ, svcscan tìm kiếm thông tin về dịch vụ Windows (Chương 12) và cmdscan tìm lịch sử lệnh (Chương 17). Đôi khi, bạn cần tìm các dấu vết khác trong bộ nhớ quy trình - có thể sử dụng các plugin hiện có hoặc tự xây dựng. Dưới đây là một số ví dụ về các API bạn có thể sử dụng để giúp việc này trở nên dễ dàng hơn.

Ví dụ đầu tiên cho thấy cách lặp qua các VADs, đọc một lượng dữ liệu đã chỉ định từ mỗi phạm vi, tìm kiếm chữ ký và báo cáo kết quả. Lưu ý rằng chúng tôi cung cấp -p 1080 khi gọi plugin volshell, vì vậy chúng tôi sẽ bắt đầu trong ngữ cảnh của quy trình có PID 1080. Điều này chỉ là một phím tắt để thay đổi ngữ cảnh bằng cách sử dụng cc(pid = 1080) sau khi đã vào shell. Tiếp theo, chúng tôi gán giá trị proc() (đây là đối tượng _EPROCESS cho PID 1080) cho biến khác được đặt tên là process - chỉ cho mục đích đọc hiểu. Bạn có thể có được không gian địa chỉ của quy trình sử dụng hàm get_process_address_space() và sau đó sử dụng nó để đọc bộ nhớ quy trình:

```
$ python vol.py -f memory.dmp --profile=Win7SP1x64 volshell -p 1080
Volatility Foundation Volatility Framework 2.4
Current context: process explorer.exe, pid=1080, ppid=1452 DTB=0x19493000
To get help, type 'hh()'
>>> process = proc()
>>> process_space = process.get_process_address_space()
```

Vòng lặp sau thực hiện hầu hết công việc. Nó sử dụng API VadRoot.traverse() để tạo ra một đối tượng VAD cho mỗi node trong cây. Các node xác định số trang bắt đầu và kết thúc bằng cách sử dụng các thành viên StartingVpn và EndingVpn. Tuy nhiên, bạn phải nhân chúng với kích thước của một trang để có được địa chỉ thực tế trong bộ nhớ quy trình. Tất cả điều này được xử lý tự động thông qua các phương thức trợ giúp mà Volatility gắn vào các đối tượng VAD. Ví dụ: Start() trả về địa chỉ bắt đầu đã được tính toán chính xác. Bạn có thể thấy giá trị này được truyền vào hàm read() của không gian địa chỉ:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20231017.png)

Nếu việc đọc dữ liệu thành công, dữ liệu sẽ được quét để tìm kiếm chữ ký "MZ" (để xác định có thể là một tệp thực thi). Địa chỉ bắt đầu của phạm vi chứa chữ ký "MZ" sẽ được báo cáo trên bảng điều khiển. Các plugin khác đã được xây dựng sẵn trong Volatility (như dlllist, ldrmodules, và malfind) tự động tìm kiếm các tệp thực thi trong bộ nhớ tiến trình. Do đó, ví dụ trên giúp bạn làm quen với cách thực hiện các tìm kiếm đơn giản có thể mang lại kết quả hữu ích. Thông thường, sau khi tìm thấy dữ liệu, bạn có thể triển khai một bước xử lý sau đó để xác nhận dữ liệu, trích xuất dữ liệu ra ổ đĩa và thực hiện các phân tích bổ sung.

#### Passwords in Browser Memory

Một điểm yếu của thuật toán trước đó là nó chỉ tìm kiếm 1024 byte đầu tiên của mỗi phạm vi và chỉ báo cáo xem có tìm thấy một mẫu duy nhất hay không. Hàm search_process_memory() hiện có chức năng chấp nhận nhiều đầu vào và quét toàn bộ dữ liệu truy cập được trong phạm vi và báo cáo địa chỉ của mỗi xuất hiện. Trong ví dụ sau, chúng tôi sử dụng chức năng này để tìm các mật khẩu Gmail trong bộ nhớ của trình duyệt Google Chrome. Nếu bạn sử dụng một proxy web để ghi lại dữ liệu POST đi ra trong quá trình đăng nhập, bạn sẽ thấy rằng các tham số POST bao gồm &Email và &Passwd. Do đó, hai chuỗi này trở thành tiêu chí của chúng tôi.

Như được hiển thị trong mã dưới đây, chúng tôi bắt đầu với volshell theo cùng cách như trước đó, nhưng với PID 3660 (chrome.exe). Biến criteria là một danh sách được điền với phiên bản Unicode (utf_16_le) của hai chuỗi. Sau đó, chúng tôi lặp lại kết quả mà search_process_memory() tạo ra và tạo một đối tượng String tại mỗi địa chỉ:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-31%20231038.png)

Tên người dùng và mật khẩu của tài khoản đều hiển thị rõ ràng trong bộ nhớ tiến trình, ngay cả khi đang sử dụng SSL. Một số chuỗi có vẻ bị cắt bỏ một phần, nhưng điều này chỉ là do chúng ta đã chỉ định một độ dài tối đa là 64 khi tạo đối tượng String. Như bạn có thể tưởng tượng, các loại dữ liệu pháp y mà bạn có thể tìm thấy trong bộ nhớ tiến trình là rất đa dạng, và có nhiều khả năng về việc bạn có thể làm sau khi tìm thấy các dữ liệu như vậy (trong việc xử lý tiếp theo, phân tích tự động, v.v.).

#### Scanning Memory with Yara

Yara là một công cụ do Victor M. Alvarez phát triển (http://plusvic.github.io/yara) cho việc tìm kiếm mẫu nhanh chóng và linh hoạt trong các tập dữ liệu tùy ý. Nó hoạt động trên các tập tin, bản sao bộ nhớ, bản ghi gói tin và những tập dữ liệu khác. Một điều cần nhớ là bạn có thể dễ dàng quét qua một tập tin bản sao bộ nhớ vật lý, nhưng hãy nhớ rằng các địa chỉ ảo liền mạch có thể bị phân mảnh trong bộ nhớ vật lý. Do đó, các ký tự chữ ký của bạn có thể không khớp với các mẫu thực sự tồn tại trong bộ nhớ nếu chúng xảy ra chạm qua ranh giới trang. Plugin yarascan của Volatility cho phép bạn quét qua bộ nhớ ảo, vì vậy vấn đề phân mảnh ở tầng vật lý không còn là một vấn đề. Hơn nữa, khi bạn tìm thấy kết quả khớp, bạn có thể liên kết chúng lại với quá trình hoặc mô-đun nhân hạt nhân sở hữu bộ nhớ - mang lại ngữ cảnh và một khung tham chiếu mạnh mẽ.

