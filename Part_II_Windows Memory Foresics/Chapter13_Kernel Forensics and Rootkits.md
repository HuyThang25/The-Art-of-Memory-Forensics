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
