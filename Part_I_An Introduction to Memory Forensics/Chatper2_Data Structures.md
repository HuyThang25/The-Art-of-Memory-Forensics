Hiểu cách dữ liệu được tổ chức trong bộ nhớ tạm là một khía cạnh quan trọng của phân tích bộ nhớ. Tương tự như các tập tin trong phân tích hệ thống tập tin, các cấu trúc dữ liệu cung cấp mẫu để diễn giải cách dữ liệu được sắp xếp trong bộ nhớ. Cấu trúc dữ liệu là những khối cơ bản mà các lập trình viên sử dụng để thực hiện phần mềm và tổ chức cách dữ liệu của chương trình được lưu trữ trong bộ nhớ. Điều này rất quan trọng để bạn có được hiểu biết cơ bản về các cấu trúc dữ liệu phổ biến nhất thường xuyên gặp phải và cách mà các cấu trúc dữ liệu đó được biểu thị trong RAM. Tận dụng kiến thức này giúp bạn xác định các kỹ thuật phân tích hiệu quả nhất, hiểu rõ các giới hạn liên quan đến các kỹ thuật đó, nhận ra các sửa đổi dữ liệu độc hại và suy luận về các hoạt động trước đó đã được thực hiện trên dữ liệu. Chương này không nhằm cung cấp một cuộc thám hiểm toàn diện về cấu trúc dữ liệu, mà chỉ nhằm giúp đánh giá lại các khái niệm và thuật ngữ được tham khảo thường xuyên trong phần còn lại của cuốn sách.

## Các kiểu dữ liệu cơ bản

Bạn xây dựng các cấu trúc dữ liệu bằng cách sử dụng các loại dữ liệu cơ bản mà ngôn ngữ lập trình cụ thể cung cấp. Các loại dữ liệu cơ bản này được sử dụng để xác định cách một tập hợp cụ thể của các bit được sử dụng trong một chương trình. Bằng cách chỉ định một loại dữ liệu, người lập trình quy định tập giá trị có thể được lưu trữ và các phép toán có thể được thực hiện trên những giá trị đó. Những loại dữ liệu này được gọi là các loại dữ liệu cơ bản hoặc nguyên thủy vì chúng không được xác định dựa trên các loại dữ liệu khác trong ngôn ngữ lập trình đó. Trong một số ngôn ngữ lập trình, các loại dữ liệu cơ bản có thể ánh xạ trực tiếp vào các loại dữ liệu phần cứng cơ bản được hỗ trợ bởi kiến trúc bộ vi xử lý. Điều quan trọng là nhấn mạnh rằng các loại dữ liệu cơ bản thường khác nhau giữa các ngôn ngữ lập trình và kích thước lưu trữ của chúng có thể thay đổi tùy thuộc vào phần cứng cơ bản.

### Ngôn ngữ lập trình C

Cuốn sách tập trung chủ yếu vào các loại dữ liệu cơ bản của ngôn ngữ lập trình C. Vì tính hữu ích trong lập trình hệ thống và khả năng quản lý trực tiếp việc phân bổ bộ nhớ, C thường xuất hiện khi phân tích trạng thái đặt trong bộ nhớ của các hệ điều hành hiện đại. Bảng 2-1 dưới đây mô tả các loại dữ liệu cơ bản của ngôn ngữ lập trình C và kích thước lưu trữ thông thường cho cả kiến trúc 32-bit và 64-bit.

>**GHI CHÚ:** Chúng tôi đã bao gồm loại dữ liệu con trỏ trong Bảng 2-1. Một con trỏ là một giá trị lưu trữ địa chỉ bộ nhớ ảo. Chương trình có thể khai báo một con trỏ cho bất kỳ loại dữ liệu nào (ví dụ: char, long hoặc một trong các loại trừu tượng được thảo luận sau). Để truy cập dữ liệu đã lưu trữ, bạn phải thực hiện giải tham chiếu (de-reference) cho con trỏ, điều này yêu cầu chuyển đổi địa chỉ bộ nhớ ảo. Do đó, việc không thể dịch các địa chỉ sẽ hạn chế các loại phân tích bạn có thể thực hiện trên một mẫu bộ nhớ vật lý.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20220738.png)

Các loại cơ bản trong Bảng 2-1 là những loại mà tiêu chuẩn C định nghĩa. Chúng được sử dụng rộng rãi cho các kernels Windows, Linux và Mac OS X. Windows cũng định nghĩa nhiều loại riêng dựa trên các loại cơ bản này mà bạn có thể thấy trong các tệp tiêu đề và tài liệu của Windows. Bảng 2-2 mô tả một số trong số các loại này.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20220925.png)

Trình biên dịch quyết định kích thước thực tế của bộ nhớ được cấp phát cho các loại dữ liệu cơ bản, điều này thường phụ thuộc vào phần cứng cơ bản. Quan trọng là ghi nhớ rằng hầu hết các loại dữ liệu cơ bản là các giá trị nhiều byte, và thứ tự endian cũng phụ thuộc vào phần cứng cơ bản xử lý dữ liệu. Các phần sau minh họa ví dụ về cách ngôn ngữ lập trình C cung cấp cơ chế để kết hợp các loại dữ liệu cơ bản thành các loại dữ liệu hỗn hợp được sử dụng để thực hiện cấu trúc dữ liệu.

### Abstract Data Types (Các kiểu dữ liệu trừu tượng)

Thảo luận về các ví dụ cụ thể về cấu trúc dữ liệu được giới thiệu bằng các khái niệm lưu trữ dưới dạng các loại dữ liệu trừu tượng. Các loại dữ liệu trừu tượng cung cấp mô hình cho cả dữ liệu và các hoạt động được thực hiện trên dữ liệu. Những mô hình này độc lập với bất kỳ ngôn ngữ lập trình cụ thể nào và không quan tâm đến chi tiết của dữ liệu cụ thể được lưu trữ.

Khi thảo luận về các loại dữ liệu trừu tượng này, chúng ta sẽ dùng thuật ngữ chung "element" (phần tử) để đại diện cho một loại dữ liệu không xác định. Phần tử này có thể là một loại dữ liệu cơ bản hoặc một loại dữ liệu hỗn hợp. Giá trị của một số phần tử - con trỏ - cũng có thể được sử dụng để đại diện cho các liên kết giữa các phần tử. Tương tự như việc thảo luận về con trỏ C lưu trữ địa chỉ bộ nhớ, giá trị của những phần tử này được sử dụng để tham chiếu đến một phần tử khác. 

Bằng cách bàn luận các khái niệm này ban đầu dưới dạng các loại dữ liệu trừu tượng, bạn có thể xác định tại sao bạn sử dụng một tổ chức dữ liệu cụ thể và làm thế nào dữ liệu được lưu trữ sẽ được điều chỉnh. Cuối cùng, chúng ta sẽ thảo luận về các ví dụ về cách ngôn ngữ lập trình C thực hiện các loại dữ liệu trừu tượng như cấu trúc dữ liệu và cung cấp các ví dụ về cách sử dụng chúng trong hệ điều hành. Bạn sẽ thường xuyên gặp các cài đặt cấu trúc dữ liệu mà có thể bạn không quen thuộc ngay lập tức. Bằng cách sử dụng kiến thức về cách chương trình sử dụng dữ liệu, các đặc tính của cách dữ liệu được lưu trữ trong bộ nhớ và các quy ước của ngôn ngữ lập trình, bạn thường có thể nhận ra một mẫu dữ liệu trừu tượng sẽ giúp cung cấp gợi ý về cách xử lý dữ liệu.

### Arrays (Mảng)

Cơ chế đơn giản nhất để tổ chức dữ liệu là mảng một chiều. Đây là một tập hợp các cặp ,<index, element> (<chỉ mục, phần tử>), trong đó các phần tử có cùng một kiểu dữ liệu đồng nhất. Kiểu dữ liệu của các phần tử được lưu trữ thường được gọi là kiểu mảng. Một đặc điểm quan trọng của mảng là kích thước của nó được cố định khi một phiên bản của mảng được tạo, điều này giới hạn số phần tử có thể được lưu trữ. Bạn truy cập các phần tử của mảng bằng cách chỉ định chỉ mục mảng tương ứng với vị trí của phần tử trong tập hợp. Figure 2-1 trình bày một ví dụ về một mảng một chiều.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20222419.png)

Trong Figure 2-1, bạn có một ví dụ về một mảng có thể chứa năm phần tử. Mảng này được sử dụng để lưu trữ các ký tự và sau đó có thể được gọi là một mảng các ký tự. Theo quy ước phổ biến mà bạn sẽ thấy trong suốt cuốn sách, phần tử đầu tiên của mảng được tìm thấy tại chỉ mục 0. Do đó, giá trị của phần tử tại chỉ mục 2 là ký tự 'C'. Giả sử mảng được đặt tên là grades, chúng ta cũng có thể gọi phần tử này là grades[2].

Mảng thường được sử dụng bởi vì nhiều ngôn ngữ lập trình cung cấp các triển khai được thiết kế để truy cập dữ liệu đã lưu trữ một cách rất hiệu quả. Ví dụ, trong ngôn ngữ lập trình C, yêu cầu lưu trữ là cố định và trình biên dịch cấp phát một khối bộ nhớ liên tục để lưu trữ các phần tử của mảng. Bạn thường tham chiếu đến vị trí của mảng bằng địa chỉ bộ nhớ của phần tử đầu tiên, gọi là base address (địa chỉ cơ sở) của mảng. Sau đó, bạn có thể truy cập các phần tử tiếp theo của mảng như một độ lệch từ địa chỉ cơ sở. Ví dụ, giả sử một mảng được lưu trữ tại địa chỉ cơ sở X và lưu trữ các phần tử có kích thước S, bạn có thể tính địa chỉ của phần tử tại chỉ mục I bằng cách sử dụng phương trình sau: 
	
	Address(I) = X + (I * S).

Đặc tính này thường được gọi là random access (truy cập ngẫu nhiên) vì thời gian để truy cập một phần tử không phụ thuộc vào phần tử bạn đang truy cập.

Vì mảng cực kỳ hiệu quả cho việc truy cập dữ liệu, bạn thường gặp chúng trong quá trình phân tích bộ nhớ. Điều này đặc biệt đúng khi phân tích các cấu trúc dữ liệu của hệ điều hành. Cụ thể, hệ điều hành thường lưu trữ các mảng các con trỏ và các loại phần tử cố định khác mà bạn cần truy cập nhanh chóng tại các chỉ mục cố định. Ví dụ, trong Chương 21, bạn tìm hiểu cách Linux sử dụng một mảng để lưu trữ các bộ xử lý tệp liên quan đến một quá trình. Trong trường hợp này, chỉ số mảng là số mô tả tệp. Ngoài ra, Chương 13 hiển thị bảng MajorFunction của Microsoft: một mảng con trỏ hàm cho phép một ứng dụng giao tiếp với một trình điều khiển. Mỗi phần tử của mảng chứa địa chỉ của mã (quá trình điều khiển) thực thi để thực hiện một yêu cầu I/O. Chỉ số cho mảng tương ứng với mã hoạt động đã xác định trước (ví dụ: 0 = đọc; 1 = ghi; 2 = xóa).

Figure 2-2 cung cấp một ví dụ về cách các con trỏ hàm cho bảng MajorFunction của một trình điều khiển được lưu trữ trong bộ nhớ trên một hệ thống dựa trên IA32, giả sử rằng mục nhập đầu tiên được lưu trữ tại địa chỉ cơ sở (base_address).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20224004.png)

Nếu bạn biết địa chỉ cơ sở của một mảng và kích thước của các phần tử trong mảng, bạn có thể nhanh chóng liệt kê các phần tử được lưu trữ liên tiếp trong bộ nhớ. Hơn nữa, bạn có thể truy cập vào bất kỳ thành viên cụ thể nào bằng cách sử dụng các phép tính giống như những phép tính mà trình biên dịch tạo ra. Bạn cũng có thể nhận ra các mẫu trong các khối bộ nhớ liên tiếp không xác định giống như một mảng, điều này cung cấp gợi ý về cách dữ liệu được sử dụng, cái gì đang được lưu trữ và cách nó có thể được giải thích.

### Bitmaps

Một biến thể của mảng được sử dụng để biểu diễn tập hợp là bitmap, còn được gọi là vector bit hoặc mảng bit. Trong trường hợp này, chỉ mục đại diện cho một số nguyên liên tiếp cố định và các phần tử lưu trữ một giá trị boolean {1, 0}. Trong phân tích bộ nhớ, các bitmap thường được sử dụng để xác định một đối tượng cụ thể có thuộc tập hợp hay không (ví dụ: bộ nhớ đã được cấp phát hay chưa, ưu tiên thấp hay cao, và cetera). Chúng được lưu trữ dưới dạng một mảng bit, được gọi là bản đồ, và mỗi bit đại diện cho việc một đối tượng có hợp lệ hay không. Sử dụng bitmap cho phép biểu diễn tám đối tượng trong một byte, điều này phù hợp với các tập hợp lớn.

Một biến thể của mảng được sử dụng để biểu diễn các tập hợp là bitmap, còn được gọi là vector bit hoặc mảng bit. Trong trường hợp này, chỉ mục đại diện cho một số nguyên liên tiếp cố định, và các phần tử lưu trữ một giá trị boolean {1, 0}. Trong phân tích bộ nhớ, bitmap thường được sử dụng để xác định một đối tượng cụ thể có thuộc tập hợp hay không (ví dụ: bộ nhớ đã được cấp phát hay chưa, ưu tiên thấp hay cao, v.v.). Chúng được lưu trữ dưới dạng một mảng bit, được gọi là bản đồ, và mỗi bit đại diện cho việc một đối tượng có hợp lệ hay không. Sử dụng bitmap cho phép biểu diễn tám đối tượng trong một byte, điều này phù hợp với các tập hợp dữ liệu lớn. Ví dụ, trong hạt nhân Windows, bitmap được sử dụng để duy trì các cổng mạng đã được cấp phát. Các cổng mạng được biểu diễn dưới dạng unsigned short, chiếm 2 byte và cung cấp ((2^16) - 1) hoặc 65535 khả năng. Số lượng cổng lớn này được biểu diễn bằng một bitmap 65535 bit (khoảng 8KB). Figure 2-3 cung cấp một ví dụ về cấu trúc Windows này.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20224822.png)

Hình vẽ cho thấy cả góc nhìn cấp bit và góc nhìn cấp byte của bitmap. Trong góc nhìn cấp byte, bạn có thể thấy rằng chỉ mục đầu tiên có giá trị là 4e theo hệ thập lục phân, tương ứng với 1001110 theo hệ nhị phân. Giá trị nhị phân này cho thấy các cổng 1, 2, 3 và 6 đang được sử dụng, vì đó là các vị trí của các bit đã được thiết lập.

### Records (Bản ghi)

Một cơ chế khác thường được sử dụng để tổng hợp dữ liệu là loại bản ghi (hoặc cấu trúc). Khác với mảng yêu cầu các phần tử phải có cùng loại dữ liệu, một bản ghi có thể bao gồm các phần tử không đồng nhất. Nó được tạo thành từ một tập hợp các trường, trong đó mỗi trường được chỉ định bằng cặp <tên, phần tử>. Mỗi trường cũng thường được gọi là thành viên của bản ghi. Do bản ghi là tĩnh, tương tự như mảng, việc kết hợp các phần tử và thứ tự của chúng được cố định khi tạo một phiên bản của bản ghi. Bằng cách chỉ định tên thành viên cụ thể, nó hoạt động như một khóa hoặc chỉ số để truy cập các phần tử của bản ghi. Một tập hợp các phần tử có thể được kết hợp trong một bản ghi để tạo thành một phần tử mới. Tương tự, cũng có thể có một phần tử của một bản ghi hoặc một mảng trong chính bản ghi đó.

Figure 2-4 thể hiện một ví dụ về bản ghi kết nối mạng bao gồm bốn thành viên mô tả các đặc điểm của nó: id, port, addr và hostname. Mặc dù các thành viên có thể có nhiều loại dữ liệu và kích thước đi kèm, bằng cách tạo thành một bản ghi bạn có thể lưu trữ và tổ chức các đối tượng liên quan.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20232855.png)

Trong ngôn ngữ lập trình C, bản ghi được thực hiện bằng cách sử dụng cấu trúc (structure). Một cấu trúc cho phép người lập trình xác định tên, kiểu và thứ tự của các thành viên. Tương tự như mảng, kích thước của cấu trúc C được biết đến khi tạo một phiên bản, các cấu trúc được lưu trữ trong một khối bộ nhớ liền kề, và các phần tử được truy cập bằng cách sử dụng tính toán cơ sở cộng với lệch. Các lệch từ cơ sở của một bản ghi thay đổi tùy thuộc vào kích thước của các loại dữ liệu được lưu trữ. Trình biên dịch xác định các lệch để truy cập một phần tử dựa trên kích thước của các loại dữ liệu đứng trước nó trong cấu trúc. Ví dụ, bạn có thể biểu diễn định nghĩa của một cấu trúc C cho thông tin kết nối mạng từ Figure 2-4 theo định dạng sau:

```C
struct Connection {
	short id;
	short port;
	unsigned long addr;
	char hostname[32];
};
```

Như bạn có thể thấy, `Connection` là tên của cấu trúc; và `id`, `port`, `addr` và `hostname` là tên của các thành viên trong cấu trúc. Trường đầu tiên của cấu trúc này được đặt tên là `id`, có độ dài là 2 byte và được sử dụng để lưu trữ một định danh duy nhất liên kết với bản ghi này. Trường thứ hai được đặt tên là `port`, cũng có độ dài 2 byte và được sử dụng để lưu trữ cổng lắng nghe trên máy từ xa. Trường thứ ba được sử dụng để lưu trữ địa chỉ IP của máy từ xa dưới dạng dữ liệu nhị phân trong 4 byte với tên trường là `addr`. Trường thứ tư được gọi là `hostname` và lưu trữ tên máy từ xa sử dụng 32 byte. Sử dụng thông tin về các loại dữ liệu, trình biên dịch có thể xác định các lệch thích hợp (0, 2, 4 và 8) để truy cập vào mỗi trường. Bảng 2-3 liệt kê các thành viên của `Connection` và các lệch tương ứng của chúng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20233602.png)

Dưới đây là một ví dụ về cách một phiên bản của cấu trúc dữ liệu sẽ xuất hiện trong bộ nhớ, như được hiển thị bởi một công cụ đọc dữ liệu raw:

```
0000000: 0100 0050 ad3d de09 7777 772e 766f 6c61 ...P.>X.www.vola
0000010: 7469 6c69 7479 666f 756e 6461 7469 6f6e tilityfoundation
0000020: 2e6f 7267 0000 0000 			 .org....
```

Các cấu trúc C-style là một trong những cấu trúc dữ liệu quan trọng nhất gặp phải khi thực hiện phân tích bộ nhớ. Sau khi xác định được địa chỉ cơ sở của một cấu trúc cụ thể, bạn có thể tận dụng định nghĩa của cấu trúc như một mẫu để trích xuất và diễn giải các phần tử của bản ghi. Một điều bạn phải nhớ là trình biên dịch có thể thêm các byte lùi vào một số trường của cấu trúc được lưu trữ trong bộ nhớ. Trình biên dịch thực hiện việc này để bảo tồn sự căn chỉnh của các trường và tăng hiệu suất CPU.

Ví dụ, chúng ta có thể sửa đổi kiểu cấu trúc từ Bảng 2-3 để loại bỏ trường port. Nếu chương trình được biên dịch lại với trình biên dịch được cấu hình để căn chỉnh trên ranh giới 32 bit, chúng ta sẽ tìm thấy dữ liệu sau đây trong bộ nhớ:

```
0000000: 0100 0000 ad3d de09 7777 772e 766f 6c61 ...P.>X.www.vola
0000010: 7469 6c69 7479 666f 756e 6461 7469 6f6e tilityfoundation
0000020: 2e6f 7267 0000 0000 			 .org....
```

Trong kết quả, bạn có thể thấy rằng mặc dù đã loại bỏ trường `port`, các trường còn lại (`id`, `addr`, `hostname`) vẫn được tìm thấy tại các vị trí lùi giống như trước đó (0, 4, 8). Bạn cũng sẽ nhận thấy rằng các byte từ 2 đến 3 (chứa giá trị 0000), trước đây lưu giữ giá trị cổng, bây giờ được sử dụng hoàn toàn cho việc padding.

Như bạn sẽ thấy trong các chương sau, định nghĩa của một cấu trúc dữ liệu và các ràng buộc liên quan đến các giá trị có thể được lưu trữ trong một trường cũng có thể được sử dụng như một mẫu để khắc phục các trường hợp có thể của cấu trúc trực tiếp từ bộ nhớ.

### Strings (Chuỗi) 

Một trong những khái niệm lưu trữ quan trọng mà bạn thường gặp là chuỗi (string). Chuỗi thường được xem như một trường hợp đặc biệt của mảng (array), trong đó các phần tử lưu trữ phải biểu diễn mã ký tự được lấy từ một bảng mã ký tự được xác định trước. Tương tự như một mảng, một chuỗi bao gồm một tập hợp các cặp <vị trí, phần tử>. Ngôn ngữ lập trình thường cung cấp các hàm thao tác chuỗi, có thể thay đổi các ánh xạ giữa vị trí và phần tử. Trong khi bản ghi và mảng thường được coi là các loại dữ liệu tĩnh, một chuỗi có thể chứa một chuỗi biến đổi các phần tử, và độ dài của chuỗi có thể thay đổi theo động trong quá trình xử lý. Do đó, chuỗi phải cung cấp cơ chế để xác định độ dài của tập hợp lưu trữ.

Cách triển khai đầu tiên mà chúng ta sẽ xem xét là chuỗi kiểu C (C-style strings). Chuỗi kiểu C cung cấp một cách triển khai rất giống với cách ngôn ngữ lập trình C triển khai mảng. Đặc biệt, chúng ta sẽ tập trung vào chuỗi kiểu C trong đó các phần tử là kiểu char - một kiểu dữ liệu cơ bản của C - và được mã hóa bằng bảng mã ký tự ASCII. Bảng mã này gán một giá trị số có 7 bit (thường được lưu trữ trong một byte đầy đủ) cho các ký tự trong tiếng Anh. Sự khác biệt chính giữa chuỗi kiểu C và mảng là chuỗi kiểu C tự động duy trì độ dài chuỗi. Điều này được thực hiện bằng cách đánh dấu cuối chuỗi bằng một ký tự kết thúc chuỗi được gắn trong chuỗi. Trong trường hợp của chuỗi kiểu C, ký tự kết thúc là ký tự ASCII NULL, có giá trị 0x00. Sau đó, bạn có thể tính độ dài của chuỗi bằng cách xác định số ký tự giữa địa chỉ bắt đầu của chuỗi và ký tự kết thúc.

Ví dụ, hãy xem xét cấu trúc dữ liệu được thảo luận trước đó để lưu trữ thông tin mạng, cấu trúc của nó được hiển thị trong Bảng 2-3. Phần tử thứ tư của cấu trúc dữ liệu được sử dụng để lưu trữ tên máy chủ là một chuỗi kiểu C. Hình 2-5 cho thấy cách chuỗi này được lưu trữ trong bộ nhớ sử dụng bảng mã ký tự ASCII. Chuỗi bắt đầu tại byte offset 8 và tiếp tục cho đến ký tự kết thúc, 0x00, tại offset 36. Do đó, bạn có thể xác định rằng chuỗi này được sử dụng để lưu trữ 28 ký tự và ký tự kết thúc.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-23%20205623.png)

Bạn sẽ thường xuyên gặp một cách triển khai chuỗi thay thế khi phân tích bộ nhớ liên quan đến hệ điều hành Microsoft Windows. Chúng ta sẽ gọi cách triển khai này là _UNICODE_STRING vì đó là tên của cấu trúc dữ liệu hỗ trợ. Khác với cách triển khai chuỗi kiểu C sử dụng bảng mã ký tự ASCII để lưu trữ các phần tử 1 byte, _UNICODE_STRING hỗ trợ mã hóa Unicode cho phép các phần tử có kích thước lớn hơn 1 byte và cung cấp hỗ trợ cho các ký tự trong nhiều ngôn ngữ, không chỉ tiếng Anh Mỹ.

Trừ khi được chỉ định khác, các phần tử của _UNICODE_STRING được mã hóa bằng phiên bản UTF-16 của Unicode. Điều này có nghĩa là các ký tự được lưu trữ dưới dạng giá trị 2 hoặc 4 byte. Một khác biệt khác với chuỗi kiểu C là _UNICODE_STRING không yêu cầu một ký tự kết thúc mà thay vào đó lưu trữ độ dài một cách rõ ràng. Như đã đề cập trước đó, _UNICODE_STRING được triển khai bằng cách sử dụng một cấu trúc lưu trữ độ dài, tính bằng byte, của chuỗi (Length), số byte tối đa có thể lưu trữ trong chuỗi này (MaximumLength), và một con trỏ đến địa chỉ bộ nhớ bắt đầu mà các ký tự được lưu trữ (Buffer). Định dạng cấu trúc dữ liệu _UNICODE_STRING được cho trong Bảng 2-4.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-23%20205824.png)

Chuỗi là một thành phần cực kỳ quan trọng trong phân tích bộ nhớ, vì nó được sử dụng để lưu trữ dữ liệu văn bản (mật khẩu, tên tiến trình, tên tệp, v.v.). Trong quá trình điều tra một sự cố, bạn thường xuyên tìm kiếm và trích xuất các chuỗi liên quan từ bộ nhớ. Trong các trường hợp như vậy, các phần tử của chuỗi có thể cung cấp các gợi ý quan trọng về dữ liệu là gì và cách sử dụng nó. Trong một số trường hợp, bạn có thể sử dụng vị trí của chuỗi (liên quan đến các chuỗi khác) để cung cấp ngữ cảnh cho một khu vực bộ nhớ không xác định. Ví dụ, nếu bạn tìm thấy các chuỗi liên quan đến URL và thư mục tệp tạm thời Internet, bạn có thể đã tìm thấy các đoạn của tệp nhật ký lịch sử Internet.

Cũng quan trọng để bạn hiểu là cách triển khai cụ thể của các chuỗi có thể gây ra những thách thức, tùy thuộc vào loại phân tích bộ nhớ đang được thực hiện. Ví dụ, vì chuỗi kiểu C ngầm định chứa một ký tự kết thúc đánh dấu cuối các ký tự được lưu trữ, việc trích xuất chuỗi thường được thực hiện bằng cách xử lý từng ký tự cho đến khi gặp ký tự kết thúc. Một số thách thức có thể xuất hiện nếu không tìm thấy ký tự kết thúc. Ví dụ, khi phân tích không gian địa chỉ vật lý của một hệ thống sử dụng bộ nhớ ảo được phân trang, bạn có thể gặp một chuỗi vượt qua ranh giới trang sang một trang không còn nằm trong bộ nhớ, điều này đòi hỏi xử lý hoặc thủ thuật đặc biệt để xác định kích thước thực sự của chuỗi.

Cách triển khai _UNICODE_STRING cũng có thể làm cho việc phân tích không gian địa chỉ vật lý trở nên phức tạp. Cấu trúc dữ liệu _UNICODE_STRING chỉ chứa siêu dữ liệu cho chuỗi (tức là địa chỉ bộ nhớ ảo bắt đầu, kích thước trong byte, v.v.). Do đó, nếu không thể thực hiện dịch vụ địa chỉ ảo, bạn không thể xác định nội dung thực tế của chuỗi. Tương tự, nếu bạn tìm thấy nội dung của một chuỗi thông qua các phương tiện khác, bạn có thể không thể xác định độ dài phù hợp của nó, vì siêu dữ liệu kích thước được lưu trữ một cách riêng biệt.

### Linked Lists (Danh sách liên kết)

Danh sách liên kết (linked-list) là một kiểu dữ liệu trừu tượng thường được sử dụng để lưu trữ một tập hợp các phần tử. Khác với mảng có kích thước cố định và bản ghi (record), danh sách liên kết được thiết kế để cung cấp một cấu trúc linh hoạt. Cấu trúc này hỗ trợ cập nhật động một cách hiệu quả và không giới hạn về số lượng phần tử có thể lưu trữ. Một khác biệt chính khác là danh sách liên kết không có chỉ mục và được thiết kế để cung cấp truy cập tuần tự thay vì truy cập ngẫu nhiên. Danh sách liên kết được thiết kế để hiệu quả hơn đối với các chương trình cần thao tác thay đổi bộ sưu tập lưu trữ bằng cách thêm, xóa hoặc sắp xếp các phần tử. Hiệu quả này được thực hiện bằng cách sử dụng liên kết để chỉ định các mối quan hệ giữa các phần tử và sau đó cập nhật các liên kết đó khi cần thiết. Phần tử đầu tiên của danh sách thường được gọi là "head" và phần tử cuối cùng là "tail". Bằng cách theo dõi liên kết từ "head" đến "tail" và đếm số phần tử, ta có thể xác định số lượng phần tử được lưu trữ trong danh sách liên kết.

#### Singly Linked List (Danh sách liên kết đơn)

Hình 2-6 minh họa một ví dụ về danh sách liên kết đơn của bốn phần tử. Mỗi phần tử của danh sách liên kết đơn được kết nối bởi một liên kết duy nhất với phần tử kế tiếp của nó và do đó, danh sách có thể được duyệt theo một hướng duy nhất. Như được hiển thị trong Hình 2-6, việc chèn các phần tử mới và xóa các phần tử khỏi danh sách chỉ đòi hỏi một số thao tác để cập nhật các liên kết.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-23%20210423.png)

Để hỗ trợ tính động của danh sách liên kết và giảm thiểu yêu cầu lưu trữ, các phiên bản ngôn ngữ lập trình C thường cấp phát và giải phóng bộ nhớ để lưu trữ các phần tử khi cần thiết. Kết quả là, bạn không thể giả định rằng các phần tử sẽ được lưu trữ liên tục trong bộ nhớ. Hơn nữa, thứ tự tuần tự của các phần tử không ngầm định liên quan đến vị trí bộ nhớ của chúng, như trường hợp của mảng. Thay vào đó, mỗi phần tử của danh sách được lưu trữ riêng rẽ và các liên kết được duy trì với các phần tử láng giềng. Trong một số phiên bản, các liên kết được lưu trữ nhúng trong phần tử (lưu trữ nội bộ). Trong các phiên bản khác, các nút của danh sách liên kết chứa các liên kết đến các nút láng giềng và một liên kết đến địa chỉ trong bộ nhớ nơi phần tử được lưu trữ (lưu trữ ngoại vi). Trong cả hai trường hợp, bạn thực hiện các liên kết giữa các nút bằng cách sử dụng con trỏ chứa địa chỉ bộ nhớ ảo của nút láng giềng. Do đó, để truy cập một phần tử bất kỳ trong danh sách, bạn phải duyệt qua danh sách liên kết bằng cách theo dõi các con trỏ tuần tự qua không gian địa chỉ ảo.

#### Doubly Linked List (Danh sách liên kết đôi/hai chiều)

Cũng có thể tạo một danh sách liên kết hai chiều (doubly linked list) trong đó mỗi phần tử lưu trữ hai con trỏ: một con trỏ tới phần tử trước đó trong chuỗi và một con trỏ tới phần tử kế tiếp của nó. Nhờ vậy, bạn có thể duyệt qua danh sách liên kết hai chiều cả theo chiều thuận và ngược lại. Như bạn sẽ thấy trong các ví dụ tiếp theo, các phiên bản danh sách liên kết có thể sử dụng nhiều cơ chế để đánh dấu đầu và đuôi của danh sách.

#### Cicular Linked List (Danh sách liên kết vòng)

Một phiên bản thường xuyên được sử dụng trong nhân Linux là danh sách liên kết vòng (circular linked list). Nó được gọi là danh sách liên kết vòng bởi vì liên kết cuối cùng lưu trữ với phần tử cuối cùng của danh sách (đuôi) trỏ đến nút ban đầu của danh sách (đầu). Điều này đặc biệt hữu ích cho các danh sách mà thứ tự không quan trọng. Một danh sách liên kết vòng được duyệt bằng cách bắt đầu từ một nút danh sách bất kỳ và dừng lại khi việc duyệt danh sách trở lại nút đó. Hình 2-7 cho thấy một ví dụ về danh sách liên kết vòng. Như được thảo luận chi tiết hơn trong các chương sau, loại danh sách liên kết này đã được sử dụng trong nhân Linux để tính toán quá trình (process accounting).

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-23%20210827.png)

#### Embedded Doubly Linked Lists (Danh sách liên kết đôi nhúng)

Khi phân tích việc tính toán quá trình (process accounting) trên Microsoft Windows, bạn thường gặp một phiên bản khác của danh sách liên kết: danh sách liên kết đôi nhúng (embedded doubly linked list). Chúng tôi gọi nó là "nhúng" vì nó sử dụng bộ nhớ trong (internal storage) để nhúng một cấu trúc dữ liệu _LIST_ENTRY64 vào phần tử đang được lưu trữ. Như được hiển thị trong mô tả định dạng _LIST_ENTRY64 trong Bảng 2-5, cấu trúc dữ liệu chỉ chứa hai thành viên: một con trỏ trỏ đến _LIST_ENTRY64 nhúng của phần tử kế tiếp (Flink) và một con trỏ trỏ đến _LIST_ENTRY64 nhúng của phần tử tiền nhiệm (Blink). Vì các liên kết lưu trữ địa chỉ của các cấu trúc _LIST_ENTRY64 nhúng khác, bạn tính toán địa chỉ cơ sở của phần tử chứa bằng cách trừ đi khoảng cách từ cấu trúc _LIST_ENTRY64 nhúng trong phần tử đó. Khác với danh sách liên kết vòng, phiên bản này sử dụng một _LIST_ENTRY64 riêng làm nút báo hiệu (sentinel node), được sử dụng chỉ để đánh dấu nơi bắt đầu và kết thúc của danh sách. Chúng tôi sẽ thảo luận chi tiết hơn về các phiên bản danh sách liên kết này và các phiên bản khác trong suốt quá trình đọc sách.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-23%20210847.png)

#### Lists in Physical and Virtual Memory (Danh sách trong bộ nhớ vật lý và bộ nhớ ảo)

Khi phân tích bộ nhớ, bạn thường gặp nhiều phiên bản của danh sách liên kết. Ví dụ, bạn có thể quét không gian địa chỉ vật lý để tìm dữ liệu giống với các phần tử được lưu trữ trong một danh sách cụ thể. Tuy nhiên, chỉ qua phân tích không gian địa chỉ vật lý, bạn không thể xác định liệu dữ liệu bạn tìm thấy có phải là thành viên hiện tại của danh sách hay không. Bạn không thể xác định một trật tự cho danh sách, cũng như không thể sử dụng các liên kết lưu trữ để tìm các phần tử láng giềng. Tuy nhiên, phân tích không gian địa chỉ vật lý cho phép bạn có thể tìm thấy các phần tử có thể đã bị xóa hoặc loại bỏ bí mật để làm rối phân tích. Các cấu trúc dữ liệu động, như danh sách liên kết, thường là mục tiêu thường xuyên của sự thay đổi độc hại vì chúng có thể dễ dàng bị thay đổi chỉ bằng cách cập nhật một số liên kết.

Ngoài phân tích không gian địa chỉ vật lý, bạn có thể tận dụng phân tích không gian địa chỉ ảo để dịch các con trỏ địa chỉ ảo và duyệt qua các liên kết giữa các nút. Sử dụng phân tích không gian địa chỉ ảo, bạn có thể nhanh chóng liệt kê các mối quan hệ giữa các phần tử trong danh sách và trích xuất thông tin quan trọng về trật tự của danh sách.

### Hash Tables (Bảng băm)

Bảng băm (hash tables) thường được sử dụng trong các trường hợp yêu cầu chèn và tìm kiếm hiệu quả, trong đó dữ liệu được lưu trữ dưới dạng cặp <khóa, phần tử>. Ví dụ, bảng băm được sử dụng trong khắp các hệ điều hành để lưu thông tin về các quy trình hoạt động, kết nối mạng, các hệ thống tập tin được gắn kết và các tệp đệm. Một cách triển khai phổ biến mà bạn thường gặp khi phân tích bộ nhớ liên quan đến bảng băm bao gồm bảng băm dạng mảng của danh sách liên kết, còn được gọi là bảng băm tràn chồng. Lợi ích của triển khai này là nó cho phép cấu trúc dữ liệu linh hoạt hơn. Một hàm băm, h(x), được sử dụng để chuyển đổi khóa thành chỉ số mảng, và những va chạm (collision, tức là các giá trị có cùng khóa) được lưu trữ trong danh sách liên kết liên kết với mục nhập trong bảng băm. Hình 2-8 thể hiện một ví dụ về bảng băm được triển khai dưới dạng mảng của danh sách liên kết.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-23%20211419.png)

Trong ví dụ, việc tìm kiếm một khóa cụ thể gây ra va chạm có thể yêu cầu duyệt qua một danh sách liên kết của các va chạm, nhưng điều này lý tưởng là nhanh hơn nhiều so với duyệt qua một danh sách của tất cả các khóa. Ví dụ, nếu một bảng băm được hỗ trợ bởi một mảng gồm 16.000 chỉ mục và bảng băm có 64.000 phần tử, hàm băm tối ưu sẽ đặt 4 phần tử vào mỗi chỉ mục của mảng. Khi thực hiện một tìm kiếm, khóa đã được băm sẽ trỏ đến một danh sách chỉ chứa 4 phần tử thay vì có thể duyệt qua 64.000 phần tử mỗi lần tra cứu.

Linux sử dụng bảng băm tràn chồng liên kết (được gọi là bảng băm ID quy trình) để liên kết các ID quy trình với cấu trúc quy trình. Chương 21 giải thích cách các công cụ phân tích bộ nhớ sử dụng bảng băm này để tìm các quy trình đang hoạt động.

### Trees (Cây)

Cây là một khái niệm lưu trữ động khác mà bạn có thể gặp khi phân tích bộ nhớ. Mặc dù các mảng, bản ghi, chuỗi và danh sách liên kết cung cấp cơ chế tiện lợi để biểu diễn tổ chức tuần tự của dữ liệu, một cây cung cấp một tổ chức dữ liệu có cấu trúc hơn trong bộ nhớ. Tổ chức này có thể được sử dụng để tăng hiệu suất đáng kể cho các hoạt động được thực hiện trên dữ liệu đã lưu trữ. Như được hiển thị trong các chương sau, cây được sử dụng trong các hệ điều hành khi hiệu suất là rất quan trọng. Phần này giới thiệu thuật ngữ và khái niệm cơ bản liên quan đến cây phân cấp (gốc), là lớp cây gặp phổ biến nhất khi phân tích bộ nhớ.

#### Hierarchical Trees  (Cây phân cấp)

Cây phân cấp thường được thảo luận dưới khái niệm của cây gia đình hoặc cây gen. Một cây phân cấp bao gồm một tập hợp các nút được sử dụng để lưu trữ các phần tử và một tập hợp các liên kết được sử dụng để kết nối các nút. Mỗi nút cũng có một khóa được sử dụng để sắp xếp các nút. Trong trường hợp của một cây phân cấp, một nút được đánh dấu là gốc, và các liên kết giữa các nút đại diện cho cấu trúc phân cấp thông qua mối quan hệ cha con. Các liên kết được sử dụng để kết nối một nút (cha) với các nút con của nó. Ví dụ, cây nhị phân là một cây phân cấp trong đó mỗi nút giới hạn không quá hai nút con. Một nút không có bất kỳ nút con nào (con đúng) được gọi là lá. Nút không phải lá (nút nội) không có cha là gốc. Bất kỳ nút nào trong cây và tất cả các con của nó tạo thành một cây con. Một chuỗi các liên kết được kết hợp để kết nối hai nút của cây được gọi là một đường đi giữa các nút đó. Một thuộc tính cơ bản của cây phân cấp là nó không có bất kỳ chu trình nào. Do đó, một đường duy nhất tồn tại giữa hai nút bất kỳ trong cây. Hình 2-9 hiển thị một biểu diễn đơn giản của một cây.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-23%20211438.png)

Hình 2-9 là một ví dụ về cây nhị phân chứa sáu nút (node) được sử dụng để lưu trữ các số nguyên. Trong trường hợp của cây nhị phân, các liên kết liên quan đến mỗi nút được gọi là liên kết trái (left link) và liên kết phải (right link), tương ứng. Vì tính đơn giản, khóa tương ứng với số nguyên đang được lưu trữ, và mỗi nút được tham chiếu bằng cách sử dụng khóa. Gốc (root) của cây là nút 4; nút 2 và 4 là các nút nội (internal node); và các nút lá (leaf node) là 1, 3 và 5. Kết hợp chuỗi các liên kết giữa các nút 4, 2 và 3 được sử dụng để tạo thành đường đi duy nhất từ nút 4 đến nút 3. Nút 2 và các con của nó, nút 1 và 3, được xem là một cây con gốc tại 2.

#### Tree Traversal (Duyệt cây)

Duyệt cây là một trong những thao tác quan trọng khi phân tích bộ nhớ và làm việc với cấu trúc dữ liệu phân cấp như cây. Khác với kiểu dữ liệu trừu tượng danh sách liên kết, trong đó bạn duyệt qua các nút bằng cách theo liên kết tới nút lá, mỗi nút của cây có thể duyệt đến nhiều nút con. Do đó, một thứ tự được sử dụng để xác định cách xử lý các cạnh đến các nút con. Các kỹ thuật duyệt cây phổ biến nhất và thường gặp nhất là duyệt tiền thứ tự (preorder), duyệt trung thứ tự (inorder) và duyệt hậu thứ tự (postorder). Mỗi kỹ thuật sau đây được thực hiện đệ quy tại mỗi nút gặp phải. Dùng duyệt tiền thứ tự, bạn trước tiên thăm nút hiện tại và sau đó duyệt qua các cây con từ trái qua phải. Trong khi duyệt trung thứ tự, bạn thăm cây con bên trái, sau đó thăm nút hiện tại và cuối cùng là các cây con còn lại từ trái qua phải. Cuối cùng, với duyệt hậu thứ tự, bạn duyệt qua cây con từ trái qua phải và sau đó thăm nút hiện tại. Thứ tự duyệt của các nút là: Gốc, Trái, Phải cho duyệt tiền thứ tự; Trái, Gốc, Phải cho duyệt trung thứ tự; Trái, Phải, Gốc cho duyệt hậu thứ tự. Mỗi trong các kỹ thuật duyệt cây này có thể được thực hiện đệ quy cho mỗi nút gặp phải trong cây. Sự lựa chọn của kỹ thuật duyệt phụ thuộc vào công việc hoặc phân tích cụ thể đang được thực hiện và thứ tự xử lý nút mong muốn.

```
Preorder: 4, 2, 1, 3, 6, 5
Inorder: 1, 2, 3, 4, 5, 6
Postorder: 1, 3, 2, 5, 6, 4
````

Các cài đặt của cây trong ngôn ngữ lập trình C có nhiều đặc điểm giống với danh sách liên kết. Sự khác biệt chính là số lượng và loại liên kết giữa các nút và cách thức sắp xếp các liên kết đó. Giống như cài đặt danh sách liên kết, chúng ta tập trung vào cây trong đó bộ nhớ cho các nút của cây được cấp phát và giải phóng một cách động khi cần. Chúng ta cũng tập trung chủ yếu vào cài đặt rõ ràng sử dụng bộ nhớ nội trong để nhúng các liên kết vào các phần tử được lưu trữ. Những liên kết này được triển khai dưới dạng các cạnh trực tiếp, trong đó mỗi nút giữ các con trỏ tới các địa chỉ bộ nhớ ảo của các nút liên quan.

#### Analyzing Trees in Memory (Phân tích cây trong bộ nhớ)

Phân tích cây trong bộ nhớ cũng gặp các thách thức tương tự như khi phân tích danh sách liên kết. Ví dụ, phân tích bộ nhớ vật lý cung cấp khả năng tìm thấy các phiên bản của các phần tử đã lưu trữ hoặc từng lưu trữ rải rác trong bộ nhớ, nhưng không có ngữ cảnh để phân biệt mối quan hệ giữa các phần tử đó. Trong khi đó, bạn có thể sử dụng phân tích bộ nhớ ảo để duyệt cây để trích xuất mối quan hệ giữa các nút và các phần tử đã lưu trữ. Trong trường hợp của cây được sắp xếp, nếu bạn biết thứ tự duyệt hoặc có thể xác định nó từ tên các trường, bạn có thể trích xuất danh sách được sắp xếp các phần tử. Bằng cách kết hợp cấu trúc tổng thể của cây với tiêu chí sắp xếp, bạn có thể thậm chí tìm ra thông tin về cách các phần tử được thêm hoặc xóa khỏi cây khi nó phát triển theo thời gian.

Cuốn sách này đề cập đến nhiều biến thể cây khác nhau và cách chúng được sử dụng trong hệ điều hành. Ví dụ, trình quản lý bộ nhớ của Windows sử dụng cây mô tả địa chỉ ảo (VAD) để tìm kiếm hiệu quả các phạm vi bộ nhớ được sử dụng bởi một quy trình. Cây VAD là một ví dụ về cây nhị phân tự cân bằng sử dụng dải địa chỉ bộ nhớ như là khóa. Một cách trực quan, điều này có nghĩa là các nút chứa các dải địa chỉ bộ nhớ thấp hơn được tìm thấy trong cây con trái của một nút, và các nút chứa các dải địa chỉ cao hơn được tìm thấy trong cây con phải. Như bạn sẽ thấy trong Chương 7, sử dụng những con trỏ này và phương pháp duyệt theo thứ tự, bạn có thể trích xuất một danh sách được sắp xếp của các dải địa chỉ bộ nhớ mô tả trạng thái của không gian địa chỉ ảo của quy trình.

## Tổng quan

Các cấu trúc dữ liệu đóng một vai trò quan trọng trong phân tích bộ nhớ. Ở mức cơ bản nhất, chúng giúp chúng ta hiểu được những byte ngẫu nhiên trong một bản ghi bộ nhớ vật lý. Bằng cách hiểu mối quan hệ giữa các dữ liệu (tức là các nút trong một cây, các phần tử trong một danh sách, các ký tự trong một chuỗi), bạn có thể bắt đầu xây dựng một biểu đồ biểu diễn chính xác và đầy đủ hơn về bằng chứng. Hơn nữa, việc hiểu về cách cụ thể mà hệ điều hành triển khai một cấu trúc dữ liệu trừu tượng là quan trọng để tìm hiểu tại sao một số cuộc tấn công (mà chi phối các cấu trúc) thành công và làm thế nào các công cụ phân tích bộ nhớ có thể giúp bạn phát hiện các cuộc tấn công như vậy.
