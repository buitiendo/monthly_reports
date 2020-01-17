<h1 class="center"> Tổng hợp vài cách viết ngắn gọn trong ruby on rails</h1>

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

<strong>Giảm số dòng bằng cách sử dụng toán tử tam phân</strong>

Tên gọi thuần việt là: toán tử ba ngôi, cách dùng này thì cũng khá quen thuộc với dev Java sang

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
Lưu ý: khi dùng toán tử tam phân, cũng đừng lạm dụng quá, kiểu lồng nhau nested như thế này mình nghĩ là không nên

```
user.admin? ? user.active? ? "I appreciate for that." : "Are you OK?" : "Thanks."
```

<strong>Khai báo biến và dùng luôn</strong>

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

<strong>Kiểm tra nil của 1 đổi tượng</strong>

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
<strong>Không cần dùng return cuối phương thức</strong>

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

<strong>Sử dụng Object #tap, 1 cho tất cả</strong>

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

<strong>Lược dấu ngoặc đơn cuối method</strong>

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

<strong>Sử dụng try thay vì kiểm tra blank?</strong>

```
user.id if !user.id.blank?
```
```
use.try(:id)
```

<strong>Dùng select để duyệt toàn bộ mảng?</strong>

```
[1, 2, 3, 4, 5].select { |element| element.even? }
```

```
[1, 2, 3, 4, 5].select(&:even)
```

<strong>dig là một method dùng cho hash nó dùng để rút gọn cho phép &&.</strong>

```
if params[:user] && params[:user][:address] && params[:user][:address][:somewhere_deep]
```

```
if params.dig(:user, :address, :somewhere_deep)
```
<strong>presence_in trả về giá trị boolean, thay thế cho phép include?</strong>

```
sort_opt = [:by_date, :by_title, :by_author]
sort = sort_opt.include?(params[:sort])? params[:sort] : :by_date
```

```
sort = params[:sort].presence_in(sort_opt) : :by_date
```

<strong>Dùng pluck thay vì map</strong>

pluck là method để lấy 1 column cho trước trong các record, mà không load toàn bộ các record đó. Vì thế mà tốc độ xử lý và RAM cũng hiệu quả hơn.

```
def admin_user_ids
  User.where(admin: true).map(&:id)
end
```

```
def admin_user_ids
  User.where(admin: true).pluck(:id)
end
```

<strong>Khi ghép các chuỗi kí tự, không dùng “+” mà dùng “#{ }”</strong>

```
"Hello, " + user.name + "!"
```

```
"Hello, #{user.name}!"
```

<strong>Find: trả lại yếu tố đầu tiên tìm thấy</strong>

```
def find_admin(users)
  users.each do |user|
    return user if user.admin?
  end
  nil
end
```

```
def find_admin(users)
  users.find(&:admin?)
end
```

<strong>Select: lấy tất cả những yếu tố thoả mãn điều kiện</strong>

```
def find_admins(users)
  admins = []
  users.each do |user|
    admins << user if user.admin?
  end
  admins
end
```

```
def find_admins(users)
  users.select(&:admin?)
end
```

<strong>Sử dụng presence</strong>

```
if user.name.blank?
  name = "What's your name?"
else
  name = user.name
end
```

```
name = user.name.presence || "What's your name?"
```

<strong>Các method thời gian hay</strong>

```
Date.current # => Tue, 05 Nov 2013

Date.yesterday  # => Tue, 04 Nov 2013
Date.tomorrow # =>  # => Tue, 06 Nov 2013

2.years.ago   # => 2011-11-05 06:21:40 +0900
2.years.since # => 2015-11-05 06:21:40 +0900

2.months.ago   # => 2013-09-05 06:21:40 +0900
2.months.since # => 2014-01-05 06:21:40 +0900
```

<strong>Khi cần filter, nên dùng query thay vì logic</strong>

Ruby cung cấp rất nhiều method hay và đơn giản để thao tác với array, nhưng khi cần thực hiện filter trong model của Rails, thì nên sử dụng query để tốc độ xử lý được nhanh hơn.

```
def admin_users
  User.all.select(&:admin?)
end
```

```
def admin_users
  User.where(admin: true)
end
```

<strong>Xoá các space không cần thiết</strong>

```
"    My    \r\n  \t   \n   books       ".squish # => "My books"
```