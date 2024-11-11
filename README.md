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





# MongoDB Overview

## MongoDB là gì?
MongoDB là một cơ sở dữ liệu NoSQL mã nguồn mở, được thiết kế để lưu trữ và quản lý dữ liệu không có cấu trúc cố định (schema-less). MongoDB lưu dữ liệu dưới dạng các documents thay vì các bảng và hàng như trong cơ sở dữ liệu quan hệ. Documents được lưu trữ ở định dạng BSON (Binary JSON), giúp MongoDB phù hợp với dữ liệu phức tạp, đa dạng và dễ tương tác với các ứng dụng JavaScript.

## Ưu điểm của MongoDB so với MySQL
- **Tương thích JavaScript**: MongoDB lưu dữ liệu dưới dạng JSON/BSON, giúp dễ dàng tích hợp với các ứng dụng JavaScript.
- **Embedded Documents**: Cho phép lưu lồng (nested) các document, giảm sự phụ thuộc vào JOIN và cải thiện hiệu suất truy vấn.
- **Scalability**: Hỗ trợ sharding để phân phối dữ liệu trên nhiều máy chủ, giúp mở rộng quy mô hiệu quả.
- **Schema-less**: Không yêu cầu cấu trúc cố định, linh hoạt khi thay đổi dữ liệu mà không cần thay đổi toàn bộ hệ thống.

## Các khái niệm cốt lõi
- **Documents**: Đơn vị lưu trữ cơ bản, lưu dưới dạng BSON, có thể chứa dữ liệu lồng.
- **Collections**: Nhóm các documents, tương tự như bảng trong SQL nhưng không bắt buộc phải có cấu trúc giống nhau.
- **NoSQL & Schema-less**: Cho phép lưu trữ linh hoạt và thay đổi cấu trúc dữ liệu mà không cần cập nhật schema cố định.
- **Indexing**: Hỗ trợ tạo chỉ mục (index) để tăng tốc độ truy vấn.
- **Aggregation**: Framework thực hiện các truy vấn phức tạp như GROUP BY, JOIN.
- **Sharding**: Phân chia dữ liệu trên nhiều máy chủ.
- **Replication**: Tạo bản sao lưu để đảm bảo tính sẵn sàng của dữ liệu.

## Cách tạo Document
Có thể tạo document bằng MongoDB Shell, sau đây là một số ví dụ cơ bản về thao tác với MongoDB Shell:

### Tạo document:
```javascript
// Thêm một document vào collection `students`
db.students.insertOne({
    name: "Nguyễn Văn A",
    age: 22,
    courses: ["Math", "Science"],
    address: {
        city: "Hà Nội",
        street: "Nguyễn Trãi"
    }
});
```

### Tìm kiếm document:
```javascript
// Tìm tất cả các document trong collection `students`
db.students.find({});

// Tìm document với điều kiện
db.students.find({ age: { $gte: 20 } });
```

### Sắp xếp document:
```javascript
// Sắp xếp theo `age` giảm dần
db.students.find().sort({ age: -1 });
```

### Nested document và operator:
```javascript
// Truy vấn các document có "city" là "Hà Nội" trong nested document `address`
db.students.find({ "address.city": "Hà Nội" });

// Sử dụng toán tử $in để tìm các document có khóa `courses` chứa "Math" hoặc "Science"
db.students.find({ courses: { $in: ["Math", "Science"] } });
```

## MongoDB Driver
MongoDB Driver là một thư viện cung cấp giao diện lập trình ứng dụng (API) để kết nối và tương tác với cơ sở dữ liệu MongoDB từ các ngôn ngữ lập trình khác nhau. Nó cung cấp các chức năng cơ bản như kết nối, truy vấn, thêm, xóa và cập nhật dữ liệu trong MongoDB. Driver cho phép các ứng dụng thực hiện các tác vụ MongoDB qua mạng một cách dễ dàng và có thể mở rộng.

### Chức năng chính của MongoDB Driver:
- **Kết nối với MongoDB**: Cho phép các ứng dụng thiết lập và quản lý kết nối tới cơ sở dữ liệu MongoDB.
- **Tương tác với dữ liệu**: Cho phép thêm, xóa, cập nhật và truy vấn dữ liệu trong MongoDB.
- **Hỗ trợ bất đồng bộ**: Hỗ trợ thực thi các truy vấn không đồng bộ (async), tối ưu hóa hiệu suất.
- **Cấu hình dễ dàng**: Driver cung cấp nhiều tùy chọn cấu hình để tối ưu hóa cho các nhu cầu khác nhau.

## Sử dụng MongoDB Driver với FastAPI
MongoDB Driver cho Python là `motor`, hỗ trợ thao tác không đồng bộ (asynchronous) với MongoDB, rất phù hợp khi sử dụng cùng với FastAPI.

### Cài đặt Motor:
```bash
pip install motor
```

### Kết nối MongoDB với FastAPI:
```python
from fastapi import FastAPI, HTTPException
from motor.motor_asyncio import AsyncIOMotorClient
from pydantic import BaseModel
from typing import List

app = FastAPI()

# Thiết lập kết nối MongoDB
client = AsyncIOMotorClient("mongodb://localhost:27017")
db = client.my_database  # Thay thế 'my_database' với tên database của bạn
collection = db.students  # Thay thế 'students' với tên collection của bạn

# Định nghĩa schema sử dụng Pydantic
class Student(BaseModel):
    name: str
    age: int
    courses: List[str]
    address: dict

# Thêm một document vào MongoDB
@app.post("/students/")
async def create_student(student: Student):
    result = await collection.insert_one(student.dict())
    return {"id": str(result.inserted_id)}

# Lấy danh sách tất cả các document
@app.get("/students/", response_model=List[Student])
async def get_students():
    students = await collection.find().to_list(100)
    return students

# Lấy một document theo ID
@app.get("/students/{student_id}", response_model=Student)
async def get_student(student_id: str):
    student = await collection.find_one({"_id": student_id})
    if student is None:
        raise HTTPException(status_code=404, detail="Student not found")
    return student
```

## MongoDB Aggregation Framework
Aggregation Framework của MongoDB là một công cụ mạnh mẽ để thực hiện các truy vấn và phân tích phức tạp trên dữ liệu. Nó tương tự như các phép toán `GROUP BY` trong SQL và cho phép bạn thao tác với dữ liệu theo nhiều cách khác nhau.

### Các bước cơ bản của Aggregation
Aggregation hoạt động dựa trên một pipeline, nơi dữ liệu được truyền qua nhiều bước khác nhau để biến đổi và tổng hợp. Mỗi bước trong pipeline thực hiện một hoạt động cụ thể như lọc (`$match`), nhóm (`$group`), hay sắp xếp (`$sort`).

### Ví dụ về Aggregation
Dưới đây là một ví dụ về cách sử dụng Aggregation để tính số lượng sinh viên theo từng thành phố:
```javascript
// Đếm số lượng sinh viên theo thành phố
db.students.aggregate([
    { $group: { _id: "$address.city", totalStudents: { $sum: 1 } } },
    { $sort: { totalStudents: -1 } }
]);
```
Trong ví dụ trên:
- `$group`: Nhóm các document theo trường `address.city` và tính tổng số sinh viên trong mỗi nhóm.
- `$sort`: Sắp xếp kết quả theo `totalStudents` giảm dần.

### Một số toán tử phổ biến trong Aggregation
- **$match**: Lọc các document theo điều kiện nhất định (tương tự như `WHERE` trong SQL).
- **$group**: Nhóm các document lại với nhau và có thể thực hiện các phép tính tổng, trung bình, đếm, v.v.
- **$project**: Chọn các trường cụ thể để hiển thị (tương tự như `SELECT` trong SQL).
- **$sort**: Sắp xếp các document theo một hoặc nhiều tiêu chí.
- **$limit**: Giới hạn số lượng document trả về.

Aggregation Framework giúp bạn có thể thực hiện các thao tác phức tạp trên dữ liệu một cách hiệu quả, phù hợp với các ứng dụng phân tích dữ liệu và báo cáo.

Phân tích trên cung cấp một cái nhìn tổng quan về MongoDB, cách tạo document, và các thao tác với MongoDB sử dụng FastAPI. MongoDB Driver (Motor) rất phù hợp để làm việc với FastAPI do tính chất bất đồng bộ, giúp tối ưu hóa hiệu năng và tăng tính phản hồi của ứng dụng.

