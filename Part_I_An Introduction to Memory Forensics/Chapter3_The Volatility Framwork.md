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

