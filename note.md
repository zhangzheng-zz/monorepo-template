turborepo + pnpm = monorepo

1、初始化

```ts
npx create-turbo@latest
```

选择 pnpm

- turbo.json

pipeline：
https://turborepo.org/docs/reference/configuration#pipeline
turbo run \*\*\*命令的时候 turbo 会根据我们定义的 Pipelines 里对命令的各种配置去对我们的每个 package 中的 package.json 中 对应的 script 执行脚本进行有序的执行和缓存输出的文件。

```json
{
  "pipeline": {
    "build": {
      // A package's `build` script depends on that package's
      // dependencies' and devDependencies'
      // `build` tasks  being completed first
      // (the `^` symbol signifies `upstream`).
      "dependsOn": ["^build"],
      // "Cache all files emitted to workspace's dist/** or build/** directories by a `build` task"
      "outputs": ["dist/**", "build/**"],
      // Only show output from cache misses
      "outputMode": "new-only"
    },
    "lint": {
      "outputs": []
    },
    "dev": {
      "cache": false,
      "dependsOn": ["^build"]
    }
  }
}
```

pnpm monorepo 管理是 通过 workspaces 指定我们的包目录，推荐 apps 放应用程序工程，packages 放公共配置，工具包等

配置工作空间 pnpm workplace：

```json
// pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"

```

```json
package.json:
    "workspaces": [
        "apps/*",
        "packages/*"
    ],
```

先在 apps 初始化一个 项目 vue3-project

```js
npm init vue@3
```

然后 在 packages 初始化两个工具个包 utils hook 一个配置 tsconfig,
包名添加同个域：`@zpz`

指定好包入口、文件，导出等等内容，初始化包的内容。

两个工具包使用 tsc 编译，定义脚本内容：

```json
"scripts": {
    "build": "tsup",
    "build:fix": "esno scripts/postbuild.ts"
  },
```

2、共享模块
https://juejin.cn/post/7139502662080266247
全局依赖安装：

pnpm 通用模块的全局安装：将会安装在 根目录的 node_modules

```
pnpm add eslint -D -w

pnpm i fast-glob -w
```

添加共享 ts 配置

```
--filter
 指定包/应用程序、目录和 git 提交的组合作为执行的入口点
```

```
pnpm add @zpz/tsconfig --filter=@zpz/hooks

pnpm add @zpz/tsconfig --filter=@zpz/utils
```

包的依赖，publish 的时候回映射成实际版本：

```json
"dependencies": {
    "@zpz/tsconfig": "workspace:^1.0.0"
  }
```

模块的配置和开发
使用 tsup （esbuild）打包 ts 模块：

```
pnpm add tsup -Dw
```

试一下打包：

```
npm run build
```

给应用程序安装内部依赖

```
pnpm add @zpz/utils zpz/hooks --filter vue3-project
```

TODO: 解决模块内部跳转源码，配置全局 tsconfig，谁在包路径 paths:

```ts
{
  "compilerOptions": {
    // 内部模块路径
    "baseUrl": "./packages",
    "paths": {
      "@zpz/*": ["*/src"]
    }
  },
  "include": ["packages/*", "apps/*"],
  "exclude": ["node_modules", "lib"]
}

```

执行开发环境脚本

```
pnpm run dev
```

防止 yarn npm 安装：

```
"preinstall": "npx only-allow pnpm"
```

发布工作流

pnpm publish

```
// before
"dependencies": {
  "@monorepo/package-a": "workspace:^1.0.0"
}
// after
"dependencies": {
  "@monorepo/package-a": "^1.0.0"
}
```

changeset:
https://pnpm.io/zh/using-changesets

```
pnpm add -Dw @changesets/cli

pnpm changeset init
```

配置文件：

changelog: changelog 生成方式
commit: 不要让 changeset 在 publish 的时候帮我们做 git add
linked: 配置哪些包要共享版本
access: 公私有安全设定，内网建议 restricted ，开源使用 public
baseBranch: 项目主分支
updateInternalDependencies: 确保某包依赖的包发生 upgrade，该包也要发生 version upgrade 的衡量单位（量级）
ignore: 不需要变动 version 的包
\_\_\_experimentalUnsafeOptions_WILL_CHANGE_IN_PATCH: 在每次 version 变动时一定无理由 patch 抬升依赖他的那些包的版本，防止陷入 major 优先的未更新问题

代码提交规范：

- 使用 cz-customizable 自定义提交信息
- cz-conventional-changelog commitizen

https://commitlint.js.org/#/reference-cli

```
  "config": {
    "commitizen": {
      "path": "node_modules/cz-customizable"
    }
  }
```

脚本参考：https://juejin.cn/post/7139502662080266247

lint-stage 对暂存区的代码执行校验 防止垃圾代码溜进仓库 配合 husky 使用

prettier eslint 格式化插件 eslint-plugin-prettier

eslint-config-prettier - 关闭所有与 eslint 冲突的规则，请注意，该插件只有关闭冲突的规则的作用

```
pnpm i -Dw eslint lint-staged eslint-plugin-prettier @typescript-eslint/eslint-plugin typescript @typescript-eslint/parser eslint-config-prettier
```

格式化和 commit 自动化 https://juejin.cn/post/7136009620979449893#heading-1

husky: 添加 git-hooks 提交拦截

- pre-commit: lint-stage 做暂存区文件的校验
  - prettier 代码格式化 为了防止和 eslint 冲突我们一般安装 eslint-config-prettier
  - eslint 语法检查和 typescript 插件校验（不推荐 tslint？）
  - stylelint 做 css 样式格式化
- commit：
  - commitizen 工具搭配 cz-conventional-changelog 适配器模块帮助生成符合 angular 规范的 commit 信息
    （commitizen 初始化暂时不支持 pnpm，因为它通过 npm 和 yarn 安装 cz-conventional-changelog，可以手动操作，添加 path key 到 package config.commitizen ）
  - commitlint 为 commit message 做校验，防止使用 git commit 提交不符合规范的信息

重点参考：https://juejin.cn/post/7098609682519949325#heading-12

https://juejin.cn/post/7053807488952434719#heading-1

https://maimai.cn/article/detail?fid=1748110352&efid=s8mFm7vRRBNphi0ccP3CBg

主要包管理工具 pnpm（workplace） + turbo (https://turbo.build/) + changeset(发布和版本控制相关)

turbo demo: https://github.com/vercel/turbo/tree/main/examples
