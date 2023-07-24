Volatility Framework là một tập hợp hoàn toàn mã nguồn mở các công cụ được triển khai bằng Python và được cấp phép theo GNU General Public License 2. Người phân tích sử dụng Volatility để trích xuất các tư liệu số từ các mẫu bộ nhớ tức thời (RAM). Vì Volatility là mã nguồn mở và miễn phí sử dụng, bạn có thể tải xuống và bắt đầu thực hiện phân tích nâng cao mà không phải trả bất kỳ chi phí nào. Hơn nữa, khi bạn muốn hiểu cách công cụ của mình hoạt động dưới nền, không có gì ngăn cản bạn khám phá và học hỏi trong khả năng tối đa.

Chương này cung cấp thông tin cơ bản để cài đặt Volatility, cấu hình môi trường và làm việc với các plugin phân tích. Nó cũng giới thiệu lợi ích của việc sử dụng Volatility và mô tả một số thành phần nội bộ giúp công cụ trở thành một thư viện đúng nghĩa. Hãy nhớ rằng phần mềm thay đổi theo thời gian. Do đó, khả năng, plugin, các yếu tố cài đặt và những yếu tố khác của framework có thể thay đổi trong tương lai.

## Why Volatiltiy?

Trước khi bạn bắt đầu sử dụng Volatility, bạn nên hiểu một số tính năng đặc biệt của nó. Như đã đề cập trước đó, Volatility không phải là ứng dụng pháp y trí nhớ duy nhất - nó được thiết kế để khác biệt. Dưới đây là một số lý do vì sao nhanh chóng trở thành công cụ lựa chọn của chúng tôi:

- **Khung giao diện đơn, hợp nhất.** Volatility phân tích bộ nhớ từ các hệ thống Windows 32 và 64 bit, Linux, Mac (và Android 32 bit). Thiết kế mô đun của Volatility cho phép nó dễ dàng hỗ trợ các hệ điều hành và kiến trúc mới khi chúng được phát hành.

- **Nó là mã nguồn mở GPLv2.** Điều này có nghĩa là bạn có thể đọc mã nguồn, học hỏi và mở rộng nó. Bằng cách tìm hiểu cách Volatility hoạt động, bạn sẽ trở thành một nhà phân tích hiệu quả hơn.

- **Nó được viết bằng Python.** Python là một ngôn ngữ đảo ngược kỹ thuật số đã được xác định với rất nhiều thư viện có thể tích hợp dễ dàng vào Volatility.

- Chạy trên hệ thống phân tích Windows, Linux hoặc Mac. Volatility chạy bất cứ nơi nào Python có thể được cài đặt - một giải pháp mới mẻ so với các công cụ phân tích bộ nhớ khác chỉ chạy trên Windows.

- **API mở rộng và kịch bản.** Volatility cho phép bạn vượt qua và tiếp tục đổi mới. Ví dụ, bạn có thể sử dụng Volatility để điều khiển hộp cát độc hại của bạn, thực hiện điều tra VM hoặc khám phá bộ nhớ kernel một cách tự động.

- **Bộ tính năng không giới hạn.** Các tính năng được tích hợp vào framework dựa trên nghệ thuật đảo ngược và nghiên cứu chuyên sâu. Volatility cung cấp chức năng mà thậm chí cả trình gỡ lỗi kernel của Microsoft cũng không hỗ trợ.

- **Đáp ứng đầy đủ các định dạng tập tin.** Volatility có thể phân tích các bản ghi gốc, bản sao sụp đổ, các tệp ngủ đông và các định dạng khác nhau khác (xem Chương 4). Bạn thậm chí có thể chuyển đổi qua lại giữa các định dạng này.

- **Thuật toán nhanh và hiệu quả.** Điều này cho phép bạn phân tích các bản sụp đổ RAM từ các hệ thống lớn trong một phần nhỏ thời gian mà không tốn nhiều bộ nhớ không cần thiết.

- **Cộng đồng nghiêm túc và mạnh mẽ.** Volatility đã kết hợp các đóng góp từ các công ty thương mại, cơ quan thực thi pháp luật và các cơ sở giáo dục trên toàn thế giới. Volatility cũng đang được xây dựng bởi một số tổ chức lớn như Google, National DoD Laboratories, DC3 và nhiều cửa hàng chống vi-rút và bảo mật.

- **Tập trung vào pháp y, phản ứng sự cố và mã độc.** Mặc dù Volatility và Windbg chia sẻ một số chức năng, chúng được thiết kế với mục đích chính khác nhau. Một số khía cạnh thường rất quan trọng đối với nhà phân tích pháp y nhưng không quan trọng đối với người gỡ lỗi một trình điều khiển kernel (chẳng hạn như lưu trữ chưa được phân bổ, tượng trưng gián tiếp và vân vân).

## What Volatility Is Not

Volatility có rất nhiều tính năng, nhưng cũng có một số danh mục mà nó không phù hợp. Các danh mục này bao gồm:

- Nó không phải là công cụ lấy dữ liệu bộ nhớ: Volatility không thể lấy dữ liệu bộ nhớ từ các hệ thống mục tiêu. Bạn cần lấy dữ liệu bộ nhớ bằng một trong những công cụ đã được đề cập trong Chương 4 và sau đó phân tích nó bằng Volatility. Một ngoại lệ là khi bạn kết nối với một máy tính trực tiếp qua giao diện Firewire và sử dụng plugin imagecopy của Volatility để sao chép RAM vào một tập tin. Trong trường hợp này, bạn thực chất đang lấy dữ liệu bộ nhớ.

- Nó không phải là một giao diện người dùng đồ họa (GUI): Volatility là một công cụ dòng lệnh và một thư viện Python mà bạn có thể nhập vào ứng dụng riêng của mình, nhưng nó không bao gồm một giao diện người dùng đồ họa. Trong quá khứ, các thành viên trong cộng đồng pháp y đã phát triển các GUI cho Volatility, nhưng hiện tại chúng không được hỗ trợ bởi đội phát triển chính thức.

- Nó không hoàn toàn không lỗi: Phân tích bộ nhớ có thể mỏng manh và nhạy cảm. Hỗ trợ cho các bản RAM từ nhiều phiên bản của hầu hết các hệ điều hành chính (thường đang chạy phần mềm bên thứ ba không rõ ràng) đi kèm với một chi phí: Điều này có thể dẫn đến các điều kiện phức tạp và các vấn đề khó tái sản xuất. Mặc dù đội phát triển cố gắng hết sức để không có lỗi, đôi khi điều đó không thể làm được do sự phức tạp của nhiệm vụ và đa dạng các tình huống gặp phải trong phân tích bộ nhớ thực tế.

- Nó không phải là một giải pháp đa dạng: Volatility là một công cụ mạnh mẽ và linh hoạt, nhưng có thể không bao gồm mọi khía cạnh của phân tích bộ nhớ cho từng trường hợp sử dụng cụ thể. Mặc dù cung cấp một tập hợp các tính năng rộng lớn, có thể có các tình huống nơi cần các công cụ chuyên dụng hoặc kịch bản tùy chỉnh để giải quyết những thách thức phân tích bộ nhớ cụ thể.

## Cài đặt 

Mỗi khi có phiên bản chính, Volatility được phân phối trong một số định dạng khác nhau, bao gồm một tệp thực thi độc lập trên Windows, một trình cài đặt module Python cho Windows và các gói mã nguồn trong các tập tin nén zip và gzip/tarball. Người dùng thường chọn định dạng nào để tải xuống dựa trên hệ điều hành máy chủ (mà họ dự định chạy Volatility) và các loại hoạt động mà họ dự định thực hiện với framework, chẳng hạn như đơn giản chỉ sử dụng nó để phân tích bộ nhớ hoặc cho phát triển và tích hợp với các công cụ bên ngoài. Bạn có thể tìm mô tả về các định dạng này trong các phần tiếp theo.

### Standalone Windows Executable

Sử dụng tệp thực thi là cách nhanh nhất và dễ dàng nhất để bắt đầu sử dụng Volatility trên máy tính phân tích Windows. Tệp thực thi bao gồm trình thông dịch Python 2.7 và tất cả các phụ thuộc của Volatility. Không cần cài đặt gì cả; chỉ cần khởi chạy tệp thực thi từ dấu nhắc lệnh, như ví dụ dưới đây:

```
C:\>volatility-2.4.exe --help
Volatility Foundation Volatility Framework 2.4
Usage: Volatility - A memory forensics analysis platform.
Options:
 	-h, --help 	list all available options and their default values.
			Default values may be set in the configuration file
			(/etc/volatilityrc)
[snip]
```
>**GHI CHÚ:** Hãy nhớ rằng Volatility không phải là một giao diện đồ họa (GUI), vì vậy không cố gắng nhấp đúp vào tệp thực thi từ Trình duyệt Windows (Windows Explorer).

Bạn có thể chạy tệp thực thi trực tiếp từ phương tiện di động như ổ đĩa USB, điều này biến Volatility thành một tiện ích di động mà bạn có thể mang theo khi thực hiện các điều tra từ xa (nơi bạn có thể không được phép mang theo máy tính xách tay của riêng mình). Trước đây, nhiều người đã viết các tập lệnh để tự động thu thập dữ liệu từ tệp thực thi độc lập bằng cách chuyển hướng đầu ra của nó thành các tệp văn bản.

### Windows Python Module Installer

Hãy chọn gói này nếu hệ thống phân tích ưa thích của bạn là Windows và bạn dự định xem xét hoặc sửa đổi mã nguồn Volatility - cho dù đó là để gỡ lỗi, cho mục đích giáo dục, hoặc để xây dựng các công cụ trên nền tảng của framework. Trong trường hợp này, bạn cũng phải cài đặt một trình thông dịch Python 2.7 và các phụ thuộc (danh sách đầy đủ được cung cấp sau trong chương).
Mặc định, các tệp nguồn được sao chép vào C:\PythonXX\Lib\site-packages\volatility (trong đó XX là phiên bản Python của bạn), và tệp chính vol.py được sao chép vào C:\PythonXX\Scripts. Do đó, bạn có thể sử dụng framework theo cách sau:

```
C:\>python C:\Python27\Scripts\vol.py --help
Volatility Foundation Volatility Framework 2.4
Usage: Volatility - A memory forensics analysis platform.
Options:
 -h, --help 	list all available options and their default values.
		Default values may be set in the configuration file
		(/etc/volatilityrc)
[snip]
```

### Source Code Packages

Đây là định dạng linh hoạt nhất - bạn có thể sử dụng nó trên Windows, Linux hoặc Mac, miễn là bạn đã có một trình thông dịch Python 2.7 hoạt động và các phụ thuộc cần thiết. Tuy nhiên, quan trọng lưu ý rằng bạn có hai tùy chọn để cài đặt mã nguồn trong các gói này sau khi giải nén. Mỗi phương pháp có điểm mạnh của riêng nó:

- **Phương pháp 1 (sử dụng setup.py):** Giải nén tệp lưu trữ và chạy python setup.py install. Bạn có thể cần quyền quản trị viên để hoàn tất cài đặt này. Phương pháp này sẽ sao chép các tệp vào các vị trí đúng trên đĩa của bạn để tên không gian Volatility có thể truy cập từ các kịch bản Python khác. Nếu bạn không dự định nhập Volatility như một thư viện cho các dự án phát triển, hãy xem xét việc sử dụng phương pháp 2 thay thế. Hạn chế của phương pháp 1 là khó nâng cấp hoặc gỡ cài đặt.

- **Phương pháp 2 (không sử dụng setup.py):** Giải nén tệp lưu trữ vào một thư mục theo sự lựa chọn của bạn. Khi bạn muốn sử dụng Volatility, chỉ cần chạy python /PATH/TO/vol.py. Phương pháp này thường được coi là sạch sẽ hơn vì không có tệp nào bị di chuyển ra ngoài thư mục được chọn của bạn, do đó nó cũng không yêu cầu quyền quản trị viên. Nếu bạn sử dụng phương pháp này, bạn có thể dễ dàng có nhiều phiên bản của Volatility có sẵn cùng một lúc bằng cách giữ chúng trong các thư mục riêng biệt. Để gỡ cài đặt, chỉ cần xóa thư mục.

### Development Branch (Code Repository)

Volatility hiện đang được lưu trữ trên Github. Đây là nơi bạn có thể tìm thấy mã nguồn mới nhất, bao gồm bất kỳ bản vá lỗi nào được xác định sau một phiên bản phát hành và cũng bao gồm bất kỳ tính năng mới quan trọng nào đang được thử nghiệm trước khi chúng đến người dùng chính thức. Do đó, mã này dành cho những người ít quan tâm đến phiên bản ổn định và hơn là quan tâm đến những gì sắp tới.

Mặc định, các tiện ích điều khiển mã nguồn được tích hợp vào các hệ điều hành Mac gần đây và chúng có sẵn thông qua trình quản lý gói của hầu hết các bản phân phối Linux. Ví dụ, bạn có thể nhập lệnh apt-get install git trên Debian/Ubuntu hoặc yum install git-core trên Centos/Red Hat. Đối với Windows, bạn có thể tải xuống msysGit (http://msysgit.github.io) hoặc sử dụng ứng dụng GitHub (https://windows.github.com). Sau khi đã cài đặt các công cụ, bạn có thể sao chép mã nguồn Volatility. Dưới đây là ví dụ về cách sao chép kho mã nguồn trên Linux hoặc Mac:

```
$ git clone https://github.com/volatilityfoundation/volatility.git
Cloning into 'volatility'...
remote: Counting objects: 10202, done.
remote: Compressing objects: 100% (2402/2402), done.
remote: Total 10202 (delta 7756), reused 10182 (delta 7736)
Receiving objects: 100% (10202/10202), 12.11 MiB | 343.00 KiB/s, done.
Resolving deltas: 100% (7756/7756), done.
Checking connectivity... done.
$ python volatility/vol.py --help
Volatility Foundation Volatility Framework 2.4
Usage: Volatility - A memory forensics analysis platform.
Options:
 -h, --help 	list all available options and their default values.
		 Default values may be set in the configuration file
		 (/etc/volatilityrc)
[snip]
After devel
```

Sau khi các nhà phát triển đã gửi các bản vá hoặc thay đổi mã nguồn, bạn có thể đồng bộ bằng cách đơn giản gõ lệnh git pull, thay vì chờ đợi phiên bản mới. Trong trường hợp này, bạn chỉ cần chuyển các tệp đã thay đổi kể từ lần cập nhật trước đó.


>**GHI CHÚ**

>	Nếu vấn đề hoặc hành vi cụ thể vẫn tiếp diễn sau khi cập nhật, có thể do các tệp .pyc (tệp Python đã biên dịch) "cũ" đã lưu lại. Để làm sạch các tệp .pyc, chuyển vào thư mục gốc của Volatility và nếu bạn đang sử dụng Linux hoặc Mac, gõ lệnh sau:

>	$ make clean

>	Nếu bạn đang sử dụng Windows, mở một cửa sổ PowerShell và gõ lệnh sau:

>	PS C:\volatility> Get-ChildItem -path . -Include '*.pyc' -Recurse | Remove-Item

### Dependencies

Như đã đề cập trước đó, nếu bạn đang làm việc với tệp thực thi độc lập của Windows, bạn không cần lo lắng về các yêu cầu phụ thuộc. Trong tất cả các trường hợp khác, bạn có thể cần cài đặt các gói bổ sung tùy thuộc vào các plugin Volatility mà bạn dự định chạy. Đa số các chức năng cốt lõi sẽ hoạt động mà không cần bất kỳ yêu cầu phụ thuộc nào (ngoại trừ trình thông dịch Python tiêu chuẩn). Danh sách sau đây xác định các mô-đun của bên thứ ba mà Volatility có thể sử dụng và các plugin cụ thể sử dụng chúng.

-	**Distorm3:** Một thư viện giải mã mạnh mẽ cho x86 / AMD64 (http://code.google.com/p/distorm). Các plugin Volatility apihooks, callbacks, impscan, volshell, linux_volshell, mac_volshell và linux_check_syscall phụ thuộc vào thư viện này.
-	**Yara:** Một công cụ xác định và phân loại phần mềm độc hại (http://code.google.com/p/yara-project). Các plugin Volatility yarascan, mac_yarascan và linux_yarascan phụ thuộc vào thư viện này.
-	**PyCrypto:** Bộ công cụ Mã hóa Python (https://www.dlitz.net/software/pycrypto). Các plugin Volatility lsadump và hashdump phụ thuộc vào thư viện này.
-	**PIL:** Thư viện ảnh Python (http://www.pythonware.com/products/pil). Plugin chụp ảnh màn hình phụ thuộc vào thư viện này.
-	**OpenPyxl:** Một thư viện Python để đọc và viết các tệp Excel (https://bitbucket.org/ericgazoni/openpyxl/wiki/Home). Plugin timeliner, khi sử dụng chế độ xuất xlsx, phụ thuộc vào plugin này.

Để biết chi tiết về cách cài đặt các yêu cầu phụ thuộc, bạn nên đọc tài liệu do người duy trì dự án cung cấp. Trong hầu hết các trường hợp, bạn chỉ cần chạy python setup.py install hoặc sử dụng trình quản lý gói. Nếu các yêu cầu phụ thuộc thay đổi, bạn luôn có thể tìm danh sách hiện tại trên wiki của Volatility.

## The Framework

Volatility Framework bao gồm một số hệ thống con hoạt động cùng nhau để cung cấp một bộ tính năng mạnh mẽ. Trong vài trang tiếp theo, chúng ta sẽ giới thiệu các thành phần chính một cách súc tích nhưng tổng quát. Mặc dù một số thành phần dành cho các nhà phát triển hơn là người dùng của framework, các thuật ngữ và khái niệm mà bạn học được ở đây sẽ rất hữu ích để hiểu cách các công cụ pháp yên tạo ra hoạt động, và chúng sẽ được đề cập trong toàn bộ phần còn lại của cuốn sách.

### VTypes

Đây là ngôn ngữ định nghĩa và phân tích cấu trúc của Volatility. Nếu bạn nhớ từ Chương 2, hầu hết các hệ điều hành và ứng dụng chạy trên các hệ điều hành đó đều được viết bằng C, với việc sử dụng rất nhiều cấu trúc dữ liệu để tổ chức các biến và thuộc tính liên quan. Vì Volatility được viết bằng Python, bạn cần một cách để biểu diễn các cấu trúc dữ liệu C trong các tệp nguồn Python. VTypes cho phép bạn làm chính điều đó. Bạn có thể định nghĩa các cấu trúc có tên thành viên, vị trí và kiểu dữ liệu khớp với các cấu trúc được sử dụng bởi hệ điều hành bạn đang phân tích, để khi bạn tìm thấy một phiên bản của cấu trúc đó trong bộ nhớ, Volatility biết cách xử lý dữ liệu cơ bản (ví dụ: như một số nguyên, chuỗi hoặc con trỏ). Trong ví dụ sắp tới, giả sử bạn đang làm việc với cấu trúc dữ liệu C như sau:

```C
struct process {
 int pid;
 int parent_pid;
 char name[10];
 char * command_line;
 void * ptv;
};
```

Cấu trúc này có năm thành viên: hai số nguyên, một mảng ký tự, một con trỏ đến một chuỗi, và một con trỏ void (sẽ được giải thích thêm trong phần "Overlays" sắp tới). Cấu trúc tương đương trong ngôn ngữ VType như sau:

```python
'process' : [ 26, {
 'pid' : [ 0, ['int']],
 'parent_pid' : [ 4, ['int']],
 'name' : [ 8, ['array', 10, ['char']]],
 'command_line' : [ 18, ['pointer', ['char']]],
 'ptv' : [ 22, ['pointer', ['void']]],
}]
```

Ở cái nhìn đầu tiên, cú pháp có thể có vẻ phức tạp hơn một chút, nhưng nó thực chất chỉ là một chuỗi các từ điển và danh sách trong Python. Tên của cấu trúc, "process", là khóa từ điển đầu tiên, và sau đó là kích thước tổng cộng của cấu trúc, là 26 byte. Tiếp theo là các thành viên của cấu trúc, cùng với các offset tương ứng từ cơ sở của cấu trúc và kiểu dữ liệu của chúng. Ví dụ, thành viên "name" có offset 8 và là một mảng gồm 10 ký tự. Khi bạn làm quen với cú pháp này và bắt đầu mô hình hóa các cấu trúc phức tạp hơn, bạn sẽ nhận ra ngôn ngữ VType hỗ trợ cũng nhiều kiểu dữ liệu như C cung cấp. Ví dụ, bạn có thể định nghĩa con trỏ, trường bit, các liệt kê và các liên hiệp (union), để kể một số.

Dưới đây là một ví dụ về cấu trúc "_EPROCESS" từ một hệ thống Windows 7 x64. Tổng kích thước của cấu trúc là 0x4d0 byte và bắt đầu bằng thành viên có tên là "Pcb", thực chất là một cấu trúc khác có kiểu "_KPROCESS". Lưu ý rằng ở offset 0x1f8 có ba thành viên khác nhau: "ExceptionPortData", "ExceptionPortValue" và "ExceptionPortState".

Đây là một ví dụ về cách mà một liên hiệp (union) xuất hiện trong ngôn ngữ VType. Như bạn có thể biết, kích thước của một liên hiệp được quy định bởi phần tử lớn nhất mà nó chứa, trong trường hợp này là 8 byte (một số nguyên không dấu 8 byte). Mặc dù "ExceptionPortState" nằm trong liên hiệp này, nó được định nghĩa là một trường bit, bao gồm ba bit ít quan trọng nhất (bắt đầu từ 0 và kết thúc ở bit thứ 3) của giá trị 8 byte.

```python
'_EPROCESS' : [ 0x4d0, {
 'Pcb' : [ 0x0, ['_KPROCESS']],
 'ProcessLock' : [ 0x160, ['_EX_PUSH_LOCK']],
 'CreateTime' : [ 0x168, ['_LARGE_INTEGER']],
 'ExitTime' : [ 0x170, ['_LARGE_INTEGER']],
 'RundownProtect' : [ 0x178, ['_EX_RUNDOWN_REF']],
 'UniqueProcessId' : [ 0x180, ['pointer64', ['void']]],
 'ActiveProcessLinks' : [ 0x188, ['_LIST_ENTRY']],
 'ProcessQuotaUsage' : [ 0x198, ['array', 2, ['unsigned long long']]],
 'ProcessQuotaPeak' : [ 0x1a8, ['array', 2, ['unsigned long long']]],
 'CommitCharge' : [ 0x1b8, ['unsigned long long']],
 'QuotaBlock' : [ 0x1c0, ['pointer64', ['_EPROCESS_QUOTA_BLOCK']]],
 'CpuQuotaBlock' : [ 0x1c8, ['pointer64', ['_PS_CPU_QUOTA_BLOCK']]],
 'PeakVirtualSize' : [ 0x1d0, ['unsigned long long']],
 'VirtualSize' : [ 0x1d8, ['unsigned long long']],
 'SessionProcessLinks' : [ 0x1e0, ['_LIST_ENTRY']],
 'DebugPort' : [ 0x1f0, ['pointer64', ['void']]],
 'ExceptionPortData' : [ 0x1f8, ['pointer64', ['void']]],
 'ExceptionPortValue' : [ 0x1f8, ['unsigned long long']],
 'ExceptionPortState' : [ 0x1f8, ['BitField', dict(start_bit = 0, end_bit = 3,
 	native_type='unsigned long long')]],
 'ObjectTable' : [ 0x200, ['pointer64', ['_HANDLE_TABLE']]],
[snip]
```

### Generating VTypes

Mặc dù việc làm việc với cú pháp VType có thể là một công việc khá đơn giản sau khi bạn đã làm quen, số lượng cấu trúc mà một hệ điều hành hoặc ứng dụng sử dụng làm cho việc tạo chúng bằng tay vô cùng không thực tế. Hơn nữa, các cấu trúc này có thể thay đổi một cách drastical sau mỗi phiên bản mới của hệ điều hành, hoặc thậm chí chỉ cần một bản vá hay cập nhật bảo mật mới. Để giải quyết những khó khăn này, Brendan Dolan-Gavitt (http://www.cc.gatech.edu/grads/b/brendan/) đã thiết kế một phương pháp để tự động tạo ra các VType từ các ký hiệu gỡ lỗi (PDB files) của Microsoft. Đặc biệt, anh ta đã viết một thư viện có tên là "pdbparse" (https://code.google.com/p/pdbparse) có thể chuyển đổi định dạng tập tin nhị phân PDB của Microsoft sang ngôn ngữ VType mở mà Volatility có thể hiểu.

Nhìn chung, bạn có thể tìm thấy các cấu trúc dữ liệu chính mà Volatility cần hỗ trợ cho một phiên bản Microsoft Windows trong các ký hiệu gỡ lỗi của mô-đun NT kernel (ntoskrnl.exe, ntkrnlpa.exe, và vân vân). Tuy nhiên, điều quan trọng cần lưu ý là Microsoft không tiết lộ tất cả các cấu trúc mà hệ điều hành của họ cần - chỉ có những cấu trúc có thể giúp đỡ việc gỡ lỗi được tiết lộ. Điều này không bao gồm hàng nghìn cấu trúc liên quan đến quản lý mật khẩu, cơ chế bảo mật và các tính năng khác của hệ thống mà tốt hơn là để lại không được tiết lộ để tránh những người tấn công sử dụng thông tin một cách xấu xa (ví dụ: để thiết kế các cuộc tấn công). Trong các trường hợp này, việc tạo ra các VType rơi vào vùng lãnh đạo thủ công: các nhà phát triển hoặc nhà nghiên cứu phải phân tích ngược các thành phần của hệ điều hành và tạo các định nghĩa cấu trúc riêng của họ để sử dụng với Volatility.

### Overlays

Overlays cho phép bạn chỉnh sửa hoặc vá lỗi các định nghĩa cấu trúc tự động tạo ra. Điều này là một khía cạnh quan trọng của phân tích bộ nhớ vì mã hệ điều hành thường sử dụng các con trỏ void (void *) nhiều trong cấu trúc của họ. Một con trỏ void là một con trỏ tới dữ liệu có kiểu không xác định hoặc tùy ý tại thời điểm phân bổ. Thật không may, các ký hiệu gỡ lỗi không chứa đủ thông tin để tự động suy ra các kiểu dữ liệu, và bạn có thể cần theo dõi, hoặc giải tham chiếu, các con trỏ này trong quá trình phân tích. Giả sử bạn đã xác định kiểu cho một thành viên cấu trúc cụ thể (thường thông qua phân tích ngược hoặc thử và lỗi), bạn có thể áp dụng một overlay, sẽ ghi đè định nghĩa VType tự động.

Trong phần trước, bạn có một cấu trúc có tên là "process" với thành viên "void * ptv". Nếu bạn biết rằng "ptv" thực sự là một con trỏ tới một cấu trúc "process" khác, bạn có thể tạo một overlay như sau:

```python
'process' : [ None, {
 'ptv' : [ None, ['pointer', ['process']]],
}
```

Lưu ý hai giá trị None, nằm ở các vị trí thông thường để lưu trữ kích thước của cấu trúc và các vị trí độ lệch thành viên. Giá trị None cho biết không có thay đổi nào được thực hiện đối với các phần đó của định nghĩa. Điều duy nhất mà overlay này thay đổi là kiểu của ptv từ void * thành process *. Sau khi áp dụng overlay như vậy, khi bạn truy cập vào ptv trong các plugin của Volatility, framework sẽ biết rằng bạn đang tham chiếu tới một cấu trúc "process".

Ngoài việc cung cấp khả năng làm cho định nghĩa cấu trúc chính xác hơn, overlays cũng hữu ích cho mục đích tiện ích và nhất quán. Ví dụ, Windows lưu trữ nhiều dấu thời gian của nó trong một cấu trúc được gọi là "_LARGE_INTEGER". Cấu trúc này chứa hai số nguyên 32-bit (một phần thấp và một phần cao), được kết hợp để tạo thành giá trị thời gian 64-bit. Giá trị 1325230153, khi dịch ra, có nghĩa là 2011-12-30 07:29:13 UTC+0000. Nhiều plugin của Volatility cần báo cáo dấu thời gian trong định dạng đọc được cho con người như vậy. Do đó, việc thay đổi toàn cục cấu trúc sử dụng _LARGE_INTEGER để lưu trữ giá trị dấu thời gian thành một kiểu đặc biệt có khả năng tự động dịch giá trị là một ý tưởng hợp lý.

### Objects and Classes (Đối tượng và lớp)

Một đối tượng Volatility (hoặc đơn giản gọi là đối tượng) là một phiên bản của một cấu trúc tồn tại tại một địa chỉ cụ thể trong không gian địa chỉ (AS). Bạn sẽ tìm hiểu về các AS sau này, vì vậy trong lúc này chỉ cần xem xét AS như một giao diện vào tệp dump bộ nhớ của bạn (tương tự như một xử lý tệp). Ví dụ, bạn có thể tạo một đối tượng bằng cách khởi tạo một cấu trúc _EPROCESS tại địa chỉ 0x80dc1a70 của tệp dump bộ nhớ. Sau khi tạo đối tượng, thường thông qua một cuộc gọi tới API obj.Object() của Volatility, bạn có thể truy cập bất kỳ thành viên nào của cấu trúc cơ bản để in ra giá trị của chúng, thực hiện các phép tính dựa trên giá trị của chúng, v.v.

Một lớp đối tượng cho phép bạn mở rộng chức năng của một đối tượng. Nói cách khác, bạn có thể gắn các phương thức hoặc thuộc tính vào một đối tượng, sau đó chúng trở nên có thể truy cập được cho tất cả các phiên bản của đối tượng đó. Điều này là một cách tuyệt vời để chia sẻ mã giữa các plugin và nó cũng tạo điều kiện thuận lợi cho việc sử dụng API trong framework. Ví dụ, nếu các plugin khác nhau tạo các đối tượng _EPROCESS và tất cả đều cần xác định xem quy trình có đáng ngờ dựa trên một số yếu tố, bạn có thể sử dụng một lớp đối tượng để thêm các logic như vậy. Một ví dụ đơn giản được thể hiện trong mã sau đây:

```python
import volatility.obj as obj

class _EPROCESS(obj.CType):
	"""An object class for _EPROCESS"""

	def is_suspicious(self):
		"""Determine if a process is suspicious
		based on several factors.

		:returns <bool>
		"""

		# check the process name
		if self.ImageFileName == "fakeav.exe":
			return True

		# check the process ID
		if self.UniqueProcessId == 0x31337:
			return True

		# check the process path
		if "temp" in str(self.Peb.ProcessParameters.ImagePathName):
			return True
		return False
```

Phương thức được hiển thị kiểm tra một số đặc điểm khác nhau của tiến trình - xem liệu tên của nó có phải là fakeav.exe, PID của nó có phải là 0x31337, hay đường dẫn đầy đủ trên đĩa cứng có chứa chuỗi "temp" hay không. Mặc dù đây chưa phải là một cuộc kiểm tra đầy đủ về tiến trình (bạn sẽ học thêm cách phân tích các tiến trình trong suốt cuốn sách), nhưng nó ít nhất cho thấy cho bạn ý tưởng chung về việc gắn API vào các đối tượng. Bằng cách gắn phương thức is_suspicious() vào đối tượng tiến trình (process object), bạn có thể gọi phương thức này bất cứ khi nào đối tượng tiến trình được sử dụng trong các plugin của Volatility. Điều này giúp bạn thực hiện kiểm tra tiến trình đáng ngờ dễ dàng và nhất quán trong các plugin khác nhau.

### Profiles

Một profile (hồ sơ) là một bộ sưu tập của các VTypes, overlays và object classes dành cho một phiên bản cụ thể của hệ điều hành và kiến trúc phần cứng (x86, x64, ARM). Ngoài các thành phần này, một profile còn bao gồm các thông tin sau:

- **Metadata (siêu dữ liệu):** Các dữ liệu như tên của hệ điều hành (ví dụ: "windows", "mac" hoặc "linux"), phiên bản kernel và số phiên bản xây dựng.
System call information (thông tin lệnh hệ thống): Chỉ số và tên của các lệnh hệ thống.
- **Constant values (giá trị hằng số):** Các biến toàn cục có thể được tìm thấy tại các địa chỉ cố định trong một số hệ điều hành.
- **Native types (kiểu dữ liệu cơ bản):** Các kiểu dữ liệu cấp thấp cho ngôn ngữ native (thường là C), bao gồm các kích thước cho các kiểu số nguyên, long, và cấu trúc dữ liệu khác.
- **System map (bản đồ hệ thống):** Các địa chỉ của các biến và hàm toàn cục quan trọng (chỉ có trên hệ điều hành Linux và Mac).

Mỗi profile có một tên duy nhất, thường được lấy từ tên hệ điều hành, phiên bản, bản vá dịch vụ và kiến trúc phần cứng của hệ điều hành đó. Ví dụ, Win7SP1x64 là tên của profile cho một hệ thống Windows 7 64-bit có bản vá dịch vụ 1. Tương tự, Win2012SP0x64 tương ứng với Windows Server 2012 64-bit. Phần "Sử dụng Volatility" sắp tới sẽ hướng dẫn bạn cách tạo danh sách đầy đủ các tên profile được hỗ trợ trong phiên bản Volatility của bạn.

Ngoài Windows, Volatility hỗ trợ các bản sao bộ nhớ từ Linux, Mac và Android. Thông tin chi tiết về cách xây dựng và tích hợp profile cho các hệ thống này có thể được tìm thấy trong Phần III "Linux Memory Forensics" và Phần IV "Mac Memory Forensics".

### Address Spaces (Không gian địa chỉ)

Một không gian địa chỉ (Address Space) là một giao diện cung cấp quyền truy cập linh hoạt và nhất quán vào dữ liệu trong bộ nhớ RAM, xử lý việc chuyển đổi địa chỉ ảo thành địa chỉ vật lý khi cần thiết và tự động tính toán sự khác biệt trong định dạng tập tin bộ nhớ được sao lưu (ví dụ: các tiêu đề chủ quan được thêm vào các tập tin crash dump của Microsoft hoặc các phương pháp nén được sử dụng trong các tập tin hibernation). Do đó, một không gian địa chỉ phải có kiến thức sâu về cấu trúc bộ nhớ và các phương pháp lưu trữ để tái tạo cùng "góc nhìn" của bộ nhớ mà các ứng dụng trên một máy chạy đang gặp phải.

### Virtual/Paged Address Spaces

Đúng! Các không gian địa chỉ ảo này cung cấp hỗ trợ để tái tạo bộ nhớ ảo. Chúng sử dụng nhiều thuật toán tương tự như các bộ xử lý Intel và AMD sử dụng cho việc dịch địa chỉ (xem Chapter 1), vì vậy có thể tìm thấy dữ liệu một cách ngoại tuyến, độc lập với hệ điều hành mục tiêu mà bộ nhớ đã được sao lưu từ đó. Một khía cạnh quan trọng liên quan đến không gian ảo là chúng chỉ xử lý bộ nhớ đã được cấp phát và có thể truy cập (tức là không nằm trong vùng bộ nhớ bị đẩy ra đĩa). Một không gian ảo chứa một phần con của bộ nhớ mà các chương trình trên hệ thống có thể "nhìn thấy" vào thời điểm sao lưu mà không gây ra page fault để đọc dữ liệu đã được đẩy ra đĩa trở lại RAM.

Loại không gian ảo/paged này có thể được chia thành không gian kernel và không gian process. Một không gian kernel cung cấp một cái nhìn về bộ nhớ đã được cấp phát và có thể truy cập bởi các trình điều khiển thiết bị và các module chạy ở chế độ kernel. Ngược lại, một không gian process cung cấp một cái nhìn về bộ nhớ từ góc nhìn của một tiến trình cụ thể. Như bạn đã học trong Chapter 1, vì các tiến trình có không gian địa chỉ riêng, mỗi tiến trình có cái nhìn riêng của bộ nhớ chế độ người dùng. Ánh xạ dữ liệu lại vào các tiến trình đã truy cập bộ nhớ trong bộ nhớ sao lưu là một kỹ thuật điều tra phổ biến.

>**Lưu ý:** ở Chapter 4, chúng ta đã đề cập đến việc Volatility hiện tại không hỗ trợ phân tích các tệp trang (page file). Tuy nhiên, trong tương lai gần, nó có thể được thực hiện như một không gian địa chỉ trừu tượng, đọc từ bộ nhớ vật lý và một hoặc nhiều tệp trang để cung cấp một cái nhìn đầy đủ về bộ nhớ.

### Physical Address Spaces

Các không gian địa chỉ vật lý chủ yếu xử lý các định dạng tệp mà các công cụ thu thập bộ nhớ sử dụng để lưu trữ bộ nhớ vật lý. Định dạng đơn giản nhất là một bản sao thô của bộ nhớ (không có tiêu đề độc quyền hoặc nén). Tuy nhiên, danh mục này cũng bao gồm các tệp crash dumps, tệp hibernation và bản chụp máy ảo (như lưu trạng thái đã lưu VMware và tệp core của Virtual Box) có thể thực sự lưu trữ siêu dữ liệu cụ thể cho nhà cung cấp cùng với bộ nhớ thực sự từ máy mục tiêu. Ngược lại với các không gian địa chỉ ảo, các không gian địa chỉ vật lý thường chứa các phạm vi bộ nhớ mà hệ điều hành đã đánh dấu là đã giải phóng hoặc hủy, cho phép bạn tìm thấy các hiện vật dư thừa (hoặc lịch sử) của các hành vi đã thực thi trong quá khứ.

### Address Space Stacking

Các loại ASs khác nhau mà bạn đã học trước đó thường được sử dụng cùng nhau để hỗ trợ lẫn nhau. Để giúp bạn hình dung quá trình tương tác này, hãy xem Hình 3-1. Đây là biểu đồ mô tả cách các ASs tự động xếp chồng lên nhau để hỗ trợ các loại tệp và kiến trúc phần cứng rộng lớn mà Volatility có thể làm việc.

![](https://github.com/HuyThang25/Image/blob/main/Screenshot%202023-07-24%20214808.png)

Trong ví dụ này, bạn đang làm việc với một bản sao dữ liệu sụp đổ từ một hệ thống Windows 64-bit. Khi một plugin của Volatility yêu cầu đọc một địa chỉ ảo, địa chỉ này được truyền vào AS AMD64, nơi thực hiện việc chuyển đổi địa chỉ ảo sang địa chỉ vật lý. Kết quả là một offset trong bộ nhớ vật lý nơi dữ liệu mong muốn có thể được tìm thấy. Nếu bạn có một bản sao dữ liệu gốc (kèm theo đệm), thì offset này giống như offset trong tập tin bản sao dữ liệu. Tuy nhiên, vì tập tin sao lưu bị sụp đổ chứa các tiêu đề khác nhau và cũng có thể chia các phạm vi bộ nhớ không liên tục thành "runs", nên một offset trong bộ nhớ vật lý không giống như một offset trong tập tin sao lưu bị sụp đổ. Điều này là lúc AS tập tin sao lưu bị sụp đổ vào vai trò. AS tập tin sao lưu bị sụp đổ biết kích thước của các tiêu đề của nó và có thể phân tích thông tin liên quan đến nơi lưu trữ các phạm vi bộ nhớ, cho phép nó ghép lại khung nhìn gốc của bộ nhớ vật lý. Do đó, AS tập tin sao lưu bị sụp đổ có thể dễ dàng tìm thấy offset yêu cầu trong tập tin cơ sở.

### Plugin System

Các plugin cho phép bạn mở rộng khung làm việc Volatility hiện có. Ví dụ, một plugin không gian địa chỉ có thể giới thiệu hỗ trợ cho các hệ điều hành chạy trên các bộ vi xử lý CPU mới. Các plugin phân tích được viết để tìm và phân tích các thành phần cụ thể của hệ điều hành, ứng dụng người dùng, hoặc thậm chí là các mẫu mã độc hại. Bất kể loại plugin bạn viết, có các API và mẫu mã được cung cấp bởi khung làm việc giúp bạn dễ dàng thiết kế và tích hợp ý tưởng của mình.

Để phát triển một plugin phân tích, bạn tạo một lớp Python thừa kế từ commands.Command và ghi đè một số phương thức cơ bản. Cụ thể, hai chức năng chính bạn sẽ tùy chỉnh là calculate (nơi phần lớn công việc của plugin của bạn được thực hiện) và render_text (nơi bạn định dạng kết quả cho đầu ra dựa trên văn bản). Nói tóm lại, mã trong calculate chịu trách nhiệm tìm và phân tích các đối tượng trong bộ nhớ. Kết quả của việc tính toán được chuyển đến render_text để hiển thị trên terminal. Đoạn mã dưới đây thể hiện plugin phân tích tối thiểu, chỉ in tên của tất cả quá trình trên hệ thống. Tên của plugin này được lấy từ tên của lớp (ExamplePlugin).

```python
import volatility.utils as utils
import volatility.commands as commands
import volatility.win32.tasks as tasks

class ExamplePlugin(commands.Command):
    """This is an example plugin"""
    
    def calculate(self):
        """This method performs the work"""
        
        addr_space = utils.load_as(self._config)
        for proc in tasks.pslist(addr_space):
            yield proc
    
    def render_text(self, outfd, data):
        """This method formats output to the terminal.
        
        :param outfd    | <file>
        data            | <generator>
        
        """
        for proc in data:
            outfd.write("Process: {0}\n".format(proc.ImageFileName))
```

Để thực thi plugin này, di chuyển tệp Python chứa mã này vào thư mục volatility/plugins (hoặc bất kỳ thư mục con nào trong đó), và nó sẽ tự động được nhận dạng và đăng ký bởi hệ thống plugin.

>**GHI CHÚ** <br><br>
Bạn có thể lưu trữ các tệp plugin của mình trong một thư mục bên ngoài hoặc thậm chí trong nhiều thư mục bên ngoài. Tuy nhiên, bạn sẽ cần thông báo cho Volatility biết cách tìm chúng trong một hoặc nhiều thư mục bằng cách phân tách chúng bằng dấu hai chấm (:) trên UNIX (DIR1:DIR2:DIR3) hoặc dấu chấm phẩy (;) trên Windows (DIR1;DIR2;DIR3). Thay vì các thư mục, bạn cũng có thể truyền các đường dẫn tới các tệp zip chứa các plugin.<br><br>
Một lưu ý là tùy chọn --plugins phải được chỉ định ngay sau vol.py khi bạn sử dụng nó. Ví dụ, lệnh đầu tiên sẽ hoạt động, nhưng lệnh thứ hai sẽ không:<br><br>
**$ python vol.py --plugins=DIR pslist <br>
$ python vol.py pslist --plugins=DIR**

### Core Plugins

Hệ thống Volatility Framework bao gồm hơn 200 plugin phân tích. Vì lí do ngắn gọn, chúng tôi sẽ không liệt kê tất cả ở đây, nhưng điều quan trọng là bạn nên xem qua tên và mô tả để biết những khả năng nào bạn muốn khám phá khi bạn bắt đầu đi sâu vào nghiên cứu. Bạn cũng có thể liệt kê các plugin có sẵn trong phiên bản Volatility của bạn bất cứ lúc nào bằng cách sử dụng lệnh vol.py --info.

## Sử dụng Volatility

Bây giờ bạn đã biết một chút về Volatility, bạn có thể bắt đầu khám phá cách sử dụng dòng lệnh. Trong phần này, bạn sẽ tìm hiểu cấu trúc cơ bản của một lệnh, cách hiển thị trợ giúp và cách điều chỉnh môi trường của bạn.

### Những câu lệnh cơ bản

Lệnh cơ bản nhất của Volatility được xây dựng như sau. Bạn thực thi tập lệnh chính Python (vol.py) và sau đó truyền đường dẫn đến tệp bộ nhớ chứa bản ghi nhớ, tên của profile và tên của plugin muốn thực thi (và tùy chọn là các tham số riêng của plugin):

```shell
$ python vol.py –f <FILENAME> --profile=<PROFILE> <PLUGIN> [ARGS]
```

Đây là một ví dụ cụ thể: 

```shell
$ python vol.py –f /home/mike/memory.dmp --profile=Win7SP1x64 pslist
```

Nếu bạn sử dụng phiên bản thực thi Windows độc lập, cú pháp sẽ như sau:

```powershell
C:\>volatlity-2.4.exe –f C:\Users\Mike\memory.dmp --profile=Win7SP1x64 pslist
```

Có một số ngoại lệ đối với lệnh cơ bản như đã thể hiện, ví dụ khi gọi Volatility với các tùy chọn -h/--help (để xem toàn bộ tùy chọn), --info (để xem tất cả các không gian địa chỉ (ASs), plugin và profile có sẵn), hoặc khi bạn chưa biết sử dụng profile nào.

### Displaying Help



```
$ python vol.py --help
Volatility Foundation Volatility Framework 2.4
Usage: Volatility - A memory forensics analysis platform.
Options:
 -h, --help 	list all available options and their default values.
 		Default values may be set in the configuration file
 		(/etc/volatilityrc)
 --conf-file=/Users/michaelligh/.volatilityrc
 		User based configuration file
 -d, --debug 	Debug volatility
 --plugins=PLUGINS 	Additional plugin directories to use (colon separated)
 --info 	Print information about all registered objects
 --cache-directory=/Users/michaelligh/.cache/volatility
		Directory where cache files are stored
 --cache 	Use caching
 --tz=TZ 	Sets the timezone for displaying timestamps
 -f FILENAME, --filename=FILENAME
 		Filename to use when opening an image
 --profile=WinXPSP2x86
 		Name of the profile to load
 -l LOCATION, --location=LOCATION
 		A URN location from which to load an address space
 -w, --write	 Enable write support
 --dtb=DTB 	DTB Address
 --output=text 	Output in this format (format support is module specific)
 --output-file=OUTPUT_FILE
 		write output in this file
 -v, --verbose 	Verbose information
 --shift=SHIFT 	Mac KASLR shift address
 -g KDBG, --kdbg=KDBG 	Specify a specific KDBG virtual address
 -k KPCR, --kpcr=KPCR 	Specify a specific KPCR address

     Supported Plugin Commands:
	 apihooks Detect API hooks in process and kernel memory
	 atoms Print session and window station atom tables
	 atomscan Pool scanner for _RTL_ATOM_TABLE
 [snip]
```

Các plugin của Volatility cũng có thể đăng ký các tùy chọn riêng của chúng, mà bạn có thể xem bằng cách cung cấp cả tên plugin và -h/--help. Lệnh sau đây hiển thị các tùy chọn cho plugin `handles`:

```
Volatility Foundation Volatility Framework 2.4
Usage: Volatility - A memory forensics analysis platform.
[snip]
 -o OFFSET, --offset=OFFSET
		EPROCESS offset (in hex) in the physical address space
 -p PID, --pid=PID 	Operate on these Process IDs (comma-separated)
 -P, --physical-offset
 		Physical Offset
 -t OBJECT_TYPE, --object-type=OBJECT_TYPE
 		Show these object types (comma-separated)
 -s, --silent 	Suppress less meaningful results
---------------------------------
Module Handles
---------------------------------
Print list of open handles for each process

```

### Chọn Profile

Một trong những tùy chọn từ menu trợ giúp toàn cầu mà bạn sử dụng nhiều nhất là --profile. Tùy chọn này thông báo cho Volatility biết hệ thống của bạn đã tải bộ nhớ từ đâu, để nó biết sử dụng các cấu trúc dữ liệu, thuật toán và biểu tượng nào. Một hồ sơ mặc định là WinXPSP2x86 được đặt làm mặc định, vì vậy nếu bạn phân tích một bản ghi nhớ Windows XP SP2 x86, bạn không cần cung cấp --profile. Nếu không, bạn phải xác định tên hồ sơ chính xác.

Trong một số trường hợp, bạn không biết trước hồ sơ; ví dụ, một nhà điều tra khác có thể tải bộ nhớ và không cho bạn biết phiên bản hệ điều hành. Volatility bao gồm hai plugin có thể giúp bạn xác định hồ sơ phù hợp nếu điều đó xảy ra. Plugin đầu tiên là imageinfo, cung cấp tóm tắt cấp cao về bộ nhớ mẫu bạn đang phân tích. Dưới đây là một ví dụ về lệnh này:

```
$ python vol.py -f memory.raw imageinfo
Volatility Foundation Volatility Framework 2.4
Determining profile based on KDBG search...

         Suggested Profile(s) : Win7SP0x64, Win7SP1x64, Win2008R2SP0x64
                    AS Layer1 : AMD64PagedMemory (Kernel AS)
                    AS Layer2 : FileAddressSpace (/Users/Michael/Desktop/memory.raw)
                     PAE type : PAE
                          DTB : 0x187000L
                         KDBG : 0xf80002803070
         Number of Processors : 1
    Image Type (Service Pack) : 0
               KPCR for CPU 0 : 0xfffff80002804d00L
            KUSER_SHARED_DATA : 0xfffff78000000000L
          Image date and time : 2012-02-22 11:29:02 UTC+0000
    Image local date and time : 2012-02-22 03:29:02 -0800
```

>**Lưu ý:**<br> Plugin imageinfo cũng cho bạn biết ngày và giờ khi mẫu bộ nhớ được thu thập; số lượng CPU; một số đặc điểm của AS, chẳng hạn như việc PAE được kích hoạt; và giá trị bảng thư mục gốc (DTB) được sử dụng cho việc dịch địa chỉ.



```
$ python vol.py -f memory.raw kdbgscan
Volatility Foundation Volatility Framework 2.4
**************************************************
Offset (V)                      : 0xf80002803070
Offset (P)                      : 0x2803070
KDBG owner tag check            : True
Profile suggestion (KDBGHeader) : Win7SP0x64
Version64                       : 0xf80002803030 (Major: 15, Minor: 7600)
Service Pack (CmNtCSDVersion)   : 0
Build string (NtBuildLab)       : 7600.16385.amd64fre.win7_rtm.090
PsActiveProcessHead             : 0xfffff80002839b30 (32 processes)
PsLoadedModuleList              : 0xfffff80002857e50 (133 modules)
KernelBase                      : 0xfffff8000261a000 (Matches MZ: True)
Major (OptionalHeader)          : 6
Minor (OptionalHeader)          : 1
KPCR                            : 0xfffff80002804d00 (CPU 0)
```

Như bạn có thể thấy, tất cả dấu hiệu đều cho thấy tệp memory.raw này có nguồn gốc từ máy Windows 7 64-bit Service Pack 0. Do đó, bây giờ bạn có thể cung cấp --profile=Win7SP0x64 khi chạy các plugin khác. Bởi vì khối dữ liệu gỡ lỗi cũng chứa các con trỏ đến danh sách tiến trình và danh sách các module được tải, plugin kdbgscan có thể cho bạn biết có bao nhiêu mục trong mỗi danh sách. Trong trường hợp này, có 32 tiến trình và 133 module. Những giá trị này có thể giúp bạn phân biệt giữa các trường hợp mà một khối dữ liệu gỡ lỗi "cũ" được tìm thấy (được thảo luận sau trong phần vấn đề).

>**GHI CHÚ**<br>
Cả hai plugin imageinfo và kdbgscan đều chỉ hỗ trợ trên Windows. Như bạn sẽ tìm hiểu trong Phần III (Linux) và Phần IV (Mac) của cuốn sách, có các cách khác để xác định profile đúng cho các hệ điều hành khác.

### Issues with Profile Selection

Volatility quét _KDDEBUGGER_DATA64 bằng cách tìm các giá trị hằng số được nhúng trong cấu trúc, bao gồm chữ ký 4 byte cứng của KDBG. Những chữ ký này không quan trọng đối với việc hoạt động chính xác của hệ điều hành, do đó mã độc chạy trong kernel có thể ghi đè lên chúng trong nỗ lực để làm cho các công cụ dựa vào việc tìm thấy chữ ký bị nhầm lẫn. Các công cụ phân tích bộ nhớ khác xem xét ngày/giờ biên dịch trong tiêu đề PE của mô-đun NT kernel, điều này cũng không quan trọng và có thể bị can thiệp độc hại. Đó là lý do tại sao Volatility cung cấp tùy chọn --profile: Nếu việc xác định tự động của hệ điều hành thất bại (do các sửa đổi cố ý hoặc vô tình), bạn có thể ghi đè thủ công.

>**CẢNH BÁO**<br>
Tại hội nghị Blackhat năm 2012, Takahiro Haruyama và Hiroshi Suzuki đã trình bày One-byte Modification for Breaking Memory Forensic Analysis (Xem tại https://media.blackhat.com/bh-eu-12/Haruyama/bh-eu-12-Haruyama-Memory_Forensic-Slides.pdf). Kỹ thuật chống pháp y tế này liên quan đến việc thay đổi một byte (được biết đến là một yếu tố chấm dứt) của chữ ký KDBG để không được nhận ra đúng cách bởi các công cụ phân tích bộ nhớ tự động.

Bên cạnh đó, trong một số trường hợp, có thể có nhiều hơn một cấu trúc _KDDEBUGGER_DATA64 tồn tại trong bộ nhớ vật lý. Điều này có thể xảy ra nếu hệ thống mục tiêu chưa khởi động lại kể từ khi áp dụng một bản vá nhanh đã cập nhật một số tệp kernel, hoặc nếu máy khởi động lại quá nhanh đến mức không toàn bộ nội dung của RAM được xóa (do đó bạn gặp các cấu trúc dư thừa, tương tự như cách tấn công băng thông lạnh hoạt động). Tìm nhiều cấu trúc dữ liệu bộ gỡ lỗi có thể dẫn đến liệt kê không chính xác các quy trình và mô-đun, do đó quan trọng để nhận thức về khả năng này.

Lưu ý trong lệnh sau đây rằng kdbgscan nhận hai cấu trúc:
- Một cấu trúc không hợp lệ (với 0 quy trình và 0 mô-đun) được tìm thấy tại địa chỉ `0xf80001172cb0`
- Một cấu trúc hợp lệ (với 37 quy trình và 116 mô-đun) được tìm thấy tại địa chỉ `0xf80001175cf0`

```
$ python vol.py -f Win2K3SP2x64.vmem --profile=Win2003SP2x64 kdbgscan
Volatility Foundation Volatility Framework 2.4
**************************************************
Instantiating KDBG using: Kernel AS Win2003SP2x64 (5.2.3791 64bit)
Offset (V)                      : 0xf80001172cb0
Offset (P)                      : 0x1172cb0
KDBG owner tag check            : True
Profile suggestion (KDBGHeader) : Win2003SP2x64
Version64                       : 0xf80001172c70 (Major: 15, Minor: 3790)
Service Pack (CmNtCSDVersion)   : 0
Build string (NtBuildLab)       : T?
PsActiveProcessHead             : 0xfffff800011947f0 (0 processes)
PsLoadedModuleList              : 0xfffff80001197ac0 (0 modules)
KernelBase                      : 0xfffff80001000000 (Matches MZ: True)
Major (OptionalHeader)          : 5
Minor (OptionalHeader)          : 2

**************************************************
Instantiating KDBG using: Kernel AS Win2003SP2x64 (5.2.3791 64bit)
Offset (V)                      : 0xf80001175cf0
Offset (P)                      : 0x1175cf0
KDBG owner tag check            : True
Profile suggestion (KDBGHeader) : Win2003SP2x64
Version64                       : 0xf80001175cb0 (Major: 15, Minor: 3790)
Service Pack (CmNtCSDVersion)   : 2
Build string (NtBuildLab)       : 3790.srv03_sp2_rtm.070216-1710
PsActiveProcessHead             : 0xfffff800011977f0 (37 processes)
PsLoadedModuleList              : 0xfffff8000119aae0 (116 modules)
KernelBase                      : 0xfffff80001000000 (Matches MZ: True)
Major (OptionalHeader)          : 5
Minor (OptionalHeader)          : 2
KPCR                            : 0xfffff80001177000 (CPU 0)
```

Như đã đề cập trước đó, nhiều plugin của Volatility phụ thuộc vào việc tìm kiếm khối dữ liệu bộ gỡ lỗi và sau đó điều hướng các danh sách quy trình và mô-đun đang hoạt động. Mặc định, những plugin này chấp nhận cấu trúc bộ gỡ lỗi đầu tiên mà chúng tìm thấy qua quét; tuy nhiên, như bạn vừa thấy, lựa chọn đầu tiên không phải lúc nào cũng là lựa chọn tốt nhất. Trong những trường hợp này, sau khi xác minh thủ công giá trị chính xác hơn bằng kdbgscan, bạn có thể đặt tùy chọn toàn cục --kdbg=0xf80001175cf0. Điều này không chỉ đảm bảo rằng Volatility sử dụng giá trị chính xác mà còn giúp bạn tiết kiệm thời gian khi thực thi nhiều lệnh vì không còn cần thực hiện quét nữa.

### Alternatives to Command-Line Options

Nếu bạn chuẩn bị tiến hành một công việc dài hạn và không muốn gõ đường dẫn đến tệp nhớ, tên hồ sơ, và các tùy chọn khác mỗi lần, bạn có thể thực hiện một vài cách tiện lợi. Volatility có thể tìm kiếm các biến môi trường và tệp cấu hình (theo thứ tự đó) để tìm các tùy chọn nếu chúng không được cung cấp trên dòng lệnh. Điều này có nghĩa là bạn có thể đặt các tùy chọn một lần và sử dụng lại chúng trong suốt quá trình điều tra, điều này giúp tiết kiệm thời gian rất nhiều.

Trên hệ thống phân tích Linux hoặc Mac, bạn có thể đặt các tùy chọn bằng cách xuất chúng trong shell của bạn, như dưới đây:

```shell
$ export VOLATILITY_PROFILE=Win7SP0x86
$ export VOLATILITY_LOCATION=file:///tmp/myimage.img
$ python vol.py pslist
$ python vol.py files
```

Một số điểm quan trọng mà bạn nên lưu ý khi đặt các tùy chọn theo cách này:
- Quy ước đặt tên: Bạn nên đặt tên cho các biến môi trường của mình theo tên tùy chọn gốc, nhưng thêm tiền tố VOLATILITY. Ví dụ, thay vì đặt --profile trên dòng lệnh, biến môi trường tương đương là VOLATILITY_PROFILE.
- Vị trí so với tên tệp: Khi đặt đường dẫn đến tệp nhớ, sử dụng VOLATILITY_LOCATION và hãy chắc chắn thêm tiền tố file:/// trước đường dẫn (ngay cả khi bạn đang thực hiện phân tích trên máy tính Windows).
- Sự kiên định: Các biến môi trường bạn đặt theo cách này chỉ có hiệu lực trong khi shell hiện tại của bạn đang mở. Khi bạn đóng nó hoặc mở một shell khác, bạn phải thiết lập lại các biến này. Một cách khác, bạn có thể thêm các biến này vào tệp ~/.bashrc hoặc /etc/profile (điều này có thể khác nhau tùy thuộc vào hệ điều hành của bạn) để chúng khởi tạo cùng với mỗi shell.

Sử dụng tệp cấu hình cũng không khó khăn gì. Mặc định, Volatility tìm kiếm tệp có tên .volatilityrc trong thư mục hiện tại hoặc ~/.volatilityrc (trong thư mục người dùng của bạn), hoặc trong một đường dẫn được chỉ định bằng tùy chọn --conf-file. Tệp cấu hình sử dụng định dạng INI tiêu chuẩn. Hãy đảm bảo rằng bạn có một phần tên DEFAULT, tiếp theo là các tùy chọn bạn muốn đặt. Trong trường hợp này, các tùy chọn không có tiền tố VOLATILITY.

```
[DEFAULT]
PROFILE=Win7SP0x86
LOCATION=file:///tmp/myimage.img
```

Sau khi bạn đã đặt các tùy chọn bằng cách sử dụng một trong hai phương pháp mô tả trong phần này, bạn có thể sử dụng các plugin của Volatility như `python vol.py pslist` và bạn không cần phải lo lắng về việc gõ các tùy chọn còn lại.

### Controlling Plugin Output

Mặc định, các plugin tạo kết quả dưới dạng văn bản và ghi chúng ra đầu ra tiêu chuẩn, thường là cửa sổ terminal của bạn. Tuy nhiên, một số plugin có thể tạo ra kết quả rất chi tiết, làm cho việc quét lại kết quả trở nên khó khăn - đặc biệt nếu chúng cuộn nhanh và cửa sổ terminal của bạn chỉ lưu trữ số lượng dòng cuối cùng nhất định. Do đó, việc chuyển hướng kết quả ra một tập tin văn bản để bạn có thể xem lại sau là một thực hành phổ biến. Có một số cách để làm điều này, như được thể hiện trong các lệnh sau:

```shell
$ python vol.py pslist > pslist.txt
$ python vol.py pslist --output-file=pslist.txt
```

Lệnh đầu tiên đơn giản chỉ sử dụng khả năng chuyển hướng của terminal, và lệnh thứ hai sử dụng tùy chọn --output-file của Volatility. Cả hai lệnh đều tạo ra một tệp văn bản chứa kết quả từ plugin pslist. Lý do duy nhất chúng tôi đề cập đến cả hai kỹ thuật này là vì nó dẫn đến cuộc thảo luận về việc yêu cầu đầu ra dưới một định dạng khác với định dạng dựa trên văn bản. Bạn có thể làm điều này cho từng plugin riêng lẻ, miễn là các plugin hỗ trợ các chế độ đầu ra khác. Ví dụ, plugin mftparser có thể hiển thị kết quả dưới dạng body format phổ biến (http://wiki.sleuthkit.org/index.php?title=Body_file) nếu bạn gọi nó với tùy chọn --output=body. Một ví dụ khác là plugin impscan, bạn có thể gọi với tùy chọn --output=idc để tạo ra đầu ra dưới dạng ngôn ngữ kịch bản của IDA Pro. Trong tương lai, các plugin có thể hỗ trợ đầu ra dữ liệu dưới các định dạng JSON, XML, CSV, HTML hoặc các định dạng khác.

## Tổng quan

Volatility Frameworks là kết quả của nhiều năm nghiên cứu và phát triển từ hàng chục, nếu không phải hàng trăm, thành viên trong cộng đồng pháp lý mã nguồn mở. Khung cung cấp khả năng giải quyết các tội phạm số phức tạp liên quan đến phần mềm độc hại, các tác nhân đe dọa thông minh và các tội phạm phổ thông. Bây giờ bạn đã biết cách cài đặt và cấu hình Volatility, bạn đã sẵn sàng bắt đầu thu thập mẫu bộ nhớ và phân tích chúng. Các kỹ thuật phân tích nâng cao và triển khai được trình bày trong phần sau của cuốn sách sẽ cho phép bạn sử dụng phần mềm đến tối đa tiềm năng của nó. Nếu cần thiết, bạn cũng đã làm quen với các yếu tố nội tại của Volatility (như hồ sơ, plugin và không gian địa chỉ), giúp bạn phát triển các phần mở rộng và tùy chỉnh riêng của mình.
