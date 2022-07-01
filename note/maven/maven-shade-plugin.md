



maven-shade-plugin: 用来打可执行的 Jar 包(fat Jar).

[官方文档](https://maven.apache.org/plugins/maven-shade-plugin/)


# 一、将工程的部分 Jar 包 include/exclude 掉
```xml
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>2.4.3</version>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
            <configuration>
              <artifactSet>
                <excludes>
                  <exclude>classworlds:classworlds</exclude>
                  <exclude>junit:junit</exclude>
                  <exclude>jmock:*</exclude>
                  <exclude>*:xml-apis</exclude>
                  <exclude>org.apache.maven:lib:tests</exclude>
                  <exclude>log4j:log4j:jar:</exclude>
                </excludes>
              </artifactSet>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
</build>
```

# 二、解决版本冲突
把  com.google.common 重命名为 kino.myflink.com.google.common
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.1.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <relocations>
                    <relocation>
                        <pattern>com.google.common</pattern>
                        <shadedPattern>kino.myflink.com.google.common</shadedPattern>
                        <includes>
                            <include>com.google.common.collect.MapMaker</include>
                        </includes>
                    </relocation>
                </relocations>
            </configuration>
        </execution>
    </executions>
</plugin>
```
































