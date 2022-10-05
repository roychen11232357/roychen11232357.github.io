---
title: 使用maven-shade-plugin來單獨升級dependency
date: 2022-10-05 17:14:59
tags:
- java
- maven
---
身為兼差的java開發者, 遇到maven問題時, 總是很崩潰, 

這次的問題是要魔改Apache livy這開源框架時, 想單獨對某個maven project中的相依套件做升級,

改完該maven project的pom之後, 還是吃到其他maven project的相依套件, 後來發現可以用maven-shade-plugin來避免相依套件的共用, shade就是影子的意思, 蠻名符其實的!

```
<plugin>
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
                    <includes>
                        <include>com.google.code.gson:gson</include>
                    </includes>
                </artifactSet>
                <relocations>
                    <relocation>
                        <pattern>com.google.gson</pattern>
                        <shadedPattern>shaded.com.google.gson</shadedPattern>
                    </relocation>
                </relocations>
            </configuration>
        </execution>
    </executions>
</plugin>
```