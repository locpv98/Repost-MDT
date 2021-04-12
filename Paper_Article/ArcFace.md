# ArcFace: Additive Angular Margin Loss for Deep Face Recognition
## Introduction
Có hai hướng nghiên cứu chính để đào tạo CNN về nhận dạng khuôn mặt, một là đào tạo một bộ phân loại nhiều lớp bằng cách sử dụng bộ phân loại softmax và một là nghiên cứu các phép nhúng chẳng hạn như the triplet loss. Tuy nhiên, cả hai đều có nhược điểm của chúng:
+ Đối với softmax loss, bạn càng thêm nhiều đặc điểm nhận dạng khác nhau để nhận dạng, thì số lượng tham số sẽ tăng lên.  
+ Và đối với sự triplet loss, có một sự bùng nổ tổ hợp về số lượng bộ ba khuôn mặt đối với tập dữ liệu quy mô lớn dẫn đến số lần lặp lại lớn.  

Suy hao biên góc phụ được đề xuất trong arcface để cải thiện hơn nữa sức mạnh mô tả của mô hình nhận dạng khuôn mặt và ổn định quá trình đào tạo. Hàm arc-cosine được sử dụng để tính toán góc giữa đặc tính hiện tại và trọng lượng mục tiêu.  ArcFace trực tiếp tối ưu hóa biên khoảng cách trắc địa nhờ sự tương ứng chính xác giữa góc và cung trong siêu cầu chuẩn hóa [nơi các đặc điểm của khuôn mặt nằm].  

## Cách tiếp cận
Tổn thất được sử dụng rộng rãi nhất để phân loại, tức là softmax như sau:  
<p align="center"><img src=https://miro.medium.com/max/369/1*pIwx2O5H_tgWfCj7CYy1UA.png></p>  
Trong đó x biểu thị vectơ đặc trưng của mẫu thứ i, W và b lần lượt là weight và bias. Softmax loss không tối ưu hóa rõ ràng tính năng nhúng để thực thi độ tương đồng cao hơn cho các mẫu nội lớp và tính đa dạng cho các mẫu giữa các lớp, điều này dẫn đến chênh lệch hiệu suất để nhận dạng khuôn mặt sâu dưới các biến thể ngoại hình lớn trong nội bộ lớp (ví dụ: thay đổi tư thế, chênh lệch tuổi tác  Vân vân).

Trong phần mất mát softmax ở trên, chúng tôi đã cố định độ lệch thành 0 để đơn giản hơn và sau đó chúng tôi biến đổi logit thành:
<p align="center"><img src=https://miro.medium.com/max/270/1*Ksu1Q59UJQT6j6v4VkDB4w.png></p>
trong đó θ là góc giữa trọng lượng W và tính năng x. Trọng lượng được chuẩn hóa thành 1 bằng cách sử dụng định mức L2. Tính năng cũng được chuẩn hóa L2 và thu nhỏ lại thành s. Các bước chuẩn hóa giúp đưa ra các dự đoán chỉ phụ thuộc vào góc θ giữa đối tượng địa lý và trọng lượng. Phép nhúng đã học được phân phối trên hypersphere với bán kính s như sau:
<p align="center"><img src=https://miro.medium.com/max/486/1*RntJG4aplR-VoNm0hUTLWA.png></p>  
Một hình phạt biên góc cộng thêm m được thêm vào giữa trọng lượng và tính năng để nâng cao độ nhỏ gọn trong nội bộ lớp và sự khác biệt giữa các lớp. Vì hình phạt lề góc phụ gia được đề xuất bằng với hình phạt lề khoảng cách trắc địa trong siêu cầu chuẩn hóa, nó được đặt tên là ArcFace. Hàm mất mát cuối cùng trở thành:
<p align="center"><img src=https://miro.medium.com/max/539/1*c1UeN8NADBMqazZ_C8d0qA.png></p>

## Kiến trúc
![image](https://user-images.githubusercontent.com/80739312/114338671-bd0e9980-9b7d-11eb-89f0-c62ee31a1138.png)  

Quy trình đào tạo :
1. Sau khi chuẩn hóa tính năng x i và trọng số W, ta nhận được cos θ j (logit) cho mỗi lớp là (W j) ' x i .
2. Chúng tôi tính arccosθ yi và nhận được góc giữa đối tượng địa lý x i và trọng lượng chân lý mặt đất W yi.
3. Chúng tôi thêm một hình phạt biên góc m trên mục tiêu (chân lý mặt đất) góc θ yi .
4. Chúng ta tính cos (θ yi + m) và nhân tất cả logit với thang đặc trưng s.
5. Sau đó logits đi qua hàm softmax và đóng góp vào tổn thất entropy chéo.

## Sự khác biệt hình học B / W SphereFace, CosFace & ArcFace
Lề góc phụ gia có thuộc tính hình học tốt hơn vì lề góc có sự tương ứng chính xác với khoảng cách trắc địa.
![image](https://user-images.githubusercontent.com/80739312/114338730-e62f2a00-9b7d-11eb-9442-d1d4fa683656.png)  
So sánh ranh giới phân loại trong trường hợp phân loại nhị phân. ArcFace có biên độ góc tuyến tính không đổi trong toàn bộ khoảng thời gian. Ngược lại, SphereFace và CosFace chỉ có một lề góc phi tuyến.

## Phần kết luận
Chúng tôi đã tìm hiểu về một hàm mất mát mới để nhận dạng khuôn mặt hoạt động trong không gian góc và giúp mô hình tìm hiểu các đặc điểm rất phân biệt tạo ra một biên góc tuyến tính.
