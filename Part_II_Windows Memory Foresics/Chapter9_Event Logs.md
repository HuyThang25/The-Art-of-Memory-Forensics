Bản ghi sự kiện chứa rất nhiều thông tin pháp lý và là một phần không thể thiếu trong hầu hết các loại điều tra. Chúng chứa thông tin về lỗi ứng dụng (như khi Word bị crash sau khi sử dụng kỹ thuật heap-spray), đăng nhập tương tác và từ xa, thay đổi chính sách tường lửa và các sự kiện khác đã xảy ra trên hệ thống. Kết hợp với dấu thời gian được cung cấp với mỗi sự kiện, các bản ghi sự kiện có thể giúp bạn xác định chính xác những gì đã xảy ra trên một hệ thống, hoặc ít nhất là cung cấp một khung thời gian để tập trung vào các nỗ lực còn lại của bạn.

Chương này trình bày cách tìm kiếm bản ghi sự kiện trong RAM và phân tích chúng cho mục đích pháp lý. Nhiều tệp nhật ký được ánh xạ vào bộ nhớ trong thời gian chạy của hệ thống, do đó, thông thường, bạn có thể tìm thấy hàng trăm, thậm chí hàng ngàn bản ghi cá nhân trong các bản sao bộ nhớ. Trong một số trường hợp, bạn thậm chí có thể trích xuất các mục nhập sau khi được đánh dấu để xóa bởi một quản trị viên hoặc bị xóa một cách độc hại bởi kẻ tấn công.

## Event Logs in Memory

Bởi vì các bản ghi sự kiện được ghi lại trong suốt thời gian chạy của hệ thống, nên điều này hợp lý khi bạn sẽ tìm thấy những bản ghi này, hoặc thậm chí các tệp nhật ký sự kiện, trong bộ nhớ. Để tìm kiếm các bản ghi hoặc tệp nhật ký sự kiện, bạn cần biết cấu trúc của chúng - chúng trông như thế nào và nằm ở đâu một cách nhất quán - bởi vì phương pháp tìm kiếm và trích xuất chúng khác nhau tùy thuộc vào hệ điều hành mục tiêu.

**Mục tiêu**

-	 Xác định vị trí của các nhật ký sự kiện trong bộ nhớ: Bắt đầu từ phiên bản Vista, Microsoft đã thực hiện một số thay đổi quan trọng đối với cơ sở dữ liệu nhật ký sự kiện. Để làm việc với các nhật ký sự kiện trên các phiên bản Windows khác nhau, bạn cần hiểu và xem xét các khác biệt này.
-	 Xử lý các nhật ký sự kiện: Học cách chỉ cần phân tích các bản ghi từ bộ nhớ bằng Volatility và khi nào cần trích xuất các tệp nhật ký để phân tích bằng các công cụ bên ngoài.
-	 Phát hiện các cuộc đăng nhập bằng cách thử mật khẩu: Tìm hiểu cách xác định các nỗ lực đăng nhập bằng cách thử mật khẩu thông qua việc phân tích các thông báo lỗi trong nhật ký sự kiện.
-	 Xác định các cánh cửa sau: Phát hiện các thay đổi về chính sách tường lửa và xung đột cổng máy chủ, điều này thường cho thấy hoạt động của cánh cửa sau.
-	 Xác định các nhật ký sự kiện bị xóa: Học cách nhận biết những gì xảy ra khi nhật ký sự kiện bị xóa và làm thế nào để xác định liệu mã độc hoặc kẻ tấn công đã cố gắng xóa các nhật ký hay không.

**Cấu trúc dữ liệu**

Đầu ra bên dưới hiển thị một số thành viên được chọn của các cấu trúc nhật ký sự kiện trên Windows XP và 2003. Harlan Carvey cũng đã ghi chép các cấu trúc nhật ký sự kiện trên blog của ông (http://windowsir.blogspot.com/2007/06/eventlog-analysis.html). EVTLogHeader là cấu trúc được sử dụng làm tiêu đề tệp nhật ký sự kiện, trong khi EVTRecordStruct dành cho các bản ghi cá nhân.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-01%20235114.png)

**Các điểm chính**

- OldestID: ID của bản ghi nhật ký sự kiện cũ nhất trong tệp.
- Magic: Chữ ký của một nhật ký sự kiện và các bản ghi của nó. Chữ ký này là LfLe.
- NextID: ID của bản ghi nhật ký sự kiện tiếp theo sẽ được ghi vào.
- RecordLength: Độ dài của bản ghi nhật ký sự kiện, có thể thay đổi tùy thuộc vào loại bản ghi, vì các chuỗi thông báo khác nhau cho mỗi loại thông báo.
- RecordNumber: ID của bản ghi trong nhật ký sự kiện.
- TimeGenerated: Dấu thời gian chỉ thời điểm sự kiện được tạo ra (UTC).
- TimeWritten: Dấu thời gian chỉ thời điểm bản ghi sự kiện được ghi vào (UTC).
- EventID: ID mô tả loại sự kiện đã xảy ra.
- NumStrings: Số lượng thông điệp được bao gồm trong bản ghi giúp mô tả sự kiện. Bạn có thể cần tìm các mẫu thông điệp tương ứng để giải thích đúng ý nghĩa của các chuỗi. Để biết thêm thông tin về cách tìm các mẫu, xem ghi chú ở cuối phần "Nhật ký sự kiện Windows 2000, XP và 2003" sau trong chương.

### Windows 2000, XP, and 2003 Event Logs

Trong Windows 2000, XP và 2003, các nhật ký sự kiện có cùng định dạng bản ghi nhị phân. Các nhật ký mặc định là Ứng dụng (Application), Hệ thống (System) và Bảo mật (Security), và vị trí lưu trữ mặc định trên ổ đĩa cho các nhật ký này là %systemroot%\system32\config. Các nhật ký sự kiện này cũng được ánh xạ vào không gian địa chỉ của tiến trình services.exe.

#### Finding Event Log Files

Bạn có thể xác định chính xác đoạn bộ nhớ nào chứa dữ liệu tệp bằng cách tìm tiến trình services.exe và tìm kiếm tên tệp nhật ký sự kiện (đuôi .Evt), như được thể hiện trong đầu ra sau đây.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-01%20235132.png)

Sau khi tìm thấy nhật ký sự kiện trong bộ nhớ, bạn có thể tiến hành tiếp theo bằng cách trích xuất các phần VAD (Virtual Address Descriptor) và phân tích chúng bằng một công cụ ngoài Volatility. Tuy nhiên, cũng có thể bạn chỉ cần phân tích trực tiếp nhật ký sự kiện từ bộ nhớ mà không cần trích xuất riêng biệt.

>**LƯU Ý:**<br> Nhật ký sự kiện trong Windows XP/2003 có kích thước tối đa và được xem như một vòng lặp. Điều này có nghĩa là khi đạt đến kích thước tối đa, con trỏ ghi vào nhật ký sẽ quay lại bản ghi cũ nhất và ghi đè lên nó với dữ liệu mới. Phương pháp này có thể làm hỏng một số công cụ vì các bản ghi một phần có thể làm hỏng những gì công cụ mong đợi thấy, ngoài ra còn có khả năng một số phần của tệp nhật ký có thể không có sẵn do trang trí.

#### Extracting Event Logs

Plugin "evtlogs" (chỉ áp dụng cho Windows XP và 2003) tìm kiếm và phân tích tự động các bản ghi nhật ký sự kiện cho bạn. Nó cũng được thiết kế để xử lý các nhật ký sự kiện bị hỏng với dữ liệu bị thiếu hoặc bị ghi đè. Tiện ích này hoạt động bằng cách tìm trước tiên tiến trình services.exe, sau đó tìm kiếm trong bộ nhớ của nó các nhật ký sự kiện. Sau đó, nó chia mỗi nhật ký thành các phần dựa trên chữ ký của nó (LfLe) và phân tích cú pháp từng bản ghi sử dụng các cấu trúc được xác định ở đầu chương này. Tiện ích "evtlogs" cũng có tùy chọn để trích xuất nhật ký gốc nếu bạn muốn xử lý nó bằng các công cụ ngoài Volatility. Bên dưới là ví dụ đầu ra khi chạy tiện ích này trên một mẫu bộ nhớ công cộng (http://sempersecurus.blogspot.com/2011/04/ using-volatility-to-study-cve-2011-6011.html) được thu thập sau khi truy cập vào tệp Word có chứa lỗ hổng Flash nhúng:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-01%20235146.png)

Vì ví dụ này sử dụng tùy chọn --save-evt, tiện ích đã tạo một tệp .evt (nhật ký sự kiện nhị phân gốc) và một tệp .txt (các bản ghi đã được phân tích dưới dạng văn bản) cho mỗi tệp nhật ký. Đầu ra của tệp văn bản đã được phân tích có định dạng như sau:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-01%20235917.png)

Chú ý rằng có một bản ghi với cấp độ Warning và ID 7003, điều này có nghĩa là ứng dụng Microsoft Office đã kết thúc một cách không đáng kể. Thông tin này liên quan đến cuộc điều tra vì vector lây nhiễm là lỗ hổng Flash được nhúng. Trong Chương 18, bạn sẽ học cách tạo bảng thời gian và xem cách sự kiện này kết hợp với các sự kiện khác tạo nên một bức tranh rõ ràng về cuộc tấn công.

#### Logging Policies

Mặc định, nhật ký sự kiện Security đã bị tắt trong Windows XP. Do đó, bạn nên kiểm tra các thiết lập kiểm toán trong registry (HKLM\SECURITY\Policy\PolAdtEv) để xác nhận loại sự kiện cần kiểm tra. Lệnh sau hiển thị cách thực hiện kiểm tra bằng cách sử dụng plugin auditpol (S có nghĩa là "ghi lại các hoạt động thành công," và F có nghĩa là "ghi lại các hoạt động thất bại"):

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-01%20235930.png)

>**Lưu ý**<br>
Những tài liệu tham khảo tốt để tra cứu các mã số sự kiện và chuỗi thông báo trong nhật ký sự kiện là như sau:<br>
http://go.microsoft.com/fwlink/events.asp<br>
http://www.eventid.net/<br>
http://blogs.msdn.com/b/ericfitz/<br>
http://www.ultimatewindowssecurity.com/securitylog/encyclopedia/Default.aspx

### Windows Vista, 2008, and 7 Event Logs

Nhật ký sự kiện của Windows Vista, 2008 và 7 (chúng ta sẽ gọi là "Evtx," dựa trên phần mở rộng của chúng) được lưu trữ trong định dạng tệp hoàn toàn khác so với những tệp đã được miêu tả trong phần trước. Cụ thể, các nhật ký này được chứa trong định dạng nhị phân XML. Trên máy tính thông thường, bạn có thể tìm thấy hơn 60 nhật ký như vậy trên ổ đĩa tại đường dẫn %systemroot%\system32\ winevt\Logs. Số lượng nhật ký lớn như vậy tăng cơ hội tìm thấy các bản ghi quan trọng trên máy tính bị xâm nhập. Hơn nữa, các chuỗi mô tả được chứa trong nhật ký sự kiện này (khác với các nhật ký XP/2003), điều này làm cho việc điều tra chúng dễ dàng hơn mà không cần truy cập vào ổ đĩa của máy tính mục tiêu.

>**LƯU Ý:**<br> Để tìm các ID bảo mật tương đương cho các máy Windows Vista, 2008 và 7, hãy thêm số 4096 vào ID được sử dụng cho máy Windows XP/2003. Ví dụ, để tìm các sự kiện liên quan đến ai đó đăng nhập vào máy tính chạy Windows 7, ID quan trọng sẽ là 4624 thay vì 528. Để biết thêm thông tin, hãy xem trang web http://blogs.msdn.com/b/ericfitz/archive/2007/04/18/vista-security-events-get-noticed.aspx.

Những nhật ký sự kiện mới hơn không được ánh xạ trong bộ nhớ theo cùng cách như các nhật ký cũ. Do đó, phương pháp xử lý nhật ký này hoàn toàn khác nhau - bạn phải trích xuất các nhật ký từ bộ nhớ bằng cách sử dụng plugin dumpfiles (xem Chương 16) và sau đó phân tích chúng bằng một công cụ bên ngoài Volatility. Bạn có thể chọn phương pháp tiếp cận cụ thể (tìm và trích xuất nhật ký sự kiện mục tiêu) hoặc bạn có thể chọn trích xuất tất cả các nhật ký sự kiện bằng cách sử dụng khả năng phù hợp mẫu (biểu thức chính quy) của plugin dumpfiles. Phương pháp tiếp cận của bạn phụ thuộc vào những gì bạn đang tìm kiếm và bao nhiêu ngữ cảnh bạn đã nhận được trước khi điều tra. Ví dụ, nếu bạn đang cố gắng chứng minh rằng có ai đó đã đăng nhập vào máy tính, bạn có thể chỉ xem xét nhật ký sự kiện Bảo mật. Tuy nhiên, nếu bạn không chắc chắn vấn đề cơ bản là gì với máy tính, bạn có thể trích xuất tất cả các nhật ký sự kiện.

>**LƯU Ý**<br>
Một số công cụ mà chúng tôi đã sử dụng trong quá khứ để phân tích các nhật ký sự kiện Evtx bao gồm:<br>
>- Evtxparser: http://computer.forensikblog.de/en/2011/11/evtx-parser-1-1-1.html
>- EVTXtract: https://github.com/williballenthin/EVTXtract
>- Python-evtx: http://www.williballenthin.com/evtx/index.html
>- Libevtx: http://code.google.com/p/libevtx/
Dự án EVTXtract rất hữu ích, vì nó sẽ cố gắng sử dụng các mẫu thông báo đã biết để tái tạo lại các thông báo bị hỏng hoặc bị thiếu. Nói cách khác, tác giả của công cụ đã cố gắng xử lý dễ dàng các tệp được trích xuất từ bộ nhớ với dữ liệu bị thiếu do phân trang.

Để trích xuất tất cả các tệp Evtx từ một mẫu bộ nhớ, bạn có thể sử dụng lệnh sau:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-01%20235952.png)

Tất cả các tệp đã được trích xuất sẽ nằm trong thư mục đầu ra của bạn. Sau đó, bạn có thể sử dụng tiện ích "file" trên Linux để xác minh rằng các nhật ký đã được trích xuất:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000030.png)

Một số tệp được trích xuất có thể hiển thị "data" thay vì "MS Windows Event Vista Event Log" vì tiêu đề cho nhật ký hoặc nhật ký chính nó đã bị tráo đổi khỏi bộ nhớ vào thời điểm thu thập. Trong cả hai trường hợp, bạn nên điều tra các tệp này để xem xem có thể tìm thấy các bản ghi một phần. Dưới đây là cách điều tra một trong các nhật ký này bằng cách sử dụng tiện ích evtxdump.pl từ Evtxparser:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000140.png)

Như bạn có thể thấy, kết quả đầu ra chứa tất cả thông tin về các sự kiện đã được tạo ra. Đặc biệt, bạn có thể thấy các trường sau:
- Tên nhà cung cấp (Provider Name): Cho biết từ nhật ký nào thông tin này được trích xuất.
- EventID: Chứa ID của sự kiện mà bạn có thể tra cứu trực tuyến để tìm hiểu sự kiện đã xảy ra.
- TimeCreated: Một dấu thời gian cho biết khi sự kiện được tạo ra.
- EventRecordID: ID của bản ghi, giúp bạn xác định thứ tự của các bản ghi được tạo ra.

### Caveats About Event Logs in Memory

Như đã đề cập trước đó, mặc dù các bản ghi sự kiện được tìm thấy trong bộ nhớ, tuy nhiên không phải toàn bộ tệp nhật ký sẽ có sẵn trong hầu hết các trường hợp. Điều này có nghĩa là bạn có thể có các bản ghi liên quan đến vụ việc trong nhật ký sự kiện trên đĩa cứng, nhưng không có trong bộ nhớ. Như với hầu hết các dấu vết trong bộ nhớ, các bản ghi nhật ký sự kiện có khả năng giữ lại nếu chúng được tạo hoặc truy cập gần đây. Điều này là một tin tức tốt nếu bạn thu thập bộ nhớ gần thời điểm xâm nhập, nhưng ngược lại thì không tốt.
Ngoài ra, kẻ thù có thể xóa sạch các nhật ký sự kiện trên máy sử dụng hàm ClearEventLog. Hàm này nhận một handle của nhật ký sự kiện và một vị trí sao lưu làm tham số. Nếu không cung cấp vị trí sao lưu (tức là nó là NULL), nhật ký sự kiện sẽ được xóa sạch khỏi đĩa cứng và bộ nhớ. Định nghĩa hàm lấy từ MSDN (http://msdn.microsoft.com/en-us/library/windows/desktop/aa363637%28v=vs.85%29.aspx) được hiển thị trong đoạn mã sau: 

```
BOOL ClearEventLog(
  _In_ HANDLE hEventLog,
  _In_ LPCTSTR lpBackupFileName
);
```

>**LƯU Ý**<br> Đây là một ví dụ về kịch bản Visual Basic (nguồn gốc từ http://technet.microsoft.com/library/ee176696.aspx) mà chúng tôi đã sửa đổi để xóa nhật ký Bảo mật. Các dòng bắt đầu bằng một dấu nháy đơn là các chú thích trong Visual Basic:<br>
>![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000157.png)
>Nếu bạn đặt mã trước vào một tệp có tên clearevt.vbs, bạn có thể chạy nó như sau:<br>
>![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000213.png)

Bởi vì bạn phải chỉ định từng nhật ký riêng lẻ mà bạn muốn xóa, nên những kẻ tấn công đôi khi quên xóa các nhật ký quan trọng, như bạn sẽ thấy trong một số ví dụ cuối chương này. Tuy nhiên, có các giải pháp để xóa tất cả các nhật ký (xem http://blogs.msdn.com/b/jjameson/archive/2011/03/01/script-to-clear-and-save-event-logs.aspx). Tuy nhiên, hành động này làm cho việc có vấn đề gì đó không bình thường trở nên rõ ràng nếu đột nhiên tất cả các nhật ký đều trống hoặc nếu ai đó nhận thấy có một đợt tăng vọt trong hoạt động của máy khi xóa những nhật ký này. Hầu hết những kẻ tấn công mà chúng tôi đã gặp thường không xóa tất cả các nhật ký sự kiện.

Chỉ vì nhật ký sự kiện bị xóa không có nghĩa là bạn nên nản lòng. Nếu nhật ký Bảo mật trống hoặc thưa thớt và bạn biết rằng sự kiện nên được đăng nhập vì bạn đã sử dụng plugin auditpol, điều đó đã cho thấy nhật ký sự kiện đã bị xóa. Nếu bạn có một hình ảnh đĩa pháp y tế từ máy nghi ngờ, bạn có thể lấy các bản ghi nhật ký sự kiện hữu ích từ điểm khôi phục, bản sao bóng bên, hoặc ngay cả từ không gian chưa được cấp phát.

Nếu bạn không có hình ảnh đĩa pháp y tế, bạn có thể quét không gian địa chỉ vật lý của bản sao dump bằng cách sử dụng chữ ký LfLe (Evt) hoặc ElfChnk (Evtx), và bạn có thể may mắn đủ để khôi phục các bản ghi lịch sử. Willi Ballenthin đã viết các công cụ mà bạn có thể sử dụng ngoài Volatility cho cả hai trường hợp; chúng được gọi là LfLe (https://github.com/williballenthin/LfLe) và EVTXtract (https://github.com/williballenthin/EVTXtract).

>**LƯU Ý**<br> Việc xóa một nhật ký sự kiện thực sự tạo ra các hiện vật mới. Ví dụ, một sự kiện mới (ID 517) sẽ được thêm vào nhật ký Bảo mật để chỉ ra rằng nhật ký Bảo mật đã được xóa. Điều này hữu ích vì nó cũng chứa dấu thời gian khi nhật ký bị xóa:<br>
>![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000235.png)
>Sau đây là thông báo đã được tái tạo:
>![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000251.png)

## Real Case Examples

Các bản ghi sự kiện được trích xuất từ bộ nhớ thường mang lại nhiều kết quả hữu ích trong cuộc điều tra của chúng tôi. Trong các ví dụ trong phần này, một số thông tin như địa chỉ IP và ngày tháng đã được xóa để bảo vệ danh tính của các nạn nhân.

### The Case of the Unsuccessful Listener

Kẻ tấn công thường thích cài đặt các cửa sau trên các máy đã bị xâm nhập. Điều này giúp họ dễ dàng kết nối vào máy vào bất kỳ thời điểm nào mà họ muốn. Đầu ra sau đây là từ plugin evtlogs trên sự kiện nhật ký Security của máy nghi ngờ, cho thấy những cố gắng thất bại của các ứng dụng cố gắng thiết lập các cổng lắng nghe (sự kiện ID 861):

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000306.png)

Trong trường hợp này, các sự kiện này đáng ngờ vì chương trình thực thi (wauclt.exe) không tồn tại trên máy tính Windows theo mặc định. Tên chương trình này đã được chọn để giống như một tiến trình hợp lệ, wuauclt.exe, dành cho các cập nhật Windows (chú ý u bị thiếu). Ngoài ra, nếu ứng dụng liên quan là notepad.exe (mà không nên mở các cổng), các nhật ký có thể cho thấy có sự tiêm mã. Bạn có thể xây dựng chuỗi thông báo hoàn chỉnh bằng cách tìm mẫu thông báo trực tuyến (http://www.eventid.net/display.asp?eventid=861&eventno=4615&source=Security&phase=1):

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000320.png)

### The Case of the Unsuccessful Logon

Như bạn đã học, các sự kiện đăng nhập và đăng xuất được thu thập trong nhật ký sự kiện Bảo mật (Security). Trong ví dụ sau, từ một máy chủ Windows 2003 bị xâm nhập, bạn thấy hai ID sự kiện 529 (thử đăng nhập không thành công) và một ID sự kiện 680. ID sự kiện 680 thay đổi tùy thuộc vào hệ điều hành của máy, nhưng theo dõi các đăng nhập NTLM thành công và không thành công cho các máy Windows 2003.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000334.png)

Nếu bạn tra cứu mẫu thông báo cho ID sự kiện 529 (một số liên kết được trích dẫn trước đó trong chương), bạn sẽ có thêm bối cảnh về sự kiện này. Chuỗi thông báo đầy đủ hiển thị trong trình xem nhật ký sự kiện sẽ là:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000347.png)

Bạn có thể thấy rằng có ai đó đã cố gắng đăng nhập với tư cách là người quản trị qua mạng (Loại Đăng nhập: 3), và địa chỉ IP từ xa đến từ Trung Quốc (222.186.XX.XX). Vì máy nạn nhân nằm ở Hoa Kỳ và công ty sở hữu không có nhân viên đặt tại Trung Quốc, có thể nói đây không phải là một lần thử đăng nhập hợp lệ. Nếu bạn xem xét bản ghi sự kiện khác có ID 680 cùng cách, bạn sẽ có thêm bối cảnh về lý do tại sao đăng nhập này không thành công:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000405.png)

Các mã lỗi cho sự kiện này được tài liệu rõ ràng (http://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=680). Bạn có thể thấy rằng 0xC000006A có nghĩa là kẻ tấn công đã sử dụng tên người dùng hợp lệ trên hệ thống, nhưng mật khẩu không chính xác. Loại sự kiện này thể hiện một trong ba trường hợp sau:
- Tài khoản là tài khoản mặc định cho máy
- Kẻ tấn công đã tiến hành một số thăm dò trước đó trên hệ thống
- Kẻ tấn công đang thử đoán thông tin xác thực trên hệ thống bằng cách dùng thử đoán mật khẩu (brute force)

Trong ví dụ này, tài khoản "administrator" là tài khoản mặc định. Tuy nhiên, nếu tên tài khoản là gstanley và bạn chỉ thấy một vài lần đăng nhập thất bại (không phải là một cuộc tấn công dùng thử đoán mật khẩu), bạn sẽ biết chắc chắn rằng các kẻ tấn công đã thực hiện công việc nghiên cứu cẩn thận trước đó.

### The Case of the Impatient Brute Forcer

Trong nhiều trường hợp, các kẻ tấn công thường không có thông tin xác thực hợp lệ cho máy tính khi gặp nó lần đầu tiên. Trong những trường hợp đó, như chúng ta vừa mô tả, họ có thể thử đoán mật khẩu để truy cập vào máy tính bằng cách sử dụng một ứng dụng thử nhiều kết hợp tên người dùng và mật khẩu. Các attemps này cũng xuất hiện trong các sự kiện log và có thể cho thấy máy tính đã bị nhắm mục tiêu cũng như xác định xem kẻ tấn công có thành công trong việc tiếp cận hay không. Trong đoạn mã sau, bạn có thể thấy dữ liệu sự kiện log mô tả loại cuộc tấn công này trên máy chủ Windows 2003:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000419.png)

Chú ý rằng các cố gắng đăng nhập không thành công được thực hiện nhanh chóng liên tiếp nhau - trong trường hợp này, hai lần cố gắng mỗi giây. Tổng cộng có hơn 600 bản ghi như vậy xảy ra, tất cả trong một khoảng thời gian ngắn. Địa chỉ IP (đã che giấu) cũng được xác định không có lý do để kết nối vào máy tính mục tiêu.

Bạn cũng có thể tìm thấy dấu vết của cuộc tấn công brute force trong sự kiện log của hệ thống. Ví dụ, đầu ra sau đây cho thấy một cuộc tấn công vào một hệ thống chạy dịch vụ FTP của Microsoft.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000433.png)


Một lần nữa, bạn thấy sáu cố gắng đăng nhập mỗi giây sử dụng tên người dùng bị che giấu. Dưới đây là ví dụ về thông báo đã được xây dựng lại:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000446.png)

### The Case of the Log Wiper

Như đã đề cập trước đó, kẻ tấn công thường cố gắng xóa các nhật ký sự kiện để che dấu dấu vết trên một máy tính. Chúng tôi đã gặp một trường hợp trong đó một kẻ tấn công đăng nhập vào một máy tính bằng các chứng chỉ đã bị đánh cắp và bắt đầu một công việc trên một máy tính khác bằng các chứng chỉ đã đánh cắp khác. Mục tiêu của chúng tôi là tìm ra cả hai tài khoản bị xâm nhập để vô hiệu hóa chúng. Kẻ tấn công thông minh đủ để xóa các nhật ký sự kiện bảo mật cũng như một số nhật ký khác có thể giúp chúng tôi. Hơn nữa, chúng tôi không có ổ cứng để tìm kiếm không gian không cấp phát cho các bản ghi bị mất. May mắn thay, kẻ tấn công không nhận ra rằng có một nhật ký sự kiện khác (Microsoft-Windows-TaskScheduler.evtx) chứa thông tin chúng tôi cần. Nhật ký này tình cờ nằm trong bộ nhớ và có một bản ghi trực tiếp trỏ đến tài khoản mà chúng tôi cần vô hiệu hóa:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-08-02%20000458.png)

## Tổng quan

Trong quá trình điều tra, các bản ghi nhật ký sự kiện của Windows thường đóng một vai trò quan trọng trong việc tái tạo các sự kiện của một sự cố. Chúng có thể cung cấp thông tin quý giá về lịch sử về những gì đã xảy ra và khi nào đã xảy ra. Trong khi các nhà điều tra thường tập trung vào các bản ghi họ tìm thấy trên đĩa cứng, bộ nhớ dễ bay hơi cung cấp nguồn tài nguyên quý báu khác cho các hiện vật này. Hiểu cách trích xuất và phân tích các bản ghi cư trú trong bộ nhớ qua các phiên bản khác nhau của Windows cung cấp một khả năng mạnh mẽ cho các nhà điều tra số hóa.
