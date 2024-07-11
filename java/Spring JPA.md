# Jpa

## 默认方法

    - save 会根据Id 查询是否已存在决定使用insert还是update；
    	- 若参数Entity部分属性为null则修改数据库数据字段为null，若想只修改部分参数，需要先查询数据将不需要修改字段值进行填充;
    	- Jpa不能够将查询得到的实体更改主键值（id）后进行save()，只能够创建新的实体填充需要的参数去save();

## 实体关系

    ### @OneToOne
    	- save()实体时，需要实体间相互填充，反而表之间关联的属性并不是必须的;（在OneToMany具有同等效益）

# Bug记录

## Jpa

    ### org.hibernate.LazyInitializationException异常解决办法
    	- @OneToMany()(mappedBy = "customized",cascade = CascadeType.ALL,fetch = FetchType.EAGER)  需要使用“customized” 对应映射表@ManyToOne注解标注的字段名； 重要：@OneToMany fetch需要设置为 EAGER （立刻加载）
    	- @ManyToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY) 需要配合 @JoinColumn(name = "customized_id", referencedColumnName = "id")限定联系表字段映射，'customized_id'对应联系表@OneToMany相关联的字段  重要：@ManyToOne fetch则设置为懒加载
    	
    ### 方法抛出'java.lang.stackOverflowError"异常 无法对...Object.toString()求值
    	- bug产生原因，关联子查询，children 以及 parent，导致toString 输出parent字段时进入死循环，堆栈溢出。
    	- 修复 自定义toString()方法。

    ### 主附表关联添加，当附属表需要主表添加完成后生成的某个字段作为自身属性时。
    	- @MapsId 可以完成在附属表sql执行前将主表添加完成后生成的字段值注入字表参数中。例主表Id = 附表Id，主表添加完成后生成Id值，注入到附表Object当中，继而添加附表数据。(@OneToOne  @OneToMany 两种关系中见)
    ### entity实体子类实体注入若是lazy，子类方法会抛出 'org.hibernate.LazyInitializationException' 异常。 无法对 com.wesoft.cloud.platform.commons.entity.UserObject.toString() 求值；但方法体若是标注@Transactional(readOnly=true)则Jpa会进行子类注入。

