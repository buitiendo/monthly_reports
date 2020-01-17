<h1 class="text-center" style="text-align: center"> Elasticsearch - Khái niệm và các câu truy vấn cơ bản</h1>

<strong>1. ElasticSearch là gì?</strong>

Elasticsearch là công cụ tìm kiếm dựa trên nền tảng Apache Lucene. Nó cung cấp API cho việc lưu trữ và tìm kiếm dữ liệu một cách nhanh chóng. Nó được xây dựng, phát triển bằng ngôn ngữ java dựa trên Lucene – phần mềm tìm kiếm và trả về thông tin (information retrieval software) với hơn 15 năm kinh nghiệm về full text indexing and searching. Elasticsearch được xây dựng để hoạt động như một server cloud theo cơ chế của RESTful. Điều này giúp nó có thể tương tác và sử dụng bới rất nhiều ngôn ngữ, cũng chính do đó cũng là điểm yếu của nó khi độ bảo mật không cao.

<strong>2. Ưu và nhược điểm của ElasticSearch</strong>

<strong>- Ưu điểm</strong>

- Tìm kiếm dữ liệu rất nhanh chóng, mạnh mẽ dựa trên Apache Lucene.
Tìm kiếm trong elasticsearch gần như là realtime hay còn gọi là near-realtime searching.
- Có khả năng phân tích dữ liệu (Analysis data)
- Lưu trữ dữ liệu full-text
- Đánh index cho dữ liệu (near-realtime search/indexing, inverted index)
- Hỗ trợ tìm kiếm mờ (fuzzy), tức là từ khóa tìm kiếm có thể bị sai lỗi chính tả hay không đúng cú pháp thì vẫn có khả năng elasticsearch trả về kết quả tốt.
- Hỗ trợ nhiều Elasticsearch client phổ biến như Java, PhP, Javascript, Ruby, .NET, Python.

<strong>- Nhược điểm</strong>

- Elasticsearch được thiết kế cho mục đích search, do vậy với những nhiệm vụ khác ngoài search như CRUD thì elastic kém thế hơn so với những database khác như Mongodb, Mysql…. Do vậy người ta ít khi dùng elasticsearch làm database chính, mà thường kết hợp nó với 1 database khác.
- Trong elasticsearch không có khái niệm database transaction, tức là nó sẽ không đảm bảo được toàn vẹn dữ liệu trong các hoạt động Write, Update, Delete.
- Elasticsearch không cung cấp bất kỳ tính năng nào cho việc xác thực và phân quyền (authentication or authorization) . Điều này làm cho ElasticSearch kém sự bảo mật với các hệ quản trị cơ sở dữ liệu hiện nay.

<strong>- Khi nào nên sử dụng ElasticSearch?</strong>

- Tìm kiếm text thông thường – Searching for pure text (textual search).
- Tìm kiếm text và dữ liệu có cấu trúc – Searching text and structured data (product search by name + properties).
- Tổng hợp dữ liệu – Data aggregation.
- Tìm kiếm theo tọa độ – Geo Search.
- Lưu trữ dữ liệu theo dạng JSON – JSON document storage.
- Phục vụ cho việc lưu trữ và phân tích dữ liệu lớn.

Những web site sử dụng elasticsearch: stackoverflow, DataDog, Github ....

<strong>3. Các khái niệm cơ bản trong elasticsearch</strong>

- Cluster: tập hợp các node chứa tất cả các dữ liệu. Mỗi cluster được định danh bằng một unique name. Mỗi cluster có một node chính (master) được lựa chọn tự động và có thể thay thế khi gặp sự cố.

- Node: nơi lưu trữ dữ liệu, tham gia vào việc đánh chỉ mục của cluster cũng như thực hiện việc tìm kiếm. Mỗi node được định danh bằng một unique name.
- Index: Là một tập hợp các document.

- Shard: Tập con các document của một index. Một index có thể có nhiều shard. Có hay loại shard được sử dụng là Primary Shard và Replica Shard.

- Document: một JSON object với một số dữ liệu. Đây là đơn vị dữ liệu cơ bản trong Elasticsearch. Đối chiếu các khái niệm lưu trữ của Elasticsearch với một hệ quản trị csdl.

- Elasticsearch sử dụng inverted index để đánh chỉ mục cho các tài liệu. Inverted index là một cách đánh chỉ mục dựa trên trên đơn vị là từ nhằm mục đích tạo mối liên kết giữa các từ và các document chứa từ đó.

<strong>4. Cơ chế tìm kiếm</strong>

Giả sử có hai văn bản:

-  The quick brown fox jumped over the lazy dog
-  Quick brown foxes leap over lazy dogs in summer
![](https://user-images.githubusercontent.com/43602883/90957383-656cd200-e4b7-11ea-84d0-aed9664f9cea.png)

Khi tìm từ quick brown, ta chỉ cần tìm document mà các team xuất hiện
![](https://user-images.githubusercontent.com/43602883/90957387-6867c280-e4b7-11ea-8fbc-68e3c517d184.png)

Cả 2 document đều khớp với kết quả tìm kiếm, nhưng có thể thấy document 1 có độ chính xác cao hơn so với document 2.

<strong>5. Các câu truy vấn cơ bản</strong>

<strong>- Match query</strong>

Là truy vấn chuẩn để thực hiện full text query. Bao gồm truy vấn kết hợp và truy vấn cụm từ hoặc gần đúng. Match query chấp nhận văn bản, số, ngày tháng. Match query trả về các document chứa ít nhất 1 trong các từ trong truy vấn.

```
GET /_search
{
  "query": {
    "match": {
      "content": "Ruby on rails"
    } 
  }
}
```

<strong>- Match Phrase Query</strong>

```
GET /_search
{
  "query": {
    "match_phrase": {
      "message": "this is a test"
    }
  }
}
```

<strong>- Match Phrase Prefix Query</strong>

```
GET /_search
{
  "query": {
    "match_phrase_prefix": {
      "message": {
        "query": "quick brown f"
      }
    }
  }
}
```

<strong>- Multi Match Query</strong>

Tương tự match query nhưng cho phép tìm kiếm trên nhiều trường.

```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }
}
```

<strong>- Combining Queries</strong>
Combining queries cho phép thực hiện nhiều điều kiện trong tìm kiếm.

```
GET /_search
{
  "query": {
    "bool" : {
      "must": { "match": { "title": "ruby" }},
      "must_not": { "match": { "tag": "rails" }}, 
    }
  }
}
```

<strong>- Fuzzy Search</strong>

Fuzzy Seach (tìm kiếm "mờ"), hay còn hay được gọi là Approximate Search (tìm kiếm "xấp xỉ") là khái niệm để chỉ kỹ thuật để tìm kiếm một xâu "gần giống" (thay vì "giống hệt") so với một xâu cho trước.

Việc áp dụng kỹ thuật Fuzzy Search giúp cho người dùng dễ dàng tiếp cận được với nội dung hơn, khi mà họ có thể tìm thấy được những thứ cần thiết, ngay cả khi họ không nhớ được chính xác nội dung mình muốn tìm kiếm là gì.

Fuzzy Seach trong Elasticsearch sử dụng nền tảng dựa trên khoảng cách Levenstein. Khoảng cách Levenshtein giữa chuỗi S1 và chuỗi S2 là số bước ít nhất biến chuỗi S1 thành chuỗi S2 thông qua 3 phép biến đổi là:
- Xoá 1 ký tự
- thêm 1 ký tự.
- thay ký tự này bằng ký tự khác.

Ví dụ: Khoảng cách Levenshtein giữa 2 chuỗi "kitten" và "sitting" là 3, vì phải dùng ít nhất 3 lần biến đổi.

```
kitten -> sitten (thay "k" bằng "s")
sitten -> sittin (thay "e" bằng "i")
sittin -> sitting (thêm ký tự "g")
```

Fuzzy search trong Elasticsearch sử dụng khoảng cách Levenshtein và cho phép ta config tham số fuzziness để cho kết quả phù hợp nhất với nhu cầu:

- 0, 1, 2: Là khoảng cách Levenshtein lớn nhất được chấp thuận. Nghĩa là trong ví dụ trên nếu bạn đặt fuzziness=3 thì "cân đường" sẽ không được tìm thấy với từ khoá "con đường"
- AUTO: Sẽ tự động điều chỉnh kết quả dựa trên độ dài của term. Cụ thể:

```
0..2: bắt buộc match chính xác (khoảng cách Levenshtein lớn nhất là 0)
3..5: khoảng cách Levenshtein lớn nhất là 1
5 trở lên: khoảng cách Levenshtein lớn nhất là 2
```

Đoạn truy vấn ví dụ cho tìm kiếm mờ

```
GET /_search
{
  "query": {
    "query_string" : {
      "default_field": "content",
      "query": "rub o rail~", 
    }
  }
}
```