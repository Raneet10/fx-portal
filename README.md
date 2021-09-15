# fx-portal(Flexible portal)

FxPortal for Polygon(prev Matic)Chain. No mapping. Seamless communication with Ethereum Network.

Audit - [Fx-Portal Contracts Audit by Halborn](https://github.com/fx-portal/contracts/blob/main/Polygon_FX_Portal_Smart_Contract_Security_Audit_Halborn_v1_0%20(1).pdf)

### What is Fx bridge (fx-portal)?

**It is a powerful yet simple implementation Polygon [state sync](https://docs.matic.network/docs/contribute/state-sync) mechanism. Polygon PoS bridge is based on it. The code in the `examples` folder are examples of the usage of this methodology. You can use these examples to build your own implementations or own custom bridge.**

In short, it's Meta bridge. This bridge allows any state-syncs without mapping.

#### Some use-cases of Fx-portal

* [ERC20 token tranfer from Ethereum to Matic-Chain without mapping request](https://github.com/jdkanani/fx-portal/tree/main/contracts/examples/erc20-transfer)
* [Lazy minting of ERC20 tokens on MaticChain](https://github.com/jdkanani/fx-portal/tree/main/contracts/examples/mintable-erc20-transfer)
* [State Transfer between Ethereum-Matic](https://github.com/jdkanani/fx-portal/tree/main/contracts/examples/state-transfer)

**What about [PoS portal](https://docs.matic.network/docs/develop/ethereum-matic/pos/getting-started)?**

PoS Portal is another bridge, but it works only for few ERC standards and requires mappings. It is more developer-friendly, allows customization without much headache. 

While Fx-portal focuses on permissionless-ness and flexibility, a developer might have to write more code but more customizable than PoS Portal. It requires no mapping.

**Can I build my bridge?**

Yes. You can check docs here: https://docs.matic.network/docs/develop/l1-l2-communication/ethereum-to-matic
https://docs.matic.network/docs/develop/l1-l2-communication/matic-to-ethereum 

### What are FxChild and FxRoot?

`FxChild` (FxChild.sol)  and `FxRoot` (FxRoot.sol) are main contracts on which mapping-less bridge works. It calls and passes data to user-defined methods on another chain without mapping.

**Mumbai**

| Contract | Deployed address  |
| :----- | :- |
| [FxRoot (Goerli)](https://goerli.etherscan.io/address/0x3d1d3E34f7fB6D26245E6640E1c50710eFFf15bA#code) | `0x3d1d3E34f7fB6D26245E6640E1c50710eFFf15bA` |
| [FxChild (Mumbai)](https://explorer-mumbai.maticvigil.com/address/0xCf73231F28B7331BBe3124B907840A94851f9f11/contracts) | `0xCf73231F28B7331BBe3124B907840A94851f9f11`|

**Mainnet**


| Contract | Deployed address  |
| :----- | :- |
| [FxRoot (Ethereum Mainnet)](https://etherscan.io/address/0xfe5e5d361b2ad62c541bab87c45a0b9b018389a2#code) | `0xfe5e5D361b2ad62c541bAb87C45a0B9B018389a2` |
| [FxChild (Matic Mainnnet)](https://explorer-mainnet.maticvigil.com/address/0x8397259c983751DAf40400790063935a11afa28a/contracts) | `0x8397259c983751DAf40400790063935a11afa28a`|


You can deploy your own `FxChild` and `FxRoot`, but no need. Except you want to have some fun and have extra ETH to throw away.

### What can I build with it?

* Arbitrary state bridge (examples/state-transfer)
* Normal ERC20 bridge (examples/erc2-transfer)
* ERC20 token generator bridge (example/mintable-erc20-transfer)

## ERC20 Transfer

The child and root tunnel contracts enable deposit of tokens on the root chain and withdrawal on the child chain.

### `FxERC20RootTunnel`

- `mapToken(address rootToken)` You can call the function on the deployed contract to map your ERC20 token and create a corresponding child token on the child chain.
- `deposit(address rootToken, address user, uint256 amount, bytes memory data)` Call deposit() with the address of the mapped token, the address who can withdraw with a corresponding amount (along with data, if needed). You must have approved the contract using the standard ERC20 `approve` function to spend your tokens first.

### `FxERC20ChildTunnel`

- `withdraw(address childToken, uint256 amount)` The address assigned in deposit() can withdraw all the amount of child token using this function. They will receive the child token created when first mapped.
- `rootToChildToken` This public variable contains the root token to child token mapping. You can query the mapping with the address of the root token to know the address of the deployed child token.

### Steps for ERC20 transfer from Ethereum to Matic

1. Deploy your own ERC20 token on the root chain. You will need this address later.
2. Approve the tokens for transfer by calling the `approve()` function of the root token with the address of the root tunnel and the amount as the arguments.
3. Proceed to call `deposit()` with the address of the receiver and amount on the root chain to receive the equivalent child token on the child chain. This will also map the token automatically. Alternatively, you can call `mapToken()` first before depositing.
4. That's it! 🎉 After mapping, you should now be able to execute cross-chain transfers using the `deposit` and `withdraw` functions of the tunnel. 

**Note:** After you have performed `deposit()` on root chain, it will take 10-15 minutes for state sync to happen. Once  state sync happens, you will get the tokens deposited on the given address.  

### Steps for ERC20 transfer from Matic to Ethereum

1. Proceed to call `withdraw()` with the respective token address and amount as arguments on the child contract to move the child tokens back to the designated receiver on the root chain. **Note the tx hash** as this will be used to generate the burn proof.

### Steps for ERC721 transfer from Ethereum to Matic

1. Deploy your own ERC721 token on the root chain. You will need this address later.
2. Approve the tokens for transfer by calling the `approve()` function of the root token with the address of the root tunnel and the token ID as the arguments.
3. Proceed to call `deposit()` with the address of the receiver and token ID on the root chain to receive the equivalent child token on the child chain. This will also map the token automatically. Alternatively, you can call `mapToken()` first before depositing.

**Note:** After you have performed `deposit()` on root chain, it will take 10-15 minutes for state sync to happen. Once  state sync happens, you will get the tokens deposited on the given address.  

### Steps for ERC721 transfer from Matic to Ethereum

1. Proceed to call `withdraw()` with the respective token address and token ID as arguments on the child contract to move the child tokens back to the designated receiver on the root chain. **Note the tx hash** as this will be used to generate the burn proof.

## ERC1155 Transfer

### `FxERC1155RootTunnel`

- `mapToken(rootToken)`: Used to map your root ERC1155 token to child chain
- `deposit(rootToken, user, id, amount, data)`: Function used to deposit root tokens to child chain
- `depositBatch(rootToken, user,  ids, amounts, bytes memory data)`: Used for multiple token Ids and corresponding amounts
- `receiveMessage(inputData)`: To be called after burn proof has been generated with the payload as `inputData`

### `FxERC1155ChildTunnel`

- `withdraw(childToken, id, amount, data)`: Used to withdraw token from Matic to Ethereum
- `withdrawBatch(childToken, ids, amounts, data)`: Same as withdraw but for withdrawing multiple token Ids

### Steps for depositing ERC1155 tokens from Ethereum to Matic

1. Deploy your ERC1155 token on the root chain. You will need this address later.
2. Call `setApprovalForAll(operator, approved)` on the deployed token with FxERC1155RootTunnel's address as `operator` to allow FxERC1155RootTunnel to transfer your tokens to FxERC1155ChildTunnel on Matic.
3. Call `mapToken()` on FxERC1155RootTunnel with your deployed token's address as `rootToken` . This will send a message to FxERC1155ChildTunnel instructing it to deploy and map the ERC1155 token on Matic chain. To query your child token address, call `rootToChildToken` on FxERC1155ChildTunnel.
4. Call `deposit()` on FxERC1155RootTunnel with address of the token on Ethereum as `rootToken` , receiver as `user` , token Id as `id` and the amount as `amount` . Alternatively, you can also call `depositBatch()` for multiple token ids.

**Note:** After you have performed `deposit()` on root chain, it will take 10-15 minutes for state sync to happen. Once  state sync happens, you will get the tokens deposited on the given address.  

### Steps to withdraw ERC1155 tokens from Matic to Ethereum

1. Call `withdraw()` on FxERC1155ChildTunnel with address of the child token deployed on Matic as the `childToken` and the token id as `id` (the child token address can be queried from `rootToChildToken` mapping). Alternatively, you can also call `withdrawBatch()` for multiple token ids and corresponding amounts. **Note the tx hash** as this will be used to generate the burn proof.

### Withdrawing your tokens on the root chain

**Note:** After you have performed `withdraw()` on child chain, it will take 30-60 minutes for a checkpoint to happen. Once the next checkpoint includes the burn tx, you can withdraw the tokens on the root chain.

1. Generate the burn proof using the tx hash and MESSAGE_SENT_EVENT_SIG. An example script to generate the proof can be found [here](https://gist.github.com/QEDK/62c4503d9a6a4bc57c491ee09376d71a).
2. Feed the generated payload as the argument to `receiveMessage()` in respective root tunnel contract. 

## Mintable ERC-20 Transfer

### `FxMintableERC20RootTunnel`

- `deposit(address rootToken, address user, uint256 amount, bytes memory data)` : To deposit tokens from Ethereum to Matic
- `receiveMessage(bytes memory inputData)` : Burn proof to be fed as the `inputData` to receive tokens on root chain

### `FxMintableERC20ChildTunnel`

- `deployChildToken(uint256 uniqueId, string memory name, string memory symbol, uint8 decimals)` : To deploy a ERC-20 token on Matic chain
- `mintToken(address childToken, uint256 amount)` : Mint a particular amount of tokens on Matic chain
- `withdraw(address childToken, uint256 amount)` : To burn tokens on child chain in order to withdraw on the root chain

### Steps for minting tokens on Matic

1. Call the `deployChildToken()` on `FxMintableERC20ChildTunnel` and pass the necessary token info as parameters. This emits a `TokenMapped` event which contains `rootToken` and `childToken` address. Note these addresses.
2. Call `mintToken()` on `FxMintableERC20ChildTunnel` to mint tokens on the child chain.
3. Call `withdraw()` on `FxMintableERC20ChildTunnel` to withdraw tokens from the Matic chain. Note the tx hash as this will come handy to generate the burn proof.
4. Wait for the burn tx to be included in the checkpoint (~30-45 minutes). After this, generate the burn proof using an example script [here](https://gist.github.com/QEDK/62c4503d9a6a4bc57c491ee09376d71a). 

### Steps for withdrawing tokens on Ethereum

Feed the generated burn proof as the argument to `receiveMessage()` in `FxMintableERC20RootTunnel`. After this the token balance would be reflected on root chain.

### Steps to deposit tokens back from Ethereum to Matic

1. Make sure you approve `FxMintableERC20RootTunnel` to transfer your tokens. 
2. Call `deposit()` in `FxMintableERC20RootTunnel` with the `rootToken` as address of root token and `user` as recipient.
3. Wait for the state to sync (~10-15 mins). After this you can query the target recipient's balance on child chain. 

# Example deployments

**Goerli**:

- Checkpoint Manager: 0x2890bA17EfE978480615e330ecB65333b880928e
- FxERC20: 0x93AfF096e6B4096Cd46268D3D9C6Bb6B2E1A44Bc
- Dummy ERC20 token: 0xe9c7873f81c815d64c71c2233462cb175e4765b3
- FxERC20RootTunnel: 0xbfDeFCd92335b22b205bb5b63B9eC909D6e99C16
- FxMintableERC20RootTunnel: 0xA200766a7D64E54611E2D232AA6c1f870aCb63c1
- Dummy ERC721 token: 0x73594a053cb5ddDE5558268d28a774375C4E23dA
- FxERC721RootTunnel: 0x779Ab15408aD3952dfFe805cf5829FAb6de54975
- `mapToken` tx for dummy ERC721 token: [https://goerli.etherscan.io/tx/0x4020618fe7ad766d075274a17e68d5a26c558bbcc3a041be438b1526a8bcac31](https://goerli.etherscan.io/tx/0x4020618fe7ad766d075274a17e68d5a26c558bbcc3a041be438b1526a8bcac31)
- `deposit` tx for dummy ERC721 token: [https://goerli.etherscan.io/tx/0x371089ab5a016edcea3a150b3ed63e3146ed01c4e90189dea22a9ada7964220e](https://goerli.etherscan.io/tx/0x371089ab5a016edcea3a150b3ed63e3146ed01c4e90189dea22a9ada7964220e)
- FxERC1155RootTunnel : 0x3d6373bd5DdcB297EBFc7dBafD82736F81D0E235
- FxERC1155 : 0x54780feF8A41402BA5ac6Ef6a48faaAd57382bC4
- Dummy ERC1155 Token: 0x1906d395752FE0c930f8d061DFEb785eBE6f0B4E

**Mumbai**:

- FxERC20: 0xDDE69724AeFBdb084413719fE745aB66e3b055C7
- FxERC20ChildRootTunnel: 0xd0ddbab3bD60E591c300b2e365D732d9Ce76D3fd
- FxMintableERC20ChildTunnel: 0xA2C7eBEf68B444056b4A39C2CEC23844275C56e9
- Child token dummy ERC20: 0x346d21bc2bD3dEE2d1168E1A632b10D1d7B9c0A
- Withdraw tx for dummy ERC20: [https://explorer-mumbai.maticvigil.com/tx/0x4756b76a9611cffee3d2eb645819e988c34615621ea256f818ab788d81e1f838](https://explorer-mumbai.maticvigil.com/tx/0x4756b76a9611cffee3d2eb645819e988c34615621ea256f818ab788d81e1f838/token-transfers)
- FxERC721: 0xf2720927E048726267C0221ffA41A88528048726
- FxERC721ChildTunnel: 0x0A52BC7F2eF0F9a7fBe2e48ad8ceE0752ea900C3
- FxERC1155ChildTunnel: 0x797dFAFE5ac545038245e235Bf9f1c5966E3C5B2
- FxERC1155: 0x80be8Cf927047A40d3f5791BF7436D8c95b3Ae5C


