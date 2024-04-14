
注意：当前环境需要使用 `nix` 包管理器，并且开启实验性 `flake`

先看看我们的组件
![image.png](https://s2.loli.net/2024/04/14/cVKYSjUIBtnhCgf.png)

我的 `flake.nix`
```Nix
{
  description = "A devShell example";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    rust-overlay.url = "github:oxalica/rust-overlay";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = {
    self,
    nixpkgs,
    rust-overlay,
    flake-utils,
    ...
  }:
    flake-utils.lib.eachDefaultSystem (
      system: let
        overlays = [(import rust-overlay)];
        pkgs = import nixpkgs {
          inherit system overlays;
        };
        rustpkg = pkgs.rust-bin.selectLatestNightlyWith (toolchain:
          toolchain.default.override {
            extensions = ["rust-src" "rustfmt" "clippy"];
            targets = ["thumbv7m-none-eabi"];
          });
      in
        with pkgs; {
          devShells.default = mkShell {
            buildInputs = [
              openssl
              pkg-config
              eza
              fd
              probe-rs
              rust-analyzer
              rustpkg
              # youcan also rust-bin.{stable, beta, nightly}.{lastest, "2121-01-01"...}.default
              # where override {extensions = []; targets = [];}
            ];
          };
        }
    );
}
```
主要请参考 [oxalica/rust-overlay](https://github.com/oxalica/rust-overlay)
这里使用 `nightly` channel, `probe-rs` 是嵌入式工具, [[probe-rs 简易使用]]

`cargp new app` 创建项目
添加如下的依赖

![image.png](https://s2.loli.net/2024/04/14/dq9vPUiVx3Dupja.png)

编辑 `src/main.rs`
```Rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;
use panic_halt as _;

#[entry]
fn main() -> ! {
    loop {}
}
```

添加 `memory.x`, `cortex-m-rt` 需要
```
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}
```

添加 `.cargo/config.toml`, 配置选项， 也可以手动指定 `target`, 然后添加我们的链接脚本
```
[build]
target = "thumbv7m-none-eabi"


rustflags = [
  "-C", "link-arg=-Tlink.x",
]
```


当前是逻辑开发， 是没有标准库和标准入口 `main`. 所以我们引入 `cortex-m-rt::entry` 作为入口， 入口函数返回值是 `!` Never 类型。没有标准库也没有 `panic` 我们引入 `panic_halt`. 我们也可以关闭 `panic`, 有总比没有好。

你可能需要添加 `udev rules` 允许访问相应的烧录设备, 或者sudo一把梭
我的配置为： cmsis-dap和stm32f1


## 烧录

 `openocd` 
 
![[Pasted image 20240414185324.png]]

`probe-rs`

![image.png](https://s2.loli.net/2024/04/14/zBqcGsObUiAEtDF.png)
不需要手动指定 `interface`, 它可以自动识别. 


得益于 `nix`,  只需要一人配好环境， 他人动动小手即可

## 后语

关于 `probe-rs` 的讨论在 [[probe-rs 简易使用]], 这里我大概研究如何手动添加支持

rust嵌入式截至目前, `embedded-hal` 已经 `release`

如果对 `rust` 如何构建一个最小的启动可以参考 [The Embedonomicon](https://docs.rust-embedded.org/embedonomicon/), 这也是构建一个 `rt entry` 的教程

关于 `rust` 嵌入式的概况与技巧可以参考 [The Embedded Rust Book](https://docs.rust-embedded.org/book/), 这里主要告诉你 `rust` 在 `bare metal` 中的一些问题与指南

这是 `Google` 发起的文档， 对 `rust` 在各领域的简易介绍 [Comprehensive Rust](https://google.github.io/comprehensive-rust/bare-metal.html)

这里会提供一些 `Rust Embedded` 的前言新闻与 `blog`, 对应新手可以来这里学习. [# The Embedded Rustacean](https://www.theembeddedrustacean.com/), 所以不要抱怨没有 `Tutorial` 了

## 参考

[The rustup book](https://rust-lang.github.io/rustup/concepts/components.html)

[dev-templates](https://github.com/the-nix-way/dev-templates) - 有点落后版本了

[probe-rs](https://probe.rs/)  集合烧录，调试， 模拟于一身。

