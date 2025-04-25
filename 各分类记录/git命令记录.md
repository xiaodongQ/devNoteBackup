## Git命令记录

### fork开源仓库时如何只同步部分分支或tag

1、fork仓库时只fork master
2、原始仓库添加为一个新的远程仓库：`git remote add upstream 原始仓库`
    比如`git remote add upstream https://github.com/ceph/ceph`
3、拉取上游分支 或 标签
    所有分支和tag：`git fetch upstream`
    指定分支：`git fetch upstream feature-branch`
    指定标签：`git fetch upstream tag v1.0.0`
4、切换本地分支，并关联到上游分支
    分支：`git checkout -b feature-branch upstream/feature-branch`
        推送：`git push origin feature-branch`
        （也可以 `git checkout` 切换分支后，手动设置上游分支 `git push --set-upstream origin rocksdb-v6.15.5`
    标签：`git checkout 标签`
        推送：`git push origin v1.0.0`
        如果只是有tag，可以自己基于这个tag建个分支，然后推送到自己的仓库：`git switch -c rocksdb-v6.15.5`而后推送

### git clone --depth=1 浅克隆

git clone --depth 1 https://github.com/username/repository.git

--depth 1 只克隆最近的一次提交（即 HEAD 提交）
注意：浅克隆无法访问完整的提交历史记录，也无法切换到其他分支

* 默认情况下，git clone 会下载整个仓库的所有分支和提交历史记录，但只会检出默认分支。
* 如果只想克隆某个分支，使用 `--branch` 和 `--single-branch` 参数。
* 如果想节省空间，可以使用 `--depth` 参数进行浅克隆

