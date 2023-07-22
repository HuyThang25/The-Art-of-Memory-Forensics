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
0000020: 2e6f 7267 0000 0000 										 .org....
```

Các cấu trúc C-style là một trong những cấu trúc dữ liệu quan trọng nhất gặp phải khi thực hiện phân tích bộ nhớ. Sau khi xác định được địa chỉ cơ sở của một cấu trúc cụ thể, bạn có thể tận dụng định nghĩa của cấu trúc như một mẫu để trích xuất và diễn giải các phần tử của bản ghi. Một điều bạn phải nhớ là trình biên dịch có thể thêm các byte lùi vào một số trường của cấu trúc được lưu trữ trong bộ nhớ. Trình biên dịch thực hiện việc này để bảo tồn sự căn chỉnh của các trường và tăng hiệu suất CPU.

Ví dụ, chúng ta có thể sửa đổi kiểu cấu trúc từ Bảng 2-3 để loại bỏ trường port. Nếu chương trình được biên dịch lại với trình biên dịch được cấu hình để căn chỉnh trên ranh giới 32 bit, chúng ta sẽ tìm thấy dữ liệu sau đây trong bộ nhớ:

```
0000000: 0100 0000 ad3d de09 7777 772e 766f 6c61 ...P.>X.www.vola
0000010: 7469 6c69 7479 666f 756e 6461 7469 6f6e tilityfoundation
0000020: 2e6f 7267 0000 0000 										 .org....
```

Trong kết quả, bạn có thể thấy rằng mặc dù đã loại bỏ trường `port`, các trường còn lại (`id`, `addr`, `hostname`) vẫn được tìm thấy tại các vị trí lùi giống như trước đó (0, 4, 8). Bạn cũng sẽ nhận thấy rằng các byte từ 2 đến 3 (chứa giá trị 0000), trước đây lưu giữ giá trị cổng, bây giờ được sử dụng hoàn toàn cho việc padding.

Như bạn sẽ thấy trong các chương sau, định nghĩa của một cấu trúc dữ liệu và các ràng buộc liên quan đến các giá trị có thể được lưu trữ trong một trường cũng có thể được sử dụng như một mẫu để khắc phục các trường hợp có thể của cấu trúc trực tiếp từ bộ nhớ.

