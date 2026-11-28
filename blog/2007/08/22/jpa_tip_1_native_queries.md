# JPA Tip #1 - Native Queries

OK, native queries in JPA. Not really well documented. Of course, I understand that we really should not be doing that, but hey, there are times where it is just plain easier.

The following code snippet goes after an Oracle sequence and gets the value:

```java
EntityManager em = this.getEntityManager();
Query query = em.createNativeQuery(
    "SELECT BLOG_ITEM_SEQ.nextval FROM DUAL");
Vector blogItemRow = (Vector) query.getSingleResult();
Integer blogItemId = (Integer) blogItemRow.elementAt(0);
```

As you can see by reading the Java code, if you do a native query you can map the result to a Vector and then for each row you have a Vector for all the columns. In this particular case, only one, and the first column is an Integer.

*Posted August 22, 2007*

[Up](../../../)
