# Spring对事务的支持

[TOC]

---

## 1. Spring中事务的两种管理方式

### 1.1. 编程式事务

通过使用`TransactionTemplate`或`TransactionManager`来手动管理事务的开启、提交或回滚，实际工作中由于过于繁琐，很少使用。

1. 使用`TransactionTemplate`管理事务的代码如下：

```java
	//方式一：使用TransactionTemplate事务模板
	@Autowired
	private TransactionTemplate transactionTemplate;

	public void testTransaction() {
		transactionTemplate.execute(new TransactionCallbackWithoutResult() {
			@Override
			protected void doInTransactionWithoutResult(TransactionStatus status) {
				try {
					//do dabatase opertions
				} catch (Exception e) {
					status.setRollbackOnly();
				}
			}

		});
	}
	
```

2. 使用`TransactionManager`管理事务的代码如下：

```java
    //方式二：使用TransactionManager事务管理器,这里使用的PlatformTransactionManager
	@Autowired
	private PlatformTransactionManager platformTransactionManager;
	
	public void testTransaction2() {
		TransactionStatus status = platformTransactionManager.getTransaction(new DefaultTransactionDefinition());
		try {
			//do dabatase opertions
			platformTransactionManager.commit(status);
		} catch (Exception e) {
			platformTransactionManager.rollback(status);
		}
	}
```



### 1.2.声明式事务

使用@`Transaction`注解表示方法，最常用的方法。此种方法代码入侵性最小，原理上是通过**AOP**实现的。实例代码如下

```java
	@Transactional(propagation = Propagation.REQUIRED)
	public void testTransaction3() {
		//do databse opertions
        
	}
```



## 2. Spring中事务管理接口

Spring事务中定义了3个重要的接口：

- `PlatformTransactionManager`: 事务管理器，定义了事务的管理器接口，具体的实现交由各个平台来完成，如有JDBC(DataSourceTransactionManager),Hibernate框架下(HibernateTransactionManager)、JPA(JpaTransactionManager)、Jta(JtaTransactionManager,支持分布式事务)等等。
- `TransactionDefinition`: 事务定义信息，定义了事务的特性：隔离级别、传播行为、是否只读、回滚规则、超时策略。

- `TransactionStatus`: 事务运行状态。

![image-20210317013055293](C:\Users\Yegenyao\AppData\Roaming\Typora\typora-user-images\image-20210317013055293.png)



### 2.1 事务传播行为

TransactionDefinition中定义了7种事务的传播行为：

#### **1. REQUIRED**

支持当前事务，如果没有，则创建一个新的事务。

> <img src="https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/images/tx_prop_required.png" alt="tx prop required"  />
>
> 使用`PROPAGATION_REQUIRED`时，两个事务同处于一个事务中。
>
> 如果一个事务方法1调用另外一个事务方法2，事务方法1和事务方法2都会创建一个逻辑事务作用域。每一个作用域都独立地可以决定自己的事务回滚状态，并且外部的事务作用域独立于内部事务作用域。但由于`PROPAGATION_REQUIRED`的行为特性，事务A和事务同属于一个物理事务，内部事务标记为回滚时，会影响外部事务的提交，即便外部事务标记为提交。这时一个`UnexpectedRollbackException`会被抛出给调用者，避免实际上没提交的事务被调用方误解为已提交。



#### 2. REQUIRES_NEW

创建一个新的事务，如果当前存在一个事务，则会挂起当前事务。

> ![tx prop requires new](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/images/tx_prop_requires_new.png)
>
> 途中描述了事务方法2会运行在一个与事务方法1不同的事务中，并且执行事务方法2的时候，事务方法1会被挂起。
>
> 和`PROPAGATION_REQUIRES`相比，`PROPAGATION_REQUIRES_NEW`则是为每一个事务方法创建完全独立的事务。在这种情况下，每个底层的事务独立的提交或回滚，并且互不影响，内部事务。

#### 3. NESTED

> 使用一个可以带保存点（savepoint）的事务，事务能回滚到某个保存点。内部事务发生了回滚，外部事务仍能够继续进行事务的提交。这个设置对应于JDBC的保存点，这个配置只支持JDBC事务。

#### 4. SUPPORTS

支持在当前事务事务中运行，或者在当前没有事务时，则以非事务方式运行。

#### 5. MANDATORY

支持使用事务，如果不存在事务则抛出异常。

#### 6. NOT_SUPPORTED

不以事务方式运行，如果当前有事务则挂起当前事务。

#### 7. NEVER

不以事务运行，如果当前有事务，则抛出异常。



参考地址：

1. Spring官网