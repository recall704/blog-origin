---
title: "权限、RBAC 和 ObjectPermission"
date: 2017-10-21T14:39:44+08:00
categories: 开发
tags: ["rabc","permission","权限"]
type: post
slug: permission-rbac-and-objectpermission
toc: true
---


## 一. 从简单权限控制到 RBAC

### 1. 只有一个权限

在系统初期，我们的用户是这样的

| id | 用户名 | 密码 | 是否管理员 |
| ---- |--------|--------| ----- |
|1 | admin | 123456 | True |
|2| test1 | 123123 | False |
|3| test2 | 123123 | False |


系统只有两类用户，管理员和普通用户，判断用户权限的时候，代码如下


```python
def func():
	user = User.objects.get(id=1)
	if user.is_admin:
	    do_someting()
	else:
	    pass
```





### 2. 少量权限

再后来，我们的系统需要三个权限，发布文章，修改文章，删除文章。

我们的数据库变成了这样



| id | 用户名 | 密码 | 是否允许发布文章 | 是否允许修改文章 | 是否允许删除文章 |
| ---- |--------|:--------:| :-----: |:-----: |:-----: |
|1| test1 | 123123 | True | False | False |
|2| test2 | 123123 | False | True | False |
|3| test3 | 123123 | False | False | True |



同样的，我们的代码也会多一些判断



```python

def func():
	user = User.objects.get(id=1)
	if user.is_add_allowed:
	    do_something1()
	if user.is_edit_allowed:
	    do_something2()
	if user.is_delete_allowed:
	    do_something3()

```



### 3. 多个权限

当我们的权限越来越多，增加字段已经无法满足了， 我们需要新的方式来完成这个功能



用户表：



| id | 用户名 | 密码 |
| --- | ------ | ----|
|1| test1 | 123123 |
|2| test2 | 123123 |
|3| test3 | 123123 |



权限表



| id | 用户id | 权限 |
| ---- |:--------:| :---:|
|1| 1 | add |
|2| 1 | edit |
|3| 2 | edit |
|4| 2 | delete |  



```python
def func():
	user = User.objects.get(id=1)
	permission_list = Permission.objects.get(user_id=user.id).values_list("permission")

	# sql : select permission_name from table_permission where user_id = 1;
	# permission_list = ['add', 'edit', ...]

	if 'add' in permission_list:
		do_add_action()
	if 'edit' in permission_list:
		do_edit_action()
	...

```



### 4. RBAC

RBAC 的全名叫做 Role-Based Access Control， 中文翻译为 `基于角色的访问控制`。



从使用的角度来看，上面我们的用户和权限表已经完全够用了。

但是：  
1. 当我们的权限特别多的时候（不只有 add, view, delete, edit, 还有很多其他权限的时候），用户的增长会给权限表带来指数级的增长。  
2. 很多用户的权限其实是一样的（user1 和 user2 都有同样的权限， user3 又和 user4 有相同的权限），权限表中包含了大量的冗余数据。  


这个时候，我们就很自然的想到 `分组` 这样一个概念。  
权限1 和 权限2 分为一个组 r1；  
权限3 和 权限4 分为一个组 r2；  

如果用户属于 组r1 ，那么，他就拥有权限1 和 权限2  
如果用户属于 组r2 ，那么，他就拥有权限3 和 权限4  

* user1
    * role1
        * permission1
        * permission2
        * permission3
    * role2
        * permission4
        * permission5
    * role3
        * permission6
* user2
    * role2
        * permission4
        * permission5
    * role3
        * permission6



这样，用户的权限，就是用户角色下的权限之和。



除了对权限进行分组外，我们还有对用户进行分组的需求，

这样，我们可以把角色的授权加入到用户组上。



* group1
    * role1
        * permission1
        * permission2
        * permission3
    * role2
        * permission4
        * permission5
    * role3
        * permission6
* group2
    * role2
        * permission4
        * permission5
    * role3
        * permission6



只要我们的用户属于某个用户组，那么他就拥有该用户组的所有权限





## 二、ObjectPermission

前面谈论的权限，基本上都是 `一对多`， 或者 `多对多` 的关系。  

在开发过程中，我们还会遇到一种特别的需求，那就是 `一个用户(或者一个对象)对于一个具体对象的权限`，而且在程序运行过程中，权限会动态添加和删除。  



这个时候，我们就会需要 ObjectPermission  



表示方式大概如下：


| id | 用户id | 对象 | 权限 |
| ---- |:--------:| :----:| :---:|
|1| 1 | car1 | drive |
|2| 1 | dog2 | walk |
|3| 2 | cat3 | kiss |
|4| 2 | teacher4 | teach  |



但我们都知道，对象是无法直接存储到数据库的，特别是对象的类型还不一样的时候。



### 1. 简单原型

我们可以尝试把对象类型也存储在数据库中，比如


| id   | 用户id    | 对象id | 对象类型 | 权限 |
| ---- |:--------:| :-----:| :---:  | :----: |
|  1   | 1        | 1      | car    | drive |  
|  2   | 1        | 1      | dog    | walk |
|  3   | 2        | 2      | cat    | kiss |
|  4   | 2        | 4      | teacher  | teach  |


```python
    user = User.objects.get(id=1)
    car = Car.objects.get(id=1)
    check_permisson(user, car, 'drive')
```

假设我们在定义 数据模型的时候， Car 有一个 name 属性，值为 car   
那我们就可以直接查数据库，判断 user 对 car 是否有 drive 权限。



```python

"SELECT * FROM Permissions WHERE uid={user_id} AND object_id={object_id} AND object_type={object_type} \

AND permission_name={permission_name}".format(
    user_id=user.id,
    object_id=car.id,
    object_type = getattr(car, 'name'),
    permission_name='drive'
)

```



### 2. django 原型

在 django 中，有一种叫做 content_type 的概念，我们可以参考来实现一个 `对象权限控制`。

具体可以参考  `django-guardian`