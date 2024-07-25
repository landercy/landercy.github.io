# SIMPLE JAVA APPLICATION WITHOUT 3RD PARTY LIBRARY

HelloWorld.java

注意文件名和类名必须一致，否则会报错：class HelloWorld is public, should be declared in a file named HelloWorld.java

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

Manifest.txt

```txt
Manifest-Version: 1.0
Main-Class: HelloWorld
```

```bash
javac HelloWorld.java
jar cvfm hello_world.jar Manifest.txt HelloWorld.class
java -jar hello_world.jar
```