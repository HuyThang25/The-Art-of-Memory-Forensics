![image](https://github.com/HuyThang25/The-Art-of-Memory-Forensics/assets/93728466/fe5648aa-46f7-4ceb-b7a8-0b012019612e)Bản đăng ký chứa các cài đặt và cấu hình khác nhau cho hệ điều hành Windows, các ứng dụng và người dùng trên máy tính. Là một thành phần cốt lõi của máy tính Windows, nó được truy cập liên tục trong quá trình chạy. Do đó, hệ thống lưu cache toàn bộ hoặc một phần các tệp đăng ký trong bộ nhớ. Hơn nữa, bản đăng ký Windows chứa rất nhiều thông tin hữu ích cho mục đích pháp y. Ví dụ, bạn có thể sử dụng nó để xác định các chương trình đã chạy gần đây, trích xuất các băm mật khẩu cho mục đích kiểm toán, hoặc điều tra các khóa và giá trị mà mã độc đã giới thiệu vào hệ thống. 

Trong chương này, bạn sẽ tìm hiểu cách tìm và truy cập các tệp đăng ký trong bộ nhớ thông qua việc đi qua các ví dụ về một số tình huống đã đề cập ở trên. Hơn nữa, bạn sẽ được làm quen với sự khác biệt giữa dữ liệu đăng ký ổn định và dữ liệu đăng ký không ổn định và cách xem xét các hive (nguồn dữ liệu) trong bộ nhớ có thể mở ra một lĩnh vực phân tích hoàn toàn mới mà không thể thực hiện được với pháp y đĩa.

## Windows Registry Analysis

Nghiên cứu ban đầu về việc truy cập các tệp đăng ký trong bộ nhớ được thực hiện bởi Brendan Dolan-Gavitt vào năm 2008. Bài báo của ông có tiêu đề "Phân tích pháp y của bản đăng ký Windows trong bộ nhớ" (dfrws.org/2008/proceedings/p26-dolan-gavitt.pdf) và mã nguồn ban đầu của ông đã cung cấp nền tảng nghiên cứu tiên phong cho tất cả các khả năng hiện tại của Volatility liên quan đến bản đăng ký.

**Mục tiêu**

- Xác định vị trí các tệp đăng ký: Hiểu cách Volatility xác định vị trí các tệp đăng ký một cách nhất quán trong các bộ nhớ bị đổ của các phiên bản hệ điều hành khác nhau.

- Phân tích dữ liệu đăng ký: Học cách chuyển đổi địa chỉ để tìm kiếm các khóa và in ra các khóa con, giá trị và dữ liệu của chúng.

- Khám phá các khóa đăng ký liên quan đến pháp y: Tìm và trích xuất các khóa đăng ký liên quan đến pháp y, chẳng hạn như những khóa được mã độc sử dụng để duy trì tính bền vững.

- Tìm hiểu về các khóa đặc biệt của bản đăng ký: Userassist, Shimcache và Shellbags là các khóa đăng ký chứa dữ liệu nhị phân yêu cầu xử lý bổ sung. Bạn sẽ học cấu trúc cho những khóa này cũng như khi và cách sử dụng chúng trong một cuộc điều tra.

**Cấu trúc dữ liệu**

Đầu ra sau đây hiển thị một số cấu trúc đã chọn từ Windows 7 32-bit. Cấu trúc _CMHIVE đại diện cho một tệp hive của bản đăng ký trên đĩa, và _HHIVE (tiêu đề hive) giúp mô tả nội dung và trạng thái hiện tại của hive.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20194353.png)

**Những điểm quan trọng:**

- Hive: Tiêu đề hive đăng ký chứa một chữ ký cũng như một cấu trúc được sử dụng cho việc dịch địa chỉ.

- HiveList: Một danh sách liên kết kép tới các cấu trúc _CMHIVE khác.

- FileFullPath: Đường dẫn của thiết bị kernel (ví dụ: \Device\HarddiskVolume1\WINDOWS\system32\config\software) tới hive đăng ký. Thành viên này không được sử dụng trong Windows 7, mặc dù nó vẫn tồn tại trong cấu trúc (xem http://gleeda.blogspot.com/2011/04/windows-registry-paths.html).

- FileUserName: Đường dẫn tới hive đăng ký trên đĩa, được đặt trước bởi SystemRoot hoặc \??\C:\ (ngoại trừ hive BCD sử dụng đường dẫn thiết bị kernel và hive HARDWARE sử dụng "Registry" path như đã mô tả cho thành viên HiveRootPath). Một số hive không sử dụng thành viên này trong Windows Vista/2008 và 7.

- HiveRootPath: Giới thiệu trong Windows Vista, thành viên này chứa đường dẫn "Registry" (ví dụ: \REGISTRY\MACHINE\SOFTWARE).

- Signature: Chữ ký của tệp đăng ký. Tệp đăng ký hợp lệ có chữ ký là 0xbee0bee0.

- BaseBlock: Được sử dụng để tìm khóa gốc (khóa đầu tiên) của đăng ký.

- Storage: Ánh xạ không gian địa chỉ ảo cho các khóa trong đăng ký.

### Data in the Registry

Để xem có bao nhiêu thông tin có thể thu được từ bản đăng ký trong bộ nhớ trên hệ thống đang chạy, Brendan Dolan-Gavitt đã tiến hành các thí nghiệm trên các máy XP 32-bit ở các trạng thái khác nhau. Ông kết luận rằng 98% dữ liệu hive có thể khôi phục được trên các hệ thống ít được sử dụng và khoảng 50% dữ liệu hive có thể khôi phục được trên các hệ thống được sử dụng nhiều. Như với hầu hết các đối tượng trong bộ nhớ, chiến lược "sử dụng hoặc mất" có hiệu lực. Do đó, các khóa và dữ liệu không được truy cập thường xuyên (hoặc gần đây) có thể được tráo đổi vào đĩa. Điều này quan trọng khi bạn phân tích các bản đăng ký trong bộ nhớ bị đổ. Ví dụ, sự hiện diện của một khóa trong bộ nhớ là bằng chứng cho việc khóa đó tồn tại trên máy tính vào thời điểm thu thập. Tuy nhiên, sự vắn mất một khóa không nhất thiết có nghĩa là khóa đó không tồn tại - nó có thể bị thiếu do trang hoặc nó có thể không được đọc vào bộ nhớ từ đầu. 

Từ quan điểm pháp y, bạn có thể tìm thấy rất nhiều thông tin trong bản đăng ký. Danh sách sau tóm tắt một số khả năng:

- Các chương trình tự động khởi chạy: Xác định các ứng dụng chạy tự động khi hệ thống khởi động hoặc người dùng đăng nhập.

- Phần cứng: Liệt kê các thiết bị truyền thông ngoại vi đã được kết nối vào hệ thống.

- Thông tin tài khoản người dùng: Kiểm tra mật khẩu người dùng, tài khoản, các mục đã được sử dụng gần đây (MRU) và sở thích của người dùng.

- Các chương trình chạy gần đây: Xác định các ứng dụng đã thực thi gần đây (sử dụng dữ liệu từ các khóa Userassist, Shimcache và MUICache).

- Thông tin hệ thống: Xác định các thiết lập hệ thống, phần mềm đã cài đặt và các bản vá bảo mật đã được áp dụng.

- Cấu hình mã độc: Trích xuất dữ liệu liên quan đến các trang web điều khiển và điều khiển của mã độc, đường dẫn đến các tệp bị nhiễm trên đĩa và khóa mã hóa (bất cứ điều gì mã độc viết vào đăng ký).

### Stable and Volatile Data

Ngoài các mục đã đề cập thông thường được sử dụng trong pháp y đĩa, còn tồn tại các khóa và hive đăng ký dễ bay hơi chỉ có trong bộ nhớ. Jamie Levy đã tiến hành nghiên cứu cho cuộc trình bày của cô, "Time is on My Side" (http://gleeda.blogspot.com/2011/08/volatility-20-and-omfw.html), cho thấy rằng có khá nhiều thông tin chỉ được lưu trữ trong bộ nhớ. Ví dụ, bạn có thể tìm thấy dữ liệu về các ổ đĩa, thiết bị và các thiết lập. Chỉ trong hive SYSTEM và hive NTUSER.DAT của một người dùng, chúng tôi đã đếm được hơn 400 khóa dễ bay hơi.

Có một mối quan hệ chặt chẽ giữa các khóa ổn định được tìm thấy trong bản đăng ký trên đĩa và các khóa trong bộ nhớ. Khi máy tính hoạt động, các khóa mới được tạo ra, và các khóa hiện có có thể thay đổi. Điều này làm cho việc các sửa đổi này được lưu trở lại đĩa tại một thời điểm nào đó. Mark Russinovich đã chứng minh trong "Microsoft Windows Internals, 6th Edition," rằng dữ liệu được đẩy trở lại đĩa khoảng mỗi năm một lần nếu sử dụng các API Windows (ví dụ: RegCreateKeyEx, RegSetValueEx). Brendan đã chỉ ra trong bài báo của ông rằng nếu bản đăng ký được điều chỉnh trực tiếp trong bộ nhớ mà không sử dụng các API Windows, các thay đổi không bị đẩy trở lại đĩa hoàn toàn. Trong quá trình nghiên cứu của mình, ông đã chứng minh điều này bằng cách thực hiện các bước sau:

1. Tìm băm mật khẩu của quản trị viên trong bộ nhớ.
2. Sửa đổi trực tiếp bộ nhớ để thay đổi giá trị thành băm mật khẩu cho một mật khẩu đã biết.
3. Đăng xuất khỏi hệ thống (để phân hệ LSA nhận thấy sự thay đổi và cập nhật).
4. Đăng nhập lại bằng mật khẩu mới.

Bởi vì những thay đổi sẽ không bao giờ được đẩy trở lại đĩa, bạn sẽ không biết rằng loại cuộc tấn công này đã xảy ra chỉ bằng việc thực hiện pháp y đĩa. Tuy nhiên, với một mẫu bộ nhớ, việc phát hiện loại tấn công này rất dễ dàng bằng cách trích xuất các băm mật khẩu từ hive đăng ký và so sánh chúng với những băm trên đĩa.

### Finding Registry Hives

Volatility tìm các hive đăng ký trong bộ nhớ bằng cách sử dụng phương pháp quét pool (xem Chương 5). Cấu trúc _CMHIVE được cấp phát trong một pool với tag CM10. Sau khi tìm thấy một phân bổ như vậy, bạn có thể xác minh rằng có một hive hợp lệ bằng cách kiểm tra thành viên Signature (_CMHIVE.Hive.Signature). Tại thời điểm này, bạn có thể sử dụng thành viên HiveList (_CMHIVE.HiveList) để xác định tất cả các hive khác. Cấu trúc _CMHIVE được hiển thị trong Hình 10-1.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200707.png)

Plugin hivelist quét các hive đăng ký và sau đó in ra thông tin về vị trí vật lý và ảo của chúng cùng với đường dẫn. Dưới đây là một ví dụ:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200723.png)

>**LƯU Ý:**<br> Chú ý rằng ngoài các bản đăng ký phổ biến (SAM, SYSTEM, NTUSER.DAT), còn có một bản đăng ký có tên là [no name]. Bản đăng ký này chứa các khóa và liên kết tượng trưng cho REGISTRY\A (một hive ứng dụng), REGISTRY\MACHINE và REGISTRY\USER. Để biết thông tin về các hive ứng dụng, xem http://msdn.microsoft.com/en-us/library/windows/hardware/jj673019%28v=vs.85%29.aspx.

Việc xác định vị trí của các hive đăng ký là quan trọng vì khả năng in ra dữ liệu khóa và giá trị thực tế của bạn phụ thuộc vào việc tìm thấy các hive trước tiên. Định dạng tệp đăng ký được tài liệu rõ ràng bởi Timothy D. Morgan (http://sentinelchicken.com/data/TheWindowsNTRegistryFileFormat.pdf). Bạn có thể thấy cấu trúc đơn giản của một tệp đăng ký trên đĩa trong Hình 10-2. Tệp đăng ký chứa một tiêu đề và được chia thành các phần gọi là hive bins (thùng hive). Sau đó, mỗi hive bin có một tiêu đề và được chia thành các cell (ô). Các cell chứa dữ liệu khóa và giá trị thực tế.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200739.png)

### Address Translations

Vì sự dịch địa chỉ, việc xử lý hive đăng ký trong bộ nhớ trở nên phức tạp hơn so với các tệp đăng ký trên đĩa. Trình quản lý cấu hình (Configuration Manager - CM) là thành phần của kernel quản lý registry (http://msdn.microsoft.com/en-us/library/windows/hardware/ff565712%28v=vs.85%29.aspx) và đặc biệt đối mặt với việc dịch địa chỉ. CM tạo ra một ánh xạ giữa các chỉ mục cell (giá trị được sử dụng để tìm các cell chứa dữ liệu của khóa registry) và các địa chỉ ảo. Sau đó, CM lưu trữ ánh xạ này trong cấu trúc _HHIVE. Thành viên quan trọng là Storage, có kiểu là _DUAL. Nếu bạn tra cứu cấu trúc _DUAL, bạn sẽ thấy thành viên Map.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200753.png)

Nếu bạn theo dõi thành viên Map, bạn sẽ tìm thấy tất cả các cấu trúc cần thiết để đúng mục đích xác định địa chỉ ảo cho một khóa đăng ký:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200806.png)

Mỗi chỉ mục cell được chia thành từng tập hợp chỉ mục và được sử dụng như một tập hợp chỉ mục vào các cấu trúc đã nêu ở trên. Hình 10-3 cho thấy một ví dụ về cách chỉ mục cell được chia để nhận địa chỉ ảo, như đã được mô tả trong bài thuyết trình của Brendan Dolan-Gavitt (http://www.dfrws.org/2008/proceedings/p26-dolan-gavitt_pres.pdf). Dưới đây là mô tả cho từng trường bit:

- Bit 0: Chỉ ra xem khóa có ổn định hay bay hơi. Khóa ổn định cũng có thể được tìm thấy trong tệp đăng ký trên đĩa, trong khi khóa bay hơi chỉ được tìm thấy trong bộ nhớ.

- Bit 1–10: Chỉ mục vào thành viên Directory.

- Bit 11–19: Chỉ mục vào thành viên Table.

- Bit 20–31: Độ lệch trong BlockAddress nơi dữ liệu khóa nằm. Đây là cell trong registry. Cell chứa độ dài của dữ liệu. Do đó, sau khi xác định được độ lệch trong BlockAddress, bạn phải cộng thêm 4 (kích thước của thành viên Length) để truy cập vào dữ liệu thực tế.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200818.png)

### Printing Keys and Values

Các khóa registry được lưu trữ dưới dạng cấu trúc cây, trong đó tồn tại một khóa gốc. Các con, hay còn gọi là khóa con, được duyệt qua cho đến khi đến đỉnh cuối cùng (thành phần cuối cùng của đường dẫn khóa). Do đó, để truy cập vào một khóa đăng ký và dữ liệu của nó, bạn phải bắt đầu từ khóa gốc và đi xuống cây cho đến khi đến đỉnh cuối cùng. Cấu trúc cho các node (_CM_KEY_NODE) được hiển thị trong đoạn mã sau:

![](https://github.com/HuyThang25/The-Art-of-Memory-Forensics/assets/93728466/0a8af32e-4002-4cf9-be5c-abe5dd87c398)

Khi bạn sử dụng tiện ích printkey, bạn truyền đường dẫn của khóa đăng ký mong muốn thông qua dòng lệnh (thông qua tham số -K/--key). Tiện ích này tìm tất cả các registry có sẵn trong bộ nhớ và truy cập các thành viên SubKeyLists và ValueLists để duyệt qua cây khóa. Nhờ vậy, tiện ích này cho phép bạn in ra một khóa, các khóa con của nó và các giá trị của nó. Ví dụ sau đây sẽ hướng dẫn bạn cách sử dụng tiện ích này:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200832.png)

Trong kết quả đầu ra, bạn có thể thấy đường dẫn registry, tên khóa, thời gian ghi cuối cùng (last write time), các khóa con (subkeys) và các giá trị mà khóa đó chứa (trong trường hợp này, không có giá trị nào). Tiện ích printkey cũng cho bạn biết liệu khóa đăng ký hoặc các khóa con của nó có tính ổn định (S - stable) hay bay hơi (V - volatile).

>**LƯU Ý:**<br> Bạn cũng có thể truyền tiện ích printkey một địa chỉ offset (sử dụng -o/--offset), đó là địa chỉ ảo của một hive đăng ký cụ thể. Điều này hữu ích nếu bạn muốn tập trung chỉ vào một hive đăng ký duy nhất. Bạn có thể lấy địa chỉ ảo này từ tiện ích hivelist, như đã được hiển thị trước đó trong chương.

### Detecting Malware Persistence

Có một số registry keys liên quan trong các cuộc điều tra liên quan đến phần mềm độc hại. Ví dụ, phần mềm độc hại có thể cần một cách để tồn tại trên hệ thống ngay cả sau khi hệ thống được khởi động lại. Một trong những cách dễ dàng để thực hiện công việc này là sửa đổi một trong những registry keys khởi động. Những khóa này chứa thông tin về các chương trình chạy khi hệ thống khởi động hoặc khi một người dùng đăng nhập. Do đó, bạn nên kiểm tra các registry keys đã biết này để xem liệu phần mềm độc hại có sử dụng chúng để tồn tại trên máy tính. Dưới đây là một số khóa khởi động đã biết:

- Để khởi động hệ thống:
  
    HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
    HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
    HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

- Để đăng nhập người dùng:

    HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows
    HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows\Run
    HKCU\Software\Microsoft\Windows\CurrentVersion\Run
    HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
      
Để có một danh sách khởi động chi tiết hơn về các khóa, bạn có thể tham khảo wiki của RegRipper (https://code.google.com/p/regripper/wiki/ASEPs) hoặc tiện ích AutoRuns của Sysinternals (http://technet.microsoft.com/en-us/sysinternals/bb963902.aspx). 

>**LƯU Ý**<br>
HKEY_CURRENT_USER (hoặc gọi tắt là HKCU) đề cập đến registry cụ thể cho từng người dùng. HKEY_LOCAL_MACHINE (hoặc gọi tắt là HKLM) đề cập đến registry được sử dụng bởi hệ thống.

Một ví dụ về sự tồn tại của phần mềm độc hại được hiển thị trong đầu ra sau đây. Chương trình thực thi độc hại C:\WINDOWS\system32\svchosts.exe được chạy mỗi khi hệ thống khởi động. Điều này ngay lập tức gây nghi ngờ vì không có tệp thực thi svchosts.exe nào tồn tại trên máy tính Windows sạch—nó đang cố gắng giả mạo svchost.exe hợp lệ (không có "s" thêm vào). Lưu ý rằng bạn không cần phải thêm tiền tố -K/--key với HKLM\SOFTWARE vì thực tế nó không phải là một phần của đường dẫn trong registry, mà thay vào đó chỉ định registry nào chứa khóa (ví dụ, hive SOFTWARE trên máy cục bộ).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200857.png)

Đầu ra sau đây là một ví dụ về sự tồn tại khi người dùng đăng nhập. Trong trường hợp này, chương trình quan tâm được xác định là một key logger, chạy mỗi khi người dùng Andrew (như được hiển thị trong đường dẫn registry) đăng nhập vào hệ thống. Lưu ý rằng toàn bộ đường dẫn sau HKCU được đưa cho tiện ích printkey:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200912.png)

Một phương pháp khác mà phần mềm độc hại thường sử dụng để tồn tại là tạo dịch vụ (service). Khi tạo dịch vụ, registry (đặc biệt là HKLM\SYSTEM\CurrentControlSet\Services) sẽ được sửa đổi để chứa thông tin về dịch vụ. Bạn có thể in ra khóa này và xác định xem tên dịch vụ có nổi bật là đáng nghi không. Nếu bạn sử dụng timelines, như đã thảo luận trong Chương 18, và kiểm tra khóa registry của dịch vụ, bạn có thể nhanh chóng xác định các dịch vụ mới được thêm vào dựa trên các mốc thời gian ghi cuối cùng. Dưới đây là đầu ra ví dụ từ một mẫu bộ nhớ chứa Stuxnet, cho thấy cơ chế tồn tại này:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200923.png)

>**LƯU Ý**<br>
>Bạn có thể lấy CurrentControlSet bằng cách truy vấn khóa bay hơi (volatile key) sau đây:
>![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200936.png)
>Điều này rất quan trọng để lưu ý vì có thể có nhiều bộ điều khiển (control set) chứa các cấu hình hệ thống khác nhau (xem http://support.microsoft.com/kb/100010). CurrentControlSet chứa các thiết lập mà máy tính đang sử dụng tại thời điểm hiện tại. Do đó, nếu bạn sử dụng bộ điều khiển không chính xác, có thể bạn sẽ không thấy được các cấu hình hiện tại. Điều này có thể dẫn đến các kết quả không chính xác hoặc thiếu sót trong quá trình điều tra hệ thống.

Mặc dù tiện ích printkey rất hữu ích, nhưng nó cũng có hạn chế là chỉ in ra các giá trị khóa gốc. Điều này phù hợp cho các giá trị số nguyên hoặc chuỗi, nhưng không đủ cho các khóa chứa dữ liệu nhị phân hoặc dữ liệu nhúng, chẳng hạn như khóa Userassist. Những khóa này yêu cầu một số xử lý bổ sung để giải thích trước khi hiển thị dữ liệu cho người dùng; nếu không, nó sẽ chỉ trông như một khối các byte hex. Ngoài ra, tiện ích printkey chỉ kiểm tra một registry key mỗi lần. Vì những lý do này, Volatility Registry API được tạo ra.

## Volatility’s Registry API


API Registry (Registry API) được thiết kế để cho phép xử lý dễ dàng của các registry keys phức tạp hoặc nhiều keys cùng một lúc. Ví dụ, bạn có thể sử dụng nó để kiểm tra tự động 20 khóa khởi động phổ biến nhất, sắp xếp các khóa dựa trên thời gian ghi cuối cùng của chúng, và nhiều hơn thế nữa. Trong đoạn mã dưới đây, chúng tôi sẽ chỉ cho bạn cách nhập và khởi tạo Registry API từ tiện ích volshell. Bạn cũng sử dụng mã gần như giống nhau để gọi API từ các tiện ích của riêng bạn.

```
>>> import volatility.plugins.registry.registryapi as registryapi
>>> regapi = registryapi.RegistryApi(self._config)
```

Khi đối tượng Registry API được khởi tạo, một từ điển (dictionary) chứa thông tin về tất cả các tệp registry trong mẫu bộ nhớ được lưu lại. Điều này làm cho việc chuyển đổi giữa các hive hiệu quả hơn mà không cần quét lại. Bạn có thể tham khảo dự án RegRipper (http://code.google.com/p/regripper/) để có cái nhìn về các khả năng viết plugin. Trong thực tế, Brendan đã tạo ra một dự án thử nghiệm mang tên VolRip cho một phiên bản cũ của Volatility, cho phép nhà điều tra chạy các lệnh RegRipper trực tiếp trên registry hives trong bộ nhớ (xem http://moyix.blogspot .com/2009/03/regripper-and-volatility-prototype.html). Tuy nhiên, điều này đã được thay thế bằng Registry API, có tính di động cao hơn vì nó không dựa vào "glue" từ Perl đến Python.

>**LƯU Ý**<br> Một phương pháp thay thế cho việc phân tích các tệp registry trong bộ nhớ là sử dụng tiện ích dumpfiles (được mô tả trong Chương 16) để trích xuất hives từ trình quản lý cache của Windows, sau đó phân tích chúng bằng các công cụ bên ngoài. Phương pháp này trích xuất bản sao cached của tệp hive, vì vậy các khóa volatile sẽ không được bao gồm. Hơn nữa, có thể có các khoảng trống được điền bằng số 0 trong mỗi tệp registry do việc phân trang. Hầu hết các công cụ registry ngoại tuyến mong đợi phân tích một tệp registry hoàn chỉnh từ đĩa (không phải từ bộ nhớ), và có thể cần điều chỉnh một số thứ để xử lý các tệp registry được trích xuất như vậy.<br>
Cũng lưu ý rằng hệ thống Windows 7 không lưu bản sao cached của tệp hive theo cách tương tự như các phiên bản Windows trước đó. Do đó, tiện ích dumpfiles không thể được sử dụng theo cách được mô tả.

Dưới đây là một ví dụ từ trong tiện ích volshell sử dụng Registry API để in ra các subkeys của một registry key đã chỉ định. Trước tiên, bạn thiết lập ngữ cảnh hiện tại là hive NTUSER.DAT của người quản trị, sau đó bạn sử dụng hàm reg_get_all_subkeys. Trong trường hợp này, bạn chỉ in ra tên, nhưng bạn có thể xử lý kết quả như bất kỳ registry key nào có kiểu _CM_KEY_NODE:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20200959.png)

Ngoài ra, đoạn mã sau đây cho thấy cách lấy giá trị registry. Một lần nữa, đây là từ trong tiện ích volshell. Nếu bạn muốn lấy một giá trị cụ thể bằng tên, bạn có thể sử dụng hàm reg_get_value.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201012.png)

Đoạn mã sau cho thấy cách in ra nhiều giá trị registry. Ở đây, bạn có thể thấy một trong các khóa khởi động được sử dụng, và mỗi giá trị của nó được in ra. Chú ý rằng bạn có thể thấy chương trình độc hại chạy mỗi khi hệ thống khởi động lại:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201025.png)

Nếu bạn muốn lấy mười khóa đã sửa đổi gần đây nhất từ registry NTUSER.DAT của người quản trị, đoạn mã sau cho thấy cách thực hiện điều đó. Trong trường hợp này, hoạt động cuối cùng trong hive NTUSER.DAT cho thấy đã tạo một chia sẻ mạng:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201037.png)

Nếu bạn muốn xem các subkeys và giá trị cho mỗi khóa đã sửa đổi, bạn có thể kết hợp các hàm bạn đã thấy. Dưới đây là mã ví dụ và một phần kết quả, trong đó bạn có thể thấy rằng chia sẻ mạng \DC01\response được ánh xạ thành ổ đĩa "z". Bạn có thể sử dụng tiện ích consoles (xem Chương 17) để tìm lệnh net use và xem xem ổ đĩa được ánh xạ bởi kẻ tấn công hay bởi người đã thu thập mẫu bộ nhớ.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201049.png)

## Parsing Userassist Keys

Các khóa Userassist là một tài liệu quan trọng trong registry được sử dụng để xác định các chương trình mà người dùng đã chạy, cũng như thời gian chạy của chúng. Các khóa này được tìm thấy trong các registry NTUSER.DAT của mỗi người dùng trên máy tính. Tất cả thông tin được chứa trong một đoạn binary blob cần được phân tích theo cách đặc biệt. Đường dẫn của khóa có thể khác nhau tùy thuộc vào hệ thống bạn đang điều tra. Ví dụ, trên Windows XP, 2003, Vista và 2008, đường dẫn khóa Userassist là:

```
HKCU\software\microsoft\windows\currentversion\explorer\userassist
 \{75048700-EF1F-11D0-9888-006097DEACF9}\Count
```

Bắt đầu từ Windows 7, đường dẫn khóa Userassist có thể là một trong các đường dẫn sau:

```
HKCU\software\microsoft\windows\currentversion\explorer\userassist
 \{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count
HKCU\software\microsoft\windows\currentversion\explorer\userassist
 \{F4E57C4B-2036-45F0-A9AB-443BCFE33D9F}\Count
```

Ngoài dữ liệu nhị phân cần được giải mã cho mỗi khóa này, tên giá trị chứa đường dẫn của chương trình (hoặc liên kết) đã được truy cập. Tuy nhiên, nó được mã hóa rot13, một mã Caesar đơn giản trong đó các chữ cái được dịch chuyển 13 vị trí. Dưới đây là dữ liệu gốc được trích xuất bằng tiện ích printkey. Giá trị, như bạn có thể thấy, không thể đọc được vì nó đã được mã hóa rot13. Ngoài ra, dữ liệu nhị phân chứa một timestamp:

```
REG_BINARY   HRZR_EHACNGU:P:\JVAQBJF\flfgrz32\pzq.rkr : (S)
 0x00000000 01 00 00 00 06 00 00 00 b0 41 5e b0 95 b6 ca 01
```

Giải mã dữ liệu Userassist sử dụng các cấu trúc được xác định trước. Tương tự như đường dẫn khóa, các cấu trúc này thay đổi tùy thuộc vào hệ điều hành. Bạn có thể thấy cấu trúc cho các máy Windows XP, 2003, Vista và 2008 trong đoạn mã dưới đây. Các thành viên quan trọng là `CountStartingAtFive`, đó là số lần ứng dụng đã chạy, và `LastUpdated`, đó là timestamp của lần chạy cuối cùng của ứng dụng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201114.png)

Đầu ra dưới đây cho thấy dữ liệu đã được giải mã từ tiện ích userassist. Bạn có thể thấy đường dẫn tới chương trình (cmd.exe) và xác định rằng nó đã chạy một lần, lúc 3:42:15 ngày 26 tháng 2 năm 2010. Dựa vào thông tin này, bạn có thể sử dụng các tiện ích cmdscan hoặc consoles (xem Chương 17) để xem xem có tồn tại các lệnh của kẻ tấn công trong bộ nhớ không. Tiện ích userassist cũng xuất ra dữ liệu nhị phân gốc nếu bạn cần xác minh tính đúng đắn của đầu ra.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201128.png)

## Detecting Malware with the Shimcache

Các khóa registry Shimcache là một phần của Cơ sở dữ liệu Tương thích Ứng dụng, nơi "xác định các vấn đề tương thích ứng dụng và các giải pháp của chúng" (xem http://msdn.microsoft.com/en-us/library/bb432182(v=vs.85).aspx). Các khóa này chứa một đường dẫn đến một tệp thực thi và thời gian sửa đổi lần cuối cùng từ thuộc tính $STANDARD_INFORMATION của MFT entry. Điều này rất hữu ích để chứng minh rằng một phần mã độc đã tồn tại trên hệ thống và chạy vào thời gian nào. Tùy thuộc vào hệ điều hành, có hai khóa registry có thể được sử dụng:

- Đối với Windows XP:
 
    HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatibility
- Đối với Window 2003, Vista, 2008, 7 và 8:
 
    HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache

Nếu bạn in ra những khóa registry đó, bạn sẽ thấy rất nhiều dữ liệu nhị phân. Dữ liệu này phải được giải mã bằng cách sử dụng các cấu trúc cụ thể. Đoạn mã sau cho thấy các cấu trúc được sử dụng để biểu diễn các bản ghi Shimcache trên hệ thống Windows XP:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201144.png)

Một thành viên quan trọng là danh sách Entries của ShimRecords, đó là một danh sách các đối tượng AppCompatCacheEntry. Các đối tượng AppCompatCacheEntry chính là các đối tượng thực sự chứa thông tin về bản ghi Shimcache, chẳng hạn như đường dẫn tới tệp và các dấu thời gian. Đoạn mã sau đây cho thấy dữ liệu gốc cho giá trị AppCompatCache trên một hệ thống Windows XP:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201157.png)


Đoạn mã sau đây cho thấy một ví dụ về đầu ra (đã loại bỏ thông tin nhạy cảm) từ một máy chủ Windows 2003. Như bạn có thể thấy, một số tệp thực thi với tên kỳ lạ đã xuất hiện:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201209.png)

**>Chú ý**<br> Một điều về các mục nhập Shimcache là các mục nhập cũ thường bị ghi đè bởi các mục nhập mới hơn khi kích thước tối đa của giá trị khóa đạt đến (kích thước này thay đổi tùy thuộc vào hệ thống). Do đó, bạn có thể thấy dữ liệu dư thừa (tương tự như không gian lửng trên ổ cứng) trên một máy tính đã chạy trong một khoảng thời gian dài.

## Reconstructing Activities with Shellbags

Shellbags là một thuật ngữ thường được sử dụng để mô tả một tập hợp các khóa registry cho phép "hệ điều hành Windows theo dõi sở thích xem cửa sổ của người dùng cụ thể cho Windows Explorer" (xem http://www.dfrws.org/2009/proceedings/p69-zhu.pdf). Các khóa này chứa nhiều thông tin quan trọng cho công tác điều tra pháp lý. Dưới đây là một số ví dụ về các dấu vết bạn có thể tìm thấy:

-	Kích thước và ưu tiên của cửa sổ Windows
-	Cài đặt biểu tượng và xem thư mục
-	Dữ liệu siêu dữ liệu như các dấu thời gian MAC
-	Tệp MRU (Most Recently Used) và loại tệp (zip, thư mục, trình cài đặt)
-	Tệp, thư mục, tệp zip và trình cài đặt từng tồn tại trên hệ thống (ngay cả khi đã bị xóa)
-	Các chia sẻ và thư mục mạng trong các chia sẻ
-	Siêu dữ liệu liên quan đến bất kỳ loại nào trong số này có thể bao gồm các dấu thời gian và đường dẫn tuyệt đối
-	Thông tin về các ổ đĩa TrueCrypt

>**GHI CHÚ**<br>
Để biết thêm thông tin về các cấu trúc dữ liệu Shellbag, các khóa registry tương ứng và cách sử dụng chúng trong điều tra pháp lý, bạn có thể tham khảo các nguồn tài liệu sau:<br>
>>-	 Shellbag Analysis của Harlan Carvey: http://windowsir.blogspot.com/2012/08/shellbag-analysis.html
>>-	 Windows Shellbag Forensics của Willi Ballenthin: http://www.williballenthin.com/forensics/shellbags
>>-	 Shellbags Forensics: Addressing a Misconception: http://www.4n6k.com/2013/12/shellbags-forensics-addressing.html

### Shellbags in Memory

Đối với Volatility, plugin shellbags sử dụng Registry API để trích xuất dữ liệu từ các khóa thích hợp. Sau đó, nó phân tích cú pháp dữ liệu đó bằng các loại dữ liệu Shellbag và xuất phiên bản đã định dạng cùng với thông tin MRU. Bằng cách sử dụng chi tiết MRU, bạn có thể tương quan thời gian ghi cuối cùng của khóa registry với mục shellbags được cập nhật cuối cùng. Dưới đây là một ví dụ về việc sử dụng plugin shellbags:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201229.png)

Dưới đây là một số điều lưu ý về các mục Shellbags:
-	Các mục SHELLITEM vẫn tồn tại trong registry ngay cả sau khi tệp đã bị xóa.
-	Thời gian gắn với các mục SHELLITEM không được cập nhật, ngay cả khi tệp được sửa đổi hoặc truy cập sau đó.
-	Các mục ITEMPOS được cập nhật nếu tệp được di chuyển, xóa hoặc truy cập.
-	Nếu người dùng không đăng nhập vào hệ thống vào thời điểm lấy mẫu bộ nhớ, các hive của người dùng đó sẽ không có sẵn trong bộ nhớ và do đó dữ liệu Shellbag không được xử lý.

### Finding TrueCrypt Volumes with Shellbags

Các ổ đĩa TrueCrypt cũng xuất hiện trong các khóa Shellbags, thường là dưới dạng các mục ITEMPOS. Vì các mục ITEMPOS được cập nhật, nếu ổ đĩa TrueCrypt được di chuyển hoặc xóa, mục của nó sẽ được cập nhật hoặc loại bỏ để phản ánh thay đổi. Tuy nhiên, các tệp được truy cập từ ổ đĩa này sẽ giữ nguyên các mục Shellbags của chúng. Trong đầu ra dưới đây, bạn có thể thấy máy có một ổ đĩa TrueCrypt được gắn kết vào ổ T: và nó đã được truy cập vào ngày 25/09/2012 lúc 11:48:46.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201245.png)

>**LƯU Ý**<br>
Lưu ý rằng tên MyTrueCryptVolume được chọn có ý định khi tạo ổ đĩa để nó nổi bật trong registry. Ổ đĩa TrueCrypt thực tế sẽ có thể không có tên rõ ràng như vậy.

Sau khi ổ đĩa TrueCrypt bị xóa, mục ITEMPOS của nó biến mất khỏi registry, như được thể hiện trong đầu ra dưới đây:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201256.png)

Mặc dù mục MyTrueCryptVolume không còn tồn tại trong registry, bất kỳ tập tin cá nhân nào đã được truy cập từ ổ đĩa TrueCrypt trong khi nó đã được gắn kết có thể vẫn còn mục trong registry. Vì liên kết giữa ổ đĩa TrueCrypt và các tập tin của nó trong các mục Shellbags đã bị chia rời sau khi nó bị xóa, việc xác định chính xác những tệp tồn tại trong ổ đĩa TrueCrypt trở nên khó khăn, vì các mục này chỉ hiển thị tên tệp chứ không phải là đường dẫn đầy đủ. Tuy nhiên, nếu bạn có dấu thời gian khi ổ đĩa TrueCrypt tồn tại trên hệ thống (từ bản sao hình ảnh trước đó, bản ghi registry, hoặc tệp MFT từ đĩa), bạn có thể suy ra những thông tin đó để tìm các mục ITEMPOS cho các tập tin trong khoảng thời gian đó. Trong ví dụ dưới đây, bạn có thể kết nối tệp news.txt và customer emails.txt gần đây với ổ đĩa TrueCrypt vì chúng đã được truy cập trong khoảng một giờ sau thời gian truy cập MyTrueCryptVolume:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201308.png)

### Timestomping Registry Keys

Cần lưu ý rằng thời gian dấu thời gian của các khóa registry có thể bị ghi đè hoặc "stomped". Kỹ thuật chống pháp y giúp ẩn các đối tượng khỏi phân tích dựa trên timeline. Joakim Schicht đã viết một công cụ proof-of-concept, SetRegTime, để minh họa khả năng này (xem http://code.google.com/p/mft2csv/wiki/SetRegTime). Công cụ này ghi đè các dấu thời gian mong muốn trong các khóa registry bằng cách sử dụng Windows API (cụ thể là NtSetInformationKey). Như đã thảo luận trước đó, vì sử dụng Windows API, các thay đổi được phản ánh trong bộ nhớ trong khoảng thời gian xả lúc 5 giây. Ví dụ dưới đây cho thấy kết quả đầu ra từ plugin shellbags sau khi dấu thời gian của một khóa registry bị ghi đè bằng SetRegTime:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201322.png)

Kết quả đầu ra hiển thị LastWriteTime là 3024-05-21 00:00:00, một ngày rõ ràng thuộc tương lai. Lưu ý rằng ngày mới này không ảnh hưởng đến các dấu thời gian đã nhúng trong các mục Shellbags (hoặc bất kỳ khóa registry nào khác có dấu thời gian nhúng như đã thảo luận trong chương này). Với các mục Shellbags, bạn biết rằng bạn ít nhất phải có một dấu thời gian nhúng có cùng ngày với LastWriteTime, điều này rõ ràng không đúng trong trường hợp này. Do đó, nếu bạn thấy các dấu thời gian LastWriteTime không đồng bộ với các giá trị dấu thời gian nhúng, đây là một dấu hiệu rõ ràng rằng có điều gì đó không ổn với khóa đó.

Nếu các khóa không có dấu thời gian nhúng được chọn để thực hiện timestomping, và nếu các ngày timestomped mới nằm trong khoảng thời gian có vẻ bình thường (ví dụ: không phải trong năm 3024), thì khó khăn hơn để bạn phát hiện ra xem dấu thời gian của những khóa registry đó đã thay đổi thực sự hay chưa. Trong các trường hợp này, bạn có thể phải sử dụng dấu thời gian của các hợp lệ hệ thống khác, kết hợp với dấu thời gian của khóa registry, để phát hiện những khóa bị timestomped này. Để thực hiện điều này, bạn có thể áp dụng các phương pháp được thảo luận trong Chương 18, nơi đề cập đến việc tạo timeline kỹ lưỡng để phát hiện những hoạt động độc hại như vậy.

## Dumping Password Hashes

Bạn có thể trích xuất băm mật khẩu từ mẫu bộ nhớ bằng cách sử dụng plugin hashdump. Plugin hashdump sử dụng các khóa từ cả hai hive SYSTEM và SAM, được tìm thấy tự động bằng cách sử dụng Registry API. Sau đó, các băm có thể được cung cấp cho công cụ crack hash để lấy được mật khẩu văn bản rõ ràng. Plugin này được ưa chuộng trong cộng đồng tấn công, như bạn có thể tưởng tượng.

Như Brendan Dolan-Gavitt giải thích trong blog của mình (xem http://moyix.blogspot.com/2008/02/syskey-and-sam.html), thông thường có hai loại băm mật khẩu được lưu trữ trong SAM: băm LanMan (LM) và băm NT. Băm LM, có một số lỗ hổng thiết kế làm cho nó dễ bị crack, được coi là lỗi thời. Do đó, nó bị vô hiệu hóa theo mặc định trên Windows Vista, 2008, 7 và 8. Nó cũng có thể bị vô hiệu hóa rõ ràng trên Windows XP và 2003 (xem http://www.microsoft.com/security/sir/strategy/default.aspx#!password_hashes). Tuy nhiên, băm NT được hỗ trợ bởi tất cả các hệ điều hành Windows hiện đại. Plugin hashdump thu thập cả hai loại băm.

Ví dụ dưới đây minh họa cách sử dụng plugin hashdump để trích xuất các băm mật khẩu:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201335.png)

Sau khi bạn đã thu thập các băm mật khẩu, bạn có thể sử dụng một công cụ crack mật khẩu như John the Ripper (http://www.openwall.com/john/) để giải mã chúng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201348.png)

Bạn đã có mật khẩu, nhưng nó chỉ gồm các chữ cái in hoa, có thể không chính xác. Nếu bạn sử dụng phiên bản "jumbo" của phần mềm John (xem tại http://insidetrust .blogspot.com/2011/01/password-cracking-using-john-ripper-jtr.html), bạn có thể thu được mật khẩu chính xác, trong trường hợp này là "password" (tất cả các chữ cái đều viết thường).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201401.png)

Từ góc độ tấn công, điều này rất hữu ích. Ví dụ, hãy tưởng tượng rằng bạn đã có quyền truy cập vào một máy chủ VMware ESX với nhiều máy ảo. Bằng cách truy cập vào các tệp bộ nhớ snapshot của mỗi máy ảo, bạn có thể thử crack mật khẩu cho từng máy. Bất chợt không gian tấn công của bạn đã tăng đáng kể!

## Obtaining LSA Secrets

Lsadump plugin được sử dụng để trích xuất các LSA secrets đã được giải mã từ registry của tất cả các máy chủ Windows được hỗ trợ (http://moyix.blogspot.com/2008/02/decrypting-lsa-secrets.html). Điều này tiết lộ thông tin như mật khẩu mặc định (cho các hệ thống có bật chế độ đăng nhập tự động), khóa riêng tư RDP và các thông tin đăng nhập được sử dụng bởi Data Protection API (DPAPI). Lsadump plugin sử dụng cả hives SYSTEM và SECURITY, được tìm thấy tự động bằng cách sử dụng Registry API.

Một số thông tin mà bạn có thể tìm thấy trong LSA Secrets bao gồm:
- $MACHINE.ACC: Xác thực miền (http://support.microsoft.com/kb/175468).
- DefaultPassword: Mật khẩu được sử dụng để đăng nhập vào Windows khi bật chế độ đăng nhập tự động.
- NL$KM: Khóa bí mật được sử dụng để mã hóa mật khẩu miền được lưu trữ tạm thời (http://moyix
.blogspot.com/2008/02/cached-domain-credentials.html).
- L$RTMTIMEBOMB_*: Dấu thời gian chỉ ngày mà một bản sao của Windows chưa kích hoạt sẽ ngừng hoạt động.
- L$HYDRAENCKEY_*: Khóa riêng tư được sử dụng cho Giao thức Điều khiển Máy tính từ xa (RDP). Nếu bạn cũng có một bản ghi từ gói tin từ một hệ thống bị tấn công qua RDP, bạn có thể trích xuất khóa công khai của máy khách từ bản ghi này và khóa riêng tư của máy chủ từ bộ nhớ; sau đó giải mã giao thông.

Bạn có thể xem ví dụ về plugin lsadump trong hình thức hoạt động trong đầu ra sau, nó hiển thị LSA Secret cho khóa RDP riêng tư:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201415.png)

Trong đầu ra sau, bạn có thể thấy các LSA Secrets cho DefaultPassword và DPAPI_SYSTEM:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20201425.png)

## Tổng quan

Đối với các cuộc điều tra kỹ thuật số, registry là một thành phần cốt lõi của hệ điều hành Windows và do đó là một khía cạnh quan trọng của hầu hết các cuộc điều tra kỹ thuật số. Kỹ thuật nhớ trạng thái (memory forensics) cho phép nhà điều tra truy cập vào các phần của registry tạm thời mà không thể tìm thấy trên đĩa và khám phá các sự thay đổi trong registry mà có thể không bao giờ được ghi lại trở lại đĩa. Trong khi nhà điều tra có thể truy cập các phiên bản được lưu trữ trong bộ nhớ cache của dữ liệu registry truyền thống thường được lưu trong hệ thống tệp tin, khả năng của Volatility trong việc phân tích các hiện vật registry nằm trong bộ nhớ mở ra một lĩnh vực phân tích mới mà không thể thực hiện được với phân tích đĩa. Khi kết hợp sức mạnh đó với phân tích cấu trúc của các dữ liệu nhúng trong các khóa Userassist, Shellbags và Shimcache, bạn có khả năng theo dõi nhiều khía cạnh của hoạt động của người dùng. Ngoài ra, thông qua việc truy vấn dữ liệu registry trong bộ nhớ trạng thái (memory dump), bạn có thể nhanh chóng xác định sự tồn tại của malware, mật khẩu được lưu trong bộ nhớ cache, và nhiều hơn thế nữa.
