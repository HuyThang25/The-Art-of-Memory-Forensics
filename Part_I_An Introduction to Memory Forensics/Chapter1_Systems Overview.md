Chương này cung cấp một tổng quan chung về các thành phần phần cứng và cấu trúc hệ điều hành ảnh hưởng đến phân tích bộ nhớ. Mặc dù các chương tiếp theo sẽ thảo luận về các chi tiết thực hiện liên quan đến các hệ điều hành cụ thể, chương này cung cấp thông tin nền hữu ích cho những người mới vào lĩnh vực này hoặc có thể cần làm mới kiến thức nhanh chóng. Chương bắt đầu bằng việc nhấn mạnh các khía cạnh quan trọng của kiến trúc phần cứng và kết thúc bằng việc cung cấp một tổng quan về các nguyên tắc thông thường của hệ điều hành. Các khái niệm và thuật ngữ được thảo luận trong chương này thường được đề cập trong toàn bộ phần còn lại của cuốn sách.

## Digital Environment (Môi trường kỹ thuật số)

Cuốn sách này tập trung vào điều tra các sự kiện diễn ra trong môi trường kỹ thuật số. Trong bối cảnh của môi trường kỹ thuật số, phần cứng cơ bản cuối cùng quyết định các ràng buộc về những gì một hệ thống cụ thể có thể thực hiện. Nhiều mặt khác, điều này tương tự như cách luật vật lý ràng buộc môi trường vật lý. Ví dụ, các nhà điều tra hiện trường vật lý hiểu về các luật vật lý liên quan đến chất lỏng có thể sử dụng các vết máu hoặc mô Figure phân tán để hỗ trợ hoặc phủ nhận những yêu cầu về một vụ án cụ thể. Bằng cách áp dụng kiến thức về thế giới vật lý, các nhà điều tra có cái nhìn sâu sắc về cách hoặc tại sao một hiện vật cụ thể liên quan đến cuộc điều tra. Tương tự, trong môi trường kỹ thuật số, phần cứng cơ bản xác định các hướng dẫn có thể được thực thi và các tài nguyên có thể được truy cập. Các nhà điều tra có thể xác định các thành phần phần cứng độc đáo của một hệ thống và tác động của những thành phần đó có thể ảnh hưởng đến phân tích, họ đang ở vị trí tốt nhất để thực hiện một cuộc điều tra hiệu quả.

Trên hầu hết các nền tảng, phần cứng được truy cập thông qua một lớp phần mềm được gọi là hệ điều hành, điều khiển xử lý, quản lý tài nguyên và tạo điều kiện cho việc giao tiếp với các thiết bị bên ngoài. Hệ điều hành phải đối mặt với các chi tiết cấp thấp của bộ xử lý, các thiết bị và phần cứng bộ nhớ được cài đặt trong hệ thống cụ thể. Thông thường, hệ điều hành cũng thực hiện một tập hợp các dịch vụ và giao diện cấp cao xác định cách phần cứng có thể được truy cập bởi các chương trình của người dùng.

Trong quá trình điều tra, bạn tìm kiếm các hiện vật mà phần mềm hoặc người dùng nghi ngờ có thể đã đưa vào môi trường kỹ thuật số và cố gắng xác định cách môi trường kỹ thuật số đã thay đổi để phản ứng với những hiện vật đó. Sự quen thuộc của một nhà điều tra kỹ thuật số với phần cứng và hệ điều hành của một hệ thống cung cấp một khung tham chiếu quan trọng trong quá trình phân tích và xây dựng lại các sự kiện.

## PC  Architecture (Kiến trúc PC)

Phần này cung cấp tổng quan chung về cơ bản về phần cứng mà những nhà điều tra kỹ thuật số quan tâm đến phân tích bộ nhớ nên quen thuộc. Đặc biệt, cuộc thảo luận tập trung vào kiến trúc phần cứng tổng quát của máy tính cá nhân (PC). Chúng ta chủ yếu sử dụng thuật ngữ liên quan đến các hệ thống dựa trên Intel. Điều quan trọng cần lưu ý là thuật ngữ đã thay đổi theo thời gian và các chi tiết triển khai liên tục được cải tiến để cải thiện chi phí và hiệu suất. Mặc dù các công nghệ cụ thể có thể thay đổi, các chức năng chính mà các thành phần này thực hiện vẫn không thay đổi.


> **LƯU Ý:** Chúng tôi chỉ đề cập đến PC một cách tổng quát là máy tính với bộ vi xử lý Intel hoặc tương thích có thể chạy hệ điều hành Windows, Linux hoặc Mac OS X.

### Physical Organization (Tổ chức Vật lý)

Một PC bao gồm các mạch in được nối với nhau để kết nối các thành phần khác nhau và cung cấp các kết nối cho các thiết bị ngoại vi. Bảng chính trong loại hệ thống này, được gọi là bo mạch chủ, cung cấp các kết nối cho phép các thành phần của hệ thống giao tiếp với nhau. Những kênh giao tiếp này thường được gọi là bus máy tính. Phần này tập trung vào các thành phần và bus mà một nhà điều tra cần phải quen thuộc. Figure 1-1 minh họa cách các thành phần khác nhau được tổ chức theo cách thông thường trong phần này.

#### CPU và MMU

Hai thành phần quan trọng nhất trên bo mạch chủ là bộ xử lý, thực hiện các chương trình, và bộ nhớ chính, tạm thời lưu trữ các chương trình đã thực hiện và dữ liệu liên quan của chúng. Bộ xử lý thường được gọi là đơn vị xử lý trung tâm (CPU). CPU truy cập vào bộ nhớ chính để lấy các hướng dẫn và sau đó thực hiện các hướng dẫn đó để xử lý dữ liệu.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-21%20144154.png)

Đọc từ bộ nhớ chính thường chậm hơn đáng kể so với việc đọc từ bộ nhớ riêng của CPU. Do đó, các hệ thống hiện đại sử dụng nhiều lớp bộ nhớ nhanh, gọi là bộ nhớ cache, để giúp bù đắp sự chênh lệch này. Mỗi cấp độ cache (L1, L2 và tiếp theo) có tốc độ chậm hơn và dung lượng lớn hơn so với các cấp trước. Trong hầu hết các hệ thống, các cache này được tích hợp vào bộ xử lý và mỗi lõi của nó. Nếu dữ liệu không được tìm thấy trong cache cụ thể, dữ liệu phải được truy xuất từ cache cấp độ tiếp theo hoặc bộ nhớ chính.

Bộ xử lý phụ thuộc vào đơn vị quản lý bộ nhớ (MMU) của nó để giúp tìm nơi lưu trữ dữ liệu. MMU là đơn vị phần cứng dịch địa chỉ mà bộ xử lý yêu cầu thành địa chỉ tương ứng trong bộ nhớ chính. Như chúng ta sẽ mô tả sau trong chương này, các cấu trúc dữ liệu để quản lý việc dịch địa chỉ cũng được lưu trữ trong bộ nhớ chính. Bởi vì một dịch địa chỉ cụ thể có thể yêu cầu nhiều hoạt động đọc bộ nhớ, bộ xử lý sử dụng một bộ nhớ cache đặc biệt, được gọi là bộ đệm tìm kiếm dịch (TLB), cho bảng dịch MMU. Trước mỗi truy cập bộ nhớ, TLB được tham khảo trước khi yêu cầu MMU thực hiện một hoạt động dịch địa chỉ tốn kém.

Chương 4 sẽ thảo luận thêm về cách các cache này và TLB có thể ảnh hưởng đến việc thu thập bằng chứng bộ nhớ trong lĩnh vực pháp y số.

#### North and Southbridge

Bộ xử lý (CPU) dựa vào bộ điều khiển bộ nhớ để quản lý việc giao tiếp với bộ nhớ chính. Bộ điều khiển bộ nhớ chịu trách nhiệm điều phối các yêu cầu có thể xảy ra đồng thời cho bộ nhớ hệ thống từ bộ xử lý (hoặc các bộ xử lý) và các thiết bị. Bộ điều khiển bộ nhớ có thể được triển khai trên một chip riêng biệt hoặc tích hợp trong bộ xử lý chính. Trước đây trên các máy tính cá nhân cũ hơn, bộ xử lý kết nối với northbridge (trung tâm điều khiển bộ nhớ) bằng cổng trước và northbridge kết nối với bộ nhớ chính thông qua bus bộ nhớ. Các thiết bị (ví dụ: card mạng và điều khiển ổ đĩa) được kết nối thông qua một chip khác, gọi là southbridge hoặc trung tâm điều khiển vào/ra, có một kết nối chung duy nhất với northbridge để truy cập vào bộ nhớ và CPU.

Để cải thiện hiệu suất và giảm chi phí cho các hệ thống mới hơn, hầu hết các khả năng liên quan đến trung tâm điều khiển bộ nhớ hiện được tích hợp vào bộ xử lý. Các chức năng chipset còn lại, trước đây được triển khai trong southbridge, tập trung vào một chip được gọi là trung tâm điều khiển nền tảng.

#### Direct Memory Access ( Truy cập bộ nhớ trực tiếp)

Để cải thiện hiệu suất tổng thể, hầu hết các hệ thống hiện đại cung cấp khả năng truyền trực tiếp dữ liệu từ các thiết bị I/O (Input/Output) lưu trữ trong bộ nhớ hệ thống mà không cần sự can thiệp của bộ xử lý chính (CPU). Khả năng này được gọi là "Direct Memory Access" (DMA). Trước khi có DMA, CPU sẽ bị tiêu tốn hoàn toàn trong quá trình truyền dữ liệu I/O và thường phải hoạt động như một trung gian. Trong kiến trúc hiện đại, CPU có thể khởi tạo việc truyền dữ liệu và cho phép bộ điều khiển DMA quản lý việc truyền dữ liệu, hoặc một thiết bị I/O có thể khởi tạo việc truyền mà không cần sự can thiệp của CPU.

Ngoài tác động rõ ràng về hiệu suất hệ thống, DMA còn có những tác động quan trọng đối với pháp y số bộ nhớ. Nó cung cấp một cơ chế để truy cập trực tiếp nội dung của bộ nhớ vật lý từ một thiết bị ngoại vi mà không liên quan đến phần mềm không đáng tin cậy đang chạy trên máy. Ví dụ, bus PCI hỗ trợ các thiết bị hoạt động như người chủ của bus, có nghĩa là chúng có thể yêu cầu kiểm soát bus để khởi chạy giao dịch. Do đó, một thiết bị PCI với chức năng chủ bus và hỗ trợ DMA có thể truy cập vào bộ nhớ hệ thống mà không liên quan đến CPU.

Một ví dụ khác là giao diện IEEE 1394, thường được gọi là Firewire. Chip điều khiển máy chủ IEEE 1394 cung cấp một bus mở rộng tuân theo ngang hàng dành cho việc kết nối các thiết bị ngoại vi tốc độ cao với máy tính cá nhân. Mặc dù giao diện IEEE 1394 thường chỉ có sẵn một cách tự nhiên trên các hệ thống cao cấp hơn, bạn có thể thêm giao diện này vào cả máy tính để bàn và máy tính xách tay bằng thẻ mở rộng.

### Volatile Memory (RAM)

Bộ nhớ chính (RAM) là bộ nhớ chính của máy tính được triển khai bằng bộ nhớ truy cập ngẫu nhiên (RAM), nơi lưu trữ mã và dữ liệu mà bộ xử lý truy cập và lưu trữ một cách hoạt động. Trái với lưu trữ truy cập tuần tự thường liên quan đến ổ đĩa, truy cập ngẫu nhiên đề cập đến tính chất của việc truy cập dữ liệu trong thời gian hỗn hợp bất kể dữ liệu được lưu trữ ở vị trí nào trên phương tiện lưu trữ. Bộ nhớ chính trong hầu hết các máy tính là bộ nhớ động (DRAM). Nó là động vì nó sử dụng sự khác biệt giữa trạng thái đã nạp và không nạp của một tụ điện để lưu trữ một bit dữ liệu. Để tụ điện duy trì trạng thái này, nó phải được làm mới định kỳ - một nhiệm vụ mà bộ điều khiển bộ nhớ thường thực hiện.

RAM được coi là bộ nhớ thoáng qua vì nó yêu cầu điện năng để dữ liệu có thể tiếp tục truy cập. Do đó, trừ khi có các cuộc tấn công khởi động lạnh (https://citp.princeton.edu/research/memory), sau khi máy tính được tắt nguồn, dữ liệu trong bộ nhớ thoáng qua sẽ bị mất. Đây là lý do chính tại sao chiến thuật đáng chú ý "rút phích cắm" không được khuyến nghị nếu bạn dự định bảo tồn chứng cứ về trạng thái hiện tại của hệ thống.

### (CPU Architetures) Kiến trúc CPU

Như đã đề cập trước đó, CPU là một trong những thành phần quan trọng nhất của hệ thống máy tính. Để hiệu quả trong việc trích xuất cấu trúc từ bộ nhớ vật lý và hiểu cách mã độc có thể gây nguy hiểm đến bảo mật hệ thống, bạn nên có kiến thức vững chắc về mô Figure lập trình mà CPU cung cấp để truy cập bộ nhớ. Mặc dù phần trước tập trung vào tổ chức vật lý của phần cứng, phần này tập trung vào tổ chức logic được tiết lộ cho hệ điều hành. Phần này bắt đầu bằng việc thảo luận về một số chủ đề chung liên quan đến kiến trúc CPU và sau đó là những tính năng liên quan đến phân tích bộ nhớ. Cụ thể, phần này tập trung vào tổ chức 32-bit (IA-32) và 64-bit (Intel 64), được chỉ định trong Tài liệu Phát triển Phần mềm Kiến trúc Intel 64 và IA-32 (http://www.intel.com/content/dam/www/public/us/en/documents/
manuals/64-ia-32-architectures-software-developer-manual-325462.pdf).

#### Address Spaces (Không gian địa chỉ)

Để CPU thực hiện các chỉ thị và truy cập dữ liệu được lưu trữ trong bộ nhớ chính, nó phải xác định một địa chỉ duy nhất cho dữ liệu đó. Các bộ vi xử lý được thảo luận trong cuốn sách này sử dụng cách địa chỉ byte, và bộ nhớ được truy cập dưới dạng một chuỗi byte. Không gian địa chỉ đề cập đến một loạt địa chỉ hợp lệ được sử dụng để xác định dữ liệu được lưu trữ trong một phạm vi bộ nhớ cố định. Cụ thể, cuốn sách này tập trung vào các hệ thống xác định một byte là một lượng 8 bit. Cấu trúc địa chỉ này thường bắt đầu từ byte 0 và kết thúc ở địa chỉ lùi của byte cuối cùng trong bộ nhớ đã được cấp phát. Không gian địa chỉ duy nhất liên tục được tiết lộ cho một chương trình đang chạy được gọi là không gian địa chỉ tuyến tính. Dựa trên các mô Figure bộ nhớ được thảo luận trong cuốn sách và việc sử dụng trang, chúng tôi sử dụng các thuật ngữ địa chỉ tuyến tính và địa chỉ ảo một cách thay thế. Chúng tôi sử dụng thuật ngữ không gian địa chỉ vật lý để tham chiếu đến các địa chỉ mà bộ vi xử lý yêu cầu để truy cập bộ nhớ vật lý. Các địa chỉ này được thu được bằng cách chuyển đổi các địa chỉ tuyến tính thành các địa chỉ vật lý, sử dụng một hoặc nhiều bảng trang (được thảo luận chi tiết hơn trong thời gian sớm). Các phần tiếp theo sẽ thảo luận về cách thức thực hiện không gian địa chỉ bộ nhớ trong các kiến trúc bộ vi xử lý khác nhau.

> **LƯU Ý:** Khi làm việc với các bản nhớ dữ liệu thô (xem Chương 4), địa chỉ vật lý là về cơ bản một độ lệch vào tệp bộ nhớ.

### Intel IA-32 Architecture (Kiến trúc Intel IA-32)

Kiến trúc IA-32 thường đề cập đến dòng kiến trúc x86 hỗ trợ tính toán 32 bit. Cụ thể, nó chỉ định tập lệnh và môi trường lập trình cho các bộ vi xử lý 32 bit của Intel. IA-32 là một máy nhỏ endian sử dụng cách địa chỉ byte. Phần mềm chạy trên bộ vi xử lý IA-32 có thể có không gian địa chỉ tuyến tính và không gian địa chỉ vật lý lên đến 4GB. Như bạn sẽ thấy sau này, bạn có thể mở rộng kích thước bộ nhớ vật lý lên 64GB bằng tính năng IA-32 Physical Address Extension (PAE). Phần này và phần còn lại của cuốn sách tập trung vào hoạt động ở chế độ bảo vệ của kiến trúc IA-32, đó là chế độ hoạt động cung cấp hỗ trợ cho các tính năng như bộ nhớ ảo, phân trang, cấp độ đặc quyền và phân đoạn. Đây là trạng thái chính của bộ vi xử lý và cũng là chế độ mà hầu hết các hệ điều hành hiện đại thực thi.

#### Registers (Thanh ghi)

Kiến trúc IA-32 xác định một số lượng nhỏ bộ nhớ cực kỳ nhanh, được gọi là registers, mà CPU sử dụng để lưu trữ tạm thời trong quá trình xử lý. Mỗi lõi bộ vi xử lý chứa 8 thanh ghi 32 bit để thực hiện các phép toán logic và số học, cũng như một số thanh ghi khác để điều khiển hành vi của bộ vi xử lý. Phần này nêu bật một số thanh ghi điều khiển liên quan đến phân tích bộ nhớ.

Thanh ghi EIP, còn được gọi là bộ đếm chương trình, chứa địa chỉ tuyến tính của lệnh tiếp theo được thực thi. Kiến trúc IA-32 cũng có năm thanh ghi điều khiển khác nhau chỉ định cấu Figure của bộ vi xử lý và các đặc điểm của tác vụ đang thực thi. CR0 chứa các cờ điều khiển chế độ hoạt động của bộ vi xử lý, bao gồm một cờ cho phép phân trang. CR1 được dành riêng và không nên truy cập. CR2 chứa địa chỉ tuyến tính gây ra lỗi trang. CR3 chứa địa chỉ vật lý của cấu trúc ban đầu được sử dụng cho việc dịch địa chỉ. Nó được cập nhật trong quá trình chuyển đổi ngữ cảnh khi một tác vụ mới được lên lịch. CR4 được sử dụng để kích hoạt các tiện ích kiến trúc, bao gồm PAE.

#### Segmentation (Phân đoạn)

Bộ vi xử lý IA-32 thực hiện hai cơ chế quản lý bộ nhớ: phân đoạn và phân trang. Phân đoạn chia không gian địa chỉ tuyến tính 32 bit thành nhiều đoạn có độ dài biến đổi. Tất cả các tham chiếu bộ nhớ IA-32 đều được địa chỉ hóa bằng cách sử dụng một bộ chọn đoạn 16 bit, xác định một miêu tả đoạn cụ thể, và một phần tử tuyến tính 32 bit vào đoạn đã chỉ định. Một miêu tả đoạn là một cấu trúc dữ liệu cư trú bộ nhớ xác định vị trí, kích thước, loại và quyền cho một đoạn đã cho. Mỗi lõi bộ vi xử lý chứa hai thanh ghi đặc biệt GDTR và LDTR, trỏ đến bảng các miêu tả đoạn, gọi là Bảng Miêu tả Toàn cầu (GDT) và Bảng Miêu tả Cục bộ, tương ứng. Các thanh ghi phân đoạn CS (cho mã), SS (cho ngăn xếp) và DS, ES, FS và GS (mỗi loại dữ liệu) luôn phải chứa bộ chọn đoạn hợp lệ.

Trong khi phân đoạn là bắt buộc, các hệ điều hành được thảo luận trong cuốn sách này che giấu địa chỉ hóa phân đoạn bằng cách định nghĩa một tập hợp các đoạn trùng lấp có địa chỉ cơ sở là không, tạo ra cảm giác của một không gian địa chỉ tuyến tính "bằng phẳng" liên tục duy nhất. Tuy nhiên, bảo vệ phân đoạn vẫn được áp dụng cho mỗi đoạn và các miêu tả đoạn riêng phải được sử dụng cho các tham chiếu mã và dữ liệu.

>**GHI CHÚ:** Do hầu hết các hệ điều hành không tận dụng các mô Figure phân đoạn IA-32 phức tạp hơn, nên địa chỉ hóa phân đoạn đã bị vô hiệu hóa trong chế độ 64 bit. Cụ thể, các địa chỉ cơ sở đoạn được coi như tự động bằng không. Lưu ý rằng bảo vệ phân đoạn vẫn được áp dụng trong chế độ 64 bit.

#### Paging (Phân trang)

Phân trang (Paging) cung cấp khả năng ảo hóa không gian địa chỉ tuyến tính. Nó tạo môi trường thực thi trong đó không gian địa chỉ tuyến tính lớn được mô phỏng bằng một lượng nhỏ bộ nhớ vật lý và lưu trữ đĩa. Mỗi không gian địa chỉ tuyến tính 32-bit được chia thành các phần có độ dài cố định, gọi là trang, có thể được ánh xạ vào bộ nhớ vật lý theo thứ tự tùy ý. Khi một chương trình cố gắng truy cập một địa chỉ tuyến tính, ánh xạ này sử dụng các bảng trang và bảng địa chỉ cư trú trong bộ nhớ để chuyển đổi địa chỉ tuyến tính thành địa chỉ vật lý. Trong trường hợp thông thường với trang kích thước 4KB, như được hiển thị trong Figure 1-2, địa chỉ tuyến tính 32-bit được chia thành ba phần, mỗi phần được sử dụng như một chỉ mục trong cấu trúc phân cấp của hệ thống trang hay trang vật lý tương ứng.

Kiến trúc IA-32 cũng hỗ trợ các trang kích thước 4MB, việc chuyển đổi này chỉ yêu cầu một bảng trang. Bằng cách sử dụng các cấu trúc phân trang khác nhau cho các tiến trình khác nhau, một hệ điều hành có thể cung cấp cho mỗi tiến trình giao diện của môi trường lập trình đơn qua không gian địa chỉ tuyến tính ảo. Figure 1-3 hiển thị một phân chia chi tiết hơn về các bit chuyển đổi một địa chỉ ảo thành một địa chỉ vật lý.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-21%20211815.png)

Để tính địa chỉ mục bảng trang (PDE-page directory entry), bạn kết hợp các bit 31:12 từ thanh ghi CR3 với các bit 31:22 từ địa chỉ ảo. Sau đó, bạn tìm vị trí mục bảng trang (PTE-page table entry) bằng cách kết hợp các bit 31:12 từ PDE với các bit 21:12 của địa chỉ ảo. Cuối cùng, bạn có thể lấy được địa chỉ vật lý (PA) bằng cách kết hợp các bit 31:12 của PTE với các bit 11:0 của địa chỉ ảo. Bạn sẽ thấy các phép tính này được áp dụng trong phần tiếp theo khi thực hiện chuyển đổi một địa chỉ bằng tay.

#### Chuyển đổi địa chỉ (Address Translation)

Toàn bộ việc hỗ trợ một kiến trúc CPU có hỗ trợ bộ nhớ ảo, các phần mềm pháp y như Volatility phải mô phỏng không gian địa chỉ ảo và xử lý chuyển đổi địa chỉ ảo sang địa chỉ vật lý một cách minh bạch. Thực hiện các bước chuyển đổi địa chỉ bằng tay giúp củng cố hiểu biết về cách các công cụ này hoạt động và cung cấp nền tảng để khắc phục sự cố không mong muốn.

>**LƯU Ý:** Các lớp Python trong Volatility xử lý chuyển đổi địa chỉ ảo sang vật lý xuất hiện một phương thức được gọi là vtop (virtual to physical). Người gọi truyền hàm một địa chỉ ảo và nó trả về độ lệch vật lý, tính toán bằng các bước mô tả trong phần này. Tương tự, nếu bạn đang làm việc với trình gỡ lỗi của Microsoft (WinDbg), bạn có thể sử dụng lệnh !vtop.

Để phân tích, ta giả định bạn đang phân tích một trong các mẫu bộ nhớ, ENG-USTXHOU-148, được bao gồm trong cuộc thách thức forensics của Jack Crook vào tháng 11 năm 2012 (xem http://blog.handlerdiaries.com/?p=14). Trong quá trình phân tích, bạn đã tìm thấy một tham chiếu đến địa chỉ ảo, 0x10016270, trong không gian địa chỉ ảo của tiến trình svchost.exe có PID 1024. Địa chỉ cơ sở của bảng điều khiển trang (CR3) cho PID 1024 là 0x7401000. Bạn muốn tìm địa chỉ vật lý tương ứng để xem liệu có dữ liệu nào khác ở gần nhau không.

Bước đầu tiên là chuyển đổi địa chỉ ảo, 0x10016270, từ hệ thập phân sang định dạng nhị phân vì bạn sẽ làm việc với các dãy bit địa chỉ:
```0001 0000 0000 0001 0110 0010 0111 0000```

Tiếp theo, bạn phân tách địa chỉ thành các độ lệch liên quan được sử dụng trong quá trình chuyển đổi. Dữ liệu này được hiển thị trong Bảng 1-1.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-21%20213234.png)

Như đã thấy trong Figure 1-2 và Figure 1-3, bạn có thể tính toán địa chỉ vật lý của PDE bằng cách nhân chỉ số thư mục trang (page directory index) với kích thước mục (entry size) (4 byte) và sau đó cộng thêm vào địa chỉ cơ sở bảng điều khiển trang (page directory base), 0x7401000. 10 bit từ địa chỉ ảo có thể chỉ mục tới 1024 mục (210) trong bảng điều khiển trang.

	Địa chỉ PDE = 0x40 * 4 + 0x7401000 = 0x7401100

Tiếp theo, bạn phải đọc giá trị từ bộ nhớ vật lý được lưu trữ tại địa chỉ PDE. Hãy chắc chắn tính đến việc giá trị được lưu trữ theo định dạng little endian. Lúc này, bạn biết giá trị của PDE là 0x17bf9067. Dựa vào Figure 1-3, bạn biết rằng các bit 31:12 của PDE cung cấp địa chỉ vật lý cho cơ sở của bảng trang (page table). Các bit 21:12 của địa chỉ ảo cung cấp chỉ mục của bảng trang vì bảng trang gồm 1024 mục (210). Bạn có thể tính địa chỉ vật lý của PTE bằng cách nhân kích thước mục (4 byte) với chỉ mục bảng trang và sau đó cộng giá trị đó vào cơ sở bảng trang.

	Địa chỉ PTE = 0x16 * 4 + 0x17bf9000 = 0x17bf9058 

Giá trị của PTE được lưu trữ tại địa chỉ đó là 0x170b6067. Dựa vào Figure 1-3, bạn biết rằng các bit 31:12 của địa chỉ vật lý là từ PTE và các bit 11:0 là từ địa chỉ ảo. Do đó, địa chỉ vật lý cuối cùng được chuyển đổi như sau:

	Địa chỉ vật lý = 0x170b6000 + 0x270 = 0x170b6270

Sau khi hoàn thành quá trình dịch, bạn đã tìm thấy địa chỉ ảo 0x10016270 tương ứng với địa chỉ vật lý 0x170b6270. Figure 1-4 cung cấp một Figure ảnh minh họa về các bước đã được thực hiện. Bạn có thể tìm địa chỉ byte trong mẫu bộ nhớ và tìm kiếm các đối tượng liên quan có thể ở gần nhau. Đây là cùng quá trình mà không gian địa chỉ IA32PagedMemory của Volatility thực hiện mỗi khi truy cập vào địa chỉ ảo. Trong phần văn bản tiếp theo, bạn sẽ thấy làm thế nào để mở rộng quá trình này để hỗ trợ không gian địa chỉ ảo lớn hơn.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-21%20214407.png)

>**Lưu ý:** cũng rất quan trọng để nhấn mạnh một số bit cờ được lưu trữ trong các mục của cấu trúc trang (paging structure) ảnh hưởng trực tiếp đến quá trình dịch cho tất cả ba chế độ phân trang được thảo luận trong cuốn sách. Quá trình dịch địa chỉ sẽ kết thúc nếu một mục của cấu trúc trang có bit 0 (cờ hiện tại) được thiết lập thành 0, điều này chỉ ra "không có" (not present). Do đó, nó tạo ra một ngoại lệ page fault. Nếu bạn đang xử lý một cấu trúc trang trung gian, có nghĩa là còn lại nhiều hơn 12 bit trong địa chỉ tuyến tính, bit 7 của mục cấu trúc trang hiện tại được sử dụng như cờ kích thước trang (PS). Khi bit này được thiết lập, nó chỉ định rằng các bit còn lại ánh xạ vào một trang của bộ nhớ thay vì một cấu trúc trang khác.

#### Physical Address Extension 

Physical Address Extension (PAE) là một phần mở rộng của cơ chế phân trang trong kiến trúc IA-32, cho phép bộ xử lý hỗ trợ không gian địa chỉ vật lý lớn hơn 4GB. Mặc dù các chương trình vẫn có không gian địa chỉ tuyến tính lên đến 4GB, nhưng đơn vị quản lý bộ nhớ ánh xạ các địa chỉ đó vào không gian địa chỉ vật lý mở rộng 64GB. Trên các hệ thống đã kích hoạt PAE, địa chỉ tuyến tính được chia thành bốn chỉ mục:

- Bảng chỉ mục bộ định tuyến trang (PDPT)
- Bảng chỉ mục bộ định tuyến (PD)
- Bảng chỉ mục trang (PT)
- Độ lệch trang

Figure 1-5 thể hiện một ví dụ về việc dịch địa chỉ thành trang 4KB bằng cách sử dụng phân trang 32-bit PAE. Những điểm khác biệt chính là sự giới thiệu thêm một cấp độ trong cấu trúc phân trang gọi là bảng chỉ mục bộ định tuyến trang và các mục cấu trúc phân trang bây giờ có kích thước 64 bit. Với những thay đổi này, thanh ghi CR3 bây giờ chứa địa chỉ vật lý của bảng chỉ mục bộ định tuyến trang.

Figure 1-6 thể hiện các định dạng của các địa chỉ cấu trúc phân trang được sử dụng trong phân trang 32-bit PAE. Khi PAE được kích hoạt, bảng phân trang đầu tiên chỉ có 4 (2^2) mục. Các bit 31:30 từ địa chỉ tuyến tính chọn mục bảng chỉ mục bộ định tuyến trang (PDPTE). Các bit 29:21 là một chỉ mục để chọn từ 512 (2^9) PDEs. Nếu cờ PS được thiết lập, PDE ánh xạ một trang 2MB. Ngược lại, 9 bit được trích xuất từ các bit 20:12 được chọn từ 512 (2^9) PTEs. Giả sử tất cả các mục đều hợp lệ và địa chỉ đang ánh xạ một trang 4KB, 12 bit cuối cùng của địa chỉ tuyến tính chỉ định độ lệch trong trang cho địa chỉ vật lý tương ứng.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-21%20220113.png)

### Intel 64 Architecture (Kiến trúc Intel 64)

Kiến trúc Intel 64 tương tự như IA-32 trong môi trường thực thi, nhưng có một số khác biệt. Các thanh ghi được tô đậm trong kiến trúc IA-32 vẫn tồn tại trong Intel 64, nhưng đã được mở rộng để chứa 64 bit. Thay đổi quan trọng nhất là Intel 64 giờ có thể hỗ trợ địa chỉ tuyến tính 64 bit. Kết quả là, kiến trúc Intel 64 hỗ trợ không gian địa chỉ tuyến tính lên tới 2^64 byte. Lưu ý rằng các bản hiện thực hiện tại của kiến trúc (vào thời điểm viết bài này) không hỗ trợ toàn bộ 64 bit, chỉ hỗ trợ 48 bit địa chỉ tuyến tính. Do đó, các địa chỉ ảo trên các hệ thống này đều ở định dạng canon (canonical format).

Điều này có nghĩa là bit 63:48 được thiết lập thành tất cả là 1 hoặc tất cả là 0, phụ thuộc vào trạng thái của bit 47. Ví dụ, địa chỉ 0xfffffa800ccc0b30 có các bit 63:48 được thiết lập vì bit 47 được thiết lập (đây cũng được gọi là mở rộng dấu chữ).

Ngoài ra, điều quan trọng là bạn tập trung vào các thay đổi trong quản lý bộ nhớ vì chúng có tác động trực tiếp đến phân tích bộ nhớ. Khác biệt quan trọng nhất là kiến trúc Intel 64 bây giờ hỗ trợ thêm một cấp độ bảng chỉ mục phân trang được gọi là page map level 4 (PML4). Tất cả các mục trong cấu trúc phân trang đều là 64 bit, và chúng có thể ánh xạ địa chỉ ảo vào các trang có kích thước 4KB, 2MB hoặc 1GB. Figure 1-7 thể hiện một ví dụ về việc dịch địa chỉ thành trang 4KB bằng cách sử dụng phân trang 64-bit/IA-32e.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-21%20221352.png)

Figure 1-8 thể hiện các định dạng địa chỉ cấu trúc trang được sử dụng trong 64-bit/IA-32e paging. Mỗi cấu trúc trang bao gồm 512 mục nhập (2^9) và được chỉ mục bởi các giá trị trích xuất từ các phạm vi sau trong địa chỉ ảo 48 bit:

• Bit 47:39 (PML4E offset): Được sử dụng để chọn mục nhập trong bảng PML4 (Page Map Level 4).
• Bit 38-30 (PDPTE offset): Được sử dụng để chọn mục nhập trong bảng PDPT (Page Directory Pointer Table).
• Bit 29:21 (PDE offset): Được sử dụng để chọn mục nhập trong bảng PD (Page Directory).
• Bit 20:12 (PTE offset): Được sử dụng để chọn mục nhập trong bảng PT (Page Table).

Nếu cờ PS được đặt trong PDPTE, mục nhập ánh xạ một trang 1GB nếu nó được hỗ trợ. Tương tự, nếu cờ PS được đặt trong PDE, PDE ánh xạ một trang 2MB. Giả sử tất cả các mục nhập trung gian đều tồn tại, thì 12 bit cuối cùng của địa chỉ ảo sẽ xác định vị trí byte trong trang vật lý.

Quá trình dịch địa chỉ sẽ đi qua từng cấp bậc của cấu trúc trang, sử dụng các chỉ mục trích xuất để điều hướng qua các bậc thang đến khi nó đến trang vật lý tương ứng với địa chỉ ảo.

Nếu bạn quan tâm đến chi tiết về cách các cờ trong các mục nhập cấu trúc trang khác nhau ảnh hưởng đến pháp y trong bộ nhớ, bạn nên xem tài liệu của Intel và không gian địa chỉ AMD64PagedMemory của Volatility.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-21%20221607.png)

#### Interrupt Descriptor Table (Bảng mô tả ngắt)

Các kiến trúc máy tính thường cung cấp một cơ chế để ngắt thực thi quá trình và chuyển quyền kiểm soát cho một quy trình phần mềm ở chế độ đặc quyền (privileged mode). Trong các kiến trúc IA-32 và Intel 64, các quy trình này được lưu trữ trong Bảng Mô tả Ngắt (Interrupt Descriptor Table - IDT). Mỗi bộ xử lý có một IDT riêng gồm 256 mục nhập có độ dài 8 byte hoặc 16 byte, trong đó 32 mục nhập đầu tiên được dành riêng cho các ngoại lệ và ngắt được định nghĩa bởi bộ xử lý. Mỗi mục nhập chứa địa chỉ của quy trình phục vụ ngắt (ISR) có thể xử lý ngắt hoặc ngoại lệ cụ thể. Trong trường hợp xảy ra ngắt hoặc ngoại lệ, số ngắt được chỉ định sẽ được sử dụng làm chỉ mục vào IDT (kết hợp gián tiếp một đoạn trong GDT), và CPU sẽ gọi tới bộ xử lý tương ứng.

Sau hầu hết các ngắt, hệ điều hành sẽ tiếp tục thực thi từ nơi nó bị ngắt. Ví dụ, nếu một luồng cố gắng truy cập một trang bộ nhớ không hợp lệ, nó sẽ tạo ra một ngoại lệ gọi là lỗi trang. Số ngoại lệ 0xE xử lý lỗi trang trên các kiến trúc x86 và Intel 64. Do đó, mục nhập IDT cho 0xE chứa con trỏ hàm cho bộ xử lý lỗi trang của hệ điều hành. Sau khi bộ xử lý lỗi trang thực thi, quyền kiểm soát có thể trở lại luồng đã cố gắng truy cập trang bộ nhớ. Hệ điều hành cũng sử dụng IDT để lưu trữ các bộ xử lý cho nhiều sự kiện khác nhau, bao gồm các cuộc gọi hệ thống, các điểm dừng gỡ lỗi và các ngoại lệ khác.

>**CẢNH BÁO**
	Vì vai trò quan trọng mà IDT đóng đối với các hệ điều hành, nó thường là mục tiêu của phần mềm độc hại. Phần mềm độc hại có thể cố gắng chuyển hướng các mục nhập, sửa mã bộ xử lý, thêm các mục nhập mới hoặc thậm chí tạo hoàn toàn các bảng ngắt mới. Ví dụ, Shadow Walker (https://www.blackhat.com/presentations/bh-jp-05/ bh-jp-05-sparks-butler.pdf) đã nắm bắt bộ xử lý lỗi trang bằng cách sửa đổi IDT và đã có thể trả lại các trang "giả mạo" cho người gọi.
	Một bài báo thú vị liên quan đến việc sử dụng IDT cho mục đích rootkit và chống pháp lý là Stealth Hooking: Another Way to Subvert the Windows Kernel (http://phrack.org/ issues/65/4.html). Bạn có thể sử dụng các plugin của Volatility như idt (dành cho Windows) và linux_idt (dành cho Linux) để kiểm tra IDT.

## Hệ điều hành

Phần này cung cấp một cái nhìn tổng quan về các khía cạnh của hệ điều hành hiện đại có tác động đến phân tích bộ nhớ. Đặc biệt, nó tập trung vào các tính năng quan trọng chung cho ba hệ điều hành được thảo luận trong cuốn sách này: Microsoft Windows, Linux và Mac OS X. Mặc dù những chủ đề có thể quen thuộc, phần này thảo luận về chúng trong bối cảnh phân tích bộ nhớ. Các nhà điều tra quen thuộc với cấu trúc bên trong của hệ điều hành có thể chọn bỏ qua hầu hết các thông tin trong phần này hoặc sử dụng nó như một tài liệu tham khảo cho các chủ đề được đề cập trong các chương sau.

### Tách biệt đặc quyền

Để ngăn chặn các ứng dụng người dùng có thể bị hỏng hoặc độc hại truy cập hoặc thay đổi các thành phần quan trọng của hệ điều hành, hầu hết các hệ điều hành hiện đại thực thi một hình thức cách ly đặc quyền giữa người dùng và kernel. Cách ly này cố gắng ngăn ứng dụng ảnh hưởng đến sự ổn định của hệ điều hành hoặc các quy trình khác. Mã lệnh liên quan đến các ứng dụng người dùng (không tin tưởng) thực thi trong chế độ người dùng và mã lệnh liên quan đến hệ điều hành (đáng tin cậy) thực thi trong chế độ kernel.

Sự tách biệt này được áp dụng bởi kiến ​​trúc bộ vi xử lý IA-32 thông qua việc sử dụng bốn cấp độ đặc quyền thường được gọi là các vòng bảo vệ (protection rings). Trong hầu hết các hệ điều hành, chế độ kernel được thực hiện trong vòng 0 (cao nhất đặc quyền) và chế độ người dùng trong vòng 3 (thấp nhất đặc quyền). Khi bộ vi xử lý đang thực thi trong chế độ kernel, mã lệnh có quyền truy cập không giới hạn vào phần cứng cơ bản, bao gồm các chỉ thị có đặc quyền và các khu vực bộ nhớ của kernel và quy trình (trừ trên các hệ thống mới với SMEP, ngăn cản thực thi vòng 0 của các trang người dùng). Để ứng dụng người dùng có thể truy cập các thành phần quan trọng của hệ điều hành, ứng dụng phải chuyển từ chế độ người dùng sang chế độ kernel bằng cách sử dụng một tập hợp xác định trước định rõ các cuộc gọi hệ thống. Hiểu mức độ truy cập mà mã độc hại đã đạt được có thể giúp cung cấp thông tin có giá trị về loại thay đổi nó có thể thực hiện trên hệ thống.

### (System Calls (Cuộc gọi hệ thống)

Hệ điều hành được thiết kế để cung cấp dịch vụ cho các ứng dụng người dùng. Một ứng dụng người dùng yêu cầu một dịch vụ từ kernel của hệ điều hành bằng cách sử dụng một cuộc gọi hệ thống. Ví dụ, khi một ứng dụng cần tương tác với một tập tin, giao tiếp qua mạng, hoặc khởi chạy một quá trình khác, thì yêu cầu cuộc gọi hệ thống. Như kết quả, cuộc gọi hệ thống xác định giao diện lập trình ứng dụng cấp thấp giữa các ứng dụng người dùng và kernel của hệ điều hành. Lưu ý rằng hầu hết các ứng dụng không được triển khai trực tiếp dưới dạng cuộc gọi hệ thống. Thay vào đó, hầu hết các hệ điều hành định nghĩa một tập hợp các API ổn định tương ứng với một hoặc nhiều cuộc gọi hệ thống (ví dụ, các API được cung cấp bởi ntdll.dll và kernel32.dll trên Windows).

Trước khi một ứng dụng người dùng thực hiện một cuộc gọi hệ thống, nó phải cấu hình môi trường thực thi để chuyển đối số cho kernel thông qua một thỏa thuận đã được quy định trước (ví dụ, trên ngăn xếp hoặc trong các thanh ghi cụ thể). Để gọi cuộc gọi hệ thống, ứng dụng thực hiện một ngắt phần mềm hoặc một lệnh cụ thể cho kiến ​​trúc, điều này lưu trữ bối cảnh đăng ký chế độ người dùng, thay đổi chế độ thực thi sang kernel, khởi tạo ngăn xếp kernel và gọi bộ xử lý cuộc gọi hệ thống. Sau khi yêu cầu được phục vụ, thực thi trở lại chế độ người dùng và bối cảnh đăng ký không đặc quyền được khôi phục. Kiểm soát sau đó trở lại lệnh sau cuộc gọi hệ thống.

Bởi vì đó là một cầu nối quan trọng giữa các ứng dụng người dùng và hệ điều hành, mã được sử dụng để phục vụ ngắt cuộc gọi hệ thống thường bị chặn bởi các sản phẩm bảo mật và bị nhắm mục tiêu bởi mã độc hại. Sau này trong cuốn sách, bạn sẽ tìm hiểu cách sử dụng phân tích bộ nhớ để phát hiện các sửa đổi được thực hiện trên giao diện quan trọng này trên các hệ thống Windows, Linux và Mac.

## Process Management (Quản lý quy trình)

Quy trình là một phiên bản của một chương trình đang chạy trong bộ nhớ. Hệ điều hành có trách nhiệm quản lý việc tạo, tạm dừng và kết thúc quy trình. Hầu hết các hệ điều hành hiện đại đều có tính năng gọi là đa chương trình, cho phép nhiều quy trình xuất hiện để thực hiện đồng thời. Khi một chương trình thực thi, một quy trình mới được tạo và liên kết với một tập hợp các thuộc tính riêng, bao gồm một ID quy trình duy nhất và không gian địa chỉ riêng. Không gian địa chỉ của quy trình trở thành một container cho mã ứng dụng, các thư viện chia sẻ, dữ liệu động và ngăn xếp thời gian thực. Một quy trình cũng có ít nhất một luồng thực thi duy nhất. Quy trình cung cấp môi trường thực thi, tài nguyên và ngữ cảnh để các luồng chạy. Một khía cạnh quan trọng của phân tích bộ nhớ liên quan đến liệt kê các quy trình đã thực thi trên một hệ thống và phân tích dữ liệu được lưu trữ trong không gian địa chỉ của chúng, bao gồm mật khẩu, URL, khóa mã hóa, email và lịch sử trò chuyện.

### Threads (Luồng)

Luồng là đơn vị cơ bản sử dụng và thực thi CPU. Một luồng thường được đặc trưng bởi một ID luồng, tập hợp thanh ghi CPU và các ngăn xếp thực thi, giúp xác định ngữ cảnh thực thi của một luồng. Mặc dù có ngữ cảnh thực thi riêng biệt, các luồng của một quy trình chia sẻ cùng mã, dữ liệu, không gian địa chỉ và tài nguyên hệ điều hành. Một quy trình có nhiều luồng có thể xuất hiện như đang thực hiện đồng thời nhiều nhiệm vụ. Ví dụ, một luồng có thể giao tiếp qua mạng trong khi luồng khác hiển thị dữ liệu trên màn hình. Liên quan đến phân tích bộ nhớ, các cấu trúc dữ liệu luồng rất hữu ích vì thường chứa thông tin về thời gian dấu thời và địa chỉ bắt đầu. Thông tin này có thể giúp bạn xác định mã nào trong một quy trình đã được thực thi và thời gian nó bắt đầu.

### CPU Scheduling (Lập lịch CPU)

Khả năng của hệ điều hành phân phối thời gian thực thi của CPU cho nhiều luồng được gọi là lập lịch CPU. Một mục tiêu của lập lịch là tối ưu hóa việc sử dụng CPU khi các luồng chuyển đổi giữa việc chờ các hoạt động I/O và thực hiện tính toán yêu cầu nhiều CPU. Lập lịch CPU của hệ điều hành thực hiện các chính sách điều hành quyết định các luồng nào được thực thi và trong khoảng thời gian bao lâu.

Chuyển đổi thực thi từ một luồng sang luồng khác được gọi là chuyển ngữ cảnh. Một ngữ cảnh thực thi bao gồm các giá trị của các thanh ghi CPU, bao gồm cả con trỏ chỉ thị hiện tại. Trong quá trình chuyển ngữ cảnh, hệ điều hành tạm dừng thực thi của một luồng và lưu trữ ngữ cảnh thực thi của nó trong bộ nhớ chính. Hệ điều hành sau đó lấy ngữ cảnh thực thi của một luồng khác từ bộ nhớ, cập nhật trạng thái của các thanh ghi CPU và tiếp tục thực thi từ nơi đã tạm dừng trước đó. Ngữ cảnh thực thi đã lưu trữ liên quan đến các luồng bị tạm dừng có thể cung cấp thông tin quý giá trong quá trình phân tích bộ nhớ. Ví dụ, nó có thể cung cấp thông tin về các phần mã được thực thi hoặc thông số đã truyền vào các lời gọi hệ thống.

### System Resources (Tài nguyên Hệ thống)

Một dịch vụ quan trọng khác mà hệ điều hành cung cấp là giúp quản lý các tài nguyên của quy trình. Như đã đề cập trước đó, một quy trình hoạt động như một container cho các tài nguyên hệ thống có thể truy cập được bởi các luồng của nó. Hầu hết các hệ điều hành hiện đại duy trì các cấu trúc dữ liệu để quản lý các tài nguyên đang được truy cập, quy trình nào có thể truy cập chúng và cách chúng được truy cập. Một số ví dụ về các tài nguyên hệ thống thường được theo dõi bao gồm quy trình, luồng, tập tin, socket mạng, đối tượng đồng bộ hóa và vùng nhớ chia sẻ.

Loại tài nguyên đang được quản lý và cấu trúc dữ liệu được sử dụng để theo dõi chúng thường khác nhau giữa các hệ điều hành. Ví dụ, Windows sử dụng một bộ quản lý đối tượng để giám sát việc sử dụng các tài nguyên hệ thống và sau đó lưu trữ thông tin đó trong một bảng handle. Một handle cung cấp quy trình với một định danh duy nhất để truy cập và điều khiển các tài nguyên hệ thống. Nó cũng được sử dụng để thiết lập kiểm soát truy cập vào các tài nguyên đó và theo dõi việc sử dụng chúng. Linux và Mac cũng sử dụng các bộ xử lý tệp tương tự. Trong các phần tiếp theo của cuốn sách, chúng ta sẽ mô tả cách trích xuất thông tin này từ các bảng handle hoặc bộ xử lý tệp và cách sử dụng nó để có cái nhìn sâu hơn về hoạt động của quy trình đó.

## Memory Management (Quản lý Bộ nhớ)

Quản lý bộ nhớ đề cập đến các thuật toán của hệ điều hành để quản lý việc phân bổ, giải phóng và tổ chức bộ nhớ vật lý. Những thuật toán này thường phụ thuộc vào hỗ trợ phần cứng đã được thảo luận trước đó.

### Virtual Memory (Bộ nhớ ảo)

Hệ điều hành cung cấp cho mỗi quy trình một không gian địa chỉ ảo riêng tư. Trừu tượng này tạo ra một phân tách giữa bộ nhớ logic mà một quy trình thấy và bộ nhớ vật lý thực tế được cài đặt trên máy tính. Kết quả là bạn có thể viết các chương trình như thể chúng có quyền truy cập vào toàn bộ không gian địa chỉ và trong đó tất cả các phạm vi đều được tồn tại trong bộ nhớ. Trên thực tế, một số trang của không gian địa chỉ có thể không tồn tại. Đằng sau hậu trường, bộ quản lý bộ nhớ chịu trách nhiệm chuyển các vùng bộ nhớ vào lưu trữ phụ để giải phóng không gian trong bộ nhớ vật lý. Trong quá trình thực thi, bộ quản lý bộ nhớ và MMU (đơn vị quản lý bộ nhớ) hoạt động cùng nhau để dịch các địa chỉ ảo thành địa chỉ vật lý. Nếu một luồng truy cập vào một địa chỉ ảo đã được chuyển đến lưu trữ phụ, dữ liệu đó sẽ được đưa trở lại bộ nhớ vật lý (thông thường thông qua lỗi trang). Tương tác này được thể hiện trong Figure 1-9.

Kích thước thực tế của không gian địa chỉ ảo thường phụ thuộc vào các đặc điểm của phần cứng và hệ điều hành. Hệ điều hành thường chia phạm vi các địa chỉ truy cập thành các địa chỉ liên quan đến hệ điều hành và các địa chỉ riêng tư của quy trình. Phạm vi địa chỉ dành cho hệ điều hành thường là nhất quán trên tất cả các quy trình, trong khi các phạm vi riêng tư phụ thuộc vào quy trình đang thực thi. Với sự hỗ trợ của phần cứng, bộ quản lý bộ nhớ có thể chia dữ liệu để ngăn một quy trình độc hại hoặc không đúng đắn đọc hoặc ghi vào bộ nhớ thuộc về bộ nhớ kernel hoặc các quy trình khác.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20104129.png)

### Demand Paging (Yêu cầu Phân Trang)

Cơ chế thường được sử dụng để triển khai bộ nhớ ảo là yêu cầu phân trang, đây là một chính sách quản lý bộ nhớ để xác định các vùng nào đang nằm trong bộ nhớ chính và các vùng nào sẽ được chuyển đến lưu trữ phụ chậm hơn khi cần thiết. Lưu trữ phụ phổ biến nhất là một tập tin hoặc phân vùng trên đĩa cứng nội bộ, được gọi là tệp trang hoặc swap, tương ứng. Việc triển khai yêu cầu phân trang cố gắng chỉ tải các trang cần thiết vào bộ nhớ thay vì toàn bộ quy trình.

Yêu cầu phân trang dựa vào đặc điểm sử dụng bộ nhớ được biết đến với tên gọi gần như của tài liệu tham chiếu, dựa vào quan sát rằng các vị trí bộ nhớ có thể được truy cập thường xuyên trong một khoảng thời gian ngắn, cũng như các vị trí lân cận của chúng. Lý tưởng nhất, yêu cầu phân trang giảm thời gian cần để tải một quy trình vào bộ nhớ và tăng số lượng quy trình đang tồn tại trong bộ nhớ tại bất kỳ thời điểm nào. Để cải thiện hiệu suất và ổn định, bộ quản lý bộ nhớ của hệ điều hành thường có cơ chế để chỉ định các vùng của bộ nhớ được phân trang so với những vùng phải được giữ lại.

Bộ quản lý bộ nhớ thường theo dõi các trang nào đang nằm trong bộ nhớ chính và các trang nào không nằm trong các cấu trúc dữ liệu phân trang đã được thảo luận trước đó. Nếu một luồng cố gắng truy cập vào một trang không nằm trong bộ nhớ chính, phần cứng sẽ tạo ra một lỗi trang. Trong khi phần cứng tạo ra lỗi trang, hệ điều hành sử dụng thông tin trạng thái được mã hóa trong các cấu trúc phân trang để xác định cách xử lý lỗi. Ví dụ, trang có thể liên quan đến một vùng của một tập tin chưa được tải vào bộ nhớ, hoặc trang có thể đã được chuyển đến tệp trang.

Yêu cầu phân trang cung cấp những lợi ích đáng kể cho hệ điều hành và là trong suốt với các ứng dụng đang chạy. Như bạn sẽ thấy trong các chương sau, nó thêm một số phức tạp vào phân tích bộ nhớ vì một số trang có thể không nằm trong bộ nhớ chính vào thời điểm thu thập mẫu bộ nhớ. Trong một số trường hợp, có thể kết hợp dữ liệu không nằm trong bộ nhớ trên đĩa với dữ liệu được lưu trữ trong bộ nhớ để cung cấp cái nhìn hoàn chỉnh hơn về bộ nhớ ảo.

### Shared Memory (Bộ nhớ chia sẻ)

Các phần trước đã thảo luận về cách không gian địa chỉ quy trình được cô lập khỏi nhau để cải thiện tính bảo mật và ổn định của hệ thống. Tuy nhiên, hệ điều hành hiện đại cũng cung cấp các cơ chế cho phép quy trình chia sẻ bộ nhớ. Bạn có thể xem bộ nhớ chia sẻ như bộ nhớ có thể truy cập từ nhiều không gian địa chỉ ảo. Ví dụ, Figure 1-10 cho thấy Quy trình A và Quy trình B có các khu vực của không gian địa chỉ ảo riêng của họ mà ánh xạ vào các trang chung trong bộ nhớ vật lý. Một trong những cách thông thường để sử dụng bộ nhớ chia sẻ là cung cấp một phương tiện hiệu quả để giao tiếp giữa các quy trình. Sau khi một vùng chia sẻ được ánh xạ vào không gian địa chỉ ảo, các quy trình có thể sử dụng vùng đó để trao đổi thông điệp và dữ liệu.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-22%20154740.png)

Bộ nhớ chia sẻ cũng thường được sử dụng để tiết kiệm bộ nhớ vật lý. Thay vì cấp phát nhiều trang vật lý chứa cùng dữ liệu, bạn có thể tạo một phiên bản duy nhất của dữ liệu trong bộ nhớ vật lý và ánh xạ các vùng khác nhau của bộ nhớ ảo vào nó. Ví dụ bao gồm các thư viện chia sẻ hoặc động chứa mã và dữ liệu chung. Trong những trường hợp này, các trang chia sẻ thường được ánh xạ với chế độ copy-on-write, cho phép trình quản lý bộ nhớ trì hoãn việc tạo bản sao riêng tư của dữ liệu trong không gian địa chỉ của quy trình cho đến khi bộ nhớ đã được sửa đổi. Sau khi trang đã được ghi, trình quản lý bộ nhớ cấp phát một bản sao riêng tư của trang đó với các sửa đổi tương ứng và cập nhật ánh xạ bộ nhớ ảo cho quy trình đó. Các quy trình khác không bị ảnh hưởng và vẫn ánh xạ vào trang chia sẻ ban đầu.

Cả bộ nhớ chia sẻ và ánh xạ copy-on-write thường gặp trong quá trình phân tích bộ nhớ vì phần mềm độc hại thường cố gắng thay đổi mã của các thư viện chia sẻ để chiếm quyền điều khiển luồng thực thi. Trong Chương 17, bạn sẽ thấy một ví dụ về cách nhận ra sự không nhất quán bằng cách so sánh dữ liệu được chia sẻ giữa nhiều quy trình.

### Stacks và Heaps

Không gian địa chỉ của người dùng thường được chia thành nhiều vùng. Vùng ngăn xếp (stack) chứa dữ liệu tạm thời liên quan đến việc thực thi các hàm. Dữ liệu trong vùng này được lưu trữ trong một cấu trúc dữ liệu gọi là khung ngăn xếp (stack frame). Mỗi khung bao gồm thông tin như các tham số hàm, biến cục bộ và thông tin cần thiết để phục hồi khung ngăn xếp trước đó. Khi một luồng đang thực thi, khung ngăn xếp được lưu trữ (đẩy vào - pushed) khi gọi một hàm và bị gỡ bỏ (rút ra - popped) khi trở về từ hàm đó. Vì một tiến trình có thể thực thi ở chế độ kernel hoặc chế độ người dùng, hệ điều hành thường sử dụng một ngăn xếp riêng biệt cho các hàm thực thi trong mỗi chế độ.

Phân tích các khung ngăn xếp còn sót lại và đang hoạt động rất hữu ích trong phân tích bộ nhớ vì chúng cung cấp thông tin quý giá về mã nào đang được thực thi và dữ liệu nào đang được xử lý. Ví dụ, khóa có thể được truyền vào các hàm mã hóa, dữ liệu ăn cắp từ máy tính (phím ấn, nội dung tệp) có thể được gửi đến các hàm để tiết lộ thông tin, và còn nhiều khả năng khác. Trong phân tích phần mềm độc hại, khung ngăn xếp có thể được sử dụng để suy ra phần nào của phần mềm độc hại đang hoạt động và các phần của hệ thống mà phần mềm độc hại đang tương tác.

>**Lưu ý:** Carl Pulley đã viết một plugin unwind stack cho Volatility có tên là "exportstack" (https://github.com/carlpulley/volatility). Nó tích hợp với các ký hiệu gỡ lỗi của Microsoft để đặt tên chính xác các địa chỉ và liên kết chúng với tên hàm API.
Edwin Smulders cũng đã viết một plugin tương tự có tên "linux_process_stack" (https://github.com/Dutchy-/volatility-plugins) để phân tích các ngăn xếp trong bộ nhớ Linux.

Dữ liệu của ứng dụng cần được cấp phát động thường được lưu trữ trong vùng gọi là heap. Khác với dữ liệu được cấp phát trên ngăn xếp, chỉ tồn tại trong phạm vi của một hàm, dữ liệu được cấp phát trong heap có thể tồn tại trong suốt thời gian sống của tiến trình. Heap lưu trữ thông tin có chiều dài và nội dung có thể không biết trước khi biên dịch. Ứng dụng có thể cấp phát vùng nhớ trên heap khi cần và sau đó giải phóng chúng sau khi sử dụng.

Hệ điều hành cũng có thể có các vùng nhớ được cấp phát động trong chế độ kernel. Ví dụ, Windows tạo các vùng paged và nonpaged trong kernel được gọi là pools. Các ví dụ thông thường về dữ liệu thú vị mà bạn có thể tìm thấy trong heap bao gồm dữ liệu đọc từ tệp trên đĩa, dữ liệu truyền qua mạng và đầu vào được gõ từ bàn phím. Do tính chất của dữ liệu được lưu trữ trong heap, heap có thể cung cấp bằng chứng quý giá trong quá trình điều tra pháp lý. Vì dữ liệu có thể phụ thuộc vào ứng dụng, có thể yêu cầu phân tích thủ công, như xem dữ liệu với trình biên tập hex hoặc trích xuất chuỗi để kiểm tra thêm.

## File System (Hệ thống tệp tin)

Trước đó, chúng ta đã thảo luận về cách hệ thống quản lý bộ nhớ tận dụng lưu trữ phụ để giải phóng bộ nhớ chính. Hệ điều hành cũng sử dụng lưu trữ phụ để quản lý các đối tượng dữ liệu bền vững mà người dùng muốn truy cập trong một khoảng thời gian dài hơn tuổi thọ của bất kỳ tiến trình cụ thể nào. Không giống như bộ nhớ chính tạm thời, lưu trữ phụ thường được tạo thành từ các thiết bị khối bền vững như ổ cứng. Bộ các cấu trúc dữ liệu cho phép ứng dụng thực hiện các hoạt động cơ bản trên dữ liệu được lưu trữ được gọi là hệ thống tập tin. Phân tích phục hồi hệ thống tập tin liên quan đến tìm kiếm các tập tin hoặc nội dung quan trọng, phục hồi các tàn tích tập tin (đã xóa, mảnh vụn, ẩn) và sử dụng siêu dữ liệu thời gian như dấu thời gian để xây dựng lại các sự kiện trong một sự cố.

Mặc dù hệ thống tập tin trước đây đã là một trong những nguồn bằng chứng kỹ thuật số phổ biến nhất, nhưng phân tích pháp y tế của hệ thống tập tin không phải là trọng tâm của cuốn sách này. Cuốn sách này thảo luận về những đối tượng hệ thống tập tin mà bạn tìm thấy trong bộ nhớ bay hơi, những tàn tích trong bộ nhớ chính mà bạn tìm thấy trong hệ thống tập tin và cách bạn có thể kết hợp những loại dữ liệu này để cung cấp cái nhìn tổng thể hơn về trạng thái của một hệ thống. Ví dụ, dữ liệu được lưu trữ trong các tập tin và các cấu trúc thư mục phải được tải vào bộ nhớ khi cần thiết. Hệ điều hành cũng lưu trữ dữ liệu được truy cập thường xuyên trong bộ nhớ chính để giảm thiểu chi phí liên quan đến truy vấn liên tục của lưu trữ phụ chậm hơn.

Các phần trước đây đã thảo luận về cách hệ thống quản lý bộ nhớ sử dụng demand paging (yêu cầu trang) và shared memory (bộ nhớ chia sẻ) để tối ưu hóa việc sử dụng bộ nhớ. Hầu hết các hệ điều hành hiện đại cũng hỗ trợ memory mapped files (bản đồ bộ nhớ từng tập tin), cho phép tập tin hoặc phần của tập tin được ánh xạ vào không gian địa chỉ ảo. Sau khi tập tin được ánh xạ vào bộ nhớ, bạn có thể truy cập và sửa đổi chúng cũng giống như các cấu trúc dữ liệu trong bộ nhớ truyền thống như mảng. Kết quả là, các chức năng tối ưu hóa của hệ điều hành được sử dụng để xử lý I/O đĩa một cách trong suốt, trong đó tập tin trở thành bộ nhớ hậu cần. Các trang dữ liệu tập tin được đọc vào bộ nhớ khi các địa chỉ trong trang được truy cập và các vùng dữ liệu tập tin có thể dễ dàng được chia sẻ giữa các tiến trình.

Nhà điều tra có thể sử dụng thông tin về dữ liệu tập tin được lưu trữ trong bộ nhớ đệm để hỗ trợ triển khai và cung cấp bối cảnh về dữ liệu được truy cập gần đây và thường xuyên được truy cập. Các đặc điểm cũng có thể cung cấp thông tin về người dùng hoặc tiến trình nào đã truy cập vào dữ liệu. Bằng cách so sánh dữ liệu trong bộ nhớ đệm với dữ liệu được lưu trữ trên đĩa, các nhà điều tra cũng có thể xác định các thay đổi được thực hiện trên dữ liệu đang trú trong bộ nhớ. Ngoài ra, trong quá trình phân tích hệ thống tập tin, các nhà điều tra có thể tìm thấy các tàn tích bộ nhớ trong các tệp lưu trữ sự cố hoặc các tệp đánh dấu tiết kiệm, điều này có thể cung cấp thông tin về trạng thái trước đó của hệ thống. Do đó, mặc dù cuốn sách này không bao gồm phân tích pháp y tế hệ thống tập tin tổng quát, việc quen thuộc với hệ thống tập tin là hữu ích.

## I/O Sybsystem (Hệ thống vào/ra)

Một trong những dịch vụ chính mà hệ điều hành cung cấp là quản lý và cung cấp giao diện cho các thiết bị đầu vào và đầu ra (I/O). Hệ thống I/O trừu tượng hóa các chi tiết về các thiết bị này và cho phép tiến trình giao tiếp với chúng bằng cách sử dụng một tập hợp tiêu chuẩn các thủ tục. Nhiều hệ điều hành tổng quát hóa giao diện với các thiết bị bằng cách xem xét chúng như các tập tin. Bởi vì bạn không thể dự đoán loại thiết bị mà người dùng sẽ kết nối với hệ thống, hệ điều hành sử dụng các mô-đun nhân gọi là trình điều khiển thiết bị như một cơ chế để mở rộng khả năng của nhân để hỗ trợ các thiết bị mới.

### Device Drivers (Trình điều khiển thiết bị)

Trình điều khiển thiết bị trừu tượng hóa các chi tiết về cách thiết bị kiểm soát và chuyển dữ liệu. Trình điều khiển thiết bị thường giao tiếp với các thanh ghi của bộ điều khiển thiết bị. Mặc dù một số kiến trúc CPU cung cấp một không gian địa chỉ riêng cho các thiết bị I/O và sau đó yêu cầu các chỉ thị I/O có đặc quyền, các kiến trúc khác ánh xạ bộ nhớ và thanh ghi của các thiết bị I/O vào không gian địa chỉ ảo. Điều này thường được gọi là I/O được ánh xạ vào bộ nhớ. Như bạn sẽ thấy trong các chương sau, phần mềm thường lạm dụng trình điều khiển thiết bị để sửa đổi trạng thái của hệ thống.

Hệ điều hành cũng sử dụng các trình điều khiển thiết bị để thực hiện các thiết bị ảo, chỉ tồn tại trong phần mềm. Ví dụ, một số hệ điều hành cung cấp một biểu diễn của bộ nhớ vật lý thông qua một thiết bị phần mềm (ví dụ: \Device\PhysicalMemory trên Windows). Giao diện thiết bị này thường được sử dụng phổ biến để thu thập mẫu pháp y của bộ nhớ vật lý. Lưu ý rằng bộ nhớ và thanh ghi của các thiết bị cũng có thể được ánh xạ vào bộ nhớ, điều này có thể có hậu quả thú vị cho việc tạo mẫu bộ nhớ (xem Chương 4).

### I/O Controls (IOCTLs)

I/O Control (IOCTL) commands, hay còn gọi là "Lệnh Điều Khiển Đầu Vào/ra," là một cơ chế thông thường khác để truyền thông giữa chế độ người dùng và chế độ kernel. Mặc dù các lệnh hệ thống (system calls) cung cấp một giao diện tiện lợi để truy cập các dịch vụ cố định do hệ điều hành cung cấp, nhưng các ứng dụng người dùng có thể cần truyền thông với nhiều thiết bị ngoại vi hoặc các thành phần khác của hệ điều hành. IOCTLs cho phép một ứng dụng người dùng truyền thông với một trình điều khiển thiết bị ở chế độ kernel. Nó cũng cung cấp cơ chế cho các thiết bị phần cứng và trình điều khiển của bên thứ ba để định nghĩa các giao diện và chức năng riêng của họ.

Tương tự như các lệnh hệ thống, phần mã độc tấn công ở cấp độ kernel có thể lừa đảo các chức năng IOCTL để lọc kết quả hoặc thay đổi luồng điều khiển. Phần mã độc cũng có thể sử dụng các xử lý IOCTL để giao tiếp giữa các thành phần ở chế độ người dùng và chế độ kernel (ví dụ, yêu cầu một thành phần kernel nâng cao đặc quyền, tắt một dịch vụ hoặc thay đổi cài đặt tường lửa). Kỹ thuật phân tích bộ nhớ có thể phát hiện các IOCTL đã bị sửa đổi hoặc không rõ nguồn gốc và cung cấp thông tin quan trọng về cách các kẻ tấn công sử dụng chúng.

## Tổng quan

Bây giờ bạn đã quen thuộc với các khái niệm cơ bản về phần cứng và phần mềm trong môi trường số, bạn có thể tiếp tục khám phá các loại dữ liệu mà bạn sẽ gặp trong quá trình phân tích forensics trên phương tiện số, chẳng hạn như RAM. Hãy nhớ rằng thông tin chia sẻ trong chương này không đặc thù cho bất kỳ hệ điều hành cụ thể nào - nó áp dụng cho Windows, Linux và Mac OS X. Do đó, khi bạn đọc tiếp phần còn lại của cuốn sách, bạn có thể cần tham khảo lại chương này để làm mới kiến thức về các từ khóa mà bạn thấy chúng tôi sử dụng trong các cuộc thảo luận khác.
