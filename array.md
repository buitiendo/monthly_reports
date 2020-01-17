<h1 class="center"> 6 phương thức xử lí Array của Ruby nên biết</h1>

Mảng (Array) là một trong những kiểu cấu trúc dữ liệu cơ bản trong lập trình. Việc nhanh chóng vận dụng và đọc, xử lí dữ liệu từ mảng là điều khá quan trọng để xây dựng những thứ vĩ mô hơn với máy tính. Dưới đây là 6 phương phương thức bạn có thể dùng hoặc không.

<strong>1. Map và Each</strong>

- Hai phương thức này trông có vẻ khá là giống nhau. Cho phép lướt qua từng phần từ và xử lí chúng.

```
[01] pry(main)> array = [1, 2, 3]
=> [1, 2, 3]
[02] pry(main)> effects = array.each{|x|}
=> [1, 2, 3]
[03] pry(main)> added = array.map{ |x| x + 2 }
=> [3, 4, 5]
```

- Nhanh hơn và tốt hơn: .map(&:method). Giả sử ta có 1 mảng chữ và mình muốn convert nó thành số, thì như cú pháp ở trên ta làm như sau:

```
[04] pry(main)> numbers = ["5", "4", "8"]
=> ["5", "4", "8"]
[05] pry(main)> string_numbers = numbers.map { |number| number.to_i }
=> [5, 4, 8]
```

- Nhưng có 1 cách nhanh gọn lẹ hơn nữa, đó là dùng cú pháp .map(&:method):

```
[06] pry(main)> numbers = ["5", "4", "8"]
=> ["5", "4", "8"]
[07] pry(main)> string_numbers = numbers.map(&:to_i)
=> [5, 4, 8]
```

Side effects in map


Nếu sử dụng để tạo các function, thì  .Map trong Ruby's có vẻ rất lạ. Nhìn vào ví dụ này. Ta có một lớp Sự kiện đơn giản trong project:

```
# we create an array of records
2.3.0 :07 > array = [e, e2, e3]
 => [#<Event id: 1, name: nil>, #<Event id: 2, name: nil">, #<Event id: 3, name: nil>]
# so far so good
2.3.0 :07 > new_array = array.map{|e| e.name = "a name"; e}
 => [#<Event id: 1, name: "a name">, #<Event id: 2, name: "a name">, #<Event id: 3, name: "a name">]
# uh-oh, that ain't right
2.3.0 :07 > array
 => [#<Event id: 1, name: "a name">, #<Event id: 2, name: "a name">, #<Event id: 3, name: "a name">]
```
<strong>2. Select</strong>

- Select giúp tìm các phần tử thoả mãn điều kiện trong mảng. Bạn cần đưa cho .select một câu điều kiện trả về true/false. Select sẽ giúp lấy ra các phần tử thoả mãn điều kiện đó.


```
[08] pry(main)> array = ['sun', 'framgia', 'fsoft', 'fpt']
=> ["sun", "framgia", "fsoft", "fpt"]
[09] pry(main)> array.select{|word| word.length == 3}
=> ["sun", "fpt"]
[10] pry(main)> 
```


<strong>3. Reject</strong>

- Reject là phương thức ngược lại của select. ví dụ kết hợp reject và map:

```
hot_girls =
  [
    {name: "Ngọc trinh", height: 168}, 
    {name: "Hari won", height: 168}, 
    {name: "Thuỳ Dung sun*", height: 172}
  ]
```

- Bây giờ chúng ta muốn reject đi những cô gái cao bằng hoặc dưới 1m70:

```
[11] pry(main)> hot_girls =
[11] pry(main)* [  
[11] pry(main)*   {name: "Ngọc trinh", height: 168},     
[11] pry(main)*   {name: "Hari won", height: 168},     
[11] pry(main)*   {name: "Thuỳ Dung sun*", height: 172}    
[11] pry(main)* ]    
=> [{:name=>"Ngọc trinh", :height=>168}, {:name=>"Hari won", :height=>168}, {:name=>"Thuỳ Dung sun*", :height=>172}]
[12] pry(main)> hot_girls.reject { |girl| girl[:height] <= 170 }.map { |girl| girl[:name] } 
=> ["Thuỳ Dung sun*"]
[13] pry(main)>
```

Thay vì chọn những phần tử thoả mãn điều kiện như select, chúng ta sẽ loại bỏ mọi thứ không thoả mãn điều kiện đã cho. Mọi phần tử thoả mãn điều kiện sẽ bị loại bỏ đi (như Ngọc trinh và Hari won ở ví dụ trên).

<strong>4. Reduce</strong>

Reduce có cấu trúc hơi phức tạp 1 xíu so với các phương thức khác. Tuy nhiên nó thường được sử dụng trong các vấn đề tính toán cộng trừ nhân chia các thứ trong Ruby. Sau mỗi lần chạy, mảng đó sẽ return lại cái gì?.

- Tính tổng trong mảng:

```
[13] pry(main)> array = [1, 2, 3]
=> [1, 2, 3]
[14] pry(main)> array.reduce{|sum, x| sum + x}
=> 6
```

- Có thể truyền tham số mặc định cho reduce để xác định giá trị ban đầu:

```
[15] pry(main)> array = [1, 2, 3]
=> [1, 2, 3]
[16] pry(main)> array.reduce(4) { |sum, x| sum+x }
=> 10
```

- Tương tự với mảng string

```
[17] pry(main)> unit3s = ["Sun", "Digital", "Creative", "Studio"]
=> ["Sun", "Digital", "Creative", "Studio"]
[18] pry(main)> unit3s.reduce { |sum, unit3| sum+unit3 }
=> "SunDigitalCreativeStudio"
```

- Nhanh hơn và tốt hơn: .reduce(default value, :operation)

```
[19] pry(main)> array = [1, 2, 3]
=> [1, 2, 3]
[20] pry(main)> array.reduce(4, :+)
=> 10
```

- *Lưu ý: khi thực hiện với kiểu dữ liệu không phải là số hoặc chuỗi, sẽ phải khai báo giá trị ban đầu trong tham số của reduce:

```
members = 
  [
    {team: "ACE", number: 7},
    {team: "3S", number: 7},
    {team: "Wolf Tsunami", number: 6},
    {team: "Spark", number: 7},
    {team: "Dopamine", number: 5},
    {team: "DongAm", number: 3},
    {team: "ToLai", number: 6}
  ]

# Truyền giá trị ban đầu bằng 0

[21] pry(main)> members.reduce(0) {|sum, member| sum + member[:number] }
=> 41

# Không truyền giá trị mặc định

[22 pry(main)> members.reduce {|sum, member| sum + member[:number] }
NoMethodError: undefined method `+' for {:team=>"ACE", :number=>7}:Hash
from (pry):43:in `block in __pry__'
[23] pry(main)> 
```

<strong>5. Join</strong>

Nếu muốn kiểm tra các điều kiện bằng cách gọi các thuộc tính và phương thức của nó chỉ khi có một đối tượng con. Đúng như tên gọi của nó, join là một phương thức để kết hợp nối các phần tử trong mảng lại với nhau. join khá là giống với reduce ngoại trừ việc nó có syntax sạch sẽ hơn. Join cần 1 đối số truyền vào chuỗi sẽ được chèn giữa các phần tử được nối của mảng. Join sẽ tạo ra một chuỗi string dài bất kể bạn đưa cho nó những thứ không phải String.

```
places = Place.all.limit 3
places.join(" ")
=> "#<Place:0x00005645c39f9200> #<Place:0x00005645c39f9070> #<Place:0x00005645c39f8c38>"
```

```
cars = [{type: 'porsche', color: 'red'}, {type: 'mustang', color: 'orange'}]
cars.join(" and ")
 => "{:type=>\"porsche\", :color=>\"red\"} and {:type=>\"mustang\", :color=>\"orange\"}"
```

Tham khảo: https://medium.freecodecamp.org/six-ruby-array-methods-you-need-to-know-5f81c1e268ce https://ruby-doc.org/core-2.5.3/Array.html