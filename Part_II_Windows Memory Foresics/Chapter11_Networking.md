Hầu hết các loại phần mềm độc hại đều có khả năng kết nối mạng, cho dù mục đích là liên lạc với máy chủ điều khiển và điều phối, lây lan sang các máy khác, hoặc tạo một cửa sau trên hệ thống. Bởi vì hệ điều hành Windows phải duy trì trạng thái và chuyển tiếp các gói tin nhận được đến các tiến trình hoặc trình điều khiển chính xác, không có gì ngạc nhiên khi các chức năng API liên quan dẫn đến tạo ra các hiện vật đáng kể trong bộ nhớ. Ngoài ra, kẻ tấn công, dù từ xa hay cục bộ, không thể tránh được việc để lại các dấu vết về các hoạt động mạng của họ trong lịch sử trình duyệt web, bộ đệm DNS, và những khía cạnh khác.

Chương này cung cấp cho bạn hiểu biết về cách tạo ra các hiện vật mạng trong bộ nhớ và những yếu tố quan trọng nhất trong cuộc điều tra của bạn. Bên cạnh đó, bạn sẽ tìm hiểu sự quan trọng của việc Microsoft hoàn toàn thiết kế lại bộ chồng TCP/IP bắt đầu từ Windows Vista; và bạn sẽ khám phá hai phương pháp không được tiết lộ để khôi phục các socket và kết nối từ các bản sao bộ nhớ. Hơn nữa, bạn sẽ khám phá tại sao việc đáp ứng nhanh chóng đối với các sự cố tiềm ẩn là rất quan trọng, và tại sao việc liên kết các bằng chứng liên quan đến mạng trong bộ nhớ với các nguồn dữ liệu bên ngoài như các bản ghi gói tin và log tường lửa/proxy/IDS là vô giá.

## Network Artifacts

Có hai loại chính của các hiện vật mạng là sockets (tiếng Việt: ổ cắm) và connections (tiếng Việt: kết nối). Sockets định nghĩa các điểm cuối cho giao tiếp. Các ứng dụng tạo client sockets để khởi tạo kết nối đến máy chủ từ xa và tạo server sockets để lắng nghe trên một giao diện để chờ kết nối từ xa. Có một số cách để tạo các sockets này:

- Trực tiếp từ chế độ người dùng: Ứng dụng có thể gọi hàm socket từ API Winsock2 (ws2_32.dll).
- Gián tiếp từ chế độ người dùng: Ứng dụng có thể gọi các hàm trong các thư viện như WinINet (wininet.dll), cung cấp các bao quanh cho các hàm của Winsock2.
- Trực tiếp từ chế độ kernel: Các trình điều khiển kernel có thể tạo sockets thông qua việc sử dụng giao diện điều khiển truyền dẫn (TDI), đây là giao diện chính dùng để truyền dẫn bởi các thành phần cấp cao hơn như Winsock2.

### Windows Sockets API (Winsock)

Khi một ứng dụng gọi hàm socket, nó truyền các thông tin sau:
- Một loại địa chỉ (AF_INET cho IPv4, AF_INET6 cho IPv6)
- Một kiểu (SOCK_STREAM, SOCK_DGRAM, SOCK_RAW)
- Một giao thức (IPPROTO_TCP, IPPROTO_UDP, IPPROTO_IP, IPPROTO_ICMP)

Sau khi ứng dụng gọi hàm socket, socket không sẵn sàng sử dụng ngay. Đối với các máy chủ, khi gọi hàm bind và listen, chúng phải cung cấp địa chỉ và cổng cục bộ. Tương tự, đối với các máy khách, khi gọi hàm connect, chúng phải cung cấp địa chỉ và cổng từ xa (bind là tùy chọn cho máy khách). Một socket không thể hoạt động cho đến khi nó biết địa chỉ IP và cổng. Do đó, việc cấp phát cho _ADDRESS_OBJECT (tên của cấu trúc đại diện cho các đối tượng socket) diễn ra sau khi gọi hàm bind hoặc connect thay vì sau khi gọi hàm socket.

>**GHI CHÚ**<br>
Nhiều tên cấu trúc trong chương này, bao gồm _ADDRESS_OBJECT, không phải là các tên của Microsoft. Bởi vì chúng không được tài liệu chính thức công bố, chúng tôi đã đặt tên cho chúng dựa trên cách nghe có vẻ hợp lý.

Hình 11-1 và hình 11-2 cho thấy chuỗi cuộc gọi API cần thiết để tạo một máy chủ TCP đơn giản và mối quan hệ giữa các API đó và các hiện vật trong bộ nhớ. Đối với toàn bộ mã nguồn, xem tài liệu về socket trên MSDN (http://msdn.microsoft.com/en-us/library/ms740673%28VS.85%29.aspx).

Các hình ảnh hiển thị như sau:
1. Cả máy chủ và máy khách đều bắt đầu bằng cách gọi socket, điều này khiến quá trình gọi mở một handle đến \Device\Afd\Endpoint. Handle này cho phép quá trình chế độ người dùng giao tiếp với Afd.sys trong chế độ kernel, đây là Driver Chức năng Phụ cho Winsock2. Đây không phải là một handle tùy chọn; nó phải duy trì mở trong suốt tuổi thọ của socket, nếu không thì socket sẽ trở nên không hợp lệ.
2. Máy chủ gọi bind (điều này là tùy chọn cho máy khách), dẫn đến các hiện vật sau đây. Lưu ý rằng máy chủ cũng gọi listen, không tạo ra các hiện vật mới.
    - Quá trình gọi mở một handle đến \Device\Tcp, \Device\Udp hoặc \Device\Ip, tùy thuộc vào giao thức được chỉ định trong cuộc gọi đến socket.
    - Bộ nhớ được cấp phát trong kernel cho một cấu trúc _ADDRESS_OBJECT, và các thành viên của nó được điền vào theo các tham số được gửi đến socket và bind.
3. Máy khách gọi connect, dẫn đến các hiện vật giống như đã thảo luận trước đó, bên cạnh việc cấp phát một _TCPT_OBJECT (tức là đối tượng kết nối). Đối với mỗi kết nối được thiết lập với một máy khách (khi accept trả về), quá trình máy chủ cũng sẽ trở thành một _TCPT_OBJECT và một tập mới của các handle. Các hiện vật này tồn tại cho đến khi các ứng dụng gọi closesocket, lúc đó các handle được đóng và các đối tượng được giải phóng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223320.png)

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223338.png)

>**Chú ý**<br>Việc giải phóng một đối tượng không có nghĩa là bộ nhớ cho đối tượng đó được ghi đè ngay lập tức. Do đó, bạn có thể tìm thấy các dấu vết của các đối tượng trước đó trong bộ nhớ đã được giải phóng hoặc hủy bỏ sau một thời gian dài sau khi các socket đã được sử dụng. Điều này sẽ được thảo luận chi tiết hơn ở phần sau của chương.

Bây giờ sau khi bạn đã hiểu được cách và khi nào các đối tượng mạng khác nhau được tạo ra trong bộ nhớ, bạn sẽ tiếp tục tìm hiểu cách định vị chúng trong bộ nhớ và các kết luận mà bạn có thể rút ra trong quá trình điều tra.

**Mục tiêu**

-	Xác định các listeners giả mạo: Học cách phân biệt giữa các server sockets hợp pháp và những cái đang được sử dụng để chấp nhận các kết nối đến từ các kẻ tấn công. Trong một số trường hợp, biên giới có thể mỏng - ví dụ, nếu một máy chủ tập tin đang lắng nghe trên cổng 21 (FTP), điều này là đáng mong đợi; nhưng nếu nó chứa các tài liệu đã bị đánh cắp hoặc nếu nó cho phép máy bị xâm phạm do một lỗ hổng trong mã của máy chủ FTP, điều này nhanh chóng trở thành một phần quan trọng hơn trong cuộc điều tra của bạn.
-	Phát hiện các kết nối từ xa đáng ngờ: Một trong những cách thông thường mà các nhà điều tra sử dụng các đối tượng mạng trong bộ nhớ là phân tích các kết nối từ xa. Một quá trình cụ thể trên hệ thống đã truy cập vào cổng TCP trên một máy chủ ở một quốc gia nước ngoài? Nhân viên đã sử dụng một ứng dụng TOR để duyệt web? Phần mềm độc hại có giao tiếp với trung tâm điều khiển của nó bằng cách sử dụng giao thức trò chuyện thời gian thực như Jabber hay IRC? Có bất kỳ địa chỉ IP từ xa nào nằm trong danh sách đen hoặc được đánh dấu bởi các dịch vụ xếp hạng IP?
-	Xác định các hệ thống có card mạng chế độ lắng nghe: Bạn có thể xem xét các socket trong bộ nhớ của một máy nghi ngờ và xác định xem một trong các card mạng của nó có ở chế độ lắng nghe không. Điều này có nghĩa là bạn có thể phát hiện ra các máy trên mạng của bạn có thể đang cố gắng nghe lén dữ liệu từ/đến các hệ thống khác hoặc thực hiện các cuộc tấn công trung gian.
-	Phát hiện các cổng ẩn trên các hệ thống hoạt động: Nhiều rootkit lọc các cổng và địa chỉ IP bằng cách kết nối API trên các hệ thống hoạt động. Vì bộ nhớ thẩm tra không phụ thuộc vào các API của hệ điều hành, các kết nối bị ẩn dễ dàng nhìn thấy và không có tác động. Do đó, bằng cách so sánh dữ liệu có sẵn thông qua API Windows trên một máy tính hoạt động với những gì Volatility thấy trong RAM của máy tính, hoạt động mạng bị ẩn có thể được tiết lộ.
-	Tái tạo lịch sử trình duyệt: Học cách xác định các URL mà trình duyệt (hoặc mẫu phần mềm độc hại sử dụng API của trình duyệt) đã truy cập trên máy nghi ngờ. Nếu các tập tin lịch sử bị xóa từ ổ đĩa, vẫn còn một cơ hội bạn có thể tìm thấy các bản ghi bộ nhớ được lưu trữ trong bộ nhớ được lưu trữ trong bộ nhớ, cùng với thông tin như thời gian truy cập cuối cùng, kích thước dữ liệu được trả về bởi máy chủ web và thậm chí là tiêu đề phản hồi HTTP.

**Cấu trúc dữ liệu (XP và 2003)**

Các cấu trúc _ADDRESS_OBJECT và _TCPT_OBJECT không được Microsoft tài liệu hóa, nhưng nhiều người đã thực hiện đảo ngược chúng trong quá khứ. Dưới đây là những biến thể được sử dụng trong khung Volatility cho các hệ thống Windows XP và Server 2003 64-bit.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223355.png)

**Các điểm chính** 

- Next: Con trỏ đến đối tượng tiếp theo, tạo thành một danh sách liên kết đơn các mục nhập. Mục nhập kết thúc có giá trị Next là 0. Trường này có thể được sử dụng để liệt kê các socket và kết nối hoạt động.
- LocalIpAddress: Địa chỉ IP cục bộ, được định dạng dưới dạng số nguyên (packed integer). Địa chỉ này có thể là 0.0.0.0 nếu socket đang lắng nghe trên tất cả các IP.
- LocalPort: Cổng cục bộ (big endian).
- Protocol: Số giao thức IP (xem http://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml). Thành viên này không cần thiết cho _TCP_OBJECT vì những cấu trúc đó chỉ dành cho TCP theo định nghĩa.
- Pid: ID quy trình (PID) của quy trình đã mở socket hoặc tạo kết nối.
- CreateTime: Thời điểm tạo socket, là một timestamp UTC (chỉ dành cho socket).
- RemotePort: Cổng từ xa (chỉ dành cho kết nối) trong định dạng big endian.
- RemoteIpAddress: Địa chỉ IP từ xa (chỉ dành cho kết nối) dưới dạng số nguyên (packed integer).

### Active Sockets and Connections

Hệ điều hành duy trì các socket và kết nối hoạt động bằng cách sử dụng bảng băm có dây nối, bao gồm danh sách liên kết đơn (xem Chương 2). Do đó, một cách để liệt kê các socket hiện có trên hệ thống là tìm và đi qua tất cả các mục nhập trong bảng băm. Bạn xem xét mỗi mục nhập khác không trong bảng băm như là điểm bắt đầu của một danh sách liên kết đơn các cấu trúc _ADDRESS_OBJECT, và theo dõi các con trỏ Next cho đến khi đạt đến cuối danh sách (được chỉ ra bằng giá trị Next là 0). Tương tự, bạn có thể làm điều tương tự với danh sách _TCPT_OBJECT để liệt kê các kết nối mở trên hệ thống.

Thực tế, đây là cách mà các plugin sockets và connections trong Volatility hoạt động. Đối với cả hai lệnh, Volatility tìm modul tcpip.sys trong bộ nhớ kernel và xác định một biến toàn cục không xuất khẩu trong phần dữ liệu của nó. Đối với sockets, biến mà Volatility tìm thấy được đặt tên là _AddrObjTable, lưu trữ một con trỏ đến mục nhập _ADDRESS_OBJECT đầu tiên. Đối với kết nối, nó tìm một biến có tên là _TCBTable, lưu trữ một con trỏ đến mục nhập _TCPT_OBJECT đầu tiên. Hình 11-3 cho thấy sơ đồ của quy trình liệt kê.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223409.png)

Dưới đây là một ví dụ về việc sử dụng công cụ Volatility để hiển thị các socket trong một bản sao bộ nhớ bị nhiễm Zeus malware.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223425.png)

Trong đầu ra, bạn có thể thấy ID quy trình của quy trình sở hữu, cổng, giao thức và thời gian tạo. Hãy bắt đầu phân tích bằng cách nhìn vào mục được đánh dấu đậm ở đầu, cho thấy một quy trình có PID 892 đang sử dụng cổng TCP 19705. Vì một _ADDRESS_OBJECT được cấp phát cho cả các socket client và server, bạn không thể biết liệu quy trình đó có đang lắng nghe kết nối đến trên cổng TCP 19705 hay chỉ mới thiết lập một kết nối TCP với một điểm cuối từ xa (ví dụ: memoryanalysis.net: 80) sử dụng cổng 19705 làm cổng nguồn.

Một điều bạn biết là các cổng dưới 1025 thường được dành riêng cho các máy chủ. Các cổng trên 1025 có thể là các cổng khách tạm thời (ví dụ: có tuổi thọ ngắn) hoặc các cổng máy chủ cho các ứng dụng không có đặc quyền cần thiết để kết nối đến các dãy cổng thấp hơn. Tất nhiên, luôn có những ngoại lệ (như giao thức Remote Desktop Protocol [RDP], nó gắn với TCP 3389 ngay cả khi có đặc quyền quản trị). Do đó, bạn sẽ cần nhiều thông tin hơn để phân biệt mục đích của socket TCP sử dụng cổng 19705.

Tiếp tục với những gì bạn biết về cổng khách tạm thời: Chúng tăng lên một đơn vị cho đến khi đạt đến giá trị tối đa (giá trị này thay đổi, xem Ghi chú sắp tới), khi đó chúng quay trở lại cổng 1025. Nếu TCP 19705 xảy ra là một socket client, các quy trình khác trên hệ thống đã tạo socket client trong vài giây sẽ được gán giá trị gần 19705. Hãy sắp xếp tất cả các socket được tạo trong cùng một khoảng thời gian theo thời gian tạo và xem liệu có bằng chứng nào hỗ trợ lý thuyết của chúng ta không.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223441.png)

Lúc 03:38:12, hệ thống gán cổng 1275 và 1276 cho một quy trình có PID 1064. Ba giây sau đó, lúc 03:38:15, hệ thống gán cổng 1277 cho một quy trình có PID 892. Trong khoảng thời gian giữa hai sự kiện này, lúc 03:38:14, bạn thấy có các socket được tạo với các số rất xa nhau là 19705 và 35335. Mẫu này cho thấy các socket có cổng 1275, 1276 và 1277 có thể là các socket khách tạm thời, trong khi các socket có cổng 19705 và 35335 có thể là các socket máy chủ. Hơn nữa, vì hai socket khách tạm thời đầu tiên sử dụng giao thức UDP, chúng có thể liên quan đến việc thực hiện các yêu cầu DNS.

>LƯU Ý<br>
Các phạm vi cổng khách tạm thời thực tế có thể thay đổi giữa các phiên bản hệ điều hành. Bạn cũng có thể cấu hình các phạm vi này thủ công bằng cách chỉnh sửa registry. Để biết thêm thông tin, hãy xem http://www.ncftp.com/ncftpd/doc/misc/ephemeral_ports.html#Windows.

Bạn có thể điều tra sâu hơn bằng cách xác định các tiến trình đang sử dụng các socket này và xem có bất kỳ kết nối nào đang hoạt động. Đầu ra sau đây cho thấy rằng các socket được tạo bởi hai phiên bản khác nhau của tiến trình svchost.exe và rằng TCP 1277, thực sự là một socket khách tới cổng 80 của XX.XX.117.254 (địa chỉ ở Ukraina). Lưu ý: XX biểu thị cho các giá trị đã được che đậy.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223456.png)

Như bạn đã học trong Chương 8, một số phần mềm độc hại, bao gồm cả Zeus, tiêm mã vào các quy trình khác để giữ cho mình ẩn danh. Bạn có thể thấy hiệu quả của việc tiêm mã và cách nó khiến svchost.exe trở nên có vẻ như đang thực hiện các hoạt động liên quan đến mạng của Zeus. Mặc dù không có kết nối hoạt động cho các socket TCP 19705 và TCP 35335, điều này có thể chỉ là do kẻ tấn công không kết nối vào hệ thống vào thời điểm thu thập bộ nhớ hoặc hệ thống bị chặn bởi tường lửa và không thể tiếp cận từ Internet.

### Attributing Connections to Code

Mặc dù chúng ta đã giải quyết nhiều mảnh ghép của câu đố đến thời điểm này, vẫn còn một số câu hỏi chưa có câu trả lời. Ví dụ, mục đích của các socket TCP đang lắng nghe là gì? Chúng có cung cấp một shell điều khiển từ xa hoặc một SOCKS proxy mà kẻ tấn công có thể sử dụng để định tuyến kết nối thông qua máy bị nhiễm vào các hệ thống khác trong mạng nội bộ? Đây là những câu hỏi bạn phải trả lời bằng cách trích xuất mã độc hại từ bộ nhớ và phân tích nó tĩnh. Tuy nhiên, việc tìm đoạn mã chính xác đã khởi tạo một kết nối có thể phức tạp.

Chúng tôi đề xuất đầu tiên cố gắng xác định xem các tệp thực thi chính (đuôi .exe) có phải là độc hại không. Nếu đúng vậy, bạn có thể trích xuất quá trình (sử dụng plugin procdump) và bắt đầu phân tích đảo mã từ đó. Kiểm tra bảng địa chỉ nhập (IAT) và theo dấu chéo đến các API socket, connect và send. Thông thường, thủ tục này sẽ dẫn bạn thẳng đến các hàm xử lý mạng.

Nếu quá trình chính có vẻ hợp lệ (ví dụ: là explorer.exe hoặc svchost.exe), có thể nó đã bị tiêm mã. Trong trường hợp này, bạn có thể thực hiện quét qua bộ nhớ của quá trình để tìm các khối mã bị tiêm (sử dụng plugin malfind) hoặc để tìm các tiêu chí cụ thể liên quan đến kết nối cụ thể - như một phần của URL hoặc tên máy chủ DNS. Bạn có thể thực hiện các quét này với plugin yarascan, như được thể hiện trong đầu ra sau đây. Giả sử địa chỉ IP (XX.XXX.5.140) được trích xuất từ nhật ký tường lửa trên hệ thống mạng của máy nạn nhân.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223512.png)

Tìm kiếm chuỗi con XX.XXX.5.140 dẫn đến một trường hợp của XX.XXX.5.140:8080/ zb/v_01_a/in/ tại địa chỉ 0x5500e9ae, điều này chắc chắn trông giống như một phần của một URL. Nếu bạn thực hiện một truy vấn đảo ngược với dlllist, bạn sẽ nhận thấy phạm vi địa chỉ này nằm trong một DLL có tên là ab.dll bắt đầu từ địa chỉ 0x55000000.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223527.png)

Tại điểm này, bạn có thể trích xuất DLL với dlldump và phân tích nó theo cùng một cách như một tập tin thực thi. Tuy nhiên, không may, các chuỗi không luôn luôn ánh xạ lại với DLL. Tất cả phụ thuộc vào cách thiết kế của phần mã độc hại. Thay vì trồng một URL văn bản thô trong tệp nhị phân, nó có thể được giải mã vào thời điểm chạy và được sao chép vào heap hoặc một khối bộ nhớ được cấp phát ảo khác trong không gian tiến trình. Hãy xem xét ví dụ tiếp theo, trong đó bạn tìm thấy một URL thú vị tại địa chỉ 0x75d82438 chỉ bằng cách tìm kiếm http:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223540.png)

Sau khi tiến hành điều tra kỹ càng hơn, địa chỉ 0x75d82438 không nằm trong bất kỳ DLL nào đang được tải. Sự xuất hiện của URL tại vị trí này không cung cấp cho bạn thông tin nào hơn về mục đích thực sự của kết nối so với việc thấy địa chỉ IP tương ứng (XXX.134.176.126) trong kết quả của các plugin về sockets hoặc connections. Tuy nhiên, bạn vẫn có một số thông tin hữu ích. Và dù việc này có đòi hỏi tính kiên nhẫn, đôi khi bạn có thể đạt được thành công tốt bằng cách tìm kiếm các con trỏ tới địa chỉ đã tham chiếu. Trước khi làm điều đó, bạn phải chuyển đổi số nguyên 0x75d82438 thành các byte riêng lẻ và đảm bảo chúng ở đúng thứ tự cho hệ điều hành đích. Vì chúng ta đang điều tra trên Windows, thường chạy trên phần cứng little endian, tiêu chí tìm kiếm sẽ có dạng như sau:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223553.png)

Dựa vào kết quả, có vẻ như có một con trỏ tới địa chỉ 0x75d82438 được lưu tại địa chỉ 0x75d47500. Sau đó, bạn có thể giải mã mã lệnh xung quanh con trỏ đã lưu để xem nó được sử dụng như thế nào. Lưu ý rằng một số byte đã được trừ đi để hiển thị các lệnh trước và sau địa chỉ 0x75d47500.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223614.png)

Con trỏ tới URL (0x75d82438) đang được truyền làm tham số thứ hai cho hàm tại địa chỉ 0x75d4ace9. Vì vậy, bạn có thể giải mã hàm đó để xác định nơi và, quan trọng hơn, cách URL đang được sử dụng.

>**LƯU Ý**<br> Hãy lưu ý rằng bạn không phải luôn luôn tìm kiếm chỉ các chuỗi. Ví dụ, thay vì để "badsite.com" (chuyển thành địa chỉ IP 12.34.56.78) hiển thị trong chương trình và sau đó thực hiện một DNS lookup trong thời gian chạy, kẻ tấn công có thể mã hóa một số nguyên cứng vào chương trình. Đoạn mã sau cho thấy cách chuyển đổi địa chỉ IP từ chuỗi địa chỉ dạng dot-quad thành một số nguyên trong mạng-byte order.
```
$ python
>>> import socket
>>> import struct
>>> struct.unpack(">I", socket.inet_aton("12.34.56.78"))[0]
203569230
```
>Trong trường hợp này, bạn nên tìm kiếm trong bộ nhớ giá trị bốn byte là 203569230.

### Inactive Sockets and Connections

Thay vì duyệt các linked-lists trong không gian địa chỉ ảo (như các lệnh sockets và connections làm), các lệnh sockscan và connscan quét không gian vật lý của bộ nhớ để tìm kiếm các phân bổ kernel pool với các thẻ, kích thước và loại (paged hoặc nonpaged) phù hợp, như đã mô tả trong Chương 5. Do đó, bằng cách sử dụng connscan và sockscan, bạn có thể xác định các sockets và connections có thể đã được sử dụng trong quá khứ - vì bạn cũng tìm kiếm trong các khối bộ nhớ được giải phóng và hủy bỏ. Dưới đây là một ví dụ: (Tiếng Anh bị trích dẫn không rõ nghĩa, có thể bạn muốn xem xét lại nội dung này).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223646.png)

Có một mục cuối cùng có owning PID là số không (0), đây không phải là một số hợp lệ cho một định danh quá trình. Điều này không phải là công việc của một rootkit thay đổi PID thành số không hoặc bất cứ điều gì như vậy; đây là một cấu trúc dư thừa đã bị ghi đè một phần. Bạn có thể nhận biết rằng tại một thời điểm nó chứa thông tin hợp lệ vì địa chỉ IP nguồn, cổng nguồn, địa chỉ IP đích và cổng đích đều có vẻ hợp lệ. Có khả năng hoàn toàn lọc bỏ các PID không hợp lệ, nhưng điều đó sẽ làm mất mục đích của công cụ quét - và bạn sẽ bỏ qua một gợi ý có thể quan trọng rằng máy cục bộ đã liên hệ với một địa chỉ IP (12.206.53.84) trên cổng 443.

>LƯU Ý<br>
Trong một số trường hợp, nhiều trường trong kết quả đầu ra là không hợp lệ. Ví dụ, bạn có thể có một hoặc nhiều kết nối với PIDs và cổng không hợp lệ, nhưng địa chỉ IP là hợp lệ. Trong các trường hợp khác, các địa chỉ IP bị lỗi, nhưng các cổng và PIDs trông tốt. Một lần nữa, đây là sự đánh đổi của việc quét brute force thông qua các khối bộ nhớ đã được giải phóng và không được giải phóng so với đi bộ qua danh sách kết nối hoạt động (trong trường hợp này, tất cả các trường nên hợp lệ, nhưng bạn không có cơ hội phát hiện hoạt động trong quá khứ).<br>
Một cách để giảm thiểu một số tiếng ồn liên quan đến các trường không hợp lệ là rút ra danh sách địa chỉ IP của máy từ registry hoặc thu thập nó trong quá trình phản hồi trực tiếp bằng cách chạy lệnh ipconfig. Sau đó, điều chỉnh đầu ra của bạn để chỉ hiển thị các kết nối mà địa chỉ cục bộ hoặc từ xa nằm trong danh sách các IP.

## Hidden Connections

Bạn có nhiều cách để ẩn cổng lắng nghe và các kết nối hoạt động trên hệ thống thời gian thực. Bảng 11-1 tóm tắt một số khả năng và thảo luận về cách bạn có thể phát hiện chúng trong bộ nhớ bằng cách sử dụng Volatility.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-03%20165345.png)

### IP Packets and Ethernet Frames

Phần trước đã thảo luận về khả năng của các tác giả malware viết các trình điều khiển NDIS riêng của họ, từ đó tránh Winsock2 APIs và các tạo vật liên quan. Tuy nhiên, ngay cả trong trường hợp này, họ vẫn phải xây dựng các gói tin IP và các khung Ethernet trong RAM trước khi gửi chúng qua dây. Cả hai loại dữ liệu này phải tuân theo một tiêu chuẩn bao gồm việc sử dụng một tiêu đề có cấu trúc phổ biến và các giá trị hằng số có thể dự đoán (ví dụ: phiên bản IP, độ dài tiêu đề IP). Do đó, việc quét qua bộ nhớ và tìm các tiêu đề, thường là ngay sau đó là các tải liệu, khá dễ dàng.

Bản thực hiện sớm nhất là plugin có tên linpktscan (Linux packet scanner) được viết cho Thử thách Pháp lý DFRWS 2008 (xem http://sandbox.dfrws.org/2008/Cohen_Collet_Walters/Digital_Forensics_Research_Workshop_2.pdf). Plugin này tìm kiếm các gói tin IP có checksum hợp lệ, cho phép các tác giả định danh một số gói tin trở lại hệ thống mục tiêu, cụ thể là các gói mang các phần của các tệp zip và truyền tệp FTP đã bị đánh cắp.

Gần đây hơn, Jamaal Speights đã viết một plugin có tên ethscan (http://jamaaldev.blogspot.com/2013/07/ethscan-volatility-memory-forensics.html) để tìm các khung Ethernet và do đó các gói tin IP và tải liệu được bao gói. Dưới đây là ví dụ về cách chạy plugin với tùy chọn -C để lưu dữ liệu vào tập tin out.pcap, sau đó bạn có thể phân tích với công cụ bên ngoài như Wireshark hoặc Tcpdump.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223702.png)

Đầu ra sau đây cho thấy một yêu cầu DNS IPv6 đã khôi phục được từ hệ thống Linux. Bởi vì DNS (và bất kỳ lưu lượng nào, nói chung) là một hoạt động tương đối nhanh chóng, bạn khó có thể chụp bộ nhớ trong khi socket UDP đang hoạt động. Ngay cả khi bạn đã làm được điều đó, đầu ra của lệnh sockets không hiển thị cho bạn biết máy chủ tên miền được giải quyết là gì. Do đó, ethscan là một nguồn tài nguyên cực kỳ quý giá. Plugin đã khôi phục được máy chủ tên miền mong muốn: itXXXn.org.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223713.png)

>GHI CHÚ<br>
Ngoài ethscan của Volatility, có nhiều công cụ khác có thể lấy dữ liệu mạng từ các tệp nhị phân tùy ý như bộ nhớ được sao lưu. Dưới đây là một số ví dụ:
>- Bulk Extractor của Simson Garfinkel: https://github.com/simsong/bulk_extractor
>- The Network Appliance Forensic Toolkit (NAFT) của Didier Stevens: http://blog.didierstevens.com/2012/03/12/naft-release/
>- CapLoader của Netresec: http://www.netresec.com/?page=CapLoader

### DKOM Attacks

Các cuộc tấn công DKOM trực tiếp vào đối tượng Kernel không đe dọa bằng cách như vậy đối với các đối tượng socket và connection. Nghĩa là bạn có thể không thấy mã độc cố gắng gỡ bỏ hoặc ghi đè lên một _ADDRESS_OBJECT để ẩn một socket đang lắng nghe hoặc một _TCPT_OBJECT để ẩn một kết nối đang hoạt động. Trong quá trình thử nghiệm của chúng tôi, được trình bày trong Recipe 18-3 của Malware Analyst’s Cookbook, chúng tôi đã phát hiện rằng bạn không nên ghi đè lên những đối tượng này, nếu không, khả năng của một quá trình giao tiếp qua mạng sẽ bị hỏng.

Tuy nhiên, bạn có thể thực hiện DKOM lên dữ liệu không thiết yếu như các pool tag (mà tồn tại ngoài cấu trúc mục tiêu) để ẩn khỏi sockscan và connscan.

## Raw Sockets and Sniffers

Nếu một quy trình đang chạy với quyền quản trị viên, nó có thể bật raw sockets (xem http://msdn.microsoft.com/en-us/library/ms740548%28VS.85%29.aspx) từ chế độ người dùng bằng cách sử dụng API Winsock2. Raw sockets cho phép các chương trình truy cập dữ liệu lớp giao vận cơ bản (chẳng hạn như tiêu đề IP hoặc TCP), điều này có thể cho phép hệ thống làm giả hoặc giả mạo gói tin. Ngoài ra, mã độc có thể sử dụng raw sockets ở chế độ thu gom để bắt mật khẩu được truyền tải bởi máy nhiễm và các máy khác trên cùng mạng con.

>GHI CHÚ<br>
Hai yếu tố giảm thiểu rủi ro của raw sockets. Đầu tiên, bắt đầu từ Windows XP Service Pack 2, Windows ngăn các quy trình gửi dữ liệu TCP qua raw sockets và không cho phép các gói dữ liệu UDP được gửi bằng cách sử dụng địa chỉ nguồn không hợp lệ. Thứ hai, việc bắt các gói tin trên các mạng chuyển mạch hoặc kết nối không dây được mã hóa là khó (hoặc không thể thực hiện được).

### Creating Raw Sockets

Bạn có thể tạo một socket chế độ promiscuous bằng cách sử dụng Winsock2 với các bước sau đây:

1. Tạo một socket raw bằng cách chỉ định các cờ SOCK_RAW và IPPROTO_IP cho hàm socket:

        SOCKET s = socket(AF_INET, SOCK_RAW, IPPROTO_IP);

2. Đặt cổng là 0 khi khởi tạo cấu trúc sockaddr_in mà bạn truyền vào hàm bind. Trong trường hợp này, cổng 0 chỉ đơn giản là không cần thiết.

        struct sockaddr_in sa;
        struct hostent *host = gethostbyname(the_hostname);
        
        memset(&sa, 0, sizeof(sa));
        memcpy(&sa.sin_addr.s_addr,
            host->h_addr_list[in],
            sizeof(sa.sin_addr.s_addr));
        
        sa.sin_family = AF_INET;
        sa.sin_port = 0;
        
        bind(s, (struct sockaddr *)&sa, sizeof(sa));

3. Sử dụng hàm WSAIoctl hoặc ioctlsocket với cờ SIO_RCVALL để bật chế độ tiếp nhận mọi gói tin (hay còn gọi là "chế độ sniffing") cho NIC được liên kết với socket:

        int buf;
        
        WSAIoctl(s, SIO_RCVALL, &buf, sizeof(buf),
                        0, 0, &in, 0, 0);

### Detecting Raw Sockets

Trên máy Windows đang hoạt động, bạn có thể sử dụng công cụ có tên promiscdetect (xem http:// ntsecurity.nu/toolbox/promiscdetect/) để phát hiện sự hiện diện của một card mạng trong chế độ nhận mọi gói tin (promiscuous mode). Để phát hiện chúng trong một bản sao trích xuất bộ nhớ, bạn có thể sử dụng các lệnh sockets hoặc handles trong Volatility. Bạn thậm chí không cần một plugin đặc biệt! Các dấu vết còn lại trong bộ nhớ từ việc thực hiện ba bước trước đó sẽ nổi bật như một vết loét. Hãy xem liệu bạn có nhận ra quy trình với raw socket trong bản sao trích xuất bộ nhớ này của một hệ thống bị nhiễm Gozi (còn được gọi là Ordergun và UrSniff) hay không.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223741.png)

Dễ dàng quá! Tóm lại, các tiến trình mở các raw socket, có hoặc không có chế độ promiscuous, sẽ có một socket được gắn với cổng 0 của giao thức 0 và một handle mở đến \Device\RawIp\0.

## Next Generation TCP/IP Stack

Bắt đầu từ Vista và Windows Server 2008, Microsoft giới thiệu Next Generation TCP/IP Stack (xem http://technet.microsoft.com/en-us/network/bb545475.aspx). Mục tiêu chính của nó là tăng cường hiệu suất cho cả IPv4 và IPv6. Để làm được điều này, hầu hết kernel module tcpip.sys đã được viết lại hoàn toàn; và do những thay đổi mạnh mẽ đó, cách chúng ta khôi phục các artifacts liên quan đến mạng từ bộ nhớ cần phải thích ứng.

**Cấu trúc dữ liệu**

Những biến _AddrObjTable và _TCBTable mà trước đây trỏ tới đầu các cấu trúc sockets và connections hoạt động đã bị loại bỏ. Ngoài ra, Microsoft đã thiết kế lại và đổi tên các cấu trúc socket và connection, và chuyển đổi các nhãn kernel pool cho các phân bổ lưu trữ chúng. Đoạn kết quả sau đây thể hiện các cấu trúc mạng mà Volatility định nghĩa cho các hệ thống 64-bit Windows 7: 

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223756.png)

Phần lớn các mô tả thành viên phù hợp với các cấu trúc được miêu tả trước đây cho Windows XP và 2003.

### Working Backward from netstat.exe

Dù có thay đổi gì từ một phiên bản Windows sang phiên bản khác, điều mà bạn có thể chắc chắn là netstat.exe sẽ luôn hoạt động trên một máy tính sống. Do đó, xác định nơi mà netstat.exe thu thập thông tin là một bước khởi đầu tốt để tìm các hiện vật liên quan đến mạng trong bộ nhớ. Đây là cách chúng tôi thực hiện khi phát triển khả năng của Volatility để tìm kiếm các cấu trúc socket và kết nối trong các bản sao bộ nhớ từ hệ điều hành Vista và các phiên bản sau đó.

Cụ thể, chúng tôi đã đảo ngược mã của các API và các mô-đun (cả từ chế độ người dùng và chế độ kernel) liên quan đến việc tạo ra hoạt động mạng trên hệ thống đang chạy. Mọi thứ bắt đầu khi netstat.exe gọi InternetGetTcpTable2 từ iphlpapi.dll. Luồng thực thi dẫn về tcpip.sys trong một hàm có tên là TcpEnumerateAllConnections. Để biết thêm thông tin về cách chúng tôi theo dõi các mối quan hệ này, hãy xem http://mnin.blogspot.com/2011/03/volatilitys-new-netscan-module.html.

### Volatility’s Netscan Plugin

Sau khi xác định nguồn thông tin chính xác mà netstat.exe in trên máy sống, chúng tôi đã có thể xây dựng một plugin của Volatility để truy cập trực tiếp vào dữ liệu trong RAM. Khả năng này được thực hiện bằng plugin netscan. Nó sử dụng phương pháp quét pool (xem Chương 5) để xác định các cấu trúc _TCP_ENDPOINT, _TCP_LISTENER và _UDP_ENDPOINT trong bộ nhớ. Dưới đây là ví dụ về đầu ra của nó trên một máy Windows 7 64-bit:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223812.png)

Trong đầu ra, một số dòng hiển thị dấu gạch ngang (-) thay vì địa chỉ IP cục bộ hoặc từ xa. Dấu gạch ngang cho biết thông tin không thể truy cập được trong bộ nhớ bị rò rỉ. Khác với cấu trúc của XP và 2003 lưu thông tin địa chỉ IP trong cấu trúc thực tế, các cấu trúc của Vista và phiên bản sau lưu trữ con trỏ đến con trỏ. Do đó, để truy cập dữ liệu, Volatility phải giải tham chiếu nhiều con trỏ trong bộ nhớ ảo - một đường dẫn thường dễ bị hỏng nếu một hoặc nhiều trang trên đường đi được đẩy vào đĩa.

>GHI CHÚ<br>
Plugin netscan sử dụng cùng phương pháp quét pool-tag như sockscan và connscan. Như đã thảo luận trước đó trong chương, điều này có thể dẫn đến kết quả sai và dữ liệu không hợp lệ vì bạn đang quét các khối bộ nhớ đã giải phóng và hủy bỏ trong không gian vật lý. Xem lại các ghi chú trước đó về cách giảm thiểu kết quả sai (tức là lọc theo địa chỉ IP) và triển khai kết nối trở lại mã để xác minh.

>GHI CHÚ<br>
The Next Generation TCP/IP stack hỗ trợ dual-stack sockets. Nghĩa là khi bạn tạo một socket trên hệ thống Vista và sau này, nó áp dụng cho cả IPv4 và IPv6 trừ khi bạn yêu cầu rõ ràng chỉ IPv4 khi tạo. Do đó, netscan có thể báo cáo các kết nối cho cả hai giao thức.

### Partition Tables

Một trong những cách mà Microsoft tăng cường hiệu suất trong TCP/IP stack đã được thiết kế lại là bằng cách chia công việc thành nhiều lõi xử lý. Một biến toàn cục trong module tcpip.sys có tên là PartitionTable lưu trữ một con trỏ tới một cấu trúc _PARTITION_TABLE, chứa một mảng các _PARTITION. Số lượng chính xác của các phân vùng này phụ thuộc vào số lõi CPU tối đa mà hệ thống có thể hỗ trợ. Trong quá trình khởi động cho module tcpip.sys, một hàm có tên TcpStartPartitionModule cấp phát bộ nhớ cho các cấu trúc phân vùng và khởi tạo chúng. Được cho rằng, mỗi lõi xử lý đảm nhận việc xử lý các kết nối trong phân vùng của nó; và khi một tiến trình hoặc trình điều khiển yêu cầu kết nối, chúng được thêm vào phân vùng có tải nhẹ nhất.

Hình 11-4 thể hiện cách phân tích thông tin kết nối dựa trên dữ liệu trong bảng phân vùng.

Một _PARTITION chứa ba cấu trúc _RTL_DYNAMIC_HASH_TABLE – một cho các kết nối ở các trạng thái sau: thiết lập, SYN gửi (đang chờ đợi đầu xa xác nhận) và time wait (sắp trở thành đã đóng). Các bảng băm động trỏ tới danh sách kép các cấu trúc kết nối, như _TCP_ENDPOINT. Do đó, khá dễ dàng để bắt đầu từ một biến đã biết (tcpip!PartitionTable) trong bộ nhớ và thu thập tất cả thông tin kết nối hiện tại.

>**GHI CHÚ**<br>
Có thể khiến bạn ngạc nhiên, nhưng số lượng phân vùng phụ thuộc vào số lượng bộ xử lý tối đa, chứ không phải số lượng bộ xử lý hoạt động (ví dụ, một hệ thống có thể hỗ trợ lên đến 16 CPU, nhưng chỉ có một CPU được cài đặt). Chúng ta biết điều này vì hàm tcpip!TcpStartPartitionModule hoạt động theo cách sau đây:
>![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20223827.png)
