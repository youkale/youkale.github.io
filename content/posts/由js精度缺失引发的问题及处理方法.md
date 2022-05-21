---
title: 由js精度缺失引发的问题及处理方法
date: 2018-11-18 12:10:54
tag:
---

# 问题场景

    在使用java的Long类型过程中,当这个Long类型数值超过 1^0+1^1+1^2+...1^58的范围后就会引发精度丢失, 详见JS[IEEE754规范](https://en.wikipedia.org/wiki/IEEE_754)
    常见的现象就是比如:1000967854800048128,通过js转换成Number类型成为了:1000967854800048100, 由此会引发一系列问题,比如当年著名的爱国者导弹.
    因为精度缺失造成问题,大多数情况发生在前后端交互过程中,那么下面就分别说说处理方法

## 普通场景的处理方法

> 简单的处理方法，jackson,gson等json工具包已经包含了Long类型处理方法了,基本上都是转String.

### gson处理方法

    GsonBuilder builder = new GsonBuilder();
    //默认将所有的Long转换成String
    builder.setLongSerializationPolicy(LongSerializationPolicy.STRING);

### jackson处理方法

    ObjectMapper objectMapper = new ObjectMapper();
    SimpleModule module = new SimpleModule();
    module.addSerializer(Long.class, ToStringSerializer.instance);
    module.addSerializer(Long.TYPE, ToStringSerializer.instance);

## 高级场景处理

>为了避免将其他无需转换成String的Long类型转换,这里提供一个高级的用法，
思路很简单就是定义一个Id类型,通过实现Id类型的序列化和反序列化功能即可进行实现.
下面就用Gson和jackson来演示

### 核心代码

        //Gson
        GsonBuilder builder = new GsonBuilder();
        builder.registerTypeAdapter(Id.class, (JsonSerializer<Id>) (src, typeOfSrc, context) ->
                new JsonPrimitive(src.getVal()));
        builder.registerTypeAdapter(Id.class, (JsonDeserializer<Id>) (json, typeOfT, context) ->
                new Id(json.getAsLong()));
        Gson gson = builder.create();
        User user = new User();
        user.setUserId(new Id(11111L));
        user.setUserName("youkale.github.io");
    
        String s = gson.toJson(user);
        System.out.println(s);
    
        User user1 = gson.fromJson(s, User.class);
        System.out.println("gson  " + user1);
    
        //Jackson
        ObjectMapper objectMapper = new ObjectMapper();
        SimpleModule module = new SimpleModule();
        module.addSerializer(Id.class, new IdJacksonSerializer(Id.class));
        module.addDeserializer(Id.class, new IdJacksonDeserializer(Id.class));
        objectMapper.registerModule(module);
        String s1 = objectMapper.writeValueAsString(user);
        System.out.println(s1);
        User user2 = objectMapper.readValue(s1, User.class);
        System.out.println("jackson  " + user2);

### 定义类User

    public static class User {
    
        private Id userId;
    
        private String userName;
    
        public Id getUserId() {
            return userId;
        }
    
        public void setUserId(Id userId) {
            this.userId = userId;
        }
    
        public String getUserName() {
            return userName;
        }
    
        public void setUserName(String userName) {
            this.userName = userName;
        }
    
        @Override
        public String toString() {
            return "User{" +
                    "userId=" + userId +
                    ", userName='" + userName + '\'' +
                    '}';
        }
    }

### 定义Id类

    public static class Id {
    
        private final Long val;
    
        public Long getVal() {
            return val;
        }
    
        public Id(Long val) {
            this.val = val;
        }
    
        @Override
        public String toString() {
            return String.valueOf(val);
        }
    }

### jackson序列反序列实现

    public static class IdJacksonDeserializer extends StdDeserializer<Id> {
        public IdJacksonDeserializer(Class<?> vc) {
            super(vc);
        }
    
        @Override
        public Id deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
            return new Id(p.getValueAsLong());
        }
    }
    
    public static class IdJacksonSerializer extends StdSerializer<Id> {
    
        public IdJacksonSerializer(Class<Id> t) {
            super(t);
        }
        @Override
        public void serialize(Id value, JsonGenerator gen, SerializerProvider provider) throws IOException {
            gen.writeString(String.valueOf(value.getVal()));
        }
    }

# 总结

>从Gson和jackson的API看出这类的功能老早就由大神们想到了,有时候总在感叹,自己多渺小.