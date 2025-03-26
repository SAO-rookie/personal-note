## window安装
[windows下载地址]([GraphicsMagick - Browse /graphicsmagick-binaries at SourceForge.net](https://sourceforge.net/projects/graphicsmagick/files/graphicsmagick-binaries/))
## linux安装
```bash
apt install -y graphicsmagick
```
## 命令文档
[官方文档](http://www.graphicsmagick.org/utilities.html)

## java 调用
添加命令行调用工具包
```xml
<dependency>  
    <groupId>org.apache.commons</groupId>  
    <artifactId>commons-exec</artifactId>  
    <version>1.4.0</version>  
</dependency>
```
代码
```java
public static void main(String[] args)throws IOException {  
    CommandLine cmd = new CommandLine("gm");  
    cmd.addArgument("convert");  
    cmd.addArgument("1079629.png");  
    cmd.addArgument("-resize");  
    cmd.addArgument("600x600");  
    cmd.addArgument("1.jpg");  
  
    DefaultExecutor executor = DefaultExecutor.builder().get();  
    executor.setStreamHandler(new PumpStreamHandler(System.out, System.err)); // 重定向输出  
    int exitCode = executor.execute(cmd);  
  
    if (exitCode != 0) {  
        throw new IOException("生成失败 " + exitCode);  
    }  
}
```