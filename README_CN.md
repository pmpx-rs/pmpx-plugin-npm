# pmpx-plugin-npm

> English see [README.md](README.md)

[pmpx](https://crates.io/crates/pmpx) 的 **npm** 后端。它只做一件事：把 pmpx 的动词映射成
npm 命令 —— 不读文件、不看环境变量、不联网。

```console
$ pmpx install left-pad    # Node 项目里 → npm install left-pad
$ pmpx run dev --port 3000 #             → npm run dev -- --port 3000
$ pmpx test --watch        #             → npm test -- --watch
```

## 映射表

| pmpx | npm |
| ---- | --- |
| `install` | `npm install` |
| `install <pkg>...` | `npm install <pkg>...` |
| `remove <pkg>...` | `npm uninstall <pkg>...` |
| `run <script> [arg]...` | `npm run <script> -- [arg]...` |
| `build [arg]...` | `npm run build -- [arg]...` |
| `test [arg]...` | `npm test -- [arg]...` |
| `update` | `npm update` |
| `update <pkg>...` | `npm update <pkg>...` |
| `exec` | `npx <args>...` |

npm 里没有 `build` 这个子命令，所以这个动词展开成项目自己的 `build` 脚本：
`pmpx build --watch` 就是 `npm run build -- --watch`。

## npm 是唯一一个要自己插 `--` 的后端

上面三行里的 `--` 不是装饰。npm 把它之前的参数都当成自己的选项，本该给脚本的参数会被吃掉：

```console
$ npm run probe --foo
npm warn Unknown cli config "--foo". This will stop working in the next major version of npm.
                                   # npm 把 --foo 当成自己的配置吃掉了，脚本根本收不到

$ npm run probe -- --foo
GOT:--foo                          # 加了 -- 才对
```

**pnpm / yarn / bun 正好相反**：它们会把 `--` 原样传给脚本，所以那几个后端**不能**插。
这个后端在 `run`、`build`、`test` 三个动词上都要插。

`run` 的第一个参数就是脚本名，`--` 插在它**后面** —— npm 需要的是这个位置的边界，而不是行尾的
一个符号。没有参数要转发时就不插：`pmpx run dev` 就是 `npm run dev`，npm 接受这个写法；而留一个
光秃秃的尾部 `--` 会让 npm 执行 `node probe.js --`，脚本平白多收到一个 `--` 参数。

## 安装

```console
$ pmpx plugin add npm
```

## 检测

依据随这个 crate 一起发布的 `pmpx-plugin.toml`：

| 文件 | 权重 | 能证明什么 |
| ---- | ---- | ---------- |
| `package-lock.json` | 强（100） | 这个项目确实被 npm 解析过 |
| `npm-shrinkwrap.json` | 强（100） | 同上，用于随包发布的锁文件 |
| `package.json` | 弱（10） | 只证明属于这个生态，不证明用了哪个工具 |

**两档之间的差距才是重点。** `package.json` 是 pnpm、yarn、bun 共用的，单靠它只有 10 分，
在同一棵树里可能输给别的后端 —— 这正是 `.pmpx.toml` 里那些固化项存在的理由。

## 环境要求

Rust **1.82+**，这是契约 crate 的地板。npm 本身在这里没有下限：上面这些映射都是普通子命令，
而 `npx` 随 npm 一起发布。

## 仓库

<https://github.com/pmpx-rs/pmpx-plugin-npm>

## 许可

MIT —— 见 [LICENSE](LICENSE)。
