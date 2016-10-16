### 关于Mongodb gridfs中的save

**第一次更新**

后面的assignment要求更改一下逻辑，补充一个如果文件存在可以直接复写的功能，功能添加在下文所示位置添加

```ruby
unless persisted?

else
  params = {}
  params[:metadata] = {}
  params[:metadata][:location] = @location.to_hash
  self.class.mongo_client.database.fs.find(:_id =>    BSON::ObjectId.from_string(@id))
        .update_one(params)
end
```

# 这里要讲一下我个人对GridFS的理解

**GridFS**会将图片分成若干部分并存储，其中最重要的是一个叫做*metadata*的东西，**GridFS**存的东西是这个形式的

```ruby
irb(main):033:0> b = Photo.mongo_client.database.fs.find(:_id=>BSON::ObjectId('57fd63561d41c84b76000078'))
=> #<Mongo::Collection::View:0x47196112758620 namespace='places_development.fs.files @selector={:_id=>BSON::ObjectId('57fd63561d41c84b76000078')} @options={:read=>#<Mongo::ServerSelector::Primary:0x0055d9664079d8 @options={"database"=>"places_development"}, @tag_sets=[], @server_selection_timeout=30>}>

irb(main):035:0> b.first
MONGODB | localhost:27017 | places_development.find | STARTED | {"find"=>"fs.files", "filter"=>{:_id=>BSON::ObjectId('57fd63561d41c84b76000078')}}
MONGODB | localhost:27017 | places_development.find | SUCCEEDED | 0.00046925s
=> {"_id"=>BSON::ObjectId('57fd63561d41c84b76000078'), "chunkSize"=>261120, "uploadDate"=>2016-10-11 22:10:30 UTC, "contentType"=>"image/jpeg", "metadata"=>{"location"=>{"type"=>"Point", "coordinates"=>[-116.30161960177952, 33.87546081542969]}}, "length"=>624744, "md5"=>"871666ee99b90e51c69af02f77f021aa"}

```

而标准的**Model**存的是这个样子的

```ruby
irb(main):026:0> a=Photo.find "57fd63561d41c84b76000078"
MONGODB | localhost:27017 | places_development.find | STARTED | {"find"=>"fs.files", "filter"=>{:_id=>BSON::ObjectId('57fd63561d41c84b76000078')}}
MONGODB | localhost:27017 | places_development.find | SUCCEEDED | 0.000372298s
=> #<Photo:0x0055d966b11b70 @id="57fd63561d41c84b76000078", @location=#<Point:0x0055d966b11940 @longitude=-116.30161960177952, @latitude=33.87546081542969>, @contents=nil>

```

可以看到两者共享一个**ID**，即==57fd63561d41c84b76000078==， 但是两者的**ID**形式不一样，一个是**ObjectId**形式，另一个则是**String**形式。还有就是最重要的，query的方法：

1. 使用**mongo_client.database.fs.find(:_id => BSON::ObjectId(' '))**来查询详细的Gridfs文件
2. 使用普通的自定义**find**来查询字符串



**初始**

这个module2的作业要起实现一个**save**功能，这个功能有如下几个要求：

1. 检查这个文件是否已经存在

   * 使用如下检查

     ```ruby
     unless persisted?

     end
     ```

2. 使用**exfir**这个**gem**来获取geolocation information

   * 使用如下方法获得

     * 首先将*@contents*读取到文件中并生成一个空**description**数组

       ```ruby
       f = @contents
       descrption = {}
       gps = EXIFR::JPEG.new(f).gps
       @location = Point.new(:lng=>gps.longitude, :lat=>gps.latitude)
       f.rewind #加入这一行是由于EXIFR和GridFS读的是同一个东西
       ```

3. 将**GridFS**的**contentType**这个属性设置为*image/jpeg*

   ```ruby
   descrption[:content_type] = "image/jpeg"
   ```

   ​

4. 将**exfir**中得到的**GeoJSON**点存储到**GridFS**中的**metadata**里面的**location**属性

   ```ruby
   if @location
     descrption[:metadata] = {}
     descrption[:metadata][:location] = @location.to_hash unless @location.nil?##这里to_hash必须要加，需要讲GeoJson变成hash
   end
   ```

5. 存储这个**GridFS**, 将生成的**_id**变成Photo model的一个instance

   ```ruby
   if @contents
     grid_file = Mongo::Grid::File.new(@contents.read, descrption)
     id = self.class.mongo_client.database.fs.insert_one(grid_file)
     @id = id.to_s
   end
   ```

   ​

6. 全部的代码片段

   ```ruby
   def save
       unless persisted?
         f = @contents
         descrption = {}
         #Obtain the location
         gps = EXIFR::JPEG.new(f).gps
         @location = Point.new(:lng=>gps.longitude, :lat => gps.latitude)
         f.rewind
         descrption[:content_type] = "image/jpeg"
         
         if @location
           descrption[:metadata] = {}
           descrption[:metadata][:location] = @location.to_hash unless @location.nil?
         end
         if @contents
           grid_file = Mongo::Grid::File.new(@contents.read, descrption)
           id = self.class.mongo_client.database.fs.insert_one(grid_file)
           @id = id.to_s
         end 
       end
     end
   ```

   ​

