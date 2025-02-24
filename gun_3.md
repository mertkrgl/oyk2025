# 3. Gün

## 1. Hotwire

Hotwire, JSON yerine HTML kullanarak sayfa güncellemelerini hızlı ve verimli hale getiren bir teknolojidir. Bu sayede minimum JavaScript kullanımıyla dinamik kullanıcı arayüzleri oluşturulabilir.

### 1.1. Turbo Drive

Turbo Drive, sayfa yenilemeden bağlantılar ve formlar üzerinde otomatik gezinmeyi sağlar.

```ruby
<%= link_to "Show", post_path(post), data-turbo: {false} %>
# Bu kullanımda sayfa tamamen yeniden yüklenir.

<%= link_to "Show", post_path(post) %>
# Burada linke tıklandığında sayfanın yalnızca belirli bir bölümü güncellenir.
```

### 1.2. View Transition

Sayfa içinde gezinirken geçişlerin yumuşak bir şekilde gerçekleşmesini sağlar.

```html
<meta name="view-transition" content="same-origin" />
```

### 1.3. Turbo Frame

Turbo Frame, belirli bir HTML bölümünü güncelleyerek sadece ilgili içeriğin değişmesini sağlar.

```erb
# posts/show.html.erb
<%= turbo_frame_tag dom_id(@post , :comments), src: new_post_comment_path(@post) %>
```

```erb
# comments/new.html.erb
<%= turbo_frame_tag dom_id(@post , :comments) do %>
  <!-- Kod içeriği... -->
<% end %>
```

Turbo Frame kullanırken dikkat edilmesi gereken en önemli nokta, güncellenmek istenen alan ile yeni içeriğin aynı `id` değerine ve uygun HTML etiketi yapısına sahip olmasıdır. Eğer Turbo Frame, belirtilen `id` ile eşleşen bir içerik bulamazsa "Content Missing" hatası döndürecektir. Bu hatayı önlemek için, dinamik içeriklerin çerçeve içerisinde doğru şekilde yapılandırıldığından emin olunmalıdır.

```erb
# comments/_form.html.erb
data: {turbo_frame: "_top"}
```

- `_top` kullanımı ile tam bir yenileme yapılır ve eklenen elemanlar doğrudan gelir.
- `_top` kullanılmazsa form kaybolabilir.

### 1.4. Turbo Stream

Turbo Stream, belirli HTML öğelerini dinamik olarak güncelleyerek içeriği canlı tutar.

```ruby
# comments_controller.rb
if @comment.save
  respond_to do |format|
    flash.now[:notice] = "Comment created successfully"
    format.turbo_stream
  end    
else
  render :new, status: :unprocessable_entity
end
```

- `format.turbo_stream` kullanımı, içeriğin sayfa yenilenmeden dinamik olarak güncellenmesini sağlar.
- Turbo Stream'in kullanılabilmesi için ilgili işlemle eşleşen bir `create.turbo_stream.erb` dosyası oluşturulmalıdır.
- Alternatif olarak, Turbo Stream yanıtlarını doğrudan ilgili controller içerisinde `render turbo_stream:` yöntemi ile işleyebilirsiniz.

```erb
# comments/create.turbo_stream.erb
<%= turbo_stream.prepend dom_id(@post, :comments), @comment %>

<%= turbo_stream.replace dom_id(@post, :comments) do %>
  <%= turbo_frame_tag dom_id(@post, :comments), src: new_post_comment_path(@post) %>
<% end %>
```

- `prepend` ile yeni yorum sayfanın en üstüne eklenir.
- `replace` ile form sıfırlanarak yeni bir form gösterilir.

### 1.5. Stimulus

Stimulus, HTML öğeleri ile etkileşime girebilen küçük ve kolay kullanılabilir JavaScript controller'ları oluşturmayı sağlar.

**Örnek Stimulus Controller:**

```javascript
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  greet() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

**Kullanımı:**

```html
<div data-controller="hello">
  <input data-hello-target="name" type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

- `this.element` controller’in bağlı olduğu HTML öğesini işaret eder.

### 1.6. Turbo Native

[Hotwire Native ile yaptığımız İOS uygulaması](https://github.com/azimcan/LKD2025-blog-native-ios)

[Hotwire Native ile yaptığımız Android uygulaması](https://github.com/azimcan/LKD2025-blog-android-native)

Turbo Native, web uygulamalarının mobil uygulamalar ile entegre şekilde çalışmasını sağlar.

Mobil uygulamalar için, web görünümü (`WKWebView` veya `WebView`) ile Turbo Native Bridge kullanılarak etkileşim sağlanabilir.

#### **Turbo Native Bridge**

Turbo Native, iOS ve Android uygulamalarında web görünümleri ile yerel bileşenler arasında iletişimi sağlar. `turbo-ios` ve `turbo-android` kütüphaneleri, native uygulamalara entegre edilerek sayfa geçişleri ve köprü işlevleri oluşturulabilir.

#### **Mobil View Katmanı**

Mobilde özel sayfa gösterimleri yapmak için `+mobile` uzantısını kullanabilirsiniz:

```erb
# application.html+mobile.erb
```

Bu sayede mobil kullanıcılar için özelleştirilmiş görünümler sunulabilir.

#### **Mobil ve Web Ayrımı**

Mobil ve web kullanıcılarını ayırt etmek için `turbo_helper.rb` dosyasında aşağıdaki gibi bir yardımcı metod tanımlanabilir:

```ruby
# app/helpers/turbo_helper.rb
def mobile?
  request.user_agent =~ /Turbo Native/
end
```

Bu metod, kullanıcının mobil mi yoksa web mi olduğunu anlamak için kullanılabilir.

#### **Örnek Kullanım:**

```erb
<% if mobile? %>
  <p>Mobil görünüm</p>
<% else %>
  <p>Web görünüm</p>
<% end %>
```

Bu yapı sayesinde mobil ve web arasında dinamik olarak içerik değiştirilebilir.
