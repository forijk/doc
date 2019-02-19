## Part1 ESLint 介绍

ESLint 是在 ECMAScript/JavaScript 代码中识别和报告模式匹配的工具，它的目标是保证代码的一致性和避免错误。

- ESLint 使用 Espree 解析 JavaScript。
- ESLint 使用 AST 去分析代码中的模式
- ESLint 是完全插件化的。每一个规则都是一个插件并且你可以在运行时添加更多的规则。

如果你想让 ESLint 成为你项目构建系统的一部分，我们建议在本地安装:

```
$ npm install eslint --save-dev

$ ./node_modules/.bin/eslint --init

$ ./node_modules/.bin/eslint yourfile.js
```

运行 eslint --init 之后，.eslintrc 文件会在你的文件夹中自动创建。你可以在 .eslintrc 文件中看到许多像这样的规则：

```
{
    "rules": {
        "semi": ["error", "always"],
        "quotes": ["error", "double"]
    }
}
```

"semi" 和 "quotes" 是 ESLint 中 规则 的名称。第一个值是错误级别，可以使下面的值之一：

- "off" or 0 - 关闭规则
- "warn" or 1 - 将规则视为一个警告（不会影响退出码）
- "error" or 2 - 将规则视为一个错误 (退出码为 1)

.eslintrc 配置文件可以包含下面的一行：

```
"extends": "eslint:recommended"
```

由于这行，所有在 **规则页面** 被标记为绿色对号的规则将会默认开启。

## Part2 ESLint 配置

两种主要的方式来配置 ESLint：

- **Configuration Comments** - 使用 JavaScript 注释把配置信息直接嵌入到一个代码源文件中。
- **Configuration Files** - 使用 JavaScript、JSON 或者 YAML 文件为整个目录（处理你的主目录）和它的子目录指定配置信息。可以配置一个独立的 .eslintrc.\* 文件，或者直接在 package.json 文件里的 eslintConfig 字段指定配置，ESLint 会查找和自动读取它们，再者，你可以在命令行运行时指定一个任意的配置文件。

有很多信息可以配置：

- Environments - 指定脚本的运行环境。每种环境都有一组特定的预定义全局变量。
- Globals - 脚本在执行期间访问的额外的全局变量。
- Rules - 启用的规则及其各自的错误级别。

### parserOptions 解析器选项

请注意，对 JSX 语法的支持不用于对 React 的支持。React 使用了一些特定的 ESLint 无法识别的 JSX 语法。**如果你正在使用 React 并且想要 React 语义支持，我们推荐你使用 eslint-plugin-react。**

同样的，支持 ES6 语法并不意味着同时支持新的 ES6 全局变量或类型（比如 Set 等新类型）。**使用 { "parserOptions": { "ecmaVersion": 6 } } 来启用 ES6 语法支持；要额外支持新的 ES6 全局变量，使用 { "env":{ "es6": true } }(这个设置会同时自动启用 ES6 语法支持)。**

解析器选项可以在 .eslintrc.\* 文件使用 parserOptions 属性设置。可用的选项有：

- ecmaVersion - 默认设置为 3，5（默认）， 你可以使用 6、7、8 或 9 来指定你想要使用的 ECMAScript 版本。你也可以用使用年份命名的版本号指定为 2015（同 6），2016（同 7），或 2017（同 8）或 2018（同 9）
- sourceType - 设置为 "script" (默认) 或 "module"（如果你的代码是 ECMAScript 模块)。
- ecmaFeatures - 这是个对象，表示你想使用的额外的语言特性:
  - globalReturn - 允许在全局作用域下使用 return 语句
  - impliedStrict - 启用全局 strict mode (如果 ecmaVersion 是 5 或更高)
  - jsx - 启用 JSX
  - experimentalObjectRestSpread - 启用实验性的 object rest/spread properties 支持。(重要：这是一个实验性的功能,在未来可能会有明显改变。 建议你写的规则 不要 依赖该功能，除非当它发生改变时你愿意承担维护成本。)

.eslintrc.json 文件示例：

```
{
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module",
        "ecmaFeatures": {
            "jsx": true
        }
    },
    "rules": {
        "semi": 2
    }
}
```

设置解析器选项能帮助 ESLint 确定什么是解析错误，所有语言选项默认都是 false。

### Specifying Environments

一个环境定义了一组预定义的全局变量。可用的环境包括：

- browser - 浏览器环境中的全局变量。
- node - Node.js 全局变量和 Node.js 作用域。
- commonjs - CommonJS 全局变量和 CommonJS 作用域 (用于 Browserify/WebPack 打包的只在浏览器中运行的代码)。
- shared-node-browser - Node.js 和 Browser 通用全局变量。
- es6 - 启用除了 modules 以外的所有 ECMAScript 6 特性
  ...

要在你的 JavaScript 文件中使用注释来指定环境，格式如下：

```
/* eslint-env node, mocha */
```

要在配置文件里指定环境，使用 env 关键字指定你想启用的环境，并设置它们为 true。

```
{
    "env": {
        "browser": true,
        "node": true
    }
}
```

或在 package.json 文件中：

```
{
    "name": "mypackage",
    "version": "0.0.1",
    "eslintConfig": {
        "env": {
            "browser": true,
            "node": true
        }
    }
}
```

### Specifying Globals

设置全局变量

### Configuring Plugins

配置插件

### Configuring Rules

ESLint 附带有大量的规则。你可以使用注释或配置文件修改你项目中要使用的规则。要改变一个规则设置，你必须将规则 ID 设置为下列值之一：

- "off" 或 0 - 关闭规则
- "warn" 或 1 - 开启规则，使用警告级别的错误：warn (不会导致程序退出)
- "error" 或 2 - 开启规则，使用错误级别的错误：error (当被触发的时候，程序会退出)

为了在文件注释里配置规则，使用以下格式的注释：

```
/* eslint eqeqeq: "off", curly: "error" */
```

or

```
/* eslint eqeqeq: 0, curly: 2 */
```

如果一个规则有额外的选项，你可以使用数组字面量指定它们，比如：

```
/* eslint quotes: ["error", "double"], curly: 2 */
```

可以使用 rules 连同错误级别和任何你想使用的选项，在配置文件中进行规则配置。例如：

```
{
    "rules": {
        "eqeqeq": "off",
        "curly": "error",
        "quotes": ["error", "double"]
    }
}
```

### Disabling Rules with Inline Comments

可以在你的文件中使用以下格式的块注释来临时禁止规则出现警告：

```
/* eslint-disable */

alert('foo');

/* eslint-enable */
```

你也可以对指定的规则启用或禁用警告:

```
/* eslint-disable no-alert, no-console */

alert('foo');
console.log('bar');

/* eslint-enable no-alert, no-console */
```

如果在整个文件范围内禁止规则出现警告，将 /_ eslint-disable _/ 块注释放在文件顶部：

```
/* eslint-disable */

alert('foo');
```

你也可以对整个文件启用或禁用警告:

```
/* eslint-disable no-alert */

// Disables no-alert for the rest of the file
alert('foo');
```

可以在你的文件中使用以下格式的行注释或块注释在某一特定的行上禁用所有规则：

```
alert('foo'); // eslint-disable-line

// eslint-disable-next-line
alert('foo');

/* eslint-disable-next-line */
alert('foo');

alert('foo'); /* eslint-disable-line */
```

### Configuration File Formats

ESLint 支持几种格式的配置文件：

- JavaScript - 使用 .eslintrc.js 然后输出一个配置对象。
- YAML - 使用 .eslintrc.yaml 或 .eslintrc.yml 去定义配置的结构。
- JSON - 使用 .eslintrc.json 去定义配置的结构，ESLint 的 JSON 文件允许 JavaScript 风格的注释。
- (弃用) - 使用 .eslintrc，可以使 JSON 也可以是 YAML。
- package.json - 在 package.json 里创建一个 eslintConfig 属性，在那里定义你的配置。

如果同一个目录下有多个配置文件，ESLint 只会使用一个。优先级顺序如下：

1. .eslintrc.js
2. .eslintrc.yaml
3. .eslintrc.yml
4. .eslintrc.json
5. .eslintrc
6. package.json

### Configuration Cascading and Hierarchy

**如果同一目录下 .eslintrc 和 package.json 同时存在，.eslintrc 优先级高会被使用，package.json 文件将不会被使用。**
完整的配置层次结构，从最高优先级最低的优先级，如下:

- 行内配置

  1. /_eslint-disable_/ 和 /_eslint-enable_/
  2. /_global_/
  3. /_eslint_/
  4. /_eslint-env_/

- 命令行选项（或 CLIEngine 等价物）：

  1. --global
  2. --rule
  3. --env
  4. -c、--config

- 项目级配置：
  1. 与要检测的文件在同一目录下的 .eslintrc.\* 或 package.json 文件
  2. 继续在父级目录寻找 .eslintrc 或 package.json 文件，直到根目录（包括根目录）或直到发现一个有"root": true 的配置。
- 如果不是（1）到（3）中的任何一种情况，退回到 ~/.eslintrc 中自定义的默认配置。

### Ignoring Files and Directories

你可以通过在项目根目录创建一个 .eslintignore 文件告诉 ESLint 去忽略特定的文件和目录。.eslintignore 文件是一个纯文本文件，其中的每一行都是一个 glob 模式表明哪些路径应该忽略检测。例如，以下将忽略所有的 JavaScript 文件：

```
**/*.js
```

- 以 # 开头的行被当作注释，不影响忽略模式。
- 路径是相对于 .eslintignore 的位置或当前工作目录。这也会影响通过 --ignore-pattern 传递的路径。
- 忽略模式同 .gitignore 规范
- 以 ! 开头的行是否定模式，它将会重新包含一个之前被忽略的模式。

例如：把下面 .eslintignore 文件放到当前工作目录里，将忽略项目根目录下的 node_modules，bower_components 以及 build/ 目录下除了 build/index.js 的所有文件。

```
# /node_modules/* and /bower_components/* in the project root are ignored by default

# Ignore built files except build/index.js
build/*
!build/index.js
```

如果没有发现 .eslintignore 文件，也没有指定替代文件，ESLint 将在 package.json 文件中查找 eslintIgnore 键，来检查要忽略的文件。

```
{
  "name": "mypackage",
  "version": "0.0.1",
  "eslintConfig": {
      "env": {
          "browser": true,
          "node": true
      }
  },
  "eslintIgnore": ["hello.js", "world.js"]
}
```

### Extending Configuration Files

一个配置文件可以从基础配置中继承已启用的规则。

extends 属性值可以是：

- 在配置中指定的一个字符串
- 字符串数组：每个配置继承它前面的配置

ESLint 递归地进行扩展配置，所以一个基础的配置也可以有一个 extends 属性。

rules 属性可以做下面的任何事情以扩展（或覆盖）规则：

- 启用额外的规则
- 改变继承的规则级别而不改变它的选项：

  1. 基础配置："eqeqeq": ["error", "allow-null"]
  2. 派生的配置："eqeqeq": "warn"
  3. 最后生成的配置："eqeqeq": ["warn", "allow-null"]

- 覆盖基础配置中的规则的选项
  1. 基础配置："quotes": ["error", "single", "avoid-escape"]
  2. 派生的配置："quotes": ["error", "single"]
  3. 最后生成的配置："quotes": ["error", "single"]

值为 **"eslint:recommended"** 的 extends 属性启用一系列核心规则，这些规则报告一些常见问题，在 [规则页面](https://cn.eslint.org/docs/rules/) 中被标记为绿色对号 ✅。这个推荐的子集只能在 ESLint 主要版本进行更新。

**Using the configuration from a plugin**
插件 是一个 npm 包，通常输出规则。一些插件也可以输出一个或多个命名的 配置。要确保这个包安装在 ESLint 能请求到的目录下。

plugins 属性值 可以省略包名的前缀 eslint-plugin-。

extends 属性值可以由以下组成：

- plugin:
- 包名 (省略了前缀，比如，react)
- /
- 配置名称 (比如 recommended)

JSON 格式的一个配置文件的例子：

```
{
    "plugins": [
        "react"
    ],
    "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
    ],
    "rules": {
       "no-set-state": "off"
    }
}
```

## 命令行

### --cache

存储处理过的文件的信息以便只对有改变的文件进行操作。缓存默认被存储在 .eslintcache。启用这个选项可以显著改善 ESLint 的运行时间，确保只对有改变的文件进行检测。

**注意：** 如果你运行 ESLint --cache，然后又运行 ESLint 不带 --cache，.eslintcache 文件将被删除。这是必要的，因为检测的结果可能会改变，使 .eslintcache 无效。如果你想控制缓存文件何时被删除，那么使用 --cache-location 来指定一个缓存文件的位置。

## Miscellaneous

### --init

这个选项将会配置初始化向导。它被用来帮助新用户快速地创建 .eslintrc 文件，用户通过回答一些问题，选择一个流行的风格指南，或检查你的源文件，自动生成一个合适的配置。

生成的配置文件将被创建在当前目录。

### --fix

该选项指示 ESLint 试图修复尽可能多的问题。修复只针对实际文件本身，而且剩下的未修复的问题才会输出。

### --debug

这个选项将调试信息输出到控制台。当你看到一个问题并且很难定位它时，这些调试信息会很有用。ESLint 团队可能会通过询问这些调试信息帮助你解决 bug。

### --no-inline-config

这个选项会阻止像 /_eslint-disable_/ 或者 /_global foo_/ 这样的内联注释起作用。这允许你在不修改文件的情况下设置一个 ESLint 配置。所有的内联注释都会被忽略，比如：

- /\*eslint-disable\*/
- /\*eslint-enable\*/
- /\*global\*/
- /\*eslint\*/
- /\*eslint-env\*/
- // eslint-disable-line
- // eslint-disable-next-line

### --print-config

这个选项输出传递的文件使用的配置。当有这个标记时，不进行检测，只有配置相关的选项才是有效的。

示例：

```
eslint --print-config file.js
```
