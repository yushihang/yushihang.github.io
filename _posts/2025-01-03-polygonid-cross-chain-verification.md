---
layout: post
title: PrivadoID (polygonID) Cross-Chain Verification
subtitle:
categories: Web3 PrivadoID polygonID ZKProof Cross-Chain Verification
tags: [Web3, PrivadoID, polygonID, ZKProof, Cross-Chain, Verification]
---

## PrivadoID (polygonID) Cross-Chain Verification

### 资料

- Privado.ID Cross Chain Verification: <https://docs.privado.id/docs/verifier/on-chain-verification/cross-chain/>

- oracle repo:
  orig: <https://github.com/0xPolygonID/driver-did-polygonid>
  fork: <https://github.com/privadoID-study/archive-2025-01-driver-did-polygonid>

- flutter sdk repo:
  orig: <https://github.com/0xPolygonID/polygonid-flutter-sdk>
  fork: <https://github.com/privadoID-study/archive-2025-01-polygonid-flutter-sdk>

  - commit: around 2023-12-01

- c-polygonid lib repo:
  orig: <https://github.com/0xPolygonID/c-polygonid>
  fork: <https://github.com/privadoID-study/archive-2025-01-c-polygonid>

  - commit: https://github.com/0xPolygonID/c-polygonid/commit/e420c85f5ec2011053768102ad8adc37aab590a2#diff-1025cc72a1ead96612ce3c7351a4ac5b62776b020c032fc12b67c39b14f1fe33

- test cases repo:
  orig: <https://github.com/0xPolygonID/contracts>
  fork: <https://github.com/privadoID-study/archive-2025-01-polygonid-contracts>

- demo ios app repo:
  <https://github.com/privadoID-study/archive-2025-01-hsbc-polygonid-flutter-sdk>

- off-line vp verification script repo:
  <https://github.com/privadoID-study/archive-2025-01-python-verifier-for-iden3-zkp>

### 如何编译 原生库 + Flutter SDK + iOS App

参见各 repo 的 README.md 或者 buildxxx.sh 脚本

### 如何构建 oracle 并运行

- 安装 Docker Desktop 并运行
  <https://www.docker.com/products/docker-desktop/>

- 安装 docker cli 和 golang
  brew install docker
  brew install go

- clone oracle repo

  ```bash
  git clone https://github.com/privadoID-study/archive-2025-01-driver-did-polygonid
  cd archive-2025-01-driver-did-polygonid
  ```

- 配置 (fork repo 里已经包含)
  创建 resolvers.settings.yaml(以 Amoy 为例)
  不同的合约地址可以在这里查询:<https://docs.privado.id/docs/smart-contracts/>

  ```yaml
  polygon:
  amoy:
    contractAddress: "0x1a4cC30f2aA0377b0c3bc9848766D90cb4404124"
    networkURL: "https://rpc-amoy.polygon.technology/"
  ```

- 编译

  ```bash
  docker build -t driver-did-polygonid:local .
  ```

  ```bash
  ❯  docker build -t driver-did-polygonid:local .
  [+] Building 31.2s (18/18) FINISHED                                                                   docker:desktop-linux
  => [internal] load build definition from Dockerfile                                                                  0.0s
  => => transferring dockerfile: 666B                                                                                  0.0s
  => [internal] load metadata for docker.io/library/golang:1.18-alpine                                                 3.4s
  => [internal] load .dockerignore                                                                                     0.1s
  => => transferring context: 2B                                                                                       0.0s
  => [internal] load build context                                                                                     0.0s
  => => transferring context: 217.98kB                                                                                 0.0s
  => [base 1/9] FROM docker.io/library/golang:1.18-alpine@sha256:77f25981bd57e60a510165f3be89c901aec90453fd0f1c5a4569  8.4s
  => => resolve docker.io/library/golang:1.18-alpine@sha256:77f25981bd57e60a510165f3be89c901aec90453fd0f1c5a45691f6cb  0.0s
  => => sha256:da3aa66ae7ca151fbd33126b15fa4d01bd5ceb35d53de36a8c6c82ecde58b596 286.26kB / 286.26kB                    2.4s
  => => sha256:72d1fefe56e2997391a3dee0aa70cbe76557be6201cca145ad63b40e3767e061 110.45MB / 110.45MB                    6.6s
  => => sha256:a13fd264aa8ef04a29b827fe2454ac2bb9c52f70bf0b0eadfb45ca4867190c00 156B / 156B                            0.4s
  => => sha256:a9eaa45ef418e883481a13c7d84fa9904f2ec56789c52a87ba5a9e6483f2b74f 3.26MB / 3.26MB                        2.6s
  => => extracting sha256:a9eaa45ef418e883481a13c7d84fa9904f2ec56789c52a87ba5a9e6483f2b74f                             0.4s
  => => extracting sha256:da3aa66ae7ca151fbd33126b15fa4d01bd5ceb35d53de36a8c6c82ecde58b596                             0.0s
  => => extracting sha256:72d1fefe56e2997391a3dee0aa70cbe76557be6201cca145ad63b40e3767e061                             1.7s
  => => extracting sha256:a13fd264aa8ef04a29b827fe2454ac2bb9c52f70bf0b0eadfb45ca4867190c00                             0.0s
  => [stage-1 1/4] COPY ./resolvers.settings.yaml /app/resolvers.settings.yaml                                         0.0s
  => [base 2/9] WORKDIR /build                                                                                         0.4s
  => [base 3/9] RUN apk add --no-cache --update git                                                                    2.1s
  => [base 4/9] COPY ./cmd ./cmd                                                                                       0.0s
  => [base 5/9] COPY ./pkg ./pkg                                                                                       0.0s
  => [base 6/9] COPY go.mod ./                                                                                         0.0s
  => [base 7/9] COPY go.sum ./                                                                                         0.0s
  => [base 8/9] RUN go mod download                                                                                   11.8s
  => [base 9/9] RUN CGO_ENABLED=0 go build -o ./driver ./cmd/driver/main.go                                            4.1s
  => [stage-1 2/4] COPY --from=base /build/driver /app/driver                                                          0.1s
  => [stage-1 3/4] COPY --from=base /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/                                 0.0s
  => [stage-1 4/4] WORKDIR /app                                                                                        0.0s
  => exporting to image                                                                                                0.5s
  => => exporting layers                                                                                               0.4s
  => => exporting manifest sha256:9e27f616eb9e3ce448fb8a8bdb467d558ec1d6b7facc33dfcd93be1a1b8977d1                     0.0s
  => => exporting config sha256:43fef02282245cbf66cd5980b1724b12492a1eb8efc20d2c26b9775767f231e7                       0.0s
  => => exporting attestation manifest sha256:80648b9ca86f152317b4abcb2898ab511d6b3eb1ad08c608254c7eb4c752cfa1         0.0s
  => => exporting manifest list sha256:cdbe96c89bdd1ce8e0e04f6b3d9f50f6a3e964512f69d4f6aa928650c1de2952                0.0s
  => => naming to docker.io/library/driver-did-polygonid:local                                                         0.0s
  => => unpacking to docker.io/library/driver-did-polygonid:local                                                      0.1s

  ```

- 运行

  ```bash
  docker run -p 8080:8080 driver-did-polygonid:local
  ```

- 测试

  ```bash
  npm install -g newman
  ```

  ```bash
  newman run tests/e2e/users_tests.postman_collection.json
  ```
