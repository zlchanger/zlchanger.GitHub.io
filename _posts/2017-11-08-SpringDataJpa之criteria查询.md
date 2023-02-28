---
layout: post
title: JPA的Specifications查询
date: 2017/11/08
categories: blog
tags: [springboot]
description: 数据查询必备。

---
## Spring Data JPA
Spring-Data-JPA支持JPA2.0的Criteria查询，相应的接口是JpaSpecificationExecutor。
Criteria 查询：是一种类型安全和更面向对象的查询
 
这个接口基本是围绕着Specification接口来定义的， Specification接口中只定义了如下一个方法：

    Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
 
要理解这个方法，以及正确的使用它，就需要对JPA2.0的Criteria查询有一个足够的熟悉和理解，因为这个方法的参数和返回值都是JPA标准里面定义的对象。
 
### Criteria查询基本概念

Criteria 查询是以元模型的概念为基础的，元模型是为具体持久化单元的受管实体定义的，这些实体可以是实体类，嵌入类或者映射的父类。
* CriteriaQuery接口：代表一个specific的顶层查询对象，它包含着查询的各个部分，比如：select 、from、where、group by、order by等
注意：CriteriaQuery对象只对实体类型或嵌入式类型的Criteria查询起作用
* Root接口：代表Criteria查询的根对象，Criteria查询的查询根定义了实体类型，能为将来导航获得想要的结果，它与SQL查询中的FROM子句类似
* CriteriaBuilder接口：用来构建CritiaQuery的构建器对象
* Predicate：一个简单或复杂的谓词类型，其实就相当于条件或者是条件组合。

 1.Root实例是类型化的，且定义了查询的FROM子句中能够出现的类型。
 2.查询根实例能通过传入一个实体类型给 AbstractQuery.from方法获得。
 3.Criteria查询，可以有多个查询根。 
 4.AbstractQuery是CriteriaQuery 接口的父类，它提供得到查询根的方法。
 
 ### Criteria查询
 
> 基本对象的构建

 1.通过EntityManager的getCriteriaBuilder或EntityManagerFactory的getCriteriaBuilder方法可以得到CriteriaBuilder对象
 2.通过调用CriteriaBuilder的createQuery或createTupleQuery方法可以获得CriteriaQuery的实例
 3.通过调用CriteriaQuery的from方法可以获得Root实例
> 过滤条件

 1.过滤条件会被应用到SQL语句的FROM子句中。在criteria 查询中，查询条件通过Predicate或Expression实例应用到CriteriaQuery对象上。
 2.这些条件使用 CriteriaQuery .where 方法应用到CriteriaQuery 对象上
 3.CriteriaBuilder也作为Predicate实例的工厂，通过调用CriteriaBuilder 的条件方法（ equal，notEqual， gt， ge，lt， le，between，like等）创建Predicate对象。
 4.复合的Predicate 语句可以使用CriteriaBuilder的and, or andnot 方法构建。
 
 构建简单的Predicate示例：
 
     Predicate p1=cb.like(root.get(“name”).as(String.class), “%”+uqm.getName()+“%”);
     Predicate p2=cb.equal(root.get("uuid").as(Integer.class), uqm.getUuid());
     Predicate p3=cb.gt(root.get("age").as(Integer.class), uqm.getAge());
 构建组合的Predicate示例：
 
    Predicate p = cb.and(p3,cb.or(p1,p2));
 当然也可以形如前面动态拼接查询语句的方式，比如：
 
 java代码：
 
     Specification<UserModel> spec = new Specification<UserModel>() {  
     public Predicate toPredicate(Root<UserModel> root,  
             CriteriaQuery<?> query, CriteriaBuilder cb) {  
         List<Predicate> list = new ArrayList<Predicate>();  
               
         if(um.getName()!=null && um.getName().trim().length()>0){  
             list.add(cb.like(root.get("name").as(String.class), "%"+um.getName()+"%"));  
         }  
         if(um.getUuid()>0){  
             list.add(cb.equal(root.get("uuid").as(Integer.class), um.getUuid()));  
         }  
         Predicate[] p = new Predicate[list.size()];  
         return cb.and(list.toArray(p));  
     }  
     }; 
 也可以使用CriteriaQuery来得到最后的Predicate，示例如下：
   
 java代码：
 
     Specification<UserModel> spec = new Specification<UserModel>() {  
         public Predicate toPredicate(Root<UserModel> root,  
                 CriteriaQuery<?> query, CriteriaBuilder cb) {  
             Predicate p1 = cb.like(root.get("name").as(String.class), "%"+um.getName()+"%");  
             Predicate p2 = cb.equal(root.get("uuid").as(Integer.class), um.getUuid());  
             Predicate p3 = cb.gt(root.get("age").as(Integer.class), um.getAge());  
             //把Predicate应用到CriteriaQuery中去,因为还可以给CriteriaQuery添加其他的功能，比如排序、分组啥的  
             query.where(cb.and(p3,cb.or(p1,p2)));  
             //添加排序的功能  
             query.orderBy(cb.desc(root.get("uuid").as(Integer.class)));  
               
             return query.getRestriction();  
         }  
     };   

### 多表查询
> 多表连接查询稍微麻烦一些，下面演示一下常见的1:M，顺带演示一下1:1

> 使用Criteria查询实现1对多的查询

1.首先要添加一个实体对象DepModel，并设置好UserModel和它的1对多关系，如下：

    @Entity
    @Table(name="tbl_user")
    public class UserModel {
    @Id
    private Integer uuid;
    private String name;
    private Integer age;
    @OneToMany(mappedBy = "um", fetch = FetchType. LAZY, cascade = {CascadeType. ALL})
    private Set<DepModel> setDep;
    //省略getter/setter
    }
     
    @Entity
    @Table(name="tbl_dep")
    public class DepModel {
    @Id
    private Integer uuid;
    private String name;
    @ManyToOne()
      @JoinColumn(name = "user_id", nullable = false)
    //表示在tbl_dep里面有user_id的字段
    private UserModel um = new UserModel();
    //省略getter/setter
    }

2.配置好Model及其关系后，就可以在构建Specification的时候使用了，示例如下：

    Specification<UserModel> spec = new Specification<UserModel>() {
    public Predicate toPredicate(Root<UserModel> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
    Predicate p1 = cb.like(root.get("name").as(String.class), "%"+um.getName()+"%");
    Predicate p2 = cb.equal(root.get("uuid").as(Integer.class), um.getUuid());
    Predicate p3 = cb.gt(root.get("age").as(Integer.class), um.getAge());
    SetJoin<UserModel,DepModel> depJoin = root.join(root.getModel().getSet("setDep",DepModel.class) , JoinType.LEFT);
     
    Predicate p4 = cb.equal(depJoin.get("name").as(String.class), "ddd");
    //把Predicate应用到CriteriaQuery去,因为还可以给CriteriaQuery添加其他的功能，比如排序、分组啥 的
    query.where(cb.and(cb.and(p3,cb.or(p1,p2)),p4));
    //添加分组的功能
    query.orderBy(cb.desc(root.get("uuid").as(Integer.class)));
    return query.getRestriction();
    }};

接下来看看使用Criteria查询实现1:1的查询
1.在UserModel中去掉setDep的属性及其配置，然后添加如下的属性和配置：

    @OneToOne()
    @JoinColumn(name = "depUuid")
    private DepModel dep;
    public DepModel getDep() { return dep;}
    public void setDep(DepModel dep) {this.dep = dep;  }
2.在DepModel中um属性上的注解配置去掉，换成如下的配置：

    @OneToOne(mappedBy = "dep", fetch = FetchType. EAGER, cascade = {CascadeType. ALL})
3.在Specification实现中，把SetJoin的那句换成如下的语句：

    Join<UserModel,DepModel> depJoin =
    root.join(root.getModel().getSingularAttribute("dep",DepModel.class),JoinType.LEFT);
    //root.join(“dep”,JoinType.LEFT); //这句话和上面一句的功能一样，更简单

实现分组

    @Override
    public Page<Trace> findTracePage(int page, int size, SortModel sortModel,
          final Long companyId) {
       String sortCol = sortModel.getSortCol();
       if (StringUtils.isEmpty(sortModel.getSortCol())) {
          sortCol = "createTime";
       }
       Sort sort = new Sort(Direction.DESC, sortCol);
       if (sortModel.getSortDirect() != null && sortModel.getSortDirect().equals(SortModel.DIRECT_ASC)) {
          sort = new Sort(Direction.ASC, sortCol);
       }
       Pageable pageable = new PageRequest(page, size, sort);
       Page<Trace> list = traceRepository.findAll(new Specification<Trace>() {
          @Override
          public Predicate toPredicate(Root<Trace> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
             List<Predicate> list = new ArrayList<Predicate>();
             if (null != companyId) {
                list.add(cb.equal((root.get("shippingOrder").get("company").get("id").as(Long.class)), companyId));
             }
             Predicate[] p = new Predicate[list.size()];
             query.where(cb.and(list.toArray(p)));
             query.groupBy(root.get("shippingOrder").get("orderNumber"));
    
             //query.getGroupRestriction();
             return query.getGroupRestriction();
          }
    
       }, pageable);
       return list;
    }