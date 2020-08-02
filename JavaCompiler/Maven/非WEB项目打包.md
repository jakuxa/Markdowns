## 非WEB项目打包

> 特指非Spring-Boot项目

### 使用Maven-Assembly-Plugin打包

**记得修改入口类**

```xml
<!--pom.xml-->

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.1</version>
      <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <skip>true</skip>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-assembly-plugin</artifactId>
      <!--尽量用3.1以上的新版本，老版本会强制添加非常多多余的依赖-->
      <version>3.3.0</version>
      <configuration>
        <descriptorRefs>
          <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
          <manifest>
            <!--入口类，切记修改-->
            <mainClass>DecodeDBInfo</mainClass>
          </manifest>
        </archive>
      </configuration>
      <executions>
        <execution>
          <id>make-assembly</id>
          <phase>package</phase>
          <goals>
            <goal>single</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```



### 删除传递依赖（依赖的依赖）

```xml
<dependency>
  <groupId>pentaho-kettle</groupId>
  <artifactId>kettle-core</artifactId>
  <version>9.1.0.0-295</version>
  <exclusions>
    <exclusion>
      <!--*表示排除此包的所有传递依赖-->
      <groupId>*</groupId>
      <artifactId>*</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

