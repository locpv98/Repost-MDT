# Tăng tốc phát hiện đối tượng với 6 bước:
## Giới thiệu
Làm cho code của chúng ta chạy nhanh trên GPU cần một cách tiếp cận rất khác so với việc chạy trên CPU vì kiến trúc phần cứng khác nhau. Các kỹ sư ML/AI nên quan tâm đến việc tăng hiệu suất từ các mô hình và phần cứng không chỉ cho mục đích sản xuất mà còn cho nghiên cứu và đào tạo.

Bài viết này sẽ đi sâu vào thực tế về việc làm cho mô hình học sâu cụ thể (SSD300 của Nvidia) chạy nhanh trên một máy chủ GPU, các nguyên tắc, phương pháp chung có thể được áp dụng cho tất cả các loại GPU khác.

## Giai đoạn 0: PyTorch Hub Baseline
Phiên bản cơ sở của mã sẽ sử dụng các chức năng sau xử lý trong kho SSD300 theo trang PyTorch Hub. Những người triển khai mô hình này không giả vờ rằng mã mẫu này đã sẵn sàng sản xuất và chúng ta sẽ tìm nhiều cách để cải thiện nó. 
