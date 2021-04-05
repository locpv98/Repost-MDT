# ArcFace: Additive Angular Margin Loss for Deep Face Recognition
## Introduction
Có hai hướng nghiên cứu chính để đào tạo CNN về nhận dạng khuôn mặt, một là đào tạo một bộ phân loại nhiều lớp bằng cách sử dụng bộ phân loại softmax và một là nghiên cứu các phép nhúng chẳng hạn như the triplet loss. Tuy nhiên, cả hai đều có nhược điểm của chúng:
+ Đối với softmax loss, bạn càng thêm nhiều đặc điểm nhận dạng khác nhau để nhận dạng, thì số lượng tham số sẽ tăng lên.  
+ Và đối với sự triplet loss, có một sự bùng nổ tổ hợp về số lượng bộ ba khuôn mặt đối với tập dữ liệu quy mô lớn dẫn đến số lần lặp lại lớn.  

Suy hao biên góc phụ được đề xuất trong arcface để cải thiện hơn nữa sức mạnh mô tả của mô hình nhận dạng khuôn mặt và ổn định quá trình đào tạo. Hàm arc-cosine được sử dụng để tính toán góc giữa đặc tính hiện tại và trọng lượng mục tiêu.  ArcFace trực tiếp tối ưu hóa biên khoảng cách trắc địa nhờ sự tương ứng chính xác giữa góc và cung trong siêu cầu chuẩn hóa [nơi các đặc điểm của khuôn mặt nằm].  

## Cách tiếp cận
Tổn thất được sử dụng rộng rãi nhất để phân loại, tức là softmax như sau:  
![](https://miro.medium.com/max/369/1*pIwx2O5H_tgWfCj7CYy1UA.png)  
Trong đó x biểu thị vectơ đặc trưng của mẫu thứ i, W và b lần lượt là weight và bias. Softmax loss không tối ưu hóa rõ ràng tính năng nhúng để thực thi độ tương đồng cao hơn cho các mẫu nội lớp và tính đa dạng cho các mẫu giữa các lớp, điều này dẫn đến chênh lệch hiệu suất để nhận dạng khuôn mặt sâu dưới các biến thể ngoại hình lớn trong nội bộ lớp (ví dụ: thay đổi tư thế, chênh lệch tuổi tác  Vân vân).
