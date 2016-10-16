### CRUD Operation

#### create, find, update, upsert and delete

---

##### Operation - Model.create

* Model.create

```ruby
irb(main):017:0> movie = Movie.create(
irb(main):018:1* title: "Martian",
irb(main):019:1* type: "Thriller",
irb(main):020:1* rated: "R",
irb(main):021:1* year: 2015
irb(main):022:1> )
```

* 将会插入一个document在"movies" collection之中
  *  Movie这个class会自动创建一个colleciton叫做movies(如果movies本身不存在的话)

Demo

1. 将文件创建并存入数据库中

   ```ruby
   irb(main):017:0> movie = Movie.create(
   irb(main):018:1* title: "Martian",
   irb(main):019:1* type: "Thriller",
   irb(main):020:1* rated: "R",
   irb(main):021:1* year: 2015
   irb(main):022:1> )
   => #<Movie _id: 57ff640d1d41c8082e000000, created_at: 2016-10-13 10:38:05 UTC, updated_at: 2016-10-13 10:38:05 UTC, title: "Martian", type: "Thriller", rated: "R", year: 2015, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
   ```

2. 检查这个文件是否顺利插入

   ```ruby
   irb(main):025:0* m = Movie.find_by(:title => "Martian")
   => #<Movie _id: 57ff640d1d41c8082e000000, created_at: 2016-10-13 10:38:05 UTC, updated_at: 2016-10-13 10:38:05 UTC, title: "Martian", type: "Thriller", rated: "R", year: 2015, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
   ```

---

#### Operation - Model (save)

1. Save其实是两步：

   1. 使用Movie.new建立一个

      ```ruby
      movie = Movie.new(
        title: "Rocky",
        type: "Action",
        rated: "R",
        year: 1975
      )
      ```

   2. 使用save来存储

      ```movie.save```

#### Demo

```ruby
irb(main):026:0> movie = Movie.new(
irb(main):027:1*   title: "Rocky",
irb(main):028:1*   type: "Action",
irb(main):029:1*   rated: "R",
irb(main):030:1*   year: 1975
irb(main):031:1> )
=> #<Movie _id: 57ff678b1d41c8082e000001, created_at: nil, updated_at: nil, title: "Rocky", type: "Action", rated: "R", year: 1975, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
irb(main):032:0> movie.save
=> true
```

2. 使用save的快速update功能


1. ```movie.year = 1986```
2. ```movie.save```

#### Demo

```ruby
irb(main):035:0> movie.year = 1986
=> 1986
irb(main):036:0> movie.save
=> true
irb(main):041:0> m = Movie.find_by(:title => "Rocky")
=> #<Movie _id: 57ff678b1d41c8082e000001, created_at: 2016-10-13 10:53:34 UTC, updated_at: 2016-10-13 10:55:23 UTC, title: "Rocky", type: "Action", rated: "R", year: 1986, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
```

---

#### Operation - update_attributes 

```ruby
#第一步，生成一个新的movie
movie = Movie.new(
  title: "Rocky31",
  rated: "PG-13"
)
movie.save
#第二步，更改属性
movie.update_attributes(:rated => "R")
```

#### Demo

```ruby
irb(main):042:0> movie = Movie.new(
irb(main):043:1*   title: "Rocky31",
irb(main):044:1*   rated: "PG-13"
irb(main):045:1> )
=> #<Movie _id: 57ff692c1d41c8082e000002, created_at: nil, updated_at: nil, title: "Rocky31", type: nil, rated: "PG-13", year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
irb(main):047:0> movie.save
=> true
irb(main):048:0> m = Movie.find_by(:title => "Rocky31")
=> #<Movie _id: 57ff692c1d41c8082e000002, created_at: 2016-10-13 11:00:08 UTC, updated_at: 2016-10-13 11:00:08 UTC, title: "Rocky31", type: nil, rated: "PG-13", year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
irb(main):049:0> movie.update_attributes(:rated => "R")
=> true
```

---



#### Operation - Model#upsert

**upsert**的作用：

1. 如果document存在，将会被overwritten
2. 如果document不存在，将会新建一条并插入

```ruby
movie = Movie.create(
  title: "Rocky31",
  rated: "PG-13"
)
movie.new(:_id => "<ID>",
  :title=>"Rocky31", :rated=>"R").upsert
```

#### Demo

1. 使用create创建一个新的Movie object

```ruby
irb(main):054:0> movie = Movie.create(
irb(main):055:1*   title: "Rocky31",
irb(main):056:1*   rated: "PG-13"
irb(main):057:1> )
=> #<Movie _id: 57ff6a8f1d41c8082e000003, created_at: 2016-10-13 11:05:51 UTC, updated_at: 2016-10-13 11:05:51 UTC, title: "Rocky31", type: nil, rated: "PG-13", year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
```

2. 查找一下这个object确认存在

```ruby
irb(main):061:0* Movie.find_by(:title => "Rocky31")
=> #<Movie _id: 57ff692c1d41c8082e000002, created_at: 2016-10-13 11:00:08 UTC, updated_at: 2016-10-13 11:01:30 UTC, title: "Rocky31", type: nil, rated: "R", year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
```

3. 将刚存储的文件的id复制下来并执行如下命令

```ruby
irb(main):065:0* Movie.new(:_id=>"57ff692c1d41c8082e000002", :title=>"Rocky31", :rated=>"R").upsert
=> true
#这个意思是Movie将会寻找id是57ff692c1d41c8082e000002的documents，如果找到的话就更改rated的值，如果没找到就添加一个新的object
```

4. 重新查找刚刚的文件看是否rated的值被更改

```ruby
irb(main):066:0> Movie.find_by(:title =>"Rocky31")
=> #<Movie _id: 57ff692c1d41c8082e000002, created_at: nil, updated_at: nil, title: "Rocky31", type: nil, rated: "R", year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>
```

5. 另一种情况就不写了，反正是如果删除了可以重新添加一个文件

---

#### Operation - Model#delete

1. ```movie.delete```

   将会删除当前这个**movie** document

2. ```Movie.delete_all```

   将会从**movies** collection 中删除所有的documents

**Demo**就不做了，反正找到了之后写delete就行