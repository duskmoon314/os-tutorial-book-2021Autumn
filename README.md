# **[WIP]** os-Tutorial-Book

【施工中】清华大学计算机系操作系统课程实验指导书 2021 秋季合并版

> 稳定版请参看 rCore 和 uCore 各自的指导书

## 本地开发

建议安装依赖方式：

```bash
python -m venv .env
source .env/bin/activate
pip install -r requirements.txt
```

本地编译

```bash
make html
```

文档编译到 build/html 中，可以使用浏览器查看，或者使用 nginx、serve（node.js 可安装工具）、simple-http-server（cargo 可安装工具）等工具 host 在某端口。
