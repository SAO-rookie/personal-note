# Mac的💩处理
## git
全局删除.DS.store文件
```
# 配置核心文件
echo ".DS_Store" >> ~/.gitignore_global
# git全局配置
git config --global core.excludesfile ~/.gitignore_global

```
## 7zi