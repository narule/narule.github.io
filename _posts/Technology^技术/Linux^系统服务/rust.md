# rust 

https://www.rust-lang.org/tools/install

## install



#### linux 安装

```shell
curl https://sh.rustup.rs -sSf | sh
```



运行完后，

```shell
Rust is installed now. Great!

To get started you need Cargo's bin directory ($HOME/.cargo/bin) in your PATH
environment variable. Next time you log in this will be done
automatically.

To configure your current shell run source $HOME/.cargo/env

```



#### 配置环境变量

需要 运行  设置环境变量的脚本

运行下面两句就好了

```shell
source ~/.profile 
source ~/.cargo/env

#这里在安装的时候，应该是在~/.profile 文件追加了一句 export PATH="$HOME/.cargo/bin:$PATH"
#这里运行是将 rustc 添加到环境变量
```

