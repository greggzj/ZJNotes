

# ubuntu 18 安装python3.7

## 下载.tar.gz解压编译安装

ubuntu18.04.3自带python3.6.9,如果要下载python3.7可以下载tar包解压安装：

- sudo apt update
- sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget
- cd /tmp
- wget https://www.python.org/ftp/python/3.7.5/Python-3.7.5.tgz

- 解压  tar –xf Python-3.7.5.tgz
- cd python-3.7.5
- ./configure ––enable–optimizations

The ./configure command evaluates and prepares Python to install on your system. Using the ––optimization option speeds code execution by 10-20%.

- 产生另一个python实例(python3.7)   sudo make altinstall

推荐使用altinstall命令来产生另一个Python实例而不是覆盖原先

- python3.7 --version -->ok!


[参考链接](https://phoenixnap.com/kb/how-to-install-python-3-ubuntu)

## 想要系统默认的Python3为python3.7

不太建议这么做，这样就会让系统原生态的python变为自己安装的python版本，可能会有一些后果，但是如果一定要这么做：

[方法一：](https://stackoverflow.com/questions/41986507/unable-to-set-default-python-version-to-python3-in-ubuntu)

最后一个1是表示优先级
```
update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 1
```

[方法二：(未尝试，应该可行)](https://askubuntu.com/questions/1120474/setting-python3-to-version-3-7-on-wsl)


## 可能产生的问题和解决办法

### 如果在18.01上升级到Python3.7，使用默认pip3会报错：'lsb_release -a' returned non-zero exit status

据说是ubuntu的bug

```
sudo mv /usr/bin/lsb_release /usr/bin/lsb_release_back
```

参考链接：
https://github.com/pypa/pip/issues/4924
https://askubuntu.com/questions/853377/error-with-lsb-release-a-in-ubuntu-16-04-xenial



# ubuntu18 更新pip的bug

直接在ubuntu18中使用sudo pip install pip --upgrade
命令更新v9.0的pip会出错，更新后使用Pip命令时报错: no module named main

https://stackoverflow.com/questions/49836676/error-after-upgrading-pip-cannot-import-name-main/49836753#49836753
https://askubuntu.com/questions/778052/installing-pip3-for-python3-on-ubuntu-16-04-lts-using-a-proxy
https://github.com/pypa/pip/issues/5221
https://askubuntu.com/questions/981118/correct-way-to-install-python-2-7-on-ubuntu-17-10/981279
https://askubuntu.com/questions/1025189/pip-is-not-working-importerror-no-module-named-pip-internal

这是由于debian系统的bug导致，详细见下：

解决办法一：
1） recover
	sudo python3 -m pip uninstall pip && sudo apt install python3-pip --reinstall.
2） 	curl -k "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
	python3 get-pip.py --user
	or python get-pip.py --user

解决办法二：
python -m pip install --force-reinstall pip

First, some notes about what happened here on Debian/Ubuntu (and likely a few more Linux Distros):

pip does not support the use of it's internals by importing it. More on that in the documentation here.
Debian (hence Ubuntu) does not support modifying their package manager managed files using something, that well, isn't their package manager.
This issue is caused by, well, both being violated in a way.

Debian uses the pip's internal methods (which no longer work due to a reorganization of pip's internals). Debian's assuming here that the pip version in their repositories is the one that would be installed.
Running pip install --upgrade pip, as root, without any other parameters modifies files that are supposed to managed by apt, which breaks the script by Debian.
Some general tips on Linux:

It's a good habit to use --user whenever outside a venv.

pip install --upgrade --user pip
Never run pip with sudo unless you know what you're doing.


What's the workaround?

@standag's solution is useful when this is caused by bash's caching of executables.

hash -r pip # or hash -d pip
If you've modified your OS package manager's installation of pip (eg by using sudo pip) and python -m pip is still working, one workaround is to uninstall the pip installed version and reinstall the package manager installed version.

python -m pip uninstall pip  # this might need sudo
sudo apt install --reinstall python-pip
If you're not on Debian/Ubuntu and pip broke for you, try running:

python -m pip install --force-reinstall pip
If the above this doesn't resolve your problems, please file a new issue.


hash -r pip # or hash -d pip
python -m pip uninstall pip this might need sudo
sudo apt install --reinstall python-pip
python -m pip install --force-reinstall pip


	
	