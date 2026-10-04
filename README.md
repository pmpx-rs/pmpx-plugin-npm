# pmpx-plugin-npm

> 中文版见 [README_CN.md](README_CN.md)

The **npm** backend for [pmpx](https://crates.io/crates/pmpx). It maps pmpx's verbs onto npm
commands, and does nothing else: no file reads, no environment, no network.

```console
$ pmpx install left-pad    # in a Node project → npm install left-pad
$ pmpx run dev --port 3000 #                  → npm run dev -- --port 3000
$ pmpx test --watch        #                  → npm test -- --watch
```

## The mapping

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

`build` has no npm subcommand behind it, so it expands to the project's own `build` script:
`pmpx build --watch` is `npm run build -- --watch`.

## npm is the one backend that inserts a `--` itself

The `--` in the three rows above is not decoration. npm treats everything before it as its own
options, so an argument meant for the script is otherwise eaten:

```console
$ npm run probe --foo
npm warn Unknown cli config "--foo". This will stop working in the next major version of npm.
                                   # npm swallowed --foo as its own configuration; the script
                                   # never received it

$ npm run probe -- --foo
GOT:--foo                          # with the `--` it arrives
```

**pnpm, yarn and bun are the opposite**: they pass a `--` through to the script verbatim, so their
plugins must not insert one. This backend does, in `run`, `build` and `test` alike.

`run` receives its script name as the first argument and puts the `--` directly after it, because
that is where npm needs the boundary -- not at the end of the line. When there is nothing to
forward the `--` is left out: `pmpx run dev` is exactly `npm run dev`, which npm accepts, whereas
a bare trailing `--` makes npm run `node probe.js --` and hand the script a `--` argument it never
asked for.

## Install

```console
$ pmpx plugin add npm
```

Every release also publishes prebuilt assets for the common targets -- Linux x64, Windows
x64 and both macOS architectures. `crate-plugin-kit` downloads them from the release of the
same tag, so an install usually takes a second instead of a build; a target without assets
falls back to compiling from source, which is slower, not broken.

## Detection

From `pmpx-plugin.toml`, which travels with this crate:

| File | Weight | What it proves |
| ---- | ------ | -------------- |
| `package-lock.json` | strong (100) | the project was actually resolved by npm |
| `npm-shrinkwrap.json` | strong (100) | the same, for a published lockfile |
| `package.json` | weak (10) | the ecosystem, not the tool |

The gap between the two tiers is the point. `package.json` is shared with pnpm, yarn and bun, so
on its own it scores 10 and can lose to another backend in the same tree -- that is what the pins
in `.pmpx.toml` are for.

## Requirements

Rust **1.82+**, which is the contract crate's floor. `npm` itself has no floor here: the mappings
above are plain subcommands, and `npx` ships with npm.

## Repository

<https://github.com/pmpx-rs/pmpx-plugin-npm>

## License

MIT — see [LICENSE](LICENSE).
