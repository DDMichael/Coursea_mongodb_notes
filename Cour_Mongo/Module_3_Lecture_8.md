### M:1 Relationship

---

* M:1 Embedded link(Anooated Link)

* Many actors in a movie, but each play a specific role

  每个电影都有一些特定的角色，这些role是固定的，不变的，不会被Actor信息的变动而影响（**死亡、结婚**等等）

* Movie Role <—> Actor

  ```ruby
  class MovieRole
    include Mongoid::Document
    field :character, type: String
    field :actorName, as: :actor_name, type: String
    field :main, type: Mongoid::Boolean
    embedded_in :movie
    ...
    belongs_to :actor, :foreign_key => :_id
    ...
  end
  ```

  由于**belongs_to** actor,所以在想获得更多actor的信息的时候可以直接调用foreign_key来寻找actor的信息

* 在**Actor**类之中我们可以声明M:1 Embedded Linked (MovieRole <-> Actor)

  ```ruby
  class Actor
    include Mongoid::Document
    field :name, type: String
    def roles
    
    def roles
      Movie.where(:"roles._id"=>self.id)
      	 .map {|m| m.roles.where(:_id => self.id).first}
    end
  end
  ```

  这个roles method意思是：

  进入到Movie这个collection，遍历所有的电影，找到某电影中角色的:_id等于self.id，然后返回.

---

1. 在Actor类中找一个演员，这里是Matt Damon

   ```ruby
   irb(main):039:0> damon = Actor.where(:name => {:$regex=>"Matt Da"}).first
   ##信息太多了，在这里就暂时删掉了
   ```


1. 在Movie中通过nested search找到roles._id等于damon的id的movie

   ```ruby
   irb(main):041:0> movie = Movie.where(:"roles._id"=>damon.id).first
   => #<Movie _id: tt0119217, created_at: nil, updated_at: nil, title: "Good Will Hunting", type: "Movie", rated: "R", year: 1997, release_date: 1998-01-09 00:00:00 UTC, runtime: {"amount"=>126, "units"=>"min"}, votes: 552171, countries: ["USA"], languages: ["English"], genres: ["Drama"], filmingLocations(filming_locations): ["Massachusetts Turnpike", "Massachusetts", "USA"], metascore: "70/100", simplePlot(simple_plot): "Will Hunting, a janitor at M.I.T., has a gift for mathematics, but needs help from a psychologist to find direction in his life.", plot: "A touching tale of a wayward young man who struggles to find his identity, living in a world where he can solve any problem, except the one brewing deep within himself, until one day he meets his soul mate who opens his mind and his heart.", urlIMDB(url_imdb): "http://www.imdb.com/title/tt0119217", urlPoster(url_poster): "http://ia.media-imdb.com/images/M/MV5BMTk0NjY0Mzg5MF5BMl5BanBnXkFtZTcwNzM1OTM2MQ@@._V1_SY317_CR1,0,214,317_AL_.jpg", directors: [{"name"=>"Gus Van Sant", "_id"=>"nm0001814"}], actors: nil, writer_ids: ["nm0000354", "nm0000255"], sequel_of: nil>
   ```

2. 在刚刚找到的movie中找damon的角色（**MovieRole对象**）

   ```ruby
   irb(main):042:0> role = movie.roles.where(:id => damon.id).first
   => #<MovieRole _id: nm0000354, character: "Will Hunting", actorName(actor_name): "Matt Damon", main: true, urlCharacter(url_character): "http://www.imdb.com/character/ch0003602", urlPhoto(url_photo): "http://ia.media-imdb.com/images/M/MV5BMTM0NzYzNDgxMl5BMl5BanBnXkFtZTcwMDg2MTMyMw@@._V1_UY44_CR0,0,32,44_AL_.jpg", urlProfile(url_profile): "http://www.imdb.com/name/nm0000354">
   ```

3. 使用pp来看attributes

   ```ruby
   irb(main):043:0> pp role.attributes
   {"actorName"=>"Matt Damon",
    "character"=>"Will Hunting",
    "main"=>true,
    "urlCharacter"=>"http://www.imdb.com/character/ch0003602",
    "urlPhoto"=>
     "http://ia.media-imdb.com/images/M/MV5BMTM0NzYzNDgxMl5BMl5BanBnXkFtZTcwMDg2MTMyMw@@._V1_UY44_CR0,0,32,44_AL_.jpg",
    "urlProfile"=>"http://www.imdb.com/name/nm0000354",
    "_id"=>"nm0000354"}
   => {"actorName"=>"Matt Damon", "character"=>"Will Hunting", "main"=>true, "urlCharacter"=>"http://www.imdb.com/character/ch0003602", "urlPhoto"=>"http://ia.media-imdb.com/images/M/MV5BMTM0NzYzNDgxMl5BMl5BanBnXkFtZTcwMDg2MTMyMw@@._V1_UY44_CR0,0,32,44_AL_.jpg", "urlProfile"=>"http://www.imdb.com/name/nm0000354", "_id"=>"nm0000354"}
   ```

4. roles有很多的method，我们试试下面这种来将电影名和角色名摆在一起

   ```ruby
   irb(main):044:0> damon.roles.map{|role| "#{role.movie.title} => #{role.character}"}
   D, [2016-10-16T17:20:57.210472 #30068] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"movies", "filter"=>{"roles._id"=>"nm0000354"}}
   D, [2016-10-16T17:20:57.214285 #30068] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.003687256s
   => ["Good Will Hunting => Will Hunting", "Saving Private Ryan => Private Ryan", "The Bourne Ultimatum => Jason Bourne", "The Martian => Mark Watney"]
   irb(main):045:0> pp damon.roles.map{|role| "#{role.movie.title} => #{role.character}"}
   D, [2016-10-16T17:21:12.207153 #30068] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"movies", "filter"=>{"roles._id"=>"nm0000354"}}
   D, [2016-10-16T17:21:12.210573 #30068] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.003284118s
   ["Good Will Hunting => Will Hunting",
    "Saving Private Ryan => Private Ryan",
    "The Bourne Ultimatum => Jason Bourne",
    "The Martian => Mark Watney"]
   => ["Good Will Hunting => Will Hunting", "Saving Private Ryan => Private Ryan", "The Bourne Ultimatum => Jason Bourne", "The Martian => Mark Watney"]
   ```

5. 为了建立relationship，我们为Matt Damon创建一个假的电影和角色

   ```ruby
   irb(main):050:0* rocky26=Movie.create(:_id => "tt9000015", :title => "Rocky 26") 
   => #<Movie _id: tt9000015, created_at: 2016-10-16 16:24:51 UTC, updated_at: 2016-10-16 16:24:51 UTC, title: "Rocky 26", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
   ```

6. 将damon放入这个新建的movie中并给予一个Rocky的角色

   ```ruby
   irb(main):051:0> rocky=rocky26.roles.create(:_id=>damon.id, :character=>"Rocky", :actorName=>"Matt", :main=>true)
   => #<MovieRole _id: nm0000354, character: "Rocky", actorName(actor_name): "Matt", main: true, urlCharacter(url_character): nil, urlPhoto(url_photo): nil, urlProfile(url_profile): nil>
   ```

7. 寻找新建的电影，看一下新加入的角色

   ```ruby
   irb(main):052:0> pp Movie.collection.find(:title=>"Rocky 26").first
   {"_id"=>"tt9000015",
    "title"=>"Rocky 26",
    "updated_at"=>2016-10-16 16:24:51 UTC,
    "created_at"=>2016-10-16 16:24:51 UTC,
    "roles"=>
     [{"_id"=>"nm0000354",
       "character"=>"Rocky",
       "actorName"=>"Matt",
       "main"=>true}]}
   => {"_id"=>"tt9000015", "title"=>"Rocky 26", "updated_at"=>2016-10-16 16:24:51 UTC, "created_at"=>2016-10-16 16:24:51 UTC, "roles"=>[{"_id"=>"nm0000354", "character"=>"Rocky", "actorName"=>"Matt", "main"=>true}]}
   ```

8. 重复第四步中的指令，看看Matt Damon现在有什么电影什么角色,可以看到新添加了一个电影。

   ```ruby
   irb(main):055:0> pp damon.roles.map{|role| "#{role.movie.title}=>#{role.character}"}
   ["Good Will Hunting=>Will Hunting",
    "Saving Private Ryan=>Private Ryan",
    "The Bourne Ultimatum=>Jason Bourne",
    "The Martian=>Mark Watney",
    "Rocky 26=>Rocky"]
   => ["Good Will Hunting=>Will Hunting", "Saving Private Ryan=>Private Ryan", "The Bourne Ultimatum=>Jason Bourne", "The Martian=>Mark Watney", "Rocky 26=>Rocky"
   ```


---

### Summary

* **1** side has a primary key and typically has no reference to the child within the document.
* **M** side will typically host the foreign key