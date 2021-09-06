# Redis - RedisTemplate及4种序列化方式深入解读

# 概述

使用Spring 提供的 Spring Data Redis 操作redis 必然要使用Spring提供的模板类 RedisTemplate， 今天我们好好的看看这个模板类 。

------

# RedisTemplate

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/2ac0681f5b76255932c9fbbdec90a9f8.png?imageView2/2/w/1620)

看看4个序列化相关的属性 ，主要是 用于 KEY 和 VALUE 的序列化 。 举个例子，比如说我们经常会将POJO [对象存储](https://cloud.tencent.com/product/cos?from=10680)到 Redis 中，一般情况下会使用 JSON 方式序列化成字符串，存储到 Redis 中 。

Spring提供的Redis数据结构的操作类

- ValueOperations 类，提供 Redis String API 操作
- ListOperations 类，提供 Redis List API 操作
- SetOperations 类，提供 Redis Set API 操作
- ZSetOperations 类，提供 Redis ZSet(Sorted Set) API 操作
- GeoOperations 类，提供 Redis Geo API 操作
- HyperLogLogOperations 类，提供 Redis HyperLogLog API 操作

------

# StringRedisTemplate

再看个常用的 StringRedisTemplate

RedisTemplate 支持泛型，StringRedisTemplate K V 均为String类型。

`org.springframework.data.redis.core.StringRedisTemplate` 继承 RedisTemplate 类，使用 `org.springframework.data.redis.serializer.StringRedisSerializer` 字符串序列化方式。

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/d7d00fc959ab788e6521a074844e87f4.png?imageView2/2/w/1620)

------

# RedisSerializer 序列化 接口

RedisSerializer接口 是 Redis 序列化接口，用于 Redis KEY 和 VALUE 的序列化

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/5784b97087175c3ea87b7fbdc5e4fcb8.png?imageView2/2/w/1620)

RedisSerializer 接口的实现类 如下 

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/9a2f0857fcf24833583849f0a60215b0.png?imageView2/2/w/1620)

归类一下

- JDK 序列化方式 （默认）
- String 序列化方式
- JSON 序列化方式
- XML 序列化方式

------

## JDK 序列化方式 （默认）

`org.springframework.data.redis.serializer.JdkSerializationRedisSerializer` ，默认情况下，RedisTemplate 使用该数据列化方式。

我们来看下源码 `RedisTemplate#afterPropertiesSet()`

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/f87c912f6f30d22d825120ef8b26e428.png?imageView2/2/w/1620)

 Spring Boot 自动化配置 RedisTemplate Bean 对象时，就未设置默认的序列化方式。

 绝大多数情况下，不推荐使用 JdkSerializationRedisSerializer 进行序列化。主要是不方便人工排查数据。

我们来做个测试

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/5a344809f7d63d8913648a07aecf4919.png?imageView2/2/w/1620)

运行单元测试 

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/e5ddf11dae102c16209153115424a70e.png?imageView2/2/w/1620)

看不懂呀 ，老哥

KEY 前面带着奇怪的 16 进制字符 ， VALUE 也是一串奇怪的 16 进制字符 。。。。。

>  为什么是这样一串奇怪的 16 进制？ ObjectOutputStream#writeString(String str, boolean unshared) 实际就是标志位 + 字符串长度 + 字符串内容 

KEY 被序列化成这样，线上通过 KEY 去查询对应的 VALUE非常不方便，所以 KEY 肯定是不能被这样序列化的。

VALUE 被序列化成这样，除了阅读可能困难一点，不支持跨语言外，实际上也没还OK。不过，实际线上场景，还是使用 JSON 序列化居多。

------

## String 序列化方式

`org.springframework.data.redis.serializer.StringRedisSerializer` ，字符串和二进制数组的直接转换

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/c8c9354365d73ae4a3e77ed60facce69.png?imageView2/2/w/1620)

 绝大多数情况下，我们 KEY 和 VALUE 都会使用这种序列化方案。

------

## JSON 序列化方式

`org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer` 使用 Jackson 实现 JSON 的序列化方式，并且从 Generic 单词可以看出，是支持所有类。

```javascript
public GenericJackson2JsonRedisSerializer(@Nullable String classPropertyTypeName) {

			.....
			..... 
		if (StringUtils.hasText(classPropertyTypeName)) {
			mapper.enableDefaultTypingAsProperty(DefaultTyping.NON_FINAL, classPropertyTypeName);
		} else {
			mapper.enableDefaultTyping(DefaultTyping.NON_FINAL, As.PROPERTY);
		}
	}
```

 classPropertyTypeName 不为空的话，使用传入对象的 classPropertyTypeName 属性对应的值，作为默认类型（Default Typing） ，否则使用传入对象的类全名，作为默认类型（Default Typing）。

我们来思考下，在将一个对象序列化成一个字符串，怎么保证字符串反序列化成对象的类型呢？Jackson 通过 Default Typing ，会在字符串多冗余一个类型，这样反序列化就知道具体的类型了



先说个结论

- 标准JSON

```javascript
{
  "id": 100,
  "name": "小工匠",
  "sex": "Male"
}
```

- 使用 Jackson Default Typing 机制序列化

```javascript
{
  "@class": "com.artisan.domain.Artisan",
  "id": 100,
  "name": "小工匠",
  "sex": "Male"
}
```

------

### 示例

测试一把

【配置类】

```javascript
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        // 创建 RedisTemplate 对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();

        // 设置 RedisConnection 工厂。 它就是实现多种 Java Redis 客户端接入的秘密工厂
        template.setConnectionFactory(connectionFactory);

        // 使用 String 序列化方式，序列化 KEY 。
        template.setKeySerializer(RedisSerializer.string());

        // 使用 JSON 序列化方式（库是 Jackson ），序列化 VALUE 。
        template.setValueSerializer(RedisSerializer.json());

        return template;
    }
```

【单元测试】

```javascript
   @Test
    public void testJacksonSerializer() {
        Artisan artisan  = new Artisan();
        artisan.setName("小工匠");
        artisan.setId(100);
        artisan.setSex("Male");
        // set
        redisTemplate.opsForValue().set("artisan", artisan);
    }
```

【结果】

![img](https://ask.qcloudimg.com/http-save/yehe-8916337/a49c5690abd715dfc18f4238a3713444.png?imageView2/2/w/1620)

是不是多了@class 属性，反序列化的对象的类型就可以从这里获取到。

@class 属性看似完美解决了反序列化后的对象类型，但是带来 JSON 字符串占用变大，所以实际项目中，我们很少采用 Jackson2JsonRedisSerializer

------

## XML 序列化方式

`org.springframework.data.redis.serializer.OxmSerializer`使用 Spring OXM 实现将对象和 String 的转换，从而 String 和二进制数组的转换。 没见过哪个项目用过，不啰嗦了