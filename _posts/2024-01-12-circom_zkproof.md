## Circom 电路描述语言 和 零知识证明(ZKProof) 学习记录

Circom 的文档链接: [官方文档](https://docs.circom.io/)

其他文章:

[<==和<--的区别(赋值了为什么还需要约束)]https://github.com/0xPARC/zk-bug-tracker#8-assigned-but-not-constrained

[一篇很不错的科普]https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649#5539

---

### 搭建 Circom 开发环境

- VSCode 里有一些 extension, 从下载数看用的人不多, 如果有适合 Circom 的 extension, 请告诉我

- 安装 rustup

  ```shell
  curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
  ```

- 安装 node.js 和 npm, yarn 等包管理工具

- clone circom git

  ```shell
  git clone https://github.com/iden3/circom.git
  ```

- 用 rust 的编译工具 cargo 编译 circom

  ```shell
  cd circom
  cargo build --release
  ```

- 安装 circom

  ```shell
  cargo install --path circom
  ```

- 运行 circom -h 看看是否安装成功

  ```shell
  circom -h
  circom compiler 2.1.7
  IDEN3
  Compiler for the circom programming language


  USAGE:
  circom [FLAGS] [OPTIONS] [--] [input]

  FLAGS:
  --r1cs Outputs the constraints in r1cs format
  --sym Outputs witness in sym format
  --wasm Compiles the circuit to wasm
  --json Outputs the constraints in json format
  --wat Compiles the circuit to wat
  -c, --c Compiles the circuit to c
  --O0 No simplification is applied
  --O1 Only applies var to var and var to constant   simplification
  --O2 Full constraint simplification
  --verbose Shows logs during compilation
  --inspect Does an additional check over the constraints   produced
  --use_old_simplification_heuristics Applies the old   version of the heuristics when performing linear
  simplification
  --simplification_substitution Outputs the substitution in   json format
  -h, --help Prints help information
  -V, --version Prints version information

  OPTIONS:
  -o, --output <output> Path to the directory where the   output will be written [default: .]
  -p, --prime <prime> To choose the prime number to use to   generate the circuit. Receives the
  name of the curve (bn128, bls12381, goldilocks, grumpkin,   pallas, vesta)
  [default: bn128]
  -l <link_libraries>... Adds directory to library search   path
  --O2round <simplification_rounds> Maximum number of   rounds of the simplification process

  ARGS:
  <input> Path to a circuit with a main component   [default: ./circuit.circom]
  ```

- 安装 snarkjs

  ```shell
  npm install -g snarkjs
  ```

  snarkjs 可以从 circom 产出的文件中生成和验证 ZKProof(零知识证明)。也可以输出对应的 solidity 代码，用于部署到以太坊上完成验证 ZKProof 的工作。
