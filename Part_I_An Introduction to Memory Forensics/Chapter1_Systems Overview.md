Chương này cung cấp một tổng quan chung về các thành phần phần cứng và cấu trúc hệ điều hành ảnh hưởng đến phân tích bộ nhớ. Mặc dù các chương tiếp theo sẽ thảo luận về các chi tiết thực hiện liên quan đến các hệ điều hành cụ thể, chương này cung cấp thông tin nền hữu ích cho những người mới vào lĩnh vực này hoặc có thể cần làm mới kiến thức nhanh chóng. Chương bắt đầu bằng việc nhấn mạnh các khía cạnh quan trọng của kiến trúc phần cứng và kết thúc bằng việc cung cấp một tổng quan về các nguyên tắc thông thường của hệ điều hành. Các khái niệm và thuật ngữ được thảo luận trong chương này thường được đề cập trong toàn bộ phần còn lại của cuốn sách.

## Digital Environment (Môi trường kỹ thuật số)

Cuốn sách này tập trung vào điều tra các sự kiện diễn ra trong môi trường kỹ thuật số. Trong bối cảnh của môi trường kỹ thuật số, phần cứng cơ bản cuối cùng quyết định các ràng buộc về những gì một hệ thống cụ thể có thể thực hiện. Nhiều mặt khác, điều này tương tự như cách luật vật lý ràng buộc môi trường vật lý. Ví dụ, các nhà điều tra hiện trường vật lý hiểu về các luật vật lý liên quan đến chất lỏng có thể sử dụng các vết máu hoặc mô hình phân tán để hỗ trợ hoặc phủ nhận những yêu cầu về một vụ án cụ thể. Bằng cách áp dụng kiến thức về thế giới vật lý, các nhà điều tra có cái nhìn sâu sắc về cách hoặc tại sao một hiện vật cụ thể liên quan đến cuộc điều tra. Tương tự, trong môi trường kỹ thuật số, phần cứng cơ bản xác định các hướng dẫn có thể được thực thi và các tài nguyên có thể được truy cập. Các nhà điều tra có thể xác định các thành phần phần cứng độc đáo của một hệ thống và tác động của những thành phần đó có thể ảnh hưởng đến phân tích, họ đang ở vị trí tốt nhất để thực hiện một cuộc điều tra hiệu quả.

Trên hầu hết các nền tảng, phần cứng được truy cập thông qua một lớp phần mềm được gọi là hệ điều hành, điều khiển xử lý, quản lý tài nguyên và tạo điều kiện cho việc giao tiếp với các thiết bị bên ngoài. Hệ điều hành phải đối mặt với các chi tiết cấp thấp của bộ xử lý, các thiết bị và phần cứng bộ nhớ được cài đặt trong hệ thống cụ thể. Thông thường, hệ điều hành cũng thực hiện một tập hợp các dịch vụ và giao diện cấp cao xác định cách phần cứng có thể được truy cập bởi các chương trình của người dùng.

Trong quá trình điều tra, bạn tìm kiếm các hiện vật mà phần mềm hoặc người dùng nghi ngờ có thể đã đưa vào môi trường kỹ thuật số và cố gắng xác định cách môi trường kỹ thuật số đã thay đổi để phản ứng với những hiện vật đó. Sự quen thuộc của một nhà điều tra kỹ thuật số với phần cứng và hệ điều hành của một hệ thống cung cấp một khung tham chiếu quan trọng trong quá trình phân tích và xây dựng lại các sự kiện.

## PC  Architecture (Kiến trúc PC)

Phần này cung cấp tổng quan chung về cơ bản về phần cứng mà những nhà điều tra kỹ thuật số quan tâm đến phân tích bộ nhớ nên quen thuộc. Đặc biệt, cuộc thảo luận tập trung vào kiến trúc phần cứng tổng quát của máy tính cá nhân (PC). Chúng ta chủ yếu sử dụng thuật ngữ liên quan đến các hệ thống dựa trên Intel. Điều quan trọng cần lưu ý là thuật ngữ đã thay đổi theo thời gian và các chi tiết triển khai liên tục được cải tiến để cải thiện chi phí và hiệu suất. Mặc dù các công nghệ cụ thể có thể thay đổi, các chức năng chính mà các thành phần này thực hiện vẫn không thay đổi.


> **LƯU Ý:** Chúng tôi chỉ đề cập đến PC một cách tổng quát là máy tính với bộ vi xử lý Intel hoặc tương thích có thể chạy hệ điều hành Windows, Linux hoặc Mac OS X.

### Physical Organization (Tổ chức Vật lý)

Một PC bao gồm các mạch in được nối với nhau để kết nối các thành phần khác nhau và cung cấp các kết nối cho các thiết bị ngoại vi. Bảng chính trong loại hệ thống này, được gọi là bo mạch chủ, cung cấp các kết nối cho phép các thành phần của hệ thống giao tiếp với nhau. Những kênh giao tiếp này thường được gọi là bus máy tính. Phần này tập trung vào các thành phần và bus mà một nhà điều tra cần phải quen thuộc. Hình 1-1 minh họa cách các thành phần khác nhau được tổ chức theo cách thông thường trong phần này.

#### CPU và MMU

Hai thành phần quan trọng nhất trên bo mạch chủ là bộ xử lý, thực hiện các chương trình, và bộ nhớ chính, tạm thời lưu trữ các chương trình đã thực hiện và dữ liệu liên quan của chúng. Bộ xử lý thường được gọi là đơn vị xử lý trung tâm (CPU). CPU truy cập vào bộ nhớ chính để lấy các hướng dẫn và sau đó thực hiện các hướng dẫn đó để xử lý dữ liệu.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-21%20144154.png)

Đọc từ bộ nhớ chính thường chậm hơn đáng kể so với việc đọc từ bộ nhớ riêng của CPU. Do đó, các hệ thống hiện đại sử dụng nhiều lớp bộ nhớ nhanh, gọi là bộ nhớ cache, để giúp bù đắp sự chênh lệch này. Mỗi cấp độ cache (L1, L2 và tiếp theo) có tốc độ chậm hơn và dung lượng lớn hơn so với các cấp trước. Trong hầu hết các hệ thống, các cache này được tích hợp vào bộ xử lý và mỗi lõi của nó. Nếu dữ liệu không được tìm thấy trong cache cụ thể, dữ liệu phải được truy xuất từ cache cấp độ tiếp theo hoặc bộ nhớ chính.

Bộ xử lý phụ thuộc vào đơn vị quản lý bộ nhớ (MMU) của nó để giúp tìm nơi lưu trữ dữ liệu. MMU là đơn vị phần cứng dịch địa chỉ mà bộ xử lý yêu cầu thành địa chỉ tương ứng trong bộ nhớ chính. Như chúng ta sẽ mô tả sau trong chương này, các cấu trúc dữ liệu để quản lý việc dịch địa chỉ cũng được lưu trữ trong bộ nhớ chính. Bởi vì một dịch địa chỉ cụ thể có thể yêu cầu nhiều hoạt động đọc bộ nhớ, bộ xử lý sử dụng một bộ nhớ cache đặc biệt, được gọi là bộ đệm tìm kiếm dịch (TLB), cho bảng dịch MMU. Trước mỗi truy cập bộ nhớ, TLB được tham khảo trước khi yêu cầu MMU thực hiện một hoạt động dịch địa chỉ tốn kém.

Chương 4 sẽ thảo luận thêm về cách các cache này và TLB có thể ảnh hưởng đến việc thu thập bằng chứng bộ nhớ trong lĩnh vực pháp y số.













