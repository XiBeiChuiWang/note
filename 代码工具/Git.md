# Git

## 安装

在安装完成之后，我们需要执行两条命令来指定用户签名

```bash
git config --global user.name
git config --global user.email
```

## 基本操作

|              命令               |      作用      |
| :-----------------------------: | :------------: |
|  git config --global user.name  |  设置用户签名  |
| git config --global user.email  |  设置用户签名  |
|            git init             |  初始化本地库  |
|           git status            | 查看本地库状态 |
|         git add 文件名          |  添加到暂存区  |
| git commit -m "日志信息" 文件名 |  添加到本地库  |
|           git reflog            |  查看历史记录  |
|     git reset --hard 版本号     |    版本穿梭    |

## 分支

|         命令          |    作用    |
| :-------------------: | :--------: |
|     git branch -v     |  查看分支  |
| git branch 新的分支名 | 创建新分支 |
|  git checkout 分支名  |  切换分支  |
|   git merge 分支名    |  合并分支  |

## 远程库

|             命令             |          作用          |
| :--------------------------: | :--------------------: |
|        git remote -v         | 查看当前所有远程库别名 |
| git remote add 别名 远程链接 |        创建别名        |
|   git push 别名或链接 分支   |    推送本地库到远程    |
|   git pull 别名或链接 分支   |    拉取远程库到本地    |
|      git clone 远程地址      |        克隆代码        |

