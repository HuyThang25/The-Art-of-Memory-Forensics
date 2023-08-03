Dịch vụ trên Windows thường không tương tác (không nhận trực tiếp đầu vào từ người dùng), chạy liên tục ở nền và thường chạy với đặc quyền cao hơn so với hầu hết các chương trình mà người dùng khởi chạy. Các ví dụ về dịch vụ bao gồm cơ sở dữ liệu sự kiện, trình in ảnh, tường lửa máy chủ và bộ đồng hồ thời gian. Nhiều sản phẩm chống vi-rút, bao gồm cả Windows Defender và Security Center của Microsoft, cũng chạy như các dịch vụ. Ngoài ra, mã độc và đối thủ thường tận dụng dịch vụ để tồn tại sau khi khởi động lại hệ thống, tải trình điều khiển kernel và trộn lẫn với các thành phần hợp pháp trên hệ thống.

Chương này giới thiệu về cấu trúc bên trong của các dịch vụ trên Windows và cho thấy cách hiểu biết này có thể giúp bạn điều tra các hệ thống bị xâm nhập. Nó giải thích những lợi ích chính của việc trích xuất thông tin liên quan đến dịch vụ từ bộ nhớ bay thay vì chỉ dựa vào dữ liệu từ registry. Để minh họa các khái niệm, bạn sẽ xem xét một số kịch bản liên quan đến phần mềm độc hại như Conficker, TDL3, Blazgel và các công cụ mà đối thủ sử dụng như Nhóm Comment (còn được gọi là APT1).

## Service Architecture

Sơ đồ trong Hình 12-1 cho thấy cách các thành phần chính của kiến trúc dịch vụ trên Windows hoạt động cùng nhau. Một danh sách các dịch vụ đã cài đặt và cấu hình của chúng được lưu trữ trong registry dưới khóa HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services. Mỗi dịch vụ có một khóa con riêng biệt với các giá trị khác nhau mô tả cách và khi nào dịch vụ được khởi động; liệu dịch vụ đó là cho một tiến trình, thư viện liên kết động (DLL) hay trình điều khiển kernel; và bất kỳ thiết lập cụ thể nào của dịch vụ. Trong trường hợp các dịch vụ được thực hiện dưới dạng DLL, chúng sẽ chạy từ một quá trình chung (svchost.exe).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230610.png)

Cơ chế tiến trình chung là tuyệt vời về hiệu suất vì nhiều hệ thống chạy hàng trăm dịch vụ dựa trên DLL cùng lúc và không cần mỗi dịch vụ có một tiến trình riêng. Tuy nhiên, nó có thể gây hại cho cả bảo mật và công tác pháp y. Chỉ những dịch vụ chạy với cùng mức đặc quyền mới nên chia sẻ một tiến trình chung; nếu không, mã của một dịch vụ có thể truy cập vào các handle, bộ nhớ và tài nguyên khác được tạo bởi một dịch vụ khác. Tương tự, trong quá trình điều tra, nếu bạn theo dõi hoạt động như kết nối mạng trở lại một tiến trình, và tiến trình đó chỉ là một thùng chứa cho nhiều DLL dịch vụ, bạn vẫn cần xác định dịch vụ nào đang khởi tạo các kết nối (xem phần "Xác định kết nối từ mã" trong Chương 11).

Như bạn có thể thấy, khi hệ thống khởi động, Bộ điều khiển Dịch vụ (SCM), còn được gọi là services.exe, đọc từ registry và khởi chạy bất kỳ dịch vụ nào được cấu hình tự động khởi động. Một số dịch vụ chỉ định các phụ thuộc - ví dụ: TermService (RDP) phụ thuộc vào hỗ trợ mạng, vì vậy dịch vụ Tcpip phải khởi động trước tiên. Khi SCM phân tích các mục trong registry này, nó tạo một danh sách liên kết các cấu trúc bản ghi dịch vụ trong bộ nhớ lưu trữ trạng thái hiện tại của dịch vụ (dừng, chạy, tạm dừng, vv.) và các ID tiến trình (PIDs) liên quan. Dữ liệu tạm thời này không bao giờ được ghi vào registry, khiến RAM là nguồn duy nhất.

## Installing Services

Trước khi bạn tìm hiểu về các tàn dư vật liệu ex post facto, điều quan trọng là hiểu cách dịch vụ được tạo ra ban đầu. Điều này cũng sẽ giúp bạn nhận ra những khả năng như vậy khi bạn thực hiện phân tích tĩnh của các mẫu mã độc. Các cách phổ biến nhất mà kẻ tấn công tạo dịch vụ là như sau:
- Các lệnh thủ công: Nếu kẻ thù đã có quyền truy cập vào dòng lệnh, họ có thể gõ lệnh sc create và sc start để tạo và bắt đầu các dịch vụ, tương ứng. Trong trường hợp này, ngoài các tàn dư vật liệu dịch vụ thực tế, bạn có thể khôi phục lại chính xác các lệnh bằng cách chạy các plugins consoles hoặc cmdscan trong Volatility (xem Chương 17).
- Các tập lệnh Batch: Nhiều mẫu mã độc thả và thực thi các tập lệnh Batch sử dụng các lệnh đã đề cập trên - chỉ là cách tự động hóa. Kỹ thuật này cũng dẫn đến các tàn dư vật liệu mới trên đĩa. Dưới đây là một ví dụ về một tập lệnh Batch tạo dịch vụ có tên MyService và cấu hình nó tự động tải MyService.dll khi khởi động hệ thống bên trong một tiến trình chung:

	![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230640.png)
- Các APIs của Windows: Đằng sau các lệnh sc, các hàm như CreateService và StartService được gọi trực tiếp - cả hai hàm này có thể được nhập trực tiếp bởi mã độc được viết bằng ngôn ngữ C hoặc C++. Trước khi CreateService trả về, một subkey mới được thêm vào registry với cấu hình của dịch vụ. StartService để lại các dấu vết trong nhật ký sự kiện để chỉ ra thời gian dịch vụ được khởi chạy.
- WMI: Có một số giao diện cấp cao cho phép bạn tương tác với dịch vụ. Ví dụ, Windows Management Instrumentation (WMI) tiết lộ một lớp Win32_Service với các phương thức Create và Start (xem http://msdn.microsoft.com/en-us/library/aa394418(v=vs.85).aspx).

## Tricks and Stealth

Vì việc cài đặt dịch vụ gây ra nhiều đầu mối (nghĩa là nó tạo các khóa registry mới và mục nhập nhật ký sự kiện), các kẻ tấn công đã tạo ra một số cách để tránh bị phát hiện bởi các kỹ thuật phân tích phổ biến. Ví dụ, một trong những heuristics mà một số sản phẩm antivirus sử dụng là theo dõi các tiến trình gọi CreateService và StartService theo thứ tự. Tất nhiên, những hành động này không gây hại khi được thực hiện một mình, nhưng ít nhất nó cung cấp một cơ hội cho phần mềm antivirus quét các tệp nhị phân của dịch vụ trước khi cho phép dịch vụ chạy. Những gì nhiều nhà phân tích không biết là không cần phải sử dụng các APIs đã đề cập để cài đặt dịch vụ.

Nếu bạn tạo thủ công các khóa registry cần thiết (thông qua RegCreateKey, RegSetValue, và các hàm tương tự), không cần sử dụng CreateService. Sau đó, bạn có thể khởi chạy dịch vụ bằng cách gọi trực tiếp NdrClientCall2 (một giao diện RPC được gọi trong StartService). Hoặc, nếu dịch vụ dùng cho một kernel driver, sau khi đã tạo các khóa registry, bạn có thể gọi NtLoadDriver. Cả hai phương pháp này đều tạo ra một dịch vụ mới đang chạy, nhưng không tạo các mục nhập trong nhật ký sự kiện để chỉ ra rằng đã tạo hoặc khởi chạy dịch vụ. Bằng cách tránh sử dụng các APIs tiêu chuẩn, SCM được né tránh và do đó nó không tạo ra các cấu trúc hồ sơ dịch vụ trong bộ nhớ.

Mặc dù các kỹ thuật này ngăn không tạo mục nhập trong nhật ký sự kiện và một số tác nhân nhớ khác, nhưng vẫn cần thêm vào registry. Tuy nhiên, sau khi một dịch vụ đang chạy, phần mềm độc hại có thể đơn giản là xóa các khóa và giá trị trong registry. Hình 12-2 cho thấy TDL3 làm chính xác điều đó - nó bỏ qua StartService bằng cách gọi NtLoadDriver và sau đó sử dụng SHDeleteKeyA để xóa các dấu vết trong registry.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230657.png)

Tại thời điểm này, không còn dấu vết của dịch vụ tồn tại trong nhật ký sự kiện, registry hoặc bộ nhớ của services.exe. Bạn vẫn có thể tìm thấy quy trình đang chạy, DLL hoặc kernel driver trong bộ nhớ (trừ khi một rootkit cũng ẩn chúng), nhưng các siêu dữ liệu quan trọng mô tả cách và khi nào chúng được tải ban đầu sẽ không dễ dàng truy cập. Đó chỉ là một số trong những cách tình báo mật và malware có thể tương tác với dịch vụ. Trong phần còn lại của chương này, bạn sẽ gặp một số cách khác, từ đó có cái nhìn toàn diện về các hoạt động liên quan đến dịch vụ.

Ngoài ra, kẻ thù không chỉ tạo mới dịch vụ để sử dụng mà còn dừng các dịch vụ hiện có để tắt hoặc giảm bớt bảo mật. Ví dụ, một số biến thể của Conficker dừng các dịch vụ sau đây để có thể hoạt động tự do hơn trên máy tính của nạn nhân:

- Wscsvc (Dịch vụ Trung tâm Bảo mật Windows)
- Wuauserv (Dịch vụ Cập nhật Tự động Windows)
- BITS (Dịch vụ Truyền thông Thông minh Nền)
- WinDefend (Dịch vụ Windows Defender)
- WerSvc (Dịch vụ Báo cáo Lỗi Windows)

Có nhiều cách để dừng một dịch vụ. Hai phương pháp như vậy bao gồm việc sử dụng hàm API ControlService và thả một tệp batch chứa các lệnh như net stop SERVICENAME. Malware cũng có thể sử dụng TerminateProcess, nhưng điều này không cho phép quy trình dịch vụ tắt một cách sạch sẽ hoặc thông báo cho SCM về trạng thái mới của dịch vụ, điều này thường dẫn đến vấn đề về tính ổn định. Do đó, khi bạn xem xét các dịch vụ trên một hệ thống đáng nghi, hãy nhớ rằng các dịch vụ bị dừng thường cũng có thể là các chỉ báo cũng như dịch vụ được bắt đầu / đang chạy.

## Investigating Service Activity

Windows đi kèm với một số công cụ mà bạn có thể sử dụng để điều tra các dịch vụ trên máy tính hoạt động. Tuy nhiên, hãy nhớ rằng chúng đều có thể bị giám sát hoặc bị thao túng bởi rootkit đang hoạt động trên hệ thống. Tuy nhiên, tốt nhất là biết các tùy chọn của mình trong trường hợp bạn muốn so sánh những gì bạn thấy trên hệ thống hoạt động với dữ liệu trong bộ nhớ RAM. Ví dụ, bạn có thể sử dụng Microsoft Management Console (MMC) để điều tra hoặc điều khiển các dịch vụ trên hệ thống. Để mở nó lên, đi tới Start ➪ Run và sau đó gõ services.msc và nhấn Enter.

Từ dòng lệnh, hoặc nếu bạn cần liệt kê các dịch vụ tự động một cách tự động với một kịch bản thu thập, bạn có một số tùy chọn sẵn có. Một trong số đó là lệnh Get-Service trong PowerShell. Bạn có thể sử dụng tính năng này như sau:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230711.png)

Một tùy chọn khác mà bạn có thể truy cập từ một command shell tiêu chuẩn (không phải PowerShell) là lệnh sc query. Danh sách sau đây hiển thị các dịch vụ đã được cài đặt cùng với loại và trạng thái hiện tại của chúng:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230727.png)

>**LƯU Ý**<br>
Bạn cũng có thể xem xét tiện ích PsService của Sysinternals (http://technet.microsoft.com/en-us/sysinternals/bb897542.aspx), nó cũng cho phép bạn liệt kê các dịch vụ trên các hệ thống từ xa.

Mặc dù những công cụ này có thể cung cấp cái nhìn tổng quan về hoạt động liên quan đến dịch vụ, nhưng thường thiếu chi tiết cần thiết để thực hiện điều tra chi tiết. Đối với phân tích toàn diện, bạn thực sự cần tham khảo RAM cũng như dữ liệu trong registry. Nếu bạn có một hình ảnh đĩa dự phòng của máy đối tượng, bạn có thể khôi phục hive registry hệ thống (C:\Windows\system32\config\system) và phân tích nó bằng một công cụ như RegRipper hoặc RegDecoder. Tuy nhiên, hãy nhớ rằng một phần lớn registry đang có trong bộ nhớ (xem Chương 10), vì vậy việc truy cập vào một bản nhớ dump thường là đủ để thực hiện các mục tiêu phân tích sắp tới.

**Mục tiêu**

-	Xác định các dịch vụ được tạo gần đây: Bằng cách sử dụng các dấu thời gian chỉnh sửa lần cuối trên khóa registry của dịch vụ, bạn có thể xác định được thời điểm dịch vụ được thêm vào hệ thống. Tuy nhiên, các thay đổi cấu hình sau khi cài đặt (như loại bắt đầu, phụ thuộc, hoặc mô tả) sẽ ghi đè lên giá trị này và che giấu thời gian tạo.
-	Phát hiện các dịch vụ trong trạng thái không hợp lệ: Bạn có thể phát hiện khi các dịch vụ không được ủy quyền đang chạy (ví dụ: RDP hoặc Telnet) hoặc khi các dịch vụ bảo mật đã bị dừng đột ngột. Bằng cách thực hiện kiểm tra định kỳ trên các hệ thống trực tiếp, bạn cũng có thể phát hiện bất kỳ thay đổi nào trong trạng thái dịch vụ giữa các khoảng thời gian.
-	Xác định các dịch vụ bị chiếm đoạt: Trong nhiều trường hợp, phần mềm độc hại tránh việc tạo một dịch vụ mới. Thay vào đó, nó chọn một dịch vụ hiện có và ghi đè đường dẫn tới tệp nhị phân của dịch vụ trong registry - chuyển hướng nó đến một tệp độc hại. Ngoài ra, phần mềm độc hại có thể patch hoặc thay thế tệp nhị phân dịch vụ trên ổ đĩa (nếu có quyền).
-	Xác định các bản ghi dịch vụ bị ẩn (không liên kết): Rootkit Blazgel là mẫu phần mềm độc hại đầu tiên mà chúng tôi gặp phải (nhưng chắc chắn không phải là cuối cùng) ẩn đi khỏi SCM bằng cách thao tác danh sách các cấu trúc bản ghi dịch vụ trong bộ nhớ. Như bạn sẽ thấy trong ví dụ sắp tới, bạn có thể phát hiện điều này bằng cách sử dụng plugin svcscan trong Volatility.

**Cấu trúc dữ liệu**

Như đã nói trước đó, SCM (Service Control Manager) duy trì một danh sách liên kết các cấu trúc chứa thông tin về các dịch vụ đã được cài đặt. Các cấu trúc này có thành viên tại một offset cố định với giá trị hằng số là sErv hoặc serH (tùy thuộc vào phiên bản hệ điều hành), điều này giúp chúng dễ dàng tìm thấy. Tuy nhiên, Microsoft không tài liệu hóa các cấu trúc này, vì vậy chúng tôi phải đặt tên riêng cho các cấu trúc và thành viên của chúng. Dưới đây là cấu trúc của các hệ thống Windows 7 64 bit:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230741.png)

**>Chú ý:**<br> Các cấu trúc _SERVICE_RECORD trên các phiên bản cũ hơn của Windows như XP và 2003 bao gồm một ServiceList (danh sách liên kết hai chiều) thay vì PrevEntry (danh sách liên kết một chiều). Bất kể chúng được liên kết như thế nào, vị trí của một dịch vụ trong danh sách cho thấy khi nào SCM đã đọc cấu hình của nó từ registry (thường theo thứ tự bảng chữ cái), không phải là thứ tự mà các dịch vụ đã bắt đầu. Một ngoại lệ là các dịch vụ được tạo sau khi hệ thống khởi động - trong trường hợp này, dịch vụ mới sẽ được thêm vào cuối danh sách và nó sẽ có trật tự cao nhất bất kể tên của nó là gì. Điều này hữu ích vì bạn có thể dễ dàng phân biệt các dịch vụ được cài đặt sau đó từ các dịch vụ đã tồn tại từ trước.

**Điểm chính**

Các điểm chính cho cấu trúc _SERVICE_HEADER như sau:
- Tag: Thành viên này chứa giá trị cố định là sErv hoặc serH để xác định tiêu đề bản ghi dịch vụ.
- ServiceRecord: Con trỏ đến cấu trúc bản ghi dịch vụ đầu tiên mà tiêu đề chứa.

Các điểm chính cho cấu trúc _SERVICE_RECORD như sau:
- PrevEntry: Danh sách liên kết một chiều này kết nối cấu trúc dịch vụ với cấu trúc trước đó.
- ServiceName: Thành viên này trỏ đến một chuỗi Unicode chứa tên dịch vụ (ví dụ: spooler hoặc SharedAccess).
- DisplayName: Một tên mô tả chi tiết hơn cho dịch vụ - ví dụ: tên hiển thị của dịch vụ Smb là Message-oriented TCP/IP và TCP/IPv6 Protocol (SMB session).
- FullServicePath: Thành viên này có thể có ý nghĩa khác nhau tùy thuộc vào loại dịch vụ. Nếu dịch vụ là một trình điều khiển hệ thống tệp hoặc trình điều khiển kernel, thành viên FullServicePath trỏ đến một chuỗi Unicode chứa tên đối tượng trình điều khiển (ví dụ: \Driver\Tcpip). Nếu dịch vụ là một quy trình, thành viên FullServicePath trỏ đến một cấu trúc _SERVICE_PATH chứa đường dẫn đầy đủ trên đĩa đến tệp thực thi và ID quy trình hiện tại nếu dịch vụ đang chạy.
- ServiceType: Thành viên này xác định loại dịch vụ.
- CurrentState: Thành viên này xác định trạng thái hiện tại của dịch vụ.

>Lưu ý rằng trên trang web Microsoft Developer Network (MSDN), chi tiết danh sách đầy đủ về các loại và trạng thái có thể có được tìm thấy tại đây: http://msdn.microsoft.com/en-us/library/windows/desktop/ms685996(v=vs.85).aspx. Bạn cũng có thể tìm thấy chúng trong các tệp tiêu đề WinNt.h và WinSvc.h từ Bộ công cụ Phát triển Phần mềm Windows (SDK).

### Scanning Memory

Có một vài cách để liệt kê các dịch vụ bằng cách phân tích bộ nhớ của quy trình. Một nhà lập trình có tên EiNSTeiN_ đã viết một công cụ được gọi là Hidden Service Detector (hsd), chạy trên các hệ thống Windows đang hoạt động. Nó hoạt động bằng cách quét bộ nhớ của services.exe để tìm PServiceRecordListHead - một biểu tượng chỉ đến đầu của danh sách kép các cấu trúc _SERVICE_RECORD trên các hệ thống XP và 2003. Cụ thể, hsd quét services.exe để tìm các mã byte của các chỉ thị sau đây:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230758.png)

Trong mã code, các byte xx biểu thị cho các ký tự đại diện (wildcards). Sau khi hsd tìm thấy chữ ký của danh sách đầu, nó liệt kê tất cả các bản ghi dịch vụ bằng cách duyệt qua danh sách. Đây là một phương pháp thú vị, nhưng giống như các danh sách liên kết khác, phần mềm độc hại có thể tách các mục nhập để ẩn các dịch vụ đang chạy. Hơn nữa, ký hiệu PServiceRecordListHead đã bị loại bỏ sau Windows 2003, vì vậy kỹ thuật này chỉ áp dụng cho các hệ thống cũ hơn.

### Volatility’s SvcScan Plugin

Do đó, do các điểm yếu trong kỹ thuật quét bộ nhớ được đề cập trước đó và các vấn đề liên quan đến tính di động, plugin svcscan của Volatility hoạt động một chút khác biệt. Nó vẫn sử dụng danh sách liên kết, nhưng nó cũng thực hiện một quét brute force qua tất cả các bộ nhớ thuộc sở hữu của services.exe để tìm các thẻ sErv hoặc serH. Vì các thẻ này được nhúng trong các thành viên của từng cấu trúc _SERVICE_RECORD, bạn có thể tìm thấy tất cả các phiên bản của các cấu trúc ngay cả khi chúng đã bị tách khỏi danh sách. Trên thực tế, sau đó bạn có thể so sánh các mục được tìm thấy bằng cách quét với các mục được tìm thấy thông qua việc đi qua danh sách và xác định chính xác các dịch vụ (nếu có) đã bị tách khỏi danh sách một cách độc hại - điều này sẽ được thảo luận chi tiết hơn trong phần sau của chương. 

Tạm thời, chỉ để bạn làm quen với định dạng đầu ra của plugin, đây là một ví dụ từ một hệ thống Windows sạch sẽ:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230819.png)

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230831.png)

Kết quả đầu ra cho thấy hai dịch vụ đầu tiên (BITS và CertPropSvc) đều được triển khai dưới dạng các tập tin DLL (tương ứng là qmgr.dll và certprop.dll) và chúng đang chạy trong cùng một quy trình chủ (PID 892). Dịch vụ WSearch là một quy trình riêng biệt (SearchIndexer.exe) và đang chạy với PID 1240. Hai dịch vụ cuối cùng (usbuhci và AFD) đều là cho các trình điều khiển kernel, nhưng chỉ có một trong số chúng đang chạy. Vì usbuhci đang dừng, nên đường dẫn tập tin của nó không được chỉ định, có nghĩa là không có trình điều khiển nào được tải cho dịch vụ này. Ngược lại, dịch vụ AFD đang chạy, vì vậy đường dẫn tập tin của nó là (\Driver\AFD) có thể truy cập được.

### Recently Created Services

Khóa registry của một dịch vụ sẽ được cập nhật với mốc thời gian ghi mới nhất khi dịch vụ được tạo ra và sau đó khi nó được sửa đổi vì bất kỳ lý do nào. Nếu bạn nghi ngờ malware vừa sử dụng gần đây các dịch vụ, nhưng không biết chính xác tên dịch vụ, một bước bạn có thể thực hiện là sắp xếp tất cả các dịch vụ theo thời gian ghi và sau đó xem các dịch vụ mới nhất. Ví dụ sau đây phân tích Stuxnet, một phần mềm độc hại, tránh cơ chế SCM bằng cách sử dụng "NtLoadDriver trick" đã thảo luận trước đó. Do đó, plugin svcscan không phát hiện ra các dịch vụ của nó, nhưng bạn vẫn có thể phân tích các khóa registry. Để làm điều này trong các cuộc điều tra của riêng bạn, bạn có thể sử dụng plugin volshell.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230843.png)

Bây giờ bạn có thể nhập mô-đun Registry API, như đã thảo luận trong Chương 10:

```
>>> import volatility.plugins.registry.registryapi as registryapi
>>> regapi = registryapi.RegistryApi(self._config)
```

Các dòng mã sau đọc tất cả các khóa con trong khóa ControlSet001\Services được chỉ định trong hệ thống hive:

```
>>> key = "ControlSet001\Services"
>>> subkeys = regapi.reg_get_all_subkeys("system", key)
```

Trước khi bạn có thể sắp xếp theo thời gian, hãy xây dựng một từ điển các dịch vụ với tên dịch vụ là các khóa và giá trị là thời gian nguyên. Sau đó, sắp xếp các đồng nhất thời gian theo thứ tự đảo ngược (để các thời gian gần đây nhất xuất hiện đầu danh sách):

```
>>> services = dict((s.Name, int(s.LastWriteTime)) for s in subkeys)
>>> times = sorted(set(services.values()), reverse=True)
```

Để lọc ba timestamps gần đây nhất và lặp qua từ điển đã tạo trước đó, in tên các dịch vụ khớp với các timestamps đó:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230856.png)

Đầu ra cho thấy có mười một dịch vụ được in ra trong ba khoảng thời gian duy nhất. Khoảng thời gian gần đây nhất (1307075207) tương đương với ngày 3 tháng 6 năm 2011 lúc 04:26:47 UTC. Tại thời điểm này, các dịch vụ MRxCls và MRxNet đã được tạo hoặc sửa đổi. Nên cảnh giác ngay lập tức với việc không có dịch vụ nào xuất hiện trong đầu ra của lệnh svcscan. Điều này là một dấu hiệu mạnh cho thấy hai dịch vụ này bị ẩn (hoặc chúng được khởi động không đúng cách); nếu không, SCM sẽ biết về chúng:

```
$ python vol.py -f stuxnet.vmem --profile=WinXPSP3x86 svcscan | egrep -i '(mrxnet|mrxcls)'
Volatility Foundation Volatility Framework 2.4

```

Một cách để xác minh xem các dịch vụ thực sự đang chạy, mặc dù không có cấu trúc _SERVICE_RECORD, là xác định trước tiên các module nhân liên quan. Đường dẫn được lưu trữ trong giá trị ImagePath của khóa registry tương ứng. Như bạn có thể thấy trong đầu ra dưới đây, module là mrxnet.sys:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230913.png)

Bạn có thể so sánh tên module đó với các kernel module đang được tải:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230926.png)

Module nghi ngờ thực sự đã được tải, điều này có nghĩa là dịch vụ MRxNet đang hoạt động. Tại thời điểm này, bạn đã sử dụng các timestamps trong registry để xác định dịch vụ(s) được tạo hoặc chỉnh sửa gần đây nhất, và từ đó xác định rằng hai trong số chúng đã được ẩn khỏi SCM.

### Detecting Hijacked Services

Nếu kẻ tấn công chỉ muốn sử dụng dịch vụ như một phương tiện để tồn tại, thì thường không cần tạo một dịch vụ hoàn toàn mới. Ví dụ, có quá nhiều dịch vụ trên máy tính thông thường bị tắt hoặc không được sử dụng, vậy tại sao không tái sử dụng (tức là chiếm đoạt) một dịch vụ hiện có? Nhiều nhóm đe dọa áp dụng phương pháp này. Nếu bạn chỉ dựa vào việc tìm kiếm các tên dịch vụ không tiêu chuẩn hoặc không được nhận dạng, bạn sẽ bỏ qua những cuộc tấn công này. Dưới đây là hai biến thể của việc chiếm đoạt dịch vụ hiện có, được mô tả cụ thể.

#### Registry-based Hijacks

Phương pháp này, được sử dụng bởi Nhóm Comment (còn được gọi là APT1), đơn giản là thay đổi giá trị ImagePath hoặc ServiceDll trong registry của một dịch vụ hợp pháp để nó trỏ tới một tập tin độc hại. Ví dụ, mẫu WEBC2-ADSPACE (http://contagiodump.blogspot.com/2013/03/mandiant-apt1-samples-categorized-by.html) đã chiếm đoạt dịch vụ ERSvc (Error Reporting Service) bằng cách sửa đổi đường dẫn DLL từ %SystemRoot%\system32\ersvc.dll thành %SystemRoot%\system\ersvc.dll.

Dưới đây là một ví dụ về cách đầu ra của Volatility trên một hệ thống bị nhiễm bệnh:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230938.png)


Đôi khi Microsoft thay đổi tên của một DLL dịch vụ hợp pháp giữa các phiên bản hệ điều hành. Ví dụ, trên Windows XP, tệp nhị phân của Dịch vụ Giao thức Cấu hình Động (DCHP) là dhcpsvc.dll; trên Windows 7, nó là dhcpcore.dll. Do đó, việc theo dõi các thay đổi này là một công việc phức tạp. Thay vào đó, bạn có thể sử dụng đoạn mã sau để xây dựng một danh sách trắng các đường dẫn đã biết đến cho các tệp nhị phân dịch vụ thông qua việc xuất thông tin từ một hệ thống sạch:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20230952.png)



