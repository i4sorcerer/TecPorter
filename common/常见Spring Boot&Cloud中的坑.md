# 常见Spring Boot&Cloud中的坑

1. @Value("server.port")不一定靠谱，比如server.port=0的情况，取出来的值就是0

   可以注入 Environment bean 然后荣国getProperty("local.server.port")进行获取

2. @LocalServerPort也不一定靠谱，因为注入阶段，属性，local.server.port不一定存在

3. spring cloud+Netflix Ribbon有30秒的延迟

4. 