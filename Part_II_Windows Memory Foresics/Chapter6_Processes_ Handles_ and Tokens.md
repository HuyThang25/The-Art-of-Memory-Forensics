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

Một token của tiến trình mô tả bối cảnh bảo mật của nó. Bối cảnh này bao gồm các định danh bảo mật (SIDs) của người dùng hoặc nhóm mà tiến trình đang chạy và các đặc quyền khác nhau (nhiệm vụ cụ thể) mà nó được phép thực hiện. Khi kernel cần quyết định liệu một tiến trình có thể truy cập một đối tượng hoặc gọi một API cụ thể không, nó sẽ tham khảo dữ liệu trong token của tiến trình. Kết quả là, cấu trúc này quy định nhiều điều khiển liên quan đến bảo mật liên quan đến các tiến trình. Phần này mô tả cách tận dụng token để bổ sung cho quá trình điều tra của bạn.

**Phân tích mục tiêu**

Các mục tiêu của bạn là như sau:

- Ánh xạ SIDs thành tên người dùng: Token của một tiến trình chứa giá trị SID dưới dạng số, bạn có thể dịch chúng thành chuỗi và sau đó tìm tên người dùng hoặc nhóm tương ứng. Điều này cuối cùng cho phép bạn xác định tài khoản người dùng chính mà một tiến trình đang chạy dưới đó.

- Phát hiện chuyển động ngang: Khi các kỹ thuật hack như pass-the-hash thành công, chúng để lại dấu vết rõ ràng trong token của một tiến trình. Cụ thể, bạn thấy bối cảnh bảo mật của tiến trình chuyển sang tài khoản Domain Admin hoặc Enterprise Admin.

- Xem xét hành vi tiến trình: Một đặc quyền (được thảo luận sau trong chương) là quyền thực hiện một nhiệm vụ cụ thể. Nếu một tiến trình có kế hoạch thực hiện một nhiệm vụ, trước tiên nó phải đảm bảo rằng đặc quyền đó tồn tại và đã được kích hoạt trong token của nó. Do đó, việc phân tích hậu quả sau này về các đặc quyền mà một tiến trình đã thu thập có thể cung cấp gợi ý về những gì tiến trình đã làm (hoặc đã dự định làm).

- Phát hiện leo thang đặc quyền: Một số cuộc tấn công đã chứng minh rằng các công cụ trực tiếp như Process Explorer có thể bị đánh lừa thành báo cáo rằng một tiến trình có ít đặc quyền hơn thực tế. Bạn có thể sử dụng phân tích nhớ trí để xác định chính xác hơn sự thật.

**Cấu trúc dữ liệu**


Cấu trúc _TOKEN có kích thước lớn, vì vậy chúng ta sẽ không hiển thị tất cả các thành viên. Hơn nữa, cấu trúc này đã thay đổi đáng kể về cách lưu trữ thông tin đặc quyền giữa Windows 2003 và Vista. Dưới đây là ví dụ về các phiên bản cấu trúc trước đây, từ hệ thống 64-bit Windows 2003:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-30%20165652.png)

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-30%20165735.png)


Dưới đây là các cấu trúc tương đương cho hệ thống 64-bit Windows 7:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-30%20165834.png)

**Các điểm chính**

- UserAndGroupCount: Đây là một số nguyên lưu trữ kích thước của mảng UserAndGroups.
- UserAndGroups: Một mảng các cấu trúc _SID_AND_ATTRIBUTES liên quan đến thông tin token. Mỗi phần tử trong mảng mô tả một người dùng hoặc nhóm khác nhau mà tiến trình là thành viên. Thành viên Sid của _SID_AND_ATTRIBUTES trỏ tới cấu trúc _SID, có các thành viên IdentifierAuthority và SubAuthority mà bạn có thể kết hợp để tạo thành chuỗi SID S-1-5-[snip].
- PrivilegeCount (Chỉ có trên Windows XP và 2003): Đây là một số nguyên lưu trữ kích thước của mảng Privileges.
- Privileges (Chỉ có trên Windows XP và 2003): Một mảng các cấu trúc _LUID_AND_ATTRIBUTES mô tả mỗi đặc quyền và thuộc tính của nó (tức là có mặt, được bật, bật theo mặc định).
- Privileges (Trên Windows Vista và các phiên bản sau): Đây là một phiên bản của cấu trúc _SEP_TOKEN_PRIVILEGES, có ba giá trị song song 64-bit (Present, Enabled, EnabledByDefault). Các vị trí bit tương ứng với các đặc quyền cụ thể, và các giá trị của bit (bật hoặc tắt) mô tả trạng thái của đặc quyền đó.

### Live Response: Accessing Tokens

Trên một máy tính hoạt động, một tiến trình có thể truy cập vào token của chính nó thông qua API OpenProcessToken. Sau đó, để liệt kê các SID hoặc đặc quyền, nó có thể sử dụng GetTokenInformation với các tham số mong muốn. Với quyền quản trị, nó cũng có thể truy vấn (hoặc đặt) token của các tiến trình của người dùng khác, bao gồm cả các tiến trình quan trọng của hệ thống. Tất nhiên, các công cụ hiện có đã cung cấp loại chức năng này cho bạn, chẳng hạn như Sysinternals Process Explorer. Hình 6-8 cho thấy thông tin token cho explorer.exe. Dữ liệu SID xuất hiện ở phía trên, và dữ liệu đặc quyền xuất hiện ở mức dưới.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-30%20170227.png)

Thông qua thông tin token của tiến trình explorer.exe này, chúng ta có thể nhận ra rằng nó thuộc về người dùng có tên là Jimmy, và SID tương ứng của Jimmy là S-1-5-21-[snip]-1000. Bằng cách phân tích các SID khác trong token của tiến trình này, chúng ta cũng nhận thấy nó thuộc vào các nhóm Everyone, LOCAL và NT AUTHORITY\Authenticated Users. Token của tiến trình này cũng chứa năm đặc quyền, nhưng chỉ có một trong số chúng được kích hoạt. Các trang tiếp theo sẽ đi sâu vào các khái niệm này để cung cấp thông tin chi tiết hơn.

### Extracting and Translating SIDs in Memory

Các API trực tiếp trên hệ thống Windows mà chúng ta đã đề cập trước đó là cách tiện lợi để liệt kê các SID trên các hệ thống đang chạy. Windows cũng cung cấp API ConvertSidToStringSid để chuyển đổi dữ liệu số trong cấu trúc _SID thành định dạng chuỗi S-1-5-[snip] có thể đọc được. Ngoài ra, Windows cũng cung cấp API LookupAccountSid để trả về tên tài khoản cho một SID cụ thể. Tuy nhiên, vì chúng ta đang làm việc với các bản ghi bộ nhớ, Volatility (đặc biệt là plugin getsids) sẽ làm nhiệm vụ tìm token của từng tiến trình, trích xuất các thành phần số trong cấu trúc _SID và chuyển đổi chúng thành chuỗi. Sau khi hoàn thành, plugin sẽ ánh xạ các chuỗi này thành tên người dùng và nhóm trên máy tính cục bộ hoặc miền.

Có một số cách khác nhau để thực hiện việc ánh xạ này. Đầu tiên, các SIDs đã biết (xem http://support.microsoft.com/kb/243330) được cài đặt sẵn trong Windows và do đó có thể cài đặt sẵn trong plugin của Volatility. Chúng bao gồm các SID như S-1-5 (NT Authority) và S-1-5-32-544 (Administrators). Cũng có các SID dành cho Dịch vụ được tiền tố bằng S-1-5-80. Phần còn lại của SID trong trường hợp này bao gồm băm SHA1 của tên dịch vụ tương ứng (trong Unicode, chữ hoa) - một thuật toán được mô tả chi tiết hơn tại đây: http://volatility-labs.blogspot.com/2012/09/movp-23-event-logs-and-service-sids.html.

Cuối cùng, có các SID của Người dùng như S-1-5-21-4010035002-774237572-2085959976-1000. Các SID này bao gồm các thành phần sau:
- S: Tiền tố chỉ ra rằng chuỗi là một SID.
- 1: Cấp độ sửa đổi (phiên bản của quy định SID) từ _SID.Revision.
- 5: Giá trị thông tin nhận diện từ _SID.IdentifierAuthority.Value.
- 21-4010035002-774237572-2085959976: Nhận diện máy tính cục bộ hoặc miền từ các giá trị _SID.SubAuthority.
- 1000: Một nhận diện tương đối đại diện cho bất kỳ người dùng hoặc nhóm nào không tồn tại theo mặc định.

Bạn có thể ánh xạ chuỗi SID thành tên người dùng bằng cách truy vấn registry. Dưới đây là một ví dụ về cách thực hiện điều này:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-30%20170747.png)

Bằng cách nối chuỗi SID vào khóa registry ProfileList, bạn có thể xem giá trị có tên ProfileImagePath. Tên người dùng sau đó được xác định trong đường dẫn hồ sơ. Trong trường hợp này, tên người dùng là "nkESis3ns88S" - đó là một tài khoản sau cửa sau được tạo ra ngẫu nhiên bởi kẻ tấn công để giữ quyền truy cập vào hệ thống. Ánh xạ các SID sang tên người dùng có thể giúp xác định người dùng hoặc nhóm liên quan đến các tiến trình cụ thể và hiểu về ngữ cảnh bảo mật của một tiến trình.

>GHI CHÚ<br>
Để biết thêm thông tin về việc dịch các giá trị SID, hãy tham khảo các liên kết sau đây:
>-	Kết nối tiến trình với người dùng: http://moyix.blogspot.com/2008/08/linking-processes-to-users.html.
> -	Cách liên kết tên người dùng với Security Identifier (SID): http://support.microsoft.com/kb/154599/en-us
> -	Cấu trúc Security Identifier: http://technet.microsoft.com/en-us/library/cc962011.aspx.

### Detecting Lateral Movement

Nếu bạn cần liên kết một tiến trình với một tài khoản người dùng hoặc điều tra các nỗ lực chuyển động bên, hãy sử dụng tiện ích getsids. Đây là đầu ra từ thử thách pháp lý GrrCON của Jack Crook (xem http://michsec.org/2012/09/misec-meetup-october-2012/).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-30%20171514.png)

Lệnh này hiển thị các SID liên kết với tiến trình explorer.exe cho người dùng đã đăng nhập hiện tại. Bạn sẽ ngay lập tức nhận thấy rằng một SID (S-1-5-21-[snip]-1115) không hiển thị tên tài khoản. Trên các hệ thống không xác thực đến một miền, bạn sẽ thấy tên người dùng cục bộ bên cạnh SID. Tuy nhiên, trong trường hợp này, do Volatility không có quyền truy cập vào registry máy từ xa (tức là máy điều khiển miền hoặc máy chủ Active Directory), nó không thể thực hiện quá trình giải quyết.

Câu hỏi mà bạn có thể trả lời với getsids là: Kẻ tấn công đã đạt được mức truy cập nào? Đừng dừng lại sau khi thấy rằng explorer.exe là thành viên của nhóm Quản trị viên. Trên máy này, kẻ tấn công thực tế đã tham gia vào các nhóm Quản trị viên miền và Doanh nghiệp, cho phép anh ta di chuyển dọc theo toàn bộ mạng doanh nghiệp. Trong kịch bản cụ thể này, kẻ tấn công kết hợp Remote Access Trojan (RAT) Poison Ivy (PI) với việc sử dụng tấn công Pass the Hash (PtH). Bạn có thể đọc một phân tích đầy đủ về cuộc tấn công này tại đây: http://volatility-labs.blogspot.com/2012/10/solving-grrcon-networkforensics.html.

## Privileges

Quyền đặc biệt là một thành phần quan trọng khác liên quan đến bảo mật và kiểm soát truy cập. Một quyền đặc biệt là sự cho phép thực hiện một nhiệm vụ cụ thể, chẳng hạn như gỡ lỗi một tiến trình, tắt máy tính, thay đổi múi giờ hoặc tải một trình điều khiển kernel. Trước khi một tiến trình có thể bật một quyền, quyền phải có trong token của tiến trình đó. Người quản trị quyết định quyền nào có mặt bằng cách cấu hình chúng trong Chính sách Bảo mật Cục bộ (LSP), như được hiển thị trong Hình 6-9, hoặc theo cách lập trình bằng cách gọi LsaAddAccountRights. Bạn có thể truy cập LSP bằng cách vào Start ➪ Run và gõ SecPol.msc.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-30%20171901.png)

### Commonly Exploited Privileges

Sau khi một quyền xuất hiện trong token của một tiến trình, nó phải được bật (enabled). Dưới đây là một số cách để kích hoạt các quyền:

- Kích hoạt theo mặc định: LSP có thể chỉ định rằng các quyền được kích hoạt theo mặc định khi một tiến trình bắt đầu.
- Kế thừa: Trừ khi được chỉ định khác, các tiến trình con sẽ kế thừa ngữ cảnh bảo mật từ người tạo ra chúng (tiến trình cha).
- Kích hoạt rõ ràng: Một tiến trình có thể kích hoạt một quyền rõ ràng bằng cách sử dụng API AdjustTokenPrivileges.

Từ góc nhìn pháp y, bạn nên quan tâm nhất đến các quyền sau khi chúng được kích hoạt rõ ràng. Để có danh sách đầy đủ các quyền có thể có và mô tả của chúng, hãy xem tại http://msdn.microsoft.com/en-us/library/windows/desktop/bb530716(v=vs.85).aspx.

- SeBackupPrivilege: Quyền này cấp quyền truy cập đọc vào bất kỳ tệp nào trên hệ thống tệp, bất kể ACL (Access Control List) được chỉ định của nó. Kẻ tấn công có thể tận dụng quyền này để sao chép các tệp bị khóa.

- SeDebugPrivilege: Quyền này cấp quyền cho việc đọc từ hoặc ghi vào không gian bộ nhớ riêng của tiến trình khác. Nó cho phép phần mềm độc hại bypass các ranh giới bảo mật thường tách biệt các tiến trình. Thực tế, hầu hết các phần mềm độc hại thực hiện việc tiêm mã từ chế độ người dùng đều dựa vào việc kích hoạt quyền này.

- SeLoadDriverPrivilege: Quyền này cấp quyền cho việc tải hoặc hủy tải các trình điều khiển hạt nhân.

- SeChangeNotifyPrivilege: Quyền này cho phép người gọi đăng ký một hàm gọi lại được thực thi khi các tệp và thư mục cụ thể thay đổi. Kẻ tấn công có thể sử dụng nó để xác định ngay lập tức khi một trong các tệp cấu hình hoặc tệp thực thi của họ bị loại bỏ bởi phần mềm diệt virus hoặc quản trị viên.

- SeShutdownPrivilege: Quyền này cho phép người gọi khởi động lại hoặc tắt hệ thống. Một số nhiễm trùng, chẳng hạn như những thay đổi Master Boot Record (MBR), sẽ không được kích hoạt cho đến lần khởi động hệ thống kế tiếp. Do đó, bạn thường thấy phần mềm độc hại cố gắng tăng tốc thủ tục bằng cách khởi động lại máy tính.

>**Lưu ý:**<br> Cem Gurkok đã hỗ trợ thiết kế tính năng của Volatility để phân tích đặc quyền trong bộ nhớ. Bạn có thể đọc bài thuyết trình của anh ấy, Reverse Engineering with Volatility on a Live System: The Analysis of Process Token Privileges, tại đây: http://volatility-labs.blogspot.com/2012/10/omfw-2012-analysis-of-process-token.html.

### Analyzing Explicit Privileges

Lý do tại sao chúng ta thường quan tâm đến đặc quyền được bật một cách rõ ràng là bởi vì nó thể hiện ý thức hoặc ý định. Nếu một tiến trình có thể thay đổi múi giờ vì LSP (Chính sách Bảo mật Cục bộ) cấp cho tất cả tiến trình khả năng đó hoặc vì tiến trình cha của nó có khả năng đó, điều đó không cho bạn biết gì về chức năng dự định của tiến trình đó. Tuy nhiên, nếu một tiến trình cụ thể đã bật đặc quyền để thay đổi múi giờ, bạn có thể chắc chắn rằng nó sẽ cố gắng thay đổi múi giờ.

Dưới đây là đầu ra của tiện ích Volatility privs. Bạn sẽ thấy tên đặc quyền cùng với các thuộc tính của nó (hiện có, đã bật, và/hoặc được bật mặc định).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-30%20172751.png)

Bạn có thể thấy một số đặc quyền xuất hiện trong đầu ra. Chỉ có sáu đặc quyền đã được bật, nhưng ba trong số đó được bật mặc định. Do đó, bạn có thể kết luận rằng explorer.exe đã bật đặc quyền undock, debug và load driver một cách rõ ràng. Theo thời gian, bạn sẽ nhận ra rằng explorer.exe luôn bật đặc quyền undock, vì vậy đặc quyền này không đáng lo ngại. Nhưng tại sao Windows Explorer cần phải gỡ lỗi các tiến trình khác và tải các trình điều khiển kernel? Câu trả lời đơn giản là: Nó không cần! Tiến trình này đang chứa một mẫu injected Poison Ivy (PI)  và PI đã bật các đặc quyền này một cách rõ ràng.

### Detecting Token Manipulation

Như đã đề cập trước đó, API của Windows (AdjustTokenPrivileges) không cho phép và không nên cho phép bật một đặc quyền mà không có trong mã thông báo. Do đó, việc các API như GetTokenInformation (và các công cụ dựa trên API này) sẽ trước tiên kiểm tra xem cái gì đã có trong mã thông báo và sau đó trả về tập con đã được bật là hoàn toàn hợp lý. Tuy nhiên, có một điểm cần lưu ý: Một nhà nghiên cứu tài năng tên là Cesar Cerrudo đã phát hiện rằng khi kiểm tra xem một quy trình có thể thực hiện một tác vụ hay không, kernel chỉ quan tâm đến những đặc quyền đã được bật. Kết quả là, Cesar đã đề xuất trong bài báo của mình có tiêu đề "Easy Local Windows Kernel Exploitation" (https://www.blackhat.com/html/bh-us-12/bh-us-12-archives.html#Cerrudo) một phương pháp để bypass các API của Windows và bật tất cả các đặc quyền cho một quy trình, ngay cả khi chúng không có sẵn trong mã thông báo.

#### Attack Simulation with Volshell




