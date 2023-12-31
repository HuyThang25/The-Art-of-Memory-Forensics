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

>Lưu ý các điểm quan trọng sau đây:
>- PointerCount: Chứa tổng số con trỏ đến đối tượng, bao gồm cả các tham chiếu ở chế độ kernel.
>- HandleCount: Chứa số lượng handles mở đến đối tượng.
>- TypeIndex: Giá trị này cho bạn biết bạn đang làm việc với loại đối tượng gì (ví dụ, quy trình, luồng, tập tin).
>- InfoMask: Giá trị này cho bạn biết các tiêu đề tùy chọn, nếu có, có hiện diện hay không.
>- SecurityDescriptor: Lưu trữ thông tin về các hạn chế bảo mật cho đối tượng, chẳng hạn như người dùng nào có thể truy cập nó để đọc, viết, xóa và vân vân.
>- Body: Thành viên này chỉ là một bộ giữ chỗ đại diện cho phần bắt đầu của cấu trúc chứa trong đối tượng.

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

Sau các bước được liệt kê, một đối tượng _FILE_OBJECT mới xuất hiện trong bộ nhớ và phân bổ được đánh dấu bằng một thẻ bốn byte cụ thể. Điều này không phải là tất cả những gì xảy ra, tất nhiên - một con trỏ đến tiêu đề đối tượng được thêm vào bảng quản lý xử lý của quy trình gọi, một cơ sở dữ liệu theo dõi thẻ hệ thống rộng được cập nhật tương ứng và các thành viên cá nhân của _FILE_OBJECT được khởi tạo với đường dẫn đến tệp được tạo và quyền truy cập yêu cầu (ví dụ, đọc, ghi, xóa).

>**GHI CHÚ:**<br> Các phân bổ tuần tự sử dụng cùng kích thước, thẻ và loại bộ nhớ không nhất thiết sẽ nằm kề nhau trong bộ nhớ. Mặc dù hệ điều hành cố gắng nhóm các phân bổ cùng kích thước lại với nhau, nếu không có khối trống có kích thước yêu cầu, hạt nhân sẽ chọn một khối từ nhóm kích thước lớn kế tiếp. Kết quả là, bạn có thể thấy các phân bổ chứa cùng loại đối tượng phân tán khắp trong pool. Hơn nữa, điều này cho phép các cấu trúc nhỏ chiếm các khối bộ nhớ trước đó được sử dụng để lưu trữ các cấu trúc lớn hơn - tạo ra tình trạng "slack space" nếu bộ nhớ không được xoá sạch.<br>Để biết thêm thông tin, xem tại Andreas Schuster's The Impact of Windows Pool Allocation Strategies on Memory Forensics: http://dfrws.org/2008/proceedings/p58-schuster.pdf

#### De-allocation and Reuse

Tiếp tục với ví dụ của một quy trình tạo một tập tin mới, thời gian tồn tại (tức là hiện diện trong bộ nhớ vật lý) của _FILE_OBJECT tương ứng sẽ phụ thuộc vào các yếu tố khác nhau. Yếu tố quan trọng nhất duy nhất là quy trình báo hiệu (bằng cách gọi CloseHandle) rằng nó đã hoàn tất việc đọc hoặc viết tập tin mới. Lúc này, nếu không có quy trình nào khác đang sử dụng đối tượng tập tin, khối bộ nhớ sẽ được giải phóng trở lại "danh sách tự do" của pool, nơi nó có thể được phân bổ lại cho mục đích khác. Trong khi chờ được phân bổ lại, hoặc bất kỳ lúc nào trước khi có dữ liệu mới được ghi vào khối bộ nhớ, hầu hết _FILE_OBJECT ban đầu sẽ vẫn giữ nguyên.

Thời gian mà khối bộ nhớ kéo dài trong trạng thái này phụ thuộc vào mức hoạt động của hệ thống. Nếu máy đang bị quá tải, và kích thước các khối được yêu cầu nhỏ hơn hoặc bằng kích thước _FILE_OBJECT, nó sẽ bị ghi đè nhanh chóng. Nếu không, đối tượng có thể tồn tại trong vài ngày hoặc vài tuần - lâu sau khi quy trình tạo tập tin đã kết thúc. Trong quá khứ, học viên thường hỏi về điều này: Sau khi một kết nối mạng được đóng, thời gian nào là tối thiểu cần thiết để thu thập bộ nhớ để bảo tồn chứng cứ? Câu trả lời là không thể dự đoán và có thể thay đổi theo máy hoặc thậm chí thời điểm trong ngày.

Để làm rõ thêm, khi các khối bộ nhớ pool được giải phóng, chúng chỉ được đánh dấu là miễn phí, không được ghi đè ngay lập tức. Cùng một khái niệm áp dụng cho pháp trích xuất dữ liệu trên đĩa. Khi một tệp NTFS bị xóa, chỉ có mục bảng điều khiển tệp Master File Table (MFT) được sửa đổi để phản ánh trạng thái đã thay đổi. Nội dung tệp vẫn không được chạm đến cho đến khi các sector được chỉ định lại cho một tệp mới và các hoạt động ghi diễn ra. Kết quả của hành vi này là bạn có thể tìm thấy các đối tượng điều hành (hoặc bất kỳ phân bổ bộ nhớ nào, trong trường hợp này) trong RAM sau khi chúng đã bị loại bỏ bởi hệ điều hành. Điều này mang lại cho bạn một cái nhìn độc đáo vào hệ thống không chỉ bao gồm danh sách các đối tượng điều hành đang sử dụng mà còn các tài nguyên đã tồn tại trong quá khứ.

## Pool-Tag Scanning

Pool-tag scanning, hoặc đơn giản là quét pool, đề cập đến việc tìm các phân bổ dựa trên các thẻ bốn byte đã đề cập ở trên. Ví dụ, để xác định các đối tượng quy trình, bạn có thể tìm ký hiệu kernel trỏ đến đầu của danh sách quy trình hoạt động kép và sau đó liệt kê các mục bằng cách duyệt qua danh sách. Một lựa chọn khác, quét pool, bao gồm tìm kiếm toàn bộ tệp bộ nhớ đổ trong Proc (thẻ bốn byte liên kết với _EPROCESS). Lợi thế của phương pháp sau là bạn có thể tìm thấy các mục đã tồn tại trong quá khứ (quy trình không còn chạy) cũng như đánh bại một số kỹ thuật ẩn rootkit, chẳng hạn như Manipulation Đối tượng Kernel Trực tiếp (DKOM), dựa trên việc thao tác danh sách đối tượng hoạt động.

Khi bạn thực hiện quét pool-tag, thẻ bốn byte chỉ là điểm bắt đầu. Nếu chỉ dựa vào thẻ này, sẽ có rất nhiều kết quả giả. Do đó, Volatility xây dựng một "chữ ký" mạnh mẽ hơn của những gì bộ nhớ xung quanh các phân bổ mong muốn trông như thế nào, và nó dựa trên thông tin đã được mô tả trước đó trong chương trình. Ví dụ, kích thước của phân bổ và loại bộ nhớ (paged, nonpaged) đóng vai trò quan trọng trong việc loại bỏ các kết quả giả. Nếu bạn đang tìm kiếm một _EPROCESS 100-byte và tìm thấy Proc trong một phân bổ 30-byte, nó không thể là một quy trình thực sự vì khối bộ nhớ quá nhỏ.


>**GHI CHÚ**<br> Ngoài các tiêu chí ban đầu (thẻ, kích thước, loại bộ nhớ), hạ tầng quét pool của Volatility cho phép bạn thêm ràng buộc tùy chỉnh cho mỗi loại đối tượng. Ví dụ, nếu mốc thời gian tạo quy trình không bao giờ là số không, bạn có thể cấu hình trình quét dựa trên thông tin đó. Bộ quét sẽ chỉ báo cáo các kết quả có thời gian dấu thời gian không phải là số không.

### Pool Tag Sources

Bảng 5-3 hiển thị các tiêu chí ban đầu mà Volatility sử dụng để tìm các đối tượng quản lý theo cơ chế quét pool. Kích thước tối thiểu cho bảng này được tính bằng cách cộng thêm kích thước của _EPROCESS (cho quy trình), _OBJECT_HEADER và _POOL_HEADER.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20194030.png)


Mặc dù dữ liệu trong Bảng 5-3 được thu thập bằng cách xem mã nguồn của Volatility, việc hiểu nơi lấy thông tin gốc là rất quan trọng. Ví dụ, bạn có thể cần điều chỉnh các trường khi có phiên bản mới của Windows được phát hành hoặc tạo một trình quét pool tìm một đối tượng quản lý mà các plugin hiện có của Volatility không bao gồm. Hơn nữa, nếu một trình điều khiển kernel độc hại cấp phát các pool để lưu trữ dữ liệu của nó (cấu hình, gói điều khiển và điều khiển lệnh, tên của các tài nguyên hệ thống để ẩn, vv), bạn sẽ cần một cơ chế để thu thập các tiêu chí để tìm các khối bộ nhớ.

Ngoài ra, hãy chú ý đến cột Tag(Protected) trong Bảng 5-3. Một trong những chi tiết ít được tài liệu hóa nhất về các pool tag là bit bảo vệ. Khi bạn giải phóng một pool với ExFreePoolWithTag, bạn phải cung cấp cùng một tag đã được cung cấp cho ExAllocatePoolWithTag. Đây là một kỹ thuật mà hệ điều hành sử dụng để ngăn các trình điều khiển giải phóng bộ nhớ theo vô tình. Nếu tag được chuyển đến hàm giải phóng không khớp, hệ thống sẽ gây ra một ngoại lệ. Điều này ảnh hưởng lớn đến phân tích hồ sơ bộ nhớ vì bạn thực sự cần tìm phiên bản được bảo vệ của pool tag.

>**Chú ý**<br> rằng không phải tất cả các phân bổ đều được thiết lập bit bảo vệ, chỉ đối với một số loại đối tượng quản lý. Hơn nữa, kể từ Windows 8 và Server 2012, dường như bit bảo vệ đã không còn tồn tại. Để biết thêm thông tin về bit bảo vệ, bạn có thể xem tại địa chỉ http://msmvps.com/blogs/windrvr/archive/2007/06/15/tag-you-re-it.aspx.

#### Pooltag File

Như đã đề cập trước đó, Microsoft đã tạo ra các pool tag cho mục đích gỡ lỗi và kiểm tra. Do đó, một số phiên bản của Windows Driver Development Kit (DDK) và Debugging Tools for Windows bao gồm một tệp pooltag.txt mà bạn có thể sử dụng để tra cứu. Ví dụ, dựa vào một pool tag, bạn có thể xác định mục đích của các phân bổ và driver kernel sở hữu. Vì tệp chứa các mô tả, bạn cũng có thể bắt đầu với các từ khóa chính như "process object" hoặc "file object," và tìm ra pool tag tương ứng. Dưới đây là một ví dụ về nội dung tệp pooltag.txt:

```
rem
rem Pooltag.txt
rem
rem This file lists the tags used for pool allocations by kernel mode components
rem and drivers.
rem
rem The file has the following format:
rem <PoolTag> - <binary-name> - <Description>
rem
rem Pooltag.txt is installed with Debugging Tools for Windows (%windbg%\triage)
rem and with the Windows DDK (in %winddk%\tools\other\platform\poolmon, where
rem platform is amd64, i386, or ia64).
rem

AdSv - vmsrvc.sys   - Virtual Machines Additions Service
ARPC - atmarpc.sys  - ATM ARP Client
ATMU - atmuni.sys   - ATM UNI Call Manager

[snip]

Proc - nt!ps        - Process objects *
Ps   - nt!ps        - general ps allocations

[snip]

RaDA - tcpip.sys    - Raw Socket Discretionary ACLs
RaEW - tcpip.sys    - Raw Socket Endpoint Work Queue Contexts
RaJP - tcpip.sys    - Raw Socket Join Path Contexts
RaMI - tcpip.sys    - Raw Socket Message Indication Tags
```

Dòng đánh dấu in đậm cho thấy rằng các đối tượng tiến trình sử dụng tag Proc và được phân bổ bởi nt!ps, đây là phần hệ thống tiến trình của module NT. Mặc dù thông tin này hữu ích, nhưng đó chỉ là điểm bắt đầu. Bây giờ bạn đã biết tag cho các đối tượng tiến trình, bạn vẫn phải tìm kích thước xấp xỉ của phân bổ và loại bộ nhớ (paged, nonpaged).

>**GHI CHÚ:**<br> Tập tin pooltag.txt chỉ chứa các tag được sử dụng bởi các thành phần chế độ kernel của Microsoft. Nó không bao gồm thông tin về các trình điều khiển của bên thứ ba hoặc độc hại. Tuy nhiên, có các cơ sở dữ liệu trực tuyến (ví dụ: http://alter.org.ua/docs/win/pooltag) mà cộng đồng có thể gửi các mô tả tag bộ nhớ. Hãy nhớ rằng thông tin trên các trang web này được gửi ẩn danh và có thể không chính xác.

#### PoolMon Utility

PoolMon (http://msdn.microsoft.com/en-us/library/windows/hardware/ff550442(v=vs.85).aspx) là một công cụ giám sát bộ nhớ chia theo dạng pool đi kèm với Bộ công cụ phát triển trình điều khiển (DDK) của Microsoft. Nó cung cấp thông tin cập nhật trực tiếp về các tag bộ nhớ đang được sử dụng trên hệ thống, kèm theo các thông tin sau:
- Loại bộ nhớ (Paged hoặc Nonpaged)
- Số lượng bản cấp phát
- Số lượng bản giải phóng
- Tổng số byte bị chiếm bởi các bản cấp phát
- Trung bình byte cho mỗi bản cấp phát

```
C:\WinDDK\7600.16385.1\tools\Other\i386> poolmon.exe -b
Memory: 2096696K Avail: 1150336K PageFlts: 8135 InRam Krnl: 5004K P:158756K
 Commit:1535208K Limit:4193392K Peak:2779016K Pool N:43452K P:187796K
 System pool information
 Tag    Type    Allocs          Frees           Diff        Bytes           Per Alloc
 
 CM31   Paged   169392 (   0)   153744 (   0)   15648   74838016 (    0)    4782
 MmSt   Paged   673616 (  16)   656049 (  17)   17567   28286672 ( -184)    1610
 MmRe   Paged    67417 (   0)    66213 (   0)    1204   12613400 (    0)    10476
 CM25   Paged     2404 (   0)        0 (   0)    2404   10678272 (    0)    4441
 NtfF   Paged   105073 (   0)    94672 (   0)   10401   10484208 (    0)    1008
 Cont   Nonp      2582 (   0)      251 (   0)    2331    9996936 (    0)    4288
 Ntff   Paged   426392 (   0)   418603 (   0)    7789    6978944 (    0)    896
 FMfn   Paged  5163318 (   0)  5145133 (   0)   18185    5632928 (    0)    309
 Pool   Nonp        16 (   0)       11 (   0)       5    4318792 (    0)    863758
 File   Nonp 126693501 ( 128)126675442 ( 132)   18059    3311048 ( -736)    183
 Ntfx   Nonp    563917 (   0)   545668 (   0)   18249    3059312 (    0)    167
 CIcr   Paged    68617 (   0)    67257 (   0)    1360    3026600 (    0)    2225
 MmCa   Nonp    617058 (  16)   601659 (  17)   15399    2197152 ( -120)    142
 vmmp   Nonp        19 (   0)       15 (   0)       4    2105480 (    0)    526370
 AWP6   Nonp        12 (   0)       10 (   0)       2    2007040 (    0)    1003520
 NtFs   Paged 13782497 (   9) 13760105 (   9)   22392    1963432 (    0)    87
 FSim   Paged   245313 (   0)   230460 (   0)   14853    1901184 (    0)    128
 FMsl   Nonp    555526 (   0)   537329 (   0)   18197    1892488 (    0)    104
 FIcs   Paged  2591469 (   0)  2573319 (   0)   18150    1887600 (    0)    104
 Ntfo   Paged  2726359 (   9)  2716000 (   9)   10359    1635912 (    0)    157
 [snip  ]
```

Như bạn có thể thấy, CM31 đứng đầu về số byte. Kể từ khi hệ thống khởi động, đã có 169,392 lần gọi ExAllocatePoolWithTag cho tag CM31, và trong số đó có 153,744 lần bị giải phóng. Điều này để lại sự khác biệt là 15,648 khối bộ nhớ hiện đang được cấp phát, chiếm tổng cộng 74,838,016 byte (khoảng 75MB) bộ nhớ. Trung bình, mỗi lần cấp phát là 4,782 byte.

Tag CM25 không quá xa sau tag CM31, sử dụng khoảng 10MB bộ nhớ. CM trong các tên tag này đều đại diện cho Configuration Manager, đây là thành phần kernel duy trì Windows Registry. Do đó, bạn có thể kết luận rằng ít nhất 85MB RAM được dành riêng cho việc lưu trữ các khóa và dữ liệu registry. Tuy nhiên, không nên kỳ vọng sẽ trích xuất toàn bộ 85MB này từ bản sao bộ nhớ, vì cả hai tag đều nằm trong bộ nhớ có phân trang, một số dữ liệu có thể được đẩy xuống đĩa.

Một điểm thú vị khác là về tag File (cấp phát chứa cấu trúc _FILE_OBJECT). Như được hiển thị bởi PoolMon, đã tạo ra hơn 126,000,000 _FILE_OBJECT kể từ lần khởi động cuối cùng, điều này hợp lý vì mỗi lần mở hoặc tạo file đều được cấp phát một _FILE_OBJECT. Tuy nhiên, vì _FILE_OBJECT khá nhỏ, 18,000 khối hiện đang được cấp phát chỉ tốn tổng cộng khoảng 3.5MB (trung bình 183 byte mỗi lần cấp phát).

Ngoài thông tin về cách hoạt động bên trong hệ điều hành, đầu ra từ PoolMon bổ sung thông tin từ pooltag.txt. Cùng nhau, chúng cung cấp cho bạn tag bể, mô tả, trình điều khiển kernel sở hữu, kích thước cấp phát và loại bộ nhớ - hầu như tất cả những gì bạn cần để bắt đầu quét các bản sao bộ nhớ để tìm các trường hợp cấp phát.


>**GHI CHÚ**<br>
Bộ gỡ lỗi kernel của Windows cũng có thể giúp bạn xác định các liên kết của các pool tag. Ví dụ, lệnh !poolfind thực hiện tìm kiếm theo tag mong muốn và cung cấp thông tin về loại bộ nhớ và kích thước. Bạn có thể sử dụng nó như sau:


```
  kd> !poolfind Proc
  Searching NonPaged pool (fffffa8000c02000 : ffffffe000000000) for Tag: Proc
  *fffffa8000c77000 size: 430 previous size: 0 (Allocated) Proc (Protected)
  *fffffa8001346000 size: 430 previous size: 0 (Allocated) Proc (Protected)
  *fffffa8001361000 size: 430 previous size: 0 (Allocated) Proc (Protected)
  *fffffa800138f7a0 size: 430 previous size: 30 (Free) Pro.
  *fffffa80013cb1e0 size: 430 previous size: c0 (Allocated) Proc (Protected)
  *fffffa80013e4460 size: 430 previous size: f0 (Allocated) Proc (Protected)
  *fffffa80014fd000 size: 430 previous size: 0 (Allocated) Proc (Protected)
  *fffffa800153ebd0 size: 10 previous size: 70 (Free) Pro.
  [snip]
```
>Kết quả đã được cắt giảm để tóm gọn, nhưng trong các kết quả được hiển thị, có sáu khối đang được cấp phát và hai khối được đánh dấu là miễn phí. Một trong những khối miễn phí có cùng kích thước với các khối được cấp phát (430), và khối còn lại nhỏ hơn nhiều (10). Do đó, có khả năng khối miễn phí tại 0xfffffa800138f7a0 chứa một _EPROCESS đã kết thúc; trong khi một phần của khối tại 0xfffffa800153ebd0 đã được sử dụng lại.

#### Pool Tracker Tables

Vì PoolMon được thiết kế để cung cấp các cập nhật thời gian thực về thay đổi trong việc sử dụng pool tag, nên nó chỉ hoạt động trên hệ thống thực. Nhưng nếu bạn chỉ có một bản sao bộ nhớ (memory dump)? May mắn thay, bộ nhớ thực tế chứa các số liệu thống kê mà PoolMon đọc. Chúng có thể truy cập từ cùng một khối dữ liệu bộ gỡ lỗi kernel (_KDDEBUGGER_DATA64) mà lưu trữ danh sách quy trình hoạt động và danh sách module được tải. Cụ thể, thành viên PoolTrackTable trỏ đến một mảng các cấu trúc _POOL_TRACKER_TABLE - một cấu trúc cho mỗi pool tag duy nhất được sử dụng. Dưới đây là cách các cấu trúc này xuất hiện trên hệ thống Windows 7 64-bit: (Hình 5-6)

```
>>> dt("_POOL_TRACKER_TABLE")
'_POOL_TRACKER_TABLE' (40 bytes)
0x0  : Key              ['long']
0x4  : NonPagedAllocs   ['long']
0x8  : NonPagedFrees    ['long']
0x10 : NonPagedBytes    ['unsigned long long']
0x18 : PagedAllocs      ['unsigned long']
0x1c : PagedFrees       ['unsigned long']
0x20 : PagedBytes       ['unsigned long long']
```

Như bạn có thể thấy, mỗi bảng theo dõi có một Key, đó là tag bốn byte. Các thành viên còn lại cho bạn biết có bao nhiêu phân bổ, giải phóng và tổng số byte được sử dụng cho cả bộ nhớ không được trang hóa và được trang hóa. Mặc dù thông tin không được cập nhật theo thời gian thực (điều này có ý nghĩa vì hệ thống không còn hoạt động nữa), nhưng bạn ít nhất có thể xác định trạng thái của nó vào thời điểm bản sao bộ nhớ được lấy. Dưới đây là một ví dụ về việc chạy plugin pooltracker và lọc một số tag được sử dụng bởi các executive objects. Các tên cột bắt đầu bằng "Np" cho bộ nhớ không được trang hóa hoặc "Pg" cho bộ nhớ được trang hóa: (Hình 5-7)

```
$ python vol.py -f win7x64.dd pooltracker
                   --profile=Win7SP0x64
                   --tags=Proc,File,Driv,Thre

Volatility Foundation Volatility Framework 2.4
Tag    NpAllocs NpFrees  NpBytes   PgAllocs PgFrees PgBytes
------ -------- -------- -------- -------- -------- --------
Thre     614895   614419   606688        0        0        0
File   75346601 75336591  3350912        0        0        0
Proc       4193     4154    51728        0        0        0
Driv        143        6    67504        0        0        0
```

Bạn có thể nhận ra rằng cả bốn tag đều ở trong bộ nhớ không được trang hóa vì ba cột (NpAllocs, NpFrees và NpBytes) không phải là số không. Để xác định kích thước xấp xỉ cho mỗi phân bổ, hãy chia số phân bổ hiện tại cho tổng số byte. Ví dụ, với tag Thre (Thread objects), kích thước trung bình là 606688 / (614895 - 614419) = 1274 byte. Do đó, để tìm tất cả các phân bổ chứa các threads, bạn sẽ tìm các tag Thre trong bộ nhớ không được trang hóa có ít nhất 1274 byte.


>**LƯU Ý**<br>
Dưới đây là một số điểm quan trọng về bảng theo dõi pool (pool tracker tables):
>- **Filtering options (Tùy chọn lọc):** Nếu bạn chạy plugin pooltracker mà không sử dụng --tags, nó sẽ hiển thị thống kê cho tất cả các tag pool.
>- **Verbose display (Hiển thị chi tiết):** Bạn có thể tích hợp dữ liệu từ tệp pooltag.txt (sử dụng tùy chọn --tagfile) để đánh dấu đầu ra với mô tả và tên của kernel driver sở hữu (nếu có sẵn).
>- **Supported systems (Hỗ trợ cho các hệ thống):** Windows chỉ bắt đầu ghi thống kê vào các bảng theo dõi pool theo mặc định sau phiên bản XP và 2003. Do đó, plugin pooltracker chỉ hoạt động với các hệ điều hành Vista và các phiên bản sau đó.
>- **Limitations (Giới hạn):** Bảng theo dõi pool chỉ ghi lại thống kê sử dụng; nó không ghi lại địa chỉ của tất cả các phân bổ của một tag cụ thể.

### Building a Pool Scanner

Như đã đề cập trước đó, tất cả các plugin Volatility được liệt kê trong Bảng 5-3 thực hiện phương pháp quét pool để tìm kiếm các đối tượng trong bộ nhớ. Framework cung cấp một lớp cơ sở PoolScanner mà bạn có thể mở rộng để tùy chỉnh hành vi của bộ quét (ví dụ: tag nào để tìm kiếm, kích thước của các phân bổ, loại bộ nhớ) và một lớp AbstractScanCommand cung cấp tất cả các tùy chọn dòng lệnh mà plugin quét pool sẽ cần.

#### Extending the PoolScanner

Đoạn mã sau đây cho thấy cấu hình cần thiết cho psscan (một công cụ quét pool để tìm kiếm các đối tượng quy trình):

```py
class PoolScanProcess(poolscan.PoolScanner):
    """Pool scanner for process objects"""
    def __init__(self, address_space, **kwargs):
        poolscan.PoolScanner.__init__(self, address_space, **kwargs)
        
        self.struct_name = "_EPROCESS"
        self.object_type = "Process"
        self.pooltag = obj.VolMagic(address_space).ProcessPoolTag.v()
        size = self.address_space.profile.get_obj_size("_EPROCESS")

        self.checks = [
         ('CheckPoolSize', dict(condition = lambda x: x >= size)),
         ('CheckPoolType', dict(non_paged = True, free = True)),
         ('CheckPoolIndex', dict(value = 0)),
         ]
```

Trên dòng đầu tiên, PoolScanProcess mở rộng PoolScanner, vì vậy nó kế thừa các chức năng được chia sẻ giữa tất cả các công cụ quét pool. Những điều chỉnh duy nhất mà bạn cần thực hiện để tìm các đối tượng quy trình là như sau:

- Tên cấu trúc: Ở dòng 7, tên cấu trúc được đặt là _EPROCESS. Điều này cho phép công cụ quét pool biết kiểu cấu trúc được chứa trong các phân bổ bộ nhớ.

- Loại đối tượng: Ở dòng 8, loại đối tượng quản lý (executive object) được đặt là "Process," đây là một cách thức bổ sung để xác nhận. Khi công cụ quét tìm thấy một phân bổ có khả năng, nó sẽ so sánh giá trị đã cung cấp với thành viên Name của _OBJECT_HEADER. Đối với các công cụ quét pool không chứa executive object (ví dụ: kết nối mạng và socket), loại đối tượng này sẽ không được đặt.

- Pool tag: Ở dòng 9, pool tag được đặt. Thay vì cứng đặt một giá trị như Proc, tag được lấy từ một container chuyên dụng cho từng profile. Điều này là cần thiết vì tag có thể thay đổi khi các phiên bản hệ điều hành mới được phát hành.

- Kích thước phân bổ: Ở dòng 10, kích thước tối thiểu của một phân bổ có thể chứa các đối tượng quy trình được tạo ra (dựa trên kích thước của _EPROCESS) được tạo ra. Điều này cũng không được cứng đặt vì kích thước thay đổi giữa các profile, đặc biệt là trên các hệ thống 32-bit và 64-bit. Ràng buộc về kích thước được áp dụng ở dòng 13.

- Loại bộ nhớ: Ở dòng 14, loại bộ nhớ hợp lệ được khai báo. Trong trường hợp này, công cụ quét sẽ tìm các phân bổ trong bộ nhớ không được trang và không được sử dụng. Nó sẽ bỏ qua các kết quả trong bộ nhớ được trang.

#### Extending AbstractScanCommand

Sau khi bạn đã thấy cách khởi tạo một bộ quét pool, bước tiếp theo là tạo một plugin để tải bộ quét và hiển thị các trường bạn mong muốn lên terminal. Dưới đây là một ví dụ về mã như sau:

```py
class PSScan(common.AbstractScanCommand):
    """Pool scanner for process objects"""

    scanners = [poolscan.PoolScanProcess]

    def render_text(self, outfd, data):
        self.table_header(outfd, [('Offset(P)', '[addrpad]'),
                                  ('Name', '16'),
                                  ('PID', '>6'),
                                  ('PPID', '>6'),
                                  ('PDB', '[addrpad]'),
                                  ('Time created', '30'),
                                  ('Time exited', '30')
                                  ])

        for pool_obj, eprocess in data:
            self.table_row(outfd,
                eprocess.obj_offset,
                eprocess.ImageFileName,
                eprocess.UniqueProcessId,
                eprocess.InheritedFromUniqueProcessId,
                eprocess.Pcb.DirectoryTableBase,
                eprocess.CreateTime or '',
                eprocess.ExitTime or '')
```

Như bạn có thể thấy ở dòng đầu tiên, tên lớp (PSScan) mở rộng từ AbstractScanCommand trở thành tên plugin. Ở dòng 4, plugin được liên kết với lớp PoolScanProcess. Hãy lưu ý rằng biến "scanners" có thể chứa nhiều lớp, nếu bạn muốn tìm các đối tượng khác nhau trong một lần chạy qua bộ nhớ (điều này tiết kiệm thời gian đáng kể). Còn về phần còn lại, các dòng 7-14 tạo tiêu đề bảng trong đầu ra văn bản, và các dòng 16-24 thêm một hàng vào bảng cho mỗi kết quả được tìm thấy bởi bộ quét. Trong trường hợp này, mỗi kết quả là một đối tượng _EPROCESS.

Bằng cách mở rộng AbstractScanCommand cơ bản, plugin mới của bạn được trang bị các tùy chọn dòng lệnh khác nhau cho phép người dùng điều chỉnh hành vi của bộ quét. Bạn sẽ thấy chúng ở cuối đầu ra từ --help, như được hiển thị bởi đoạn đầu ra sau đây:

```
$ python vol.py psscan –-help

[snip]

 -V, --virtual          Scan virtual space instead of physical
 -W, --show-unallocated
                        Show unallocated objects (e.g. 0xbad0b0b0)
 -S START, --start=START
                        The starting address to begin scanning
 -L LENGTH, --length=LENGTH
                        Length (in bytes) to scan from the starting address
---------------------------------
Module PSScan
---------------------------------
Pool scanner for process objects
```

Dưới đây là các mô tả về các tùy chọn:
- -V/--virtual: Khi quét các phân bổ pool, bạn có thể sử dụng không gian địa chỉ kernel ảo hoặc không gian địa chỉ vật lý. Theo mặc định, Volatility sử dụng không gian vật lý vì nó bao gồm càng nhiều bộ nhớ càng tốt - ngay cả các khối không nằm trong bảng trang của kernel hiện tại. Điều này cho phép bạn khôi phục các đối tượng từ "slack space" trong RAM. Để chuyển sang chế độ chỉ quét các trang hoạt động mà kernel hiện đang ánh xạ, sử dụng tùy chọn -V/--virtual.
- -W/--show-unallocated: Thiết lập này điều khiển xem liệu plugin có hiển thị các đối tượng mà hệ điều hành đánh dấu rõ ràng là không được phân bổ hay không. Để biết thêm thông tin, xem bài đăng trên blog của Andreas Schuster tại đây: http://computer.forensikblog.de/en/2009/04/0xbad0b0b0.html.
- -S/--start và –L/--length: Nếu bạn muốn quét chỉ một phạm vi bộ nhớ cụ thể thay vì toàn bộ bộ nhớ, bạn có thể chỉ định địa chỉ bắt đầu và độ dài mong muốn bằng cách sử dụng các tùy chọn này. Địa chỉ được xác định là một vị trí trong bộ nhớ vật lý hoặc ảo tùy thuộc vào việc có đặt cờ -V/--virtual hay không.

#### Pool Scanner Algorithm

Lớp cơ sở PoolScanner (và do đó bất kỳ bộ quét nào kế thừa nó) sử dụng logic được hiển thị trong Hình 5-3 để tạo kết quả.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20203905.png)

Nếu bạn quét bằng không gian địa chỉ vật lý, mã sẽ bắt đầu tìm kiếm tag pool 4 byte tại vị trí 0 của tệp dump bộ nhớ và tiếp tục cho đến khi đạt đến cuối tệp. Nếu bạn chọn không gian địa chỉ ảo, mã sẽ liệt kê và quét tất cả các trang trong bảng trang của kernel. Tất nhiên, nếu bạn đặt địa chỉ bắt đầu và độ dài cụ thể bằng cách sử dụng các tùy chọn dòng lệnh đã đề cập, điều đó sẽ ghi đè lên hành vi mặc định của thuật toán. Trước khi bộ quét trả về một địa chỉ, dữ liệu tại địa chỉ đó phải vượt qua tất cả các ràng buộc. Càng nhiều kiểm tra bạn có, càng giảm khả năng xuất hiện các kết quả sai.

#### Finding Terminated Processes


Một ví dụ về việc chạy plugin psscan với tất cả các tùy chọn mặc định được hiển thị trong kết quả sau đây:


![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20204252.png)

Kết quả hiển thị hai quá trình có thời gian kết thúc, điều này có nghĩa là chúng đã kết thúc. Cả hai quá trình ipconfig.exe và ping.exe chỉ thực thi trong một vài giây (hoặc ít hơn), vì vậy điều này là hợp lý khi chúng chỉ chạy trong một khoảng thời gian ngắn sau khi bắt đầu. Nếu bạn duyệt danh sách quá trình hoạt động của hệ điều hành, bạn sẽ không thấy bất kỳ quá trình nào vì cả hai đã kết thúc trước khi dump bộ nhớ được thực hiện. Tuy nhiên, bằng cách sử dụng kỹ thuật quét thẻ pool trong không gian vật lý, chúng ta đã tìm thấy dấu vết có thể hỗ trợ lý thuyết của bạn về việc một kẻ tấn công trên hệ thống đã tiến hành khảo sát mạng bằng hai tiện ích trên. 

Ví dụ này chỉ làm nổi bật một trong những trường hợp sử dụng phổ biến nhất của việc quét thẻ pool. Các chương sau sẽ tái khám phá chủ đề này và sử dụng quét thẻ pool để phát hiện sự hiện diện của rootkit cấp kernel.

## Limitations of Pool Scanning

Phương pháp quét thẻ pool cung cấp một cách mạnh mẽ để tìm kiếm các đối tượng mà không cần can thiệp hoặc hỗ trợ từ hệ điều hành mục tiêu. Tuy nhiên, nó cũng có một số hạn chế, mà bạn nên hiểu rõ trước khi rút ra kết luận dựa trên kết quả của các plugin quét thẻ pool.

### Non-malicious Limitations

Dưới đây là danh sách những hạn chế không phải do sự can thiệp xâm phạm của bằng chứng.

- Untagged pool memory (Bộ nhớ pool không được gắn thẻ): ExAllocatePoolWithTag là cách được đề xuất bởi Microsoft để các trình điều khiển và các thành phần chế độ kernel cấp phát bộ nhớ, nhưng không phải là tùy chọn duy nhất. Một trình điều khiển cũng có thể sử dụng ExAllocatePool, một API đang trong quá trình bị loại bỏ, nhưng vẫn có sẵn trên nhiều phiên bản Windows. API này cấp phát bộ nhớ nhưng không có thẻ, khiến bạn không có cách dễ dàng để theo dõi hoặc quét các phân bổ này.

- False positives (Các kết quả giả vờ): Vì phương pháp quét thẻ pool dựa essentially dựa trên phù hợp mẫu và heuristic, việc có kết quả giả vờ là khả năng có thể xảy ra. Điều này đặc biệt đúng khi quét không gian địa chỉ vật lý vì nó bao gồm dữ liệu mà hệ điều hành đã loại bỏ. Để giải quyết các kết quả giả vờ, thường bạn cần xem xét ngữ cảnh của đối tượng (nơi nó được tìm thấy), xem giá trị của các thành viên có hợp lý (điều này có thể thay đổi cho từng đối tượng) và xem xét liệu bạn có tìm thấy đối tượng bằng các phương pháp khác như danh sách thay thế.

- Large allocations (Các phân bổ lớn): Kỹ thuật quét thẻ pool không hoạt động với các phân bổ lớn hơn 4096 byte (xem phần "Big Page Pool" sắp tới). May mắn thay, tất cả các đối tượng thực thi đều nhỏ hơn kích thước này.

### Malicious Limitations (Anti-Forensics)

Dưới đây là danh sách các lưu ý về quét thẻ pool liên quan đến các cuộc tấn công chống pháp luật:

- Arbitrary tags (Thẻ tùy ý): Một trình điều khiển có thể cấp phát bộ nhớ bằng cách sử dụng một thẻ thông thường, hoặc mặc định, chẳng hạn như "Ddk " (ký tự cuối cùng là dấu cách). Thẻ này được sử dụng trong toàn bộ hệ điều hành và cũng trong mã của bên thứ ba khi không xác định thẻ cụ thể. Nói cách khác, nếu các trình điều khiển độc hại sử dụng "Ddk " làm thẻ của họ, khối bộ nhớ sẽ hoà quyện với các phân bổ khác.

- Decoy tags (Các thẻ giả mạo): Như được nêu bởi Walters và Petroni, một trình điều khiển có thể tạo ra các đối tượng giả mạo (hoặc thẻ giả mạo) trông "giống cuộc sống" để đánh lừa nhà điều tra, tăng hiệu suất tín hiệu và nhiễu.

- Manipulated tags (Thẻ bị thao tác): Vì thẻ được dùng cho mục đích gỡ lỗi, chúng không quan trọng đối với sự ổn định của hệ điều hành. Rootkit chạy trong kernel có thể thay đổi thẻ pool (hoặc bất kỳ giá trị nào trong _POOL_HEADER, chẳng hạn như kích thước khối và loại bộ nhớ) mà không có sự khác biệt đáng kể trên máy thực, nhưng sự thao tác này ngăn trình quét thẻ pool của Volatility hoạt động đúng cách.

Bạn có thể đối phó với hầu hết, nếu không phải tất cả, các kỹ thuật chống pháp luật bằng cách xác minh bằng các nguồn dữ liệu khác. Ví dụ, trong Chương 6, chúng tôi thảo luận về ít nhất sáu cách để tìm quy trình - chỉ một trong số đó liên quan đến quét thẻ pool. Để bác bỏ một đối tượng kết nối TCP giả mạo, bạn nên tham khảo các ghi nhận gói tin mạng hoặc nhật ký tường lửa, và tiếp tục kiểm tra xem hoạt động thực sự đã xảy ra hay chưa.

## Big Page Pool

Như đã đề cập trước đó, kernel Windows sẽ cố gắng nhóm các phân bổ có kích thước tương tự cùng nhau. Tuy nhiên, nếu kích thước yêu cầu vượt quá một trang (4096 byte), khối bộ nhớ được phân bổ từ một pool đặc biệt (gọi là big page pool) dành riêng cho các phân bổ lớn. Trong trường hợp này, _POOL_HEADER, chứa tag bốn byte và tồn tại tại địa chỉ cơ sở cho các phân bổ nhỏ hơn, không được sử dụng. Do đó, việc quét tag pool sẽ thất bại vì không có tag nào được tìm thấy. Hình 5-4 cho thấy sự khác biệt trong cấu trúc bộ nhớ giữa hai phân bổ lân cận của kernel có kích thước nhỏ hơn 4096 byte và hai phân bổ lớn hơn 4096 byte.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20233439.png)

Như bạn có thể thấy, các cấu trúc _POOL_HEADER không được lưu trữ cho các phân bổ lớn. Nhằm chứng minh ý tưởng, chúng tôi đã viết một trình điều khiển kernel mà kẹp lấy API ObCreateObject và tăng kích thước các phân bổ được sử dụng để lưu trữ đối tượng _EPROCESS. Như chúng tôi nghi ngờ, plugin psscan của Volatility không tìm thấy các quy trình mới do tag Proc không tồn tại. Tuy nhiên, cũng có một điều may mắn, bạn vẫn không hoàn toàn tuyệt vọng; bạn chỉ cần tìm ở một nơi khác so với thông thường. Ví dụ, Hình 5-4 cho thấy bảng theo dõi trang lớn (big page track tables) có thể trỏ trực tiếp đến các đối tượng trong big page pools.

### Big Page Track Tables

Bảng theo dõi trang lớn (big page track tables) có sự khác biệt đáng kể so với các bảng theo dõi pool đã được đề cập trước đó trong chương. Các bảng theo dõi pool (_POOL_TRACKER_TABLE) cho các khối bộ nhớ nhỏ lưu trữ thông tin thống kê về số lượng phân bổ và sử dụng byte; nhưng chúng không cung cấp địa chỉ của tất cả các phân bổ (do đó cần phải quét). Bảng theo dõi trang lớn, tuy nhiên, không lưu trữ thông tin thống kê, nhưng chúng bao gồm các địa chỉ của các phân bổ. Nếu bạn có thể tìm thấy các bảng theo dõi trang lớn, chúng có thể phục vụ như bản đồ giúp bạn xác định bất kỳ phân bổ lớn nào trong bộ nhớ kernel.

Thật không may, ký hiệu kernel nt!PoolBigPageTable, trỏ đến mảng các cấu trúc _POOL_TRACKER_BIG_PAGES (một cho mỗi phân bổ lớn), không được xuất khẩu hoặc sao chép vào khối dữ liệu kernel debugger. Tuy nhiên, chúng tôi đã phát hiện ra rằng ký hiệu này luôn luôn có thể được tìm thấy ở vị trí có thể dự đoán được liên quan đến nt!PoolTrackTable (được sao chép vào khối dữ liệu debugger). Do đó, nếu bạn có thể tìm thấy các bảng theo dõi pool, bạn có thể dễ dàng tìm thấy các bảng theo dõi trang lớn.

Đầu ra sau đây cho thấy cấu trúc bảng theo dõi trang lớn cho hệ thống Windows 7 64-bit:

```
>>> dt("_POOL_TRACKER_BIG_PAGES")
'_POOL_TRACKER_BIG_PAGES' (24 bytes)
0x0  : Va               ['pointer64', ['void']]
0x8  : Key              ['unsigned long']
0xc  : PoolType         ['unsigned long']
0x10 : NumberOfBytes    ['unsigned long long']
```

Phần tử "Va" (Virtual Address) chỉ đến địa chỉ cơ sở của phân bổ. Bạn cũng có thể thấy giá trị "Key" (pool tag), "PoolType" (trang hoặc không trang) và "NumberOfBytes" (kích thước của phân bổ). Hãy nhớ rằng mặc dù cấu trúc này lưu trữ pool tag, nó nằm ở một vị trí hoàn toàn khác với phân bổ, mà được trỏ đến bởi "Va". Đối với các phân bổ nhỏ, pool tag nằm trong phân bổ chính (hãy nhớ lại những gì bạn đã thấy trong Hình 5-4).

### Bigpools Plugin

Để tạo thông tin về các phân bổ pool kernel lớn trong bộ nhớ, bạn có thể sử dụng plugin "bigpools". Lệnh dưới đây cho thấy một ví dụ về đầu ra:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20234251.png)

Cột "Allocation" cho biết địa chỉ bộ nhớ kernel mà phân bổ bắt đầu. Nếu bạn muốn xem nội dung của khu vực đó, bạn có thể trích xuất dữ liệu tại địa chỉ đó bằng volshell. Một số tài liệu cho biết các phân bổ với địa chỉ kết thúc bằng số 1 (ví dụ: 0xfffff8a00f9a8001) nằm trong bộ nhớ không trang; tuy nhiên, theo nghiên cứu của chúng tôi, số 1 thể hiện trạng thái "đã giải phóng" (free). Do đó, bạn có thể thử hiển thị những địa chỉ đó, nhưng chúng có thể không chứa những gì bạn mong đợi dựa trên pool tag. Ví dụ, so sánh một số khối CM31 như sau: 

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20234320.png)

Địa chỉ đầu tiên (0xfffff8a003747000) chứa "hbin" ở đầu và một số ví dụ về "vk." Như đã đề cập trước đó, "CM" trong CM31 đại diện cho Configuration Manager, là thành phần của cơ sở dữ liệu Registry của kernel. "hbin" và "vk" là các chữ ký cho các khối HBIN của Registry và các giá trị riêng lẻ, tương ứng (xem Chương 10). Các địa chỉ thứ hai (0xfffff8a00f9a8001) và thứ ba (0xfffff8a00861d001) đều được đánh dấu là miễn phí, nhưng có sự khác biệt đáng kể giữa chúng. Một trong số này không có sẵn, có thể vì đã bị đẩy ra đĩa cứng - cuối cùng, nó nằm trong bộ nhớ được phân trang. Còn một cái khác có vẻ đã được cấp phát lại và ghi đè lên, vì chữ ký "hbin" đã biến mất.

### Exploring Big Page Pools

Trên một hệ thống thông thường, có hàng nghìn phân vùng trong big page pool, vì vậy bạn có thể muốn sử dụng tùy chọn --tags cho plugin (một danh sách các tag phân tách bằng dấu phẩy để tìm kiếm). Nếu không, nếu bạn chỉ đang khám phá, bạn có thể lưu danh sách các phân vùng vào một tệp văn bản, sau đó sắp xếp dựa trên tần suất của tag. Ví dụ, xem mã sau đây:

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-28%20234341.png)

>**LƯU Ý**<br>
Các mô tả cho các pool tag không được tạo tự động bởi các lệnh hiển thị. Chúng tôi đã tìm kiếm thủ công trong tệp pooltag.txt và thêm chúng để chú thích kết quả đầu ra.

Dựa vào các mô tả, bạn có thể thấy rằng dữ liệu trong big page pools chứa một số tài liệu rất thú vị - từ các bảng dịch chuyển đến các bản ghi bảng trang (PTE) và nhật ký truy cập của bộ quản lý bộ nhớ (Mm), chi tiết về Address Space Layout Randomization (ASLR), bảng xử lý quy trình (bảng đối tượng), và các phân bổ cổng Internet. Để giải thích dữ liệu trong các phân bổ này, bạn cần hiểu các cấu trúc và định dạng mà driver sở hữu sử dụng, nhưng việc biết chính xác các mô tả và vị trí cụ thể của các khối trong bộ nhớ sẽ giúp bạn nhanh chóng tăng tốc quá trình nghiên cứu.

## Pool-Scanning Alternatives


Bây giờ khi bạn đã nắm vững những ưu điểm và nhược điểm của kỹ thuật pool tag scanning trong phân tích nhớ, chúng ta sẽ kết thúc chương này bằng một cuộc thảo luận nhanh về một số phương pháp thay thế.

### Dispatcher Header Scans

Đối với một số loại đối tượng quản lý của hệ thống như tiến trình (processes), luồng (threads) và mutexes, chúng đều có khả năng đồng bộ hóa. Điều này có nghĩa là các luồng khác có thể đồng bộ hóa với, hoặc chờ đợi, những đối tượng này để thực hiện các hành động như bắt đầu, kết thúc hoặc thực hiện các hành động khác. Để hỗ trợ tính năng này, kernel lưu trữ thông tin về trạng thái hiện tại của đối tượng trong một cấu trúc con được gọi là _DISPATCHER_HEADER và nó xuất hiện ngay tại đầu tiên (offset 0) của các cấu trúc đối tượng quản lý của hệ thống. Quan trọng hơn, cấu trúc này chứa một số giá trị được duy trì nhất quán qua các bản sao bộ nhớ từ một phiên bản cụ thể của Windows, do đó bạn có thể dễ dàng xây dựng một chữ ký (signature) cho từng profile để tìm kiếm các đối tượng này.

>**Lưu ý**<br> Để biết thêm thông tin về việc sử dụng dispatcher headers cho phân tích bộ nhớ, bạn có thể tham khảo bài viết "Searching for Processes and Threads in Microsoft Windows Memory Dumps" của Andreas Schuster tại đường dẫn: http://www.dfrws.org/2006/proceedings/2-Schuster.pdf.

Đầu ra dưới đây là từ một hệ thống Windows XP Service Pack 2 32-bit:

```
>>> dt("_DISPATCHER_HEADER")
'_DISPATCHER_HEADER' (16 bytes)
0x0 : Type          ['unsigned char']
0x1 : Absolute      ['unsigned char']
0x2 : Size          ['unsigned char']
0x3 : Inserted      ['unsigned char']
0x4 : SignalState   ['long']
0x8 : WaitListHead  ['_LIST_ENTRY']
```

Andreas Schuster phát hiện rằng các trường Absolute và Inserted luôn luôn là 0 trên các hệ thống Windows 2000, XP và 2003. Các trường Type và Size được mã hóa cứng, chỉ định loại đối tượng và kích thước của đối tượng tương ứng. Ví dụ, trên máy tính Windows XP 32-bit, trường Type là 3 cho các tiến trình (_EPROCESS), và trường Size là 0x1b, tạo thành một chữ ký 4 byte là \x03\x00\x1b\x00. Tương tự như pool tag, bạn có thể quét qua bộ nhớ để tìm các trường hợp của chữ ký này để tìm tất cả các đối tượng _EPROCESS. Đây là cách mà một trong những công cụ phân tích bộ nhớ rất sớm gọi là PTFinder đã hoạt động: http://computer.forensikblog.de/en/2007/11/ptfinder-version-0305.html.
Một trong những nhược điểm của quét tiêu đề bộ điều phối là nó chỉ giúp tìm các đối tượng có khả năng đồng bộ hóa. Ví dụ, các đối tượng file không thể đồng bộ hóa, do đó chúng không có _DISPATCHER_HEADER được nhúng. Do đó, bạn không thể tìm các trường hợp của đối tượng _FILE_OBJECT bằng phương pháp này. Hơn nữa, bắt đầu từ Windows 2003, cấu trúc _DISPATCHER_HEADER được mở rộng thành 10 thành viên thay vì chỉ có 6; và phiên bản Windows 7 có gần 30 thành viên. Với quá nhiều thành viên như vậy, sự không chắc chắn trong việc duy trì tính nhất quán của chúng khiến việc xây dựng một chữ ký trở nên khó khăn.

>**LƯU Ý:**<br> Một ví dụ về trình quét tiêu đề bộ điều phối để tìm các tiến trình có thể được tìm thấy trong tệp contrib/plugins/pspdispscan.py trong mã nguồn của Volatility. Nó chỉ được cung cấp như một chứng cớ về khả năng và hiện tại chỉ hoạt động với các tệp mẫu Windows XP 32 bit.

### Robust Signature Scans

Cả hai tiêu đề pool và tiêu đề bộ điều phối không thiết yếu đối với hệ điều hành, điều này có nghĩa là chúng có thể bị thay đổi độc hại để đánh bại quá trình quét dựa trên chữ ký mà không gây ra sự không ổn định của hệ thống. Tuy nhiên, có một cách tiếp cận khác để tìm các đối tượng trong các bản ghi nhớ mà chống lại các sự thay đổi như vậy. Brendan Dolan-Gavitt và các đồng nghiệp đã viết một bài báo có tựa đề Robust Signatures for Kernel Data Structures (http://www.cc.gatech.edu/~brendan/ccs09_siggen.pdf) mô tả phương pháp này. Họ đã thử nghiệm tác động lên hệ điều hành bằng cách thay đổi các thành viên của _EPROCESS và ghi lại những thành viên nào gây ra sự cố của hệ thống. Những thành viên này được gọi là cần thiết. Sau đó, một chữ ký được xây dựng dựa chỉ trên các thành viên cần thiết này.

>**Lưu ý**<br> rằng công cụ kiểm thử bằng cách sử dụng Volatility ở chế độ ghi để truy cập vào bộ nhớ vật lý từ các máy ảo Xen và VMware Server.

Một plugin Volatility ví dụ có tên là psscan3 (http://www.cc.gatech.edu/~brendan/volatility/dl/psscan3.py) đã được phát triển và phân phối cùng với bài báo. Danh sách sau cung cấp tóm tắt về những gì Dolan-Gavitt và đồng nghiệp của ông coi là một chữ ký mạnh mẽ cho các đối tượng _EPROCESS.

- Căn chỉnh DTB: Base bảng thư mục (_DirectoryTableBase) phải được căn chỉnh trên ranh giới 32-bit.

- Cờ quyền truy cập được cấp: Thành viên GrantedAccess phải có các cờ 0x1F07FB được đặt.

- Độ dài con trỏ: Các thành viên VadRoot, ObjectTable, ThreadListHead và ReadyListHead phải chứa các địa chỉ chế độ kernel hợp lệ.

- Danh sách bộ nhớ làm việc: Thành viên VmWorkingSetList không chỉ phải trỏ vào chế độ kernel mà còn cần phải lớn hơn 0xC0000000 (đối với các hệ thống 32-bit).

- Đếm khóa: Đếm khóa WorkingSetLock và AddressCreationLock phải bằng 1.

Bất kỳ cố gắng sửa đổi các thành viên này thành các giá trị bên ngoài giá trị đã chỉ định sẽ gây ra lỗi màn hình xanh. Do đó, trình quét của bạn trở nên mạnh mẽ hơn đối với những thay đổi tiềm năng mà một kẻ tấn công (hoặc mẫu mã độc) có thể thử làm. Tuy nhiên, kỹ thuật này yêu cầu một khung kiểm tra lỗi và thời gian lớn để xác thực và kiểm tra các kết quả. Vì lý do này, plugin psscan3 hiện không được bao gồm trong phiên bản mới nhất của Volatility - nhưng bạn có thể tải xuống và sử dụng nó với Volatility 1.3.

## Tổng quan

Trình quản lý đối tượng của Windows đóng một vai trò quan trọng trong việc tạo và xóa nhiều đối tượng quan trọng, mà các nhà điều tra dựa vào để phân tích (quy trình, tệp, khóa registry và nhiều hơn nữa). Tuy nhiên, tính đáng tin cậy của bằng chứng bạn thấy phụ thuộc vào cách mà framework dò tìm và xác thực dữ liệu trong bộ nhớ. Mặc dù quét thông qua bộ nhớ vật lý (bao gồm cả các khối trống) là mạnh mẽ, nhưng nó cũng rất dễ bị hỏng vì thường dựa trên các chữ ký không quan trọng. Để trở thành một nhà phân tích hiệu quả, bạn cần hiểu cách các kỹ thuật quét hoạt động và do đó, cách mà các kẻ tấn công có thể né tránh các công cụ điều tra bộ nhớ. Hơn nữa, bạn nên quen với việc xác nhận nhiều nguồn bằng chứng trước khi rút ra kết luận.
