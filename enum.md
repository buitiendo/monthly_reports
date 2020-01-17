Enum là một kiểu dữ liệu đặc biệt, kiểu dữ liệu liệt kê. Các biến thuộc kiểu Enum này chỉ được phép chứa các giá trị đã được định nghĩa từ trước.

**1. Thực hiện**

Tạo một project demo nhỏ:
```
  rails new enum_mysql -d mysql
```

 sau đó cd  enum_mysql/  rồi sửa file database.yml
```
  default: &default
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: 123456
  socket: /var/run/mysqld/mysqld.sock

  development:
  <<: *default
  database: enum_mysql_development

  test:
  <<: *default
  database: enum_mysql_test

  production:
  <<: *default
  database: enum_mysql_production
  username: enum_mysql
  password: <%= ENV['ENUM_MYSQL_DATABASE_PASSWORD'] %>
```

tạo migration:
```
  rails g migration createPosts title:string body:text
```
khi đã migrate mà muốn thêm 1 column vào bảng posts: 
```
  rails g migration AddStatusToPosts
```
với nội dung:

```
  class AddStatusToPosts < ActiveRecord::Migration[5.2]
    def change
      add_column :posts, :status, :integer
    end
  end
```
Định nghĩa model Post như sau: 
```
  class Post < ApplicationRecord
    enum status: %i[draft reviewed published]
  end
```

class Post sẽ có một thuộc tính có tên là status, tương ứng với cột status trong database, trường này sẽ nhận 3 giá trị trạng thái tương ứng với 3 symbol đã định nghĩa là :draft, :reviewed và :published
Với thiết lập như trên, Rails sẽ tự động gán giá trị 0 cho trạng thái :draft, 1 cho trạng thái :reviewed và 2 cho trạng thái :published.

chạy thử rails console:

```
  irb(main):001:0> post = Post.new
  => #<Post id: nil, title: nil, body: nil, status: nil>
  irb(main):002:0> post.draft?
  => false
  irb(main):003:0> post.reviewed?
  => false
  irb(main):004:0> post.published?
  => false
```

Một instance Post được tạo ra sẽ được bổ sung 3 hàm draft?, reviewed? và published?  tương ứng với 3 giá trị đã khai báo của enum. Lúc mới khởi tạo, do giá trị status của instance đang là nil nên 3 hàm này đều trả về kết quả là false => sẽ rất nguy hiểm nếu nếu không khai báo giá trị mặc định cho trường dùng làm enum => khi đó giá trị sẽ không được thể hiện chính xác.

Với các trường sử dụng enum, ta set giá trị mặc định cho nó và không bao giờ được cho phép nó nil, tức là null: false.

sửa lại migration

```
  class AddStatusToPosts < ActiveRecord::Migration[5.2]
    def change
      add_column :posts, :status, :integer, null: false, default: 0
    end
  end
```

File schema sau khi chạy rails db:migrate

```
  ActiveRecord::Schema.define(version: 2020_01_17_031724) do
    create_table "posts", options: "ENGINE=InnoDB DEFAULT CHARSET=utf8", force: :cascade do |t|
      t.string "title"
      t.text "body"
      t.integer "status", default: 0, null: false
    end
  end
```

Chạy lại migration và test lại trên rails console:
```
  irb(main):005:0> post = Post.new
  => #<Post id: nil, title: nil, body: nil, status: "draft">
  irb(main):006:0> post.draft?
  => true
  irb(main):007:0> post.reviewed?
  => false
  irb(main):008:0> post.published?
  => false
```

Lúc này trường status đã có giá trị mặc định là 0, do đó khi khởi tạo một instance Post, mặc định status là draft, khá an toàn để thao tác. Chúng ta để thêm null: false để tránh các trường hợp set giá trị nil mà vẫn lưu vào được DB
```
  irb(main):009:0> post = Post.new
  => #<Post id: nil, title: nil, body: nil, status: "draft">
  irb(main):010:0> Post.status = nil
  => nil
  irb(main):011:0> Post.save
    Post Create (0.8ms)  INSERT INTO `posts` (`status`) VALUES (NULL)
    (0.2ms)  ROLLBACK
  Traceback (most recent call last):1: from (irb):7
  ActiveRecord::NotNullViolation (Mysql2::Error: Column 'status' cannot be null: INSERT INTO `posts` (`status`) VALUES (NULL))
```
Chúng ta có thể set giá trị nil cho trường status. Do đó nếu ở DB ko có ràng buộc null: false, chúng ta có thể lưu giá trị nil cho trường status lúc nào tùy thích, Vì thế, khi làm việc với các trường enum, nên set giá trị mặc định và không cho phép được nil.

Việc sử dụng enums cũng thêm scope thuận tiện:

```
  Post.draft     # => Lấy ra tất cả Post với status là draft
  Post.reviewed  # => Lấy ra tất cả Post với status là reviewed
  Post.published # => Lấy ra tất cả Post với status là published
```

**2. Ưu điểm và nhược điểm**

Bên cạnh các methods và scope tiện lợi mà tôi đã đề cập với bạn, ưu điểm chính của việc sử dụng enums, Một exception có tên là ArgumentError sẽ được throw ra khi bạn set giá trị status không phải là các giá trị của enum đã cho.

ví dụ sau đây sẽ throw ra một exception:

```
  post = Post.new

  post.status = "unknown"
  => ArgumentError ('unknown' is not a valid status)
```
nếu sau này model post của chúng ta mở rộng,status sẽ cần lưu thêm vài trạng thái nữa, ví dụ như :reject, :inprogress và chúng ta sẽ sửa lại model Post như sau

```
  class Post < ActiveRecord::Base
    enum status: %i[inprogress rejected draft reviewed published]
  end
```
Instance Post lúc đầu sẽ mang giá trị mặc định là :inprogress chứ ko phải là :draft nữa, các giá trị trong DB sẽ bị chuyển đổi.

Chạy lại migration và test lại trên rails console:
```
  irb(main):012:0> post = Post.new
  => #<Post id: nil, title: nil, body: nil, status: "inprogress">
  irb(main):013:0> post.draft?
  => false
  irb(main):014:0> post.inprogress?
  => true
  irb(main):015:0> post.published?
  => false
```
Chúng ta có thể thấy logic cho trường status đã bị biến đổi => ảnh hưởng đến logic => cần lưu ý.
Vậy Không thay đổi thứ tự các enum đã được sự dụng trước đó, nếu không logic sẽ bị thay đổi theo.
Để giải quyết việc này Rails cho phép chúng ta set các giá trị status theo kiểu hash, gồm key và value:

```
  class Post < ActiveRecord::Base
    enum status: {
      inprogress: 3, 
      rejected: 4,
      draft: 0,
      reviewed: 1,
      published: 2
    }
  end
```
Chạy rails console:
```
  irb(main):016:0> post = Post.new
  => #<Post id: nil, title: nil, body: nil, status: "draft">
  irb(main):017:0> post.inprogress?
  => false
  irb(main):018:0> post.draft?
  => true
  irb(main):019:0> post.published?
  => false
  irb(main):020:0> post.status = 3
  => "inprogress"
  irb(main):021:0> post.save
    (0.5ms)  BEGIN
  Post Create (0.3ms)  INSERT INTO `posts` (`status`) VALUES (3)
   (3.8ms)  COMMIT
  => true 
```
=> nó đã chạy đúng logic

integer column khó hiểu nếu không có ngữ cảnh thực tế.
Thay vì sử dụng integer, ta có thể sử dụng string. migration sẽ như thế sau:

```
  # db/migrate/20200113164051_add_status_to_posts.rb
  class AddStatusToPosts < ActiveRecord::Migration[5.2]
    def change
      add_column :posts, :status, :string null: false, default: "draft"
    end
  end
```
Tiếp theo, cần thay đổi enum như sau:

```
  class Post < ApplicationRecord
    enum status: {
      draft: "draft",
      reviewed: "reviewed",
      published: "published"
    }
  end
```
Table post có colum status là string và do đó, bất kỳ ai nhìn vào nó cũng sẽ biết là status của post

**3. kết luận**

ActiveRecord Enums thực rất ích khi được sử dụng đúng cách. Sử dụng dễ dàng hơn đối với những người có quyền truy cập ghi/đọc vào cơ sở dữ liệu bằng cách sử dụng string thay vì integer. Chúng ta cần thêm các ràng buộc tương tự trên cơ sở dữ liệu để đảm bảo tính toàn vẹn dữ liệu.

Tham khảo: https://sipsandbits.com/2018/04/30/using-database-native-enums-with-rails/
