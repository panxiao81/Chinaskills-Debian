# Debian 快速入门指南 for Chinaskills

本书为应学校所需，以全国职业院校技能大赛为主要范围，所写的一本 Debian 系 Linux 快速入门与参考书。

本书从基本系统安装讲起，并着重讲述网络服务配置与系统维护，适合 Linux 学习和运维学习者阅读。

本书将不再重复讲述 Linux 通用知识，尤其前几章，更适合具有一些 Linux 使用经验的人阅读。

本书主要参考 [Debian 管理员手册](https://www.debian.org/doc/manuals/debian-handbook/)写成，并按照原文需求使用 [GPL](https://www.gnu.org/licenses/) 与 [CC-BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) 方式放出。

由于笔者能力有限，书中有误的地方在所难免，如有读者发现问题，可到 [GitHub 项目](https://github.com/panxiao81/Chinaskills-Debian) 中提 `Issue`，我会在力所能及的范围内尽力改正。

## 开发

需要 Python 3 环境

### 环境准备

#### Windows

```sh
python -m venv venv
# Powershell
.\venv\Scripts\activate.ps1 
# or CMD
.\venv\Scripts\activate.bat

pip install -r requirements.txt
```

#### macOS & Linux

```sh
# Debian or Ubuntu Only
sudo apt install python3-venv

python3 -m venv venv

source venv/bin/active
pip install -r requirements.txt
```

### 本地开发

```sh
mkdocs serve
```

后访问 localhost:8000

### 编译

```sh
mkdocs build
```

生成的静态页面存放在 `site/` 文件夹下