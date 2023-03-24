## Tornado Cash隐私解决方案
Tornado Cash是一个基于zkSNARKs的非托管Ethereum和ERC20隐私解决方案，它通过打破接收者和目标地址之间的链上联系来提高交易隐私。它使用一个智能合约接受ETH存款，并可由不同的地址提取。每当新地址提取ETH时，就没有办法将提取与存款关联起来，确保完全的隐私。

用户生成一个秘密并将其哈希（称为承诺）以及存款金额发送到Tornado智能合约来进行存款。合约接受存款并将承诺添加到其存款列表中。

稍后，用户决定进行提取。为了实现提取，用户应提供证明表明他或她拥有未使用的存款承诺的秘密，并且zkSnark技术可以实现在不透露对应关系的情况下进行证明。智能合约将检查证明并将存款资金转移到指定的提取地址。外部观察者将无法确定此提取来自哪个存款。

您可以在[此Medium文章](https://medium.com/@tornado.cash/introducing-private-transactions-on-ethereum-now-42ee915babe0)中了解更多信息。

## Specs
-   Deposit gas cost: 1088354 (43381 + 50859 * tree_depth)
-   Withdraw gas cost: 301233
-   Circuit Constraints = 28271 (1869 + 1325 * tree_depth)
-   Circuit Proof time = 10213ms (1071 + 347 * tree_depth)
-   Serverless

[![image](https://github.com/Guyungy/tornado-core/raw/master/docs/diagram.png)](https://github.com/Guyungy/tornado-core/blob/master/docs/diagram.png)

## 它通过审核了吗？

Tornado.cash协议、电路和智能合约由专门从事零知识、密码学和智能合约的[ABDK Consulting](https://www.abdk.consulting/)的一组专家进行了审计。

在审计过程中，未发现任何重大问题，并修复了所有未解决的问题。结果可以在以下链接中找到：

-   密码学审查 [https://tornado.cash/audits/TornadoCash_cryptographic_review_ABDK.pdf](https://tornado.cash/audits/TornadoCash_cryptographic_review_ABDK.pdf)
-   智能合约审计 [https://tornado.cash/audits/TornadoCash_contract_audit_ABDK.pdf](https://tornado.cash/audits/TornadoCash_contract_audit_ABDK.pdf)
-   Zk-SNARK电路审计 [https://tornado.cash/audits/TornadoCash_circuit_audit_ABDK.pdf](https://tornado.cash/audits/TornadoCash_circuit_audit_ABDK.pdf)

当前正在对底层circomlib依赖项进行审核，团队已经发布了大部分修复发现问题的解决方案。

## 要求

1.  `node v11.15.0`
2.  `npm install -g npx`

## 用法

您可以在cli.js中看到示例用法，它可以在控制台和浏览器中使用。

1.  `npm install`
2.  `cp .env.example .env`
3.  `npm run build` - 这可能需要10分钟或更长时间
4.  `npx ganache-cli`
5.  `npm run test` - 可选地运行测试。第一次尝试可能会失败，请再次运行。

在Kovan上使用浏览器版本：

1.  `vi .env` - 将您的Kovan私钥添加到部署合约
2.  `npm run migrate`
3.  `npx http-server` - 服务当前目录，您可以使用任何其他静态http服务器
4.  打开`localhost:8080`

使用命令行版本。适用于Ganache、Kovan和Mainnet：

### 初始化

1.  `cp .env.example .env`
2.  `npm run download`
3.  `npm run build:contract`

### Ganache

1.  确保您完成了初始化步骤
2.  `ganache-cli -i 1337`
3.  `npm run migrate:dev`
4.  `./cli.js test`
5.  `./cli.js --help`

### Kovan、Mainnet

1.  请使用[https://github.com/tornadocash/tornado-cli](https://github.com/tornadocash/tornado-cli)。原因：因为tornado-core使用websnark`2041cfa5fa0b71cd5cca9022a4eeea4afe28c9f7`提交哈希以便与本地可信设置配合使用。Tornado-cli使用了`tornadoCash`生产可信设置的提交哈希为`4c0af6a8b65aabea3c09f377f63c44e7a58afa6d`。
2. `./cli.js deposit ETH 0.1 --rpc https://kovan.infura.io/v3/27a9649f826b4e31a83e07ae09a87448`

> 你的提示：tornado-eth-0.1-42-0xf73dd6833ccbcc046c44228c8e2aa312bf49e08389dadc7c65e6a73239867b7ef49c705c4db227e2fadd8489a494b6880bdcb6016047e019d1abec1c7652 旋风 ETH 余额为 8.9，发送者账户 ETH 余额为 1004873.470619891361352542，正在提交存款交易，旋风 ETH 余额为 9，发送者账户 ETH 余额为 1004873.361652048361352542。

`./cli.js withdraw tornado-eth-0.1-42-0xf73dd6833ccbcc046c44228c8e2aa312bf49e08389dadc7c65e6a73239867b7ef49c705c4db227e2fadd8489a494b6880bdcb6016047e019d1abec1c7652 0x8589427373D6D84E98730D7795D8f6f8731FDA16 --rpc https://kovan.infura.io/v3/27a9649f826b4e31a83e07ae09a87448 --relayer https://kovan-frelay.duckdns.org`

> 中继地址为：0x6A31736e7490AbE5D5676be059DFf064AB4aC754，从旋风合约获取当前状态，生成 SNARK 证明，证明时间为 9117.051 毫秒，通过中继发送提款交易，交易通过中继提交。在 etherscan 查看交易：[https://kovan.etherscan.io/tx/0xcb21ae8cad723818c6bc7273e83e00c8393fcdbe74802ce5d562acad691a2a7b，交易在区块](https://kovan.etherscan.io/tx/0xcb21ae8cad723818c6bc7273e83e00c8393fcdbe74802ce5d562acad691a2a7b%EF%BC%8C%E4%BA%A4%E6%98%93%E5%9C%A8%E5%8C%BA%E5%9D%97) 17036120 中被确认，完成。

## 部署 ETH Tornado Cash

1.  `cp .env.example .env`
2.  调整所有必要参数
3.  `npx truffle migrate --network kovan --reset --f 2 --to 4`

## 部署 ERC20 Tornado Cash

1.  `cp .env.example .env`
2.  调整所有必要参数
3.  `npx truffle migrate --network kovan --reset --f 2 --to 3`
4.  `npx truffle migrate --network kovan --reset --f 5`

**注意**：如果您想要在所有实例中重复使用相同的验证器，则在部署一个实例后，应只对 ETH 或 ERC20 合约运行第四或第五个迁移（`--f 4 --to 4` 或 `--f 5`）。

## 如何将 ENS 名称解析为中继的 DNS 名称

1.  访问 [[https://etherscan.io/enslookup](](https://etherscan.io/enslookup%5D()[https://etherscan.io/enslookup](https://etherscan.io/enslookup)
2. 复制名称哈希（1），然后点击“解析器”链接（2）。
3. ![[Pasted image 20230324152728.png]]
4. Go to the `Contract` tab. Click on `Read Contract` and scroll down to the `5. text` method.
5.  Put the values: [![resolver](https://github.com/Guyungy/tornado-core/raw/master/docs/resolver.png)](https://github.com/Guyungy/tornado-core/blob/master/docs/resolver.png)
6.  Click `Query` and you will get the DNS name. Just add `https://` to it and use it as `relayer url`
## Credits

特别感谢 @barryWhiteHat 和 @kobigurk 的宝贵意见，以及 @jbaylina 提供的精彩 [Circom](https://github.com/iden3/circom) 和 [Websnark](https://github.com/iden3/websnark) 框架。

## 最小演示示例

1.  `npm i`
2.  `ganache-cli -d`
3.  `npm run download`
4.  `npm run build:contract`
5.  `cp .env.example .env`
6.  `npm run migrate:dev`
7.  `node minimal-demo.js`

## 运行测试/覆盖率

准备测试环境：

`yarn install yarn download cp .env.example .env npx ganache-cli > /dev/null & npm run migrate:dev`

运行测试：

`yarn test`

运行覆盖率：

`yarn coverage`

## 模拟 MPC 可信设置仪式

`cargo install zkutil npx circom circuits/withdraw.circom -o build/circuits/withdraw.json zkutil setup -c build/circuits/withdraw.json -p build/circuits/withdraw.params zkutil export-keys -c build/circuits/withdraw.json -p build/circuits/withdraw.params -r build/circuits/withdraw_proving_key.json -v build/circuits/withdraw_verification_key.json zkutil generate-verifier -p build/circuits/withdraw.params -v build/circuits/Verifier.sol sed -i -e 's/pragma solidity \^0.6.0/pragma solidity 0.5.17/g' ./build/circuits/Verifier.sol`
