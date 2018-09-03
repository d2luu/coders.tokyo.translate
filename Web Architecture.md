# Kiến trúc web

#### *Những khái niệm kiến trúc cơ bản nên biết khi bắt đầu sự nghiệp như một nhà phát triển web*

![](https://cdn-images-1.medium.com/max/1600/1*K6M-x-6e39jMq_c-2xqZIQ.png)

Sơ đồ trên thể hiện khá tốt về kiến trúc web của chúng tôi tại [Storyblocks](https://www.storyblocks.com/). Nếu bạn không phải là một nhà phát triển web có kinh nghiệm, bạn sẽ thấy nó khá phức tạp. Đừng lo lắng, những nội dung dưới đây sẽ làm cho nó dễ tiếp cận hơn khi đi vào chi tiết của mỗi thành phần.

Quá trình 1 người dùng truy cập vào trang web của chúng tôi diễn ra như sau:

> Một người dùng tìm kiếm trên Google cho cụm từ "Sương mù và những tia nắng đẹp trong khu rừng". [Kết qủa đầu tiên](https://www.graphicstock.com/stock-image/strong-beautiful-fog-and-sunbeams-in-the-forest-246703) đến từ Storyblocks - trang về hình ảnh và vec-tơ hàng đầu của chúng tôi. Khi ấn vào kết quả đó, trình duyệt sẽ được điều hướng đến trang ảnh chi tiết. Ở bên dưới, trình duyệt sẽ gửi 1 yêu cầu đến máy chủ DNS, tìm cách liên lạc và gửi yêu cầu đó tới Storyblocks.
> 
> Yêu cầu sau đó được gửi tới cân bằng tải (Load balancer), nơi mà sẽ chọn ngẫu nhiên 1 trong 10 máy chủ mà chúng tôi đang dùng để chạy trang web và xử lý yêu cầu đó. Máy chủ web tìm kiếm thông tin về bức ảnh từ dịch vụ caching (Caching service) và lấy dữ liệu còn lại từ cơ sở dữ liệu (Database). Chú ý rằng màu sắc của hình ảnh vẫn chưa được tính toán vào lúc này, chúng tôi đưa việc tính toán màu sắc vào hàng đợi (Job queue), nơi mà máy chủ (Job servers) sẽ thực hiện 1 cách bất đồng bộ và cập nhật kết quả thích hợp vào cơ sở dữ liệu.
> 
> Tiếp theo, chúng tôi thử tìm những bức ảnh tương tự bằng cách gửi yêu cầu đến dịch vụ tìm kiếm toàn văn bản (Full text search service) sử dụng đầu vào là tiêu đề của bức ảnh. Người dùng truy cập vào Storyblocks với tư cách là thành viên, vì thế chúng tôi sẽ tìm kiếm thông tin tài khoản của họ từ dịch vụ tài khoản (Account service).  Cuối cùng, chúng tôi kích hoạt một sự kiện xem trang cho Data Firehose để được ghi lại trên hệ thống lưu trữ đám mây (Cloud Storage), sau đó tải vào kho dữ liệu (Data warehouse), thứ mà sẽ được các nhà phân tích sử dụng để trả lời các vấn đề nghiệp vụ.
> 
> Bây giờ máy chủ gửi trang HTML về cho trình duyệt của người dùng để hiển thị. Trang web bao gồm Javascript và CSS được lấy từ hệ thống lưu trữ đám mây của chúng tôi, nơi được kết nối với mạng lưới chia sẻ nội dung (CDN). Cuối cùng, trang web sẽ được hiển thị trên trình duyệt của người dùng. 

Tiếp theo, tôi sẽ nói về từng thành phần cụ thể, việc này sẽ cung cấp cho bạn một cái nhìn sâu hơn về kiến trúc web cũng như những thứ ở phía sau của nó.

1. **DNS**

DNS (Domain Name Server): máy chủ phân giải tên miền, nó là xương sống của công nghệ web ngày nay. Về cơ bản, DNS cũng giống như một cuốn sổ danh bạ, nó cung cấp những cặp khoá/giá trị để tra cứu một tên miền ( ví dụ: google.com) có điạ chỉ IP là gì (85.129.83.20) để máy tính có thể điều hướng yêu cầu đến đúng nơi cần thiết.

Ví dụ, khi bạn truy cập đến google.com, DNS sẽ điều hướng yêu cầu đến máy chủ có địa chỉ IP là 85.129.83.20 để xử lý yêu cầu đó. Điều này tương tự như việc chúng ta mở điện thoại lên và bấm "call John" thay cho việc phải nhớ số điện thoại của John là "201-867-5039".

2. **Load Balancer (Cân bằng tải)**

Trước khi đi vào tìm hiểu cân bằng tải là gì, hãy cùng nhau nói về việc mở rộng ứng dụng theo chiều ngang và theo chiều dọc trước, chúng là gì và đâu là sự khác biệt giữa chúng? Câu trả lời đơn giản được tìm thấy trên [StackOverflow](https://stackoverflow.com/questions/11707879/difference-between-scaling-horizontally-and-vertically-for-databases): mở rộng theo chiều ngang là thêm nhiều máy chủ vào nguồn tài nguyên của bạn trong khi mở rộng theo chiều dọc là tăng sức mạnh hiện tại của máy chủ đã có (bằng cách nâng RAM, CPU...).

Đối với việc phát triển web, việc mở rộng theo chiều ngang sẽ thường được sử dụng nhiều hơn vì một số lý do như xác suất máy chủ bị hỏng, đường truyền mạng ở máy chủ có thể bị đứt... dẫn đến việc truyền dữ liệu bị gián đoạn và sẽ ảnh hưởng đến việc sử dụng của người dùng. Việc có nhiều hơn một máy chủ cho phép phòng bị được các rủi ro trên khi ứng dụng của bạn đang hoạt động. Ngoài ra, mở rộng theo chiều ngang còn giúp đơn giản hoá việc kết hợp các phần khác nhau của ứng dụng (máy chủ web, cơ sở dữ liệu, các dịch vụ ...) bằng cách chạy chúng trên những máy chủ khác nhau. Lý do cuối cùng là, đến một lúc nào đó, máy chủ của bạn sẽ không thể mở rộng theo chiều dọc thêm được nữa. Khi đó, không có một máy tính nào trên thế giới đủ lớn để thực hiện mọi hoạt động tính toán của ứng dụng. Hãy nghĩ nền tảng tìm kiếm Google như một ví dụ cho điều này, tuy nhiên có thể áp dụng cho những công ty nhỏ hơn. Đơn cử như tại Storyblocks, chúng tôi đang chạy 150-400 máy chủ AWS EC2 tại cùng một thời điểm cho ứng dụng web. Thật khó để xây dựng một máy chủ duy nhất (mở rộng theo chiều dọc) tương đương với sức mạnh của 150-400 máy chủ AWS hiện tại chúng tôi đang dùng.

Trở lại với cân bằng tải. Đó là một thứ ma thuật làm cho việc mở rộng theo chiều ngang trở nên có thể. Cân bằng tải định tuyến các yêu cầu đến một trong nhiều máy chủ đang chạy ứng dụng của bạn, thông thường, các máy chủ này là nhân bản của nhau, chúng xử lý và gửi kết quả lại cho máy khách. Bất kỳ một trong số các máy chủ sẽ đều xử lý yêu cầu theo một cách giống nhau (đây là lý do chúng được gọi là nhân bản của nhau). Do đó việc của cân bằng tải là làm sao phân phối các yêu cầu xử lý đến các máy chủ phân tán trên để không có yêu cầu nào bị quá tải.

3. **Web Application Server (Máy chủ ứng dụng Web)**

Máy chủ web là nơi thực hiện các hành động tính toán logic chính của ứng dụng như xử lý yêu cầu của người dùng, trả lại trang HTML cho trình duyệt của họ. Để làm được như thế, nó sẽ phải giao tiếp và liên kết nhiều thành phần của hạ tầng backend lại với nhau như cơ sở dữ liệu, bộ nhớ đệm, hàng đợi, dịch vụ tìm kiếm, microservices... Như đã nhắc ở trên, thường sẽ có hai hoặc nhiều hơn các máy chủ được cắm vào bộ cân bằng tải để xử lý yêu cầu của người dùng.

Bạn nên biết rằng, việc triển khai máy chủ ứng dụng yêu cầu chọn một ngôn ngữ cụ thể (Node.js, Ruby, PHP, Scala, Java ...) và 1 web MVC framework cho ngôn ngữ đó (Express cho Node.js, Ruby on Rails, Laravel cho PHP ...). Tuy nhiên, trong phạm vi bài viết này sẽ không đi sâu vào ngôn ngữ và những framework đó.

4. **Database Servers (Máy chủ  cơ sở dữ liệu)**

Những ứng dụng web hiện đại đều sử dụng cơ sở dữ liệu để lưu trữ thông tin. Cơ sở dữ liệu cung cấp những cách thức để định nghĩa cấu trúc của dữ liệu, thêm mới, tìm kiếm, cập nhật, hay xoá dữ liệu, thực hiện tính toán trên dữ liệu đó... Mỗi dịch vụ phụ trợ trong 1 ứng dụng web đều có thể có cơ sở dữ liệu cho riêng mình và độc lập với phần còn lại của ứng dụng đó.

Mặc dù tôi tránh đi sâu vào các công nghệ cho từng thành phần, nhưng thật thiếu sót nếu không đề cập đến ở đây, đó là có 2 loại cơ sở dữ liệu thường được sử dụng đó là: SQL và NoSQL.

SQL - Structured Query Language là ngôn ngữ truy vấn dữ liệu theo cấu trúc, được phát minh vào những năm 1970, cung cấp những quy chuẩn chung để truy vấn dữ liệu quan hệ. Cơ sở dữ liệu SQL lưu trữ dữ liệu dưới dạng bảng và các bảng được liên kết với nhau qua các khoá. Dưới đây là một ví dụ đơn giản cho cơ sở dữ liệu SQL. Bạn có thể thấy ở đây có 2 bảng là users và user_addresses, chúng được liên kết với nhau qua trường id của user vì ta thấy cột user_id trong bảng user_addresses là "khoá ngoài" trỏ tới cột id trong bảng user.  

  ![](https://cdn-images-1.medium.com/max/1600/1*Ln39QPggpJVMAScUBsrcCQ.png)

NoSQL là một công nghệ cơ sở dữ liệu mới hơn đã nổi lên gần đây để xử lý một lượng lớn dữ liệu có thể được tạo ra bởi các ứng dụng web quy mô lớn (hầu hết các biến thể của SQL không thể mở rộng theo chiều ngang tốt và chỉ có thể mở rộng theo chiều dọc đến một mức độ nhất định). Ở đây tôi có thể giới thiệu cho bạn một số nguồn tài nguyên để có thể tìm hiểu sâu hơn về NoSQL:

・[https://www.w3resource.com/mongodb/nosql.php](https://www.w3resource.com/mongodb/nosql.php)

・[http://www.kdnuggets.com/2016/07/seven-steps-understanding-nosql-databases.html](http://www.kdnuggets.com/2016/07/seven-steps-understanding-nosql-databases.html)

・[https://resources.mongodb.com/getting-started-with-mongodb/back-to-basics-1-introduction-to-nosql](https://resources.mongodb.com/getting-started-with-mongodb/back-to-basics-1-introduction-to-nosql)

5. **Caching Service (Dịch vụ caching)**

Một caching service cung cấp dữ liệu dạng khoá/giá trị để cho việc lưu trữ và tìm kiếm thông tin một cách nhanh nhất, tiệm cận tới độ phức tạp thuật toán O(1). Các ứng dụng thường tận dụng caching services để lưu trữ kết quả của những tính toán phức tạp cho lần sử dụng sau thay vì phải tính toán lại. Một ứng dụng có thể caching kết quả từ một truy vấn cơ sở dữ liệu, lời gọi tới những dịch vụ khác từ bên ngoài, các trang HTML và hơn thế nữa. Một số ví dụ thực tế của caching service: 

・ Google caching kết quả cho những câu tìm kiếm thông dụng như "dog" hay "Taylor Swift" hơn là việc phải tính toán lại mỗi lần người dùng tìm kiếm.

・ Facebook caching hầu hết dữ liệu bạn thấy sau khi bạn đăng nhập, như dữ liệu các bài đăng, bạn bè...

・ Storyblocks caching trang HTML mà được React (một framework javascript) tạo ra từ phía máy chủ, các kết quả tìm kiếm...

Hai đại diện phổ biến nhất về máy chủ caching là Redis và Memcache. Cũng như các phần trước, tôi sẽ không đề cập sâu về chúng ở bài viết này.

6. **Job Queue & Server (Máy chủ và hàng đợi Job)**

Hầu hết các ứng dụng cần phải thực hiện một số thao tác bất đồng bộ mà không trực tiếp liên kết với kết quả trả về cho yêu cầu của người dùng. Ví dụ, Google cần thu thập thông tin và đánh chỉ mục cho toàn bộ các trang web hay dữ liệu trên mạng internet để phục vụ cho việc trả lại kết quả tìm kiếm. Nó không làm điều này mỗi khi bạn tìm kiếm, thay vào đó, nó thu thập thông tin web một cách bất đồng bộ cùng với đó là cập nhật các chỉ mục tìm kiếm.

Trong khi có nhiều kiến trúc khác nhau cho phép thực hiện các công việc bất đồng bộ, nhưng phổ biến nhất là kiến trúc "job queue". Nó bao gồm 2 thành phần: một hàng đợi của các "jobs" cần thực thi và một hoặc nhiều server sẽ thực thi các công việc trong hàng đợi đó.

Lấy ví dụ tại Storyblocks, chúng tôi tận dụng job queue cho rất nhiều công việc như: mã hoá video và ảnh, xử lý CSV để gắn thẻ metadata, thống kê người dùng, gửi email để đặt lại mật khẩu ... Chúng tôi khởi tạo một hàng đợi FIFO (First In First Out) để đảm bảo các hoạt động cần ưu tiên theo thời gian như gửi email đặt lại mật khẩu được thực hiện càng sớm càng tốt.

7. **Full-text Search Service (Dịch vụ tìm kiếm toàn văn bản)**

Nhiều ứng dụng web có hỗ trợ việc tìm kiếm, nơi người dùng nhập văn bản (thường được gọi là truy vấn) và ứng dụng trả về các kết quả có liên quan nhiều nhất. Công nghệ để thực hiện việc này chính là "full-text seach", cái mà sử dụng các chỉ mục đảo ngược để nhanh chóng tra cứu các tài liệu chứa từ khoá truy vấn.

![](https://cdn-images-1.medium.com/max/1600/1*gun_BpdDH9KrNna1NnaocA.png)

  *Ví dụ cho thấy cách thức mà ba tiêu đề của tài liệu (trong bảng Document) được đảo ngược thành một bảng có chỉ mục đảo ngược (trong bảng Inverted Index) để tạo điều kiện tra cứu nhanh từ một từ khoá cụ thể đến các từ khoá có trong tiêu đề đó. Lưu ý, các từ như "in", "the", "with", ... (thường được gọi là từ dừng - stop word) thường sẽ không được liệt kê trong bảng Inverted Index*

Nền tảng tìm kiếm toàn văn bản phổ biến nhất hiện nay là Elasticsearch, ngoài ra còn có các lựa chọn khác như Sphinx hoặc Apache Solr. Một số database như MySQL cũng có thể thực hiện tìm kiếm toàn văn bản, nhưng thông thường việc này sẽ được tách riêng thành một dịch vụ độc lập.

8. **Services (Dịch vụ)**

Khi một ứng dụng đạt đến một quy mô nhất định, có khả năng sẽ có một số "services" được tách ra để chạy dưới dạng dịch vụ riêng biệt. Ứng dụng và các services khác sẽ tương tác với chúng một cách độc lập. Ví dụ tại Storyblock chúng tôi có các services như:

・ Account service: lưu trữ dữ liệu người dùng của toàn bộ ứng dụng, giúp chúng tôi tạo trải nghiệm nguời dùng nhất quán hơn.

・ Content service: lưu trữ metadata cho toàn bộ video, audio và nội dung ảnh.Nó cung cấp giao diện để tải nội dung và xem lịch sử tải xuống.

・Payment service: cung cấp giao diện để người dùng thanh toán qua thẻ tín dụng.

・ HTML -> PDF service: cung cấp giao diện để chuyển đổi HTML thành tài liệu PDF tương ứng.

9. **Data**

Ngày nay, sự tồn tại của các công ty phụ thuộc vào cách họ khai thác dữ liệu mà họ có. Gần như mọi ứng dụng khi đạt đến một quy mô nhất định, sẽ tận dụng đường ống dữ liệu (data pipeline) để đảm bảo dữ liệu có thể được thu thập, lưu trữ và phân tích. Một đường ống điển hình có 3 giai đoạn chính: 

1. Ứng dụng gửi dữ liệu, thường là các sự kiện tương tác của người dùng, đến "data firehose" - nơi cung cấp các giao diện trực tuyến để nhận và xử lý dữ liệu. Thông thường, dữ liệu thô được chuyển đổi hoặc tăng cường và đưa đến một data firehose khác. AWS Kinesis và Kafka là 2 trong số những công nghệ nổi bật nhất phục vụ cho mục đích này.  

2. Dữ liệu thô cũng như dữ liệu đã được chuyển đổi/ tăng cường sẽ được lưu trữ trên cloud. AWS Kinesis cung cấp một cài đặt rất dễ để cấu hình gọi là "firehose" giúp tiết kiệm dữ liệu thô tới nơi lưu trữ đám mây (S3) .

3. Dữ liệu được chuyển đổi / tăng cường thường được tải vào kho dữ liệu (data warehouse) để phân tích. Chúng tôi sử dụng AWS Redshift giống như hầu hết các startup đang phát triển, trong khi đó các công ty lớn thường sử dụng Oracle hoặc các công nghệ độc quyền của riêng họ. 

Một bước mà không được đề cập trong sơ đồ kiến trúc trên: tải dữ liệu từ cơ sở dữ liệu và các services vào kho. Ví dụ tại Storyblocks, chúng tôi tải các VideoBlocks, AudioBlocks, StoryBlocks, account service và các cơ sở dữ liệu cộng tác khác vào Redshift mỗi tối. Điều này cung cấp cho các nhà phân tích của chúng tôi một số liệu tổng thể về dữ liệu cũng như các hoạt động của người dùng.

10. **Cloud storage (Lưu trữ trên đám mây)**

Theo AWS: "Cloud storage là một cách đơn giản và dễ mở rộng để lưu trữ, truy cập và chia sẻ dữ liệu qua Internet". Bạn có thể sử dụng nó để lưu trữ và truy cập bất cứ thứ gì mà bạn đang lưu trữ trên hệ thống file cục bộ trên máy tính của mình, lợi ích của việc này là bạn có thể tương tác với những dữ liệu đó ở bất cứ đâu thông qua RESTful API và giao thức HTTP. Amazon S3 là bộ nhớ đám mây phổ biến nhất hiện nay và chúng tôi sử dụng nó ở Storyblocks để lưu trữ video, ảnh, và các tài sản audio khác. CSS, Javascript, dữ liệu sự kiện từ người dùng... cũng được lưu trữ tại đây.

11. **CDN**

CDN (Content Delivery Network) là một công nghệ cung cấp cách thức để chia sẻ những tài nguyên "tĩnh" như HTML, CSS, Javascript và ảnh một cách nhanh nhất có thể, hơn là việc sẽ phải phục vụ từ máy chủ gốc chứa tài nguyên đó. Nó hoạt động bằng cách phân phối nội dung thông qua rất nhiều máy chủ (edge servers) được đặt trên các vị trí địa lý khác nhau trên toàn thế giới, từ đó, người dùng cuối sẽ tải những tài nguyên này từ các máy chủ ở nơi gần với họ nhất. Lấy ví dụ ở bức ảnh bên dưới, một người dùng ở Tây Ban Nha gửi 1 yêu cầu đến một trang web có máy chủ gốc đặt ở New York, Mỹ  nhưng những tài nguyên tĩnh của trang web đó có thể được tải ở máy chủ CDN đặt tại Anh, giúp tránh việc phải truyền dữ liệu HTTP qua Đại Tây Dương sẽ làm giảm đáng kể tốc độ phản hồi của trang web. 

![](https://cdn-images-1.medium.com/max/1600/1*ZkC_5865Hx-Cgph3iPJghw.png)

[Xem bài viết này](https://www.creative-artworks.eu/why-use-a-content-delivery-network-cdn/) để được giới thiệu sâu hơn về CDN. Nói chung, một ứng dụng web nên sử dụng CDN để phục vụ các tài nguyên tĩnh như HTML, CSS, Javascript, ảnh, video... vì điều này sẽ cải thiện đáng kể tốc độ duyệt web của người dùng để tạo trải nghiệm tốt hơn.

**Lời Kết**

Hy vọng qua bài viết này, các bạn đã có một cái nhìn tổng quan nhất về kiến trúc của ứng dụng web. Đây là những điều cơ bản nhất mà một nhà phát triển web cần biết để có thể hiểu hơn về hệ thống của mình. Hẹn gặp lại các bạn vào các bài viết sau :
