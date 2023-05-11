# Curve

* [opencurve](https://github.com/opencurve/curve)
* 一款高性能、易运维、云原生的开源分布式存储系统
	- 块存储
* [curve文档](https://github.com/opencurve/curve/blob/master/README_cn.md)
	- 具体学习记录参考下面`## 文档`章节

* CurveAdm
	- 用于快速部署和运维 CurveBS/CurveFS 集群
	- [CurveAdm](https://github.com/opencurve/curveadm/wiki/overview)

## 编译环境(开发调试代码才需要)

* 链接：[build_and_run](https://github.com/opencurve/curve/blob/master/docs/cn/build_and_run.md)
* 容器方式：
	- docker pull opencurvedocker/curve-base:build-debian9
	- docker run -it opencurvedocker/curve-base:build-debian9 /bin/bash
	- git clone https://github.com/opencurve/curve.git
		+ cd curve
		+ bash replace-curve-repo.sh (可选，将外部依赖替换为国内下载点或镜像仓库，可以加快编译速度)
		+ 编译 curvebs: cd curve && make build (v2.0之后) 
		+ 编译 curvefs: cd curve/curvefs && make build dep=1 (v2.0之后) 

## 文档


