# Mac的💩处理
## git
全局隔离💩文件
```
# 配置核心文件
echo ".DS_Store" >> ~/.gitignore_global
# git全局配置
git config --global core.excludesfile ~/.gitignore_global

```
## 7zip
打包压缩包时，隔离💩文件
```
7z a -r output.7z . -xr!*.DS_Store
```
