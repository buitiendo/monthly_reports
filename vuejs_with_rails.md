<h1 class="text-center" style="text-align: center"> Vuejs with Rails</h1>
Vue.js gọi tắt là Vue, là một framework linh động (progressive framework) được khởi chạy vào năm 2013, dùng để xây dựng giao diện người dùng có khả năng tương tác cao, được thiết kế khác với các framework nguyên khối (monolithic - loại hỗ trợ đầy đủ tất cả mọi thứ cần có để xây dựng một ứng dụng trong một framework). Vuejs dễ dàng đáp ứng nhu cầu xây dựng một Single Page Applications với độ phức tạp cao. Khi phát triển phần giao diện, chỉ cần dùng thư viện lõi (core library) của Vue tích hợp với các thư viện hoặc dự án có sẵn khác.

Hiện nay cũng có rất nhiều các thư viện hay JS framework mạnh mẽ như React được phát triển bởi Facebook, Angular được phát triển bởi Google, Ember, ... Nhưng Vue lại là sự lựa chọn lý tưởng cho các ứng dụng web ở mức vừa và nhỏ do hiệu năng đáng nể so với các đối thủ khác, dung lượng tải thấp do lõi nhỏ gọn với khả năng tương thích cao nên tăng tốc độ tải của toàn trang, xử lý nhanh với Virtual DOM, giúp thảm thiểu tối đa công sức. Thêm vào đó tài liệu tham khảo đơn giản, API gọn nhẹ, khả năng tích hợp tuyệt vời.

<strong>1. Tích hợp Vue vào ứng dụng Rails bằng Webpacker</strong>

Webpacker là một ruby gem dùng để tích hợp Webpack vào Rails sử dụng chung với Asset Pipeline. Webpacker giúp dễ dàng sử dụng bộ tiền xử lý JS và bundler webpack để quản lý giống như ứng dụng JS trong Rails. Vì mục đích chính của webpack là xử lý như một ứng dụng JS, không phải images, CSS hay thậm chí là JS Sprinkles (tất cả vẫn có trong app/assets). Tuy nhiên, vẫn có thể sử dụng Webpacker cho CSS, images, và fonts assets, trong trường hợp đó thậm chí không cần asset pipeline. Cái này chủ yếu có liên quan khi chỉ sử dụng các JS frameworks hướng component.


<strong>2. Sử dụng</strong>

Cài đăt: Để khởi tạo một dự án với Rails kết hợp với Vuejs đơn giản chỉ cần dùng lệnh

```
rails new myapp --webpack=vue
```

Tuy nhiên, nếu muốn tích hợp VueJS với một project có sẵn thì có thể làm như sau:
Cài đặt gem: 

```
gem "webpacker", "~> 4.x"
```
và chạy 

```
bundle
rails webpacker:install
rails webpacker:install:vue
```

Nếu cài đặt thành công, sẽ thấy một message nho nhỏ như sau:

```
Webpacker now supports Vue.js 🎉
```

Có một chút sự khác biệt giữa project Rails thông thường và project Rails + VueJS. Trong app/javascript sẽ xuất hiện cấu trúc thư mục như sau

```
app/javascript
├── app.vue
└── packs
  ├── application.js
  └── hello_vue.js
end
```

Đã cài đặt xong, Chạy thử rails s => Không có gì thay đổi. 

![](https://user-images.githubusercontent.com/43602883/88071439-0527ff00-cb9e-11ea-831e-5bc2e5ff0845.png)

Trước tiên tạo ra một view

```
rails g controller static_pages index
```

sau đó vào config/routes.rb

```
Rails.application.routes.draw do
  root to: 'static_pages#index'
end
```
=> Vậy là trang mặc định đã được thiết lập

![](https://user-images.githubusercontent.com/43602883/88071842-897a8200-cb9e-11ea-89bf-91b670e3f407.png)

Giờ vào app/views/static_pages.erb vào xóa tất cả đi thay bằng đoạn code sau

```
<%= javascript_pack_tag 'hello_vue' %>
```

Load lại server 1 lần nữa: 
Ok vậy là đã load thành công VueJS, nếu muốn thay đổi đoạn chữ in ra mặc định của VueJS khi khởi tạo project, chỉ cần vào app/javascript/app.vue thay đổi

```
<template>
  <div id="app">
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      message: "Hello everybody!"
    }
  }
}
</script>

<style scoped>
p {
  font-size: 2em;
  text-align: center;
}
</style>
```

<strong>3. Demo tính toán hai số đơn giản sử dụng VueJS</strong>

Vàojavascript/app.vue và thay nội dung code cũ bằng đoạn code mới như sau:

```
<template>
  <div id="app">
    Number A: <input type="number" name="numA" v-on:input="getNumA">
    Number B: <input type="number" name="numB" v-on:input="getNumB">
    <button @click="plus">Tổng</button>
    <button @click="sub">Hiệu</button>
    <button @click="multi">Tích</button>
    <button @click="division">Thương</button>
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  data: function () {
    numA: 0;
    numB: 0;
    return {
      message: 0
    }
  },
  methods: {
    getNumA: function(event) {
      this.numA = event.target.value;
    },
    getNumB: function(event) {
      this.numB = event.target.value;
    },
    plus: function() {
      this.message = Number(this.numA) + Number(this.numB);
    },
    sub: function() {
      this.message = Number(this.numA) - Number(this.numB);
    },
    multi: function() {
      this.message = Number(this.numA) * Number(this.numB);
    },
    division: function() {
      this.message = Number(this.numA) / Number(this.numB);
    }
  }
}
</script>

<style scoped>
#app {
  font-size: 2em;
  text-align: center;
}
</style>
```

Refresh lại trang và xem lại kết quả.

![](https://user-images.githubusercontent.com/43602883/88072201-f4c45400-cb9e-11ea-8496-b741d9aae2a2.png)