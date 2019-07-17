在使用`go get`安装gopls时发现连接超时，错误如下：
#### 命令
```bash
go get golang.org/x/tools/gopls
```
#### 输出
```bash
package golang.org/x/sync/errgroup: unrecognized import path "golang.org/x/sync/errgroup" (https fetch: Get https://golang.org/x/sync/errgroup?go-get=1: dial tcp 216.239.37.1:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.)
```
#### 解决方案
由于`golang.org/x`在`github/golang`下有镜像文件，故手动在`$GOPATH/src`目录创建`golang.org/x`子目录，并进入目录中，`clone`github/golang下的仓库。
```bash
git clone https://github.com/golang/gopls
```
