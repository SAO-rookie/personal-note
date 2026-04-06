# Thumbnailator
[Thumbnailaor](https://github.com/coobird/thumbnailator/wiki/Examples#examples)是google的一个开源图形处理库，方便好用，功能强大,[ThumbnailAPI文档](https://coobird.github.io/thumbnailator/javadoc/0.4.21/)
## 1.导入依赖
使用maven导入依赖
```xml
<dependency>
    <groupId>net.coobird</groupId>
    <artifactId>thumbnailator</artifactId>
    <version>0.4.21</version>
    <scope>compile</scope>
</dependency>
```
## 2.快速使用
```java
public static void main(String[] args) {  
	Thumbnails.of(new File("original.jpg"))
        .size(160, 160)
        .toFile(new File("thumbnail.jpg"));
}
```
