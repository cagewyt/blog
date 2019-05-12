Spoiled by Hibernate, we usually let Hibernate to generate the id for the entity object, like the following:
```java
@MappedSuperclass 
public abstract class GuidEntity 
{ 
	@Id @GeneratedValue(generator = "uuid2") 
	@GenericGenerator(name = "uuid2", strategy = "uuid2")
	@Column(unique = true, nullable = false) 
	private String id; 
	
	//getter, setter, ... 
}
```
When you calls .save or .persist via Hibernate session or EntityManager, an id is generated automatically for the entity object using the specified generator by Hibernated. This is a neat feature but also means you don't have control on what the value of the id is.

What if you want Hibernate to use your specified id to persist the entity object? It requires a little twist.

### 1. Why do you even want to do this?

Simple, imagine your application has tons of data and it support bulk data import/export via xls, csv, whatever format you like. Your customer plays with your system for a while with some data on a free trail. Now they want seriously become your customer and want to integrate their business data from their other applications with yours. They have a requirement to bulk import data with their ids into your application in order to maintain the data integrity across all applications of the customer.

Another scenario is that your application does pull/push data from/to another web service (maybe another product of your company), and they share the same domain schema for both applications. It is a requirement to have the same id to represent the same data information across the two applications. Then you want to sync the data from one application to another using the same id.

### 2. How?

Write a customized id generator class.
```java
import java.io.Serializable; 
import org.hibernate.id.UUIDGenerator; 
import org.hibernate.HibernateException; 
import org.hibernate.engine.spi.SessionImplementor; 

public class UserSpecifiedIdOrGenerate extends UUIDGenerator 
{ 
	@Override 
	public Serializable generate(final SessionImplementor session, final Object obj) throws HibernateException 
	{ 
		if (obj == null) { 
			throw new HibernateException(new NullPointerException()); 
		} 

		final Serializable id = session.getEntityPersister(null, obj).getClassMetadata().getIdentifier(obj, session); 

		return id == null? super.generate(session, obj) : id; 
	} 
}
```

Then, update your entity id annotation.
```java
@MappedSuperclass 
public abstract class GuidEntity 
{ 
	@Id @GeneratedValue(strategy = GenerationType.IDENTITY, generator = "IdOrGenerated") 
	@GenericGenerator(name = "IdOrGenerated", strategy = "com.yourdomain.UserSpecifiedIdOrGenerate") 
	@Column(unique = true, nullable = false) 
	provate String id; 

	//getter, setter, ... 
}
```

Now you can use this entity class as usual. For example, you can create an entity object, set its id to whatever you want, and call getCurrentSession().save(obj). The customized id generator class will take care of the id generation.

Important thing is that you should not use the convenient method getCurrentSession().saveOrUpdate(obj). You may get StaleObjectStateException: Row was updated or deleted by another transaction (or unsaved-value mapping was incorrect).

This is because the id is set by the user, the non-empty id confuses Hibernate to use update(obj) to database instead of save, but the object with the user specified id does not exist in the database yet.

**Note:**

1.  strategy in @GenericGenerator should use the full qualified name with package:com.yourdomain.UserSpecifiedIdOrGenerate. Otherwise, you may get org.hibernate.MappingException: Could no  
    t instantiate id generator.
2.  If you don't use uuid2 as id, you can extends other Hibernate id generators, for example, SequenceGenerator.
3.  Why not use merge method? you definitely can create an entity object, set your id, and call merge(obj) to get the object attached to the current session. However, when you later on save this object, Hibernate will still generate id for you, not using the one you set. In other words, you almost have to use your customized id generator.

This article was originally published on [https://coddicting.wordpress.com/](https://coddicting.wordpress.com/).

**Reference**:

1.  http://stackoverflow.com/questions/3194721/bypass-generatedvalue-in-hibernate-merge-data-not-in-db/8535006#8535006
2.  https://forum.hibernate.org/viewtopic.php?p=2428082