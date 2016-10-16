### Relationship - Embedded

* Parent document of the relation must declare the **embeds_one** macro to indicate it has one **embedded** child

* Document that is embedded uses **embedded_in**

* Actor -> place_of_birth:Place

  Actor是一个parent document, 他有一个Place类的place_of_birth document,这里place使用embedded_in

#### 详细解释一下这个relationships

**这里的Place对Actor或者Writer的关系叫做polymorphic(多态，多型关系)，这个关系与many-to-many很像但是有区别：**

**1. Many-to-one 之中，多个Place可以被同意个Actor类下的对象所拥有**

**2. Polymorphic 之中，一个Place可以被不同的Actor或者Writer类下的对象所拥有，而单一的Actor和Writer对应Place则是一个one-to-one relationship**

```ruby
class Place
  include Mongoid::Document
  ...
  embedded_in :locatable, polymorphic: true
end
```

```ruby
class Actor
  include Mongoid::Document
  ...
  embeds_one :place_of_birth, as: :locatable, class_name: 'Place'
end
```

```ruby
class Writer
  include Mongoid::Document
  ...
  embeds_one :hometown, as: :locatable, class_name: 'Place'
end
```

1. 一个Actor embeds_one :place_of_birt, 同时一个Writer embeds_one :hometown, 这两个都是指向**Place**这个class。这个意思是，一个Actor知道一个**Place**, 一个Writer知道一个**Place**，这是一个**one-to-one** relationship，意为着一个Actor有一个place_of_birth, 一个Writer又一个hometown.
2. 同时多个Actor可以拥有同一个**Place**，多个Writer可以拥有同一个**Place**。**Place**可以属于多个Actor或者多个Writer，所以我们设置了``` polymorphic: true ```,意思是**Place**可以有不同的parents

---

#### Polymorphic Relationships

*  将一个完全一样的document嵌入到不同的parent type之中

*  Child -> polymorphic

   ```ruby
   class Place
     include Mongoid::Document
     ...
     embedded_in :locatable, polymorphic: true
   end
   ```


---

#### Demo

1. embed relationship

   * 首先找一个没有**place_of_birth**属性的**Actor**，这样的话我们可以将一个地址赋予它

     ```ruby
     irb(main):003:0> actor = Actor.where(:place_of_birth=>{:$exists=>0}).first
     => #<Actor _id: nm0012524, created_at: nil, updated_at: nil, name: "Frank Adu", birthName(birth_name): nil, date_of_birth: nil, height: nil, bio: "Frank Adu is an actor, known for Taxi Driver (1976), Guerre et amour (1975) and Across 110th Street (1972).">
     ```

   * 我们假设这个**actor**来自Oakland然后我们从**Place**中找到这个地址赋予给**actor**

     ```ruby
     irb(main):004:0> oakland=Place.where(:city=>"Oakland").first
     Overwriting existing field _id in class Place.
     => #<Place _id: Oakland, CA, USA, formatted_address: nil, geolocation: {"type"=>"Point", "coordinates"=>[-122.2711137, 37.8043637]}, street_number: nil, street_name: nil, city: "Oakland", postal_code: nil, county: "Alameda County", state: "CA", country: "US">
     ```

   * 然后我们使用```creae_place_of_birth```属性来创建**actor**中的**place_of_birth**

     ```ruby
     irb(main):005:0> actor.create_place_of_birth(oakland.attributes)
     => #<Place _id: Oakland, CA, USA, formatted_address: nil, geolocation: {:type=>"Point", :coordinates=>[-122.2711137, 37.8043637]}, street_number: nil, street_name: nil, city: "Oakland", postal_code: nil, county: "Alameda County", state: "CA", country: "US">
     ```

     这个create_很神奇，会自动将一个建立两个对象之间的embed relationship

   * 最后再次使用find来验证我们的地址被存进去了(**对比第一步**)

     ```ruby
     irb(main):006:0> pp Actor.collection.find(:_id => actor.id).first
     {"_id"=>"nm0012524",
      "bio"=>
       "Frank Adu is an actor, known for Taxi Driver (1976), Guerre et amour (1975) and Across 110th Street (1972).",
      "name"=>"Frank Adu",
      "place_of_birth"=>
       {"_id"=>"Oakland, CA, USA",
        "geolocation"=>{"type"=>"Point", "coordinates"=>[-122.2711137, 37.8043637]},
        "city"=>"Oakland",
        "county"=>"Alameda County",
        "state"=>"CA",
        "country"=>"US"}}
     => {"_id"=>"nm0012524", "bio"=>"Frank Adu is an actor, known for Taxi Driver (1976), Guerre et amour (1975) and Across 110th Street (1972).", "name"=>"Frank Adu", "place_of_birth"=>{"_id"=>"Oakland, CA, USA", "geolocation"=>{"type"=>"Point", "coordinates"=>[-122.2711137, 37.8043637]}, "city"=>"Oakland", "county"=>"Alameda County", "state"=>"CA", "country"=>"US"}}
     ```

2. 关于polymorphic的Demo(演示方法即将刚刚存的Oakland现在存入到一个writer里面。

   * 找到一个没有**hometown**属性的**writer**对象

     ```ruby
     irb(main):007:0> writer=Writer.where(:hometown => {:$exists => 0}).first
     irb(main):011:0> pp Writer.collection.find(:_id => writer.id).first
     {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"]}
     => {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"]}
     ```

     可以看到这个**writer**没有任何hometown属性

   * 将上个Demo中的Oakland赋值给这个writer

     ```ruby
     irb(main):016:0> oakland = Place.where(:city=>"Oakland").first
     => #<Place _id: Oakland, CA, USA, formatted_address: nil, geolocation: {"type"=>"Point", "coordinates"=>[-122.2711137, 37.8043637]}, street_number: nil, street_name: nil, city: "Oakland", postal_code: nil, county: "Alameda County", state: "CA", country: "US">

     irb(main):017:0> writer.create_hometown(oakland.attributes)
     => #<Place _id: Oakland, CA, USA, formatted_address: nil, geolocation: {:type=>"Point", :coordinates=>[-122.2711137, 37.8043637]}, street_number: nil, street_name: nil, city: "Oakland", postal_code: nil, county: "Alameda County", state: "CA", country: "US">
     ```

   * 重新查找并测试一下

     ```ruby
     irb(main):019:0> pp Writer.collection.find(:_id => writer.id).first
     {"_id"=>"nm0000005",
      "name"=>"Ingmar Bergman",
      "movie_ids"=>["tt0060827"],
      "hometown"=>
       {"_id"=>"Oakland, CA, USA",
        "geolocation"=>{"type"=>"Point", "coordinates"=>[-122.2711137, 37.8043637]},
        "city"=>"Oakland",
        "county"=>"Alameda County",
        "state"=>"CA",
        "country"=>"US"}}
     => {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"], "hometown"=>{"_id"=>"Oakland, CA, USA", "geolocation"=>{"type"=>"Point", "coordinates"=>[-122.2711137, 37.8043637]}, "city"=>"Oakland", "county"=>"Alameda County", "state"=>"CA", "country"=>"US"}}
     ```

3. 我们还可以通过```writer.methods.grep /hometown/```来查看writer中的hometown有哪些内置的方法

   ```ruby
   irb(main):021:0> writer.methods.grep /hometown/
   => [:hometown, :hometown=, :hometown?, :has_hometown?, :build_hometown, :create_hometown]
   ```

   可以通过设置属性为nil来删除embed relationship

   ```shell
   irb(main):027:0> writer.create_hometown ##用来创建空的hometow
   => #<Place _id: , formatted_address: nil, geolocation: nil, street_number: nil, street_name: nil, city: nil, postal_code: nil, county: nil, state: nil, country: nil>
   irb(main):028:0> writer.has_hometown? 
   => true
   irb(main):029:0> writer.hometown=nil  #设置为空，即删除属性
   => nil
   irb(main):030:0> writer.has_hometown? #再次查看发现属性被删除
   => false
   irb(main):031:0> pp Writer.collection.find(:_id => writer.id).first  #通过查询发现完全被删除了
   {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"]}
   => {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"]}
   ```

4. 最重要的一点**simple assignment**不会产生embed relationship

   ```ruby
   irb(main):035:0* writer.hometown=oakland
   => #<Place _id: Oakland, CA, USA, formatted_address: nil, geolocation: #<Point:0x0055d8b7ebe638 @longitude=-122.2711137, @latitude=37.8043637>, street_number: nil, street_name: nil, city: "Oakland", postal_code: nil, county: "Alameda County", state: "CA", country: "US">
   #在简单的赋值之后迅速查看可以看到地址目前存在
   irb(main):036:0> writer.hometown
   => #<Place _id: Oakland, CA, USA, formatted_address: nil, geolocation: #<Point:0x0055d8b7ebe638 @longitude=-122.2711137, @latitude=37.8043637>, street_number: nil, street_name: nil, city: "Oakland", postal_code: nil, county: "Alameda County", state: "CA", country: "US">
   #但是当我们从整个Writer collection之中搜索的时候会发现这个属性不见了
   irb(main):039:0* pp Writer.collection.find(:_id => writer.id).first
   {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"]}
   => {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"]}
   #我们尝试一下保存这个writer再查询，发现还是没有
   irb(main):040:0> writer.save
   => true
   irb(main):041:0> pp Writer.collection.find(:_id => writer.id).first
   {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"]}
   => {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "movie_ids"=>["tt0060827"]}
   ```

   总结，要有embed relationship必须使用**create_hometown**这个方法