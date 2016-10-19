### Constraints and Validation

---

* Field Validation

  * ActiveModel的validation可以加进Mongoid里面

  ```ruby
  class Director
    include Mongoid::Document
    include Mongoid::Timestamps
    field :name, type: String
    ......
    validates_presence_of :name #这个的意思就是name是必须要填写的
  end
  ```

#### Demo

1. 创建一个没有名字的director

   ```ruby
   irb(main):018:0> director = Director.create(:_id => "12345")
   => #<Director _id: 12345, created_at: nil, updated_at: nil, name: nil, residence_id: nil>
   ```

2. 查看director的error的数量

   ```ruby
   irb(main):019:0> director.errors.count
   => 1
   ```

3. 查看具体的error

   ```ruby
   irb(main):021:0> director.errors
   => #<ActiveModel::Errors:0x005642d78cf1c0 @base=#<Director _id: 12345, created_at: nil, updated_at: nil, name: nil, residence_id: nil>, @messages={:name=>["can't be blank"]}>
   ```

4. 如果想直接显示error的话就使用感叹号

   ```ruby
   irb(main):017:0> director = Director.create!(:_id => "12345")
   Mongoid::Errors::Validations: 
   message:
     Validation of Director failed.
   summary:
     The following errors were found: Name can't be blank
   resolution:
     Try persisting the document with valid data or remove the validations.
   	from /home/daomingdong/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/mongoid-5.0.1/lib/mongoid/persistable.rb:78:in `fail_due_to_validation!'
   	from -e:1:in `<main>'
   ```

---

#### Dependent Behavior

* Mongoid支持dependent选项来管理相关联的东西
* :delete, :destroy, :nullify, :restrict

#### Relationship Constraints

* (default) Orphans the child document
  * 1:1 和 1:M在删除parent之后还会留有一个reference
  * M:M直接会将parent的reference删除，就像:nullify
* :nullify - 孤立child doucment，当foreign key被设置为空时
* :destroy - Remove the child document after running model callbacks on the child
* :delete - Remove the child document **without** running model callbacks on the child
* :restrict - **Raise** an **error** if a child references the parent being removed

#### Callbacks - Movie Model

```ruby
before_destroy do |doc|
  puts "before_destroy Movie callback for #{doc.id}, "\
  	   "sequel_to=#{doc.sequel_to}, writers=#{doc.writer_ids}"
end
after_destroy do |doc|
  puts "after_destroy Movie callback for #{doc.id}, "\
       "sequel_to=#{doc.sequel_to}, writers = #{doc.writer_ids}"
end
```

```ruby
before_destroy do |doc|
  puts "before_destroy Movie callback for #{doc.id}, "\
       "writers=#{doc.movie_ids}"
end
after_destroy do |doc|
  puts "after_destroy Movie callback for #{doc.id}, "\
       "writers = #{doc.movie_ids}"
end
```

---

#### Script - Demo Setup

```ruby
reload!
rocky30 = Movie.create(:title => "Rocky 30")
rocky31 = Movie.create(:title => "Rocky 31", :sequel_to => rocky30)
writer = rocky30.writers.create
				  (:name => "A Writer")
```

#### Script - Data Cleanup

```ruby
p Movie.where(:title => {:$regex => "Rocky 3[0-1]"}).delete_all;
Writer.where(:name => {:$regex=>"Writer"}).delete_all
```

#### Callback

1. 打开Debug模式

   ```ruby
   irb(main):022:0> Mongo::Logger.logger.level = ::Logger::DEBUG
   => 0
   ```

2. 使用Movie和Writer的destroy会产生如下的信息

   ```ruby
   irb(main):023:0> Movie.new.destroy
   before_destroy Movie callback for 580739791d41c87ce0000000, sequel_to=, writers=[]
   D, [2016-10-19T10:14:33.886221 #31968] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | STARTED | {"delete"=>"movies", "deletes"=>[{"q"=>{"_id"=>BSON::ObjectId('580739791d41c87ce0000000')}, "limit"=>1}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-19T10:14:33.886890 #31968] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | SUCCEEDED | 0.000489144s
   after_destroy Movie callback for 580739791d41c87ce0000000, sequel_to=, writers=[]
   => true

   irb(main):024:0> Writer.new.destroy
   before_destroy Writer callback for 580739801d41c87ce0000001, movies=[]
   D, [2016-10-19T10:14:40.469988 #31968] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | STARTED | {"delete"=>"writers", "deletes"=>[{"q"=>{"_id"=>BSON::ObjectId('580739801d41c87ce0000001')}, "limit"=>1}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-19T10:14:40.470504 #31968] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | SUCCEEDED | 0.00041191100000000005s
   after_destroy Writer callback for 580739801d41c87ce0000001, movies=[]
   => true
   ```

3. 使用delete将没有如下信息

   ```ruby
   irb(main):025:0> Movie.new.delete
   D, [2016-10-19T10:14:45.824205 #31968] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | STARTED | {"delete"=>"movies", "deletes"=>[{"q"=>{"_id"=>BSON::ObjectId('580739851d41c87ce0000002')}, "limit"=>1}], "writeConcern"=>{:w=>1}, "ordered"=>true}
   D, [2016-10-19T10:14:45.824685 #31968] DEBUG -- : MONGODB | localhost:27017 | movies_development.delete | SUCCEEDED | 0.00035508999999999996s
   => true
   ```