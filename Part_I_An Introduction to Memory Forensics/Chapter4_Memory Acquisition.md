Thu thập bộ nhớ (tức là sao chép, ghi lại, lấy mẫu) liên quan đến việc sao chép nội dung của bộ nhớ tạm thời vào bộ nhớ không tạm thời. Đây có thể coi là một trong những bước quan trọng và rủi ro nhất trong quá trình pháp y bộ nhớ. Thật không may, nhiều nhà phân tích tin tưởng mù quáng vào các công cụ thu thập mà không ngừng xem xét cách thức hoạt động của những công cụ đó hoặc các vấn đề mà họ có thể gặp phải. Kết quả là họ sẽ gặp phải hình ảnh bộ nhớ bị hỏng, bằng chứng bị phá hủy và khả năng phân tích bị hạn chế, nếu có. Mặc dù chương này tập trung vào việc thu thập bộ nhớ của Windows, nhiều khái niệm cũng áp dụng cho các hệ điều hành khác. Bạn cũng sẽ tìm thấy các thảo luận riêng biệt về Linux và Mac OS X trong các chương tương ứng.

## Preserving the Digital Environment (Bảo vệ môi trường kỹ thuật số)

Mặc dù chương trình chính của cuốn sách này tập trung vào phân tích dữ liệu được lưu trữ trong bộ nhớ bay hơi, thành công của phân tích đó thường phụ thuộc vào giai đoạn thu thập thông tin của cuộc điều tra. Trong giai đoạn này, nhà điều tra phải đưa ra quyết định quan trọng về dữ liệu nào cần thu thập và phương pháp tốt nhất để thu thập dữ liệu đó. Điều cơ bản, việc thu thập bộ nhớ là quy trình sao chép nội dung của bộ nhớ vật lý vào thiết bị lưu trữ khác để bảo tồn. Chương này nhấn mạnh các vấn đề quan trọng liên quan đến việc truy cập dữ liệu được lưu trữ trong bộ nhớ vật lý và các yếu tố cần xem xét khi viết dữ liệu vào điểm đến của nó. Phương pháp và công cụ cụ thể mà bạn sử dụng thường phụ thuộc vào mục tiêu của cuộc điều tra và các đặc điểm của hệ thống bạn đang điều tra.

Nhà điều tra kỹ thuật số cố gắng bảo tồn trạng thái của môi trường kỹ thuật số một cách cho phép họ suy ra những kết luận đáng tin cậy thông qua phân tích. Dữ liệu được lưu trữ trên đĩa và trong RAM cung cấp hai trong số những thành phần quan trọng nhất của môi trường đó. Quan điểm truyền thống về điều tra kỹ thuật số tập trung vào giả định rằng độ tin cậy của các kết luận hoàn toàn phụ thuộc vào việc thu thập bằng chứng mà không làm thay đổi trạng thái của nó. Ví dụ, các quy trình xử lý bằng chứng được chấp nhận phổ biến bao gồm tắt hệ thống và tạo bản sao (ảnh) của dữ liệu trên thiết bị lưu trữ đĩa để phân tích ngoại tuyến. Những quy trình và thủ tục thu thập này tập trung vào việc giảm thiểu sự biến dạng của dữ liệu hệ thống tệp với chi phí là phá hủy các nguồn dữ liệu khác (tức là RAM, bộ nhớ thiết bị), cũng đóng vai trò trong môi trường kỹ thuật số.

Khi lĩnh vực điều tra kỹ thuật số đã phát triển, đã trở nên rõ ràng rằng việc bảo tồn lựa chọn của một số bằng chứng bằng giá của các bằng chứng quan trọng khác cũng có thể ảnh hưởng đến tính tin cậy của các kết luận suy ra từ đó. Điều này đặc biệt quan trọng khi các tác nhân xấu cố gắng khai thác hạn chế của các kỹ thuật thu thập bằng chứng kỹ thuật số truyền thống. Bằng cách phối hợp dữ liệu từ nhiều nguồn (đĩa, mạng, bộ nhớ, v.v.) trong môi trường kỹ thuật số, bạn thường có thể hiểu rõ hơn về những gì đã xảy ra trên hệ thống hơn là quan điểm hạn chế mà chỉ nội dung của bộ lưu trữ đĩa cung cấp một mình. Để bao gồm những nguồn khác nhau này, bạn phải chấp nhận rằng tất cả các phương pháp thu thập, bao gồm cả các thủ tục thu thập đĩa truyền thống, đều dẫn đến một số sự biến dạng đối với môi trường kỹ thuật số. Nhà điều tra phải nhận thức về cách biến dạng đó có thể ảnh hưởng đến phân tích của họ và thứ tự mà họ phải thu thập dữ liệu để giảm thiểu tác động đó. Hành động thường được ưu tiên dựa trên thứ tự giảm tính bay hơi (tức là bằng chứng thay đổi nhanh hơn được thu thập trước bằng chứng ổn định hơn). Như một vấn đề thực tế, điều này có nghĩa là bằng chứng bộ nhớ bay hơi cần được thu thập trước.

Ví dụ, trong hầu hết các tình huống, bạn không thể tạo ra khái niệm cổ điển về "ảnh" bộ nhớ vật lý nếu trạng thái chạy của máy thay đổi trong khi bạn thu thập bộ nhớ. Một mô tả phù hợp hơn về "ảnh hưởng bộ nhớ" là một mẫu của trạng thái bộ nhớ vật lý tại một thời điểm nhất định. Về mặt lý thuyết, có thể có khả năng đề cập đến việc thu thập các đơn vị diskret của bộ nhớ vật lý (trang), như "ảnh hưởng" đến các trang này nhưng trạng thái của bộ nhớ vật lý như một tổng thể không thể được đo trực tiếp và phải được suy ra từ trạng thái của các mẫu cá nhân này. Mặc dù quá trình thu thập mẫu bộ nhớ vật lý có thể tạo ra không chắc chắn trong giai đoạn thu thập, thông tin bổ sung mà nó cung cấp có thể dẫn đến sự tự tin cao hơn trong phân tích của nhà điều tra và giảm thiểu biến dạng của sự thật thực tế của cuộc điều tra.

### Acquisition Overview

Như đã đề cập trước đó, việc thu thập bộ nhớ không phải là một công việc đơn giản. Bạn sẽ cần một bộ công cụ linh hoạt và khả năng thích ứng với các kỹ thuật cụ thể của từng trường hợp và môi trường mà bạn gặp phải. Hình 4-1 hiển thị một cây quyết định tương đối đơn giản dựa trên một số, nhưng chắc chắn không phải là tất cả, các yếu tố thông thường mà bạn sẽ gặp phải trong lĩnh vực này. Ví dụ, một trong những câu hỏi đầu tiên bạn cần hỏi là liệu hệ thống(s) đích là một máy ảo (VM) hay không. Điều này có thể ảnh hưởng rất lớn đến phương pháp của bạn, vì nếu mục tiêu là một máy ảo, bạn có thể có các tùy chọn để thu thập bộ nhớ bằng các tính năng mà trình giả lập cung cấp để tạm dừng, treo, tạo bản snapshot hoặc sử dụng kiểm tra sâu. Tuy nhiên, quan trọng là bạn cần quen thuộc với các nền tảng ảo hóa khác nhau và các tính năng tương ứng của chúng, vì một số sản phẩm yêu cầu các bước đặc biệt và lưu trữ bộ nhớ VM dưới dạng định dạng độc quyền.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-25%20203346.png)

Nếu hệ thống đích là "bare metal", chẳng hạn như máy tính xách tay, máy tính để bàn hoặc máy chủ, bạn cần xác định liệu nó đang chạy hay không (hãy nhớ rằng máy có thể không luôn ở trong tầm kiểm soát vật lý của bạn). Nếu nó đang vào chế độ ngủ hoặc tắt nguồn, trạng thái hiện tại của bộ nhớ không còn dễ bay hơi. Tuy nhiên, trong nhiều trường hợp, dữ liệu dễ bay hơi gần đây có thể đã được ghi vào các thiết bị lưu trữ bền hơn như ổ cứng. Những nguồn dữ liệu thay thế này bao gồm tệp ngủ đông (hibernation files), tệp trang (page files) và tệp crash dump. Thu thập bộ nhớ từ các nguồn không bay hơi đòi hỏi khởi động hệ thống đích với một đĩa CD/DVD/USB sống để truy cập vào ổ cứng hoặc tạo một bản sao hình ảnh không gian lưu trữ và gắn nó (chỉ đọc) từ máy trạm phân tích của bạn.

Một hệ thống đang chạy cung cấp cơ hội để thu thập trạng thái hiện tại của bộ nhớ bay hơi nhưng bạn sẽ cần quyền quản trị viên. Nếu một nghi can hoặc nạn nhân đã đăng nhập với tài khoản quản trị viên, hoặc nếu họ hợp tác với cuộc điều tra của bạn (như cung cấp các thông tin đăng nhập chính xác), bạn may mắn. Bạn cũng có thể có quyền truy cập quản trị do vị trí của một nhà điều tra (công ty cung cấp thông tin đăng nhập cho bạn). Trong những tình huống này, bạn có thể sử dụng một tiện ích phần mềm, chúng tôi sẽ mô tả sắp tới. Nếu không, các tùy chọn không đơn giản lắm. Có thể có khả năng giành được quyền truy cập quản trị viên thông qua một lỗ hổng tăng quyền (một chiến thuật tấn công có thể làm mất tính minh bạch của bằng chứng pháp lý của bạn) hoặc bằng cách đoán mật khẩu bằng cách thử và sai.

Tùy chọn khác là thu thập hỗ trợ phần cứng. Trong trường hợp này, bạn không cần thông tin đăng nhập cho hệ thống đích - quyền truy cập vật lý là đủ. Bạn sẽ dựa vào Direct Memory Access (DMA) bằng cách sử dụng một công nghệ như Firewire, Thunderbolt, ExpressCard hoặc PCI. Nhược điểm của phương pháp này là trừ khi máy mục tiêu đã được trang bị một thiết bị (hoặc nếu nó hỗ trợ cắm nóng thiết bị), bạn phải tắt máy mục tiêu để cài đặt các bộ chuyển đổi phần cứng cần thiết (và việc này phá hủy dữ liệu bay hơi của bạn). Những khuyết điểm khác bao gồm việc Firewire chỉ cho phép thu thập 4GB đầu tiên của RAM, điều này có thể giới hạn nghiêm trọng khả năng thành công cho các hệ thống có bộ nhớ lớn. Ngoài ra, các thiết bị PCI cho thu thập bộ nhớ rất hiếm, nên chúng cũng rất đắt tiền. Trên thực tế, hiện tại, chúng tôi chỉ biết một sản phẩm hiện có và có giá khoảng 8000 USD (WindowsSCOPE).

Khi các yếu tố dẫn bạn vào hướng thu thập dựa trên phần mềm, bạn vẫn cần phải đưa ra nhiều quyết định. Hãy xem xét các điểm sau đây:

- Remote (từ xa) so với local (cục bộ): Bạn (hoặc một nhà điều tra đồng nghiệp) có quyền truy cập vật lý vào hệ thống đích không? Trường hợp có thể trở nên phức tạp nếu máy tính nằm ở một bang hoặc quốc gia khác. Bạn cũng có thể gặp tình huống máy đích là một máy chủ không có bàn phím hoặc màn hình kết nối, nằm trong một trung tâm an ninh hoặc trung tâm hoạt động mạng. Trong những tình huống này, thu thập từ xa (qua mạng) có thể là tùy chọn duy nhất của bạn.

- Chi phí: Bạn có hạn chế ngân sách cho phần mềm thu thập mà bạn có thể mua không? Đương nhiên, hạn chế này sẽ ảnh hưởng đến các công cụ có sẵn cho bạn.

- Định dạng: Bạn có yêu cầu bộ nhớ trong một định dạng tệp cụ thể không? Sau này trong chương, chúng tôi sẽ mô tả cách mà nhiều công cụ phân tích có hạn chế về các định dạng mà chúng hỗ trợ. Tuy nhiên, bạn luôn có thể chuyển đổi giữa các định dạng nếu ban đầu thu thập bộ nhớ trong một định dạng không tương thích với công cụ phân tích mong muốn của bạn.

- CLI (Command-Line Interface) so với GUI (Graphical User Interface): Bạn thích sử dụng công cụ giao diện dòng lệnh hay giao diện người dùng đồ họa (GUI)? Có một quan điểm chung rằng các công cụ GUI để lại dấu chân lớn trên hệ thống đích và có diện tích bề mặt tấn công lớn hơn, vì vậy chúng không phù hợp cho việc thu thập bằng phương pháp kỹ thuật điều tra số. Tuy nhiên, một GUI được viết tốt sẽ tạo ra ít ồn động hơn so với một công cụ dòng lệnh được viết kém. Ngoài ra, bạn có thể không có quyền truy cập console hoặc các dịch vụ Virtual Network Computing (VNC)/Remote Desktop Protocol (RDP) để chạy các ứng dụng GUI.

- Thu thập so với kiểm tra thời gian thực (runtime interrogation): Bạn cần lấy toàn bộ bộ nhớ vật lý hay chỉ cần có khả năng xác định các tiến trình đang chạy, các kết nối mạng, v.v.? Bạn có cần liên tục kiểm tra hệ thống để xem có thay đổi không? Trong môi trường doanh nghiệp với hàng trăm hoặc hàng nghìn hệ thống, việc lấy toàn bộ bộ nhớ vật lý để kiểm tra sự tồn tại của chỉ mục cụ thể không thực tế. Thay vào đó, bạn có thể thực hiện một quét nhanh trong toàn bộ doanh nghiệp, kiểm tra một số khu vực cụ thể trong bộ nhớ của mỗi máy tính.

Chúng tôi thường nhận được câu hỏi này - tôi nên sử dụng công cụ nào để thu thập bộ nhớ? Thật ra, không có câu trả lời rõ ràng cho câu hỏi này. Công cụ phù hợp cho một công việc cụ thể phụ thuộc vào công việc đó. Sau khi xác định các thông tin cụ thể của vụ việc, bạn có thể tập trung vào việc lựa chọn công cụ hỗ trợ tốt nhất cho mục tiêu của mình. Sau đây, chúng tôi sẽ trình bày một số biện pháp cụ thể mà bạn nên thực hiện khi thực hiện thu thập.

### The Risk of Acquisition

Trước khi bạn thực hiện việc thu thập bộ nhớ vật lý từ một hệ thống nghi ngờ, bạn nên luôn xem xét những rủi ro đi kèm. Vì hầu hết các hệ điều hành không cung cấp cơ chế tự nhiên được hỗ trợ để thu thập bộ nhớ vật lý, bạn sẽ sử dụng hệ thống một cách có thể khiến nó ở trong trạng thái không mong muốn. Ngoài ra, một hệ thống có mã độc viết kém có thể không ổn định và có thể hoạt động một cách không thể dự đoán được. Quyết định thu thập bộ nhớ vật lý yêu cầu bạn cân nhắc cân đối lợi ích của việc thu thập dữ liệu so với rủi ro đi kèm với quy trình thu thập. Ví dụ, nếu mục tiêu là một hệ thống quan trọng cho nhiệm vụ mà chỉ có thể tắt hoặc khởi động lại trong các tình huống cực đoan, bạn phải sẵn lòng giải thích tại sao việc thu thập bộ nhớ là cần thiết cho cuộc điều tra của bạn. Có thể có trường hợp thậm chí trong đó hậu quả (ví dụ: tử vong, hư hỏng môi trường) của việc làm mất ổn định một hệ thống sẽ không bao giờ đáng đổi lại với rủi ro.

>**Ghi chú:**<br> Điều quan trọng là người sẽ chịu hậu quả của các rủi ro (ví dụ: chủ sở hữu hệ thống, khách hàng, cấp trên của bạn) được thông báo đầy đủ trước khi chấp nhận các rủi ro này. Trên mức tổ chức, điều này có nghĩa là bạn cần phải có chính sách cụ thể về việc nào thì việc thu thập bộ nhớ vật lý và dữ liệu biến đổi khác là phù hợp trong ngữ cảnh phản ứng sự cố trước khi sự cố xảy ra thực sự.

Các phần sau mô tả một số lý do chính tại sao việc thu thập bộ nhớ có thể dẫn đến sự không ổn định của hệ thống và hỏng dữ liệu. Cần lưu ý rằng mặc dù chúng tôi sử dụng Microsoft Windows trong các ví dụ dưới đây, những vấn đề này không đặc hiệu cho bất kỳ hệ điều hành nào - chúng là chức năng của bộ vi xử lý và kiến trúc phần cứng.

#### Atomicity

Một hoạt động nguyên tử là hoạt động xuất hiện (đối với phần còn lại của hệ thống) hoàn tất ngay tức thì, không bị gián đoạn bởi các quy trình đồng thời. Tuy nhiên, việc thu thập bộ nhớ không phải là một hoạt động nguyên tử, vì nội dung của RAM liên tục thay đổi, ngay cả trên một hệ thống không hoạt động và đặc biệt trong quá trình thu thập. Như Dan Farmer và Wietse Venema đã viết "Bộ nhớ [...] có thể thay đổi rất nhanh, việc ghi lại thậm chí cả phần lớn sự biến đổi đó một cách chính xác và kịp thời không thể thực hiện mà không gây ra sự rối loạn mạnh mẽ trong hoạt động của một hệ thống máy tính điển hình" (xem Forensic Discovery). Do đó, không thể tránh được rằng công cụ thu thập của bạn sẽ làm thay đổi hệ thống (những thay đổi này nên được ghi chép). Trong quá trình thu thập, các quy trình khác đang ghi bộ nhớ, hạt nhân đang thêm/xóa các phần tử trong danh sách liên kết, các kết nối mạng đang được khởi tạo hoặc hủy bỏ, và còn nhiều thay đổi khác.

Trong trường hợp tốt nhất, bạn sẽ thu thập dữ liệu chứng cứ có thể giúp bạn suy luận về trạng thái hiện tại của hệ thống và một số hoạt động gần đây trong quá khứ. Tuy nhiên, lưu ý rằng trạng thái hiện tại có thể là thời điểm khi việc thu thập bắt đầu, khi nó kết thúc hoặc tại bất kỳ thời điểm nào giữa chúng - tùy thuộc vào thứ tự thu thập các trang bộ nhớ vật lý, chứa dữ liệu cần xét. Trong trường hợp tồi nhất, bạn sẽ có một bản ghi bộ nhớ bị hỏng mà các công cụ phân tích không thể xử lý, vì việc thu thập các trang cụ thể đã xảy ra sau khi một hoạt động quan trọng bắt đầu nhưng trước khi hoàn thành.

#### Device Memory

Physical memory trên các máy tính dựa trên x86 / x64 là một hệ thống địa chỉ hóa logic cho phép truy cập (hoặc "địa chỉ hóa") vào các tài nguyên trên bo mạch chủ khác nhau một cách thống nhất. Trong các máy tính này, firmware (BIOS) cung cấp một bản đồ bộ nhớ vật lý cho hệ điều hành với các khu vực khác nhau được đánh dấu là dành cho việc sử dụng bởi firmware, bởi các bus ISA hoặc PCI hoặc bởi các thiết bị bo mạch chủ khác nhau. Những khu vực này sau đó được gọi là các khu vực bộ nhớ thiết bị.

Hình 4-2 thể hiện một sơ đồ đơn giản về bố cục bộ nhớ vật lý, khi bạn xem xét các "lỗ" tồn tại do các khu vực bộ nhớ thiết bị. Trên các hệ thống 32-bit với bộ nhớ dưới 4GB, các "lỗ" này làm giảm tổng lượng bộ nhớ có sẵn cho hệ điều hành ít hơn so với dung lượng quảng cáo của các chip RAM (đây được gọi là "giới hạn hiệu quả bộ nhớ 32-bit cho khách hàng"). Để biết thêm thông tin, xem Pushing the Limits of Windows: Physical Memory của Mark Russinovich (http://blogs.technet.com/b/markrussinovich/archive/2008/07/21/3092070.aspx). Lưu ý làm thế nào chức năng MmGetPhysicalMemoryRanges bỏ qua các phạm vi bộ nhớ vật lý bị dành riêng cho các thiết bị. Nói cách khác, sử dụng API này tránh các khu vực bộ nhớ thiết bị.

Đọc không cẩn thận từ một trong những khu vực dành riêng này có thể gây nguy hiểm. Tùy thuộc vào tính chất của thiết bị bạn truy cập, việc đọc từ một địa chỉ vật lý trong khu vực có thể lấy dữ liệu được lưu trữ tại vị trí đó hoặc làm thay đổi trạng thái của thiết bị bạn đang truy cập. Ví dụ, có các địa chỉ vật lý được ánh xạ vào các thanh ghi thiết bị mà thay đổi trạng thái của thiết bị mỗi lần vị trí vật lý được đọc. Sự thay đổi này có thể làm rối các trình điều khiển thiết bị hoặc firmware phụ thuộc vào các giá trị trong các thanh ghi đó, dẫn đến việc hệ thống bị treo. Tình trạng treo hoặc treo đặc biệt phổ biến khi đọc các địa chỉ được chiếm dụng bởi chipset video, High Precision Event Timer (HPET) hoặc các thiết bị PCI kế thừa khó hiểu. Như một thách thức bổ sung, hầu hết các thiết bị này không được thiết kế để đáp ứng cùng một lúc từ nhiều bộ xử lý.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-25%20210327.png)

Mặc dù việc thu thập các khu vực bộ nhớ thiết bị có rủi ro, nhưng việc làm như vậy có thể tạo ra dữ liệu có giá trị pháp y cao. Ví dụ, dữ liệu trong những khu vực này có thể chứa bảng vectơ ngắt chế độ thực (IVT) với các dấu vết bị bỏ sót bởi rootkit dựa trên firmware. Bạn cũng có thể tìm thấy dấu vết của rootkit BIOS mà tiêm mã vào đỉnh bộ nhớ chế độ thực. Ngoài ra, bạn có thể sử dụng khu vực CMOS để thay đổi thứ tự khởi động và các thanh ghi truy cập gián tiếp IOAPIC để chuyển hướng ngắt. Trong số các công cụ thu thập bộ nhớ dựa trên phần mềm được thảo luận sau trong chương, chỉ có KnTDD của GMG Systems có thể thu thập các khu vực này với mức độ đáng tin cậy và chính xác khá cao.

>**Cảnh báo:**<br> Như đã được thể hiện trong bài viết của Mark, bạn có thể xem các phạm vi bộ nhớ vật lý đã được đặt dành bằng cách sử dụng Trình quản lý Thiết bị (View ➪ Resources by Connection). Theo cách lập trình, bạn có thể liệt kê chúng thông qua lớp WMI Win32_DeviceMemoryAddress (http://msdn.microsoft.com/en-us/library/aa394125%28v=vs.85%29.aspx). Cả hai phương pháp cuối cùng đều dựa vào dữ liệu trong hive Registry HARDWARE tạm thời. Do đó, mã độc có thể thêm các giá trị để ẩn bộ nhớ trong các khu vực đã được đặt dành. Trong một số trường hợp, điều này có thể phục vụ như một kỹ thuật chống pháp y (malware có thể tránh việc bị thu thập).


#### Cache Coherency

Các bộ xử lý hiện đại được thiết kế với một hoặc nhiều bộ nhớ đệm nội bộ nhằm cải thiện hiệu suất. Một bản ghi bảng trang có thể được lập trình với các thuộc tính bộ nhớ đệm khác nhau (không đệm, được đệm, ghi kết hợp), xác định cách mà bộ xử lý truy cập vào một trang bộ nhớ vật lý. Những bộ xử lý này có một hạn chế thiết kế được ghi nhận: chúng không được thiết kế để hỗ trợ việc ánh xạ đồng thời của cùng một địa chỉ vật lý với nhiều thuộc tính bộ nhớ đệm. Việc làm như vậy có thể dẫn đến hành vi không xác định của bộ xử lý, bao gồm hỏng TLB và hỏng dữ liệu tại địa chỉ bộ nhớ đã chỉ định. Do đó, một công cụ thu thập bộ nhớ viết kém có thể dễ dàng làm hỏng chính bộ nhớ đang được thu thập. Tham khảo Intel® 64 and IA-32 Architectures Developer’s Manual, Vol. 3A § 11.12.4.

Sự thống nhất bộ nhớ đệm là một trong những lý do chính tại sao Microsoft cảnh báo các nhà phát triển trình điều khiển Windows không nên ánh xạ các trang bộ nhớ vật lý trừ khi trình điều khiển của họ đã cấp phát trực tiếp bộ nhớ thông qua API đảm bảo không có thành phần hệ thống khác đã ánh xạ cùng một phạm vi bộ nhớ với một loại bộ nhớ đệm khác nhau (http://msdn.microsoft.com/en-us/library/windows/hardware/ff566481%28v=vs.85%29.aspx). Một quan điểm sai lầm phổ biến là rằng các công cụ thu thập bộ nhớ được bảo vệ khỏi xung đột bộ nhớ đệm nếu chúng sử dụng ZwMapViewOfSection để ánh xạ các trang bộ nhớ vật lý từ \Device\PhysicalMemory. Mặc dù điều này có thể đúng đối với một số phiên bản của Windows, nhưng không áp dụng cho tất cả các phiên bản; cũng không ngăn ngừa xung đột bộ nhớ đệm khi ánh xạ các trang bộ nhớ chưa được cấp phát hiện tại. 

Trong Windows 2000, API cho phép chỉ định các thuộc tính bộ nhớ đệm xung đột (xem http://ntsecurity.nu/onmymind/2006/2006-06-01.html). Bắt đầu từ Windows XP, API thực hiện so sánh các thuộc tính được yêu cầu và hiện có và thất bại bằng cách trả về lỗi STATUS_CONFLICTING_ADDRESSES nếu có sự không phù hợp. Tuy nhiên, bắt đầu từ Windows 7 và 2008 R2, lỗi này không còn được sinh ra. Phiên bản hiện tại của Windows thay thế các thuộc tính bộ nhớ đệm được lưu trữ trong cơ sở dữ liệu PFN một cách im lặng, ngay cả khi trang mục tiêu không được cấp phát hiện tại và giá trị trong mục tương ứng của cơ sở dữ liệu PFN không còn hợp lệ.

### When to Acquire Memory

Chọn thời điểm thích hợp để thu thập bằng chứng từ bộ nhớ vật lý phụ thuộc vào một số yếu tố. Các điểm sau đây chỉ là gợi ý, không phải là quy tắc cứng nhắc. Ví dụ, nếu bạn đang thu thập bằng chứng từ máy tính của một nghi phạm, bạn có thể lên kế hoạch thu thập vào thời điểm nghi phạm đang trực tuyến (hoặc ít nhất là đã đăng nhập), điều này cho phép bạn tiếp cận phiên đăng nhập của nghi phạm, thông tin về dịch vụ đám mây hoặc lưu trữ từ xa mà họ có thể truy cập và bất kỳ tài liệu được mã hóa nào mà nghi phạm có thể đã xem. Trong trường hợp khác, nếu bạn đang thu thập bằng chứng từ máy tính của nạn nhân, bạn có thể muốn định thời gian thu thập khi nghi phạm không hoạt động để tránh báo động cho nghi phạm. Thu thập dữ liệu phiên mạng đi vào và đi ra từ máy tính của nạn nhân trong một khoảng thời gian trước khi thu thập có thể giúp bạn xác định điều này.

>**Cảnh báo:**<br> Thu thập dữ liệu phiên mạng toàn diện tại chỗ có thể khó khăn nếu kẻ thù phát hiện ra công cụ theo dõi của bạn, hoặc nếu có các mạng không dây, Bluetooth hoặc radio được xác định bằng phần mềm (SDR) có sẵn và cung cấp các đường mạng thay thế. Tuy nhiên, các đường mạng thay thế này thường hội tụ vào một trung tâm mạng có dây hoặc sợi quang tại một số điểm có thể cung cấp vị trí chiến lược hơn để cài đặt thiết bị theo dõi mạng.

Hãy nhớ rằng, trong hầu hết các trường hợp, bạn đang thu thập bằng chứng từ một hệ thống máy tính đang chạy. Tăng lượng thay đổi diễn ra trong khi bạn thu thập bằng chứng từ bộ nhớ có thể làm tăng số lượng các hiện tượng bất thường bạn gặp phải khi phân tích bằng chứng. Nếu có thể, tránh thu thập bằng chứng từ bộ nhớ trong các khoảng thời gian có sự thay đổi đột ngột như khi khởi động hệ thống, tắt hệ thống hoặc khi các nhiệm vụ bảo trì hệ thống đang chạy (ví dụ: chống phân mảnh đĩa, quét virus toàn diện, sao lưu hệ thống). Cũng nên hạn chế tương tác với máy tính cho đến khi việc thu thập đã hoàn thành.

### How to Acquire Memory


Sau khi xác định thời điểm thuận tiện để thu thập bộ nhớ, bạn vẫn cần cân nhắc một số biện pháp phòng ngừa quan trọng. Phần này tập trung vào những trường hợp sử dụng phổ biến nhất và các lưu ý đi kèm, những điều này có thể dẫn đến hậu quả nghiêm trọng nếu bạn không cẩn thận.

#### Local Acquisition to Removable Media

Trong trường hợp này, bạn đang ghi dữ liệu bộ nhớ vào một ổ đĩa ngoài kết nối với hệ thống mục tiêu thông qua cổng USB, ESATA hoặc Firewire. Không bao giờ khuyến nghị ghi dữ liệu bộ nhớ vào ổ đĩa cục bộ của hệ thống mục tiêu, như phân vùng C:, vì điều này sẽ ghi đè lên một lượng lớn dữ liệu có thể liên quan đến vụ án của bạn (ví dụ: slack space - không gian chậm). Do kích thước của RAM trên máy tính hiện đại, bạn nên đảm bảo ổ đĩa đích được định dạng với hệ thống tập tin NTFS hoặc hệ thống tập tin khác với hiệu suất cao. Ngoài ra, hãy lưu ý rằng phần mềm độc hại thường lây lan bằng cách nhiễm nhiễm trên phương tiện lưu trữ ngoài. Các điểm sau đây cung cấp lời khuyên bổ sung cho việc thu thập cục bộ:

- Không bao giờ gắn cùng một phương tiện lưu trữ ngoài vào nhiều máy tính có thể bị nhiễm bệnh để tránh lây lan nhiễm trùng trong khi bạn đang thu thập chứng cứ.

- Đừng cắm các phương tiện lưu trữ ngoài có thể bị nhiễm trùng trực tiếp vào máy trạm pháp lý của bạn. Hãy kết nối phương tiện lưu trữ vào một hệ thống "hi sinh" trung gian và kiểm tra nó. Sau đó, sao chép dữ liệu chứng cứ vào máy trạm pháp lý của bạn qua một mạng nhỏ cô lập (ví dụ: qua hub hoặc switch không quản lý).

- Luôn luôn "khử trùng" (xóa an toàn) phương tiện lưu trữ ngoài trước khi sử dụng (hoặc tái sử dụng) để thu thập chứng cứ.

>**Chú ý:**<br> nếu phần mềm độc hại nhiễm vào firmware trên phương tiện lưu trữ ngoài, việc "khử trùng" pháp lý không đủ để giảm thiểu nguy cơ nhiễm bệnh thông qua việc tái sử dụng phương tiện lưu trữ.

#### Remote Acquisition

Trong kịch bản thu thập từ xa nguyên thủy, thông thường bạn sẽ đẩy các công cụ qua mạng đến máy mục tiêu thông qua PsExec (http://technet.microsoft.com/en-us/sysinternals/bb897553.aspx) hoặc sao chép chúng vào chia sẻ C$ hoặc ADMIN$ thông qua giao thức Server Message Block (SMB). Sau đó, bạn có thể lên lịch một tác vụ hoặc cài đặt một dịch vụ trên hệ thống mục tiêu chạy các công cụ và gửi nội dung của bộ nhớ vật lý trở lại bạn thông qua một máy lắng nghe netcat hoặc giao thức kết nối khác. Tuy nhiên, phương pháp này có những vấn đề chính là tiết lộ thông tin đăng nhập quản trị viên và nội dung của bộ nhớ RAM của hệ thống mục tiêu được gửi dưới dạng văn bản thuần túy qua mạng mở.

Bộ nhớ chính của máy tính chứa rất nhiều thông tin nhạy cảm có thể bị tiết lộ khi thu thập bằng văn bản thuần túy qua mạng mở. Trong môi trường tên miền hoặc doanh nghiệp, thông tin chứng nhận quản trị tên miền cung cấp một cách tiện lợi để truy cập hệ thống mục tiêu; tuy nhiên, nếu máy tính mục tiêu đã bị xâm nhập, kẻ tấn công có thể khôi phục lại các mã thông báo xác thực được tạo ra từ bộ nhớ để sử dụng trong các cuộc tấn công Pass the Hash (http://www.microsoft.com/security/sir/strategy/default.aspx#!password_hashes). Một giải pháp tốt hơn là tạo một tài khoản quản trị tạm thời chỉ cho phép truy cập vào hệ thống mục tiêu. Sau đó, tắt tài khoản quản trị tạm thời sau khi thu thập hoàn tất và theo dõi các nỗ lực sau này để sử dụng các thông tin đăng nhập này. Bạn cũng có thể xem xét chặn các kết nối từ máy mục tiêu trong tường lửa hoặc router (trừ các hệ thống liên quan đến việc thu thập từ xa). Điều này ngăn chặn bất kỳ phần mềm độc hại hoặc kẻ tấn công nào sử dụng thông tin đăng nhập đã bị đánh cắp để xâm nhập mạng.

Thu thập vào một chia sẻ mạng nên được sử dụng chỉ khi cần thiết hoặc là phương pháp cuối cùng. Bắt đầu với SMB 3.0 (Windows Server 2012), hỗ trợ mã hóa từ đầu đến cuối được hỗ trợ. Ngoài ra, một số công cụ (ví dụ: CryptCat, KnTDD, F-Response Enterprise) hỗ trợ thu thập dữ liệu qua mạng sử dụng SSL/TLS. Bạn cũng có thể xem xét nén dữ liệu chứng cứ trước khi chuyển nó qua mạng để giảm thời gian và băng thông cần thiết. Hãy nhớ tính toán các băm tích hợp trước và sau quá trình chuyển để đảm bảo chứng cứ của bạn không thay đổi trong quá trình truyền.

>**GHI CHÚ:**<br> Mã hóa kênh truyền thông không ngăn chặn tiết lộ thông tin nếu kẻ thù đã chiếm quyền sở hữu một đầu mút của kênh truyền thông hoặc có thể thực hiện cuộc tấn công Man-in-the-Middle (MITM).

#### Runtime Interrogation

Công cụ kiểm tra tại thời điểm thực thi (runtime interrogation) cho phép bạn nhanh chóng duyệt qua toàn bộ doanh nghiệp và kiểm tra các chỉ báo cụ thể trong bộ nhớ vật lý (thay vì thu thập toàn bộ bộ nhớ từ mỗi hệ thống). Thông thường, bạn thực hiện loại phân tích này theo cách tự động. Có nhiều bộ công cụ thương mại cung cấp khả năng tương tác với bộ nhớ vật lý cho doanh nghiệp, chẳng hạn như F-Response, AccessData Enterprise và EnCase Enterprise.

#### Hardware Acquisition

Hạn chế của phương pháp thu thập dữ liệu dựa trên phần cứng đã được đề cập trước đó, cuốn sách này không đi sâu vào các thu thập dữ liệu dựa trên phần cứng. Tuy nhiên, đáng để nhắc đến rằng Volatility hỗ trợ thu thập và kiểm tra bộ nhớ qua giao diện Firewire. Bạn sẽ cần thư viện libforensic1394 (https://freddie.witherden.org/tools/libforensic1394), ngăn xếp JuJu Firewire và sử dụng cú pháp đặc biệt của Volatility. Lưu ý thay đổi -l thay cho tham số -f:

```python
$ python vol.py –l firewire://forensic1394/<devno> plugin [options]
```

The `<devno>` là số thiết bị (thường là 0 nếu bạn chỉ kết nối với một thiết bị Firewire). Sử dụng plugin imagecopy để thu thập dữ liệu bộ nhớ hoặc bất kỳ plugin phân tích nào khác để xem xét hệ thống đang chạy, nhưng hãy nhớ giới hạn 4GB đã thảo luận trước đó. Một trường hợp sử dụng khác cho phân tích bộ nhớ dựa trên phần cứng là mở khóa các máy trạm. Ví dụ, công cụ Inception của Carsten Maartmann-Moe (http://www.breaknenter.org/projects/inception) tìm và sửa lỗi các chỉ thị cho phép bạn đăng nhập vào các máy tính chạy Windows, Linux và Mac OS X được bảo vệ bằng mật khẩu mà không cần mật khẩu. Tuy nhiên, như đã nêu trên trang web của công cụ, nếu các chỉ thị cần thiết không được tìm thấy trong 4GB dưới bộ nhớ, thì nó có thể không hoạt động đáng tin cậy.

## Software Tools

Tất cả các công cụ thu thập dữ liệu bộ nhớ dựa trên phần mềm đều tuân theo một giao thức tương tự để thu thập dữ liệu. Cụ thể, các công cụ này hoạt động bằng cách tải một module kernel mà sẽ ánh xạ các địa chỉ vật lý cần thiết vào không gian địa chỉ ảo của một tác vụ đang chạy trên hệ thống. Khi đó, chúng có thể truy cập dữ liệu từ không gian địa chỉ ảo và ghi nó vào bộ nhớ lưu trữ không bay hơi được yêu cầu. Phần mềm thu thập dữ liệu bộ nhớ có hai cách để thực hiện ánh xạ từ địa chỉ ảo sang địa chỉ vật lý:

- Phương pháp mà hầu hết, nếu không phải tất cả, các công cụ có sẵn thương mại sử dụng liên quan đến việc sử dụng một API hệ điều hành để tạo một mục bảng trang. Các hàm thông thường bao gồm: ZwMapViewOfSection (trên \Device\PhysicalMemory), MmMapIoSpace, MmMapLockedPagesSpecifyCache và MmMapMemoryDumpMdl.

- Phương pháp thứ hai có thể sử dụng các API hệ điều hành khác để cấp phát một mục bảng trang trống và mã hóa thủ công trang vật lý mong muốn vào mục bảng trang.

Có những rủi ro đáng kể liên quan đến cả hai cách tiếp cận để ánh xạ bộ nhớ vật lý thành bộ nhớ ảo trong việc thu thập dữ liệu bằng phần mềm. Ví dụ, các API được đề cập trước để ánh xạ địa chỉ vật lý thành địa chỉ ảo đều có một hạn chế chung là không được thiết kế để ánh xạ các trang mà một trình điều khiển không sở hữu. Nghĩa là, một công cụ thu thập dữ liệu không ổn định hơn các công cụ khác chỉ vì nó sử dụng MmMapIoSpace thay vì MmMapMemoryDumpMdl (ví dụ). Một khác biệt giữa các API quản lý bộ nhớ là một số API (ví dụ, ZwMapViewOfSection) ánh xạ địa chỉ vật lý được chỉ định vào không gian địa chỉ chế độ người dùng của ứng dụng thu thập (nơi nó có thể được truy cập bởi mã chế độ người dùng khác), trong khi các API khác ánh xạ địa chỉ vật lý vào không gian địa chỉ chế độ kernel.

>**Cảnh báo:**<br> Nhiều công cụ có thể tiến hành thu thập dữ liệu bộ nhớ bằng nhiều phương pháp để tránh các biện pháp phòng ngừa malware hoặc hệ điều hành (ví dụ: kết nối MmMapIoSpace). Vì những lý do này, Michael Cohen và Johannes Stüttgen đã nghiên cứu việc ánh xạ lại PTE (Page Table Entry) thủ công như một phương pháp thu thập có thể chống lại các kỹ thuật chống pháp y của điều tra. Chi tiết nghiên cứu có thể được tìm thấy tại địa chỉ: http://www.dfrws.org/2013/proceedings/DFRWS2013-p13.pdf.

Các công cụ thu thập cũng khác nhau trong cách xác định các địa chỉ vật lý cần bao gồm hoặc loại trừ trong quá trình thu thập. Một số công cụ bắt đầu tại địa chỉ vật lý 0 và tăng bộ đếm theo kích thước trang bình thường cho đến khi đạt đến giới hạn dự kiến (điều này cũng là một yếu tố khác trong phương trình) của bộ nhớ vật lý. Hầu hết, nếu không phải tất cả, các công cụ được thiết kế để bỏ qua các khu vực bộ nhớ thiết bị được hiển thị trong Hình 4-2. Biện pháp phòng ngừa này làm cho quá trình thu thập ổn định hơn, nhưng bỏ sót một lượng lớn chứng cứ bộ nhớ có thể chứa các hiện tượng của rootkit tinh vi. Trong khi đó, một số ứng dụng cung cấp tùy chọn để thu thập tất cả các địa chỉ vật lý từ trang 0 đến địa chỉ vật lý cuối cùng được nhận thấy. Điều này có nguy cơ hơn, đặc biệt trên các hệ thống trang bị hơn 4GB bộ nhớ, nhưng có khả năng tạo ra một biểu diễn toàn diện hơn về bộ nhớ của hệ thống mục tiêu. Lý tưởng nhất, bạn nên sử dụng một công cụ có khả năng thu thập dữ liệu liên quan từ các khu vực bộ nhớ thiết bị mà không làm đóng băng hoặc làm đứng hệ thống. Ví dụ, nếu công cụ thu thập có thể xác định thiết bị chiếm khu vực bộ nhớ vật lý cụ thể, công cụ đó có thể sử dụng một phương pháp phù hợp cho thiết bị đó.

### Tool Evaluation

Khác với các công cụ sao chép ổ đĩa, các công cụ thu thập bộ nhớ hiện chưa có các thông số kỹ thuật được phát triển hoặc các đánh giá được tiến hành. Thực tế, đây vẫn là một lĩnh vực nghiên cứu mở và chủ đề của các cuộc tranh luận nảy lửa. Một trong những thách thức chính khi đánh giá các công cụ thu thập bộ nhớ là chúng có thể hoạt động khác nhau tùy thuộc vào phiên bản của hệ điều hành, cấu hình của hệ điều hành và phần cứng được cài đặt. Cũng quan trọng là nhấn mạnh rằng các nền tảng ảo hóa thường dễ dàng đoán trước và đồng nhất hơn so với phần cứng thực, điều này có nghĩa là chúng có thể không cung cấp thông tin đáng tin cậy về cách một công cụ sẽ hoạt động trong thế giới đa dạng của phần cứng thực. Một số nghiên cứu triển vọng đang được tiến hành. Ví dụ, vào năm 2013, Stefan Voemel và Johannes Stüttgen đã công bố một bài báo nghiên cứu trong đó họ trình bày một nền tảng đánh giá cho phần mềm thu thập bộ nhớ pháp y (http://www.dfrws.org/2013/proceedings/DFRWS2013-11.pdf). Ngoài ra, công ty GMG Systems, Inc., cung cấp cho khách hàng của mình một phương pháp kiểm tra tính chính xác và đầy đủ của các công cụ thu thập bộ nhớ (MAUT).

Từ một góc độ hoạt động, các thuộc tính cơ bản của một công cụ thu thập pháp y đáng tin cậy là nó phải thu thập chứng cứ một cách chính xác, đầy đủ, được tài liệu hóa và có hệ thống ghi lỗi lỗi mạnh mẽ. Một vấn đề lớn trong số nhiều công cụ thu thập hiện tại là khi chúng thất bại (hoặc thất bại hoàn toàn hoặc khi đọc một hoặc nhiều trang), chúng thường thất bại một cách im lặng. Vấn đề này ngăn cản các nhà điều tra nhận ra rằng một vấn đề thậm chí còn tồn tại cho đến khi họ đến giai đoạn phân tích (ví dụ: Volatility không thể liệt kê các tiến trình), vào thời điểm đó thường là quá muộn để quay lại và thu thập một hình ảnh bộ nhớ khác từ hệ thống mục tiêu. Không có kỹ thuật thu thập chứng cứ nào là hoàn toàn không mắc lỗi. Tuy nhiên, nếu một công cụ thu thập có khả năng ghi nhật ký lỗi đáng tin cậy, thì nhà phân tích (hoặc người quyết định) có thể quyết định cách xử lý lỗi. Từ một góc độ hoạt động hoặc pháp lý, tình huống tồi tệ nhất là không biết bạn có gì sau khi hoàn thành việc thu thập chứng cứ.

Hãy nhớ rằng chỉ vì một công cụ không gây sự cố hoặc đóng băng máy mục tiêu, không có nghĩa là nó đã tạo ra một "hình ảnh" bộ nhớ chính xác và đầy đủ. Trên các hệ thống Microsoft Windows, bạn có thể sử dụng Microsoft driver verifier (https://support.microsoft.com/ kb/244617) để xác định liệu nhà cung cấp công cụ đã thực hiện sự cẩn thận hợp lý trong việc phát triển công cụ hay không. Ngoài ra, đường phòng thủ tốt nhất của bạn là kiểm tra công cụ trên các hệ thống giống như (càng gần càng tốt) các hệ thống mà bạn sẽ sử dụng chúng trong thực tế. Rất tiếc, điều này không luôn khả thi do sự kết hợp đa dạng của phần cứng và phần mềm tồn tại.

### Tool Selection

Dưới đây là danh sách các công cụ phổ biến để thu thập bộ nhớ, không theo thứ tự cụ thể. Phần này cũng không dự định cung cấp một danh sách toàn diện về các tính năng của các công cụ tương ứng. Hơn nữa, chúng tôi chưa xác nhận các thông tin hoặc có cơ hội sử dụng mọi tính năng được mô tả. Do đó, xin hãy không hiểu thông tin này như là một đánh giá hoặc chứng nhận cho bất kỳ sản phẩm cụ thể nào.

- GMG Systems, Inc., KnTTools: Công cụ này có những điểm nổi bật bao gồm các module triển khai từ xa, kiểm tra toàn vẹn mật mã, thu thập chứng cứ qua SSL, nén đầu ra, điều khiển băng thông tùy chọn, thu thập tự động dữ liệu trạng thái người dùng trực tiếp để tham chiếu chéo, thu thập pagefile, ghi lỗi mạnh mẽ và kiểm tra và tài liệu nghiêm ngặt. Nó cũng có thể thu thập ROM / EEPROM / NVRAM từ BIOS và bộ nhớ thiết bị ngoại vi (PCI, thẻ video, bộ chuyển mạng).

- F-Response: Bộ sản phẩm từ F-Response giới thiệu một khả năng mới đột phá trong pháp y bộ nhớ - khả năng xét vấn hệ thống trực tiếp từ xa qua kết nối iSCSI chỉ đọc. F-Response hiển thị một góc nhìn không phụ thuộc vào nhà cung cấp và hệ điều hành của hệ thống mục tiêu về bộ nhớ vật lý và ổ cứng, điều này có nghĩa là bạn có thể truy cập chúng từ các máy trạm phân tích Windows, Mac OS X hoặc Linux và xử lý chúng với bất kỳ công cụ nào.

- Mandiant Memoryze: Một công cụ bạn có thể dễ dàng chạy từ phương tiện ghi và hỗ trợ thu thập từ hầu hết các phiên bản phổ biến của Microsoft Windows. Bạn có thể nhập đầu ra XML của Memoryze vào Mandiant Redline để phân tích đồ họa các đối tượng trong bộ nhớ vật lý.

- HBGary FastDump: Một công cụ tuyên bố để để lại dấu chân nhỏ nhất có thể, có khả năng thu thập các tệp trang và bộ nhớ vật lý vào một tệp đầu ra duy nhất (HPAK) và khả năng sonde bộ nhớ quy trình (một thao tác có thể xâm phạm potay force các trang đã trao đổi được đọc lại vào RAM trước khi thu thập).

- MoonSols Windows Memory Toolkit: Gia đình MWMT bao gồm win32dd, win64dd và phiên bản mới nhất của DumpIt - một tiện ích kết hợp công cụ thu thập bộ nhớ 32- và 64-bit thành một tập tin thực thi chỉ cần nhấp chuột duy nhất để hoạt động. Không cần tương tác tiếp theo. Tuy nhiên, nếu bạn cần các tùy chọn nâng cao hơn, chẳng hạn như chọn giữa các loại định dạng đầu ra, kích hoạt mã hóa RC4 hoặc viết kịch bản thực thi trên nhiều máy tính, bạn cũng có thể làm điều đó.

- AccessData FTK Imager: Công cụ này hỗ trợ thu thập nhiều loại dữ liệu, bao gồm RAM. AccessData cũng bán một bộ dụng cụ USB phản ứng trực tiếp được cấu hình sẵn để thu thập bộ nhớ vật lý cùng với nhật ký trò chuyện, kết nối mạng và các thành phần khác.

- EnCase/WinEn: Công cụ thu thập từ Guidance Software có thể sao chép bộ nhớ theo định dạng nén và ghi lại siêu dữ liệu trong tiêu đề (như tên trường hợp, nhà phân tích, v.v.). Phiên bản Enterprise của EnCase sử dụng mã tương tự trong tác nhân của nó cho phép truy xét từ xa của hệ thống hoạt động (xem http://volatility-labs .blogspot.com/2013/10/sampling-ram-across-encase-enterprise.html).

- Belkasoft Live RAM Capturer: Một tiện ích quảng cáo khả năng thu thập bộ nhớ ngay cả khi có các cơ chế chống gỡ lỗi và chống ghi vào. Nó hỗ trợ tất cả các phiên bản Windows chính và 64 bit và có thể chạy từ một USB thumb drive.

- ATC-NY Windows Memory Reader: Công cụ này có thể lưu bộ nhớ dưới định dạng tệp hoặc tệp trình dùng đổ (crash dump) và

 bao gồm nhiều tùy chọn băm toàn vẹn. Khi được sử dụng từ môi trường giống UNIX như MinGW hoặc Cygwin, bạn có thể dễ dàng gửi đầu ra đến một bộ lắng nghe netcat từ xa hoặc qua một đường hầm SSH được mã hóa.

- Winpmem: Đây là công cụ thu thập bộ nhớ mã nguồn mở duy nhất cho Windows. Nó bao gồm khả năng đầu ra tệp theo định dạng nguyên thô hoặc đổ trình dùng, lựa chọn giữa các phương pháp thu thập khác nhau (bao gồm cả kỹ thuật PTE remapping thử nghiệm mạnh mẽ), và hiển thị bộ nhớ vật lý qua một thiết bị để phân tích trực tiếp trên hệ thống cục bộ.

### Memory Acquisition with KnTDD

KnTDD là một thành phần của gói KnTTools có sẵn từ GMG Systems, Inc. (http://www.gmgsystemsinc.com/knttools). Một phiên bản trước của phần mềm này đã giành giải thưởng Forensics Challenge (Thách thức Pháp y Kỹ thuật số) của Hội nghị Nghiên cứu Pháp y Kỹ thuật số DFRWS năm 2005 (http://www .dfrws.org/2005/challenge/kntlist.shtml) và sau đó đã được coi là một trong những gói phần mềm thu thập và phân tích bộ nhớ đáng tin cậy, tích hợp đầy đủ tính năng và có tài liệu tốt nhất. KnTTools có sẵn trong các phiên bản Basic và Enterprise. Dưới đây là các điểm nổi bật trong phiên bản Basic, được diễn đạt theo cách riêng:

- Thu thập bộ nhớ vật lý (bộ nhớ chính của máy tính) từ các hệ thống chạy hầu hết các phiên bản Windows từ XP Service Pack 3 đến Windows 8.1 và Server 2012 R2 (xem trang web để biết danh sách chính xác).

- Thu thập vào phương tiện lưu trữ dễ dàng di động; bao gồm các tập lệnh autorun.inf được cấu hình trước nhưng có thể tùy chỉnh để tự động hóa.

- Thu thập qua mạng (đến một bộ lắng nghe netcat được mã hóa tùy chỉnh) với hoặc không có giới hạn băng thông.

- Kiểm tra toàn vẹn mật mã (MD5, SHA1, SHA256, SHA512) và ghi lỗi mạnh mẽ.

- Nén đầu ra bằng zlib, gzip, bzip, lznt1 và các công cụ khác.

- Chuyển đổi hình ảnh bộ nhớ nhị phân sang định dạng Microsoft crash dump.

- Mã hóa lô đầu ra bằng chứng chỉ X509/PKCS#7.

- Thu thập một số thông tin trạng thái hệ thống, bao gồm các tiến trình đang hoạt động, các module được tải và các điểm cuối nghe lệnh bằng cách sử dụng các API chế độ người dùng (để sử dụng sau này trong thuật toán phát hiện trước sau).

- Trên Windows 2003 và phiên bản sau đó, NVRAM, bảng CMOS tiêu chuẩn, bảng IOAPIC, sector boot MBR và VBR được thu thập.

- Thu thập tệp hồ sơ của hệ thống (page file).

- Tích hợp với KnTList để phân tích và phân tích giao cắt. 

Bên cạnh đó, phiên bản Enterprise hỗ trợ các tính năng sau:

- Thu thập chứng cứ qua một đường hầm SSL/TLS.

- Thu thập chứng cứ vào một máy chủ WebDAV kích hoạt.

- Thu thập chứng cứ vào một máy chủ FTP vô danh.

- Phiên bản có thể triển khai từ xa chạy như một dịch vụ hệ thống (KnTDDSvc).

- Mô-đun triển khai từ xa (KnTDeploy) có thể kéo và triển khai các "gói" thu thập chứng cứ mã hóa từ một máy chủ web có SSL hoặc đẩy các gói ra một chia sẻ Admin$ từ xa trên máy tính "nghi phạm".

- Hỗ trợ các thiết bị giả mạo \\.\VideoMemory, \\.\NetXtremeMemory và \\.\NicMemory để thu thập RAM hoặc SRAM từ một số bộ điều khiển video và mạng chọn lọc.

### An Example of KnTDD in Action

Các bước dưới đây sẽ hướng dẫn bạn cách triển khai KnTTools từ thiết bị lưu trữ di động và thực hiện việc thu thập bộ nhớ vật lý và các tập tin trang trong định dạng mã hóa và nén:

## Memory Dump Formats

Tùy thuộc vào vai trò của bạn trong một vụ việc cụ thể, bạn có thể không phải là người chịu trách nhiệm thu thập bộ nhớ. Trên thực tế, người thực hiện việc thu thập bộ nhớ có thể không liên hệ với bạn trước khi thu thập chứng cứ. Do đó, bạn sẽ không có cơ hội chia sẻ các phương pháp tốt nhất với họ, đề xuất công cụ yêu thích của bạn hoặc yêu cầu chứng cứ theo một định dạng cụ thể. Tuy nhiên, bạn phải làm việc với những gì bạn nhận được. May mắn thay, Volatility sử dụng các vòng bầu cử không gian địa chỉ để tự động xác định định dạng tệp cho bạn. Nó tự động duyệt qua tất cả các định dạng được hỗ trợ cho đến khi tìm ra không gian địa chỉ phù hợp cho tệp mục tiêu dựa trên các byte kỳ diệu hoặc các mẫu khác.

Khung làm việc Volatility cũng cung cấp một số plugin, được liệt kê trong Bảng 4-1, để khám phá siêu dữ liệu liên quan đến nhiều định dạng tệp thông thường.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-25%20215053.png)

### Raw Memory Dump

Một bản sao thô của bộ nhớ là định dạng được hỗ trợ rộng rãi nhất trong các công cụ phân tích. Nó không chứa bất kỳ tiêu đề, siêu dữ liệu hoặc giá trị kỳ diệu nào để xác định loại tệp. Định dạng thô thông thường bao gồm các khoảng trống cho bất kỳ phạm vi bộ nhớ nào được bỏ qua cố ý (ví dụ: bộ nhớ thiết bị) hoặc không thể đọc được bởi công cụ thu thập, điều này giúp duy trì tính nguyên vẹn không gian (tương đối tại các vị trí dữ liệu).

### Windows Crash Dump

Định dạng tệp Windows crash dump được thiết kế cho mục đích gỡ lỗi. Crash dump bắt đầu với cấu trúc _DMP_HEADER hoặc _DMP_HEADER64, như được hiển thị trong mã sau đây. Thành viên Signature chứa PAGEDUMP hoặc PAGEDU64 tương ứng.

```
>>> dt("_DMP_HEADER")
'_DMP_HEADER' (4096 bytes)
0x0   : Signature                 ['array', 4, ['unsigned char']]
0x4   : ValidDump                 ['array', 4, ['unsigned char']]
0x8   : MajorVersion              ['unsigned long']
0xc   : MinorVersion              ['unsigned long']
0x10  : DirectoryTableBase        ['unsigned long']
0x14  : PfnDataBase               ['unsigned long']
0x18  : PsLoadedModuleList        ['unsigned long']
0x1c  : PsActiveProcessHead       ['unsigned long']
0x30  : MachineImageType          ['unsigned long']
0x34  : NumberProcessors          ['unsigned long']
0x38  : BugCheckCode              ['unsigned long']
0x40  : BugCheckCodeParameter     ['array', 4, ['unsigned long long']]
0x80  : KdDebuggerDataBlock       ['unsigned long long']
0x88  : PhysicalMemoryBlockBuffer ['_PHYSICAL_MEMORY_DESCRIPTOR']
[snip]
```

Header xác định phiên bản hệ điều hành chính và phiên bản phụ, DTB (DirectoryTableBase) kernel, địa chỉ của tiến trình đang hoạt động và danh sách đầu đầu vào kernel đã tải, và thông tin về các phạm vi bộ nhớ vật lý. Nó cũng hiển thị các mã kiểm tra lỗi (bug check codes), mà một bộ gỡ lỗi sử dụng để xác định nguyên nhân gây ra sự cố.

>**LƯU Ý:**<br> Tệp crash dump có nhiều kích thước và định dạng. Nếu bạn muốn crash dump của mình tương thích với Volatility, nó phải là một bản sao hoàn chỉnh của bộ nhớ (complete memory dump), không phải là kernel memory dump hoặc small dump. Bài viết "Understanding Crash Dump Files" mô tả chi tiết sự khác biệt: http://blogs.technet.com/b/askperf/archive/2008/01/08/understanding-crash-dump-files.aspx.

Dưới đây là danh sách mô tả cách tạo crash dump. Lưu ý rằng không phải tất cả các phương pháp đều phù hợp cho mục đích pháp lý:
- Blue Screens (Màn hình xanh): Bạn có thể cấu hình một hệ thống để tạo crash dump khi xảy ra lỗi màn hình xanh (BSoD) (xem KB 969028). Cho mục đích kiểm tra, bạn có thể sử dụng công cụ NotMyFault từ Sysinternals (http://download.sysinternals.com/files/NotMyFault.zip), bao gồm một trình điều khiển lỗi cố ý gây ra BSoD. Tuy nhiên, phương pháp này tắt hệ thống mạnh, có thể dẫn đến mất dữ liệu khác.

- CrashOnScrollControl: Một số bàn phím PS/2 và USB có các chuỗi phím đặc biệt tạo crash dump (xem KB 244139). Trên các hệ thống máy chủ không có bàn phím kết nối, bạn có thể sử dụng Non-Maskable Interrupts (xem KB 927069). Tuy nhiên, những phương pháp này thường yêu cầu cấu hình trước trong registry và BIOS.

- Debuggers (Công cụ gỡ lỗi): Nếu bạn đã kết nối với một hệ thống mục tiêu bằng cách sử dụng một bộ gỡ lỗi nhân hệ thống từ xa (WinDBG), bạn có thể sử dụng các lệnh .crash hoặc .dump. Điều này thuận tiện cho việc gỡ lỗi, nhưng hiếm khi áp dụng cho các tình huống pháp lý. Bạn cũng có thể sử dụng LiveKD (http://download.sysinternals.com/files/LiveKD.zip) nếu bạn đang gỡ lỗi trực tiếp trên máy tính mục tiêu, nhưng thường yêu cầu cài đặt phần mềm đặc biệt trước trên hệ thống mục tiêu.

Các phương pháp mà chúng ta vừa mô tả đều dựa vào cùng một cơ chế bên trong kernel để tạo crash dump. Do đó, chúng đều có một số điểm yếu tương tự. Chúng thường không bao gồm các khu vực bộ nhớ của thiết bị hoặc trang vật lý đầu tiên, mà có thể chứa bản sao của Master Boot Record (MBR) từ ổ đĩa và các mật khẩu xác thực trước khởi động. Hơn nữa, chúng có thể bị chiếm đoạt bởi phần mềm độc hại đăng ký một lời gọi thông báo lỗi (xem "Malicious Callbacks" trong Chương 13) hoặc bằng cách tắt truy cập vào bộ gỡ lỗi nhân hệ thống. Một số hệ thống có thể không thể tạo crash dump hoàn chỉnh do kích thước (xem http://media.blackhat.com/bh-us-10/whitepapers/Suiche/BlackHat-USA-2010-Suiche-BlueScreen-of-the-Death-is-dead-wp.pdf). Một số công cụ thu thập bộ nhớ pháp lý như MoonSols MWMT cung cấp tùy chọn tạo tệp crash dump. Trong những trường hợp này, các công cụ này xây dựng phần tiêu đề crash dump riêng của họ và sau đó ghi các khối vật lý bộ nhớ vào tệp kết quả. Nói cách khác, chúng tạo ra một tệp crash dump tương thích với bộ gỡ lỗi WinDBG (và do đó Volatility), nhưng không sử dụng cùng một cơ chế nhân hệ thống mà các kỹ thuật khác sử dụng. Do đó, nhiều điểm bất lợi như việc thiếu trang đầu tiên, chiếm đoạt thông qua callbacks và giới hạn kích thước được tránh. Volatility và KnTDD cũng có thể chuyển đổi bản sao bộ nhớ thành tệp crash dump, như bạn sẽ tìm hiểu sau trong chương.

### Windows Hibernation File


Tệp hibernation (hiberfil.sys) chứa một bản sao nén của bộ nhớ mà hệ thống ghi vào đĩa trong quá trình hibernation. Năm 2008, Matthieu Suiche (MoonSols) phát triển công cụ đầu tiên, Sandman, để phân tích các tệp này cho mục đích pháp lý. Bạn có thể đọc về nghiên cứu ban đầu của ông trong bài báo "Windows Hibernation File For Fun 'N' Profit" (http://sebug.net/paper/Meeting-Documents/BlackHat-USA2008/BH_US_08_Suiche_Windows_hibernation.pdf). Tệp hibernation bao gồm một tiêu đề tiêu chuẩn (PO_MEMORY_IMAGE), một tập hợp các ngữ cảnh kernel và thanh ghi như CR3 và một số mảng các khối dữ liệu nén. Định dạng nén sử dụng là Xpress cơ bản (http://msdn.microsoft.com/en-us/library/hh554002.aspx). Tuy nhiên, bắt đầu từ Windows 8 và Server 2012, Microsoft bắt đầu sử dụng thuật toán Xpress kèm theo mã hóa Huffman và LZ. Dưới đây là một ví dụ về tiêu đề của tệp hibernation:

```
>>> dt("PO_MEMORY_IMAGE")
'PO_MEMORY_IMAGE' (168 bytes)
0x0  : Signature                ['String', {'length': 4}]
0x4  : Version                  ['unsigned long']
0x8  : CheckSum                 ['unsigned long']
0xc  : LengthSelf               ['unsigned long']
0x10 : PageSelf                 ['unsigned long']
0x14 : PageSize                 ['unsigned long']
0x18 : ImageType                ['unsigned long']
0x20 : SystemTime               ['WinTimeStamp', {}]
[snip]
```

Thành viên Signature thường chứa các giá trị hibr, HIBR, wake hoặc WAKE. Tuy nhiên, trong một số trường hợp, toàn bộ tiêu đề PO_MEMORY_IMAGE bị xóa (xảy ra khi hệ thống tiếp tục hoạt động), điều này có thể ngăn cản việc phân tích tệp hibernation trong hầu hết các công cụ. Trong những trường hợp đó, Volatility sử dụng một thuật toán "brute force" để xác định dữ liệu cần thiết. Khi thực hiện phân tích tệp hibernation với Volatility, hãy nhớ rằng mỗi lần bạn chạy một lệnh, bạn cần giải nén các đoạn dữ liệu cụ thể. Để tiết kiệm thời gian, chúng tôi khuyến nghị bạn nén toàn bộ bản sao lưu bộ nhớ một lần (bằng cách sử dụng lệnh imagecopy, mà chúng tôi sẽ mô tả sau đây). Việc giải nén này chuyển đổi tệp hibernation thành một bản sao lưu bộ nhớ nguyên gốc mà bạn có thể phân tích mà không cần giải nén trực tiếp.

Như được giải thích trong Microsoft KB 920730, để tạo một tệp hibernation, trước tiên hãy bật chế độ hibernation trong kernel (powercfg.exe /hibernate on) và sau đó thực hiện lệnh shutdown /h để hibernate. Tùy thuộc vào phiên bản hệ điều hành của bạn, bạn cũng có thể thực hiện bằng cách nhấp chuột từ menu Start (Start ➪ Hibernate hoặc Start ➪ Shutdown ➪ Hibernate). Tuy nhiên, trong hầu hết các trường hợp pháp lý, bạn sẽ nhận được một máy tính xách tay đã được hibernated hoặc bạn sẽ nhận được một hình ảnh đĩa pháp lý từ hệ thống có sẵn tệp hibernation. Trong trường hợp này, bạn sẽ phải sao chép tệp hiberfil.sys ra khỏi ổ C: bằng cách gắn ổ đĩa từ máy tính phân tích (hoặc bằng cách sử dụng đĩa CD / DVD).

>**Cảnh báo:**<br> Trước khi một hệ thống hibernates, cấu hình DHCP (nếu có) sẽ được giải phóng và bất kỳ kết nối hoạt động nào cũng sẽ bị chấm dứt. Kết quả là, dữ liệu mạng trong các tệp hibernation có thể bị thiếu sót. Hơn nữa, trong thời gian này, phần mềm độc hại có thể loại bỏ chính nó khỏi bộ nhớ để bạn không thể phát hiện sự tồn tại của nó trong tệp hibernation.

### Expert Witness Format (EWF)

Memory được tạo bởi EnCase được lưu trữ trong định dạng Expert Witness Format (EWF). Đây là một định dạng phổ biến do sự phổ biến của EnCase trong các cuộc điều tra pháp lý. Do đó, bạn nên quen thuộc với các phương pháp sau để phân tích các bản sao bộ nhớ EWF:

- **EWFAddressSpace:** Volatility bao gồm một không gian địa chỉ có thể làm việc với các tệp EWF, nhưng nó yêu cầu bạn cài đặt libewf (https://code.google.com/p/libewf). Tại thời điểm viết bài này, libewf có thể hoàn toàn hỗ trợ các tệp EWF được sử dụng bởi EnCase v6 và các phiên bản trước đó, nhưng phiên bản mới hơn là EWF2-EX01 được giới thiệu với EnCase v7 chỉ có tính thử nghiệm.

- **Mounting với EnCase:** Bạn có thể mount một tệp EWF bằng EnCase và sau đó chạy Volatility trên thiết bị được hiển thị. Phương pháp này cũng hoạt động trong môi trường mạng cho phép lấy mẫu (xem Sampling RAM Across the EnCase Enterprise tại http://volatility-labs.blogspot.com/2013/10/sampling-ram-across-encase-enterprise.html). Phương pháp này cũng tránh phụ thuộc vào libewf và hỗ trợ các tệp EnCase v7.

- **Mounting với FTK Imager:** Một lựa chọn khác là mount tệp EWF dưới dạng "Physical & Logical" và sau đó chạy Volatility trên phần không được cấp phát của ổ đĩa (ví dụ: E:\unallocated space nếu tệp được mount trên ổ đĩa E:).

>**LƯU Ý:**<br> Mặc dù EWFAddressSpace được đi kèm với Volatility, nhưng nó không được bật mặc định (do sự phụ thuộc vào libewf). Để kích hoạt nó, sử dụng --plugins=contrib/plugins trong lệnh của bạn, như được mô tả trong Chương 3.

### HPAK Format


HPAK là một định dạng dữ liệu do HBGary thiết kế, cho phép nhúng bộ nhớ vật lý và tệp trang (page file) của một hệ thống mục tiêu vào cùng một tập tin đầu ra. Đây là một định dạng độc quyền, do đó chỉ có công cụ FastDump mới có khả năng tạo tập tin HPAK. Cụ thể, với công cụ này, bạn phải sử dụng tùy chọn dòng lệnh -hpak. Nếu không, bản sao bộ nhớ sẽ được tạo ra trong định dạng mặc định (raw), không bao gồm các tệp trang. Bạn có thể tùy chọn cung cấp tùy chọn -compress để nén dữ liệu với zlib. Tập tin kết quả sẽ có phần mở rộng .hpak.

Để đáp ứng nhu cầu của một số người dùng Volatility đã nhận các tập tin HPAK để phân tích mà không ngờ đến, chúng tôi đã tạo một không gian địa chỉ để xử lý chúng. Hãy nhớ rằng, nếu bạn không thực hiện việc thu thập dữ liệu, bạn phải xử lý dữ liệu mà bạn nhận được. May mắn là định dạng tập tin HPAK tương đối đơn giản. Nó có một tiêu đề 32 byte, như được hiển thị bên dưới, trong đó bốn byte đầu tiên là HPAK (chữ ký ma thuật). Các trường còn lại hiện tại chưa biết, nhưng không quan trọng đối với việc thực hiện phân tích bộ nhớ.

```
>>> dt("HPAK_HEADER")
'HPAK_HEADER' (32 bytes)
0x0     : Magic                 ['String', {'length': 4}]
```

Sau tiêu đề, bạn sẽ tìm thấy một hoặc nhiều cấu trúc HPAK_SECTION có dạng như sau:

```
>>> dt("HPAK_SECTION")
'HPAK_SECTION' (224 bytes)
0x0  : Header           ['String', {'length': 32}]
0x8c : Compressed       ['unsigned int']
0x98 : Length           ['unsigned long long']
0xa8 : Offset           ['unsigned long long']
0xb0 : NextSection      ['unsigned long long']
0xd4 : Name             ['String', {'length': 12}]
```

Giá trị 'Header' là một chuỗi như 'HPAKSECTHPAK_SECTION_PHYSDUMP' cho phần chứa bộ nhớ vật lý. Tương tự, nó trở thành 'HPAKSECTHPAK_SECTION_PAGEDUMP' cho phần chứa tệp trang của hệ thống mục tiêu. Các giá trị 'Offset' và 'Length' cho biết chính xác vị trí và độ dài của dữ liệu tương ứng trong tệp HPAK. Nếu 'Compressed' có giá trị khác không, điều đó có nghĩa là dữ liệu trong phần đó đã được nén bằng thuật toán zlib.

Như thể hiện trong lệnh dưới đây, bạn có thể sử dụng hpakinfo để khám phá nội dung của tệp HPAK:

```
$ python vol.py -f memdump.hpak hpakinfo
Header:     HPAKSECTHPAK_SECTION_PHYSDUMP
Length:     0x20000000
Offset:     0x4f8
NextOffset: 0x200004f8
Name:       memdump.bin
Compressed: 0
Header:     HPAKSECTHPAK_SECTION_PAGEDUMP
Length:     0x30000000
Offset:     0x200009d0
NextOffset: 0x500009d0
Name:       dumpfile.sys
Compressed: 0
```

Kết quả hiển thị cho bạn biết rằng tệp HPAK này chứa bộ nhớ vật lý và một tệp trang. Phần bộ nhớ vật lý bắt đầu tại offset 0x4f8 của tệp memdump.hpak và bao gồm 0x20000000 byte (512MB). Không có phần nào được nén. Bạn có thể chạy các plugin của Volatility trực tiếp trên memdump.hpak hoặc bạn có thể trích xuất phần bộ nhớ vật lý thành một tệp riêng bằng cách sử dụng plugin hpakextract. Khi trích xuất, bạn sẽ có một bản sao bộ nhớ tổng thể (raw memory dump) tương thích với hầu hết các framework phân tích.

### Virtual Machine Memory

Để thu thập bộ nhớ từ một máy ảo (VM), bạn có thể chạy một trong các công cụ phần mềm đã được đề cập trước đó trong hệ điều hành (OS) của máy ảo hoặc bạn có thể thực hiện việc thu thập từ hypervisor. Phần này tập trung vào việc thu thập bộ nhớ của máy ảo từ hypervisor. Kỹ thuật này thường ít xâm phạm hơn (khi thực hiện mà không tạm dừng hoặc đình chỉ VM), vì nó khó hơn để phần mềm độc hại đang hoạt động trong VM phát hiện sự hiện diện của bạn. Cuối phần này, chúng tôi cũng sẽ thảo luận về Actaeon, cho phép bạn thực hiện phân tích bộ nhớ trực tiếp của hypervisor thực sự (tức là phân tích một hệ điều hành khách trực tiếp trong bộ nhớ của máy chủ).

#### VMware

Nếu bạn đang sử dụng một sản phẩm trên máy tính cá nhân như VMware Workstation, Player hoặc Fusion, bạn chỉ cần đình chỉ hoặc tạo một bản snapshot của máy ảo. Kết quả là, một bản sao của bộ nhớ của máy ảo sẽ được ghi vào một thư mục trên hệ thống tệp của máy chủ, liên quan đến cấu hình .vmx. Nếu bạn đang sử dụng VMware Server hoặc ESX, bạn có thể thực hiện điều này từ giao diện console vSphere hoặc từ dòng lệnh với lệnh vmrun có thể thực thi bằng script (xem tại https://www.vmware.com/pdf/vix160_vmrun_command.pdf). Trong môi trường đám mây, bộ nhớ dump có thể được ghi vào một kho lưu trữ mạng (SAN) hoặc hệ thống tệp NFS (Network File System).

>**LƯU Ý:**<br> Hãy nhớ rằng việc tạm dừng hoặc đình chỉ một máy ảo không phải là không có hậu quả. Ví dụ, các kết nối SSL/TLS đang hoạt động không thể dễ dàng tiếp tục sau khi bị "đóng băng". Do đó, mặc dù bạn đang thực hiện việc lấy dữ liệu từ hypervisor, bạn vẫn gây ra (hạn chế) các thay đổi trong bộ nhớ của máy ảo.

Tùy thuộc vào sản phẩm và phiên bản VMware cùng cách tạo bản sao bộ nhớ, bạn có thể cần phải khôi phục nhiều hơn một tệp để phân tích bộ nhớ. Trong một số trường hợp, bộ nhớ của máy ảo được chứa hoàn toàn trong một tệp .vmem (theo cấu trúc gốc). Trong các trường hợp khác, thay vì tệp .vmem, bạn có thể nhận được một tệp .vmsn (các bản snapshot) hoặc .vmss (trạng thái đã lưu), đó là các định dạng tệp độc quyền chứa bộ nhớ và siêu dữ liệu (theo cấu trúc kết hợp). May mắn thay, Nir Izraeli đã tài liệu hóa định dạng đủ để tạo một không gian địa chỉ cho Volatility (xem công việc ban đầu của ông ở đây: http://code.google.com/p/vmsnparser).

Để làm tình hình phức tạp hơn một chút, Sebastian Bourne-Richard gần đây đã nhận ra rằng các sản phẩm VMware thường tạo một tệp .vmem và một trong các tệp dữ liệu siêu dữ liệu cấu trúc (theo cấu trúc phân tách). Một không gian địa chỉ hoàn toàn mới cần được viết cho Volatility để hỗ trợ cấu trúc này, vì tệp .vmem chứa các phạm vi bộ nhớ vật lý, nhưng tệp siêu dữ liệu cho biết cách ghép chúng lại để tạo thành một biểu diễn chính xác của bộ nhớ của máy ảo. Nói cách khác, khi lấy dữ liệu từ các hệ thống VMware, hãy chắc chắn khôi phục tất cả các tệp có đuôi mở rộng .vmem, .vmsn và .vmss - bởi vì không dễ biết trước tệp nào chứa bằng chứng cần thiết.

Ở offset 0 của các tệp dữ liệu siêu dữ liệu, bạn sẽ tìm thấy một cấu trúc _VMWARE_HEADER như sau:

```
>>> dt("_VMWARE_HEADER")
'_VMWARE_HEADER' (12 bytes)
0x0 : Magic         ['unsigned int']
0x8 : GroupCount    ['unsigned int']
0xc : Groups        ['array', lambda x : x.GroupCount, ['_VMWARE_GROUP']]
```

Để tệp được xem là hợp lệ, có một giá trị Magic phải là 0xbed2bed0, 0xbad1bad1, 0xbed2bed2, hoặc 0xbed3bed3. Trường Groups chỉ định một mảng các cấu trúc _VMWARE_GROUP có dạng như sau:

```
>>> dt("_VMWARE_GROUP")
'_VMWARE_GROUP' (80 bytes)
0x0  : Name          ['String', {'length': 64, 'encoding': 'utf8'}]
0x40 : TagsOffset    ['unsigned long long']
```

Mỗi nhóm có một tên cho phép các thành phần metadata được phân loại và một TagsOffset chỉ định vị trí bạn có thể tìm thấy một danh sách các cấu trúc _VMWARE_TAG. Một thẻ có dạng như sau: 

```
>>> dt("_VMWARE_TAG")
'_VMWARE_TAG' (None bytes)
0x0 : Flags      ['unsigned char']
0x1 : NameLength ['unsigned char']
0x2 : Name       ['String',
 {'length': lambda x : x.NameLength, 'encoding': 'utf8'}]
```

Các cấu trúc thẻ này là chìa khóa để tìm dữ liệu bộ nhớ vật lý trong tập tin metadata. Nếu máy ảo có ít hơn 4GB RAM, một dòng bộ nhớ vật lý duy nhất được lưu trữ trong một nhóm có tên "memory" và một thẻ có tên "Memory." Đối với các hệ thống có hơn 4GB RAM, có nhiều dòng, cũng trong một nhóm có tên "memory," nhưng bao gồm các thẻ có tên "Memory," "regionPPN," "regionPageNum," và "regionSize." Không gian địa chỉ bên trong Volatility (xem volatility/plugins/addrspaces/vmware.py) phân tích các thẻ này để xây dựng lại góc nhìn về bộ nhớ vật lý.

#### VirtualBox

VirtualBox không tự động lưu một bản sao toàn bộ RAM xuống đĩa khi bạn tạm dừng hoặc tạm ngưng một máy ảo (như các sản phẩm ảo hóa khác làm). Thay vào đó, bạn phải tạo một bản sao bộ nhớ bằng cách sử dụng một trong các kỹ thuật sau đây:

- `VBoxManage debugvm`: Dùng lệnh này để tạo một bản sao bộ nhớ core dump ELF64 nhị phân với các phần tử tùy chỉnh biểu diễn bộ nhớ vật lý của máy ảo. Thông tin chi tiết về lệnh này có thể được tìm thấy tại: http://www.virtualbox.org/manual/ch08.html#vboxmanage-debugvm.

- Sử dụng tùy chọn `--dbg` khi bắt đầu máy ảo và sau đó sử dụng lệnh `.pgmphystofile`. Phương pháp này tạo ra một bản sao bộ nhớ thô (raw memory dump). Để biết thêm thông tin, xem https://www.virtualbox.org/ticket/10222.

- Sử dụng VirtualBox Python API (vboxapi) để tạo tiện ích riêng để dump bộ nhớ. Người dùng cũng đính kèm một kịch bản Python có tên vboxdump.py cho ticket #10222 để cung cấp một ví dụ.

Những phương pháp thứ hai và thứ ba tạo ra các bản sao bộ nhớ thô (raw memory dumps), mà Volatility hỗ trợ trực tiếp. Tuy nhiên, phương pháp đầu tiên tạo ra một ELF64 core dump, yêu cầu hỗ trợ đặc biệt. Philippe Teuwen (xem http://wiki.yobi.be/wiki/RAM_analysis) đã thực hiện nghiên cứu ban đầu về định dạng tệp này và tạo ra một địa chỉ không gian (address space) Volatility hỗ trợ chúng. Nhờ đó, Cuckoo Sandbox có thể tích hợp khả năng lưu trữ bản sao bộ nhớ từ máy ảo VirtualBox dưới định dạng ELF64 core dump.

Các tệp ELF64 có một số phần tử tiêu đề chương trình tùy chỉnh. Một trong số chúng là PT_NOTE (elf64_note) với tên là VBCORE. Phần tử tiêu đề chứa một cấu trúc DBGFCOREDESCRIPTOR, như được thể hiện trong đoạn mã sau:

```
>>> dt("DBGFCOREDESCRIPTOR")
'DBGFCOREDESCRIPTOR' (24 bytes)
0x0  : u32Magic         ['unsigned int']
0x4  : u32FmtVersion    ['unsigned int']
0x8  : cbSelf           ['unsigned int']
0xc  : u32VBoxVersion   ['unsigned int']
0x10 : u32VBoxRevision  ['unsigned int']
0x14 : cCpus            ['unsigned int']
```

Cấu trúc này chứa chữ ký kỹ thuật ảo của VirtualBox (0xc01ac0de), thông tin phiên bản và số lượng CPU cho hệ thống mục tiêu. Nếu bạn tiếp tục duyệt qua các phần tử tiêu đề chương trình của tệp, bạn sẽ tìm thấy các đoạn PT_LOAD khác nhau (elf64_phdr). Mỗi đoạn PT_LOAD có thành viên p_paddr là địa chỉ bộ nhớ vật lý bắt đầu. Thành viên p_offset cho bạn biết ở vị trí nào trong tệp ELF64 bạn có thể tìm thấy phần bộ nhớ vật lý. Cuối cùng, thành viên p_memsz cho bạn biết kích thước của phần bộ nhớ (theo byte).

>**LƯU Ý**<br>
Để biết thêm thông tin về định dạng dump core ELF64 của VirtualBox, hãy xem các trang web sau đây:<br>
>- Định dạng dump core ELF64: http://www.virtualbox.org/manual/ch12.html#guestcoreformat<br>
>- Tệp tiêu đề mã nguồn của VirtualBox: http://www.virtualbox.org/svn/vbox/trunk/include/VBox/vmm/dbgfcorefmt.h<br>
>- Mã nguồn C tạo tệp dump core ELF64: http://www.virtualbox.org/svn/vbox/trunk/src/VBox/VMM/VMMR3/DBGFCoreWrite.cpp

#### QEMU

QEMU rất giống VirtualBox trong việc lưu bộ nhớ của máy ảo trong định dạng dump core ELF64. Trên thực tế, khác biệt chính duy nhất là tên PT_NOTE là CORE thay vì VBCORE. Bạn có thể tạo các dump này bằng cách sử dụng virsh, một giao diện dòng lệnh cho libvirt (http://libvirt.org/index.html). Cũng có một API Python, mà hiện tại Cuckoo Sandbox (http://www.cuckoosandbox.org/) sử dụng để tạo dump bộ nhớ của các máy ảo QEMU bị nhiễm mã độc.

#### Xen/KVM

Dự án LibVMI (https://github.com/bdpayne/libvmi) là một thư viện VM introspection hỗ trợ các hypervisor Xen và KVM. Nó cho phép bạn thực hiện phân tích thời gian thực của các VM đang chạy mà không thực thi bất kỳ mã nào bên trong VM. Điều này là một khả năng mạnh mẽ cho việc quét antivirus và rootkit trực tiếp, cũng như theo dõi tổng quát hệ thống. Bên cạnh đó, dự án còn bao gồm API Python (pyvmi) và một địa chỉ không gian Volatility (PyVmiAddressSpace) để phân tích bộ nhớ.

#### Microsoft Hyper-V

Để thu thập bộ nhớ từ các máy ảo Hyper-V, bạn cần lưu trạng thái của máy ảo hoặc tạo một bản snapshot. Sau đó, phục hồi các tệp .bin (đoạn bộ nhớ vật lý) và .vsv (metadata) từ thư mục cấu hình của máy ảo. Thật không may, Volatility hiện không hỗ trợ định dạng bộ nhớ Hyper-V, do đó bạn cần sử dụng tiện ích vm2dmp.exe (http://archive.msdn.microsoft.com/vm2dmp) để chuyển đổi các tệp .bin và .vsv thành một tệp crash dump của Windows. Sau đó, bạn có thể phân tích crash dump bằng WinDBG hoặc Volatility. Để biết thêm thông tin về quy trình này, hãy xem bài viết "Analyzing Hyper-V Saved State Files in Volatility" của Wyatt Roersma (http://www.wyattroersma.com/?p=77). Một trong những quan sát chính của Wyatt là vm2dmp.exe không hoạt động trên bất kỳ máy ảo nào có hơn 4GB RAM.

>**LƯU Ý**<br>
Bạn cũng có thể thu thập bộ nhớ từ máy ảo Hyper-V đang chạy bằng cách sử dụng Sysinternals LiveKD hoặc MoonSols LiveCloudKd (http://moonsols.com/2010/08/12/livecloudkd).

### Hypervisor Memory Forensics

Một trong những phát triển thú vị nhất trong pháp y VM memory là Actaeon (http://www.s3.eurecom.fr/tools/actaeon) do Mariano Graziano, Andrea Lanzi và Davide Balzarotti tạo ra. Với một bản sao lưu bộ nhớ vật lý của hệ thống chủ, công cụ này cho phép phân tích các hệ điều hành máy ảo trong môi trường ảo hóa sử dụng công nghệ Intel VT-x. Điều này bao gồm khả năng xác định các trình giả lập bộ nhớ cư trú (tích cực hoặc độc hại) và ảo hóa lồng nhau. Hiện tại, Actaeon cho phép trình giám sát máy ảo của các máy khách Windows 32-bit chạy dưới KVM, Xen, VMware Workstation, VirtualBox và HyperDbg. Actaeon được thực hiện dưới dạng một bản vá cho Volatility và đã giành giải nhất trong Cuộc thi Plugin Volatility năm 2013 (http://volatility-labs.blogspot.com/2013/08/results-are-in-for-1st-annual.html).

## Converting Memory Dumps

Ngoại trừ Volatility, hầu hết các framework phân tích bộ nhớ chỉ hỗ trợ một hoặc hai định dạng tập tin được đề cập ở phần trước. Nếu bạn nhận được một bản sao lưu bộ nhớ trong một định dạng không tương thích với công cụ phân tích mong muốn, bạn nên xem xét chuyển đổi nó. Như đã đề cập trước đó, định dạng raw là được hỗ trợ rộng rãi nhất, vì vậy thường trở thành định dạng đích trong quá trình chuyển đổi. Dưới đây là danh sách các công cụ có thể hỗ trợ bạn trong việc này:

- MoonSols Windows Memory Toolkit (MWMT): Bộ công cụ này cung cấp các tiện ích để chuyển đổi tệp hibernation và crash dump thành định dạng raw. Nó cũng có thể chuyển đổi tệp hibernation và raw thành crash dump.

- VMware vmss2core: Tiện ích vmss2core.exe (https://labs.vmware.com/flings/vmss2core) có thể chuyển đổi các tệp trạng thái đã lưu của VMware (.vmsn) hoặc snapshot (.vmsn) thành crash dump tương thích với Microsoft WinDBG hoặc gdb.

- Microsoft vm2dmp: Như đã mô tả trước đó, công cụ này có thể chuyển đổi một số tệp bộ nhớ Microsoft Hyper-V thành crash dump (phụ thuộc vào kích thước bộ nhớ và phiên bản máy chủ Hyper-V).

- Volatility imagecopy: Plugin imagecopy có thể sao chép một raw memory dump từ bất kỳ định dạng tệp nào sau đây: crash dump, tệp hibernation, VMware, VirtualBox, QEMU, Firewire, Mach-o, LiME và EWF.

- Volatility raw2dmp: Plugin raw2dmp có thể chuyển đổi raw memory dump thành Windows crash dump để phân tích bằng trình gỡ lỗi WinDBG của Microsoft.

Tùy thuộc vào các tùy chọn có sẵn, bạn nên đã sẵn sàng chuyển đổi từ hoặc sang bất kỳ định dạng tập tin nào mà bạn có thể gặp phải. Hãy nhớ rằng có thể cần thực hiện một chuyển đổi hai bước để đạt được mục tiêu cuối cùng của bạn. Ví dụ, nếu bạn nhận được một tệp hibernation và cần phân tích nó trong WinDBG (chỉ chấp nhận crash dump), thì bạn có thể chuyển đổi nó thành định dạng raw trước và sau đó từ raw thành crash dump.

Dưới đây là các lệnh sử dụng các plugin Volatility imagecopy và raw2dmp. Cả hai đều sử dụng tùy chọn -O/--output-image để chỉ định đường dẫn đến tập tin đích. Để chuyển đổi một crash dump (hoặc bất kỳ định dạng nào khác) thành một bản mẫu bộ nhớ raw, hãy sử dụng lệnh sau:

```
$ python vol.py -f win7x64.dmp --profile=Win7SP0x64 imagecopy -O copy.raw
Volatility Foundation Volatility Framework 2.4
Writing data (5.00 MB chunks): |........[snip]........................|
```

Để chuyển đổi một mẫu bộ nhớ raw thành tệp crash dump, sử dụng lệnh sau:
```
$ python vol.py -f memory.raw --profile=Win8SP0x64 raw2dmp -O win8.dmp
Volatility Foundation Volatility Framework 2.4
Writing data (5.00 MB chunks): |........[snip]........................|
```

Thời gian chuyển đổi phụ thuộc vào kích thước của tệp mẫu bộ nhớ ban đầu. Bạn sẽ thấy một thông báo tiến trình được in ra trên terminal cho mỗi khối 5MB được ghi vào tệp kết quả.

## Volatile Memory on Disk

Dữ liệu tạm thời thường được ghi vào lưu trữ bền vững như một phần của hoạt động hệ thống bình thường, chẳng hạn như trong quá trình ngủ đông và trang trí. Điều quan trọng là phải nhận thức về các nguồn dữ liệu tạm thời thay thế này, vì trong một số trường hợp, đó có thể là nguồn duy nhất của bạn (ví dụ, nếu một máy tính xách tay của đối tượng đang tắt khi bị tịch thu). Trên thực tế, ngay cả khi máy tính đang hoạt động và bạn có thể lấy dữ liệu tạm thời từ hệ thống trực tiếp, bạn có thể muốn khôi phục các nguồn dữ liệu tạm thời thay thế này. Chúng có thể cung cấp chứng cứ quan trọng về những gì đã xảy ra trên hệ thống trong những khoảng thời gian khác hoặc trang bị bộ nhớ đã được trang vào lưu trữ thứ cấp, mà bạn có thể sử dụng để liên kết với hoạt động hiện tại.

Kịch bản sau tập trung vào các khía cạnh kỹ thuật về cách tìm vị trí các dấu vết dữ liệu tạm thời trên đĩa và cách khôi phục chúng. Hình ảnh pháp y học (nhân bản) của ổ cứng nằm ngoài phạm vi của cuốn sách này, cũng như các bước để duy trì quy trình nắm giữ đúng cho chứng cứ. Vì vậy, chúng tôi cho rằng bạn đã chú ý nghiên cứu và tuân thủ các luật pháp của quốc gia của bạn, nếu áp dụng cho cuộc điều tra của bạn. Ngoài ra, mặc dù có nhiều sản phẩm GUI thương mại như EnCase và FTK cung cấp các phương pháp nhấp chuột để khôi phục các tệp tin, chúng tôi tập trung vào việc sử dụng The Sleuth Kit (http://www.sleuthkit.org) vì nó là mã nguồn mở và có sẵn cho tất cả mọi người.

### Recovering the Hibernation File

Nếu có sẵn, tệp ngủ đông của hệ thống sẽ tồn tại tại \hiberfil.sys trong thư mục gốc của phân vùng C:. Giả sử bạn có một bản sao ghi nhớ của ổ đĩa (image.dd trong ví dụ), bạn cần trước tiên xác định sector bắt đầu của phân vùng NTFS. Để làm điều này, bạn có thể sử dụng mmls như sau:

```
$ mmls image.dd
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

    Slot Start End Length Description
00: Meta  0000000000 0000000000 0000000001 Primary Table (#0)
01: ----- 0000000000 0000002047 0000002048 Unallocated
02: 00:00 0000002048 0031455231 0031453184 NTFS (0x07)
03: ----- 0031455232 0031457279 0000002048 Unallocated
```

Nếu có sẵn, tệp ngủ đông của hệ thống sẽ tồn tại tại \hiberfil.sys trong thư mục gốc của phân vùng C:. Giả sử bạn có một bản sao ghi nhớ của ổ đĩa (image.dd trong ví dụ), bạn cần trước tiên xác định sector bắt đầu của phân vùng NTFS. Để làm điều này, bạn có thể sử dụng mmls như sau:

```
$ fls -o 2048 image.dd | grep hiber
r/r 36218-128-1: hiberfil.sys
```

Mục tiêu hoặc số MFT trong trường hợp này là 36218. Bây giờ, tất cả những gì bạn cần làm là cung cấp phần tử lệnh và MFT cho lệnh icat để trích xuất nội dung của tệp. Trước khi thực hiện điều này, hãy đảm bảo rằng thiết bị đích có đủ không gian trống để chứa tệp ngủ đông. Dưới đây là cách thực hiện việc trích xuất:

```
$ icat –o 2048 image.dd 36218 > /media/external/hiberfil.sys

$ file /media/external/hiberfil.sys
/media/external/hiberfil.sys: data
```

Bây giờ, khi bạn đã khôi phục lại tệp ngủ đông, bạn có thể bắt đầu phân tích nó bằng Volatility. Tuy nhiên, có một lưu ý mà chúng ta sẽ khám phá tiếp theo, liên quan đến việc xác định hồ sơ (profile) thích hợp để sử dụng trong phân tích.

### Querying the Registry for a Profile

Thường thì, bạn sẽ nhận được bằng chứng, như một ổ đĩa cứng, mà không có bất kỳ thông tin chi tiết nào về hệ thống mục tiêu. Ví dụ, liệu đó có phải là 32-bit Windows 7 hay 64-bit Windows Server 2012? Bạn sẽ cần thông tin này để chọn hồ sơ (profile) Volatility phù hợp. Trong nhiều trường hợp, bạn có thể đơn giản chạy plugin kdbgscan, nhưng hãy nhớ rằng khối dữ liệu bộ gỡ lỗi (debugger data block) là không cần thiết và có thể bị tấn công và can thiệp bởi kẻ tấn công (xem Chương 3). Nếu điều đó xảy ra, bạn sẽ cần một phương pháp dự phòng để xác định hồ sơ hệ thống. Trong trường hợp này, bạn có quyền truy cập vào ổ cứng, chứa các hive registry, vì vậy bạn có thể tận dụng chúng trong quá trình phân tích. Các lệnh sau đây sẽ hướng dẫn bạn cách trích xuất các hive SYSTEM và SOFTWARE, sau đó xác minh rằng bạn đã khôi phục lại các tệp registry Microsoft Windows hợp lệ.

```
$ fls -o 2048 -rp image.dd | grep -i config/system$
r/r 58832-128-3: Windows/System32/config/SYSTEM

$ fls -o 2048 -rp image.dd | grep -i config/software$
r/r 58830-128-3: Windows/System32/config/SOFTWARE

$ icat -o 2048 image.dd 58832 > /media/external/system
$ icat -o 2048 image.dd 58830 > /media/external/software

$ file /media/external/system /media/external/software
system: MS Windows registry file, NT/2000 or above
software: MS Windows registry file, NT/2000 or above
```

Sau khi bạn trích xuất các tệp hive thích hợp, bạn có thể phân tích chúng bằng một trình phân tích registry ngoại tuyến. Trong trường hợp này, chúng ta sử dụng reglookup (một công cụ mã nguồn mở từ: http://projects .sentinelchicken.org/reglookup). Cụ thể, bạn có thể tìm giá trị ProductName trong hive SOFTWARE và giá trị PROCESSOR_ARCHITECTURE trong hive SYSTEM, như được thể hiện ở đây:

```
$ reglookup -p "Microsoft/Windows NT/CurrentVersion"
/Microsoft/Windows NT/CurrentVersion/ProductName,SZ,Windows 7 Professional,
 /media/external/software | grep ProductName

$ reglookup
 -p "ControlSet001/Control/Session Manager/Environment/PROCESSOR_ARCHITECTURE"
 /media/external/system
/ControlSet001/Control/[snip]/PROCESSOR_ARCHITECTURE,SZ,AMD64,
```

Trong đầu ra, hệ thống mục tiêu đang chạy Windows 7 Professional trên bộ vi xử lý AMD64. Do đó, hồ sơ (profile) sẽ là Win7SP0x64 hoặc Win7SP1x64. Bạn cũng có thể phân biệt giữa các bản dịch vụ bằng cách truy vấn giá trị CSDVersion trong registry.

### Recovering the Page File(s)

Thường chúng tôi đặt một câu hỏi "khó" cho học viên trong lớp đào tạo của chúng tôi: Nếu chúng tôi yêu cầu bạn khôi phục tệp trang, bạn sẽ tìm ở đâu? Hầu hết mọi người đều trả lời là C:\pagefile.sys. Mặc dù câu trả lời này không sai về mặt kỹ thuật, nhưng nó cũng không đầy đủ bởi vì một hệ thống Windows có thể có tới 16 tệp trang. Do đó, bạn nên xác định số lượng tệp trang có trong hệ thống và vị trí chúng trước khi thực hiện việc thu thập. Bạn có thể làm điều này bằng cách truy vấn registry SYSTEM, như được thể hiện dưới đây:

```
$ reglookup -p "ControlSet001/Control/Session Manager/Memory Management" -t MULTI_SZ /media/external/system

PATH,TYPE,VALUE,MTIME
/ControlSet001/Control/Session Manager/Memory
Management/PagingFiles,MULTI_SZ,\??\C:\pagefile.sys,
/ControlSet001/Control/Session Manager/Memory
Management/ExistingPageFiles,MULTI_SZ,\??\C:\pagefile.sys,
```

Hệ thống mục tiêu chỉ có một tệp trang nên C:\pagefile.sys đã là câu trả lời đúng trong trường hợp này. Đường dẫn tệp được hiển thị hai lần, bởi vì có hai giá trị: PagingFiles (được sử dụng) và ExistingPageFiles (đang được sử dụng). Nếu một hệ thống có nhiều hơn một tệp trang, bạn sẽ thấy danh sách đầy đủ các tên đường dẫn. Bạn có thể khôi phục tệp trang từ hình ảnh đĩa như sau:

```
$ fls -o 2048 image.dd | grep pagefile
r/r 58981-128-1: pagefile.sys

$ icat –o 2048 image.dd 58981 > /media/external/pagefile.sys
```

Bây giờ bạn đã tách riêng tệp trang từ hình ảnh đĩa, bạn có thể tiếp tục vào các giai đoạn phân tích.

>**Cảnh báo:**<br> Trong cùng một khóa trong hive `SYSTEM` mà chúng ta truy vấn để tìm các tệp trang, có một giá trị `ClearPageFileAtShutdown`. Chúng tôi đã thấy phần mềm độc hại thiết lập giá trị `DWORD` này thành 1 như một kỹ thuật chống pháp truy tốt, vì điều này dẫn đến việc xóa tệp trang khi hệ thống tắt nguồn. Trong trường hợp này, bạn vẫn có thể khôi phục một số dữ liệu tạm thời từ đĩa bằng cách khắc phục các khối đã giải phóng/đánh dấu lại bằng TSK hoặc một bộ công cụ pháp y tế đĩa khác.<br>
Ngoài ra, bắt đầu từ Windows 8.1, có một giá trị có tên `SavePageFileContents` trong `CurrentControlSet\Control\CrashControl` xác định xem Windows có nên bảo tồn nội dung của tệp trang trong suốt các lần khởi động lại hay không.

### Analyzing the Page File(s)

Nếu bạn nhớ từ trước trong chương này, một số công cụ phần mềm chạy trên hệ thống thời gian thực có thể thu thập tệp trang tại thời điểm thu thập. Dù bạn đã sử dụng một trong những công cụ đó hoặc trích xuất các tệp từ hình ảnh đĩa, các tùy chọn cho phân tích chi tiết nội dung tệp trang khá hạn chế. Hãy nhớ rằng một tệp trang chỉ là một tập hợp các mảnh ghép không theo thứ tự - mà không có bảng trang để cung cấp ngữ cảnh cần thiết, bạn không thể xác định chúng được sắp xếp như thế nào vào hình dung lớn hơn.

Nguyên tắc và Phân Tích Bộ Nhớ Windows bởi Nicholas Paul Maclean (http://www.4tphi.net/fatkit/papers/NickMaclean2006.pdf) đã mô tả khả năng bổ sung phân tích bộ nhớ thô bằng dữ liệu từ tệp trang để cung cấp một cái nhìn hoàn chỉnh hơn về bộ nhớ vật lý. Tuy nhiên, đối với hầu hết các trường hợp, việc thực hiện thực tế của kỹ thuật này chưa được xác minh hoặc không thể tiếp cận.

Tài liệu của HBGary Responder khẳng định rằng nó hỗ trợ phân tích tệp trang. Ngoài ra, WinDBG quảng cáo hỗ trợ tích hợp tệp trang (xem http://msdn.microsoft.com/en-us/library/windows/hardware/dn265151%28v=vs.85%29.aspx). Cụ thể, theo tài liệu, bạn có thể tạo một tệp CAB chứa bản ghi bộ nhớ và các tệp trang và phân tích nó với bộ gỡ lỗi. Tuy nhiên, một cuộc thảo luận trên danh sách gửi OSR cho thấy tuyên bố này chủ yếu là sai lầm (hoặc chỉ đơn giản là lỗi thời) (http://www.osronline.com/showthread.cfm?link=234512).

Mặc dù phân tích tệp trang đang nằm trong kế hoạch của chúng tôi, vào thời điểm viết bài này, bạn không thể thực hiện phân tích như vậy với Volatility. Do đó, tùy chọn điều tra tốt nhất của bạn hiện tại là những tùy chọn không liên quan đến ngữ cảnh hoặc phân tích cấu trúc của dữ liệu - chẳng hạn như xâu, quét antivirus hoặc chữ ký Yara. Trên thực tế, Michael Matonis đã tạo một công cụ gọi là page_brute (xem https://github.com/matonis/page_brute) để phân tích các tệp trang bằng cách chia chúng thành các khối có kích thước trang và quét từng khối bằng luật Yara. Bộ luật Yara mặc định đi kèm với công cụ có thể phát hiện các yêu cầu và phản hồi HTTP, tiêu đề thư SMTP, các lệnh FTP, v.v. Như luôn luôn, bạn có thể thêm vào các quy tắc mặc định hoặc tạo các tập luật riêng của riêng bạn để tùy chỉnh quét.

Hãy giả sử rằng bạn đang điều tra máy tính của một đối tượng bị buộc tội mua bán chất cấm trên mạng. Trình duyệt của đ ối tượng đã được cấu hình không lưu nội dung vào đĩa và không giữ một tệp lịch sử. Hơn nữa, hệ thống máy tính không hoạt động khi nó bị thu giữ, do đó tất cả những gì bạn có là một hình ảnh đĩa pháp y. Bằng cách xác định và trích xuất các tệp trang, bạn hy vọng tìm thấy một số bằng chứng liên quan đến hành vi nghi vấn của đối tượng. Bạn tạo một luật Yara sau để hỗ trợ việc tìm kiếm: 

```
rule drugs
{
    strings:
    $s0 = "silk road" nocase ascii wide
    $s1 = "silkroad" nocase ascii wide
    $s2 = "marijuana" nocase ascii wide
    $s3 = "bitcoin" nocase ascii wide
    $s4 = "mdma" nocase ascii wide

    condition:
    any of them
}
```

Lệnh sau đây sẽ thực hiện quét và phân tích tệp trang theo luật Yara có tên `drugs` mà bạn đã tạo trước đó:

```
$ python page_brute-BETA.py -r drugs.yar -f /media/external/pagefile.sys
[+] - YARA rule of File type provided for compilation: drugs.yar
..... Ruleset Compilation Successful.
[+] - PAGE_BRUTE running with the following options:
    [-] - PAGE_SIZE: 4096
    [-] - RULES TYPE: FILE
    [-] - RULE LOCATION: drugs.yar
    [-] - INVERSION SCAN: False
    [-] - WORKING DIR: PAGE_BRUTE-2014-03-24-12-49-57-RESULTS
    =================
 [snip]
    [!] FLAGGED BLOCK 58641: drugs
    [!] FLAGGED BLOCK 58642: drugs
    [!] FLAGGED BLOCK 58643: drugs
    [!] FLAGGED BLOCK 58646: drugs
    [!] FLAGGED BLOCK 58652: drugs
    [!] FLAGGED BLOCK 58663: drugs
    [!] FLAGGED BLOCK 58670: drugs
    [!] FLAGGED BLOCK 58684: drugs
    [!] FLAGGED BLOCK 58685: drugs
    [!] FLAGGED BLOCK 58686: drugs
    [!] FLAGGED BLOCK 58687: drugs
    [!] FLAGGED BLOCK 58688: drugs
    [!] FLAGGED BLOCK 58689: drugs
 [snip]
```

Các số tiếp theo sau thông báo "FLAGGED BLOCK" là chỉ mục của trang tương ứng trong tệp trang. Mỗi trang phù hợp với một chữ ký sẽ được trích xuất trong thư mục làm việc (PAGE_BRUTE-2014-03-24-12-49-57-RESULTS) được đặt tên theo chỉ mục. Sau đó, bạn có thể phân tích từng khối dữ liệu được trích xuất một cách riêng lẻ hoặc, để xem sơ bộ dữ liệu, bạn có thể chạy lệnh "strings" cho toàn bộ thư mục như sau:

```
$ cd PAGE_BRUTE-2014-03-24-12-49-57-RESULTS/drugs
$ strings * | less

https://bitcoin.org/font/ubuntu-bi-webfont.ttf
chrome://browser/content/urlbarBindings.xml#promobox
https://coinmkt.com/js/libs/autoNumeric.js?v=0.0.0.8
Bitcoin
Getting
https://bitcoin.org/font/ubuntu-ri-webfont.svg
https://bitcoin.org/font/ubuntu-ri-webfont.woff
wallet
Z N
http://howtobuybitcoins.info/img/miniflags/us.png
http://silkroaddrugs.org/silkroad-drugs-complete-step-by-step-guide/#c-3207
Location:
you want to also check out Silk Roads biggest competitor the click
silkroad6ownowfk.onion/categories/drugs-ecstasy/items
http://silkroaddrugs.org/silkroad-drugs-complete-step-by-step-guide/#c-2587

[snip]
```

Đáng kể là rằng dù tình nghi đã cố gắng giảm thiểu các dấu vết trong lịch sử duyệt web, bạn vẫn có thể tìm thấy chứng cứ về hoạt động đó bằng cách xem xét tệp trang. Điểm quan trọng là việc ẩn hoặc xóa dấu vết trong bộ nhớ nhiều khó khăn hơn so với trên đĩa cứng, đặc biệt khi hệ điều hành tự động ghi một phần bộ nhớ xuống đĩa cứng trong quá trình hoạt động thông thường như đẩy trang (paging).

>**GHI CHÚ**<br>
Người dùng chạy Windows 7 hoặc phiên bản sau đó có thể tùy chọn mã hóa các tệp trang hệ thống bằng Hệ thống Tệp Mã Hóa (EFS). Mặc dù nó bị tắt mặc định, bạn có thể gõ fsutil behavior query EncryptPagingFile tại cửa sổ lệnh quản trị để xem trạng thái hiện tại.<br>
Trên Linux, phân vùng swap thực chất là một phân vùng thay vì một tệp (bạn có thể liệt kê vị trí với lệnh cat /proc/swaps hoặc xem trong /etc/fstab). Tuy nhiên, bạn sẽ cần một hình ảnh đĩa để truy cập nội dung.  Đối với Mac OS X, swap được mã hóa theo mặc định từ phiên bản 10.7 trở đi. Bạn có thể liệt kê các tệp trong thư mục /var/vm hoặc truy vấn trạng thái bằng lệnh sysctl, như được thể hiện dưới đây:<br>
 ```
 $ ls -al /var/vm/*
 -rw------T 1 root wheel 2147483648 Mar 2 11:24 /var/vm/sleepimage
 -rw------- 1 root wheel 67108864 Apr 9 09:24 /var/vm/swapfile0
 -rw------- 1 root wheel 1073741824 Apr 28 22:28 /var/vm/swapfile1
 -rw------- 1 root wheel 1073741824 Apr 28 22:28 /var/vm/swapfile2
 $ sysctl vm.swapusage
 vm.swapusage: total = 2048.00M used = 1061.00M free = 987.00M (encrypted)
```
 
### Crash Dump Files

Nhiều hệ thống được cấu hình để ghi crash dump vào đĩa khi gặp lỗi BSOD (Blue Screen of Death). Do đó, bạn có thể muốn kiểm tra các tệp được tạo ra trong các lần crash trước đó mà có thể chưa được xóa. Mặc định, chúng được lưu tại %SystemRoot%\MEMORY.DMP; tuy nhiên, bạn có thể thay đổi đường dẫn này bằng cách chỉnh sửa khóa CurrentControlSet\Control\CrashControl trong hạt hive SYSTEM. Trong quá trình kiểm tra, bạn cũng nên kiểm tra các đường dẫn Windows Error Reporting (Dr. Watson) trong khóa Software\Microsoft\Windows\Windows Error Reporting của cả HKEY_CURRENT_USER và HKEY_LOCAL_MACHINE. Có thể bạn chỉ tìm thấy các user-mode dump (không phải là complete dump) ở đó, nhưng chúng vẫn có thể là một nguồn dữ liệu quan trọng về trạng thái tạm thời của hệ thống trong những giai đoạn không ổn định.

>**Lưu ý**<br> rằng nếu hệ thống mục tiêu đã bật dịch vụ Volume Shadow Copy Service (VSS), thì các nguồn dữ liệu tạm thời thay thế này có thể cũng có sẵn và chứa dữ liệu từ các khoảng thời gian trước đó. VSS cho phép tạo ra các bản sao "shadow" của các tệp và thư mục tại những điểm thời gian cụ thể, tạo cơ hội để khám phá thông tin từ các bản sao này.

## Tổng quan

Việc thu thập bộ nhớ vật lý một cách chính xác đòi hỏi kế hoạch cẩn thận, công cụ mạnh mẽ và tuân thủ các phương pháp tốt nhất. Hãy cân nhắc kỹ lưỡng các lựa chọn của bạn dựa trên môi trường và đặc thù của từng công việc trước khi chọn một kỹ thuật hoặc bộ công cụ phần mềm, bởi vì khả năng phân tích của bạn phụ thuộc vào việc thu thập thành công. Hãy nhớ rằng bằng chứng từ bộ nhớ thường được tìm thấy trên phương tiện không tạm thời và nó xuất hiện dưới nhiều "hình thức và kích thước" khác nhau, để nói một cách trường hợp. Hãy nhận thức về các định dạng khác nhau, cách chuyển đổi giữa các định dạng (nếu cần thiết) và những thách thức mà mỗi loại mẫu bộ nhớ đem lại.
