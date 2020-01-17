<h1 class="center"> Rails nhanh hơn, Database được đánh indexed.</h1>
Thực tế thì những người mới bắt đầu code Ruby sẽ đi theo xu hướng viết "đầy đủ". Theo cái cách tương đối "dài dòng" như các ngôn ngữ họ đã tiếp xúc trước đó, mà không biết rằng có thể viết chúng ngắn gọn hơn với các Syntax Ruby được định nghĩa lên. Bài viết này sẽ tổng hợp một vài cách viết ngắn gọn trong ruby on rails

<strong>Giảm số dòng code với mệnh đề If</strong>

```
if user.active?
  send_mail_to(user)
end
```
```
send_mail_to(user) if user.active?
```

<strong>Thay thế if + not (!if) bằng unless</strong>

```
user.destroy if !user.active?
```
```
user.destroy unless user.active?
```

<strong>Giảm số dòng bằng cách sử dụng toán tử tam phân<strong>

Tên gọi thuần việt là: toán tử ba ngôi =)), cách dùng này thì cũng khá quen thuộc với anh em từ dev Java sang

```
if user.admin?
  "I appreciate for that."
else
  "Thanks."
end
```

```
user.admin? ? "I appreciate for that." : "Thanks"
```
Lưu ý: khi dùng toán tử tam phân anh em cũng đừng lạm dụng quá, kiểu lồng nhau nested như thế này mình nghĩ là không nên

```
user.admin? ? user.active? ? "I appreciate for that." : "Are you OK?" : "Thanks."
```

<strong>Khai báo biến và dùng luôn<strong>

```
user = find_user
if user
  send_mail_to(user)
end
```

```
if user = find_user
  send_mail_to(user)
end
```

Cách viết này khá thú vị, tuy nhiên có nhiều người lại không thích vì dễ gây hiểu nhầm việc gán giá trị cho biến trong mệnh đề if thành toán tử so sánh ==

<strong>Kiểm tra nil của 1 đổi tượng<strong>

Nếu muốn kiểm tra các điều kiện bằng cách gọi các thuộc tính và phương thức của nó chỉ khi có một đối tượng con Đoạn mã sau đây cho thấy rằng parent.children có thể là nil

```
if parent.children
  if parent.children.singleton?
    singleton = parent.children.first
    send_mail_to(singleton)
  end
end
```
```
if parent.children && parent.children.singleton?
  singleton = parent.children.first
  send_mail_to(singleton)
end
```
<strong>Không cần dùng return cuối phương thức<strong>

```
def build_message(user)
  message = 'hello'
  message += '!!' if user.admin?
  return message
end
```

```
def build_message(user)
  message = 'hello'
  message += '!!' if user.admin?
  message
end
```

<strong>Sử dụng Object #tap, 1 cho tất cả<strong>

Thay vì khởi tạo object -> gán giá trị cho các thuộc tính của object đó -> trả về dưới dạng return object

```
def build_user
  user = User.new
  user.email = "hoge@hoge.com"
  user.name = "Taro Yamada"
  user
end
```
```
def build_user
  User.new.tap do |user|
    user.email = "hoge@hoge.com"
    user.name = "Taro Yamada"
  end
end
```

<strong>Lược dấu ngoặc đơn cuối method<strong>

```
def build_user(params, condition)
  users = User.get_by_scope(params, condition)
  ...
end
```
```
def build_user params, condition
  users = User.get_by_scope params, condition
  ...
end
```
