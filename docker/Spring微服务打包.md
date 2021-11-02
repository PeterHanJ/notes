# Springboot微服务打包镜像

1. 构建Springboot项目

   ```java
   @RestController
   public class HelloController {
       @RequestMapping("/hello")
       public String hello(){
           return "Hello,Spring boot";
       }
   }
   
   ```

   

2. 打包应用

3. 编写dockerfile

   ```shell
   FROM java:8
   
   COPY *.jar /app.jar
   
   CMD ["--server.port=8080"]
   
   EXPOSE 8080
   
   ENTRYPOINT ["java","-jar","/app.jar"]
   ```

   

4. 构建镜像

   ```shell
   # Build images
   docker build -t springbootdemo .
   
   # image build successful
   [root@venhjuhost springbootdemo]# docker images
   REPOSITORY       TAG        IMAGE ID       CREATED          SIZE
   springbootdemo   latest     2c3d6f4b9413   28 seconds ago   661MB
   
   
   ```

   

5. 发布运行

```shell
# run
[root@venhjuhost springbootdemo]# docker run -d -P --name demo springbootdemo
2cac6fe089ed85b9bb792dda006cf3be2c8867d0b1882609f0511cf5a9b2a220
[root@venhjuhost springbootdemo]# docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                                         NAMES
2cac6fe089ed   springbootdemo   "java -jar /app.jar …"   3 seconds ago   Up 3 seconds   0.0.0.0:49157->8080/tcp, :::49157->8080/tcp   demo

# curl hello api
[root@venhjuhost springbootdemo]# curl localhost:49157/hello
Hello,Spring boot[root@venhjuhost springbootdemo]#
```

