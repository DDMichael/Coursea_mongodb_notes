### Setup - Data files

* movies.json
* actors.json
* directors.json
* writers.json
* places.json

上述的文件全都存储在db这个folder中

---

#### Initialization

* 导入数据

  ```rake db:seed```


* 建立一个Index(为的是后面的Geospatial query)

  ```rake db:mongoid:create_indexes```

#### Demo

```shell
aomingdong@ubuntu:~/Documents/Mongodb/module3/fullstack-course3-module3-movies$ rake db:mongoid:create_indexes
D, [2016-10-13T14:14:28.521304 #2148] DEBUG -- : MONGODB | Adding localhost:27017 to the cluster.
D, [2016-10-13T14:14:28.523660 #2148] DEBUG -- : MONGODB | localhost:27017 | movies_development.createIndexes | STARTED | {"createIndexes"=>"actors", "indexes"=>[{:key=>{:"place_of_birth.geolocation"=>"2dsphere"}, :name=>"place_of_birth.geolocation_2dsphere"}]}
D, [2016-10-13T14:14:28.524106 #2148] DEBUG -- : MONGODB | localhost:27017 | movies_development.createIndexes | SUCCEEDED | 0.000357531s
```

可以看到geolocation_2dsphere index在place_of_birth里面

```ruby
class Actor
  include Mongoid::Document
  include Mongoid::Timestamps

  field :name, type: String
  field :birthName, as: :birth_name, type: String
  field :date_of_birth, type: Date
  field :height, type: Measurement
  field :bio, type: String

  embeds_one :place_of_birth, class_name: 'Place' , as: :locatable

  index ({ :"place_of_birth.geolocation" => Mongo::Index::GEO2DSPHERE })

```

---

### Model Types and Document Representation

* Custom Types
  * Measurement - represents measurement info
  * Point -  represents geolocation points
* 什么是Custom class？
  * Custom class是一些独立的class，里面有很多data，这些class里面没有:_id这个属性。所以custom class并不是一个Mongoid Document
  * Custom class的设立是为了方便query

---

* Document Model Class
  * **Place** - abstraction added to Point to hold location information about the geolocation point
  * **Actor** - represents someone who plays a role in a Movie
  * **Writer** - one of the authors of a Movie
  * **Director** - one of the directors of a Movie
  * **Movie** - core information about a Movie

这些model基本上都是一样的，唯一的区别是他们的relationship的关系

比如：

* Writer

  ```ruby
  class Writer
    include Mongoid::Document
    include Mongoid::Timestamps

    field :name, type: String

    embeds_one :hometown, as: :locatable, class_name: 'Place'
    has_and_belongs_to_many :movies
  end
  ```

  能看到writer又一个和movies的many to many relationship

* DirectorRef - is an annotated reference to a Director

  * used to cache stable/core director information that the referenceing document/view will need.
  * MovieRole - is a character in a **Movie** played by an **Actor**

---

#### Relationships - Types

* 1:1 Embedded (Actor -> place_of_birth:Place)
* M:1 Linked (Director -> residence:Place)
* 1:M Embedded (Movie <-> roles:MovieRole)
* M:1 Embedded Linked (MovieRole <-> Actor)
* 1:1 Linked (Movie -> sequel_to:Movie)
* M:M (Movie <-> writers:Writer)