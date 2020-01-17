### 1. Giới thiệu về form

Form là phương thức trong Helper để tạo ra đoạn mã html, cho phép người dùng create hoặc update các thuộc tính của model cụ thể. Form với các thuộc tính như: text_field, date_picker, radio_button...

![](https://user-images.githubusercontent.com/43602883/71637855-b9554d00-2c81-11ea-8367-cd3b7b017fd5.png)

Check_box, text_area, select_box ...

![](https://user-images.githubusercontent.com/43602883/71637855-b9554d00-2c81-11ea-8367-cd3b7b017fd5.png)

### 2. Nested Attributes

Với nested attributes, chúng ta có thể lưu đồng thời lưu dữ liệu ở bảng user và bảng profile trong cùng 1 form mà không phải thêm nhiều form khác nhau.

![](https://user-images.githubusercontent.com/43602883/71640206-0f44e780-2cb8-11ea-80ff-e26fc320e64e.png)

**2.1 Cách triển khai Nested Attributes**

- Associations:

```
  class User < ApplicationRecord
    has_many :profiles, dependent: :destroy, inverse_of: :user
      accepts_nested_attributes_for :profiles,
        allow_destroy: true, reject_if: :all_blank
  end
```

```
  class Profile < ApplicationRecord
    belongs_to :user, inverse_of: :profiles
  end
```

- Allow_destroy : true có nghĩa là cho phép xóa bản ghi con, ỏ đây là xóa address khỏi user đó. để xóa được các nested attribute cần phải có _destroy trong params profile mới hoạt động được.
- reject_if: all_bank : Nếu các trường Address bị trống thì nó sẽ không lưu bản ghi đó vào database.
- dependent: :destroy : Khi xóa user thì profiles phụ thuộc sẽ bị xóa theo.
- inverse_of: tối ưu hóa bộ nhớ khi truy cập các bản ghi liên kết. khi Gọi profile.user mà không thêm :inverse_of ở 2 quan hệ :belongs_to, :has_many.
  Khi thực hiện so sánh, nó truy vấn lại vào db để tìm kiếm và so sánh. Với :inverse_of, nếu record user thực sự tồn tại trong db.
  Khi chúng ta gọi profile.user sẽ trỏ về cùng một user.
  
 **2.2 sử dụng nested form dùng gem cocoon**
  
  <strong> Ưu điểm</strong> 
  
  Dùng gem cocoon để nhanh chóng tạo nested form ngắn gọn, đơn giản hơn so với sử dụng javascript thuần. Người dùng có thể thêm 1 trường địa chỉ khác mà không cần load lại trang ngay trên trình duyệt.
  
  <strong>  nhược điểm</strong> 
  
 Phụ thuộc vào jquery, cần thêm gem "jquery-rails" và require trong application.js.

Ngoài cách dùng gem "cocoon" còn có cách khác tạo nested form là sử dụng javascript thuần cũng cho kết quả tương tự. Có thể nested form nhưng cách làm này rườm ra và không hiệu quả do dùng javascript thuần.

Add gem 'cocoon' và 'jquery-rails' vào gem file:

```
  gem "jquery-rails"
  gem "cocoon"
```

required trong application.js:

```
  //= require jquery
  //= require cocoon
```

Chuẩn bị

```
  rails g scaffold City name:string
  rails g model User name:string email:string dob:datetime height:float gender:boolean description:text role:boolean
  rails g scaffold Profile address:string user_id:integer
```

![](https://user-images.githubusercontent.com/43602883/71654639-6b197a00-2d65-11ea-91ce-fa26a963eb09.png)

View sử dụng form_for

```
<%= form_for(@user) do |form| %>
  <div class="field">
    <%= form.label :name %>
    <%= form.text_field :name, required: true, type: "number" %>
  </div>
  <div class="field">
    <%= form.label :email %>
    <%= form.text_field :email, :required => true, maxlength: 10 %>
  </div>
  <div class="field">
    <%= form.label :dob %>
    <%= form.date_field :dob %>
  </div>
  <div class="field">
    <%= form.label :height %>
    <%= form.text_field :height, in: 0.00..3.00, step: 0.01, type: "number" %>
  </div>
  <div class="field">
    <%= form.label :role %>
    <%= form.check_box :role %>
  </div>
```

required: true bắt người dùng phải nhập vào field; type: "number" phải nhập kiểu số; maxlength: số ký tự tối đa được nhập; in: 0.00..3.00 là set value trong khoảng, với step là tăng giảm 0.01

```
  <div class="field">
    <%= form.label :gender %><br />
    <%= form.label :gender, "Male", value: true %>
    <%= form.radio_button :gender, false, checked: true %>
    <%= form.label :gender, "Female", value: false %>
    <%= form.radio_button :gender, true %>
  </div>
  <%= form.fields_for :profiles do |profile| %>
    <%= render "profile_fields", f: profile %>
  <% end %>

  <div class="links">
    <%= link_to_add_association "Add address", form, :profiles, class: "btn btn-primary" %>
  </div></br>
```

File _profile_field.html.erb

```
<div class="nested-fields">
  <%= f.label :address %>

  <%= f.text_field :address, class: "form-control" %>
  <%= link_to_remove_association "remove address", f, class: "btn btn-danger" %>
</div></br>
```

checked: true để  hiển thi dấu check ngoài form view. link_to_association: chính là đường link để hiện ra form khác cần sử dụng, f là object mà form for đang sử dụng.

```
  <div class="links">
    <%= link_to_add_association "Add address", form, :profiles, class: "btn btn-primary" %>
  </div></br>
  <div class="field">
    <%= form.label :description %>
    <%= form.text_area :description, cols: 19, rows: 3 %>
  </div>
  <div class="field">
    <%= form.label :city_id %>
    <%= form.select :city_id, City.all.collect {|c| [c.name, c.id]}, {include_blank: true} %>
  </div>
  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```

cols: 19, rows: 3 set độ dài, rộng cho text_area, include_blank: true, thêm 1 dòng trống cho item đầu tiên của select_box.

db sau khi create form

```
  2.5.1 :007 > User.last
    User Load (0.1ms)  SELECT  "users".* FROM "users" ORDER BY "users"."id" DESC LIMIT ?  [["LIMIT", 1]]
  => #<User id: 19, name: "1", email: "abc", dob: "2020-01-07 00:00:00", role: true, gender: false, description: "test", city_id: 1, created_at: "2020-01-07 04:01:19", updated_at: "2020-01-07 04:01:19"> 
  2.5.1 :008 > Profile.all
    Profile Load (0.4ms)  SELECT  "profiles".* FROM "profiles" LIMIT ?  [["LIMIT", 11]]
  => #<ActiveRecord::Relation [#<Profile id: 18, address: "address 4", user_id: 19, created_at: "2020-01-07 04:01:19", updated_at: "2020-01-07 04:01:30">, #<Profile id: 19, address: "address 3", user_id: 19, created_at: "2020-01-07 04:01:19", updated_at: "2020-01-07 04:01:19">]> 
  2.5.1 :009 >
```

**2.3. Sử dụng nested form dùng js thuần**

```
module ApplicationHelper
  def link_to_add_row(name, f, association, **args)
    new_object = f.object.send(association).klass.new
    id = new_object.object_id
    fields = f.simple_fields_for(association, new_object, child_index: id) do |builder|
      render(association.to_s.singularize, f: builder)
    end
    link_to(name, '#', class: "add_fields " + args[:class], data: {id: id, fields: fields.gsub("\n", "")})
  end
end
```

thêm 1 file custom.js

```
  $(document).on('turbolinks:load', function() {

  $('form').on('click', '.remove_record', function(event) {
    $(this).prev('input[type=hidden]').val('1');
    $(this).closest('tr').hide();
    return event.preventDefault();
  });

  $('form').on('click', '.add_fields', function(event) {
    var regexp, time;
    time = new Date().getTime();
    regexp = new RegExp($(this).data('id'), 'g');
    $('.fields').append($(this).data('fields').replace(regexp, time));
    return event.preventDefault();
  });
 
});
```

file view như sau:

```
  <div class="field">
    <%= form.label :gender %><br />
    <%= form.label :gender, "Male", value: true %>
    <%= form.radio_button :gender, false, checked: true %>
    <%= form.label :gender, "Female", value: false %>
    <%= form.radio_button :gender, true %>
  </div>
  <div class='fields'>
      <%= form.simple_fields_for :users do |builder| %>
        <%= render 'address', f: builder %>
      <% end %>
  </div>

  <div class="links">
    <%= link_to_add_association "Add address", form, :profiles, class: "btn btn-primary" %>
  </div></br>
```

_address.html.erb

```
<div>
  <td>
    <%= f.input_field :_destroy, as: :hidden %>
    <%= link_to 'Delete', '#', class: 'remove_record' %>
  </td>
 <%= f.input :address, label: false %></td>
</div>
```

=> Bài viết còn nhiều thiếu sót hy vọng được góp ý!