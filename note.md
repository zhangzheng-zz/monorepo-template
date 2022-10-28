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

配置全局 tsconfig，谁在包路径 paths:

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
