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











