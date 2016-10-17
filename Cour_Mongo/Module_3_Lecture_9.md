### 1:1 Linked Relationship

---

* 1:1 Linked - has_one
* Recursive Relationship
* Demo

---

#### Referenced 1-1: has_one

- 这是一个一对一的关系

- Children**被**Parent document所引用

- 引用使用**has_one**和**belongs_to**两个宏

  **Linked (Movie -> sequel_to: Movie)**

  ```ruby
  class Movie
    include Mongoid::Document
    field :title, type: String
    ...
    has_one :sequel, class_name:"Movie"
    belongs_to :sequel_to, class_name:"Movie"
    ...
  end
  ```

  这个是一个比较interesting的例子，通常情况下Parent document和Child document是不同的**Class**，但这个例子中的Parent document和Child document是同一个类。

  Movie有个续集（**sequel**），这个**sequel**是属于**Movie**类的， Movie同时是其他的movie的续集，**sequel_to**。

  Rocky25又个sequel是Rocky26，反之，Rocky26属于Rocky25

---

#### Demo

1. 找到两部电影，rocky25和rocky26

   ```ruby
   irb(main):002:0> rocky25 = Movie.where(:title => "Rocky xxv").first
   => #<Movie _id: tt9000000, created_at: 2016-10-16 15:02:26 UTC, updated_at: 2016-10-16 15:02:26 UTC, title: "Rocky xxv", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>

   irb(main):003:0> rocky26 = Movie.where(:title => "Rocky xxvi").first
   => #<Movie _id: tt9000001, created_at: 2016-10-16 15:24:44 UTC, updated_at: 2016-10-16 15:24:44 UTC, title: "Rocky xxvi", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
   ```

2. 建立rocky26与rocky25的**sequel_to**关系

   ```ruby
   irb(main):007:0* rocky26.sequel_to = rocky25
   => #<Movie _id: tt9000000, created_at: 2016-10-16 15:02:26 UTC, updated_at: 2016-10-16 15:02:26 UTC, title: "Rocky xxv", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>

   irb(main):008:0> rocky26.save
   => true
   ```

3. 使用pp来检查一下刚刚建立的relationship

   ```ruby
   irb(main):009:0> pp Movie.collection.find(:title => "Rocky xxvi").first
   {"_id"=>"tt9000001",
    "title"=>"Rocky xxvi",
    "updated_at"=>2016-10-17 15:06:31 UTC,
    "created_at"=>2016-10-16 15:24:44 UTC,
    "roles"=>
     [{"_id"=>"stallone.id",
       "character"=>"Rocky",
       "actorName"=>"Sly",
       "main"=>true},
      {"_id"=>"nm0000006",
       "character"=>"Challenger",
       "main"=>false,
       "actorName"=>"Ingrid Bergman"}],
    "sequel_of"=>"tt9000000"}
   => {"_id"=>"tt9000001", "title"=>"Rocky xxvi", "updated_at"=>2016-10-17 15:06:31 UTC, "created_at"=>2016-10-16 15:24:44 UTC, "roles"=>[{"_id"=>"stallone.id", "character"=>"Rocky", "actorName"=>"Sly", "main"=>true}, {"_id"=>"nm0000006", "character"=>"Challenger", "main"=>false, "actorName"=>"Ingrid Bergman"}], "sequel_of"=>"tt9000000"}
   ```

4. 如果pp parent document的话并不会看到有改变

   ```ruby
   irb(main):011:0> pp Movie.where(:title => "Rocky xxv").first
   #<Movie _id: tt9000000, created_at: 2016-10-16 15:02:26 UTC, updated_at: 2016-10-16 15:02:26 UTC, title: "Rocky xxv", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
   => #<Movie _id: tt9000000, created_at: 2016-10-16 15:02:26 UTC, updated_at: 2016-10-16 15:02:26 UTC, title: "Rocky xxv", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
   ```

5. 但是通过rocky26确实是可以进行双边查询的

   ```ruby
   irb(main):012:0> rocky25.sequel.title
   => "Rocky xxvi"
   irb(main):013:0> rocky26.sequel_to.title
   => "Rocky xxv"
   irb(main):014:0> rocky26.sequel_to.sequel.title
   => "Rocky xxvi"
   ```

   ​

