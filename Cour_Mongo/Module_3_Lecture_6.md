### M:1 Linked

---

#### Relationship - belongs_to

* Children可以存储在一个和parent**分开**的collection之中

* Child uses *belongs_to* and parent **optionally** uses *has_many*

* 例子：假设director是child，residence是parent，那么多个director可以共同拥有一个residence，比如洛杉矶、奥克兰。

  * **如果没有has_many**,那么Child和Parent的关系会变为uni-directional,如上文的例子，如果缺失has_many, director可以找到residence，但是residence不能返找回director。

    **M:1 Linked (Director -> residence:Place)**

    diretor has a bunch of director fields but belongs to the residence:Place

    ```ruby
    class Director
      include Mongoid::Document
      ...
      belongs_to :residence, class_name: 'Place'
    end	
    ```

---

### Demo

1. 首先寻找一个没有residence属性的director

   ```ruby
   irb(main):002:0> director = Director.collection.find(:residence => {:$exists => 0}).first
   => {"_id"=>"nm0000005", "name"=>"Ingmar Bergman"}

   irb(main):007:0> pp director
   {"_id"=>"nm0000005", "name"=>"Ingmar Bergman"}
   => {"_id"=>"nm0000005", "name"=>"Ingmar Bergman"}
   ```

2. 将oakland这个Place赋予给director的residence

   ```ruby
   irb(main):002:0> oakland = Place.where(:city => "Oakland").first
   => #<Place _id: Oakland, CA, USA, formatted_address: nil, geolocation: {"type"=>"Point", "coordinates"=>[-122.2711137, 37.8043637]}, street_number: nil, street_name: nil, city: "Oakland", postal_code: nil, county: "Alameda County", state: "CA", country: "US">
   irb(main):005:0> director.residence = oakland
   => #<Place _id: Oakland, CA, USA, formatted_address: nil, geolocation: {"type"=>"Point", "coordinates"=>[-122.2711137, 37.8043637]}, street_number: nil, street_name: nil, city: "Oakland", postal_code: nil, county: "Alameda County", state: "CA", country: "US">

   irb(main):006:0> director.save
   => true

   irb(main):007:0> pp Director.collection.find(:_id => director.id).first
   => {"_id"=>"nm0000005", "name"=>"Ingmar Bergman", "residence_id"=>"Oakland, CA, USA", "updated_at"=>2016-10-16 14:01:53 UTC}
   ```

---

有一点需要注意的是有关联的两个object现在处于不同的collection，director现在只在本地存储了一个相关联object的foreign key，每次查询详细信息都会query两次，除非使用```residence_id```这个method

```ruby
irb(main):008:0> director.residence.state#会query两次
=> "CA"
irb(main):009:0> director.residence_id#query一次
=> "Oakland, CA, USA"
```