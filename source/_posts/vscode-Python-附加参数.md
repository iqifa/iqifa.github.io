---
title: vscode Python 附加参数
date: 2024-09-14 00:14:55
tags: Python
---
## vscode带参数调试Python

相比于Pycharm中Editor\Configration\Paramater

接触Pythorch的时候遇见下面情形
```Python

parser = argparse.ArgumentParser()
parser.add_argument(
    '--batchSize', type=int, default=32, help='input batch size')
parser.add_argument(
    '--workers', type=int, help='number of data loading workers', default=4)
parser.add_argument(
    '--nepoch', type=int, default=25, help='number of epochs to train for')
parser.add_argument('--outf', type=str, default='seg', help='output folder')
parser.add_argument('--model', type=str, default='', help='model path')
parser.add_argument('--dataset', type=str, required=True, help="dataset path")
parser.add_argument('--class_choice', type=str, default='Chair', help="class_choice")
parser.add_argument('--feature_transform', action='store_true', help="use feature transform")

opt = parser.parse_args()
print(opt)

```
通过(argparse)解析器解析参数
出现以下错误

    the following arguments are required: --dataset

原因在于

 parser.add_argument('--dataset', type=str, <font color=red>required=True </font >, help="dataset path")
    
需要提供--dataset变量，如同vs中的项目变量
### 操作步骤

运行->添加配置->包含参数的当前文件。

在新建的lunch.json文件中  "args": ["--dataset","aaa"]如此