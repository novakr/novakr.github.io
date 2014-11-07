---
date: 2014-11-06 12:44:30+00:00
layout: post
title: spring集成mongodb
thread: 164
categories: mongodb
tags:  spring mongodb 程序
---

好久之前学习mongodb的时候搭建的spring集成mongodb例子

* 主要配置文件  
	- applicationContext.xml
<!--break-->
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
			http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
			http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
			http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
			http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
			">
	<context:property-placeholder
		ignore-resource-not-found="false" location="classpath:conf/web.properties" />
		
	<bean id="mongo" class="org.springframework.data.mongodb.core.MongoFactoryBean">
        <property name="host" value="${app.db.host}" />
        <property name="port" value="${app.db.port}" />
    </bean>

    <bean id="mongoUserCredentials" class="org.springframework.data.authentication.UserCredentials"> 
        <constructor-arg name="username" value="${app.db.username}" />
        <constructor-arg name="password" value="${app.db.password}" />
    </bean>

    <bean id="mongoDbFactory" class="org.springframework.data.mongodb.core.SimpleMongoDbFactory"> 
        <constructor-arg ref="mongo" />
        <constructor-arg name="databaseName" value="${app.db.database}" />
        <!-- constructor-arg name="credentials" ref="mongoUserCredentials" /> -->
    </bean>

    <bean id="mappingContext" class="org.springframework.data.mongodb.core.mapping.MongoMappingContext" />
     
     <!-- 去除_class属性 -->
    <bean id="defaultMongoTypeMapper" class="org.springframework.data.mongodb.core.convert.DefaultMongoTypeMapper">  
        <constructor-arg name="typeKey"><null/></constructor-arg>
     </bean>
     
    <bean id="mappingMongoConverter" class="org.springframework.data.mongodb.core.convert.MappingMongoConverter">  
        <constructor-arg name="mongoDbFactory" ref="mongoDbFactory" />  
        <constructor-arg name="mappingContext" ref="mappingContext" />  
        <property name="typeMapper" ref="defaultMongoTypeMapper" />  
     </bean>  
    
    <bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
        <constructor-arg name="mongoDbFactory" ref="mongoDbFactory" />  
        <constructor-arg name="mongoConverter" ref="mappingMongoConverter" />  
    </bean>

	<context:annotation-config />
	<context:component-scan base-package="a.b.c.dao" />
	<context:component-scan base-package="a.b.c.service" />
	
	</beans>

<!--break-->
- DAO
<!--break-->
	package a.b.c.dao;
	
	import java.util.Collection;
	import java.util.List;
	
	import org.springframework.data.mongodb.core.query.Criteria;
	import org.springframework.data.mongodb.core.query.Query;
	
	import a.b.c.domain.Entity;
	
	/**
	 * 数据库访问层底层接口类
	 * 
	 * 
	 * @param <T>
	 * @param <PK>
	 */
	public interface DAO<T extends Entity, PK> {

	/**
	 * 创建Mongodb中一个tableName的Collection
	 * 
	 * @param tableName
	 *            Collection的名称
	 */
	public abstract void createCollection(String tableName);

	/**
	 * 检查当前mongodb中以tableName命名的Collection是否存在
	 * 
	 * @param tableName
	 * @return
	 */
	public abstract boolean collectionExists(String tableName);

	/**
	 * 删除Mongodb中一个Collection表集合
	 * 
	 * @param tableName
	 *            collection的名称
	 */
	public abstract void dropCollection(String tableName);

	/**
	 * 查询一个Collection中的对象
	 * 
	 * @param tableName
	 *            Collection对象
	 * @param criteria
	 *            查询的条件
	 * @param clazz
	 *            T对象的Class
	 */
	public abstract T findOne(String tableName, Criteria criteria,
			Class<T> clazz);

	/**
	 * 查询一个Collection中的对象
	 * 
	 * @param tableName
	 *            Collection对象
	 * @param query
	 *            查询的条件
	 * @param clazz
	 *            T对象的Class
	 */
	public abstract T findOne(String tableName, Query query, Class<T> clazz);

	/**
	 * 查询符合条件的Collection中的多个对象
	 * 
	 * @param criteria
	 *            查询条件
	 * @param clazz
	 *            T对象的Class
	 * @return
	 */
	public abstract List<T> findList(Criteria criteria, Class<T> clazz);

	/**
	 * 查询符合条件的Collection中的多个对象
	 * 
	 * @param criteria
	 *            查询条件
	 * @param clazz
	 *            T对象的Class
	 * @return
	 */
	public abstract List<T> findList(Query query, Class<T> clazz);

	/**
	 * 
	 * @param query
	 * @param clazz
	 * @return
	 */
	public abstract T findAndRemove(Query query, Class<T> clazz);

	/**
	 * 查询符合条件的对象集合
	 * 
	 * @param tableName
	 * @param criteria
	 * @param clazz
	 */
	public abstract void remove(String tableName, Criteria criteria,
			Class<T> clazz);

	/**
	 * 保存Mongo对象的方法
	 * 
	 * @param tableName
	 *            Collection对象
	 * @param entity
	 *            T对象的Class
	 */
	public abstract void save(String tableName, T entity);
	
	public abstract void save(T entity);
	
	public abstract void saveAll(String tableName, Collection<T> entities);
	
	public abstract void saveAll(Collection<T> entities);
	
	}
<!--break-->
- AbstractBasicDAO
<!--break-->
	package a.b.c.dao;

	import java.util.Collection;
	import java.util.List;
	
	import javax.annotation.Resource;
	
	import org.springframework.beans.BeansException;
	import org.springframework.context.ApplicationContext;
	import org.springframework.context.ApplicationContextAware;
	import org.springframework.data.mongodb.core.MongoTemplate;
	import org.springframework.data.mongodb.core.query.Criteria;
	import org.springframework.data.mongodb.core.query.Query;
	
	import a.b.c.domain.Entity;
	
	public class AbstractBasicDAO<T extends Entity, PK> implements ApplicationContextAware, DAO<T, PK> {
	private ApplicationContext ctx = null;
	
	@Resource
	private MongoTemplate mongoTemplate;


	@Override
	public T findOne(String tableName, Criteria criteria, Class<T> clazz) {
		
		return mongoTemplate.findOne(new Query(criteria), clazz);
	//		return mongoTemplate.findOne(tableName, new Query(criteria), clazz);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see
	 * com.easyway.spring.mongodb.dao.DAO#createCollection(java.lang.String)
	 */
	public void createCollection(String tableName) {
		mongoTemplate.createCollection(tableName);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see
	 * com.easyway.spring.mongodb.dao.DAO#collectionExists(java.lang.String)
	 */
	public boolean collectionExists(String tableName) {
		return mongoTemplate.collectionExists(tableName);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see com.easyway.spring.mongodb.dao.DAO#dropCollection(java.lang.String)
	 */
	public void dropCollection(String tableName) {
		mongoTemplate.dropCollection(tableName);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see com.easyway.spring.mongodb.dao.DAO#findOne(java.lang.String,
	 * org.springframework.data.document.mongodb.query.Criteria,
	 * java.lang.Class)
	 */
	public T findOne(Criteria criteria, Class<T> clazz) {
		return mongoTemplate.findOne(new Query(criteria), clazz);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see com.easyway.spring.mongodb.dao.DAO#findOne(java.lang.String,
	 * org.springframework.data.document.mongodb.query.Query, java.lang.Class)
	 */
	public T findOne(String tableName, Query query, Class<T> clazz) {
	//		return mongoTemplate.findOne(tableName, query, clazz);
		return null;
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see
	 * com.easyway.spring.mongodb.dao.DAO#findList(org.springframework.data.
	 * document.mongodb.query.Criteria, java.lang.Class)
	 */
	public List<T> findList(Criteria criteria, Class<T> clazz) {
		return mongoTemplate.find(new Query(criteria), clazz);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see
	 * com.easyway.spring.mongodb.dao.DAO#findList(org.springframework.data.
	 * document.mongodb.query.Query, java.lang.Class)
	 */
	public List<T> findList(Query query, Class<T> clazz) {
		return mongoTemplate.find(query, clazz);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see
	 * com.easyway.spring.mongodb.dao.DAO#findAndRemove(org.springframework.
	 * data.document.mongodb.query.Query, java.lang.Class,
	 * org.springframework.data.document.mongodb.MongoReader)
	 */
	public T findAndRemove(Query query, Class<T> clazz) {
		return mongoTemplate.findAndRemove(query, clazz);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see com.easyway.spring.mongodb.dao.DAO#remove(java.lang.String,
	 * org.springframework.data.document.mongodb.query.Criteria,
	 * java.lang.Class)
	 */
	public void remove(String tableName, Criteria criteria, Class<T> clazz) {
		mongoTemplate.remove(new Query(criteria), clazz);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see com.easyway.spring.mongodb.dao.DAO#save(java.lang.String, T)
	 */
	public void save(String tableName, T entity) {
		//可能有问题
		mongoTemplate.save(entity, tableName);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see com.easyway.spring.mongodb.dao.DAO#save(T,
	 * org.springframework.data.document.mongodb.MongoWriter)
	 */
	public void save(T entity) {
		mongoTemplate.save(entity);
	}

	@Override
	public void saveAll(String tableName, Collection<T> entities) {
		mongoTemplate.insert(entities, tableName);
	}

	@Override
	public void saveAll(Collection<T> entities) {
		mongoTemplate.insertAll(entities);
	}

	/*
	 * (non-Javadoc)
	 * 
	 * @see com.easyway.spring.mongodb.dao.DAO#getApplicationContext()
	 */
	public ApplicationContext getApplicationContext() {
		return ctx;
	}


	/**
	 * 采用Spring自动注入方式实现
	 */
	public void setApplicationContext(ApplicationContext applicationCtx)
			throws BeansException {
		this.ctx = applicationCtx;
	}
	}
