# RetinaFace: Single-stage Dense Face Localisation in the Wild (Bản địa hóa khuôn mặt dày đặc một giai đoạn trong tự nhiên)

## Tóm tắt
Mặc dù đã đạt được những bước tiến to lớn trong việc phát hiện khuôn mặt không kiểm soát, 
nhưng việc xác định khuôn mặt chính xác và hiệu quả trong tự nhiên vẫn là một thách thức còn bỏ ngỏ. 
Bài báo này trình bày một máy dò khuôn mặt một giai đoạn mạnh mẽ, được đặt tên là RetinaFace, 
thực hiện định vị khuôn mặt khôn ngoan theo pixel trên các quy mô khuôn mặt khác nhau bằng cách tận dụng lợi thế của việc học đa tác vụ được giám sát và 
tự giám sát chung. Cụ thể, Chúng tôi đóng góp trong năm khía cạnh sau: 

* (1) Chúng tôi chú thích thủ công năm điểm mốc trên khuôn mặt trên tập dữ liệu WIDER FACE
và quan sát thấy sự cải thiện đáng kể trong nhận diện khuôn mặt cứng với sự hỗ trợ của tín hiệu giám sát bổ sung này.
với sự hỗ trợ của tín hiệu giám sát bổ sung này.
* (2) Chúng tôi bổ sung thêm một nhánh bộ giải mã lưới tự giám sát để dự đoán thông tin khuôn mặt hình dạng 3D theo pixel khôn ngoan song song với các nhánh được giám sát hiện có.
* (3) Trên bộ thử nghiệm cứng WIDER FACE, RetinaFace vượt trội hơn so với độ chính xác trung bình hiện đại (AP) 1,1% (đạt AP bằng 91,4%).
* (4) Trên bộ thử nghiệm IJB-C, RetinaFace cho phép các phương pháp hiện đại (ArcFace) để cải thiện kết quả của chúng trong việc xác minh khuôn mặt (TAR = 89,59% cho FAR = 1e-6). 
* (5) Bằng cách sử dụng các mạng trục trọng lượng nhẹ, RetinaFace có thể chạy thời gian thực trên một lõi CPU duy nhất cho hình ảnh có độ phân giải VGA.
## 1. Giới thiệu
Tự động định vị khuôn mặt là bước tiên quyết của phân tích hình ảnh khuôn mặt cho nhiều ứng dụng như thuộc tính khuôn mặt (ví dụ: biểu cảm [64] và tuổi [38]) và nhận dạng khuôn mặt [45, 31, 55, 11]. Định nghĩa hẹp về bản địa hóa khuôn mặt có thể đề cập đến tính năng phát hiện khuôn mặt truyền thống [53, 62], nhằm mục đích ước tính các hộp giới hạn khuôn mặt mà không có bất kỳ tỷ lệ và vị trí nào trước đó.
Tuy nhiên, trong bài báo này chúng tôi đề cập đến định nghĩa rộng hơn về bản địa hóa khuôn mặt bao gồm nhận diện khuôn mặt [39], căn chỉnh khuôn mặt [13], phân tích cú pháp khuôn mặt theo pixel [48] và hồi quy tương ứng dày đặc 3D [2, 12]. Loại định vị khuôn mặt dày đặc đó cung cấp thông tin chính xác về vị trí trên khuôn mặt cho tất cả các quy mô khác nhau.
Lấy cảm hứng từ các phương pháp phát hiện đối tượng chung [16, 43, 30, 41, 42, 28, 29], áp dụng tất cả những tiến bộ gần đây trong học sâu, tính năng nhận diện khuôn mặt gần đây đã đạt được những tiến bộ đáng kể [23, 36, 68, 8, 49]  .  Khác với tính năng phát hiện đối tượng thông thường, tính năng phát hiện khuôn mặt có các biến thể tỷ lệ nhỏ hơn (từ 1: 1 đến 1: 1.5) nhưng các biến thể tỷ lệ lớn hơn nhiều (từ vài pixel đến hàng nghìn pixel).  Các phương pháp hiện đại nhất gần đây nhất [36, 68, 49] tập trung vào thiết kế đơn nhất [30, 29] để lấy mẫu dày đặc các vị trí và tỷ lệ khuôn mặt trên các kim tự tháp đặc trưng [28], thể hiện hiệu suất đầy hứa hẹn và mang lại tốc độ nhanh hơn so với  phương pháp hai tầng [43, 63, 8].  Theo lộ trình này, chúng tôi cải thiện khung phát hiện khuôn mặt một giai đoạn và đề xuất phương pháp khoanh vùng khuôn mặt dày đặc hiện đại bằng cách khai thác tổn thất đa tác vụ đến từ các tín hiệu được giám sát và tự giám sát mạnh mẽ.  Ý tưởng của chúng tôi được chứng minh trong Hình 1.
![image](https://user-images.githubusercontent.com/80739312/112240485-00f73880-8c7b-11eb-908d-9aefd15fcd7e.png)  
**Hình 1.** Phương pháp bản địa hóa khuôn mặt pixel-khôn ngoan một giai đoạn được đề xuất sử dụng học đa tác vụ được giám sát và tự giám sát song song với các nhánh phân loại hộp và hồi quy hiện có.  Mỗi mỏ neo dương cho ra (1) điểm khuôn mặt, (2) hộp khuôn mặt, (3) năm điểm mốc trên khuôn mặt và (4) đỉnh khuôn mặt 3D dày đặc được chiếu trên mặt phẳng hình ảnh

Thông thường, quá trình đào tạo phát hiện khuôn mặt chứa cả tổn thất phân loại và hồi quy hộp [16]. Chen và cộng sự. [6] đề xuất kết hợp phát hiện khuôn mặt và căn chỉnh trong một khuôn khổ phân tầng chung dựa trên quan sát đã căn chỉnh
hình dạng khuôn mặt cung cấp các tính năng tốt hơn để phân loại khuôn mặt. Lấy cảm hứng từ [6], MTCNN [66] và STN [5] đồng thời phát hiện khuôn mặt và năm điểm mốc trên khuôn mặt. Do giới hạn về dữ liệu đào tạo, JDA [6], MTCNN [66] và STN [5] chưa xác minh được liệu tính năng nhận diện khuôn mặt nhỏ bé có thể được hưởng lợi từ việc giám sát thêm năm điểm mốc trên khuôn mặt hay không. Một trong những câu hỏi mà chúng tôi muốn trả lời trong bài báo này là liệu chúng tôi có thể thúc đẩy hiệu suất tốt nhất hiện tại (90,3% [67]) trên bộ kiểm tra cứng WIDER FACE [60] hay không bằng cách sử dụng thêm tín hiệu giám sát được xây dựng từ năm điểm mốc trên khuôn mặt.

Trong Mask R-CNN [20], hiệu suất phát hiện được cải thiện đáng kể bằng cách thêm một nhánh để dự đoán mặt nạ đối tượng song song với nhánh hiện có để nhận dạng và hồi quy hộp giới hạn. Điều đó xác nhận rằng các chú thích có mật độ pixel dày đặc cũng có lợi để cải thiện khả năng phát hiện. Thật không may, đối với các khuôn mặt đầy thách thức của WIDER FACE, không thể tiến hành chú thích khuôn mặt dày đặc (dưới dạng nhiều mốc hoặc phân đoạn ngữ nghĩa hơn). Vì không thể dễ dàng thu được các tín hiệu được giám sát, câu hỏi đặt ra là liệu chúng ta có thể áp dụng các phương pháp không được giám sát để cải thiện hơn nữa khả năng nhận diện khuôn mặt hay không.

Trong FAN [56], một bản đồ chú ý cấp neo được đề xuất để cải thiện khả năng phát hiện khuôn mặt bị che khuất. Tuy nhiên, bản đồ chú ý được đề xuất khá thô và không chứa thông tin ngữ nghĩa. Gần đây, các mô hình pha trộn 3D tự giám sát [14, 51, 52, 70] đã đạt được mô hình khuôn mặt 3D tự nhiên đầy hứa hẹn. Đặc biệt, Bộ giải mã lưới [70] đạt được tốc độ theo thời gian thực bằng cách khai thác các chập của đồ thị [10, 40] về hình dạng và kết cấu khớp. Tuy nhiên, những thách thức chính của việc áp dụng bộ giải mã lưới [70] vào máy dò một giai đoạn là: (1) các thông số máy ảnh khó ước tính chính xác và (2) hình dạng tiềm ẩn chung và biểu diễn kết cấu được dự đoán từ một vectơ đặc trưng ( 1 × 1 Ch.đổi trên kim tự tháp đối tượng) thay vì tính năng tổng hợp RoI, cho biết nguy cơ thay đổi tính năng. Trong bài báo này, chúng tôi sử dụng nhánh giải mã lưới [70] thông qua việc học tự giám sát để dự đoán hình dạng khuôn mặt 3D khôn ngoan theo pixel song song với các nhánh được giám sát hiện có.

Tóm lại, những đóng góp chính của chúng tôi là: 
* Dựa trên thiết kế một giai đoạn, chúng tôi đề xuất một phương pháp bản địa hóa khuôn mặt khôn ngoan theo pixel mới có tên là RetinaFace, sử dụng chiến lược học tập đa tác vụ để dự đoán đồng thời điểm số, khuôn mặt, năm điểm mốc trên khuôn mặt , và vị trí 3D và sự tương ứng của từng pixel trên khuôn mặt. 
* Trên tập con cứng WIDER FACE, RetinaFace làm tốt hơn AP của phương pháp hai giai đoạn hiện đại (ISRN [67]) 1,1% (AP bằng 91,4%). 
* Trên tập dữ liệu IJB-C, RetinaFace giúp cải thiện độ chính xác xác minh của ArcFace [11] (với TAR bằng 89,59% khi FAR = 1e-6). Điều này cho thấy rằng bản địa hóa khuôn mặt tốt hơn có thể cải thiện đáng kể khả năng nhận dạng khuôn mặt.
* Bằng cách sử dụng các mạng đường trục trọng lượng nhẹ, RetinaFace có thể chạy thời gian thực trên một lõi CPU duy nhất cho hình ảnh có độ phân giải VGA. 
* Các chú thích và mã bổ sung đã được phát hành để tạo điều kiện cho các nghiên cứu trong tương lai.
## 2. Công việc liên quan
**Image pyramid v.s. feature pyramid:** Mô hình trượt gió, trong đó một bộ phân loại được áp dụng trên lưới hình ảnh dày đặc, có thể được truy ngược lại những thập kỷ trước. Công trình quan trọng của Viola-Jones [53] đã khám phá chuỗi phân tầng để loại bỏ các vùng khuôn mặt giả từ kim tự tháp hình ảnh với hiệu quả thời gian thực, dẫn đến việc áp dụng rộng rãi khuôn khổ phát hiện khuôn mặt bất biến quy mô như vậy [66, 5]. Mặc dù cửa sổ trượt trên kim tự tháp hình ảnh là mô hình phát hiện hàng đầu [19, 32], với sự xuất hiện của kim tự tháp đối tượng [28], neo trượt [43] trên bản đồ đối tượng nhiều tỷ lệ [68, 49], nhanh chóng chiếm ưu thế phát hiện khuôn mặt.

**Two-stage v.s. single-stage:** Các phương pháp phát hiện khuôn mặt hiện tại đã kế thừa một số thành tựu từ các phương pháp phát hiện đối tượng chung và có thể được chia thành hai loại: phương pháp hai giai đoạn (ví dụ: Faster R-CNN [43, 63, 72]) và phương pháp một giai đoạn (ví dụ SSD [30 , 68] và RetinaNet [29, 49]). Phương pháp hai giai đoạn sử dụng cơ chế “đề xuất và sàng lọc” có độ chính xác bản địa hóa cao. Ngược lại, các phương pháp một giai đoạn lấy mẫu dày đặc các vị trí và tỷ lệ khuôn mặt, dẫn đến các mẫu âm và dương cực kỳ không cân bằng trong quá trình đào tạo. Để xử lý sự mất cân bằng này, phương pháp lấy mẫu [47] và tái trọng số [29] đã được áp dụng rộng rãi. So với phương pháp hai giai đoạn, phương pháp một giai đoạn hiệu quả hơn và có tỷ lệ thu hồi cao hơn nhưng có nguy cơ đạt tỷ lệ dương tính giả cao hơn và ảnh hưởng đến độ chính xác nội địa hóa.

**Context Modelling:** Để nâng cao khả năng lập luận theo ngữ cảnh của mô hình để chụp những khuôn mặt nhỏ bé [23], SSH [36] và PyramidBox [49] đã áp dụng mô-đun ngữ cảnh trên các kim tự tháp đặc trưng để phóng to trường tiếp nhận từ lưới Euclid. Để nâng cao năng lực mô hình hóa phép biến đổi không cứng nhắc của CNN, mạng tích chập có thể biến dạng (DCN) [9, 74] đã sử dụng một lớp có thể biến dạng mới để mô hình hóa các phép biến đổi hình học. Giải pháp quán quân của WIDER Face Challenge 2018 [33] chỉ ra rằng mô hình bối cảnh cứng nhắc (mở rộng) và không cứng nhắc (biến dạng) bổ sung và trực giao để cải thiện hiệu suất nhận diện khuôn mặt.

**Multi-task Learning:** Nhận diện và căn chỉnh khuôn mặt chung được sử dụng rộng rãi [6, 66, 5] vì các hình dạng khuôn mặt được căn chỉnh cung cấp các tính năng tốt hơn để phân loại khuôn mặt. Trong Mask R-CNN [20], hiệu suất phát hiện đã được cải thiện đáng kể bằng cách thêm một nhánh để dự đoán mặt nạ đối tượng song song với các nhánh hiện có. Densepose [1] đã sử dụng kiến ​​trúc của Mask-RCNN để có được các nhãn và tọa độ phần dày đặc trong mỗi vùng đã chọn. Tuy nhiên-ít hơn, nhánh hồi quy dày đặc trong [20, 1] được đào tạo bằng cách học có giám sát. Ngoài ra, nhánh dày đặc là một FCN nhỏ được áp dụng cho mỗi RoI để dự đoán ánh xạ dày đặc pixel-to-pixel.

## 3. RetinaFace
### 3.1.Multi-task Loss
![image](https://user-images.githubusercontent.com/80739312/112251640-3ce7c900-8c8e-11eb-9051-0bfd20f0fe98.png)  
### 3.2. Dense Regression Branch
**Mesh Decoder** Chúng tôi trực tiếp sử dụng bộ giải mã lưới (tích chập lưới và lấy mẫu tăng lưới) từ [70, 40], là một phương pháp tích chập đồ thị dựa trên lọc phổ cục bộ nhanh [10].  Để đạt được gia tốc hơn nữa, chúng tôi cũng sử dụng một bộ giải mã kết cấu và hình dạng khớp tương tự như phương pháp trong [70], trái ngược với [40] chỉ giải mã hình dạng.







