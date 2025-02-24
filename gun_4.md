# 4. Gün

## **1. Polymorphic İlişkiler**

Polymorphic ilişkiler, bir modelin birden fazla farklı modele tek bir ilişki üzerinden bağlanabilmesini sağlar. Bu, veritabanı tasarımında esneklik sunarak kod tekrarını azaltır ve ilişkilerin yönetimini kolaylaştırır.

Uygulamamızda kullanıcılar hem gönderilere (posts) hem de yorumlara (comments) beğeni (like) bırakabiliyor. Ayrı ayrı PostLike ve CommentLike modelleri oluşturmak yerine, tek bir Like modeli kullanarak polymorphic ilişki kurduk.

Veritabanı Yapılandırması:

likes tablosunda, beğeninin hangi modele ait olduğunu belirtmek için likeable_type ve likeable_id sütunları eklenir:

```ruby
# db/migrate/xxxx_create_likes.rb
class CreateLikes < ActiveRecord::Migration[8.0]
  def change
    create_table :likes do |t|
      t.belongs_to :likeable, polymorphic: true
      t.belongs_to :user
      t.timestamps
    end

    add_index :likes, [ :user_id, :likeable_id, :likeable_type ], unique: true
  end
end
```

```ruby
# app/models/like.rb
class Like < ApplicationRecord
  belongs_to :likeable, polymorphic: true, counter_cache: true
end
```

## **2. Meta Programming**

Meta Programlama:

Meta programlama, bir programın kendi kodunu veya diğer programların kodunu dinamik olarak oluşturma veya değiştirme yeteneğidir. Ruby, esnek ve dinamik yapısı sayesinde meta programlama için güçlü destek sunar.

```ruby
#Örnek: define_method ile Dinamik Metot Oluşturma
class DynamicMethods
  [:foo, :bar, :baz].each do |method_name|
    define_method(method_name) do
      puts "#{method_name} metodu çağrıldı"
    end
  end
end

obj = DynamicMethods.new
obj.foo # Çıktı: foo metodu çağrıldı
obj.bar # Çıktı: bar metodu çağrıldı
```

## **3. Lazy Load ile Infinity Scroll**

Lazy Load ile Infinity Scroll mantığı, kullanıcı sayfanın altına kaydırdıkça yeni içeriğin otomatik olarak yüklenmesini sağlar. Bu, özellikle uzun içerik listeleri için verimli bir çözüm sunar. Lazy loading, sadece ekranda görünmeye başlayan içeriklerin yüklenmesi anlamına gelir. Bu sayede başlangıçta tüm içeriği yüklemek yerine, yalnızca gerekli olan kısmı yükleyerek sayfanın hızını artırır.

```erb
# posts/index.html.erb
<% unless @page.last? %>
  <%= turbo_frame_tag :pagination, src: posts_path(page: @page.next_param, format: :turbo_stream), loading: :lazy do %>
    Loading...
  <% end %>
<% end %>
```

## **4. Kamal - Rails Deploy**

Kamal, Rails uygulamalarını Docker ile kolayca deploy etmeyi sağlayan bir araçtır. Yaptığımız uygulamayı Kamal kullanarak deploy ettik.
