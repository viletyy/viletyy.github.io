---
layout: post
title: "Rails: ORM实现union(联集)"
date:       2019-04-14 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Rails
---
##如何在rails中的ORM实现union(联集)

Rails 的 ORM 提供很便利的语法让工程师可以轻松地对资料库作查询，但是在某些场景里仍让工程师们感到就有些美中不足，例如: 联集(UNION)。

假定系统需要一个 my tracking list 的页面，这个页面必需列出 manager 和 tracking product，并且针对它们去做更进一步的查询操作，如列出 publish 的 product，你会怎么设计?

似乎这一切听起来令人头疼，但看完后面的范例，你也能了解实现联集的作法。



> product.rb

```ruby
#  name         :string(255)

#  description  :text

#  status       :string(255)

#  manager_id   :integer

 
class Product < ActiveRecord::Base 
  belongs_to :manager, :foreign_key => :manager_id, :class_name => "User"
  
  has_many :tracking_lists
  has_many :tracking_users, :through => :tracking_lists, :source => :user  
end
```

> user.rb

```ruby
class User < ActiveRecord::Base
  has_many :products 
  
  has_many :tracking_lists
  has_many :tracking_products, :through => :tracking_lists, :source => :product
end
```

> track_list.rb

```ruby
#  id           :integer

#  user_id      :integer

#  product_id   :integer

 
class TrackList < ActiveRecord::Base
  belongs_to :user
  belongs_to :product
end
```

### 起初我们会想到 Array 的简单作法

```ruby
@products = Set.new
manage_products = @user.products.is_publish
tracking_products = @user.tracking_products.is_publish

@products.merge(manage_products)
@products.merge(tracking_products)
```

维护性: 低 

- (优) 写起来快，几乎不用思考就写完了...

- (劣) 需要下两道 query，且无法合并一起查询 

- (劣) Array 的 instance 无法做更进一步的查询操作，例: order(:created_at)...etc 。

- (劣) 速度比较慢 但我们始终觉得这样的做法效率太差，且维护性非常的低...

### 然后我们想到 find_by_sql + left join 的作法

```ruby
@products = Product.find_by_sql(["
    SELECT p0.* FROM product as p0 
    LEFT JOIN track_lists as t1 on t1.product_id = p0.id AND t1.user_id = p0.manager_id
    where p0.manager_id = ? AND p0.status = 'published';
    ", user.id])
```

维护性: 中 

- (优) 只有一道 query

- (劣) 需以 product 的 query 条件为主，当 user 没有被 assigned manage 某样商品时，结果就会为空

- (劣) 由于 find_by_sql 出来的结果是 Array，无法做更进一步的查询操作，例: order(:created_at)...etc 。

-  然而这种解法还是不够方便，且当有不同的 query 需求，例如: is_draft?，我们就必须再重写一次 method。

### 最后我们发现可以这样做 from + union

```ruby
manage_products_sql = @user.products.to_sql
tracking_products_sql = @user.tracking_products.to_sql

@products = Product.from("( #{ manage_products_sql } UNION #{ tracking_products_sql } ) AS products").is_publish
```

维护性: 高 

- (优) 只有一道 query 

- (优) from 出来的结果是 ActiveRecord::Relation，我们可以续用 QueryMethods 做更进一步的查询操作 

- (优) 简单好维护 备注: to_sql 只会产生 query 的字串，并不会实际去下 query UNION 是 mysql 的联集语法，且预设的 distinct 只会取一次相同的资料