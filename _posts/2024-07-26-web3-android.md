---
layout: post
title: Web3学习日记 - Web3 Android 原生库测试 (二)
subtitle:
categories: Web3 Kotlin Java Android
tags: [Web3 Kotlin Java Android]
---

## Web3 学习日记 - Web3 Android 原生库测试 (二)

### web3j 和测试代码 repo

<https://github.com/hyperledger/web3j>

<https://github.com/yushihang/web3j.test>

### Android 项目配置

参见 web3j 文档和 测试代码 repo

### ganache 配置

参见 <https://yushihang.github.io/did/web3/smartcontracts/2024/05/10/debug-solidity-contracts-locally.html>

### 各测试用例

参见测试代码 <https://github.com/yushihang/web3j.test/blob/main/app/src/main/java/com/web3jtest/ViewModel.kt>

### 合约函数调用

合约函数的调用有两种方式

1. 通过 abi 导出对应的 java 函数定义
2. 通过函数名和动态参数/返回值调用

#### 通过 abi 导出对应的 java 函数

##### web3j-cli 安装

```bash
curl -L get.web3j.io | sh && source ~/.web3j/source.sh
```

##### java 环境

java 需要升级到 17 或以上

##### 导出 abi.json

参考 <https://yushihang.github.io/did/web3/smartcontracts/2024/05/10/debug-solidity-contracts-locally.html> 导出

##### 通过 abi.json 生成合约对应的 java 代码

```bash
web3j generate solidity -st -jt -a  abi.json -o . -p com.web3jtest
```

注意: abi.json 中如果有 uint256[64]之类的类型定义， 需要修改为非固定长度的 uint256[], 否则 java 文件导入项目后会遇到调用时的 exception 问题 ("Array types must be wrapped in a TypeReference")

(这个可能是 web3j-cli 的版本问题导致，期待后续版本可以解决)

##### java 文件修改(1)

导出后的 java 文件有些时候可能编译会失败，可以尝试 Android Studio 的自动修复。

例如

```java
    public RemoteFunctionCall<List> getGISTRootHistory(Uint256 start, Uint256 length) {
        final Function function = new Function(FUNC_GETGISTROOTHISTORY,
                Arrays.<Type>asList(start, length),
                Arrays.<TypeReference<?>>asList(new TypeReference<DynamicArray<GistRootInfo>>() {}));
        return executeRemoteCallSingleValueReturn(function);
```

修改为

```java
    public RemoteFunctionCall<Type> getGISTRootHistory(Uint256 start, Uint256 length) {
        final Function function = new Function(FUNC_GETGISTROOTHISTORY,
                Arrays.<Type>asList(start, length),
                Arrays.<TypeReference<?>>asList(new TypeReference<DynamicArray<GistRootInfo>>() {}));
        return executeRemoteCallSingleValueReturn(function);
```

##### java 文件修改(2)

如果调用函数时遇到 "parameter can not be null, try to use annotation @Parameterized to specify the parameter type" 这种 exception, 可以在对应的构造函数的参数前加上@Parameterized

例如如下代码中的@Parameterized(type = Uint256.class)

```java
 public static class GistProof extends DynamicStruct {
        public Uint256 root;

        public Bool existence;

        public DynamicArray<Uint256> siblings;

        public Uint256 index;


        public Uint256 value;

        public Bool auxExistence;

        public Uint256 auxIndex;

        public Uint256 auxValue;

        public GistProof(Uint256 root, Bool existence, @Parameterized(type = Uint256.class) DynamicArray<Uint256> siblings,
                Uint256 index, Uint256 value, Bool auxExistence, Uint256 auxIndex,
                Uint256 auxValue) {
            super(root, existence, siblings, index, value, auxExistence, auxIndex, auxValue);
            this.root = root;
            this.existence = existence;
            this.siblings = siblings;
            this.index = index;
            this.value = value;
            this.auxExistence = auxExistence;
            this.auxIndex = auxIndex;
            this.auxValue = auxValue;
        }
    }
```

##### 通过 abi 导出的 java 代码调用合约函数

公共函数

```java
    private suspend fun <T> handleContractCall(
        remoteFunctionCall: RemoteFunctionCall<T>
    ): T = suspendCancellableCoroutine { continuation ->
        val future = remoteFunctionCall.sendAsync()
        future.thenAccept { result ->
            continuation.resume(result)
        }.exceptionally { ex ->
            continuation.resumeWithException(ex)
            null
        }

        continuation.invokeOnCancellation {
            future.cancel(true)
        }
    }
```

###### view 函数

```java
    fun callContractViewFunction() {
        viewModelScope.launch {
            try {

                val gistProof = handleContractCall(stateContract.getGISTProof(Uint256(123)))
                val gistProofStr = gistProof.toPrettyString()
                println("response: $gistProofStr")
                (state.contractViewFunctionResponse as MutableStateFlow).value = gistProofStr
            } catch (e: Exception) {
                 e.printStackTrace()
            }
        }
    }

```

###### 涉及状态修改的函数

```java
   fun callContractStateModifyFunction() {
        viewModelScope.launch {
            try {
                val txHash = handleContractCall(
                    stateContract.transitStateGeneric(
                        Uint256(generateRandomBigInteger(200)),
                        Uint256(0),
                        Uint256(2),
                        Bool(true),
                        Uint256(1),
                        DynamicBytes.DEFAULT
                    )
                ).transactionHash
                println("txHash: $txHash")
                (state.contractStateModifyFunctionTxHash as MutableStateFlow).value = txHash
            } catch (e: Exception) {
                 e.printStackTrace()
            }
        }
    }

```

#### 动态构造调用合约函数

公共代码

```java
    private suspend fun <T : Response<*>> handleWeb3jRequest(
        request: Request<*, T>
    ): T = suspendCancellableCoroutine { continuation ->
        val future: CompletableFuture<T> = request.sendAsync()
        future.thenAccept { result ->
            continuation.resume(result)
        }.exceptionally { ex ->
            ex.printStackTrace()
            continuation.resumeWithException(ex)
            null
        }
```

###### view 函数

```java
   fun callContractViewFunction2() {
        viewModelScope.launch {
            try {

                val id = Uint256(BigInteger.valueOf(123))

                val function = Function(
                    "getGISTProof",
                    listOf(id),
                    listOf(
                        object : TypeReference<Uint256>() {},
                        object : TypeReference<Bool>() {},
                        object : TypeReference<DynamicArray<Uint256>>() {},
                        object : TypeReference<Uint256>() {},
                        object : TypeReference<Uint256>() {},
                        object : TypeReference<Bool>() {},
                        object : TypeReference<Uint256>() {},
                        object : TypeReference<Uint256>() {}
                    )
                )

                val encodedFunction = FunctionEncoder.encode(function)

                val transaction = Transaction.createEthCallTransaction(credentials.address, contractAddressHex, encodedFunction)

                val ethCall = handleWeb3jRequest(web3j.ethCall(transaction, DefaultBlockParameterName.LATEST))

                val result = ethCall.result
                val decodedResult = FunctionReturnDecoder.decode(result, function.outputParameters)

                // 解析返回值
                val root = decodedResult[0].value as BigInteger
                val existence = decodedResult[1].value as Boolean
                val siblings = (decodedResult[2].value as List<*>).map { it as BigInteger }
                val index = decodedResult[3].value as BigInteger
                val value = decodedResult[4].value as BigInteger
                val auxExistence = decodedResult[5].value as Boolean
                val auxIndex = decodedResult[6].value as BigInteger
                val auxValue = decodedResult[7].value as BigInteger

                // 创建 JSON 对象
                val json = JSONObject()
                json.put("root", root.toString())
                json.put("existence", existence)
                json.put("siblings", JSONArray(siblings.map { it.toString() }))
                json.put("index", index.toString())
                json.put("value", value.toString())
                json.put("auxExistence", auxExistence)
                json.put("auxIndex", auxIndex.toString())
                json.put("auxValue", auxValue.toString())

                val gistProofStr = json.toString(4)

                println("response: $gistProofStr")
                (state.contractViewFunctionResponse2 as MutableStateFlow).value = gistProofStr
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }
```

##### 涉及状态修改的函数

```java
fun callContractStateModifyFunction() {


        viewModelScope.launch {
            try {

                val func = Function("transitStateGeneric",
                    listOf(
                        Uint256(generateRandomBigInteger(200)),
                        Uint256(0),
                        Uint256(2),
                        Bool(true),
                        Uint256(1),
                        DynamicBytes.DEFAULT
                    ),
                    emptyList()
                )

                val encodedFunc = FunctionEncoder.encode(func)

                val nonce = handleWeb3jRequest(web3j.ethGetTransactionCount(credentials.address, DefaultBlockParameterName.LATEST)).transactionCount
                println("nonce: $nonce")
                val gasPrice = handleWeb3jRequest(web3j.ethGasPrice()).gasPrice
                println("gasPrice: $gasPrice")

                val gasLimit = BigInteger.valueOf(210000)

                val value = BigInteger.ZERO //we just call contract function, not sending ETH
                val rawTransaction = RawTransaction.createTransaction(
                    nonce, gasPrice, gasLimit, contractAddressHex, value, encodedFunc
                )

                val signedMessage = TransactionEncoder.signMessage(rawTransaction, credentials)

                val hexValue = Numeric.toHexString(signedMessage)

                val transactionHash = handleWeb3jRequest(web3j.ethSendRawTransaction(hexValue)).transactionHash

                (state.txHash as MutableStateFlow).value = transactionHash

                (state.contractStateModifyFunctionTxHash as MutableStateFlow).value = transactionHash

                updateContractStateModifyFunctionTxHashReceipt()

            } catch (e: Exception) {
                 e.printStackTrace()
            }
        }
    }
```
