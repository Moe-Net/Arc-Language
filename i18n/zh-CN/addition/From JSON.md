Starting with JSON
==================

ARC 可以渐进的从 JSON 过渡!

我们以如下 `json` 为例, 看看如何用 `arc` 改写增强可读性.

```json
{
  "name": "vscode-arc",
  "displayName": "Arc Language Support",
  "description": "Highlight and formatter for Arc Readable Configiration",
  "repository": {
    "type": "git",
    "url": "https://github.com/GalAster/vscode-arc.git"
  },
  "engines": {
    "vscode": "^1.8.0"
  },
  "categories": [
    "Programming Languages",
    "Formatters"
  ],
  "scripts": {
    "postinstall": "node ./node_modules/vscode/bin/install && tsc",
    "build": "yarn lint && ts-node syntax/build.ts",
    "pack": "yarn build && vsce package",
    "lint": "tslint **/*.ts --fix"
  },
  "contributes": {
    "languages": [
      {
        "id": "arc",
        "aliases": [
          "ARC"
        ],
        "extensions": [
          ".arc"
        ],
        "filenames": [],
        "mimetypes": [
          "text/x-arc"
        ],
        "configuration": "./syntax/arc.configuration.json"
      }
    ],
    "grammars": [
      {
        "language": "arc",
        "scopeName": "source.arc",
        "path": "./syntax/arc.tmLanguage.json"
      },
      {
        "scopeName": "markdown.arc.codeblock",
        "path": "./syntax/arc.markdown.json",
        "injectTo": [
          "text.html.markdown"
        ],
        "embeddedLanguages": {
          "meta.embedded.block.arc": "arc"
        }
      }
    ]
  },
  "devDependencies": {
    "@types/node": "^11.13.6",
    "vscode": "^1.1.33"
  },
  "__metadata": {
    "id": "6267dad2-7d52-462a-a1ef-7e3da7378a7d",
    "publisherDisplayName": "Aster",
    "publisherId": "3406b78c-f287-4619-8d82-7c97998693e3"
  },
}
```

首先, 去掉开头末尾的 `{ }`, 因为 `arc` 必须是键值对的形式.

[#RFC9]() 中提出了可以使用 `--default` 参数给以默认绑定, 这样就不用去掉 `{ }` 了.

现在得到的文件已经是一个合法的 `arc` 文件了, 但是不怎么好看.

---

接着你可以遵循 `arc` 的最佳实践, 获取更好的阅读效果

你可以把所有键的引号去掉, 除非他们包含 `/` 字符

右边单引号是最佳实践, 但是双引号也可以, 两者区别参考

然后你可以把 json 风格的 ` =` 换成 `=`

然后去掉键值对末尾的所有 `,`, 除非是多行列表或者字典, 每行末尾使用 `,` 分隔

IDE 能自动为你格式化, 你可以使用 `self-lint` 控制格式化

```arc
name = 'vscode-arc'
displayName = 'Arc Language Support'
description = 'Highlight and formatter for Arc Readable Configiration'
repository = {
  type = 'git',
  url = 'https://github.com/GalAster/vscode-arc.git',
}
engines = {
  vscode = '^1.8.0',
}
categories = [
  'Programming Languages',
  'Formatters',
]
scripts = {
  postinstall = 'node ./node_modules/vscode/bin/install && tsc',
  build = 'yarn lint && ts-node syntax/build.ts',
  pack = 'yarn build && vsce package',
  lint = 'tslint **/*.ts --fix',
}
contributes = {
  languages = [
    {
      id = 'arc',
      aliases = ['ARC'],
      extensions = ['.arc'],
      filenames = [ ],
      mimetypes = ['text/x-arc'],
      configuration = './syntax/arc.configuration.json',
    },
  ],
  grammars = [
    {
      language = 'arc',
      scopeName = 'source.arc',
      path = './syntax/arc.tmLanguage.json',
    },
    {
      scopeName = 'markdown.arc.codeblock',
      path = './syntax/arc.markdown.json',
      injectTo = [
        'text.html.markdown',
      ],
      embeddedLanguages = {
        'meta.embedded.block.arc' = 'arc',
      },
    },
  ],
}
devDependencies = {
  '@types/node' = '^11.13.6',
  vscode = '^1.1.33'
}
__metadata = {
  id = '6267dad2-7d52-462a-a1ef-7e3da7378a7d',
  publisherDisplayName = 'Aster',
  publisherId = '3406b78c-f287-4619-8d82-7c97998693e3',
}
```

---

接着我们要引入路由的概念, 这能大大简化输入

路由用 `/` 分割, 就像文件路径一样.

这也是为什么 key 出现 `/` 必须用字符串模式的原因.

至于为什么纯数字也要字符串, 具体原因下一节会揭示.

```arc
engines = {
  vscode = '^1.8.0',
}
```

等价于 

```arc
engines/vscode = '^1.8.0'
```

编程语言中 namespace 通常用 `::` 或者 `.` 分隔. 

`arc` 不用 `.` 的原因在于这符号个已经用于分割小数了, `arc` 为了方便解析一个符号只做一件事.

---

每个路由都要展开太过烦琐了, 接下来引入域的概念

```arc
repository = {
  type = 'git',
  url = 'https://github.com/GalAster/vscode-arc.git',
}
categories = [
  'Programming Languages',
  'Formatters',
]
```

域表示到域切换或者文件结束为止全部挂载在这个键上

字典域用 `( )` 表示, 列表域用 `[ ]` 表示.

等价的写法为如下:

```arc
(repository)
type = 'git',
url = 'https://github.com/GalAster/vscode-arc.git',
<categories>
& 'Programming Languages'
& 'Formatters'
```

列表域使用 `&` 表示插入一个值, `*`表示插入多个键构成字典.

```arc
<dependence>
* name = 'number'
  md5 = '7FF2B2E95569F56D'
* name = 'string'
  md5 = '84DD3D20D928BEE8'
& { name = 'bool', md5 = '4BABDD6E68AB3E63' }
& null
```

等价于

```ts
module.exports = [
  {
    name: 'number',
    md5: '7FF2B2E95569F56D'
  },
  {
    name: 'string',
    md5: '84DD3D20D928BEE8'
  },
  {
    name: 'bool',
    md5: '4BABDD6E68AB3E63'
  },
  null
]
```

很少有场景需要混写 `*` 和 `&`.

---

我们接着考虑如何改写那个很大的 `contributes` 字段

先介绍域继承, 考虑如下构造

```arc
root = {
  a = {c = true}
  b = {d = false}
}
```

你可以写成 

```arc
root = {
  (a) c = true
  (b) d = false
}
```

也可以写成

```arc
(root)
  (/a) c = true
  (/b) d = false
```

这取决于你喜不喜欢大括号, 这里的缩进都不是必须的.

`arc` 中空格, 缩进, 换行没有任何的实际意义如何排版是你的自由.

---

接着我们了解一下什么是索引路由.

回到这个结构

```arc
contributes = {
  languages = [
    {
      id = 'arc',
      aliases = ['ARC'],
      extensions = ['.arc'],
      filenames = [ ],
      mimetypes = ['text/x-arc'],
      configuration = './syntax/arc.configuration.json',
    },
  ],
}
```

看起来似乎可以写成 

```arc
<contributes/languages>
* id = 'arc',
  aliases = ['ARC'],
  extensions = ['.arc'],
  filenames = [ ],
  mimetypes = ['text/x-arc'],
  configuration = './syntax/arc.configuration.json',
& {  %或者手动展开
    id = 'arc',
    aliases = ['ARC'],
    extensions = ['.arc'],
    filenames = [ ],
    mimetypes = ['text/x-arc'],
    configuration = './syntax/arc.configuration.json',
 }
```

但还有更好的写法, 这里这个字典展开标记 `*` 可以省略

因为是列表中第一个元素, 所以用 1 表示即可, 此处是字典, 所以用 `( )`.

```arc
(contributes/languages/1)
id = 'arc'
aliases = ['ARC']
extensions = ['.arc']
filenames = [ ]
mimetypes = ['text/x-arc']
configuration = './syntax/arc.configuration.json'
```

---

最终改写的文件如下:

```arc
% https://github.com/GalAster/vscode-arc/blob/master/package.json
name = 'vscode-arc'
displayName = 'Arc Language Support'
description = 'Highlight and formatter for Arc Readable Configiration'
engines/vscode = '^1.8.0'

<categories>
& 'Programming Languages'
& 'Formatters'

(repository)
type = 'git'
url = 'https =//github.com/GalAster/vscode-arc.git'

(scripts)
postinstall = 'node ./node_modules/vscode/bin/install && tsc'
build = 'yarn lint && ts-node syntax/build.ts'
pack = 'yarn build && vsce package'
lint = 'tslint **/*.ts --fix'

(dependencies)
vscode = '^1.1.33'

(devDependencies)
'@types/node' = '^11.13.6'

(contributes/languages/1)
id = 'arc'
aliases = ['ARC'],
extensions = ['.arc']
filenames = [ ]
mimetypes = ['text/x-arc']
configuration = './syntax/arc.configuration.json'

<contributes/grammars>
* language = 'arc'
  scopeName = 'source.arc'
  path = './syntax/arc.tmLanguage.json'
* scopeName = 'markdown.arc.codeblock',
  path = './syntax/arc.markdown.json',
  injectTo = [
    'text.html.markdown',
  ],
  embeddedLanguages = {
    'meta.embedded.block.arc' = 'arc',
  }

(__metadata)
id = '6267dad2-7d52-462a-a1ef-7e3da7378a7d'
publisherDisplayName = 'Aster'
publisherId = '3406b78c-f287-4619-8d82-7c97998693e3'
```

注意所有的非字符空格换行都可以去掉, 进一步压缩传输.

`arc` 语言的主要要点就全部在这了

还有一个引用特性, 就是使用 `$` 标记路径, 共享版本号之类的值.

接下去就是熟悉宏的使用, 如何结合语言的反射特性, 得到可读的序列化结果