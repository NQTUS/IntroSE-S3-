# Amazon S3 Overview

## I. Introduction
Amazon S3 (Simple Storage Service) là một dịch vụ lưu trữ đối tượng của Amazon Web Services (AWS), cho phép lưu trữ và quản lý dữ liệu dưới dạng tệp từ bất cứ đâu có kết nối internet. Điểm nổi bật của S3 là tính linh hoạt và độ bền cao, giúp bảo vệ dữ liệu và tối ưu chi phí lưu trữ cho người dùng.

## II. Core Concepts

### 1. Bucket
- **Định nghĩa**: Buckets là các thùng chứa dữ liệu trong S3 (Giống với folder), nơi các đối tượng được lưu trữ. Mỗi bucket trong S3 có một tên duy nhất trên toàn bộ AWS (global namespace), không thể có hai bucket trùng tên trên toàn cầu.
- **Đặc điểm**:
  - Có thể tạo tối đa 100 buckets mỗi tài khoản (có thể yêu cầu tăng thêm).
  - Có thể thiết lập quyền truy cập cho mỗi bucket và chỉ định quyền riêng tư hoặc công khai.
- **Ứng dụng thực tế**: Mỗi bucket có thể đại diện cho một loại dữ liệu hoặc ứng dụng khác nhau, ví dụ: một bucket chứa hình ảnh từ một dự án du lịch và một bucket khác lưu trữ video của dự án.

### 2. Objects
- **Định nghĩa**: Objects là đơn vị lưu trữ cơ bản của S3, bao gồm dữ liệu và siêu dữ liệu (metadata). Mỗi object được xác định duy nhất bởi một key (tên đối tượng) trong bucket.
- **Metadata**: Siêu dữ liệu đi kèm với đối tượng, chứa thông tin về đối tượng như loại MIME, kích thước tệp, và các tag.
- **Ứng dụng thực tế**: Hình ảnh của chuyến đi có thể được lưu dưới dạng các đối tượng với metadata mô tả ngày chụp, vị trí hoặc tag phân loại.
- **Đặc điểm**:
  - Dung lượng tối đa của một đối tượng là 5TB.
  - Để truy cập một đối tượng, có thể dùng URL dạng `http://s3.amazonaws.com/<bucket_name>/<object_key>`.

```python
import boto3
s3_client = boto3.client('s3')
my_object = s3_client.get_object(Bucket='<Tên_bucket>', Key='<Tên_đối_tượng>')
```

## III. S3 Classes

### 1. Standard (Tiêu chuẩn)
- Thích hợp cho dữ liệu truy cập thường xuyên, có độ bền và sẵn sàng cao.
- Lưu trữ các tài liệu, hình ảnh cần truy cập thường xuyên, như bản đồ hoặc lịch trình chi tiết của chuyến đi.

### 2. Intelligent-Tiering (Phân tầng thông minh)
- Tự động chuyển dữ liệu giữa các tầng truy cập thường xuyên và không thường xuyên dựa trên tần suất truy cập.
- Lưu trữ ảnh và tài liệu ít truy cập trong chuyến đi nhưng vẫn cần có sẵn khi cần.

### 3. Infrequent Access (IA - Truy cập không thường xuyên)
- Tối ưu cho dữ liệu truy cập không thường xuyên, nhưng cần thời gian truy xuất nhanh.
- Lưu trữ các dữ liệu như ảnh cũ của chuyến đi để tiện truy cập khi cần nhưng không tốn chi phí cao.

### 4. Glacier (Kho lưu trữ lâu dài)
- Phù hợp với lưu trữ dữ liệu lâu dài với chi phí rất thấp, nhưng thời gian truy xuất có thể kéo dài từ vài phút đến vài giờ.
- Lưu trữ dữ liệu cũ của các chuyến đi trước như ảnh hoặc video.

### 5. Chuyển Dữ Liệu Giữa Các Lớp Lưu Trữ
- Có thể sử dụng quy tắc lifecycle (vòng đời) để tự động chuyển dữ liệu giữa các lớp lưu trữ nhằm tối ưu hóa chi phí.

```python
s3_client.put_bucket_lifecycle_configuration(
   Bucket='<Tên_bucket>',
   LifecycleConfiguration={
      'Rules': [
         {
            'Status': 'Enabled',
            'Transitions': [
               {'Days': 30, 'StorageClass': 'STANDARD_IA'},
               {'Days': 365, 'StorageClass': 'GLACIER'}
            ],
         },
      ]
   }
)
```

## IV. S3 Security

### 1. Chế Độ Riêng Tư Mặc Định
- Khi tạo bucket mới, S3 đặt ở chế độ riêng tư, đảm bảo chỉ người sở hữu mới có quyền truy cập.

### 2. Thay Đổi Quyền Truy Cập
- Có thể thay đổi quyền truy cập thành công khai hoặc riêng tư theo mã Python sau:

```python
s3_client.put_object_acl(ACL='public-read', Bucket='<Tên_bucket>', Key='<Tên_đối_tượng>')
```

## V. S3 Events và Lambda Function

### 1. S3 Events
- S3 Events là một tính năng giúp phát hiện và phản hồi các thay đổi trong bucket. Khi một sự kiện như tải lên, xóa hoặc sao chép xảy ra, bạn có thể thiết lập để kích hoạt một hành động, ví dụ như một hàm Lambda.
- **Các Loại Event**:
  - `s3:ObjectCreated:*`: Kích hoạt khi một đối tượng mới được tạo (tải lên).
  - `s3:ObjectRemoved:*`: Kích hoạt khi một đối tượng bị xóa.
  - `s3:ObjectRestore:*`: Kích hoạt khi khôi phục một đối tượng từ Glacier.
- **VD Ứng Dụng**: Có thể tạo sự kiện để gửi thông báo khi ảnh hoặc video mới được tải lên từ một chuyến đi.

![S3 Event Configuration](attachment:Screenshot 2024-10-26 142838.png)

### 2. Lambda Function
- Lambda Function là một chức năng serverless có thể được kích hoạt bởi các sự kiện từ S3, cho phép xử lý dữ liệu hoặc thực hiện các hành động như thay đổi kích thước ảnh, lưu trữ dữ liệu phân tích, hoặc cập nhật cơ sở dữ liệu.

```python
import json

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    print(f"New file uploaded: {key} in bucket: {bucket}")
```

## VI. Các Thao Tác Cơ Bản

### 1. Upload Object
```python
s3_client.upload_file('path/to/file', 'bucket_name', 'object_name')
```

### 2. Tải Object Về Local
```python
s3_client.download_file('bucket_name', 'object_name', 'path/to/save/file')
```

### 3. Sao Chép Object
```python
copy_source = {'Bucket': 'source_bucket', 'Key': 'source_object'}
s3_client.copy_object(CopySource=copy_source, Bucket='destination_bucket', Key='destination_object')
```

### 4. Delete
```python
s3_client.delete_object(Bucket='bucket_name', Key='object_name')
```

### 5. Tạo Folder
```python
s3_client.put_object(Bucket='bucket_name', Key='folder_name/')
```

## VII. Link Tham Khảo Thêm
[Video hướng dẫn Amazon S3](https://www.youtube.com/watch?v=tfU0JEZjcsg&list=PL9nWRykSBSFgTXMWNvNufDZnwhHrwmWtb&index=1)




# OpenAI API Overview

## I. Introduction
OpenAI API cung cấp nền tảng trí tuệ nhân tạo có khả năng tạo nội dung tự động, xử lý ngôn ngữ tự nhiên và trả lời các câu hỏi của người dùng. Với ứng dụng đi phượt, OpenAI API có thể giúp tự động hóa các nhiệm vụ như tạo lịch trình, gợi ý địa điểm, dịch ngôn ngữ, hay tương tác với khách hàng.

## II. Core Concepts

### 1. API Key
- **Định nghĩa**: Là một mã khóa duy nhất được cung cấp cho từng tài khoản của OpenAI, dùng để xác thực và kiểm soát quyền truy cập vào các dịch vụ của OpenAI, bao gồm các mô hình như GPT-4, DALL-E, Whisper,... 
- **Vai trò**: API Key là yếu tố bảo mật quan trọng, vì nó giúp OpenAI nhận biết ai đang sử dụng dịch vụ, hạn chế truy cập trái phép và kiểm soát giới hạn sử dụng cho từng người dùng.

### 2. Models
- **a) GPT-4**: Dùng để tạo nội dung, đưa ra đề xuất và xử lý ngôn ngữ tự nhiên. Ví dụ, GPT-4 có thể gợi ý địa điểm tham quan hoặc viết mô tả về điểm đến.
- **b) Whisper**: Chuyển đổi giọng nói thành văn bản, hữu ích khi hướng dẫn viên cung cấp thông tin bằng giọng nói.
- **c) Embeddings**: Dùng để tìm kiếm và sắp xếp các câu hỏi, gợi ý tương tự giúp tìm kiếm thông tin chuyến đi dựa trên sở thích người dùng.

## III. Example Usage
Dưới đây là ví dụ về cách sử dụng API của OpenAI để tạo ra một câu trả lời tự động.

```javascript
fetch("https://api.openai.com/v1/completions", {
    method: "POST",
    headers: {
        Authorization: `Bearer ${API_KEY}`,
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        model: "gpt-3.5-turbo",
        prompt: "hello, how are you today?",
        max_tokens: 500,
        temperature: 0.2
    })
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

### 1. Các Thành Phần Quan Trọng Trong Đoạn Code
- **URL API**: `https://api.openai.com/v1/completions` là địa chỉ endpoint để tạo câu trả lời từ mô hình.
- **Authorization Header**: Sử dụng `Bearer ${API_KEY}` để xác thực người dùng.
- **Model**: `"gpt-3.5-turbo"` là mô hình sử dụng để tạo ra nội dung.
- **Prompt**: Văn bản đầu vào để GPT-3.5 bắt đầu tạo nội dung.
- **Max Tokens**: Giới hạn số lượng từ trong câu trả lời.
- **Temperature**: Giá trị từ `0` đến `1` để điều chỉnh độ sáng tạo của mô hình; giá trị càng cao thì mô hình càng tạo ra các câu trả lời ngẫu nhiên hơn.

## IV. Security Considerations

### 1. Bảo Vệ API Key
- Không được để lộ **API Key** trong client-side code vì nó có thể bị lạm dụng.
- Nên lưu trữ API Key ở **backend server** và chỉ gửi yêu cầu đến OpenAI từ server.

### 2. Xử Lý Lỗi (Error Handling)
- Cần bổ sung phần xử lý lỗi khi gọi API để tránh các trường hợp không mong muốn.
- Ví dụ: xử lý các lỗi liên quan đến kết nối mạng, lỗi xác thực (401 Unauthorized), hoặc vượt quá giới hạn gọi API (429 Too Many Requests).

## V. Ứng Dụng Thực Tế

### 1. Tạo Lịch Trình Tự Động
GPT-4 có thể tự động tạo lịch trình du lịch dựa trên sở thích người dùng. Người dùng chỉ cần cung cấp thông tin về địa điểm muốn đến và thời gian, GPT-4 sẽ tạo ra kế hoạch chi tiết phù hợp.

### 2. Phiên Dịch Ngôn Ngữ
Whisper có thể được sử dụng để chuyển đổi giọng nói thành văn bản và GPT-4 có thể tiếp tục dịch sang ngôn ngữ khác, giúp người dùng tương tác với người dân địa phương dễ dàng hơn.

### 3. Gợi Ý Địa Điểm Tham Quan
Embeddings có thể giúp tạo ra các gợi ý về địa điểm tham quan dựa trên sở thích của người dùng. Ví dụ: nếu người dùng thích các địa điểm có cảnh thiên nhiên yên bình, Embeddings sẽ tìm kiếm và đưa ra các gợi ý phù hợp.
