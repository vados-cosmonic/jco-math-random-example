# `Math.random` in WebAssembly with `jco`

This repository showcases a basic component built with [`jco`][jco] that performs basic CLI functionality, using `Math.random`.

## Setup

> ![NOTE]
> This project uses `pnpm` but you can use `npm` or `yarn` if you prefer

To install dependencies:

```console
nvm use # optional, if you're using nvm
pnpm install
```

[jco]: https://github.com/bytecodealliance/jco

## Build

To build the project, run:

```console
pnpm jco componentize -w wit -o test.wasm test.js
```

This calls `jco`'s `componentize` functionality (powered by [`componentize-js`][componentize-js]), instructing it to use the WebAssembly Inteface Types defined in the `wit/` folder, and using the JS code from `test.js` to output a WebAssembly binary called `test.wasm`.

> [!INFO]
> Don't worry if you don't understand the line above, see [What's actually Happening](#whats-actually-happening) for a longer description! 

## Run

To run the project:

```console
pnpm jco run test.wasm
```

`jco run` looks for the `wasi:cli/run` interface to be exported (see `wit/component.wit`), and executes the exported function.

# What's actually happening?

As a way-too-short flyby of what's happening:

- Originally, [WebAssembly][wasm] was primarily concerned with modules (these deal only in terms of numbers, and are the kind that are available on the browser natively or via emscripten)
- WebAssembly *components* are a new improved standard that build upon WebAssembly modules with rich types
  - The [Component Model][cm] wraps the base WebAssembly standard
  - Rich type support is powered by [WebAssembly Interface Types][wit]
- NodeJS and Browser contexts have the equivalent of standard library (ex. `Math.random`) functions that must be converted
  - In basic WebAssembly land, there *is nothing to convert to* (there's no concept of a "standard library" in base WebAssembly) -- this is partially why the WebAssembly Components were created
  - WebAssembly Components make use of the [WebAssembly System Interface (WASI)][wasi]
- To do all this, we need the WIT that *corresponds* to those system interfaces to be in our project, in a folder (see `wit/`)
  - We could do a regular file copy of this, but I chose here to use `wit-deps` (it uses the `wit/deps.toml` file)
  - Developers must write the `component.wit` (could be named differently), which presents a `world`, and `export`s and/or `import`s functionality
  - In our case, `test.js` `export`s `wasi:cli/run`, at version `0.2.1`
- The stdlib parts of the JS code you write (ex. `Math.random` in `test.js`) is shimmed *into* the component-ready WASI version
  - See: https://github.com/bytecodealliance/jco/blob/6b84470b829b54ecdef6b0f052f75cc045e52bcd/packages/preview2-shim/lib/nodejs/random.js#L34
- `jco` outputs WebAssembly components, thanks to [`componentize-js`][componentize-js], and does the shimming of the native SDKs that you'd get from NodeJS/the JS environment
- `jco` can also *run* components, in particular, when they expose the `wasi:cli/run` interface (You can [`wasi:cli`'s WIT online][wasi-cli])

[cm]: https://component-model.bytecodealliance.org/
[wit]: https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md
[wasi]: https://wasi.dev/
[componentize-js]: https://github.com/bytecodealliance/componentize-js
[wasm]: https://webassembly.org/
[wasi-cli]: https://github.com/WebAssembly/wasi-cli

## Dependencies

To make this project work *from scratch*, you'd need a few useful tools:

- [`wit-deps`][wit-deps] (which is a way to populate the `wit` folder -- you fill out `deps.toml`, it generates `wit/deps` and `wit/deps.lock`)

> [!WARN]
> `wit-deps` isn't what we'll use forever, but it works and is a step up from copying files around.

[wit-deps]: https://github.com/bytecodealliance/wit-deps
