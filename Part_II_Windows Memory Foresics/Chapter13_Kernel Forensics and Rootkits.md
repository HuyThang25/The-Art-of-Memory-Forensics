Cho đến nay trong cuốn sách này, bạn đã học được rất nhiều về các hiện vật tồn tại trong bộ nhớ kernel, chẳng hạn như đối tượng tệp, cấu trúc mạng và các họ hive registry được lưu trữ trong bộ nhớ cache. Chúng ta đã thảo luận về các chủ đề như ẩn các tiến trình bằng cách trực tiếp sửa đổi các đối tượng kernel. Tuy nhiên, bạn chưa được học cách tìm ra và theo dõi malware chạy trong chế độ kernel bằng cách tải một trình điều khiển (driver). Hơn nữa, một rootkit khi chạy trong kernel có vô số cách để tránh bị phát hiện và tồn tại trong hệ thống bằng cách điều khiển các bảng gọi (call tables), gắn kết các hàm (hooking functions) và ghi đè lên các cấu trúc dữ liệu siêu dữ liệu (metadata structures). 

Chương này sẽ hướng dẫn cách phân tích bộ nhớ giúp bạn phát hiện các rootkit nổi tiếng như ZeroAccess, Tigger, Blackenergy và Stuxnet. Bạn cũng sẽ có cơ hội trải nghiệm cách kết hợp Volatility với IDA Pro để phân tích tĩnh sâu về các module kernel độc hại.

## Kernel Modules

Sơ đồ được hiển thị trong Hình 13-1 thể hiện, một cách tổng quan, một số khái niệm được đề cập trong chương này. Khi bạn thực hiện phân tích bộ nhớ kernel, bạn thường tìm kiếm một module kernel độc hại - và có nhiều cách để làm điều đó. Như được hiển thị trong sơ đồ, khối dữ liệu gỡ lỗi kernel có một thành viên được đặt tên là PsLoadedModuleList, trỏ đến một danh sách liên kết kép của cấu trúc KLDR_DATA_TABLE_ENTRY. Những cấu trúc này chứa thông tin dữ liệu về mỗi module kernel, chẳng hạn như vị trí cơ sở của nó (tức là điểm bắt đầu của tệp PE), kích thước của module và đường dẫn đầy đủ đến tệp của module trên đĩa. Trên hệ thống hoạt động, và do đó bất kỳ công cụ pháp y tế nào phụ thuộc vào các API này, đều liệt kê các module bằng cách duyệt qua danh sách này. Do đó, rootkit có thể ẩn sự tồn tại của nó bằng cách mất liên kết một mục. Trong sơ đồ, mục ở giữa đã bị mất liên kết.

Mặc dù một mục đã bị mất liên kết, cấu trúc dữ liệu siêu dữ liệu vẫn còn nguyên vẹn (tức là không bị xóa). Do đó, có thể tìm thấy cấu trúc(s) bằng cách sử dụng phương pháp quét bể (pool-scanning) (xem Chương 5). Cụ thể, các cấu trúc dữ liệu siêu dữ liệu tồn tại trong các bể được gắn thẻ bằng MmLd, đó là cách plugin modscan của Volatility tìm thấy chúng. Tiếp theo là một rootkit khá kỹ lưỡng hơn, giả sử rằng cấu trúc dữ liệu siêu dữ liệu đã bị mất liên kết và sau đó bị ghi đè bằng toàn bộ giá trị 0, bao gồm cả thẻ bể. Trong trường hợp này, không thể xác định được module ẩn bằng cách duyệt qua danh sách hoặc quét thẻ bể. Nhưng đừng lo lắng - chỉ có cấu trúc dữ liệu siêu dữ liệu bị nhắm mục tiêu ở đây. Dữ liệu thực tế (tức là tệp thực thi [PE] cùng với tất cả các chức năng của nó) vẫn có thể truy cập.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233728.png)

Nếu bạn gặp phải một rootkit ẩn mình theo cách đã đề cập, bạn vẫn có thể thực hiện quét brute force thông qua bộ nhớ kernel để tìm các tiêu đề PE (ví dụ, chữ ký MZ). Cụ thể, tìm các trường hợp trong đó địa chỉ cơ sở không được đại diện trong danh sách các module liên kết, đây là dấu hiệu mạnh rằng tệp PE bạn tìm thấy đã bị ẩn. Thật không may, kỹ thuật này không giúp bạn khôi phục lại đường dẫn đầy đủ trên đĩa cho module, nhưng ít nhất bạn có thể trích xuất nó từ bộ nhớ và thực hiện phân tích tĩnh của tệp nhị phân.

Bây giờ hãy xem xét một rootkit còn tinh vi hơn, cũng xóa bỏ tiêu đề PE (và chữ ký MZ). Sau khi được tải vào bộ nhớ, những giá trị này không cần thiết và có thể dễ dàng bị hỏng để phá vỡ sự phân tích pháp y. Một điều đáng an ủi mà bạn có thể dựa vào là mã của module kernel độc hại vẫn phải duy trì trong bộ nhớ để rootkit hoạt động. Nói cách khác, dù nó ẩn mình như thế nào, nó vẫn phải duy trì một số hiện diện, đó chính là điểm yếu mà bạn có thể khai thác để phát hiện.

Ví dụ, nếu module muốn giám sát cuộc gọi API, nó gắn kết bảng điều khiển Dịch vụ Hệ thống (SSDT), được mô tả sau trong chương, hoặc vá lỗ trong phần mã của module khác bằng các chỉ thị chuyển hướng điều khiển đến chức năng của rootkit. Nếu module muốn giao tiếp với các tiến trình trong chế độ người dùng, nó cần một đối tượng trình điều khiển (xem plugin driverscan) và một hoặc nhiều thiết bị (plugin devicetree). Bất kỳ lúc nào, nếu module khởi chạy các luồng bổ sung để thực hiện các nhiệm vụ đồng thời, nó sẽ dẫn đến việc tạo ra một đối tượng luồng mới có địa chỉ bắt đầu trỏ trực tiếp đến mã của module.

Trong chương này, bạn sẽ thấy cách tận dụng những hiện vật gián tiếp này để xác định và trích xuất mã của module kernel độc hại, bất kể nó ẩn mình như thế nào.

### Classifying Modules

Một hệ thống Windows thông thường có hàng trăm module kernel, vì vậy xác định module độc hại có thể đòi hỏi một lượng công sức đáng kể. Trong khi bạn đọc qua chương này, hãy cân nhắc các câu hỏi sau. Chúng sẽ giúp bạn xác định các module nào nên tập trung vào trong quá trình điều tra:

- Nó có bị mất liên kết hoặc ẩn không? Ngoại trừ một số sản phẩm antivirus cố gắng ẩn đi khỏi malware, không có lý do hợp lệ nào để mất liên kết cấu trúc dữ liệu siêu dữ liệu của module.

- Nó có xử lý bất kỳ ngắt nào không? Mặc dù các module kernel của bên thứ ba có thể đăng ký các trình xử lý ngắt riêng, module NT (ntoskrnl.exe, ntkrnlpa.exe, v.v.) luôn luôn là trình xử lý của các ngắt quan trọng như page fault, breakpoint trap và hệ thống dịch vụ dispatcher. Bạn có thể kiểm tra chúng bằng plugin idt của Volatility.

- Nó cung cấp bất kỳ API hệ thống nào không? Khi các ứng dụng chế độ người dùng gọi các API hệ thống, địa chỉ của API trong bộ nhớ kernel được giải quyết bằng cách sử dụng một bảng con trỏ gọi là SSDT. Kẻ thù có thể ghi đè lên các con trỏ này, trừ trường hợp trên các nền tảng 64 bit có Patchguard được kích hoạt. Thông thường, chỉ có các module NT, module giao diện người dùng đồ họa của Windows (win32k.sys), trình điều khiển hỗ trợ IIS (spud.sys) và một số sản phẩm antivirus mới được liên quan đến các API hệ thống.

- Trình điều khiển có được ký không? Trên các hệ thống 64 bit, tất cả các module kernel cần được ký. Bạn nên kiểm tra xem người ký là đáng tin cậy, chứng chỉ có bị hết hạn không, v.v. Tuy nhiên, bạn sẽ cần các tệp module kernel tương ứng từ đĩa để xác minh chữ ký, do sự thay đổi xảy ra khi tải các module vào bộ nhớ.

- Tên và đường dẫn có hợp lệ không? Đôi khi chỉ một chỉ báo rất đơn giản, chẳng hạn như tên module hoặc đường dẫn đầy đủ trên đĩa, có thể tiết lộ các hành vi đáng ngờ. Ví dụ, Stuxnet đã sử dụng các tên cứng nhắc như MRxNet.sys, và Blackenergy đã tải một module có tên hoàn toàn được tạo thành từ ký tự hex (ví dụ, 000000BD8.sys). Ngoài ra, kiểm tra xem các module có được tải từ các đường dẫn tạm không.

- Nó cài đặt bất kỳ hồ sơ nào không? Như bạn sẽ thấy sau trong chương, các hồ sơ cung cấp cơ chế nhận thông báo khi các sự kiện cụ thể xảy ra, chẳng hạn như khi tiến trình mới bắt đầu hoặc một bản ghi crash sẽ được tạo. Bạn nên nhận thức về các hồ sơ, nếu có, mà một module cụ thể đã đăng ký.

- Nó tạo ra bất kỳ thiết bị nào không? Thiết bị thường có tên, điều này có nghĩa là bạn có thể sử dụng chúng làm các chỉ báo về sự nhiễm độc (miễn là chúng không được tạo ngẫu nhiên). Bạn cũng nên xem xem các thiết bị của một trình điều khiển có đang hoạt động như một bộ lọc bằng cách gắn kết vào bộ lắc bàn phím, mạng hoặc tệp hệ thống điều khiển.

- Có bất kỳ chữ ký nào đã biết không? Cuối cùng nhưng không kém, các kiểm tra nội dung mã bằng phương pháp dò tìm (brute force) có thể rất hữu ích. Trích xuất một module từ bộ nhớ hoặc tất cả chúng và quét chúng với các chữ ký antivirus hoặc các luật Yara.

### How Modules Are Loaded

Hành động đơn giản của việc tải một module kernel dẫn đến việc tạo ra các hiện vật khác nhau trong bộ nhớ. Tuy nhiên, các bằng chứng cụ thể phụ thuộc vào kỹ thuật sử dụng. Dưới đây là mô tả ngắn về các phương pháp có thể có và một số dấu vết mà bạn có thể mong đợi tìm thấy:

- Trình quản lý điều khiển dịch vụ (SCM): Cách mà Microsoft khuyến nghị để tải các module kernel là tạo một dịch vụ (CreateService) có loại SERVICE_KERNEL_DRIVER và sau đó khởi động dịch vụ (StartService). Như đã mô tả trong Chương 12, những API này tự động tạo một khóa con trong registry dưới CurrentControlSet\services với tên tương ứng với dịch vụ mới. Phương pháp này cũng tạo ra các thông báo trong nhật ký sự kiện nếu việc kiểm tra tính minh bạch được kích hoạt. Hơn nữa, một cấu trúc bản ghi dịch vụ mới được tạo trong bộ nhớ của services.exe (xem plugin svcscan). Có thể gỡ bỏ trình điều khiển bằng cách dừng dịch vụ một cách đơn giản.

- NtLoadDriver: Chương 12 mô tả một mẫu malware đã bỏ qua một số hiện vật pháp y mà phương pháp SCM để lại. Mặc dù các khóa registry vẫn cần thiết, nếu bạn gọi trực tiếp NtLoadDriver (thay vì CreateService và StartService), các thông báo nhật ký sự kiện không được phát ra, và services.exe không được thông báo về hoạt động này. Bạn vẫn có thể gỡ bỏ module một cách dễ dàng bằng cách gọi NtUnloadDriver.

- NtSetSystemInformation: Một phương pháp tinh vi hơn để tải các module bao gồm gọi API này với lớp SystemLoadAndCallImage (xem http://www.shmoo.com/mail/bugtraq/aug00/msg00404.shtml). Mặc dù đây là phương pháp duy nhất không yêu cầu các mục nhập registry, sau khi bạn tải một module theo cách này, không có cách dễ dàng để gỡ bỏ nó - ngoại trừ khởi động lại máy.

Bây giờ bạn đã biết những API cần thiết cho mỗi trong những phương pháp này, bạn có thể nhận ra chúng khi phân tích các cuộc gọi hàm đã được nhập vào của một ứng dụng.

### Enumerating Modules on Live Systems

Việc làm quen với cách mà các công cụ trực tiếp liệt kê các module kernel là rất quan trọng để hiểu cách chúng thường bị xuyên tạc. Dưới đây là một danh sách các nguồn tài nguyên có sẵn:

- Process Explorer: Nếu bạn nhấp vào tiến trình "System" và chọn Xem ➪ Xem bảng dưới cùng ➪ DLLs, bạn sẽ thấy danh sách các module kernel hiện đang được tải. Hình 13-2 hiển thị hình ảnh về cách nó xuất hiện.

  ![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233748.png)

- Windows API: Hàm EnumDeviceDrivers (xem K32EnumDeviceDrivers) có thể lấy được địa chỉ tải của mỗi module kernel. Bên trong, những API trợ giúp này gọi NtQuerySystemInformation.

- Windows Management Instrumentation (WMI): Bạn có thể sử dụng lớp Win32_SystemDriver để liệt kê các trình điều khiển hệ thống. Lưu ý rằng lớp này được kế thừa từ Win32_BaseService, do đó, nó thực tế là tham khảo registry (không phải NtQuerySystemInformation) để thu thập tập con các dịch vụ đã được cài đặt tải các module kernel.

- Nirsoft: Ứng dụng GUI DriverView (http://www.nirsoft.net/utils/driverview.html) của Nirsoft hiển thị một danh sách các module đã được tải bằng cách gọi NtQuerySystemInformation.

- Native API: Chương trình C hoặc C++ có thể gọi trực tiếp NtQuerySystemInformation với lớp SystemModuleInformation để thu thập danh sách các module kernel đã được tải. API này, trên đó rất nhiều công cụ khác dựa vào, tham chiếu đến danh sách liên kết kép của các cấu trúc KLDR_DATA_TABLE_ENTRY mô tả trong Hình 13-1.

## Modules in Memory Dumps

Volatility được trang bị đầy đủ để tìm, báo cáo và trích xuất các module kernel từ bộ nhớ. Dưới đây là danh sách các plugin mà bạn sẽ sử dụng phổ biến nhất cho các hành động này:

- modules: Plugin này đi qua danh sách liên kết kép của các cấu trúc dữ liệu siêu dữ liệu được trỏ đến bởi PsLoadedModuleList. Vì các module mới được tải luôn được thêm vào cuối danh sách, plugin này có ưu điểm hiển thị quan hệ thời gian tương đối giữa các module (nghĩa là bạn có thể biết thứ tự mà các module đã được tải).

- modscan: Plugin này sử dụng quét pool tag qua không gian địa chỉ vật lý, bao gồm bộ nhớ đã giải phóng/hủy, để tìm kiếm MmLd (pool tag dữ liệu siêu dữ liệu của module). Nó cho phép bạn tìm thấy cả các module đã bị mất liên kết và các module đã được tải trước đó.

- unloadedmodules: Đối với mục đích gỡ lỗi, kernel duy trì một danh sách các module đã được gỡ bỏ gần đây. Bên cạnh tên module, nó lưu trữ các dấu thời gian để chỉ ra chính xác khi nào chúng đã bị gỡ bỏ và các vị trí trong bộ nhớ kernel chúng đã chiếm giữ.

- moddump: Plugin này trích xuất một hoặc nhiều module kernel mà bạn xác định bằng tên hoặc địa chỉ cơ sở. Nó chỉ có thể trích xuất các module đang được tải hiện tại có tiêu đề PE hợp lệ.

### Ordered List of Active Modules

Đầu ra sau đây hiển thị một ví dụ về việc sử dụng plugin modules. Quan trọng là hiểu sự khác biệt giữa các cột Offset(V) và Base. Cột trước đó (hiển thị ở cột bên trái cùng) là địa chỉ ảo của cấu trúc dữ liệu siêu dữ liệu KLDR_DATA_TABLE_ENTRY. Còn cột sau đó là địa chỉ cơ sở (cũng trong bộ nhớ ảo) của đầu của tiêu đề PE của module. Do đó, trên hệ thống cụ thể này, bạn sẽ mong đợi tìm thấy chữ ký MZ cho module NT, ntoskrnl.exe, tại địa chỉ 0xfffff80002852000.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233806.png)

The NT module là module đầu tiên tải và tiếp theo là hal.dll (hardware abstraction layer). Điều này hợp lý vì cả hai là các thành phần chính của hệ điều hành cần bắt đầu sớm. Sau đó, bạn sẽ thấy các trình điều khiển liên quan đến các dịch vụ cụ thể bắt đầu tự động khi khởi động, chẳng hạn như trình điều khiển giao tiếp bộ gỡ lỗi kernel (kdcom.dll) và trình điều khiển Bluetooth (BthEnum.sys). Cuối cùng của danh sách hiển thị module mới nhất đã được tải, PROCEXP152.SYS, liên quan đến SysInternals Process Explorer—mà người dùng đã khởi động tương tác.

Nếu hệ thống bị nhiễm rootkit kernel, bạn sẽ thấy một mục mới cho module độc hại được thêm vào cuối danh sách (giả sử bạn thu thập bộ nhớ trước khi khởi động lại tiếp theo và rootkit không cố gắng ẩn cấu trúc dữ liệu siêu dữ liệu của nó).

### Brute Force Scanning for Modules

Đầu ra của plugin modscan giống như cái bạn vừa thấy. Tuy nhiên, có một số sự khác biệt chính:

- Bởi vì cấu trúc dữ liệu siêu dữ liệu của module được tìm thấy bằng cách quét qua không gian địa chỉ vật lý, cột bên trái, Offset(P), hiển thị một phần bù vật lý thay vì một địa chỉ trong bộ nhớ ảo.

- Các module xuất hiện theo thứ tự mà chúng được tìm thấy, không phải theo thứ tự chúng được tải.

Tất nhiên, vì modscan cũng kiểm tra các khối bộ nhớ được giải phóng và hủy, bạn có thể tìm thấy các module không được liên kết và đã được tải trước đó. Dưới đây là một ví dụ về đầu ra:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233826.png)

### Recently Unloaded Modules

Đầu ra dưới đây hiển thị một ví dụ về plugin unloadedmodules. Như đã đề cập trước đó, kernel duy trì danh sách này cho mục đích gỡ lỗi. Ví dụ, một module có thể đặt một Deferred Procedure Call (DPC) hoặc lên lịch một bộ đếm thời gian, nhưng sau đó gỡ bỏ mà không hủy bỏ chúng. Do đó, khi thủ tục được kích hoạt, hàm xử lý dự kiến ​​không còn tồn tại trong bộ nhớ. Điều này có thể gây ra vấn đề con trỏ treo và dẫn đến hậu quả không thể đoán trước. Nếu kernel không duy trì danh sách này của các module đã được gỡ bỏ gần đây và các phạm vi địa chỉ mà chúng đã chiếm giữ, việc xác định module nào gây lỗi sẽ gần như không thể.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233839.png)


Danh sách các module đã được gỡ bỏ cũng rất hữu ích cho việc điều tra pháp y và phân tích mã độc, đặc biệt là khi các rootkit cố gắng gỡ bỏ nhanh chóng (tức là tiếp cận vào, tiếp cận ra). Như đã thấy trong ví dụ dưới đây từ một biến thể Rustock.C, bạn không thể tìm thấy module xxx.sys trong danh sách các module hoạt động hoặc thông qua quét theo pool tag. Tuy nhiên, trong một cấu trúc dữ liệu hoàn toàn khác, kernel vẫn ghi nhớ rằng module độc hại đã từng được tải.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233852.png)

Đáng tiếc là vì module xxx.sys thực sự đã được gỡ bỏ, bạn không còn mong đợi trích xuất nó ra khỏi bộ nhớ. Tuy nhiên, ít nhất bạn có một dấu thời gian được liên kết với hoạt động đó mà bạn có thể sử dụng trong các cuộc điều tra dựa trên dòng thời gian, và bạn cũng có tên tập tin trên đĩa, vì vậy bạn có thể cố gắng khôi phục nó từ hệ thống tập tin.

### Extracting Kernel Modules

Khi một module kernel vẫn được tải vào bộ nhớ, bạn có thể trích xuất nó để phân tích tĩnh bằng plugin moddump. Dưới đây là các tùy chọn dòng lệnh có sẵn:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233906.png)

Để trích xuất tất cả các module hiện đang được tải, chỉ cần cung cấp một đường dẫn đến thư mục đầu ra mong muốn của bạn, như sau:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233917.png)

Lưu ý rằng tên của tệp đầu ra là driver.ADDR.sys, trong đó ADDR là địa chỉ cơ sở của module trong bộ nhớ kernel. Bởi vì chỉ có một module có thể chiếm một địa chỉ cụ thể tại một thời điểm, quy ước đặt tên này đảm bảo rằng các tên tệp đầu ra là duy nhất (so với việc dựa vào tên của module, gây ra xung đột).

Trong ví dụ tiếp theo, chúng ta trích xuất các module bằng cách sử dụng biểu thức chính quy không phân biệt chữ hoa/thường. Tiêu chí tcp khớp với hai module, tcpip.sys và tcpipreg.sys.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233928.png)

Mặc dù việc tìm kiếm bằng biểu thức chính quy rất tiện lợi, hãy nhớ rằng có những trường hợp bạn sẽ không có tên để tiến hành tìm kiếm (ví dụ, nếu cấu trúc dữ liệu siêu dữ liệu bị ghi đè hoặc nếu bạn tìm thấy một tiêu đề PE trong một phân bổ pool kernel ẩn danh). Trong những trường hợp này, bạn có thể cung cấp địa chỉ cơ sở (nơi bạn thấy chữ ký MZ) và moddump sẽ thực hiện việc trích xuất. Ví dụ dưới đây giả định rằng có một tệp PE tồn tại tại địa chỉ 0xfffff88003800000:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20233941.png)

Nếu bạn dự định tải module đã trích xuất vào IDA Pro để phân tích tĩnh, hãy nhớ một điều: địa chỉ ImageBase trong tiêu đề PE cần phải được thay đổi sao cho khớp với địa chỉ tải thực tế của nó trong bộ nhớ kernel. Nói cách khác, bạn nên sử dụng 0xfffff88003800000 cho ví dụ cuối cùng đã hiển thị. Dưới đây là cách bạn có thể làm điều này bằng cách sử dụng module Python pefile từ https://code.google.com/p/pefile:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234003.png)

Cải tiến đơn giản này cung cấp cho IDA Pro ngữ cảnh bổ sung cần thiết để hiển thị đúng cuộc gọi hàm tương đối, nhảy và tham chiếu chuỗi. Tùy thuộc vào trạng thái của bảng địa chỉ nhập của tệp nhị phân, bạn cũng có thể cần sử dụng plugin impscan của Volatility để tạo các nhãn mà bạn có thể áp dụng vào cơ sở dữ liệu IDA. Bạn sẽ thấy một ví dụ về việc sử dụng impscan sau trong chương trình (xem cũng "Recipe 16-8: Scanning for Imported Functions with ImpScan" trong "Malware Analyst’s Cookbook").

## Threads in Kernel Mode

Khi các module kernel tạo các luồng mới bằng PsCreateSystemThread, quy trình System (PID 4 trên XP và phiên bản sau) trở thành chủ sở hữu của luồng. Nói cách khác, quy trình System là nơi mặc định cho các luồng bắt đầu trong chế độ kernel. Bạn có thể khám phá điều này bằng Process Explorer và thấy rằng địa chỉ bắt đầu của các luồng thuộc quy trình System là các offset trong các module kernel như ACPI.sys và HTTP.sys (xem Hình 13-3).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234018.png)

Khi phân tích qua một bản sao bộ nhớ, bạn có thể phân biệt các luồng hệ thống này khác biệt với những luồng khác dựa trên các yếu tố sau:
- Giá trị _ETHREAD.SystemThread là 1.
- Thành viên _ETHREAD.CrossThreadFlags có cờ PS_CROSS_THREAD_FLAGS_SYSTEM được thiết lập.
- Quy trình sở hữu có PID 4.

Thông tin này có thể giúp bạn tìm các gia đình mã độc như Mebroot và Tigger, cố gắng che giấu sự hiện diện của chúng trong kernel. Khi các module rootkit ban đầu tải, chúng phân bổ một pool bộ nhớ kernel, sao chép mã thực thi vào pool và gọi PsCreateSystemThread để bắt đầu thực thi khối mã mới. Sau khi luồng được tạo, module có thể gỡ bỏ. Những hành động này giúp rootkit tồn tại bằng cách thực thi các luồng từ các pool bộ nhớ không được đánh dấu. Tuy nhiên, điều này tạo ra một dấu vết rõ ràng cho việc pháp y cho vì bạn có một luồng với địa chỉ bắt đầu trỏ đến một khu vực không xác định của bộ nhớ kernel, trong đó không tồn tại module đã biết.

### Tigger’s Kernel Threads

Hình 13-4 cho thấy các luồng được sở hữu bởi quy trình System của máy bị nhiễm Tigger. Bạn có thể nhận thấy sự hiện diện của bốn luồng mới mà không tồn tại trong Hình 13-3. Process Explorer chỉ hiển thị địa chỉ bắt đầu của luồng thay vì định dạng thông thường như driverName.sys+0xabcd, bởi vì địa chỉ bắt đầu không nằm trong phạm vi bộ nhớ của bất kỳ module nào đã được tải.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234039.png)

### Detecting Orphan Threads

Các plugin threads có thể giúp bạn xác định những nỗ lực ẩn mình theo cách đã mô tả. Plugin này liệt kê các module đã được tải bằng cách đi qua danh sách liên kết gấp đôi và ghi lại địa chỉ cơ sở và kích thước của chúng. Sau đó, plugin quét các luồng hệ thống và kiểm tra xem giá trị _ETHREAD.StartAddress có nằm trong phạm vi của một trong các module hay không. Nếu plugin không thể ghép cặp một luồng với driver sở hữu của nó, nó giả định rằng luồng đó đã bị gỡ bỏ hoặc ẩn. Vì lý do này, các luồng này cũng được biết đến với cái tên "orphan threads" - những luồng mà không còn sở hữu. Dưới đây là đầu ra ví dụ cho cách các orphan threads xuất hiện trong bản sao bộ nhớ. Bạn sẽ thấy thẻ OrphanThread được hiển thị cũng như UNKNOWN bên phải địa chỉ bắt đầu (0xf2edd150).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234109.png)

Nếu bạn nghi ngờ rằng một kernel rootkit tồn tại trên hệ thống mà bạn đang điều tra, nhưng bạn không thể tìm thấy bằng chứng hỗ trợ bằng các plugin modules hoặc modscan, chúng tôi đề xuất kiểm tra các luồng mồ côi (orphan threads). Tuy nhiên, hãy lưu ý rằng địa chỉ bắt đầu của luồng sẽ trỏ đến một hàm bên trong tệp PE độc hại, chứ không phải là địa chỉ cơ sở của tệp PE. Do đó, bạn có thể cần phải thực hiện một số tính toán để tìm chữ ký MZ (đánh dấu bắt đầu của tệp PE). Gần cuối chương, bạn sẽ thấy một kịch bản volshell có thể quét ngược từ một địa chỉ cụ thể để tìm tiêu đề PE hợp lệ đầu tiên.

>**CẢNH BÁO**<br>
Rootkit có thể dễ dàng bypass kỹ thuật phát hiện các luồng mồ côi (orphan threads) bằng cách sửa đổi các giá trị _ETHREAD.StartAddress để trỏ đến một driver đã biết. Trong bài thuyết trình của họ trên VB2008 (http://www.virusbtn.com/pdf/conference_slides/2008/Kasslin-Florio-VB2008.pdf), Kimmo Kasslin và Elia Floria đã ghi nhận rằng thế hệ thứ ba của Mebroot đã bắt đầu áp dụng những bản vá này để tăng tính bất dấu của rootkit.

## Driver Objects and IRPs

Thông thường, khi một module kernel tải, ngoài việc tạo một cấu trúc KLDR_DATA_TABLE_ENTRY, một _DRIVER_OBJECT tương ứng cũng được khởi tạo. Điều này quan trọng vì đối tượng driver chứa thông tin quan trọng về module kernel của nó, chẳng hạn như bản sao của địa chỉ cơ sở của module, quy trình gỡ bỏ của nó và các con trỏ đến danh sách các hàm xử lý. Thông tin này có thể giúp bạn xác định các module độc hại khi các cấu trúc dữ liệu mô tả cho đến hiện tại trong chương trình bị tách rời hoặc bị hỏng. Hơn nữa, bằng cách tìm các đối tượng driver, bạn có thể kiểm tra các hook trong các hàm xử lý.

Để cung cấp thêm thông tin nền, các ứng dụng trong Windows giao tiếp với các driver bằng cách gửi các I/O Request Packets (IRPs). Một IRP là một cấu trúc dữ liệu bao gồm một số nguyên để xác định hoạt động mong muốn (tạo, đọc, ghi, v.v.) và bộ đệm cho bất kỳ dữ liệu nào cần được đọc hoặc ghi bởi driver. Mỗi đối tượng driver có một bảng gồm 28 con trỏ hàm mà nó có thể đăng ký để xử lý các hoạt động khác nhau. Thông thường, driver cấu hình bảng này, được biết đến là bảng hàm chính (major function table) hoặc bảng hàm IRP, trong hàm điểm nhập của nó ngay sau khi được tải. Đầu ra sau đây cho thấy bảng 28 con trỏ, được đặt tên là MajorFunction, là một phần của mỗi đối tượng driver:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234123.png)

**Các điểm chính**

- DeviceObject: Một con trỏ đến thiết bị đầu tiên được tạo bởi driver. Nếu một driver tạo nhiều hơn một thiết bị, chúng được liên kết với nhau thông qua một danh sách liên kết. Ví dụ, driver TCP/IP tạo các thiết bị có tên là RawIp, Udp, Tcp và Ip.
- DriverStart: Một bản sao của địa chỉ cơ sở của module kernel.
- DriverSize: Kích thước, tính bằng byte, của module kernel được mô tả bởi đối tượng driver.
- DriverExtension: Trỏ đến một cấu trúc với thành viên ServiceKeyName, cho biết đường dẫn trong registry lưu trữ cấu hình của driver này.
- DriverName: Đây là tên của đối tượng driver, chẳng hạn như \Driver\Tcpip hoặc \Driver\HTTP.
- DriverInit: Một con trỏ đến quy trình khởi tạo của driver.
- DriverUnload: Một con trỏ đến hàm được thực thi khi driver bị gỡ bỏ, thường để giải phóng các tài nguyên được tạo bởi driver.
- MajorFunction: Mảng chứa 28 con trỏ hàm chính. Bằng cách ghi đè một chỉ số trong mảng này, rootkit có thể hook vào các hoạt động cụ thể.

### Scanning for Driver Objects

Lệnh driverscan của Volatility tìm các đối tượng driver bằng cách quét theo pool tag. Dưới đây là một ví dụ về kết quả đầu ra của nó:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234138.png)

Đối với cấu trúc _DRIVER_OBJECT, vị trí offset vật lý hiển thị ở cột bên trái. Sau đó, bạn thấy địa chỉ bắt đầu của driver trong bộ nhớ kernel trong cột Bắt đầu (Start). Để bạn hiểu được cách điều này có thể hữu ích, địa chỉ bạn thấy cho \Driver\mouclass nên phù hợp với địa chỉ cơ sở cho mouclass.sys được hiển thị bởi các plugin modules hoặc modscan. Do đó, nếu phần mã độc ẩn hoặc xóa KLDR_DATA_TABLE_ENTRY, vẫn có một _DRIVER_OBJECT chứa không ít (nếu không nhiều hơn) thông tin về các module đang được tải trên hệ thống.

### Hooking and Hook Detection

Rootkits có thể hook các entry trong bảng IRP function của driver. Ví dụ, bằng cách ghi đè lên hàm IRP_MJ_WRITE trong bảng IRP của driver, một rootkit có thể kiểm tra nội dung của bộ đệm dữ liệu trước khi ghi xuống mạng, ổ cứng hoặc thậm chí máy in. Một ví dụ thường thấy khác là hook IRP_MJ_DEVICE_CONTROL cho tcpip.sys. Khi bạn sử dụng netstat.exe hoặc SysInternals TcpView.exe trên hệ thống, nó xác định các kết nối và socket hoạt động bằng kênh truyền thông này. Như vậy, bằng cách hook nó, rootkit có thể dễ dàng ẩn hoạt động mạng.

Để phát hiện các hook IRP function, bạn chỉ cần tìm các cấu trúc _DRIVER_OBJECT trong bộ nhớ, đọc 28 giá trị trong mảng MajorFunction và xác định nơi chúng trỏ đến. Tuy rằng việc này được tự động hoá bởi plugin driverirp, như bạn sẽ thấy trong thời gian tới, nó vẫn không đưa ra kết luận chắc chắn về việc các entry nào đã bị hook; nó vẫn yêu cầu bạn phân tích và hiểu ý nghĩa của chúng. Điều đó là vì có trường hợp hợp pháp khi một driver sẽ chuyển tiếp handler của nó đến một driver khác, làm cho nó trông giống như một hook.

Dưới đây là một ví dụ về kết quả đầu ra của plugin driverirp cho trình điều khiển Tcpip trên một hệ thống 64-bit sạch sẽ:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234153.png)

Trình điều khiển Tcpip bắt đầu từ địa chỉ 0xfffff880016bb000 và chiếm 0x20400 byte. Hầu hết các bộ xử lý của nó hoặc trỏ đến một hàm trong tcpip.sys (các hoạt động xử lý bởi chính nó) hoặc trỏ đến một module khác (các hoạt động được chuyển tiếp). Thay vì để các con trỏ là giá trị không, nếu một driver không có ý định xử lý một số hoạt động cụ thể, nó sẽ trỏ IRP đến nt!IopInvalidDeviceRequest, đây chỉ là một hàm giả (dummy) trong module NT, hoạt động như một lời gọi mặc định (tương tự như trường hợp mặc định trong câu lệnh switch trong C).

Dưới đây là một ví dụ về một máy tính 32-bit XP mà bộ xử lý IRP_MJ_DEVICE_CONTROL của trình điều khiển Tcpip đã bị hooked:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234206.png)

Chú ý rằng bộ xử lý trỏ đến một hàm trong url.sys, đây không phải là một trình điều khiển hệ thống bình thường. Trong trường hợp này, bạn có thể dump url.sys từ bộ nhớ và thực hiện reverse-engineer để xác định chính xác các socket và kết nối mạng mà nó đang cố gắng lọc trên máy tính thực tế.

### Stealthy Hooks

TDL3 là một ví dụ về rootkit đã vượt qua phương pháp phổ biến để phát hiện IRP hooks. Trong đầu ra dưới đây, tất cả các trình xử lý IRP cho vmscsi.sys đều dẫn đến một hàm mà ban đầu dường như cho thấy không có chuyển tiếp hoặc hook cho yêu cầu đó. Cụ thể, tất cả chúng đều trỏ đến địa chỉ 0xf9db9cbd, nằm trong phạm vi của bộ nhớ của trình điều khiển vmscsi.sys:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234219.png)

Hãy xem đoạn sơ đồ trong Hình 13-5, mô tả cách rootkit TDL3 vẫn có thể tiếp quản tất cả các hoạt động dành cho trình điều khiển vmscsi.sys.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234234.png)

Bảng biểu thị rằng rootkit thông thường ghi đè vào các mục bảng IRP và chỉ chúng ra bên ngoài bộ nhớ của trình điều khiển sở hữu (trong trường hợp này là vmscsi.sys). Trái lại, TDL3 ghi một khối mã nhỏ vào bộ nhớ của trình điều khiển sở hữu (vmscsi.sys trong trường hợp này), và sử dụng nó làm điểm nhảy để truy cập vào mã rootkit. Trong kịch bản này, các hàm IRP vẫn trỏ vào bên trong vmscsi.sys, khiến việc xác định xem trình điều khiển đã bị tấn công trở nên rất khó khăn. Bằng cách sử dụng cờ --verbose cho lệnh driverirp hoặc bằng cách sử dụng plugin volshell để disassemble hàm xử lý, bạn sẽ thấy nó chỉ chứa các mã sau đây: 

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234247.png)

Lệnh thực thi đầu tiên trỏ tới một con trỏ tại địa chỉ 0xffdf0308. Sau đó, bằng cách sử dụng lệnh JMP, CPU được chuyển hướng đến một địa chỉ nằm ở offset 0xFC từ con trỏ trong thanh ghi EAX. Bạn có thể dễ dàng theo dõi các bước nhảy này trong volshell như được thể hiện dưới đây:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234301.png)

Ở điểm này, bạn biết rằng mã thực tế của rootkit chiếm diện tích xung quanh địa chỉ 0x81926e31. Bất kể rootkit ẩn như thế nào, hãy nhớ rằng nó luôn phải hoạt động. Chức năng của rootkit này liên quan đến việc gắn kết IRP, và bằng cách theo dõi các gắn kết, bạn đã đi thẳng vào phần thân của module độc hại.

### High Value Targets

Đúng, trên một hệ thống thông thường có hàng trăm driver, vì vậy bạn không thể kiểm tra tất cả 28 con trỏ chức năng chính cho mỗi driver, đặc biệt nếu chúng sử dụng kỹ thuật gắn kết ẩn. Đề xuất của chúng tôi là bạn nên tập trung vào những mục tiêu có giá trị cao nhất. Ví dụ, kẻ tấn công sẽ quan tâm đến IRP_MJ_READ và IRP_MJ_WRITE của các driver hệ thống tập tin. Ngoài ra, họ sẽ quan tâm đến IRP_MJ_DEVICE_CONTROL cho các driver mạng như \Driver\Tcpip, \Driver\NDIS và \Driver\HTTP.

## Device Trees

Hệ điều hành Windows sử dụng một kiến trúc xếp lớp (hoặc lớp chồng) để xử lý các yêu cầu I/O. Nó có nghĩa là nhiều driver có thể xử lý cùng một IRP (yêu cầu I/O). Tiếp cận theo lớp này có những lợi ích của nó - cho phép sao lưu và mã hóa hệ thống tập tin một cách trong suốt (như EFS), cũng như khả năng cho các sản phẩm tường lửa lọc kết nối mạng. Tuy nhiên, nó cũng cung cấp một cách khác cho một driver độc hại để tương tác với dữ liệu mà nó không nên truy cập. Ví dụ, thay vì gắn kết chức năng IRP của một driver đích như đã mô tả trước đó, một rootkit có thể chèn hoặc kết nối vào lớp của thiết bị đích. Theo cách này, driver rootkit nhận được một bản sao của IRP, mà nó có thể ghi lại hoặc sửa đổi trước khi driver hợp lệ nhận nó. 

Hình 13-6 cho thấy một sơ đồ đơn giản về cách mà một rootkit có thể khai thác kiến trúc driver lớp chồng. Ý là driver độc hại đóng một vị trí trong lớp chồng để nó có thể "kiểm tra" hoạt động được yêu cầu bất kể yêu cầu đó bắt nguồn từ đâu. Trong trường hợp này, nó được gắn kết vào lớp chồng của driver ATA (atapi.sys) và lọc các yêu cầu ghi vào các sector cụ thể trên ổ cứng. Theo cách này, không quan trọng liệu một ứng dụng trong chế độ người dùng hay một driver antivirus trong chế độ kernel cố gắng xóa một tệp được bảo vệ; driver rootkit vẫn có cơ hội chặn hoặc loại bỏ yêu cầu đó.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234314.png)

**Cấu trúc dữ liệu**

Để thực hiện hành vi được mô tả, một driver trước tiên tạo một thiết bị có tên hoặc không có tên của một loại cụ thể (ví dụ, thiết bị hệ thống tập tin, thiết bị mạng) bằng cách gọi IoCreateDevice. Giá trị được trả về của hàm này trở thành đối tượng thiết bị nguồn (source device object). Sau đó, driver thu được một con trỏ đến đối tượng thiết bị đích (target device object) bằng cách sử dụng IoGetDeviceObjectPointer. Nó chuyển cả hai đối tượng thiết bị này cho IoAttachDeviceToDeviceStack, hoàn thành việc thiết lập. Bây giờ, thiết bị nguồn có thể nhận các yêu cầu IRP được dành cho thiết bị đích. Ngoài ra, IoAttachDevice cũng có thể được sử dụng tương tự. Đoạn mã sau đây thể hiện một đối tượng thiết bị cho Windows 7 64-bit:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234332.png)

**Điểm chính**

- DriverObject: Một con trỏ trở lại đối tượng trình điều khiển của thiết bị.
- NextDevice: Một danh sách liên kết một chiều của các thiết bị khác được tạo bởi cùng một trình điều khiển.
- AttachedDevice: Một danh sách liên kết một chiều của các thiết bị (thường được tạo bởi các trình điều khiển khác) được gắn kết vào ngăn xếp này.
- CurrentIrp: IRP hiện tại đang được xử lý bởi thiết bị này.
- DeviceExtension: Một thành viên không rõ (undefined) có thể lưu trữ bất kỳ loại cấu trúc dữ liệu tùy chỉnh và dữ liệu cấu hình nào mà một thiết bị yêu cầu. Ví dụ, trình điều khiển Truecrypt lưu trữ các khóa mã hóa chính trong phần mở rộng của thiết bị của nó.
- DeviceType: Xác định loại thiết bị; ví dụ: FILE_DEVICE_KEYBOARD, FILE_DEVICE_NETWORK, hoặc FILE_DEVICE_DISK.

### Auditing Device Trees

Để kiểm tra cây thiết bị, bạn có thể sử dụng plugin devicetree. Kết quả đầu ra của plugin này cho thấy các trình điều khiển (DRV) ở phần mép ngoài của cây và các thiết bị (DEV) của chúng được thụt lề một cấp độ. Bất kỳ thiết bị đính kèm (ATT) nào cũng được thụt lề tiếp theo. Khi phân tích kết quả, bạn nên tập trung vào các loại thiết bị quan trọng nhất (mạng, bàn phím và ổ cứng) vì đó là những mục tiêu thường xuyên bị tấn công bởi các kẻ tấn công.
Dưới đây là một ví dụ về bản ghi nhớ bị nhiễm rootkit thử nghiệm KLOG, nó gắn vào thiết bị bàn phím để nhận các bản sao của các phím được nhấn bởi người dùng:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234343.png)

KLOG tạo ra một trình điều khiển có tên là \Driver\klog, sau đó nó tạo một thiết bị không đặt tên, được chỉ định bởi dấu (?), có loại là FILE_DEVICE_KEYBOARD và gắn nó vào thiết bị KeyboardClass0 thuộc sở hữu của \Driver\Kbdclass. Bạn sẽ thấy một hiệu ứng tương tự nếu bạn cài đặt tiện ích Ctrl2cap từ SysInternals (http://technet.microsoft.com/en-us/sysinternals/bb897578.aspx) vì nó sử dụng cùng phương pháp trình điều khiển lớp để chuyển đổi ký tự caps-lock thành các ký tự điều khiển.

### Stuxnet’s Malicious Devices

Ví dụ tiếp theo cho thấy các sửa đổi được thực hiện vào hệ thống bởi trình điều khiển kernel của Stuxnet (\Driver\MRxNet):

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234400.png)

Các thiết bị không có tên được tạo bởi \Driver\MRxNet là thiết bị ở cạnh ngoài được gắn vào các trình điều khiển hệ thống tập tin vmhgfs (VMware Host to Guest File System), MRxSmb (SMB), Cdfs và Ntfs. Bây giờ Stuxnet có thể lọc hoặc ẩn các tệp và thư mục có tên cụ thể trên những hệ thống tập tin đó.

## Auditing the SSDT

Bảng mô tả dịch vụ hệ thống (SSDT) chứa các con trỏ trỏ tới các hàm ở chế độ kernel. Như được thể hiện trong Hình 13-7, khi các ứng dụng ở chế độ người dùng yêu cầu các dịch vụ hệ thống, như việc ghi vào một tệp hoặc tạo một quá trình, một đoạn mã nhỏ trong ntdll.dll (hoặc thư viện người dùng chế độ khác) hỗ trợ luồng gọi trong việc chuyển sang chế độ kernel một cách kiểm soát. Việc chuyển tiếp được thực hiện bằng cách sử dụng lệnh INT 0x2E trong Windows 2000 hoặc sử dụng SYSENTER trong XP và các phiên bản sau đó. Cả hai phương pháp đều đến hàm có tên KiSystemService, hàm này tìm địa chỉ của hàm kernel được yêu cầu trong SSDT. Việc tìm kiếm này dựa trên chỉ số vì bảng gọi là mảng các con trỏ.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234410.png)

Thứ tự và tổng số hàm trong SSDT khác nhau giữa các phiên bản hệ điều hành. Ví dụ, hàm NtUnloadDriver có thể được tìm thấy ở chỉ số 0x184 trên Windows 7 64-bit, nhưng ở chỉ số 0x1A1 trên Windows 8 64-bit. Lưu ý rằng có nhiều hơn một bảng gọi trên mỗi hệ thống. Bảng gọi đầu tiên và phổ biến nhất lưu trữ các hàm API cơ bản do module kernel executive cung cấp (ntoskrnl.exe, ntkrnlpa.exe, v.v.). Bảng gọi thứ hai, được gọi là shadow SSDT, lưu trữ các hàm giao diện đồ họa được cung cấp bởi win32k.sys. Như được thể hiện trong Hình 13-7, hai bảng gọi còn lại không được sử dụng theo mặc định, trừ khi bạn đang chạy một máy chủ IIS - trong trường hợp này bảng gọi thứ ba được sử dụng bởi spud.sys (driver dịch vụ IIS).

**Cấu trúc dữ liệu**


Mã sau đây hiển thị các cấu trúc dữ liệu liên quan cho một hệ thống Windows 7 64-bit. Các biểu tượng nt!KeServiceDescriptorTable và nt!KeServiceDescriptorTableShadow đều là các trường hợp của _SERVICE_DESCRIPTOR_TABLE chứa tối đa bốn mô tả (entries). Mỗi mô tả có thành viên KiServiceTable trỏ đến mảng các hàm, và ServiceLimit xác định số lượng hàm tồn tại trong mảng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234423.png)

### Enumerating the SSDT

Để liệt kê SSDT trong các bản sao bộ nhớ Windows, bạn có thể sử dụng tiện ích ssdt. Do sự thay đổi giữa phiên bản 32-bit và 64-bit, tiện ích này tìm dữ liệu SSDT theo cách hoàn toàn khác nhau, nhưng định dạng đầu ra vẫn giữ nguyên. Cụ thể, trên Windows 32-bit, chúng tôi liệt kê tất cả các đối tượng luồng (thread) và thu thập các giá trị duy nhất cho thành viên _ETHREAD.Tcb.ServiceTable. Thành viên này không tồn tại trên các nền tảng 64-bit, do đó thay vào đó, chúng tôi giải thể hàm nt!KeAddSystemServiceTable được xuất khẩu và trích xuất các địa chỉ ảo tương đối (RVAs) cho các ký hiệu KeServiceDescriptorTable và KeServiceDescriptorTableShadow, như thể hiện trong Hình 13-8.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234434.png)

Dưới đây là kết quả đầu ra của plugin ssdt trên một máy tính Windows 7 64-bit trong trạng thái hoạt động sạch sẽ:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234456.png)

Như bạn thấy, bảng tại địa chỉ 0xfffff800028dc300 là SSDT[0] hoặc mô tả đầu tiên trong mảng _SERVICE_DESCRIPTOR_TABLE.Descriptors. Nói cách khác, bảng này chứa các API native được xuất bởi module NT. Bảng tại địa chỉ 0xfffff960001a1f00 là SSDT[1] (mô tả thứ hai), cho thấy đó là bảng chứa các API của hệ thống GUI. Tất cả các hàm hiển thị đều thuộc sở hữu của module phù hợp (hoặc là module NT hoặc win32k.sys).

### Attacking the SSDT

Dưới đây là một số cách tấn công kiến trúc xử lý cuộc gọi hệ thống. Chúng tôi liệt kê các phương pháp và mô tả cách bạn có thể sử dụng phân tích bộ nhớ để phát hiện các cuộc tấn công:

#### Pointer Replacement

Phương pháp này liên quan đến việc ghi đè các con trỏ trong SSDT để thay thế các hàm cụ thể. Để làm điều này, bạn thường cần địa chỉ cơ sở của bảng cuộc gọi trong bộ nhớ kernel và chỉ mục của hàm bạn muốn hook. Có một số cách để tìm bảng cuộc gọi, nhưng phần mềm độc hại thường sử dụng MmGetSystemRoutineAddress (phiên bản kernel của GetProcAddress) và tìm symbol KeServiceDescriptorTable, được xuất khẩu bởi module NT. Sau đó, nó tham chiếu đến thành viên ServiceTable. Bạn thường thấy API InterlockedExchange được sử dụng để thực hiện việc thay thế con trỏ thực tế. 

Tất cả các địa chỉ trong bảng hàm native phải trỏ vào bên trong module NT, và tất cả các địa chỉ trong bảng hàm GUI phải trỏ vào bên trong win32k.sys. Phát hiện SSDT hooks đơn giản trong trường hợp này vì bạn chỉ cần kiểm tra từng mục và xác định liệu chúng có trỏ vào module đúng không. Dưới đây là một mẫu mã độc hại mà hook các hàm khác nhau và trỏ chúng vào module có tên là lanmandrv.sys. Bạn có thể lọc kết quả bằng cách sử dụng egrep -v để loại bỏ các module chính thống: 

>**LƯU Ý:**<br> Hãy nhớ rằng tên của module NT có thể không luôn luôn là ntoskrnl.exe. Nó có thể là ntkrnlpa.exe hoặc ntkrnlmp.exe, vì vậy hãy đảm bảo điều chỉnh biểu thức chính quy của bạn tương ứng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234514.png)

Chương trình gốc đã bẻ khóa bốn hàm: NtEnumerateValueKey để ẩn các giá trị registry, NtOpenProcess và NtQuerySystemInformation để ẩn các tiến trình hoạt động, và NtQueryDirectoryFile để ẩn các tập tin trên đĩa. Mặc dù tên có vẻ như mang tính lừa đảo (lanmandrv.sys nghe có thể là một thành phần hợp lệ), nó nổi bật vì nó không nên xử lý các API thường được triển khai bởi module NT.

#### Inline Hooking

Những kẻ tấn công rõ ràng nhận thức được các phương pháp sử dụng để phát hiện các sửa đổi mà các công cụ của họ gây ra trên hệ thống. Do đó, thay vì trỏ các chức năng SSDT ra ngoài module NT hoặc win32ks.sys, họ có thể sử dụng một kỹ thuật hooking ngay lập tức (inline hooking). Kỹ thuật này có cùng hiệu quả chuyển hướng thực thi đến một chức năng độc hại, nhưng không rõ ràng như cách trên. Dưới đây là một ví dụ về cách nó xuất hiện khi rootkit Skynet hook NtEnumerateKey (chúng tôi đã thêm cờ --verbose để kiểm tra các hooking inline này):

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234526.png)

Đúng, con trỏ 0x80570d64 thực sự thuộc về ntoskrnl.exe, nhưng các chỉ thị tại địa chỉ đó đã bị ghi đè bằng một JMP đưa đến 0x820f1b3c. Do đó, nếu bạn chỉ kiểm tra module sở hữu ban đầu, bạn sẽ bỏ qua việc rootkit này hook SSDT.

#### Table Duplication


Mỗi luồng trên hệ thống 32-bit có thành viên _ETHREAD.Tcb.ServiceTable để xác định bảng SSDT mà nó sử dụng. Mặc dù khả năng gán bảng gọi trên một cơ sở mỗi luồng không áp dụng cho hệ thống 64-bit, điều quan trọng là mỗi luồng có thể "nhìn" vào một bảng SSDT khác nhau, tùy thuộc vào giá trị của thành viên ServiceTable của nó. Trong trường hợp này, mã độc có thể tạo một bản sao của bảng hàm native, hook một số hàm và sau đó cập nhật giá trị ServiceTable cho một luồng cụ thể hoặc tất cả luồng trong một quá trình cụ thể để trỏ đến bản sao mới. Kết quả là nhiều công cụ không báo cáo được SSDT hook vì chúng chỉ kiểm tra bảng gốc, không kiểm tra các bản sao.

Dưới đây là một ví dụ về cách đầu ra của plugin ssdt xuất hiện khi phân tích một bộ nhớ bị nhiễm Blackenergy. Chỉ các dòng liên quan được hiển thị:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234537.png)

Lưu ý rằng có ba phiên bản khác nhau của SSDT[0], trong khi hệ thống thông thường chỉ có một phiên bản. Bạn có thể nhận biết bảng tại địa chỉ 0x80501030 là bản gốc trong sạch vì NtWriteVirtualMemory trỏ đến NT module. Tuy nhiên, cả hai bảng tại địa chỉ 0x814561b0 và 0x81882980 đã bị hook - phiên bản của NtWriteVirtualMemory trong bảng này đang trỏ đến một module có tên là 00000B9D.

### SSDT Hook Disadvantages

Ràng buộc (hook) các chức năng trong SSDT có thể cung cấp nhiều khả năng, nhưng cũng có thể không ổn định. Dưới đây là một số lý do tại sao các tác giả malware có thể sử dụng các kỹ thuật khác trong tương lai:

- Patchguard: Việc hook SSDT bị ngăn chặn trên hệ thống 64-bit do có Kernel Patch Protection (KPP), còn được gọi là Patchguard.

- Nhiều lõi: Bảng các hàm gọi hệ thống không phải là một cấu trúc riêng biệt cho mỗi CPU. Do đó, trong khi một lõi đang cố gắng áp dụng hook, một lõi khác có thể đang cố gắng gọi các API.

- Các bản sao: Nếu cho phép các driver bên thứ ba hook các entry trong SSDT, nhiều driver có thể cố gắng hook vào cùng một chức năng. Hậu quả của việc các driver thay đổi hook có thể không thể đoán trước.

- Các API không được tài liệu: Nhiều hàm trong SSDT không được tài liệu bởi Microsoft và có thể thay đổi qua các phiên bản của Windows. Do đó, việc viết một rootkit có thể di động và đáng tin cậy có thể gặp khó khăn.

## Kernel Callbacks

Các hàm gọi lại (kernel callbacks), hay còn gọi là các hàm thông báo (notification routines), là các phương pháp hook API mới. Chúng giải quyết nhiều vấn đề đã được mô tả trước đó liên quan đến hook SSDT. Đặc biệt, chúng được tài liệu, hỗ trợ trên hệ thống 64-bit và an toàn trên máy đa lõi; và hoàn toàn có thể cho nhiều module đăng ký cho cùng một loại sự kiện. Dưới đây là danh sách các loại sự kiện khác nhau mà plugin callbacks của Volatility phát hiện:

- Tạo quá trình (Process creation): Các callback này được cài đặt bằng API PsSetCreateProcessNotifyRoutine và chúng được sử dụng bởi tiện ích Process Monitor từ SysInternals, các sản phẩm antivirus và nhiều rootkit khác. Chúng được kích hoạt khi một quá trình bắt đầu hoặc kết thúc.

- Tạo luồng (Thread creation): Các callback này được cài đặt bằng API PsSetCreateThreadNotifyRoutine. Chúng được kích hoạt khi một luồng bắt đầu hoặc kết thúc.

- Tải hình ảnh (Image load): Các callback này được cài đặt bằng API PsSetLoadImageNotifyRoutine. Mục đích của các callback này là cung cấp thông báo khi bất kỳ hình ảnh thực thi nào được ánh xạ vào bộ nhớ, chẳng hạn như một quá trình, thư viện hoặc module kernel.

- Tắt hệ thống (System shutdown): Các callback này được cài đặt bằng API IoRegisterShutdownNotification. Trong trường hợp này, IRP_MJ_SHUTDOWN của driver đích được gọi khi hệ thống chuẩn bị tắt nguồn.

- Đăng ký hệ thống tập tin (File system registration): Để nhận thông báo khi một hệ thống tập tin mới được sẵn có, sử dụng API IoRegisterFsRegistrationChange.

- Tin nhắn gỡ lỗi (Debug message): Để chụp các thông báo gỡ lỗi được phát ra bởi các module kernel, sử dụng API DbgSetDebugPrintCallback.

- Thay đổi registry (Registry modification): Các driver có thể gọi CmRegisterCallback (Windows XP và 2003) hoặc CmRegisterCallbackEx (Windows Vista và phiên bản sau) để nhận thông báo khi bất kỳ luồng nào thực hiện một hoạt động trên registry.

- PnP (Plug and Play): Các callback này được cài đặt bằng API IoRegisterPlugPlayNotification và chúng được kích hoạt khi các thiết bị PnP được giới thiệu, gỡ bỏ hoặc thay đổi.

- Bugcheck: Các callback này được cài đặt bằng API KeRegisterBugCheckCallback hoặc KeRegisterBugCheckReasonCallback. Chúng cho phép driver nhận thông báo khi có một bug check (ngoại lệ không được xử lý), do đó cung cấp cơ hội để đặt lại cấu hình thiết bị hoặc thêm thông tin trạng thái cụ thể của thiết bị vào tệp crash dump (trước khi xuất hiện màn hình Blue Screen of Death [BSoD], chẳng hạn).

### Callbacks in Memory

The callbacks plugin được trình bày dưới đây cho một hệ thống Windows 7 64-bit sạch. Mặc dù đầu ra đã bị cắt ngắn để tóm gọn, nhưng có khoảng 80 callback của các loại khác nhau được cài đặt trên hệ thống này. Cột Callback cho biết địa chỉ của hàm được gọi khi sự kiện mong muốn xảy ra. Cột Module cho biết tên của module kernel mà chiếm bộ nhớ cho hàm callback. Tùy thuộc vào loại callback, bạn cũng có thể thấy tên của đối tượng driver hoặc mô tả về thành phần đã cài đặt callback.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234551.png)

### Malicious Callbacks

Nhiều rootkit nổi tiếng như Mebroot, ZeroAccess, Rustock, Ascesso, Tigger, Stuxnet, Blackenergy và TDL3 sử dụng các kernel callbacks. Trong hầu hết các trường hợp, chúng cũng cố gắng giấu đi bằng cách mất liên kết với KLDR_DATA_TABLE_ENTRY hoặc chạy như một thread mồ côi từ một kernel pool. Hành vi này làm cho các gọi lại độc hại dễ dàng phát hiện vì cột Module trong đầu ra của plugin callbacks của Volatility hiển thị UNKNOWN. Trong các trường hợp khác, tác giả của malware không giấu mô-đun của họ hoàn toàn, nhưng họ sử dụng một tên được cài đặt cứng (và do đó dễ dự đoán) mà bạn có thể sử dụng để xây dựng các chỉ số của sự xâm phạm (IOCs).

Ví dụ đầu tiên là từ Stuxnet. Nó tải hai mô-đun: mrxnet.sys và mrxcls.sys. Mô-đun đầu tiên cài đặt một gọi lại thay đổi đăng ký hệ thống tệp để nhận thông báo khi các hệ thống tệp mới xuất hiện (vì vậy nó có thể lan truyền hoặc ẩn các tệp). Mô-đun thứ hai cài đặt một gọi lại tải hình ảnh, mà nó sử dụng để tiêm mã vào quy trình khi họ cố gắng tải các thư viện liên kết động (DLLs) khác. Kỹ thuật này cho phép Stuxnet tiêm mã độc hại của nó vào các quy trình hợp pháp và thực thi trong bối cảnh của các quy trình này, giấu đi sự hiện diện của nó.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234608.png)

Ví dụ tiếp theo đến từ Rustock.C. Nó đăng ký một gọi lại bug check để dọn dẹp bộ nhớ của nó trước khi tạo một bản sao lưu crash dump (xem báo cáo của Frank Boldewin tại đây: http://www.reconstructer.org/papers/Rustock.C%20-%20When%20a%20myth%20comes%20true.pdf). Lý do duy nhất tại sao bạn thấy các hiện tượng của nó ở đây trong bản nhớ là do bộ nhớ đã được thu thập ở định dạng nguyên thô thay vì thông qua các cơ chế bình thường của hệ điều hành.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234620.png)

Dưới đây là một ví dụ cho thấy callback thay đổi registry được cài đặt bởi Ascesso. Rootkit này sử dụng chức năng này để theo dõi các khóa ghi nhớ của nó trong registry và thêm chúng trở lại nếu một quản trị viên hoặc phần mềm diệt virus loại bỏ chúng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234633.png)

Rootkit Blackenergy cài đặt một callback cho việc tạo luồng, từ đó nó có thể ngay lập tức thay thế con trỏ _ETHREAD.Tcb.ServiceTable trên tất cả các luồng bắt đầu trên hệ thống. Như đã thảo luận trong phần "Bảng Bản sao" của chương này, trên hệ thống 32-bit, thành viên ServiceTable trỏ đến bảng gọi hệ thống trong đó có các địa chỉ của tất cả các API ở chế độ kernel.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234645.png)

Tìm kiếm và phân tích các callback là một thành phần quan trọng trong việc khám phá bộ nhớ kernel. Điều đáng ngạc nhiên là không có công cụ quản trị hệ thống và rất ít công cụ chống rootkit cho các hệ thống hoạt động trực tiếp mà có khả năng phân tích các callback kernel (RkU - Rootkit Unhooker là một trong số đó). Thậm chí, bộ gỡ lỗi của Microsoft cũng không có khả năng này mặc định. Tuy nhiên, Scott Noone (http://analyze-v.com/?p=746) và Matthieu Suiche (http://www.moonsols.com/2011/02/17/global-windows-callbacks-and-windbg/) đã xuất bản các kịch bản để giúp điền vào khoảng trống đó.

## Kernel Timers

Thường thì, phần mềm độc hại sử dụng bộ định thời (timer) để đồng bộ hóa và thông báo. Một driver rootkit có thể tạo một bộ định thời (thường bằng cách gọi KeInitializeTimer) để nhận thông báo khi một thời gian nhất định trôi qua. Nếu bạn nghĩ rằng điều này tương tự như việc gọi Sleep, thì bạn đúng. Tuy nhiên, việc gọi Sleep khiến một luồng ngừng hoạt động và ngăn nó thực hiện các hoạt động khác trong khi đợi, không giống như thông báo dựa trên bộ định thời. Hơn nữa, Sleep không tạo ra bất kỳ hệ quả pháp lý gì. Bạn cũng có thể tạo bộ định thời mà sau khi hết hạn sẽ tự động thiết lập lại. Nói cách khác, thay vì chỉ nhận thông báo một lần, một luồng có thể nhận thông báo định kỳ. Có thể driver rootkit muốn kiểm tra xem tên DNS có giải quyết thành công hay không mỗi năm năm phút, hoặc theo dõi thay đổi của một khóa registry cụ thể mỗi hai giây. Bộ định thời rất hữu ích cho những loại công việc như thế này.

Khi các driver tạo bộ định thời, họ có thể cung cấp một DPC routine, còn được gọi là deferred procedure call. Khi bộ định thời hết hạn, hệ thống sẽ gọi thủ tục được chỉ định. Địa chỉ của thủ tục hoặc hàm này được lưu trong cấu trúc _KTIMER, cùng với thông tin về khi (và thường xuyên) thực hiện thủ tục. Và bây giờ bạn thấy tại sao bộ định thời kernel là các tác nhân hữu ích như thế nào cho pháp y bộ nhớ. Rootkit tải các driver vào bộ nhớ kernel và cố gắng giữ cho nó không bị phát hiện. Nhưng việc sử dụng bộ định thời cho bạn một chỉ báo rõ ràng về nơi rootkit ẩn trong bộ nhớ. Bạn chỉ cần tìm các đối tượng bộ định thời.

### Finding Timer Objects

The way Microsoft stores and manages timers in memory has evolved over the years. In Windows 2000, the symbol nt!KiTimerTableListHead pointed to an array of 128 _LIST_ENTRY structures for _KTIMER. The size of this array later changed to 256 and then to 512 in subsequent versions, until it was ultimately removed entirely in Windows 7. Nowadays, timer objects can be found through each CPU's control region (_KPCR) structure. These changes have been well-documented, and you can find more information about them in the blog post titled "Ain’t Nuthin But a K(Timer) Thing, Baby" at http://mnin.blogspot.com/2011/10/aint-nuthin-butktimerthing-baby.html.

### Malware Analysis with Timers

Ví dụ dưới đây cho thấy cách điều tra rootkit ZeroAccess bằng cách sử dụng plugin timers. Rootkit này sử dụng nhiều kỹ thuật chống pháp truy lùng để làm cho module của nó khó bị phát hiện. Tuy nhiên, điều này cũng có nghĩa là một timer trỏ vào một vùng không xác định của bộ nhớ kernel.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234709.png)

>**LƯU Ý**<br>
Trong bài viết "Timers and times" của Andreas Schuster (http://computer.forensikblog.de/en/2011/10/ timers-and-times.html), hướng dẫn cách chuyển đổi trường DueTime thành các giá trị dễ đọc được bằng cách sử dụng WinDbg.

Ngoài ra, cùng phiên bản Rustock.C mà bạn đã phân tích trong phần "Malicious Callbacks" cũng đã cài đặt một số timers. Nó cũng cố gắng che giấu module kernel của mình, do đó để lại dấu vết của hoạt động đáng ngờ dễ dàng nhìn thấy bằng plugin timers.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234721.png)

Như đã nêu trong phân tích trước đó, mặc dù bạn không biết tên của module độc hại trong những trường hợp này, nhưng bạn ít nhất đã có các con trỏ trỏ đến nơi mã rootkit tồn tại trong bộ nhớ kernel. Sau đó, bạn có thể dùng volshell để phân tích mã hoặc trích xuất mã ra một tập tin riêng biệt để phân tích tĩnh trong IDA Pro hoặc các framework khác.

>Lưu ý rằng trên các nền tảng 64-bit, một trong các tính năng liên quan đến Patchguard dẫn đến việc mã hóa địa chỉ DPC. Hệ điều hành giải mã chúng vào thời gian chạy bằng cách sử dụng thuật toán tương tự như đã mô tả trong bài viết "The Secret to Windows 8 and 2012 Raw Memory Dump Forensics" (http://volatility-labs.blogspot.com/2014/01/the-secret-to-64-bit-windows-8-and-2012.html). Volatility đã tích hợp tính năng này và có thể thực hiện việc giải mã DPC trực tiếp ngay lập tức.

## Putting It All Together

Bây giờ khi bạn đã được tiếp xúc với các phương pháp khác nhau để tìm hiểu và phân tích mã độc trong nhân hệ điều hành, chúng tôi sẽ chỉ bạn một ví dụ về cách kết hợp tất cả các yếu tố lại với nhau. Trong trường hợp này, chúng tôi nhận thấy sự hiện diện của rootkit thông qua việc sử dụng các bộ hẹn giờ và callback mà trỏ vào vùng nhớ không thuộc sở hữu của bất kỳ module nào trong danh sách các module đã nạp. Dưới đây là kết quả đầu ra liên quan từ hai plugin đã đề cập:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234731.png)

Một thủ tục tại địa chỉ 0x81b99db0 được thiết lập để thực thi mỗi 60.000 mili giây, một hàm tại địa chỉ 0x81b934e0 được thiết lập để gọi khi hệ thống tắt, và một hàm tại địa chỉ 0x81b92d60 được thông báo về tất cả các thao tác trên registry. Rootkit này đã rõ ràng "trồng một số hạt" vào nhân của hệ thống nạn nhân. Tại thời điểm này, bạn chưa biết tên của module này, nhưng bạn có thể thấy rằng shutdown callback được liên kết với một trình điều khiển có tên \Driver\03621276. Dựa vào thông tin này, bạn có thể tìm thêm thông tin chi tiết với plugin driverscan.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234743.png)

Theo kết quả xuất hiện trong phân tích, địa chỉ bắt đầu của module kernel tạo ra đối tượng trình điều khiển đáng ngờ là 0. Điều này có thể là một kỹ thuật chống phân tích (anti-forensics) để ngăn các nhà phân tích đọc mã độc. Thật vậy, cho đến thời điểm này, để trích xuất module, bạn cần tên hoặc địa chỉ cơ sở của module, nhưng bạn đã biết rằng tên không có sẵn. Tuy nhiên, có nhiều con trỏ trong mã module độc hại; bạn chỉ cần tìm hiểu nơi PE file bắt đầu. Bạn có thể làm điều này bằng một số mã trong volshell, sử dụng một trong những kỹ thuật sau đây:
- Lấy một trong các địa chỉ và quét ngược để tìm chữ ký MZ hợp lệ. Nếu PE file độc hại có nhiều tệp nhúng, kết quả đầu tiên có thể không đúng.
- Đặt địa chỉ bắt đầu ở một nơi nào đó giữa 20KB và 1MB sau địa chỉ con trỏ thấp nhất bạn có; sau đó tiến về phía trước để tìm chữ ký MZ hợp lệ.

Dưới đây là mã để thực hiện phương pháp thứ hai:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234757.png)

>**Lưu ý:**<br>
Hoặc bạn có thể chuyển đổi địa chỉ ảo thành độ lệch vật lý bằng cách gọi hàm addrspace().vtop(ĐỊA_CHỈ). Miễn là bạn có một bản sao bộ nhớ gốc được đệm, bạn có thể mở nó trong một trình biên tập hex và tìm kiếm đến độ lệch vật lý, sau đó cuộn lên để tìm chữ ký MZ.

Bạn đã tìm thấy một chữ ký MZ tại địa chỉ 0x81b91b80, khoảng cách 8KB trên các thủ tục về timers và callbacks. Bạn cũng có thể xác minh phần đầu PE trong volshell:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234808.png)

Cuối cùng, bạn có thể cung cấp một địa chỉ cơ sở cho plugin moddump và trích xuất module từ bộ nhớ:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234818.png)

Việc cuối cùng bạn phải làm trước khi tải tệp vào IDA Pro là tạo nhãn cho các hàm API. Thông thường, IDA có thể phân tích bảng địa chỉ nhập và hiển thị tên API đúng cách, nhưng nó không mong đợi nhận các tệp được trích xuất từ bộ nhớ sau khi bảng địa chỉ nhập (IAT) đã bị thay đổi. Trong các trường hợp như vậy, bạn có thể chạy plugin impscan với địa chỉ cơ sở của module nghi ngờ và đối số dòng lệnh cho định dạng đầu ra idc như sau:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234830.png)

Sau khi mở tệp module đã trích xuất trong IDA Pro, hãy đi đến File ➪ Script Command và dán kết quả từ plugin impscan vào cửa sổ. Sau khi thực hiện các bước này, bạn sẽ có một tệp nhị phân được xây dựng lại đúng đắn, với các tham chiếu chuỗi chính xác và tên hàm API, như được hiển thị trong Hình 13-9.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20234844.png)

Tùy vào mục tiêu của bạn, bạn có thể không luôn cần đào sâu vào chi tiết như vậy. Chúng tôi thường cố gắng xác định càng nhiều thông tin càng tốt về hành vi của một rootkit dựa trên các đặc điểm mà nó để lại trong bộ nhớ. Tuy nhiên, trong một số trường hợp, bạn cần phải thực hiện kỹ thuật đảo ngược để hiểu đầy đủ mã nguồn - điều này không thể tránh được. Bây giờ bạn đã biết cách tiếp cận những tình huống đó bằng cách kết hợp phân tích bộ nhớ với các công cụ phân tích tĩnh.

Lưu ý: Chỉ vài tháng sau khi tham gia khóa học của chúng tôi, một trong số các học viên cũ đã giải phân tích rootkit Uroburos trong bộ nhớ một cách thành thạo. Bạn có thể đọc phân tích tại đây: [link](http://spresec.blogspot.com/2014/03/uroburos-rootkit-hook-analysis-and.html)

## Tổng quan 

Kernel land là một khía cạnh hấp dẫn và rộng lớn của phân tích bộ nhớ. Có vô số cách để ẩn mã trong kernel, thay đổi hành vi của hệ điều hành, và nhiều hơn thế nữa. Hơn nữa, nhiều nhà phân tích chưa quen với lãnh địa này, dẫn đến giảm khả năng tìm dấu vết. Tuy nhiên, bây giờ bạn đã được tiếp xúc với các phương pháp phổ biến nhất và đã thấy các ví dụ thực tế về việc phát hiện rootkit nổi tiếng sử dụng phân tích bộ nhớ. Thông thường, phần mềm độc hại hoạt động trong kernel sẽ ở lại trong bộ nhớ để duy trì tính năng. Thường, yêu cầu chức năng liên quan đến việc sửa đổi bảng gọi, cài đặt callback, hoặc tạo luồng mới - tất cả những thao tác này để lại các dấu vết như một dấu chân để bạn khôi phục. Khi bạn tìm thấy khoảng bộ nhớ mà mã độc chiếm giữ và trích xuất nó, phần còn lại là lịch sử!

