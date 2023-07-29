![image](https://github.com/HuyThang25/The-Art-of-Memory-Forensics/assets/93728466/7a947be3-1202-4dc0-9e45-c3a5a3328993)Chương này kết hợp ba bước khởi đầu phổ biến nhất trong một cuộc điều tra: xác định các ứng dụng đang chạy, những gì chúng đang thực hiện (liên quan đến việc truy cập vào tệp, khóa registry, v.v.), và ngữ cảnh bảo mật (hoặc cấp độ đặc quyền) mà chúng đã đạt được. Trong quá trình làm việc này, bạn cũng sẽ học cách phát hiện các tiến trình ẩn, cách liên kết các tiến trình với các tài khoản người dùng cụ thể, cách điều tra di chuyển chéo qua các mạng và cách phân tích các cuộc tấn công tiến hành tăng cường đặc quyền. 

Mặc dù chương này bao gồm một loạt các kỹ thuật điều tra liên quan đến tiến trình, đây chỉ là bước đầu tiên - và chủ yếu nó liên quan đến các vật dụng tồn tại trong bộ nhớ kernel. Các phương pháp phân tích liên quan đến thư viện động (DLL), bộ nhớ tiến trình, mã được tiêm vào, và các yếu tố tương tự sẽ được đề cập trong các chương tiếp theo.

## Processes

Biểu đồ trong Hình 6-1 cho thấy một số tài nguyên cơ bản thuộc về một tiến trình. Ở trung tâm là "_EPROCESS," đó là tên của cấu trúc mà Windows sử dụng để đại diện cho một tiến trình. Mặc dù tên cấu trúc này có thể khác nhau giữa Windows, Linux và Mac, nhưng tất cả các hệ điều hành chia sẻ cùng các khái niệm được miêu tả trong biểu đồ cấp cao này. Ví dụ, tất cả họ đều có một hoặc nhiều luồng thực thi mã, và đều có một bảng xử lý các handle (hoặc mô tả tập tin) đến các đối tượng kernel như các tập tin, socket mạng và mutex.

Mỗi tiến trình có không gian bộ nhớ ảo riêng tư cô lập khỏi các tiến trình khác. Bên trong không gian bộ nhớ này, bạn có thể tìm thấy thực thi tiến trình; danh sách các mô-đun đã tải (DLL hoặc thư viện chia sẻ); và các ngăn xếp, đống và vùng bộ nhớ đã cấp phát chứa mọi thứ từ đầu vào người dùng đến các cấu trúc dữ liệu cụ thể của ứng dụng (như bảng SQL, lịch sử Internet, và các tệp cấu hình). Windows tổ chức các vùng bộ nhớ này bằng các bộ mô tả địa chỉ ảo (VADs), được thảo luận trong Chương 7.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20200321.png)

Như biểu đồ cũng thể hiện, mỗi _EPROCESS cũng trỏ đến một danh sách các định danh bảo mật (SIDs) và dữ liệu đặc quyền. Đây là một trong những cách chính mà hạt nhân thực thi bảo mật và kiểm soát truy cập. Bằng cách kết hợp tất cả các khái niệm này vào quy trình điều tra của bạn, bạn có thể thu thập một lượng đáng kể các bằng chứng để giúp xác định các tiến trình nào liên quan đến hoạt động độc hại, những vật dụng nào liên quan đến một sự cố, và tài khoản người dùng nào có thể đã bị xâm nhập.

**Phân tích mục tiêu**

Mục tiêu phân tích là như sau:

- Kiến trúc bên trong tiến trình: Hiểu cách hệ điều hành theo dõi các tiến trình và cách các Windows APIs liệt kê chúng. Điều này giúp bạn hiểu tại sao các công cụ hiện trạng dễ bị lừa đảo.

- Xác định các tiến trình quan trọng: Khám phá một số tiến trình quan trọng trên Windows và tìm hiểu cách các hệ thống bình thường hoạt động. Điều này giúp bạn sẵn sàng phát hiện những dấu hiệu bất thường, đặc biệt là những cố gắng giả mạo tiến trình quan trọng.

- Tạo các hình ảnh minh họa: Học cách tạo ra các hình ảnh minh họa cho các mối quan hệ cha mẹ giữa các tiến trình. Điều này giúp bạn xác định chuỗi sự kiện dẫn đến việc một tiến trình cụ thể bắt đầu, điều này thường rất quan trọng khi ghép các sự kiện trong một vụ việc cụ thể.

- Phát hiện Manipulation trực tiếp đối tượng kernel (DKOM): Nhận biết những cố gắng ẩn giấu tiến trình bằng cách thay đổi một hoặc nhiều danh sách tiến trình trong bộ nhớ kernel. Cụ thể, bạn sẽ học bảy cách khác nhau để định vị các tiến trình trong bộ nhớ, điều này mang lại lợi thế lớn cho việc chống lại rootkit.

**Cấu trúc dữ liệu**

Windows theo dõi các tiến trình bằng cách gán cho chúng một cấu trúc _EPROCESS duy nhất, nằm trong một non-paged pool của bộ nhớ kernel. Dưới đây là cách nó xuất hiện trên một hệ thống Windows 7 64-bit:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20200928.png)

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20201041.png)

**Những điểm chính là như sau:**

- Pcb: Process control block của kernel (_KPROCESS). Cấu trúc này nằm ở đáy của _EPROCESS và chứa một số trường quan trọng, bao gồm DirectoryTableBase để dịch địa chỉ và thời gian tiến trình đã dành trong chế độ kernel và chế độ người dùng.

- CreateTime: Mốc thời gian UTC chỉ ra khi tiến trình bắt đầu chạy.

- ExitTime: Mốc thời gian UTC chỉ ra thời điểm tiến trình đã kết thúc. Giá trị này là 0 cho các tiến trình đang chạy.

- UniqueProcessId: Một số nguyên đại diện duy nhất cho tiến trình (còn được gọi là PID).

- ActiveProcessLinks: Danh sách liên kết kép kết nối các tiến trình đang hoạt động trên máy. Hầu hết các API trên hệ thống chạy đều dựa vào việc duyệt qua danh sách này.

- SessionProcessLinks: Danh sách liên kết kép khác kết nối các tiến trình trong cùng một phiên.

- InheritedFromUniqueProcessId: Một số nguyên chỉ định PID của tiến trình cha. Sau khi tiến trình bắt đầu chạy, thành viên này không được sửa đổi, ngay cả khi tiến trình cha kết thúc.

- Session: Thành viên này trỏ đến cấu trúc _MM_SESSION_SPACE (xem Chương 14) lưu trữ thông tin về phiên đăng nhập của người dùng và các đối tượng giao diện đồ họa (GUI).

- ImageFileName: Phần tên tệp thực thi của tiến trình. Trường này lưu trữ 16 ký tự ASCII đầu tiên, do đó tên tệp dài hơn sẽ xuất hiện bị cắt đứt. Để lấy đường dẫn đầy đủ đến tệp thực thi hoặc xem tên Unicode, bạn có thể truy cập vào nút VAD tương ứng hoặc các thành viên trong PEB (xem Chương 7).

- ThreadListHead: Danh sách liên kết kép kết nối tất cả các luồng của tiến trình (mỗi phần tử trong danh sách là một _ETHREAD).

- ActiveThreads: Số nguyên chỉ ra số lượng luồng đang hoạt động trong ngữ cảnh của tiến trình. Thấy một tiến trình có số lượng luồng hoạt động bằng 0 là dấu hiệu tốt cho thấy tiến trình đã kết thúc.

- Peb: Một con trỏ tới Process Environment Block (PEB). Mặc dù thành viên này (_EPROCESS.Peb) tồn tại ở chế độ kernel, nhưng nó trỏ tới một địa chỉ ở chế độ người dùng. PEB chứa các con trỏ tới các danh sách DLL của tiến trình, thư mục làm việc hiện tại, đối số dòng lệnh, biến môi trường, các heap và các xử lý tiêu chuẩn.

- VadRoot: Nút gốc của cây VAD. Nó chứa thông tin chi tiết về các đoạn bộ nhớ đã cấp phát của tiến trình, bao gồm quyền truy cập gốc (đọc, ghi, thực thi) và xem xét xem một tệp có được ánh xạ vào vùng này hay không.

### Process Organization

Cấu trúc _EPROCESS chứa một cấu trúc _LIST_ENTRY được gọi là ActiveProcessLinks. Cấu trúc _LIST_ENTRY chứa hai thành viên: Flink (liên kết tiến) trỏ đến _LIST_ENTRY của cấu trúc _EPROCESS tiếp theo, và Blink (liên kết lùi) trỏ đến _LIST_ENTRY của cấu trúc _EPROCESS trước đó. Cùng nhau, các thành phần này tạo thành một chuỗi các đối tượng tiến trình, còn được gọi là danh sách liên kết kép (xem Chương 2). Hình 6-2 thể hiện một biểu đồ minh họa cách các cấu trúc _LIST_ENTRY liên kết các tiến trình với nhau.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20201512.png)

Trên một hệ thống đang chạy, các công cụ như Process Explorer và Task Manager dựa vào việc duyệt qua danh sách liên kết kép của các cấu trúc _EPROCESS. Một API thông thường được sử dụng cho mục đích này là NtQuerySystemInformation, nhưng nhiều API cấp cao hơn do hệ điều hành cung cấp cũng truy cập vào cùng dữ liệu này.

### Enumerating Processes in Memory

Như đã được mô tả ngắn gọn trong phần "Vấn đề với việc chọn hồ sơ" của Chương 3, để liệt kê các tiến trình, Volatility trước tiên xác định khối dữ liệu gỡ lỗi kernel (_KDDEBUGGER_DATA64). Từ đó, nó truy cập thành viên PsActiveProcessHead, trỏ tới đầu của danh sách liên kết kép của các cấu trúc _EPROCESS. Chúng tôi cũng đã thảo luận về phương pháp quét pool trong Chương 5.

Trong chương này, chúng tôi trình bày nhiều cách khác để tìm kiếm các tiến trình trong một bản sao bộ nhớ. Việc triển khai các phương pháp thay thế quan trọng vì các khối dữ liệu gỡ lỗi, các con trỏ liên kết danh sách và các thẻ pool đều không cần thiết cho tính ổn định của hệ điều hành - điều này có nghĩa là chúng có thể bị điều khiển (vô tình hoặc cố ý) để đánh bại các công cụ pháp y tế mà không làm gián đoạn hệ thống hoặc các tiến trình của nó.

### Critical System Processes

Trước khi bạn bắt đầu phân tích trạng thái của một hệ thống dựa trên các ứng dụng đang chạy, bạn nên quen thuộc với các tiến trình hệ thống quan trọng; nếu bạn biết những gì là bình thường, bạn có thể phát hiện những gì là bất thường nhanh hơn. Trong phần còn lại của chương, chúng tôi sẽ thảo luận về các bước điều tra thực tế để xác minh các thông tin bạn thấy ở đây, vì vậy vào lúc này, chúng ta chỉ tập trung vào các khái niệm lý thuyết về cách thứ đã xuất hiện trên các hệ thống trong trạng thái sạch.

>**LƯU Ý:**<br> "Know your Windows Processes or Die Trying" của Patrick Olsen (http://sysforensics.org/2014/01/know-your-windows-processes.html) là một tài liệu tuyệt vời cung cấp các mô tả chi tiết về các tiến trình quan trọng, bao gồm các hiện vật cụ thể cần kiểm tra trong quá trình phân tích. Chúng tôi dựa trên một số thông tin từ bài viết của Patrick trong các sự thật dưới đây:

- Idle và System: Đây không phải là các tiến trình thực sự (theo nghĩa là chúng không có tệp thực thi tương ứng trên đĩa). Tiến trình Idle chỉ là một bộ chứa mà kernel sử dụng để tính thời gian CPU cho các luồng rảnh rỗi. Tương tự, tiến trình System đóng vai trò là nơi chứa mặc định cho các luồng chạy ở chế độ kernel. Do đó, tiến trình System (PID 4) dường như sở hữu bất kỳ socket hoặc handle tới các tập tin mà các module kernel mở.

- csrss.exe: Subsystem chạy dịch vụ khách/máy chủ đóng vai trò trong việc tạo và xóa các tiến trình và luồng. Nó duy trì một danh sách riêng tư của các đối tượng mà bạn có thể sử dụng để giao tham chiếu với các nguồn dữ liệu khác. Trên các hệ thống trước Windows 7, tiến trình này cũng hoạt động như là người trung gian của các lệnh được thực thi thông qua cmd.exe, vì vậy bạn có thể trích xuất lịch sử lệnh từ không gian bộ nhớ của nó. Dự kiến sẽ thấy nhiều tiến trình CSRSS vì mỗi phiên được gán một bản sao riêng; tuy nhiên, hãy cẩn trọng với các cố gắng lợi dụng quy tắc đặt tên (csrsss.exe hoặc cssrs.exe). Phiên bản thực sự nằm trong thư mục system32.

- services.exe: Trình quản lý điều khiển dịch vụ (SCM) được mô tả chi tiết hơn trong Chương 12, nhưng nó ngắn gọn là quản lý các dịch vụ Windows và duy trì một danh sách các dịch vụ đó trong không gian bộ nhớ riêng tư của nó. Tiến trình này nên là tiến trình cha của bất kỳ trường hợp svchost.exe (host dịch vụ) nào mà bạn thấy, bên cạnh các tiến trình như spoolsv.exe và SearchIndexer.exe thực hiện các dịch vụ. Nên chỉ có một bản sao của services.exe trên một hệ thống, và nó nên chạy từ thư mục system32.

- svchost.exe: Một hệ thống sạch sẽ có nhiều quá trình chủ chung chạy song song, mỗi quá trình cung cấp một bộ chứa cho các DLL thực hiện các dịch vụ. Như đã đề cập trước đó, tiến trình cha của chúng nên là services.exe và đường dẫn đến tệp thực thi nên trỏ tới thư mục system32. Trong blog của Patrick, anh ấy chỉ ra một số tên thông thường (như scvhost.exe và svch0st.exe) được sử dụng bởi mã độc để làm mờ đi trong các tiến trình này.

- lsass.exe: Tiến trình hệ thống bảo mật cục bộ là trách nhiệm thực thi chính sách bảo mật, xác minh mật khẩu và tạo token truy cập. Vì vậy, đây thường là mục tiêu của việc tiêm mã vì các băm mật khẩu văn bản thuần được tìm thấy trong không gian bộ nhớ riêng tư của nó. Nên chỉ có một phiên bản lsass.exe chạy từ thư mục system32, và tiến trình cha của nó là winlogon.exe trên máy trước Vista và wininit.exe trên các hệ thống Vista và sau này. Stuxnet đã tạo hai bản sao giả mạo của lsass.exe, khiến chúng trở nên rõ ràng như cành cây bị lẹo.

- winlogon.exe: Tiến trình này hiển thị hộp thoại đăng nhập tương tác, khởi động màn hình chờ khi cần thiết, giúp tải các hồ sơ người dùng và phản hồi cho các hoạt động bàn phím chuỗi An ninh an toàn (SAS) như CTRL+ALT+DEL. Ngoài ra, tiến trình này giám sát các tệp và thư mục thay đổi trên các hệ thống thực thi Bảo vệ Tập tin Windows (WFP). Giống như hầu hết các tiến trình quan trọng khác, tệp thực thi của nó nằm trong thư mục system32.

- explorer.exe: Bạn sẽ thấy một tiến trình Windows Explorer cho mỗi người dùng đăng nhập. Nó chịu trách nhiệm xử lý nhiều tương tác của người dùng như duyệt thư mục dựa trên giao diện đồ họa (GUI), hiển thị menu bắt đầu, v.v. Nó cũng có quyền truy cập vào tài liệu bạn mở và thông tin đăng nhập mà bạn sử dụng để đăng nhập vào các trang web FTP qua Windows Explorer.

- smss.exe: Quản lý phiên là quá trình chế độ người dùng đầu tiên bắt đầu trong quá trình khởi động. Nó chịu trách nhiệm tạo các phiên (xem Chương 14) cô lập các dịch vụ hệ điều hành khỏi các người dùng khác nhau có thể đăng nhập qua console hoặc Giao thức Remote Desktop (RDP).


Mặc dù danh sách này không đầy đủ, nhưng nó cung cấp đủ thông tin để giúp bạn bắt đầu. Bạn cũng nên làm quen với một số tiến trình không quan trọng nhưng phổ biến, chẳng hạn như IEXPLORE.EXE (và các trình duyệt khác), các ứng dụng email, các ứng dụng trò chuyện, các trình đọc tài liệu (Word, Excel, Adobe), các ứng dụng diệt virus, các công cụ mã hóa đĩa, tiện ích truy cập từ xa và chuyển tệp (SSH, Telnet, RDP, VNC), các công cụ phá mật khẩu và bộ công cụ khai thác lỗ hổng.


### Analyzing Process Activity

Volatility cung cấp một số lệnh bạn có thể sử dụng để trích xuất thông tin về các tiến trình:

- pslist: Tìm kiếm và duyệt qua danh sách hai chiều của các tiến trình và hiển thị một tóm tắt dữ liệu. Phương pháp này thường không thể hiển thị các tiến trình đã kết thúc hoặc được ẩn.

- pstree: Lấy đầu ra từ lệnh pslist và định dạng nó thành dạng cây để bạn có thể dễ dàng nhìn thấy mối quan hệ cha mẹ và con cái.

- psscan: Quét các đối tượng _EPROCESS thay vì dựa vào danh sách liên kết. Plugin này cũng có thể tìm thấy các tiến trình đã kết thúc và không liên kết (ẩn).

- psxview: Xác định các tiến trình bằng cách sử dụng các danh sách tiến trình thay thế, từ đó bạn có thể so sánh chéo các nguồn thông tin khác nhau và phát hiện ra những khác biệt đáng ngờ.

Một ví dụ về câu lệnh pslist:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20230017.png)

Cột đầu tiên trong đầu ra, Offset(V), hiển thị địa chỉ ảo (trong bộ nhớ kernel) của cấu trúc _EPROCESS. Di chuyển sang phải, bạn sẽ thấy tên tiến trình (hoặc ít nhất là 16 ký tự đầu tiên của nó), PID, PID của tiến trình cha, số luồng, số bản mô tả tập tin (handles), ID phiên và thời gian tạo. Bằng cách nhìn vào dữ liệu này, bạn có thể thu thập một số thông tin thú vị:

- Có ba trình duyệt đang chạy (hai phiên bản của IEXPLORE.EXE và một firefox.exe), một ứng dụng email (thunderbird.exe) và Adobe Reader (AcroRd32.exe). Do đó, có khả năng cao máy tính này là một máy trạm hoặc máy tính cá nhân, chứ không phải là máy chủ. Ngoài ra, nếu bạn nghi ngờ một vector tấn công từ phía người dùng (như một cuộc tải xuống không rõ nguồn gốc hoặc lợi dụng lừa đảo), thì nên khai thác các tiến trình này để thu thập dữ liệu liên quan đến sự cố vì có khả năng cao một hoặc nhiều tiến trình này liên quan đến sự việc.

- Tất cả các tiến trình, kể cả các tiến trình quan trọng cho hệ thống, đang chạy trong phiên 0, điều này cho thấy đây là một máy tính cũ (Windows XP hoặc 2003) (tức là trước cách ly phiên 0) và chỉ có một người dùng đang đăng nhập vào thời điểm này.

- Hai trong số các tiến trình AcroRd32.exe có 0 luồng và con trỏ bảng bản mô tả tập tin không hợp lệ (được chỉ ra bởi các đường nét đứt). Nếu cột thời gian thoát được hiển thị (chúng tôi cắt ngắn để tránh xuống dòng), bạn sẽ thấy rằng hai tiến trình này đã thực sự đã chấm dứt. Chúng "bị kẹt" trong danh sách tiến trình hoạt động vì một tiến trình khác đang có một bản mô tả tập tin mở đối với chúng (xem The Mis-leading Active in PsActiveProcessHead: http://mnin.blogspot.com/2011/03/mis-leading-active-in.html).

- Tiến trình có PID 2280 (a[1].php) có phần mở rộng không hợp lệ cho các tập tin thực thi - nó cho rằng là một tập tin PHP. Hơn nữa, dựa vào thời gian tạo của nó, nó có mối quan hệ thời gian với một số tiến trình khác bắt đầu trong cùng một phút (14:19:XX), bao gồm một cửa sổ lệnh (cmd.exe).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20231830.png)
![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20232003.png)

Khi xem các tiến trình dưới dạng cây, việc xác định các sự kiện có thể xảy ra trong quá trình tấn công dễ dàng hơn. Bạn có thể thấy rằng firefox.exe (PID 180) được khởi động bởi explorer.exe (PID 1300). Điều này là bình thường - mỗi khi bạn khởi động một ứng dụng thông qua menu Start hoặc bằng cách nhấp đôi vào biểu tượng trên màn hình, quá trình cha là Windows Explorer. Cũng khá phổ biến là trình duyệt tạo ra các phiên bản của Adobe Reader (AcroRd32.exe) để hiển thị tài liệu PDF được truy cập qua trang web. Tình huống trở nên thú vị khi bạn thấy rằng AcroRd32.exe đã kích hoạt một cửa sổ lệnh (cmd.exe), từ đó sau đó đã khởi động a[1].php. 

Lúc này, bạn có thể giả định rằng một trang web đã được truy cập bằng Firefox khiến trình duyệt mở một tập tin PDF độc hại. Một lỗ hổng bị khai thác trong AcroRd32.exe cho phép kẻ tấn công sử dụng cửa sổ lệnh để tiến xa hơn trong việc cài đặt mã độc bổ sung trên hệ thống.

### Process Tree Visualizations

Một cách khác để hiển thị mối quan hệ cha mẹ giữa các tiến trình là sử dụng lệnh psscan với trình biên dịch đồ chấm (dot graph renderer) (--output=dot). Chức năng này dựa trên công cụ PTFinder của Andreas Schuster (http://www.dfrws.org/2006/proceedings/2-Schuster-pres.pdf), cũng tạo ra biểu đồ cho phân tích hình ảnh. Vì góc nhìn này của các tiến trình dựa trên quét pool thông qua không gian địa chỉ vật lý, nó cũng tích hợp các tiến trình đã kết thúc và ẩn vào biểu đồ. Bạn có thể tạo ra một biểu đồ như sau:

```
 python vol.py psscan –f memory.bin --profile=Win7SP1x64
    --output=dot
    --output-file=processes.dot
```

Sau đó, mở tệp đầu ra trong Graphviz (http://www.graphviz.org), như được hiển thị trong Hình 6-3.
Dựa trên biểu đồ, bạn có thể xác minh một số khẳng định đã được thảo luận trước đó. Ví dụ, System bắt đầu smss.exe, sau đó smss.exe bắt đầu csrss.exe và winlogon.exe (trên các hệ thống Windows XP và 2003). Bạn sau đó có thể thấy winlogon.exe tạo ra services.exe và lsass.exe. Tiến trình SCM tiếp tục tạo ra spoolsv.exe và các phiên bản khác của svchost.exe. Nhưng điều này chỉ là một phần của hình ảnh. Phần còn lại của biểu đồ được hiển thị trong Hình 6-4.

Như bạn thấy ở đây, một tiến trình với PID 1188 đã bắt đầu explorer.exe, nhưng cấu trúc _EPROCESS của nó không còn cư trú trong bộ nhớ nữa, vì vậy thông tin bổ sung, chẳng hạn như tên tiến trình cha, không có sẵn. Điều này phổ biến trên các hệ thống XP và 2003 vì userinit.exe khởi động Explorer và nó sẽ thoát ngay sau đó. Khi bạn theo dõi các mũi tên còn lại, biểu đồ xác nhận các lý thuyết chúng ta tạo ra khi xem đầu ra pstree.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20232511.png)

### Detecting DKOM Attacks

Nhiều cuộc tấn công có thể xảy ra với việc Manipulation trực tiếp đối tượng Kernel (DKOM), nhưng một trong những cuộc tấn công phổ biến nhất là ẩn một tiến trình bằng cách tách mục nhập của nó ra khỏi danh sách liên kết kép. Để thực hiện điều này, kẻ tấn công ghi đè lên các con trỏ Flink và Blink của các đối tượng lân cận sao cho chúng chỉ vào xung quanh _EPROCESS của tiến trình cần ẩn. Các công cụ chạy trên hệ thống đang chạy và lệnh pslist của Volatility dễ bị tấn công này, vì chúng dựa vào danh sách liên kết. Tuy nhiên, plugin psscan sử dụng phương pháp quét pool mà chúng tôi đã miêu tả trong Chương 5. Nhờ vậy, bạn có thể tìm thấy các đối tượng _EPROCESS trong bộ nhớ, ngay cả khi chúng đã được tách khỏi danh sách.

Trước khi chúng ta bắt đầu với ví dụ, hãy xem xét một số cách mà phần mã độc có thể thay đổi trực tiếp các đối tượng kernel:
- Bằng cách tải một trình điều khiển kernel, từ đó có quyền truy cập không giới hạn vào các đối tượng trong bộ nhớ kernel.
- Bằng cách ánh xạ một chế độ xem có thể ghi của đối tượng \Device\PhysicalMemory (tuy nhiên, bắt đầu từ Windows 2003 SP1 và Vista, quyền truy cập vào đối tượng này bị hạn chế từ các chương trình usermode).
- Bằng cách sử dụng một chức năng API native đặc biệt gọi là ZwSystemDebugControl.

#### The Case of Prolaco


Để minh họa cách bạn có thể sử dụng psscan để tìm các tiến trình ẩn, chúng ta sẽ tập trung vào một mẫu mã độc được các nhà cung cấp phần mềm diệt virus gọi là Prolaco (xem https://www.avira.com/en/supportthreats-description/tid/5377/). Malware này thực hiện DKOM hoàn toàn từ chế độ người dùng, mà không tải bất kỳ trình điều khiển kernel nào. Nó làm như vậy bằng cách sử dụng ZwSystemDebugControl API gần như chính xác như cách được miêu tả bởi Alex Ionescu trên trang web OpenRCE (http://www.openrce.org/blog/view/354/). Hình 6-5 thể hiện sự phân tích ngược của Prolaco, được tạo bởi IDA Pro và Hex-Rays.

Dựa trên hình ảnh này, bạn có thể đưa ra các kết luận sau về cách malware thực hiện DKOM:

- Nó kích hoạt quyền debug (SeDebugPrivilege), cho phép tiến trình có quyền truy cập cần thiết để sử dụng ZwSystemDebugControl.
- Nó gọi NtQuerySystemInformation với lớp SystemModuleInformation để xác định địa chỉ cơ sở của module NT kernel.
- Nó tìm PsInitialSystemProcess, đó là một biến toàn cục được xuất khẩu bởi module NT, trỏ đến đối tượng _EPROCESS của tiến trình đầu tiên.
- Nó duyệt danh sách liên kết của các đối tượng _EPROCESS cho đến khi tìm thấy tiến trình có PID phù hợp với PidOfProcessToHide. Số cố định 0x88 được sử dụng trong vòng lặp while là vị trí của ActiveProcessLinks trong cấu trúc _EPROCESS cho các hệ thống 32-bit Windows XP. Lưu ý rằng PidOfProcessToHide được chuyển vào hàm như một tham số. Malware tạo ra giá trị này bằng cách sử dụng GetCurrentProcessId, điều này đồng nghĩa với việc nó cố gắng ẩn chính nó.
- Nó gọi hàm WriteKernelMemory, chỉ là một lớp bọc xung quanh ZwSystemDebugControl để ghi bốn byte một lần vào một địa chỉ cụ thể trong bộ nhớ kernel. Hàm được gọi một lần để ghi đè lên con trỏ Flink và một lần nữa để ghi đè lên con trỏ Blink. Hình 6-6 thể hiện nội dung của hàm này.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20232918.png)

Tại thời điểm này, tất cả các API hệ thống thời gian thực được đề cập trước đây (và các công cụ dựa vào chúng) sẽ báo cáo thông tin không chính xác về các tiến trình. Đặc biệt, chúng sẽ không xác định được một tiến trình quan trọng nhất liên quan đến cuộc điều tra của bạn: chính là phần mã độc vừa tách mục nhập của _EPROCESS của nó khỏi danh sách liên kết.

#### Alternate Process Listings

Như đã đề cập trước đó, không bao giờ nên đưa ra kết luận chỉ dựa trên một bằng chứng duy nhất. Vì danh sách tiến trình có thể bị điều chỉnh, bạn nên biết về các phương pháp sao lưu có sẵn hoặc các phương pháp liệt kê tiến trình thay thế. Dưới đây là một số trong số chúng:

- Quét các đối tượng tiến trình: Đây là phương pháp quét bộ nhớ pool được thảo luận trong Chương 5. Hãy nhớ rằng các thẻ pool mà nó tìm thấy là không cần thiết; do đó, chúng cũng có thể bị điều chỉnh để tránh bị quét.

- Quét các luồng (threads): Vì mỗi tiến trình phải có ít nhất một luồng hoạt động, bạn có thể quét các đối tượng _ETHREAD và sau đó ánh xạ chúng trở lại tiến trình chủ sở hữu. Các thành viên được sử dụng để ánh xạ là _ETHREAD.ThreadsProcess (Windows XP và 2003) hoặc _ETHREAD.Tcb.Process (Windows Vista và phiên bản sau). Do đó, ngay cả khi một rootkit thay đổi các thẻ pool cho tất cả các luồng của tiến trình.

- Bảng xử lý CSRSS: Như đã thảo luận trong các mô tả về các tiến trình hệ thống quan trọng, csrss.exe tham gia vào việc tạo ra mọi tiến trình và luồng (trừ chính nó và các tiến trình khởi đầu trước nó). Do đó, bạn có thể duyệt qua bảng xử lý của quy trình này, như được miêu tả sau trong chương, và xác định tất cả các đối tượng _EPROCESS theo cách đó.

- Bảng PspCid: Đây là một bảng xử lý đặc biệt nằm trong bộ nhớ kernel lưu trữ một tham chiếu tới tất cả các đối tượng tiến trình và luồng hoạt động. Thành viên PspCidTable trong cấu trúc dữ liệu của trình gỡ lỗi kernel trỏ tới bảng này. Hai công cụ phát hiện rootkit, Blacklight và IceSword, dựa vào bảng PspCid để tìm các tiến trình ẩn. Tuy nhiên, tác giả của FUTo (xem http://www.openrce.org/articles/full_view/19) đã chứng minh vẫn có thể ẩn bằng cách loại bỏ tiến trình khỏi bảng này.

- Các tiến trình phiên: Thành viên SessionProcessLinks của _EPROCESS liên kết tất cả các tiến trình thuộc về phiên đăng nhập của một người dùng cụ thể. Việc gỡ bỏ một tiến trình khỏi danh sách này không khó hơn so với danh sách ActiveProcessLinks. Tuy nhiên, vì các API hệ thống thời gian thực không phụ thuộc vào nó, kẻ tấn công hiếm khi tìm thấy giá trị trong việc nhắm vào nó.

- Các luồng Desktop: Một trong các cấu trúc được thảo luận trong Chương 14 là Desktop (tagDESKTOP). Những cấu trúc này lưu trữ danh sách tất cả các luồng được gắn vào mỗi desktop, và bạn có thể dễ dàng ánh xạ một luồng trở lại tiến trình sở hữu nó.

Vẫn còn rất nhiều phương pháp khác để liệt kê tiến trình. Tuy nhiên, chúng tôi chưa từng gặp một rootkit nào thậm chí còn khó trốn khỏi plugin pslist, và cũng trốn khỏi tất cả các phương pháp này. Đó là lý do tại sao chúng tôi xây dựng plugin psxview, được miêu tả tiếp theo.

#### Process Cross-View Plugin

Plugin psxview thực hiện liệt kê tiến trình theo bảy cách khác nhau: danh sách liên kết tiến trình hoạt động và sáu phương pháp đã được xác định trước đó. Do đó, rất khó mà một rootkit có thể trốn khỏi psxview. Thực tế, việc tiêm mã vào một tiến trình không bị ẩn là dễ dàng hơn là ẩn một tiến trình bằng bảy cách khác nhau một cách đáng tin cậy (nghĩa là không có lỗi) và mang tính di động (hoạt động trên tất cả các phiên bản Windows).

Plugin psxview hiển thị một hàng cho mỗi tiến trình được tìm thấy và bảy cột chứa True hoặc False, dựa trên việc tiến trình được tìm thấy bằng phương pháp tương ứng hay không. Nếu bạn cung cấp tùy chọn --apply-rules, bạn cũng có thể thấy Okay trong các cột, cho biết mặc dù tiến trình không được tìm thấy, nó đáp ứng một trong các ngoại lệ hợp lệ được miêu tả trong danh sách sau:
- Các tiến trình khởi đầu trước csrss.exe (bao gồm System, smss.exe và chính csrss.exe) không có trong bảng xử lý CSRSS.
- Các tiến trình khởi đầu trước smss.exe (bao gồm System và smss.exe) không có trong danh sách tiến trình phiên và luồng desktop.
- Các tiến trình đã kết thúc sẽ không được tìm thấy bởi bất kỳ phương pháp nào ngoại trừ quét các đối tượng tiến trình và quét các luồng (nếu một _EPROCESS hoặc _ETHREAD vẫn tồn tại trong bộ nhớ).

Một ví dụ về đầu ra của psxview khi sử dụng với tùy chọn --apply-rules được thể hiện trong Hình 6-7. Hai tiến trình cuối cùng (msiexec.exe và rundll32.exe) chỉ được tìm thấy bởi trình quét đối tượng tiến trình. Tuy nhiên, như được hiển thị ở cột bên phải cùng, cả hai tiến trình này đều có thời gian thoát khác không, có nghĩa là chúng đã kết thúc.

>**Cảnh báo:**<br> Sau khi tấn công viên có quyền truy cập vào bộ nhớ kernel, họ có thể thao túng bất cứ thứ gì mà họ muốn. Trong trường hợp này, họ có thể ghi đè lên thành viên _EPROCESS.ExitTime để làm cho tiến trình có vẻ đã thoát; do đó, tùy chọn --apply-rules sẽ báo cáo không đúng. Tuy nhiên, các tiến trình thực sự đã thoát sẽ có số luồng là 0 và bảng xử lý không hợp lệ - vì vậy bạn luôn có thể kiểm tra lại những giá trị trong các trường này.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-29%20234652.png)

Như được thể hiện trong hình, chỉ có tiến trình 1_doc_RCData_61.exe nổi bật sau khi xem xét các quy tắc. Nó được tìm thấy bởi mọi phương pháp ngoại trừ danh sách liên kết của các tiến trình hoạt động. Điều này là dấu hiệu rõ ràng rằng nó đang cố gắng ẩn mình khỏi các API hệ thống trực tiếp bằng cách unlink từ danh sách.

## Process Tokens



