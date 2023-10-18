## 问题描述

**场景：**目前有一个使用qiankun框架搭建的微前端平台，主应用使用vue3开发。

**需求：**一个能够不受其他子应用切换影响的智能助手应用，嵌入到平台中。

**难点：**qiankun框架并不支持同时激活多个子应用，也不支持子应用保活，这是智能助手的必需条件。

**技术选型：**使用vue3的本地依赖组件，将智能助手应用作为一个本地依赖组件嵌入到主应用中。

## 解决方案

### 1. 创建一个vue3项目

```bash
npm create vue@latest
```

### 2. 改造项目为本地依赖组件

```ts
// vite.config.ts
import { fileURLToPath, URL } from 'node:url'
import { name, version } from './package.json'
import { copyFileSync } from 'fs'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import dts from 'vite-plugin-dts';
export default defineConfig({
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    }
  },
  build: {
    outDir: 'dist',
    lib: {
      entry: fileURLToPath(new URL('./src/index.ts', import.meta.url)),
      name: 'Copilot',
      fileName: 'copilot',
    },
    rollupOptions: {
      // 确保外部化处理那些你不想打包进库的依赖
      external: ['vue'],
      output: {
        // 在 UMD 构建模式下为这些外部化的依赖提供一个全局变量
        globals: {
          vue: 'Vue'
        }
      }
    }
  },
  plugins: [
    vue(),
    dts({
      afterBuild: (): void => {
        move()
      }
    }),
  ],
});
/** 打包结束之后将一些静态文件进行移入 */
const move = (): void => {
  const files = [
    { input: './README.md', outDir: 'dist/README.md' },
    { input: './package.json', outDir: 'dist/package.json' }
  ] as const

  files.forEach((item): void => {
    copyFileSync(item.input, item.outDir)
  })

  console.warn('\n' + `${name} ${version} 版本打包成功 🎉🎉🎉` + '\n')
}
```

```json
// package.json
{
  ...
  "type": "module",
  "main": "./copilot.umd.cjs",
  "module": "./copilot.js",
  "exports": {
    ".": {
      "import": "./copilot.js",
      "require": "./copilot.umd.cjs"
    },
    "./style.css": "./style.css"
  }
  ...
}
```

```ts
// src/main.ts => src/index.ts
import './assets/main.css'

import type { App, Plugin } from 'vue'
import LavaFeCopilot from './LavaFeCopilot.vue'

const CopilotPlugin: Plugin = {
  install(app: App) {
    console.log('installing copilot plugin')
    console.log('copilot plugin app', app)
    app.component('lava-fe-copilot', LavaFeCopilot)
    console.log('copilot plugin installed')
    return app
  }
}

export default CopilotPlugin
```

### 3. 改造主应用以嵌入本地依赖组件

```ts
// vue.config.js
...
configureWebpack: {
    resolve: {
      symlinks: false,
    }
}
...
```

```json
{
  "dependencies": {
    "lava-fe-copilot": "file:../lava-fe-copilot/dist",
  }
}
```

```ts
// main.ts
import CopilotPlugin from 'lava-fe-copilot'
...
instance = createApp(App)
instance.use(CopilotPlugin)
...
```

### 4. 本地依赖组件的使用

```vue
<template>
  <lava-fe-copilot />
</template>
```

## 总结

步骤中需要尤为注意的是主应用中`configureWebpack.resolve.symlinks`的配置，这个配置的作用是告诉webpack不要解析本地依赖组件的依赖，否则会导致本地依赖组件的依赖被打包进主应用中，从而导致主应用中出现多个vue实例，从而导致主应用中的vue实例和本地依赖组件中的vue实例不是同一个，从而导致本地依赖组件中的vue实例无法正常工作。

## 参考

在 webpack 中，symlinks 是一个配置项，用于控制是否解析符号链接（Symbolic Links）。符号链接是一种在文件系统中创建的指向另一个文件或目录的引用。symlinks 配置项影响 webpack 如何处理这些符号链接。

在 webpack 配置中，resolve.symlinks 配置项有三种可能的值：

1. true：Webpack 将解析符号链接，并尝试加载链接目标的实际文件。
2. false：Webpack 将忽略符号链接，不会解析它们，直接使用链接本身作为文件路径。
3. 'maintain'：Webpack 仅在 node_modules 目录中解析符号链接，而不解析其他位置的符号链接。

这个配置项的作用是为了解决符号链接可能引发的一些问题。在某些情况下，开发者可能使用符号链接来组织项目文件，但 webpack 在处理这些链接时可能会遇到一些不一致的行为，比如循环引用等问题。通过设置 symlinks 配置项，你可以控制 webpack 是否要解析符号链接以及如何解析。

在使用 webpack 的过程中，默认情况下 symlinks 是开启的，即 resolve.symlinks 默认值是 true。这通常是符合大多数项目的预期行为的，但在一些特殊情况下，你可能需要手动调整这个配置项。
