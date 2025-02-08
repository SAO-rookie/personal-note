# GO打包
在window平台 打包出linux可执行文件
1.请在对应项目地址运行命令
```
set GOARCH=amd64
go env -w GOARCH=amd64
set GOOS=linux
go env -w GOOS=linux
```
2.进行交叉编译： 在你的Go项目目录下，运行以下命令：
```
go build -o output_filename
```
3.还原环境变量
```
set GOOS=
set GOARCH=
```

在Linux平台 打包出window可执行文件
1.请在对应项目地址运行命令
```
export GOOS=windows
go env -w GOOS=windows
export GOARCH=amd64
go env -w GOARCH=amd64
```
2.进行交叉编译： 在你的Go项目目录下，运行以下命令：
```
go build -o output_filename.exe
```
3.还原环境变量
```
export GOOS=
export GOARCH=
```