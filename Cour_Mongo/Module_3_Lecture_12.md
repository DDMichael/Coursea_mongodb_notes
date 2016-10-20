### Constraints and Validation: Demo

---

* 首先看**dependent: :destroy**这个选项


* 在movie.rb之中，:writers和:sequel有如下的关系连接

  ```ruby
  class Movie
    ...
    has_and_belongs_to_many :writers, dependent: :destroy
    has_one :sequel, foreign_key: :sequel_of, class_name: "Movie", dependent: :destroy
  ...
  end
  ```

#### Demo

1. setup a script: rocky30, rocky31和writer

   ```ruby
   irb(main):002:0> rocky30 = Movie.create(:title => "Rocky 30")
   => #<Movie _id: 58087e661d41c856f3000000, created_at: 2016-10-20 08:20:54 UTC, updated_at: 2016-10-20 08:20:54 UTC, title: "Rocky 30", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: nil>

   irb(main):003:0> rocky31 = Movie.create(:title => "Rocky 31", :sequel_to => rocky30)
   => #<Movie _id: 58087e7f1d41c856f3000001, created_at: 2016-10-20 08:21:19 UTC, updated_at: 2016-10-20 08:21:19 UTC, title: "Rocky 31", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: nil, sequel_of: BSON::ObjectId('58087e661d41c856f3000000')>

   irb(main):004:0> writer = rocky30.writers.create(:name => "A Writer")
   => #<Writer _id: 58087e991d41c856f3000002, created_at: 2016-10-20 08:21:45 UTC, updated_at: 2016-10-20 08:21:45 UTC, name: "A Writer", movie_ids: [BSON::ObjectId('58087e661d41c856f3000000')]>
   ```

2. 列出上述三个object的id

   ```ruby
   irb(main):005:0> rocky30.id
   => BSON::ObjectId('58087e661d41c856f3000000')
   irb(main):006:0> rocky31.id
   => BSON::ObjectId('58087e7f1d41c856f3000001')
   irb(main):007:0> writer.id
   => BSON::ObjectId('58087e991d41c856f3000002')
   ```

3. 查看关系

   1. 从rocky30角度查看writer

      ```ruby
      irb(main):008:0> Movie.where(:id => rocky30.id).first.writer_ids
      => [BSON::ObjectId('58087e991d41c856f3000002')]
      ```

   2. 从rocky31角度查看sequel_of

      ```ruby
      irb(main):010:0> Movie.where(:id=>rocky31.id).first.sequel_of
      => BSON::ObjectId('58087e661d41c856f3000000')
      ```

   3. 从writer角度查看movie

      ```ruby
      irb(main):011:0> Writer.where(:id=>writer.id).first.movie_ids
      => [BSON::ObjectId('58087e661d41c856f3000000')]
      ```

4. 尝试destroy

   1. 首先找到rocky30

      ```ruby
      irb(main):016:0> rocky30 = Movie.find(rocky30.id)
      ```

   2. 删除这个object

      ```ruby
      irb(main):017:0> rocky30.destroy
      irb(main):015:0> rocky30.destroy
      before_destroy Movie callback for 58087e661d41c856f3000000, sequel_to=, writers=[BSON::ObjectId('58087e991d41c856f3000002')]
      before_destroy Writer callback for 58087e991d41c856f3000002, movies=[BSON::ObjectId('58087e661d41c856f3000000')]
      after_destroy Writer callback for 58087e991d41c856f3000002, movies=[BSON::ObjectId('58087e661d41c856f3000000')]
      before_destroy Movie callback for 58087e7f1d41c856f3000001, sequel_to=#<Movie:0x007f6854b152d8>, writers=[]
      after_destroy Movie callback for 58087e7f1d41c856f3000001, sequel_to=#<Movie:0x007f6854b152d8>, writers=[]
      after_destroy Movie callback for 58087e661d41c856f3000000, sequel_to=, writers=[BSON::ObjectId('58087e991d41c856f3000002')]
      => true
      ```

      注意所有的before_destroy和after_destroy

   3. 检查一下刚刚删除的是否完全被清空

      1. 检查rocky30在不在

         ```ruby
         irb(main):017:0> Movie.where(:id => rocky30.id).exists?
         => false
         ```

      2. 检查rocky30的writer在不在

         ```ruby
         irb(main):018:0> Movie.where(:id => rocky30.id).first.writer_ids if Movie.where(:id => rocky30.id).exists?
         => nil
         ```

      3. 检查rocky31是否存在并检查其与rocky30的的relationship

         ```ruby
         irb(main):019:0> Movie.where(:id => rocky31.id).exists?
         => false
         irb(main):020:0> Movie.where(:id => rocky31.id).first.sequel_of if Movie.where(:id => rocky31.id).exists?
         => nil
         ```

      4. 检查writer是否存在并检查其与rocky30的relationship

         ```ruby
         irb(main):021:0> Writer.where(:id => writer.id).exists?
         => false
         irb(main):023:0> Writer.where(:id => writer.id).first.movie_ids if Writer.where(:id=>writer.id).exists?
         => nil
         ```

---

* 看**dependent: :delete**这个选项

* 在movie.rb之中需要作出如下更改

  ```ruby
  class Movie
    ...
    has_and_belongs_to_many :writers, dependent: :delete
    has_one :sequel, foreign_key: :sequel_of, class_name: "Movie", dependent: :delete
  ...
  end
  ```

* writer 是一个**M:M**关系，而sequ

---

#### Demo

1. 使用之前那个script生成rocky30, rocky31还有writer

   ```ruby
   irb(main):005:0> rocky30.id
   => BSON::ObjectId('580883001d41c864d0000000')
   irb(main):006:0> rocky31.id
   => BSON::ObjectId('5808831d1d41c864d0000001')
   irb(main):007:0> writer.id
   => BSON::ObjectId('580883361d41c864d0000002')
   ```

2. 使用rocky30.destroy

   ```ruby
   irb(main):066:0> rocky30.destroy
   before_destroy Movie callback for 580892d41d41c864d0000009, sequel_to=, writers=[BSON::ObjectId('580892eb1d41c864d000000b')]
   D, [2016-10-20T10:48:53.022922 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"writers", "filter"=>{"$and"=>[{"_id"=>{"$in"=>[BSON::ObjectId('580892eb1d41c864d000000b')]}}]}}
   D, [2016-10-20T10:48:53.023503 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.00047937s
   D, [2016-10-20T10:48:53.024242 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | STARTED | {"update"=>"writers", "updates"=>[{"q"=>{"$and"=>[{"_id"=>{"$in"=>[BSON::ObjectId('580892eb1d41c864d000000b')]}}]}, "u"=>{"$pull"=>{"movie_ids"=>BSON::ObjectId('580892d41d41c864d0000009')}}, "multi"=>true, "upsert"=>false}], "writeConcern"=>{:w=>1}, "...
   D, [2016-10-20T10:48:53.024723 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | SUCCEEDED | 0.00038674200000000003s
   D, [2016-10-20T10:48:53.025430 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | STARTED | {"update"=>"movies", "updates"=>[{"q"=>{"_id"=>BSON::ObjectId('580892d41d41c864d0000009')}, "u"=>{"$set"=>{"writer_ids"=>[]}}, "multi"=>false, "upsert"=>false}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.025941 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | SUCCEEDED | 0.00042514699999999996s
   D, [2016-10-20T10:48:53.027060 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"movies", "filter"=>{"sequel_of"=>BSON::ObjectId('580892d41d41c864d0000009')}}
   D, [2016-10-20T10:48:53.027628 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.0004955199999999999s
   D, [2016-10-20T10:48:53.028914 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | STARTED | {"update"=>"writers", "updates"=>[{"q"=>{"$and"=>[{"_id"=>{"$in"=>[]}}]}, "u"=>{"$pull"=>{"movie_ids"=>BSON::ObjectId('580892db1d41c864d000000a')}}, "multi"=>true, "upsert"=>false}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.032432 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | SUCCEEDED | 0.0033888449999999997s
   D, [2016-10-20T10:48:53.033408 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | STARTED | {"update"=>"movies", "updates"=>[{"q"=>{"_id"=>BSON::ObjectId('580892db1d41c864d000000a')}, "u"=>{"$set"=>{"writer_ids"=>[]}}, "multi"=>false, "upsert"=>false}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.033857 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | SUCCEEDED | 0.000394598s
   D, [2016-10-20T10:48:53.034722 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"movies", "filter"=>{"sequel_of"=>BSON::ObjectId('580892db1d41c864d000000a')}}
   D, [2016-10-20T10:48:53.035182 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.000399886s
   D, [2016-10-20T10:48:53.035775 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | STARTED | {"delete"=>"movies", "deletes"=>[{"q"=>{"_id"=>BSON::ObjectId('580892db1d41c864d000000a')}, "limit"=>1}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.036209 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | SUCCEEDED | 0.000340116s
   D, [2016-10-20T10:48:53.036812 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | STARTED | {"delete"=>"movies", "deletes"=>[{"q"=>{"_id"=>BSON::ObjectId('580892d41d41c864d0000009')}, "limit"=>1}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.037199 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | SUCCEEDED | 0.000317467s
   after_destroy Movie callback for 580892d41d41c864d0000009, sequel_to=, writers=[]
   => true

   ```

   注意到一点的是，writer并没有被哦remove，但是movie被remove了，有one-to-one关系的会被删除

   **one-to-one dependent: :delete会删除parent和child，但是many-to-many在dependent: :delete中不会删除另一边，只是解除关系**

3. 测试一下哪些被删除哪些没被删除

   1. 查看rocky30

      ```ruby
      irb(main):011:0> Movie.where(:id => rocky30.id).exists?
      => false
      ```

   2. 从rocky30来看writer

      ```ruby
      irb(main):012:0> Movie.where(:id => rocky30.id).first.writer_ids if Movie.where(:id => rocky30.id).exists?
      => nil
      ```

   3. 查看rocky31

      ```ruby
      irb(main):013:0> Movie.where(:id => rocky31.id).exists?
      => false
      ```

   4. 从rocky31角度来查找rocky30

      ```ruby
      irb(main):014:0> Movie.where(:id => rocky31.id).first.sequel_of if Movie.where(:id => rocky31.id).exists?
      => nil
      ```

   5. 从**Writer**里查有没有原本是属于rocky30的writer

      ```ruby
      irb(main):015:0> Writer.where(:id => writer.id).exists?
      => true
      ```

   6. 从writer中查找现在它和rocky30还有没有关系

      ```ruby
      irb(main):016:0> Writer.where(:id => writer.id).first.movie_ids if Writer.where(:id=>writer.id).exists?
      => []
      ```

---

* Nullify: 如果在删除parent的时候不删除child我们用nullify

* 作出如下更改：

  ```ruby
  class Movie
    ...
    has_and_belongs_to_many :writers, dependent: :nullify
    has_one :sequel, foreign_key: :sequel_of, class_name: "Movie", dependent: :nullify
  ...
  end
  ```

#### Demo

1. 清楚data之后创建信息

   ```ruby
   irb(main):030:0> rocky30.id
   => BSON::ObjectId('58088d8e1d41c864d0000003')
   irb(main):031:0> rocky31.id
   => BSON::ObjectId('58088daa1d41c864d0000004')
   irb(main):032:0> Movie.where(:id=>rocky30.id).first.writer_ids
   => [BSON::ObjectId('58088dc61d41c864d0000005')]
   ```

2. 删除rocky30

   ```ruby
   irb(main):065:0> rocky30 = Movie.find(rocky30.id)
   D, [2016-10-20T10:48:47.653191 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"movies", "filter"=>{"_id"=>BSON::ObjectId('580892d41d41c864d0000009')}}
   D, [2016-10-20T10:48:47.653759 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.000456415s
   => #<Movie _id: 580892d41d41c864d0000009, created_at: 2016-10-20 09:48:04 UTC, updated_at: 2016-10-20 09:48:04 UTC, title: "Rocky 30", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: [BSON::ObjectId('580892eb1d41c864d000000b')], sequel_of: nil>

   irb(main):066:0> rocky30.destroy
   before_destroy Movie callback for 580892d41d41c864d0000009, sequel_to=, writers=[BSON::ObjectId('580892eb1d41c864d000000b')]
   D, [2016-10-20T10:48:53.022922 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"writers", "filter"=>{"$and"=>[{"_id"=>{"$in"=>[BSON::ObjectId('580892eb1d41c864d000000b')]}}]}}
   D, [2016-10-20T10:48:53.023503 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.00047937s
   D, [2016-10-20T10:48:53.024242 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | STARTED | {"update"=>"writers", "updates"=>[{"q"=>{"$and"=>[{"_id"=>{"$in"=>[BSON::ObjectId('580892eb1d41c864d000000b')]}}]}, "u"=>{"$pull"=>{"movie_ids"=>BSON::ObjectId('580892d41d41c864d0000009')}}, "multi"=>true, "upsert"=>false}], "writeConcern"=>{:w=>1}, "...
   D, [2016-10-20T10:48:53.024723 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | SUCCEEDED | 0.00038674200000000003s
   D, [2016-10-20T10:48:53.025430 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | STARTED | {"update"=>"movies", "updates"=>[{"q"=>{"_id"=>BSON::ObjectId('580892d41d41c864d0000009')}, "u"=>{"$set"=>{"writer_ids"=>[]}}, "multi"=>false, "upsert"=>false}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.025941 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | SUCCEEDED | 0.00042514699999999996s
   D, [2016-10-20T10:48:53.027060 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"movies", "filter"=>{"sequel_of"=>BSON::ObjectId('580892d41d41c864d0000009')}}
   D, [2016-10-20T10:48:53.027628 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.0004955199999999999s
   D, [2016-10-20T10:48:53.028914 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | STARTED | {"update"=>"writers", "updates"=>[{"q"=>{"$and"=>[{"_id"=>{"$in"=>[]}}]}, "u"=>{"$pull"=>{"movie_ids"=>BSON::ObjectId('580892db1d41c864d000000a')}}, "multi"=>true, "upsert"=>false}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.032432 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | SUCCEEDED | 0.0033888449999999997s
   D, [2016-10-20T10:48:53.033408 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | STARTED | {"update"=>"movies", "updates"=>[{"q"=>{"_id"=>BSON::ObjectId('580892db1d41c864d000000a')}, "u"=>{"$set"=>{"writer_ids"=>[]}}, "multi"=>false, "upsert"=>false}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.033857 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.update | SUCCEEDED | 0.000394598s
   D, [2016-10-20T10:48:53.034722 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | STARTED | {"find"=>"movies", "filter"=>{"sequel_of"=>BSON::ObjectId('580892db1d41c864d000000a')}}
   D, [2016-10-20T10:48:53.035182 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.find | SUCCEEDED | 0.000399886s
   D, [2016-10-20T10:48:53.035775 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | STARTED | {"delete"=>"movies", "deletes"=>[{"q"=>{"_id"=>BSON::ObjectId('580892db1d41c864d000000a')}, "limit"=>1}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.036209 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | SUCCEEDED | 0.000340116s
   D, [2016-10-20T10:48:53.036812 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | STARTED | {"delete"=>"movies", "deletes"=>[{"q"=>{"_id"=>BSON::ObjectId('580892d41d41c864d0000009')}, "limit"=>1}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-20T10:48:53.037199 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | SUCCEEDED | 0.000317467s
   after_destroy Movie callback for 580892d41d41c864d0000009, sequel_to=, writers=[]
   => true
   ```

3. 检查一下哪些被删除哪些没被删除

   1. 查看rocky30

      ```ruby
      irb(main):037:0> Movie.where(:id=>rocky30.id).exists?
      => false
      ```

   2. 从rocky30来看writer

      ```ruby
      irb(main):038:0> Movie.where(:id=>rocky30.id).first.writer_ids if Movie.where(:id => rocky30.id).exists?
      => nil
      ```

   3. 查看rocky31

      ```ruby
      irb(main):039:0> Movie.where(:id => rocky31.id).exists?
      => true
      ```

   4. 从rocky31来看与rocky30的关系

      ```ruby
      irb(main):040:0> Movie.where(:id => rocky31.id).first.sequel_of if Movie.where(:id => rocky31.id).exists?
      => nil
      ```

   5. 查看writer

      ```ruby
      irb(main):041:0> Writer.where(:id => writer.id).exists?
      => true
      ```

   6. 从writer来看与rocky30的关系

      ```ruby
      irb(main):042:0> Writer.where(:id => writer.id).first.movie_ids if Writer.where(:id=>writer.id).exists?
      => []
      ```

---

* 在default情况下虽然parent被删除了但是child仍然有reference。**即使被删除了**

---

* restrict
* 在movie.rb中作出如下修改

```ruby
class Movie
  ...
  has_and_belongs_to_many :writers, dependent: :restrict
  has_one :sequel, foreign_key: :sequel_of, class_name: "Movie", dependent: :restrict
...
end
```

#### Demo

1. 清楚并建立新的data

   ```ruby
   irb(main):050:0> rocky30.id
   => BSON::ObjectId('580891581d41c864d0000006')
   irb(main):051:0> rocky31.id
   => BSON::ObjectId('5808916e1d41c864d0000007')
   irb(main):052:0> Movie.where(:id => rocky30.id).first.writer_ids
   => [BSON::ObjectId('580891811d41c864d0000008')]
   ```

2. 删除

   ```ruby
   irb(main):053:0> rocky30 = Movie.find(rocky30.id)
   => #<Movie _id: 580891581d41c864d0000006, created_at: 2016-10-20 09:41:44 UTC, updated_at: 2016-10-20 09:41:44 UTC, title: "Rocky 30", type: nil, rated: nil, year: nil, release_date: nil, runtime: nil, votes: nil, countries: nil, languages: nil, genres: nil, filmingLocations(filming_locations): nil, metascore: nil, simplePlot(simple_plot): nil, plot: nil, urlIMDB(url_imdb): nil, urlPoster(url_poster): nil, directors: nil, actors: nil, writer_ids: [BSON::ObjectId('580891811d41c864d0000008')], sequel_of: nil>

   irb(main):055:0> rocky30.destroy
   before_destroy Movie callback for 580891581d41c864d0000006, sequel_to=, writers=[BSON::ObjectId('580891811d41c864d0000008')]
   D, [2016-10-20T10:44:40.779954 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.count | STARTED | {"count"=>"writers", "query"=>{"$and"=>[{"_id"=>{"$in"=>[BSON::ObjectId('580891811d41c864d0000008')]}}]}}
   D, [2016-10-20T10:44:40.783052 #25808] DEBUG -- : MONGODB | localhost:27017 | movies_development.count | SUCCEEDED | 0.0029178309999999996s
   Mongoid::Errors::DeleteRestriction: 
   message:
     Cannot delete Movie because of dependent 'writers'.
   summary:
     When defining 'writers' with a :dependent => :restrict, Mongoid will raise an error when attempting to delete the Movie when the child 'writers' still has documents in it.
   resolution:
     Don't attempt to delete the parent Movie when it has children, or change the dependent option on the relation.
   ```

   发现不给删除！所有的数据都还在那里没有变