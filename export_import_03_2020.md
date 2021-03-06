### Import & Export CSV trong ruby on rails

Export dữ liệu từ hệ thống ra các định dạng file khác nhau như xlsx, csv và Import và dữ liệu từ các trên vào hệ thống là chức năng cơ bản và cần thiết nhất của mỗi ứng dụng. Trong bài này mình xin giới thiệu các import và export dữ liệu trong ruby on rails. 

**1. Import & Export**

Tạo 1 project mới

```
  rails new Import_Export_Csv -d mysql
```

sau đó 

```
  cd Import_Export_Csv
```

sửa file config/database.yml username và password mà đã đặt khi tạo mysql

```
  default: &default
    adapter: mysql2
    encoding: utf8
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
    username: root
    password: 123456
    socket: /tmp/mysql.sock

  development:
    <<: *default
    database: Import_Export_Csv_development

  test:
    <<: *default
    database: Import_Export_Csv_test

  production:
    <<: *default
    database: Import_Export_Csv_production
    username: Import_Export_Csv
    password: <%= ENV['IMPORT_EXPORT_CSV_DATABASE_PASSWORD'] %>
```
tạo model: 
```
  rails g model user name:string email:string phone:string address:string
```

chạy rails db:create sau đó chạy rails db:migrate

**2. Export**
  
tạo controller, model, view và thêm routes

```
  class User < ApplicationRecord
    CSV_ATTRIBUTES = %w(name email phone address).freeze
  end
```

app/controllers/users_controller.rb

```
  class UsersController < ApplicationController
    def index
      @users = User.all
    end
  end
```

app/views/users/index.html.erb

```
  <h1>List users</h1>
  <table>
    <thead>
      <th>name</th>
      <th>email</th>
      <th>phone</th>
      <th>address</th>
    </thead>
    <% @users.each do |u| %>
      <tbody>
        <td><%= u.name %></td>
        <td><%= u.email %></td>
        <td><%= u.phone %></td>
        <td><%= u.address %></td>
      </tbody>
    <% end %>
  </table>
  <%= link_to "Export", export_users_path(format: :csv) %>
```

File app/controllers/export_users_controller.rb

```
  class ExportUsersController < ApplicationController
    def index
      csv = ExportCsvService.new User.all, User::CSV_ATTRIBUTES
      respond_to do |format|
        format.csv { send_data csv.perform, filename: "users.csv" }
      end
    end
  end
```

<strong>Tạo service generate CSV</strong>

app/services/export_csv_service.rb

```
  class ExportCsvService
    require "csv"

    def initialize objects, attributes
      @attributes = attributes
      @objects = objects
      @header = attributes.map { |attr| "#{attr}" }
    end

    def perform
      CSV.generate do |csv|
        csv << header
        objects.each do |object|
          csv << attributes.map{ |attr| object.public_send(attr) }
        end
      end
    end

    private
    attr_reader :attributes, :objects, :header
  end
```

Tạo app/db/seeds.rb

```
  puts "Creating User ..."
  10.times do |index|
    user_params = {
      name: "Name #{index}",
      email: "user#{index}@gmail.com",
      phone: "098765432#{index}",
      address: "Address #{index}"
    }
    User.create(user_params)
  end
  puts "successfully"
```

chạy rails db:seed để tạo dữ liệu.

rồi chạy "rails s" -> mở trình duyệt chạy:  localhost:3000

![](https://user-images.githubusercontent.com/43602883/77242628-01598f80-6c33-11ea-9a0b-d7b8d089364e.png)

click link export trên hệ thống -> mở file users.csv sau khi tải về

![](https://user-images.githubusercontent.com/43602883/77242630-074f7080-6c33-11ea-8ed1-dc20bc334147.png)

**3. Import**

Mỗi khi tạo một bản ghi với ActiveRecord một câu lệnh INSERT được tạo ra và gửi đến cơ sở dữ liệu. Nghĩa là để tạo 100.000 ngàn bản ghi cần tạo 100.000 câu lệnh INSERT, cơ sở dữ liệu phải phân tích cú pháp câu lệnh, mở và đóng kết nối 100.000 lần, điều đó gây tốn rất nhiều thời gian và tài nguyên hệ thống do phải cấp phát hoặc giải phóng bộ nhớ cho 100.000 block.
=> giải pháp hiệu quả hơn là dùng Gem ActiveRecord import

thêm vào gemfile:

```
  gem "activerecord-import"
```

tiếp theo chạy bundle install

thêm đoạn này vào model

```
require 'activerecord-import/base'
# load the appropriate database adapter (postgresql, mysql2, sqlite3, etc)
require 'activerecord-import/active_record/adapters/postgresql_adapter'
```

ở đây ta dùng mysql nên file model sẽ như sau:

Model: app/models/user.rb

```
  class User < ApplicationRecord
    require "csv"
    require "activerecord-import/base"
    require "activerecord-import/active_record/adapters/mysql2_adapter"

    CSV_ATTRIBUTES = %w(name email phone address).freeze

    def self.import_file(file)
      users = []
      CSV.foreach(file.path, headers: true) do |row|
        users << User.new(row.to_h)
      end
      User.import users, recursive: true
    end
  end
```

Tạo app/controllers/import_users_controller.rb

```
  class ImportUsersController < ApplicationController
    def index
      @import = User.new
    end

    def create
      @import = User.import_file params[:file]
      redirect_to root_path
    end
  end
```

Thêm view: app/views/import_users/index.html.erb

```
  <div>
    <h4>Import data</h4>
    <%= form_tag import_users_path, multipart: true do %>
      <%= file_field_tag :file %>
      <%= submit_tag "Import users" %>
    <% end %>
  </div>
```

File routers: config/routes.rb

```
  Rails.application.routes.draw do
  root "users#index"
    resources :export_users
    resources :import_users
  end
```

Thêm link import vào view: app/views/users/index.html.erb

```
  <h1>List users</h1>
  <table>
    <thead>
      <th>name</th>
      <th>email</th>
      <th>phone</th>
      <th>address</th>
    </thead>
    <% @users.each do |u| %>
      <tbody>
        <td><%= u.name %></td>
        <td><%= u.email %></td>
        <td><%= u.phone %></td>
        <td><%= u.address %></td>
      </tbody>
    <% end %>
  </table>

  <%= link_to "Import", import_users_path %>
  <%= link_to "Export", export_users_path(format: :csv) %>
```

Tạo file import.csv với nội dung sau: 

![](https://user-images.githubusercontent.com/43602883/77242631-07e80700-6c33-11ea-93a5-80b75c653f9c.png)

Sau khi import thành công

![](https://user-images.githubusercontent.com/43602883/77242633-08809d80-6c33-11ea-87f0-a9b022574aa7.png)

Trên đây là cách import và export dữ liệu mà mình tìm hiểu được, bài viết còn nhiều thiếu sót, mong được góp ý. Xin cảm ơn.
