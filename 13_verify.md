# 13. Validation

Verify information recorded on the blockchain.
While recording data on the blockchain is done with the agreement of all nodes, 
 **referencing data** on the blockchain is achieved by obtaining information from a single 
node. For this reason, to avoid making a new transaction based on information from an untrusted node, the data obtained from the node must be verified.

## 13.1 Transaction validation

Verify that the transaction is included in the block header. If this verification succeeds, the transaction can be considered as authorised by blockchain agreement.

Before running the sample scripts in this chapter, please load the following necessary libraries.

```js
Buffer = require("/node_modules/buffer").Buffer;
cat = require("/node_modules/catbuffer-typescript");
sha3_256 = require("/node_modules/js-sha3").sha3_256;

accountRepo = repo.createAccountRepository();
blockRepo = repo.createBlockRepository();
stateProofService = new sym.StateProofService(repo);
```

### Payload to be verified

The transaction payload to be verified in this case and the block height at which the transaction is supposed to have been recorded.

```js
payload =
  "C00200000000000093B0B985101C1BDD1BC2BF30D72F35E34265B3F381ECA464733E147A4F0A6B9353547E2E08189EF37E50D271BEB5F09B81CE5816BB34A153D2268520AF630A0A0E5C72B0D5946C1EFEE7E5317C5985F106B739BB0BC07E4F9A288417B3CD6D26000000000198414140770200000000002A769FB40000000076B455CFAE2CCDA9C282BF8556D3E9C9C0DE18B0CBE6660ACCF86EB54AC51B33B001000000000000DB000000000000000E5C72B0D5946C1EFEE7E5317C5985F106B739BB0BC07E4F9A288417B3CD6D26000000000198544198205C1A4CE06C45B3A896B1B2360E03633B9F36BF7F22338B000000000000000066653465353435393833444430383935303645394533424446434235313637433046394232384135344536463032413837364535303734423641303337414643414233303344383841303630353343353345354235413835323835443639434132364235343233343032364244444331443133343139464435353438323930334242453038423832304100000000006800000000000000B2D4FD84B2B63A96AA37C35FC6E0A2341CEC1FD19C8FFC8D93CCCA2B028D1E9D000000000198444198205C1A4CE06C45B3A896B1B2360E03633B9F36BF7F2233BC089179EBBE01A81400140035383435344434373631364336433635373237396800000000000000B2D4FD84B2B63A96AA37C35FC6E0A2341CEC1FD19C8FFC8D93CCCA2B028D1E9D000000000198444198205C1A4CE06C45B3A896B1B2360E03633B9F36BF7F223345ECB996EDDB9BEB1400140035383435344434373631364336433635373237390000000000000000B2D4FD84B2B63A96AA37C35FC6E0A2341CEC1FD19C8FFC8D93CCCA2B028D1E9D5A71EBA9C924EFA146897BE6C9BB3DACEFA26A07D687AC4A83C9B03087640E2D1DDAE952E9DDBC33312E2C8D021B4CC0435852C0756B1EBD983FCE221A981D02";
height = 59639;
```

### Payload validation

Verify the contents of the transaction.

```js
tx = sym.TransactionMapping.createFromPayload(payload);
hash = sym.Transaction.createTransactionHash(
  payload,
  Buffer.from(generationHash, "hex")
);
console.log(hash);
console.log(tx);
```

###### Sample output

```js
> 257E2CAECF4B477235CA93C37090E8BE58B7D3812A012E39B7B55BA7D7FFCB20
> AggregateTransaction
    > cosignatures: Array(1)
      0: AggregateTransactionCosignature
        signature: "5A71EBA9C924EFA146897BE6C9BB3DACEFA26A07D687AC4A83C9B03087640E2D1DDAE952E9DDBC33312E2C8D021B4CC0435852C0756B1EBD983FCE221A981D02"
        signer: PublicAccount
          address: Address {address: 'TAQFYGSM4BWELM5IS2Y3ENQOANRTXHZWX57SEMY', networkType: 152}
          publicKey: "B2D4FD84B2B63A96AA37C35FC6E0A2341CEC1FD19C8FFC8D93CCCA2B028D1E9D"
      deadline: Deadline {adjustedValue: 3030349354}
    > innerTransactions: Array(3)
        0: TransferTransaction {type: 16724, networkType: 152, version: 1, deadline: Deadline, maxFee: UInt64, …}
        1: AccountMetadataTransaction {type: 16708, networkType: 152, version: 1, deadline: Deadline, maxFee: UInt64, …}
        2: AccountMetadataTransaction {type: 16708, networkType: 152, version: 1, deadline: Deadline, maxFee: UInt64, …}
      maxFee: UInt64 {lower: 161600, higher: 0}
      networkType: 152
      signature: "93B0B985101C1BDD1BC2BF30D72F35E34265B3F381ECA464733E147A4F0A6B9353547E2E08189EF37E50D271BEB5F09B81CE5816BB34A153D2268520AF630A0A"
    > signer: PublicAccount
        address: Address {address: 'TBIL6D6RURP45YQRWV6Q7YVWIIPLQGLZQFHWFEQ', networkType: 152}
        publicKey: "0E5C72B0D5946C1EFEE7E5317C5985F106B739BB0BC07E4F9A288417B3CD6D26"
      transactionInfo: undefined
      type: 16705
```

### Signatory validation

The transaction can be verified by confirming that it has been included in the block, but just to make sure, it is possible to verify the signature of the transaction with the account's public key.

```js
res = alice.publicAccount.verifySignature(
  tx.getSigningBytes(
    [...Buffer.from(payload, "hex")],
    [...Buffer.from(generationHash, "hex")]
  ),
  "93B0B985101C1BDD1BC2BF30D72F35E34265B3F381ECA464733E147A4F0A6B9353547E2E08189EF37E50D271BEB5F09B81CE5816BB34A153D2268520AF630A0A"
);
console.log(res);
```

```js
> true
```

Only the part to be signed is extracted in getSigningBytes.  
Note that the part to be extracted is different for normal transactions and Aggregate Transactions.

### Calculation of the Merkle component hash

The hash value of the transaction does not contain information about the cosignatory.  
On the other hand, the Merkle root stored in the block header contains a hash of the transaction with the information of the cosignatory included.  
Therefore, when verifying whether a transaction exists inside a block, the transaction hash must be converted to a Merkle component hash.

```js
merkleComponentHash = hash;
if (tx.cosignatures !== undefined && tx.cosignatures.length > 0) {
  const hasher = sha3_256.create();
  hasher.update(Buffer.from(hash, "hex"));
  for (cosignature of tx.cosignatures) {
    hasher.update(Buffer.from(cosignature.signer.publicKey, "hex"));
  }
  merkleComponentHash = hasher.hex().toUpperCase();
}
console.log(merkleComponentHash);
```

```js
> C8D1335F07DE05832B702CACB85B8EDAC2F3086543C76C9F56F99A0861E8F235
```

### In block validation

Retrieve the Merkle tree from the node and check that the Merkle root of the block header can be derived from the merkleComponentHash calculated.

```js
function validateTransactionInBlock(leaf, HRoot, merkleProof) {
  if (merkleProof.length === 0) {
    // There is a single item in the tree, so HRoot' = leaf.
    return leaf.toUpperCase() === HRoot.toUpperCase();
  }

  const HRoot0 = merkleProof.reduce((proofHash, pathItem) => {
    const hasher = sha3_256.create();
    if (pathItem.position === sym.MerklePosition.Left) {
      return hasher.update(Buffer.from(pathItem.hash + proofHash, "hex")).hex();
    } else {
      return hasher.update(Buffer.from(proofHash + pathItem.hash, "hex")).hex();
    }
  }, leaf);
  return HRoot.toUpperCase() === HRoot0.toUpperCase();
}

//Calculate from transaction
leaf = merkleComponentHash.toLowerCase(); //merkleComponentHash

//Retrieve from node
HRoot = (await blockRepo.getBlockByHeight(height).toPromise())
  .blockTransactionsHash;
merkleProof = (await blockRepo.getMerkleTransaction(height, leaf).toPromise())
  .merklePath;

result = validateTransactionInBlock(leaf, HRoot, merkleProof);
console.log(result);
```

```js
> true
```

It has been verified that the transaction information is contained in the block header.

## 13.2 Block header validation

Verify that the known block hash value (e.g. finalised block) can be traced back to the block header that is being verified.

### Normal block validation

```js
block = await blockRepo.getBlockByHeight(height).toPromise();
previousBlock = await blockRepo.getBlockByHeight(height - 1).toPromise();
if (block.type === sym.BlockType.NormalBlock) {
  hasher = sha3_256.create();
  hasher.update(Buffer.from(block.signature, "hex")); //signature
  hasher.update(Buffer.from(block.signer.publicKey, "hex")); //publicKey
  hasher.update(cat.GeneratorUtils.uintToBuffer(block.version, 1));
  hasher.update(cat.GeneratorUtils.uintToBuffer(block.networkType, 1));
  hasher.update(cat.GeneratorUtils.uintToBuffer(block.type, 2));
  hasher.update(
    cat.GeneratorUtils.uint64ToBuffer([block.height.lower, block.height.higher])
  );
  hasher.update(
    cat.GeneratorUtils.uint64ToBuffer([
      block.timestamp.lower,
      block.timestamp.higher,
    ])
  );
  hasher.update(
    cat.GeneratorUtils.uint64ToBuffer([
      block.difficulty.lower,
      block.difficulty.higher,
    ])
  );
  hasher.update(Buffer.from(block.proofGamma, "hex"));
  hasher.update(Buffer.from(block.proofVerificationHash, "hex"));
  hasher.update(Buffer.from(block.proofScalar, "hex"));
  hasher.update(Buffer.from(previousBlock.hash, "hex"));
  hasher.update(Buffer.from(block.blockTransactionsHash, "hex"));
  hasher.update(Buffer.from(block.blockReceiptsHash, "hex"));
  hasher.update(Buffer.from(block.stateHash, "hex"));
  hasher.update(
    sym.RawAddress.stringToAddress(block.beneficiaryAddress.address)
  );
  hasher.update(cat.GeneratorUtils.uintToBuffer(block.feeMultiplier, 4));
  hash = hasher.hex().toUpperCase();
  console.log(hash === block.hash);
}
```

If the output was true, this block hash acknowledges the existence of the previous block hash value. In the same way, the "n"th block confirms the existence of the "n-1th" block and finally arrives at the block being verified.

Now we have a known finalised block that can be verified by querying any node to support the existence of the block to be verified.

### Importance block validation

ImportanceBlocks are the blocks where the importance value is recalculated. Importance blocks occur every 720 blocks on Mainnet and every 180 blocks on Testnet. In addition to the NormalBlock, the following information is added.

- votingEligibleAccountsCount
- harvestingEligibleAccountsCount
- totalVotingBalance
- previousImportanceBlockHash

```js
block = await blockRepo.getBlockByHeight(height).toPromise();
previousBlock = await blockRepo.getBlockByHeight(height - 1).toPromise();
if (block.type === sym.BlockType.ImportanceBlock) {
  hasher = sha3_256.create();
  hasher.update(Buffer.from(block.signature, "hex")); //signature
  hasher.update(Buffer.from(block.signer.publicKey, "hex")); //publicKey
  hasher.update(cat.GeneratorUtils.uintToBuffer(block.version, 1));
  hasher.update(cat.GeneratorUtils.uintToBuffer(block.networkType, 1));
  hasher.update(cat.GeneratorUtils.uintToBuffer(block.type, 2));
  hasher.update(
    cat.GeneratorUtils.uint64ToBuffer([block.height.lower, block.height.higher])
  );
  hasher.update(
    cat.GeneratorUtils.uint64ToBuffer([
      block.timestamp.lower,
      block.timestamp.higher,
    ])
  );
  hasher.update(
    cat.GeneratorUtils.uint64ToBuffer([
      block.difficulty.lower,
      block.difficulty.higher,
    ])
  );
  hasher.update(Buffer.from(block.proofGamma, "hex"));
  hasher.update(Buffer.from(block.proofVerificationHash, "hex"));
  hasher.update(Buffer.from(block.proofScalar, "hex"));
  hasher.update(Buffer.from(previousBlock.hash, "hex"));
  hasher.update(Buffer.from(block.blockTransactionsHash, "hex"));
  hasher.update(Buffer.from(block.blockReceiptsHash, "hex"));
  hasher.update(Buffer.from(block.stateHash, "hex"));
  hasher.update(
    sym.RawAddress.stringToAddress(block.beneficiaryAddress.address)
  );
  hasher.update(cat.GeneratorUtils.uintToBuffer(block.feeMultiplier, 4));
  hasher.update(
    cat.GeneratorUtils.uintToBuffer(block.votingEligibleAccountsCount, 4)
  );
  hasher.update(
    cat.GeneratorUtils.uint64ToBuffer([
      block.harvestingEligibleAccountsCount.lower,
      block.harvestingEligibleAccountsCount.higher,
    ])
  );
  hasher.update(
    cat.GeneratorUtils.uint64ToBuffer([
      block.totalVotingBalance.lower,
      block.totalVotingBalance.higher,
    ])
  );
  hasher.update(Buffer.from(block.previousImportanceBlockHash, "hex"));

  hash = hasher.hex().toUpperCase();
  console.log(hash === block.hash);
}
```

Verifying stateHashSubCacheMerkleRoots for accounts and metadata is described below.

### Importance block stateHash validation

```js
console.log(block);
```

```js
> NormalBlockInfo
    height: UInt64 {lower: 59639, higher: 0}
    hash: "B5F765D388B5381AC93659F501D5C68C00A2EE7DF4548C988E97F809B279839B"
    stateHash: "9D6801C49FE0C31ADE5C1BB71019883378016FA35230B9813CA6BB98F7572758"
  > stateHashSubCacheMerkleRoots: Array(9)
        0: "4578D33DD0ED5B8563440DA88F627BBC95A174C183191C15EE1672C5033E0572"
        1: "2C76DAD84E4830021BE7D4CF661218973BA467741A1FC4663B54B5982053C606"
        2: "259FB9565C546BAD0833AD2B5249AA54FE3BC45C9A0C64101888AC123A156D04"
        3: "58D777F0AA670440D71FA859FB51F8981AF1164474840C71C1BEB4F7801F1B27"
        4: "C9092F0652273166991FA24E8B115ACCBBD39814B8820A94BFBBE3C433E01733"
        5: "4B53B8B0E5EE1EEAD6C1498CCC1D839044B3AE5F85DD8C522A4376C2C92D8324"
        6: "132324AF5536EC9AA85B2C1697F6B357F05EAFC130894B210946567E4D4E9519"
        7: "8374F46FBC759049F73667265394BD47642577F16E0076CBB7B0B9A92AAE0F8E"
        8: "45F6AC48E072992343254F440450EF4E840D8386102AD161B817E9791ABC6F7F"
```

```js
hasher = sha3_256.create();
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[0], "hex")); //AccountState
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[1], "hex")); //Namespace
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[2], "hex")); //Mosaic
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[3], "hex")); //Multisig
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[4], "hex")); //HashLockInfo
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[5], "hex")); //SecretLockInfo
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[6], "hex")); //AccountRestriction
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[7], "hex")); //MosaicRestriction
hasher.update(Buffer.from(block.stateHashSubCacheMerkleRoots[8], "hex")); //Metadata
hash = hasher.hex().toUpperCase();
console.log(block.stateHash === hash);
```

```js
> true
```

It can be seen that the nine states used to validate the block headers consist of stateHashSubCacheMerkleRoots.

## 13.3 Account metadata validation

The Merkle Patricia Tree is used to verify the existence of accounts and metadata associated with a transaction.  
If the service provider provides a Merkle Patricia tree, users can verify its authenticity using nodes of their own choosing.

### Common functions for verification

```js
//Function for obtaining the hash value of a leaf
function getLeafHash(encodedPath, leafValue) {
  const hasher = sha3_256.create();
  return hasher
    .update(sym.Convert.hexToUint8(encodedPath + leafValue))
    .hex()
    .toUpperCase();
}

//Function for obtaining the hash value of a branch
function getBranchHash(encodedPath, links) {
  const branchLinks = Array(16).fill(
    sym.Convert.uint8ToHex(new Uint8Array(32))
  );
  links.forEach((link) => {
    branchLinks[parseInt(`0x${link.bit}`, 16)] = link.link;
  });
  const hasher = sha3_256.create();
  const bHash = hasher
    .update(sym.Convert.hexToUint8(encodedPath + branchLinks.join("")))
    .hex()
    .toUpperCase();
  return bHash;
}

//World State Verification
function checkState(stateProof, stateHash, pathHash, rootHash) {
  const merkleLeaf = stateProof.merkleTree.leaf;
  const merkleBranches = stateProof.merkleTree.branches.reverse();
  const leafHash = getLeafHash(merkleLeaf.encodedPath, stateHash);

  let linkHash = leafHash; //The first linkHash is a leafHash.
  let bit = "";
  for (let i = 0; i < merkleBranches.length; i++) {
    const branch = merkleBranches[i];
    const branchLink = branch.links.find((x) => x.link === linkHash);
    linkHash = getBranchHash(branch.encodedPath, branch.links);
    bit =
      merkleBranches[i].path.slice(0, merkleBranches[i].nibbleCount) +
      branchLink.bit +
      bit;
  }

  const treeRootHash = linkHash; //The last linkHash is the rootHash
  let treePathHash = bit + merkleLeaf.path;

  if (treePathHash.length % 2 == 1) {
    treePathHash = treePathHash.slice(0, -1);
  }

  //verification
  console.log(treeRootHash === rootHash);
  console.log(treePathHash === pathHash);
}
```

### 13.3.1 Account information validation

Account information ia a leaf.
Trace the branches on the Merkle tree by address and confirm whether the route can be reached.

```js
stateProofService = new sym.StateProofService(repo);

aliceAddress = sym.Address.createFromRawAddress(
  "TBIL6D6RURP45YQRWV6Q7YVWIIPLQGLZQFHWFEQ"
);

hasher = sha3_256.create();
alicePathHash = hasher
  .update(sym.RawAddress.stringToAddress(aliceAddress.plain()))
  .hex()
  .toUpperCase();

hasher = sha3_256.create();
aliceInfo = await accountRepo.getAccountInfo(aliceAddress).toPromise();
aliceStateHash = hasher.update(aliceInfo.serialize()).hex().toUpperCase();

//Obtaining up-to-date block header information from non-service provider nodes
blockInfo = await blockRepo.search({ order: "desc" }).toPromise();
rootHash = blockInfo.data[0].stateHashSubCacheMerkleRoots[0];

//Obtaining merkle information from any node, including service providers
stateProof = await stateProofService.accountById(aliceAddress).toPromise();

//Verification
checkState(stateProof, aliceStateHash, alicePathHash, rootHash);
```

### 13.3.2 Verification of metadata registered to the mosaic

Metadata values are registered in the mosaic as a leaf. Trace the branches on the Merkle tree by the hash value consisting of the metadata key, and confirm whether the root can be reached.

```js
srcAddress = Buffer.from(
  sym.Address.createFromRawAddress(
    "TBIL6D6RURP45YQRWV6Q7YVWIIPLQGLZQFHWFEQ"
  ).encoded(),
  "hex"
);

targetAddress = Buffer.from(
  sym.Address.createFromRawAddress(
    "TBIL6D6RURP45YQRWV6Q7YVWIIPLQGLZQFHWFEQ"
  ).encoded(),
  "hex"
);

hasher = sha3_256.create();
hasher.update(srcAddress);
hasher.update(targetAddress);
hasher.update(sym.Convert.hexToUint8Reverse("CF217E116AA422E2")); // scopeKey
hasher.update(sym.Convert.hexToUint8Reverse("1275B0B7511D9161")); // targetId
hasher.update(Uint8Array.from([sym.MetadataType.Mosaic])); // type: Account 0
compositeHash = hasher.hex();

hasher = sha3_256.create();
hasher.update(Buffer.from(compositeHash, "hex"));

pathHash = hasher.hex().toUpperCase();

//stateHash(Value)
hasher = sha3_256.create();
hasher.update(cat.GeneratorUtils.uintToBuffer(1, 2)); //version
hasher.update(srcAddress);
hasher.update(targetAddress);
hasher.update(sym.Convert.hexToUint8Reverse("CF217E116AA422E2")); // scopeKey
hasher.update(sym.Convert.hexToUint8Reverse("1275B0B7511D9161")); // targetId
hasher.update(Uint8Array.from([sym.MetadataType.Mosaic])); //account

value = Buffer.from("test");

hasher.update(cat.GeneratorUtils.uintToBuffer(value.length, 2));
hasher.update(value);
stateHash = hasher.hex();

//Obtaining up-to-date block header information from non-service provider nodes
blockInfo = await blockRepo.search({ order: "desc" }).toPromise();
rootHash = blockInfo.data[0].stateHashSubCacheMerkleRoots[8];

//Obtaining merkle information from any node, including service providers
stateProof = await stateProofService.metadataById(compositeHash).toPromise();

//Verification
checkState(stateProof, stateHash, pathHash, rootHash);
```

### 13.3.3 Verification of metadata registered to an account

Metadata values are registered in the account as a leaf. Trace the branches on the Merkle tree by the hash value consisting of the metadata ke, and confirm whether the root can be reached.

```js
srcAddress = Buffer.from(
  sym.Address.createFromRawAddress(
    "TBIL6D6RURP45YQRWV6Q7YVWIIPLQGLZQFHWFEQ"
  ).encoded(),
  "hex"
);

targetAddress = Buffer.from(
  sym.Address.createFromRawAddress(
    "TBIL6D6RURP45YQRWV6Q7YVWIIPLQGLZQFHWFEQ"
  ).encoded(),
  "hex"
);

//compositePathHash(Key value)
hasher = sha3_256.create();
hasher.update(srcAddress);
hasher.update(targetAddress);
hasher.update(sym.Convert.hexToUint8Reverse("9772B71B058127D7")); // scopeKey
hasher.update(sym.Convert.hexToUint8Reverse("0000000000000000")); // targetId
hasher.update(Uint8Array.from([sym.MetadataType.Account])); // type: Account 0
compositeHash = hasher.hex();

hasher = sha3_256.create();
hasher.update(Buffer.from(compositeHash, "hex"));

pathHash = hasher.hex().toUpperCase();

//stateHash(Value)
hasher = sha3_256.create();
hasher.update(cat.GeneratorUtils.uintToBuffer(1, 2)); //version
hasher.update(srcAddress);
hasher.update(targetAddress);
hasher.update(sym.Convert.hexToUint8Reverse("9772B71B058127D7")); // scopeKey
hasher.update(sym.Convert.hexToUint8Reverse("0000000000000000")); // targetId
hasher.update(Uint8Array.from([sym.MetadataType.Account])); //account
value = Buffer.from("test");
hasher.update(cat.GeneratorUtils.uintToBuffer(value.length, 2));
hasher.update(value);
stateHash = hasher.hex();

//Obtaining up-to-date block header information from non-service provider nodes
blockInfo = await blockRepo.search({ order: "desc" }).toPromise();
rootHash = blockInfo.data[0].stateHashSubCacheMerkleRoots[8];

//Obtaining merkle information from any node, including service providers
stateProof = await stateProofService.metadataById(compositeHash).toPromise();

//Verification
checkState(stateProof, stateHash, pathHash, rootHash);
```

## 13.4 Tips for use

### Trusted web

A simple explanation of the “Trusted Web” is the realisation of a Web where everything is platform-independent and nothing needs to be verified.

What the verification methods in this chapter shows is that all information held by the blockchain can be verified by the hash value of the block header. Blockchains are based on the sharing of block headers that everyone agrees upon and the existence of full nodes that can reproduce them. However, it is challenging to maintain an environment to verify these in every situation where you want to utilise the blockchain.

If the latest block headers are constantly broadcast from multiple trusted institutions, this can greatly reduce the need for verification. Such an infrastructure would allow access to trusted information even in places beyond the capabilities of the blockchain, such as urban areas where tens of millions of people are densely populated, or in remote areas where base stations cannot be adequately deployed, or during wide-area network outages during disasters.

