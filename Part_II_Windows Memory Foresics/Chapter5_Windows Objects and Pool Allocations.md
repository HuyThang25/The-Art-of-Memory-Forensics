Tất cả các hiện vật mà bạn tìm thấy trong bộ nhớ đều chia sẻ một nguồn gốc chung: Chúng đều bắt đầu dưới dạng một phân bổ. Cách, thời điểm và lý do mà các vùng bộ nhớ được phân bổ làm cho chúng khác nhau, ngoài dữ liệu thực tế được lưu trữ bên trong và xung quanh chúng. Từ góc độ pháp y bộ nhớ, nghiên cứu các đặc điểm này có thể giúp bạn đưa ra các suy luận về nội dung của một phân bổ, giúp bạn tìm và đánh dấu các loại dữ liệu cụ thể trong toàn bộ bộ nhớ lớn. Hơn nữa, việc làm quen với các thuật toán của hệ điều hành để phân bổ và giải phóng bộ nhớ có thể giúp bạn hiểu ngữ cảnh của dữ liệu khi bạn tìm thấy nó - ví dụ, liệu nó có đang được sử dụng hay được đánh dấu là miễn phí.

Chương này giới thiệu cho bạn các khái niệm về các đối tượng quản trị (executive) Windows, phân bổ bộ nhớ kernel và quét thẻ pool. Cụ thể, bạn sẽ sử dụng kiến thức này để tìm các đối tượng (như quy trình, tệp và trình điều khiển) bằng cách sử dụng một phương pháp độc lập với cách hệ điều hành liệt kê các đối tượng. Do đó, bạn có thể vượt qua các rootkit cố gắng ẩn mình bằng cách thao túng các cấu trúc dữ liệu nội bộ của hệ điều hành. Hơn nữa, bạn có thể xác định các đối tượng đã được sử dụng nhưng từ đó đã bị loại bỏ (nhưng chưa bị ghi đè), cung cấp thông tin quý giá về các sự kiện đã xảy ra trong quá khứ.

## Windows Executive Objects

Rất nhiều việc pháp y bộ nhớ liên quan đến việc tìm kiếm và phân tích các đối tượng quản trị (executive). Trong Chương 2, bạn đã học rằng Windows được viết bằng C và sử dụng mạnh các cấu trúc C để tổ chức dữ liệu và thuộc tính liên quan. Một số trong số các cấu trúc này được gọi là các đối tượng quản trị vì chúng được quản lý (tạo, bảo vệ, xóa, v.v.) bởi Trình quản lý đối tượng Windows - một thành phần của kernel được triển khai bởi module NT.

Một cấu trúc kỹ thuật thuộc về các đối tượng quản trị khi hệ điều hành thêm vào các tiêu đề khác nhau để quản lý các dịch vụ như đặt tên, kiểm soát truy cập và đếm tham chiếu. Do đó, theo định nghĩa này, tất cả các đối tượng quản trị đều là các cấu trúc, nhưng không phải tất cả các cấu trúc đều là các đối tượng quản trị. Phân biệt giữa hai loại này là quan trọng vì do được phân bổ bởi Trình quản lý đối tượng, tất cả các đối tượng quản trị sẽ có các đặc điểm tương tự. Ví dụ, tất cả các đối tượng quản trị đều có các tiêu đề đã nêu, trong khi các cấu trúc được phân bổ bởi các hệ thống con khác như gói TCP/IP (tcpip.sys) sẽ không có các tiêu đề như vậy.

Một số loại đối tượng quản trị quan trọng về pháp y được mô tả trong Bảng 5-1, cùng với tên cấu trúc tương ứng của chúng. Bạn sẽ làm quen với các loại đối tượng này trong suốt quá trình đọc sách, vì vậy chỉ xem đây là một giới thiệu ban đầu ngắn gọn. Trong thực tế, có ít nhất một plugin của Volatility phân tích từng đối tượng được liệt kê trong bảng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20134940.png)

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20140742.png)

>**LƯU Ý**<br> Các loại đối tượng quản trị khác nhau trên các phiên bản Windows khác nhau vì các đối tượng mới thường được yêu cầu để hỗ trợ các tính năng mới, và các đối tượng cũ trở nên lỗi thời. Để xem danh sách đầy đủ các loại đối tượng cho một phiên bản cụ thể của Windows bằng cách sử dụng một công cụ giao diện người dùng đồ họa (GUI), bạn có thể sử dụng WinObj từ SysInternals (http://technet.microsoft.com/en-us/sysinternals/bb896657.aspx). Sau này trong chương, bạn sẽ học cách thực hiện điều này một cách tự động bằng cách sử dụng Volatility.

### Object Headers

Một trong những đặc điểm chung thường gặp giữa tất cả các loại đối tượng quản trị là sự hiện diện của một tiêu đề đối tượng (_OBJECT_HEADER) và một hoặc nhiều tiêu đề tùy chọn. Tiêu đề đối tượng nằm ngay trước cấu trúc đối tượng quản trị trong bộ nhớ. Tương tự, bất kỳ tiêu đề tùy chọn nào tồn tại đều nằm trước tiêu đề đối tượng theo một thứ tự cố định. Điều này dẫn đến một bố cục bộ nhớ dễ dự đoán, như thể hiện trong Hình 5-1. Do đó, việc tìm cấu trúc (ví dụ, _FILE_OBJECT trong hình) dựa trên địa chỉ của _OBJECT_HEADER tương ứng, hoặc ngược lại, rất đơn giản vì hai phần này luôn luôn liền kề nhau; và kích thước của _OBJECT_HEADER nhất quán cho mỗi hệ điều hành.

Khả năng xác định tiêu đề tùy chọn nào có sẵn (và nếu có, các offset tương ứng từ đầu của tiêu đề đối tượng) dựa vào thành viên InfoMask của tiêu đề đối tượng. Trước khi chuyển sang thảo luận về điều này, hãy xem cấu trúc đầy đủ cho một tiêu đề đối tượng trên Windows 7 64-bit.

```
>>> dt("_OBJECT_HEADER")
'_OBJECT_HEADER' (56 bytes)
0x0  : PointerCount         ['long long']
0x8  : HandleCount          ['long long']
0x8  : NextToFree           ['pointer64', ['void']]
0x10 : Lock                 ['_EX_PUSH_LOCK']
0x18 : TypeIndex            ['unsigned char']
0x19 : TraceFlags           ['unsigned char']
0x1a : InfoMask             ['unsigned char']
0x1b : Flags                ['unsigned char']
0x20 : ObjectCreateInfo     ['pointer64', ['_OBJECT_CREATE_INFORMATION']]
0x20 : QuotaBlockCharged    ['pointer64', ['void']]
0x28 : SecurityDescriptor   ['pointer64', ['void']]
0x30 : Body                 ['_QUAD']
```

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20141628.png)

>Key Points<br>
Note the following key points:<br>
>-	 PointerCount: Contains the total number of pointers to the object, including kernelmode references.
>-	 HandleCount: Contains the number of open handles to the object.
>-	 TypeIndex: This value tells you what type of object you’re dealing with (e.g., process, thread, file).
>-	 InfoMask: This value tells you which of the optional headers, if any, are present.
>-	 SecurityDescriptor: Stores information on the security restrictions for the object,
such as which users can access it for reading, writing, deleting, and so on.
>-	 Body: This member is just a placeholder that represents the start of the structure
contained within the object.

#### Optional Headers

Các tiêu đề tùy chọn của một đối tượng chứa các loại dữ liệu siêu dữ liệu khác nhau để mô tả đối tượng. Rõ ràng, vì chúng là tùy chọn, không phải tất cả các loại đối tượng đều có chúng; và ngay cả các phiên bản khác nhau của cùng một loại đối tượng có thể có các kết hợp khác nhau của tiêu đề tùy chọn. Ví dụ, hệ điều hành không theo dõi thống kê hạn ngạch (sử dụng tài nguyên) cho các tiến trình Idle hoặc System, do đó hai đối tượng _EPROCESS này sẽ không có tiêu đề _OBJECT_HEADER_QUOTA_INFO. Ngoài ra, một mutex chỉ cần một tên nếu nó được chia sẻ giữa nhiều tiến trình. Do đó, các mutex có tên sẽ có tiêu đề _OBJECT_HEADER_NAME_INFO, trong khi mutex không có tên sẽ không có. Mặc dù nhiều tiêu đề tùy chọn có thể hữu ích cho mục đích pháp lý, nhưng tiêu đề tên là tiêu đề mà các nhà điều tra thường phân tích nhiều nhất.

Bảng 5-2 hiển thị các tiêu đề tùy chọn có sẵn trên các hệ thống Windows 7 64-bit. Nếu giá trị trong cột Bit Mask được thiết lập trong _OBJECT_HEADER.InfoMask của đối tượng, thì tiêu đề tùy chọn tương ứng sẽ xuất hiện. Nếu bạn nhớ cấu trúc trong Hình 5-1, các tiêu đề tùy chọn nằm ở các offset âm từ đầu của _OBJECT_HEADER. Khoảng cách chính xác phụ thuộc vào các tiêu đề khác có sẵn và kích thước của chúng (được hiển thị trong cột Kích thước của Bảng 5-2).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20142054.png)

>**Lưu ý**<br> Bắt đầu từ Windows 8 và Server 2012, đã giới thiệu một tiêu đề tùy chọn mới chứa thông tin kiểm toán (_OBJECT_HEADER_AUDIT_INFO) với bit mask 0x40. Để biết thêm thông tin về các thay đổi trong định dạng tiêu đề đối tượng giữa các phiên bản hệ điều hành, hãy xem trang web http://www.codemachine.com/article_objectheader.html.

### Object Type Objects

Hãy quay lại cái bảng mà tôi đưa ra ở Chương 5 và kiểm tra xem TypeIndex value tương ứng với loại đối tượng nào. Điều này giúp bạn xác định loại đối tượng mà theo sau _OBJECT_HEADER. Ví dụ, các mục của bảng xử lý (process handle table entries) trỏ đến các tiêu đề đối tượng (object headers). Do đó, khi bạn liệt kê các mục trong bảng xử lý, dữ liệu mà theo sau tiêu đề là tùy ý - nó có thể là _FILE_OBJECT, _EPROCESS, hoặc bất kỳ đối tượng thực thi khác. Bạn có thể phân biệt giữa các khả năng bằng cách xem TypeIndex value, xác định _OBJECT_TYPE tương ứng với chỉ số đó, và sau đó đánh giá thành viên Name. Tham khảo bảng ở Chương 5 để biết sự tương ứng giữa tên loại đối tượng và tên cấu trúc của chúng.

**Data Structures**

On a 64-bit Windows 7 system, the pool header looks like this:

```
>>> dt("_POOL_HEADER")
'_POOL_HEADER' (16 bytes)
0x0 : BlockSize                 ['BitField', {'end_bit': 24,
                                  'start_bit': 16, 'native_type': 'unsigned long'}]
0x0 : PoolIndex                 ['BitField', {'end_bit': 16,
                                  'start_bit': 8, 'native_type': 'unsigned long'}]
0x0 : PoolType                  ['BitField', {'end_bit': 32,
                                  'start_bit': 24, 'native_type': 'unsigned long'}]
0x0 : PreviousSize              ['BitField', {'end_bit': 8,
                                  'start_bit': 0, 'native_type': 'unsigned long'}]
0x0 : Ulong1                    ['unsigned long']
0x4 : PoolTag                   ['unsigned long']
0x8 : AllocatorBackTraceIndex   ['unsigned short']
0x8 : ProcessBilled             ['pointer64', ['_EPROCESS']]
0xa : PoolTagHash               ['unsigned short']
```
**Key Points**

Các điểm quan trọng là:
- BlockSize: Tổng kích thước của phân bổ, bao gồm pool header, object header và bất kỳ optional headers nào.
- PoolType: Loại bộ nhớ hệ thống (paged, nonpaged, v.v.) mà pool header này mô tả.
- PoolTag: Một giá trị 4 byte, thường được tạo bởi các ký tự ASCII, nên định danh độc nhất con đường mã được sử dụng để tạo ra phân bổ (để có thể theo dõi lại các khối dữ liệu gây rối đến nguồn gốc của chúng). Trên các hệ thống trước Windows 8 và Server 2012, một trong các ký tự có thể được sửa đổi để đặt "bit bảo vệ" (bạn có thể đọc thêm thông tin về điều này trong ghi chú gần Bảng 5-3).

### Allocation APIs

Trước khi tạo một phiên bản của một executive object (hoặc bất kỳ object nào), một khối bộ nhớ đủ lớn để lưu trữ object và các header của nó phải được cấp phát từ một trong các pool của hệ điều hành. Một giao diện lập trình ứng dụng (API) thường được sử dụng cho mục đích này là ExAllocatePoolWithTag. Prototype của hàm như sau:

```
PVOID ExAllocatePoolWithTag(
    _In_ POOL_TYPE PoolType,
    _In_ SIZE_T NumberOfBytes,
    _In_ ULONG Tag
);
```

Đối số PoolType xác định loại bộ nhớ hệ thống được sử dụng cho việc cấp phát. Giá trị NonPagedPool (0) và PagedPool (1) là các giá trị liệt kê cho bộ nhớ không có thể trang và bộ nhớ có thể trang tương ứng. Như đã hiển thị trước đó, hầu hết, nhưng không phải tất cả, các loại executive object được cấp phát bằng bộ nhớ không có thể trang - và bạn luôn có thể phân biệt bằng cách xem vào thành viên _OBJECT_TYPE.TypeInfo.PoolType của một object cụ thể.

>**LƯU Ý**<br> Có nhiều cờ khác nhau bạn có thể chỉ định để điều khiển việc bộ nhớ có thể thực thi, được căn chỉnh bộ nhớ cache và nhiều thuộc tính khác. Để biết thêm thông tin, hãy xem tại địa chỉ http://msdn.microsoft.com/en-us/library/windows/hardware/ff559707(v=vs.85).aspx.

Tham số NumberOfBytes chứa số byte cần được cấp phát. Các trình điều khiển gọi ExAllocatePoolWithTag trực tiếp có thể đặt tham số này là kích thước dữ liệu họ cần lưu trữ trong khối bộ nhớ. Nhưng các executive object khác biệt, như bạn đã học, vì chúng yêu cầu không gian bổ sung để lưu trữ object headers và optional headers. Một hàm trong kernel được gọi là ObCreateObject là điểm trung tâm từ đó tất cả các executive object được tạo ra. Nó xác định kích thước của cấu trúc được yêu cầu (ví dụ: 1232 byte cho một _EPROCESS trên 64-bit Windows 7) và thêm kích thước của _OBJECT_HEADER và bất kỳ optional headers nào cần có trước khi gọi ExAllocatePoolWithTag.

Tham số Tag chỉ định một giá trị bốn byte, thường được tạo thành từ các ký tự ASCII, để định danh duy nhất đoạn mã được sử dụng để cấp phát (nhằm theo dõi các khối bộ nhớ gây rối đến nguồn gốc của chúng). Trong trường hợp của executive object, các tag được xuất phát từ _OBJECT_TYPE.Key—điều này giải thích tại sao Tag là giống nhau cho tất cả các đối tượng cùng loại.

Giả sử một tiến trình muốn tạo một tệp mới bằng cách sử dụng Windows API, các bước sau sẽ xảy ra:

1. Tiến trình gọi CreateFileA (ASCII) hoặc CreateFileW (Unicode)—cả hai đều được xuất ra từ kernel32.dll.
2. Các API tạo file chuyển đến ntdll.dll, sau đó gọi vào kernel và đến hàm NtCreateFile native.
3. NtCreateFile sẽ gọi ObCreateObject để yêu cầu một loại đối tượng File mới.
4. ObCreateObject tính toán kích thước của _FILE_OBJECT, bao gồm không gian bổ sung cần thiết cho optional headers của nó.
5. ObCreateObject tìm _OBJECT_TYPE structure cho các đối tượng File và xác định xem có cần phải cấp phát bộ nhớ có trang hay không trang, cũng như tag bốn byte để sử dụng.
6. Gọi ExAllocatePoolWithTag với kích thước, loại bộ nhớ và tag phù hợp.
