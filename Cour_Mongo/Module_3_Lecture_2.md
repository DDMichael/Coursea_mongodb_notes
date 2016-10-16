### Document

1. Documents是Mogoid之中的**核心**对象
   * Mongoid::Document
2. Documents可以被存储在一个**collection**或者嵌入在**other documents**之中

```ruby
class Movie
  include Mongoid::Document
end
```

3. **Documents**有**fields**这个东西，**fields**就是是**Documents**的属性

---

### Fields

1. Fields 是属性
2. field
3. type - 默认情况下是String
4. 这里有一个例子，这个Movie下面又四个属性。

```ruby
class Movie
  include Mongoid::Document
  field :title, type: String
  field :type, type: String
  field :rated, type: String
  field :year, type: Integer
end
```

5. 建立一个模型的话可以使用``` rails g model```, 但是需要制定以下类型

   1. ```ruby
      rails g model Movie title type rated year:integer release_date:date ......
        ##这里声称了title、type、rated、year、release_date等等，其中year的类型是integer, release_date的类型是date
      ```

   2. ``` ruby
      rails g model Actor name birth_name data_of_birth:Date height:Measurement bio:text
      ```

---

### Field Types

| Array      | Boolean | DateTime     | Hash    |
| ---------- | ------- | ------------ | ------- |
| BigDecimal | Date    | Float        | Integer |
| BSON       | Range   | Regexp       | String  |
| Symbol     | Time    | TimeWithZone |         |

---

### Timestamps

1. Timestamp并不是默认加入Mongoid的
2. **created_at** and **updated_at**属性可以被*Mongoid::Timestamps*的mixin来实现
3. touch - Will update the document's **updated_at** timestamp

```ruby
class Movie
  include Mongoid::Document
  include Mongoid::Timestamps
  ...
end
```

#### Demo

1. 首先建立一个新的movie对象叫做rocky

```shell
irb(main):003:0> rocky=Movie.create(:_id=>"tt900031", :title=>"Rocky XXVI")

=> #<Movie _id: tt900031, created_at: 2016-10-13 09:50:11 UTC, updated_at: 2016-10-13 09:50:11 UTC, title: "Rocky XXVI", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>

```

2. 调用created_at和updated_at

```shell
irb(main):006:0> rocky.created_at
=> Thu, 13 Oct 2016 09:50:11 UTC +00:00
irb(main):007:0> rocky.updated_at
=> Thu, 13 Oct 2016 09:50:11 UTC +00:00
```

3. 使用updated_at来更新时间，有两种方法

   1. ```shell
      irb(main):008:0> rocky.year = 2015
      => 2015
      irb(main):009:0> rocky.updated_at
      => Thu, 13 Oct 2016 09:50:11 UTC +00:00
      irb(main):010:0> rocky.save
      irb(main):011:0> rocky.created_at
      => Thu, 13 Oct 2016 09:50:11 UTC +00:00
      irb(main):012:0> rocky.updated_at
      => Thu, 13 Oct 2016 09:54:50 UTC +00:00
      ```

   2. ```shell
      irb(main):013:0> rocky.touch
      irb(main):014:0> rocky.updated_at
      => Thu, 13 Oct 2016 09:55:15 UTC +00:00
      ```

---

### Field Aliases

```ruby
class Actor
  include Mongoid::Document
  include Mongoid::Timestamps

  field :name, type: String
  field :birthName, as: :birth_name, type: String
  field :date_of_birth, type: Date
  field :height, type: Measurement
  field :bio, type: String
```

在这里**birthName** in the document mapped to **birth_name** in the model

符合rails的命名原理，同时帮助后期的压缩

---

### Custom Fields

1. 可以在Mongoid之中自定义类型then determine how they are **serialized** and **deserialized**

2. 5 methods in total

   1. initialize
   2. mongoize (instance method)
   3. mongoize, demongoize, evolve (class methods)

3. Example: *Measurement*

   ```ruby
   :runtime => {:amount=>60, :units=>"min"}
   ```

   ```ruby
   #creates a DB-form of the instance 将instance object变成Database形式
     def mongoize
       @units ? {:amount => @amount, :units => @units} : {:amount => @amount}
     end
   #creates an instance of the class from the DB-form of the data 将Database形式变成instance object
     def self.demongoize(object)
       case object
       when Hash then Measurement.new(object[:amount], object[:units])
       else nil
       end
     end
   #used by criteria to convert object to DB-friendly form
     def self.evolve(object)
       case object
       when Measurement then object.mongoize
       else object
       end
     end
   ```

---

### store_in

1. Application type to Document type mapping
2. Locations gets stored into "places" colleciton
3. 其实就是可以将一个类存到其他的类里面

