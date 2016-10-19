### M:M Linked Relationship

---

#### References M:M - has_and_belongs_to_many

* M:M relationships中两边的document是分开存储在不同的collection之中的

  * 一个电影由三个作者编写，那么三个作者的primary key就被当作foreign key存储在一个电影中，同时，一个作者可以有多个电影作品，那么多个电影的primary key会被当作foreign key存储在wrtier之中

* Parent和Child都需要用**has_and_belongs_to_many**这个宏

* Foreign keys ID会在两边存储为arrays形式

  ```ruby
  class Writer
    include Mongoid::Document
    field :name, type: String
    embeds_one :hometown, as: :locatable, class_name: 'Place'
    ...
    has_and_belongs_to_many :movies
    ...
  end

  ## "belongs to" stores the foreign key to the parent document
  ## "has many" means it is the parent of many child documents
  ```

  ```ruby
  class Movie
    include Mongoid::Document
    field :title, type: String
    ...
    has_and_belongs_to_many :writer
    ...
  end
  ```

---

#### Demo

从writer角度navigating movie来看movie的properties.

1. 寻找一个叫‘stone’的writer,可以看到有两个movie_id: ["tt0086250", "tt0091763"], 同时可以通过title将这些电影的title全部列出来

   ```ruby
   main):001:0> stone = Writer.where(:name=>{:$regex=>"Stone"}).first
   => #<Writer _id: nm0000231, created_at: nil, updated_at: nil, name: "Oliver Stone", movie_ids: ["tt0086250", "tt0091763"]>

   (main):002:0> stone.movies.map{|m| m.title}
   => ["Scarface", "Platoon"]
   ```

2. 对**writer**增加一个**hometown**属性然后从**movie**中通过reference找到hometown

   1. 找一个writer

      ```ruby
      irb(main):003:0> stone = Writer.where(:name => {:$regex => "Stone"}).first
      => #<Writer _id: nm0000231, created_at: nil, updated_at: nil, name: "Oliver Stone", movie_ids: ["tt0086250", "tt0091763"]>
      ```

   2. 找一个城市，nyc

      ```ruby
      irb(main):004:0> nyc = Place.where(:_id => {:$regex =>"^New York, NY"}).first
      => #<Place _id: New York, NY, USA, formatted_address: nil, geolocation: {"type"=>"Point", "coordinates"=>[-74.0059413, 40.7127837]}, street_number: nil, street_name: nil, city: "NY", postal_code: nil, county: nil, state: "NY", country: "US">
      ```

   3. 创建hometown 属性

      ```ruby
      irb(main):005:0> stone.create_hometown(nyc.attributes)
      => #<Place _id: New York, NY, USA, formatted_address: nil, geolocation: {:type=>"Point", :coordinates=>[-74.0059413, 40.7127837]}, street_number: nil, street_name: nil, city: "NY", postal_code: nil, county: nil, state: "NY", country: "US">
      irb(main):006:0> platoon = Movie.where(:title => "Platoon").first
      ```

   4. 查找platoon这个电影，并通过最后面的writer id来反向查找这个writer的hometown

      ```ruby
      irb(main):006:0> platoon = Movie.where(:title => "Platoon").first #这个太长了，先不放结果了

      irb(main):010:0> platoon.writers.first.hometown.id
      => "New York, NY, USA"
      ```

3. 创建一个新的电影，这时新的电影并没有writer，通过这个来建立M：M关系

   1. 创建一个**Movie** 对象叫做rocky26

      ```ruby
      irb(main):005:0> rocky26 = Movie.create(:_id=>"tt9000001", :title => "Rocky XXVI")
      => #<Movie _id: tt9000001, created_at: 2016-10-19 08:29:35 UTC, updated_at: 2016-10-19 08:29:35 UTC, title: "Rocky XXVI", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
      irb(main):006:0> pp Movie.collection.find(:_id => rocky26.id).first
      {"_id"=>"tt9000001",
       "title"=>"Rocky XXVI",
       "updated_at"=>2016-10-19 08:29:35 UTC,
       "created_at"=>2016-10-19 08:29:35 UTC}
      => {"_id"=>"tt9000001", "title"=>"Rocky XXVI", "updated_at"=>2016-10-19 08:29:35 UTC, "created_at"=>2016-10-19 08:29:35 UTC}
      ```

   2. 找到**stone**这个writer

      ```ruby
      irb(main):007:0> stone = Writer.where(:name => {:$regex => "Stone"}).first
      => #<Writer _id: nm0000231, created_at: nil, updated_at: nil, name: "Oliver Stone", movie_ids: ["tt0086250", "tt0091763"]>
      ```

   3. 将stone加入到rocky26之中

      ```ruby
      irb(main):008:0> rocky26.writers << stone
      => [#<Writer _id: nm0000231, created_at: nil, updated_at: 2016-10-19 08:35:23 UTC, name: "Oliver Stone", movie_ids: ["tt0086250", "tt0091763", "tt9000001"]>]
      ```

   4. 从writer角度查看movie（id）

      ```ruby
      irb(main):010:0> pp Writer.collection.find(:_id => stone.id).first
      {"_id"=>"nm0000231",
       "name"=>"Oliver Stone",
       "movie_ids"=>["tt0086250", "tt0091763", "tt9000001"],
       "hometown"=>
        {"_id"=>"New York, NY, USA",
         "geolocation"=>{"type"=>"Point", "coordinates"=>[-74.0059413, 40.7127837]},
         "city"=>"NY",
         "state"=>"NY",
         "country"=>"US"},
       "updated_at"=>2016-10-19 08:35:23 UTC}
      => {"_id"=>"nm0000231", "name"=>"Oliver Stone", "movie_ids"=>["tt0086250", "tt0091763", "tt9000001"], "hometown"=>{"_id"=>"New York, NY, USA", "geolocation"=>{"type"=>"Point", "coordinates"=>[-74.0059413, 40.7127837]}, "city"=>"NY", "state"=>"NY", "country"=>"US"}, "updated_at"=>2016-10-19 08:35:23 UTC}
      ```

   5. 从movie角度查看writer（id）

      ```ruby
      irb(main):009:0> pp Movie.collection.find(:_id => rocky26.id).first
      {"_id"=>"tt9000001",
       "title"=>"Rocky XXVI",
       "updated_at"=>2016-10-19 08:29:35 UTC,
       "created_at"=>2016-10-19 08:29:35 UTC,
       "writer_ids"=>["nm0000231"]}
      => {"_id"=>"tt9000001", "title"=>"Rocky XXVI", "updated_at"=>2016-10-19 08:29:35 UTC, "created_at"=>2016-10-19 08:29:35 UTC, "writer_ids"=>["nm0000231"]}
      ```

   6. 列出来所有movie的title，之前是两个，现在应该是三个

      ```ruby
      irb(main):011:0> stone.movies.map{|m| m.title}
      => ["Scarface", "Platoon", "Rocky XXVI"]
      ```

   7. 从movie角度列出writer，现在只有一个writer

      ```ruby
      irb(main):012:0> rocky26.writers.map{|m| m.name}
      => ["Oliver Stone"]
      ```

   8. 现在看看delete这个功能（一边被删除了另一边也会被删除）

      ```ruby
      irb(main):014:0> stone.movies.delete rocky26
      => #<Movie _id: tt9000001, created_at: 2016-10-19 08:29:35 UTC, updated_at: 2016-10-19 08:41:08 UTC, title: "Rocky XXVI", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: [], sequel_of: nil>
      ```

   9. 检查一下writer写了几个电影(原本三个现在应为两个)

      ```ruby
      irb(main):015:0> pp Writer.collection.find(:_id => stone.id).first
      {"_id"=>"nm0000231",
       "name"=>"Oliver Stone",
       "movie_ids"=>["tt0086250", "tt0091763"],
       "hometown"=>
        {"_id"=>"New York, NY, USA",
         "geolocation"=>{"type"=>"Point", "coordinates"=>[-74.0059413, 40.7127837]},
         "city"=>"NY",
         "state"=>"NY",
         "country"=>"US"},
       "updated_at"=>2016-10-19 08:35:23 UTC}
      => {"_id"=>"nm0000231", "name"=>"Oliver Stone", "movie_ids"=>["tt0086250", "tt0091763"], "hometown"=>{"_id"=>"New York, NY, USA", "geolocation"=>{"type"=>"Point", "coordinates"=>[-74.0059413, 40.7127837]}, "city"=>"NY", "state"=>"NY", "country"=>"US"}, "updated_at"=>2016-10-19 08:35:23 UTC}
      ```

   10. 检查一下rocky26有没有writer id（现在应为nil）

       ```ruby
       irb(main):016:0> pp Movie.collection.find(:_id => rocky26.id).first
       => {"_id"=>"tt9000001", "title"=>"Rocky XXVI", "updated_at"=>2016-10-19 08:41:08 UTC, "created_at"=>2016-10-19 08:29:35 UTC, "writer_ids"=>[]}
       ```

       ​