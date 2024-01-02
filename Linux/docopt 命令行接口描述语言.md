# docopt 命令行接口描述语言

## 介绍

docopt 用于描述命令行接口的帮助信息，并且只要满足 docopt 结构化的约定描述，就是可以使用 docopt 去解析命令行，docotpt 的解析实现支持多种语言，可以 [docopt GitHub](https://github.com/docopt)仓库中查找。


```bash

Naval Fate.
Usage:
  naval_fate ship new <name>...
  naval_fate ship <name> move <x> <y> [--speed=<kn>]
  naval_fate ship shoot <x> <y>
  naval_fate mine (set|remove) <x> <y> [--moored|--drifting]
  naval_fate -h | --help
  naval_fate --version
Options:
  -h --help     Show this screen.
  --version     Show version.
  --speed=<kn>  Speed in knots [default: 10].
  --moored      Moored (anchored) mine.
  --drifting    Drifting mine.

```









## 参考文档

1. http://docopt.org/