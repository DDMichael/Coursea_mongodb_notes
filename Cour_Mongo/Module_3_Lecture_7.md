### 1:M Embedded relationship

---

* 概念

  * 处于1:M关系中的Parent document使用**embeds_many**说明他有n个被嵌入的children.

    * Movie -> Movierole, 一个movie里可以有很多的角色。

  * children元素则使用**embedded_in**

    ```ruby
    class Movie
      include Mongoid::Document
      field :title, type: String
      ...
      embeds_many :roles, class_name:"Movierole"
      ...
    end

    class MovieRole
      include Mongoid::Document
      field :character, type: String
      field :actorName, as: :actor_name, type: String
      field :main, type: Mongoid::Boolean
      ...
      embedded_in :movie
      ...
    end
    ```

---

#### Demo

1. 创建一个**Movie**对象

   ```ruby
   irb(main):010:0> rocky25 = Movie.create(:_id => "tt9000000", :title => "Rocky xxv")
   => #<Movie _id: tt9000000, created_at: 2016-10-16 15:02:26 UTC, updated_at: 2016-10-16 15:02:26 UTC, title: "Rocky xxv", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
   ```

2. 在Actor中找到一个叫Stallone的演员(返回信息就不写了，太长了)

   ```ruby
   irb(main):011:0> stallone = Actor.where(:name=>{:$regex=>"Stallone"}).first
   ```

3. 将找到的**Actor**对象赋予给**Movie.roles**里，并查看

   ```ruby
   irb(main):015:0* rocky = rocky25.roles.create(:_id => stallone.id, :character=>"Rocky", :actorName=>"Sly", :main=>true)
   => #<MovieRole _id: nm0000230, character: "Rocky", actorName(actor_name): "Sly", main: true, urlCharacter(url_character): nil, urlPhoto(url_photo): nil, urlProfile(url_profile): nil>

   irb(main):017:0> pp Movie.collection.find(:title => "Rocky xxv").first
   {"_id"=>"tt9000000",
    "title"=>"Rocky xxv",
    "updated_at"=>2016-10-16 15:02:26 UTC,
    "created_at"=>2016-10-16 15:02:26 UTC,
    "roles"=>
     [{"_id"=>"nm0000230",
       "character"=>"Rocky",
       "actorName"=>"Sly",
       "main"=>true}]}
   => {"_id"=>"tt9000000", "title"=>"Rocky xxv", "updated_at"=>2016-10-16 15:02:26 UTC, "created_at"=>2016-10-16 15:02:26 UTC, "roles"=>[{"_id"=>"nm0000230", "character"=>"Rocky", "actorName"=>"Sly", "main"=>true}]}
   ```

4. **添加一个新的role**，首先找一个Actor中的random对象

   ```ruby
   irb(main):018:0> actor = Actor.first
   ```

5. 创建新的MovieRole object，并赋予Actor中的id和其他field

   ```ruby
   irb(main):023:0* role = MovieRole.new
   => #<MovieRole _id: , character: nil, actorName(actor_name): nil, main: nil, urlCharacter(url_character): nil, urlPhoto(url_photo): nil, urlProfile(url_profile): nil>
   irb(main):024:0> role.id = actor.id
   => "nm0000006"
   irb(main):025:0> role.character="Challenger"
   => "Challenger"
   irb(main):026:0> role.main=false
   => false
   irb(main):027:0> role.actor_name=actor.name
   => "Ingrid Bergman"
   ```

6. 最后append这个role到rocky25之中

   ```ruby
   irb(main):028:0> rocky25.roles << role
   ```

7. 查看，能看到有两个演员

   ```ruby
   irb(main):029:0> pp Movie.collection.find(:title=>"Rocky xxv").first
   {"_id"=>"tt9000000",
    "title"=>"Rocky xxv",
    "updated_at"=>2016-10-16 15:02:26 UTC,
    "created_at"=>2016-10-16 15:02:26 UTC,
    "roles"=>
     [{"_id"=>"nm0000230",
       "character"=>"Rocky",
       "actorName"=>"Sly",
       "main"=>true},
      {"_id"=>"nm0000006",
       "character"=>"Challenger",
       "main"=>false,
       "actorName"=>"Ingrid Bergman"}]}
   => {"_id"=>"tt9000000", "title"=>"Rocky xxv", "updated_at"=>2016-10-16 15:02:26 UTC, "created_at"=>2016-10-16 15:02:26 UTC, "roles"=>[{"_id"=>"nm0000230", "character"=>"Rocky", "actorName"=>"Sly", "main"=>true}, {"_id"=>"nm0000006", "character"=>"Challenger", "main"=>false, "actorName"=>"Ingrid Bergman"}]}
   ```

#### Demo2: Create a collection before save

1. 创建一个新的**Movie**对象

   ```ruby
   irb(main):030:0> rocky26 = Movie.new(:_id => "tt9000001", :title => "Rocky xxvi")
   => #<Movie _id: tt9000001, created_at: nil, updated_at: nil, title: "Rocky xxvi", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
   ```

2. 使用build指令添加stallone和上个demo中的第二个actor(role)

   ```ruby
   irb(main):031:0> rocky = rocky26.roles.build(:_id => "stallone.id", :character=>"Rocky", :actorName=>"Sly", :main=>true)
   => #<MovieRole _id: stallone.id, character: "Rocky", actorName(actor_name): "Sly", main: true, urlCharacter(url_character): nil, urlPhoto(url_photo): nil, urlProfile(url_profile): nil>

   irb(main):034:0* rocky26.roles << role
   => [#<MovieRole _id: stallone.id, character: "Rocky", actorName(actor_name): "Sly", main: true, urlCharacter(url_character): nil, urlPhoto(url_photo): nil, urlProfile(url_profile): nil>, #<MovieRole _id: nm0000006, character: "Challenger", actorName(actor_name): "Ingrid Bergman", main: false, urlCharacter(url_character): nil, urlPhoto(url_photo): nil, urlProfile(url_profile): nil>]
   ```

3. 查看这个**Movie**对象

   ```ruby
   irb(main):035:0> pp Movie.collection.find(:title => "Rocky xxvi").first
   => nil
   ```

4. 发现**roles**里没东西，这是因为没有save，save之后再次查看

   ```ruby
   irb(main):036:0> rocky26.save
   => true
   irb(main):037:0> pp Movie.collection.find(:title => "Rocky xxvi").first
   {"_id"=>"tt9000001",
    "title"=>"Rocky xxvi",
    "updated_at"=>2016-10-16 15:24:44 UTC,
    "created_at"=>2016-10-16 15:24:44 UTC,
    "roles"=>
     [{"_id"=>"stallone.id",
       "character"=>"Rocky",
       "actorName"=>"Sly",
       "main"=>true},
      {"_id"=>"nm0000006",
       "character"=>"Challenger",
       "main"=>false,
       "actorName"=>"Ingrid Bergman"}]}
   => {"_id"=>"tt9000001", "title"=>"Rocky xxvi", "updated_at"=>2016-10-16 15:24:44 UTC, "created_at"=>2016-10-16 15:24:44 UTC, "roles"=>[{"_id"=>"stallone.id", "character"=>"Rocky", "actorName"=>"Sly", "main"=>true}, {"_id"=>"nm0000006", "character"=>"Challenger", "main"=>false, "actorName"=>"Ingrid Bergman"}]}

   ```

---

**另一个有趣的东西是：Embedded object can provide a reference to its parent document**

```ruby
irb(main):038:0> role.movie.title
=> "Rocky xxvi"
```

通过role找到movie再介入到title