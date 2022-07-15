# The future of scaling - zkEVM

zkEVM has been the holy grail of L2 and Ethereum scaling and is at the forefront of the blockchain && ethereum. There's a lot of zkp and engineering innovation here, bringing together all the most talented people in the Ethereum ecosystem.

Ethereum's Privacy Scaling Group has been building a native zkEVM that will eventually be integrated in L1, bringing ZKP to the L1 execution layer. This will bring huge scalability.

At present, Hermez and Scroll are also actively exploring this field, and they will become the frontier explorers of zkEVM. The use of zkEVM in L2 will become the testing ground for L1.

From an architectural point of view, the architecture used by Hermez and the Ethereum team is slightly different.

This article will mainly delve into the architecture of the native zkEVM that EF and Scroll are building.

# Three levels of zkEVM

First of all not all zkVM is equivalent to EVM, even for zkEVM itself it is divided into three levels.

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414135281-7dd4e8ae-376d-457b-8be6-74b663b3b12f.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414135281-7dd4e8ae-376d-457b-8be6-74b663b3b12f.png)

**The first level is "language-level" (EVM-compatible)**, that is, transpile an EVM-friendly language (e.g. Solidity or Yul) into a zk-friendly language (e.g. zkSync's Zinc, and Starkware's Cairo).

The later Zinc and Cairo code runs on their own VM, which may be completely different from Ethereum's EVM.

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414202014-2994d418-b438-43f8-83ec-9a65b2eb2866.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414202014-2994d418-b438-43f8-83ec-9a65b2eb2866.png)

The advantage of this solution is that we can design a zk-friendly VM from scratch without being limited by the past design of the EVM.

The disadvantage is that it is difficult for developers to get the best development experience. These zkVMs use the instruction set of their own language at the bottom and do not support many important EVM opcodes

Therefore, if developers want to get the best development experience, they may need to learn these zkVM own languages (Cairo), which may cause zk-Rollup to be unable to directly inherit the Layer1 ecology, and Layer2 developers and languages are separated

**The second level is "bytecode-level" (EVM-equivalent)**, which can not only achieve compatibility at the solidity language level but also achieve full compatibility at the EVM opcode level.

Only when it reaches the bytecode level can it be called "zkEVM". On this zkEVM, solidity developers can get the best development experience, and L1 applications and development tools can basically be migrated to L2 without modification.

The Ethereum Foundation, Scroll, ConsenSys, Hermez are all working towards "bytecode-level".

**The third level is "consensus-level"**, which is also the final zkEVM. It will not only achieve compatibility at the language and bytecode levels but also at the consensus level.

This will be integrated into the Ethereum consensus protocol. zkEVM will be coupled with EVM. The block builder will first calculate in the EVM, output information as input, input the zkEVM circuit, and generate a proof. The verifier finally verifies the proof.

The final verifier only needs to verify a succinct proof, instead of recomputing. At the same time, after achieving consensus layer compatibility, each miner will generate a proof for each block when it generates a block. When all nodes are in sync, they only need to verify that the proof is valid without recomputing all transactions. And based on the recursive proof of Halo2, a proof can be used to prove that the history of the entire block is valid.At that time, the synchronization node does not even need to verify each proof, but only needs to verify the last proof to access the network

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414404817-9453a778-04fb-49f2-92e1-1fe9e0c77756.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414404817-9453a778-04fb-49f2-92e1-1fe9e0c77756.png)

In the long run, it will only take minutes or even seconds to sync an Ethereum node, anyone can easily join the Ethereum network, and Ethereum will become more decentralized and robust.

So the ultimate goal of zkEVM is actually to apply it to L1, replacing our current EVM (Very ambitious!) This is also the ultimate goal of Ethereum Foundation, Scroll and all of us working together

For details, see the last part of the ethereum roadmap sent by VitalikButerin - "zk-snark everything" I believe VitalikButerin and Privacy Scaling Group will also explore stack solutions

[https://twitter.com/VitalikButerin/status/1466411377107558402](https://twitter.com/VitalikButerin/status/1466411377107558402)

# Zero-Knowledge proof

When the Ethereum team originally designed the EVM, they did not consider compatibility with zk-snark. Therefore, if the zero-knowledge circuit is directly applied to many places in the EVM, it will cause huge overhead, especially Keccak hash and MPT.

What makes zkEVM around the corner There are many cryptographic breakthroughs behind it that can make zkevm from imagination to reality, the most important of which is the Plonk and Halo2

For details, can see this thread by the founder of Plonk and AztecNetwork

[https://twitter.com/Zac_Aztec/status/1440295503938215947](https://twitter.com/Zac_Aztec/status/1440295503938215947)

Plonk is an innovation based on Sonic and polynomial commitment Based on sonic, Plonk has a "universal and updateable" trusted setup, that is, only one setup is required, and then it can be reused

And based on polynomial commitment (very beautiful math), we can use more expressive PLONKish Arithmetization, better than R1CS which is widely used by groth16 and other zk-snark proof schemes[https://www.youtube.com/watch?v=bz16BURH_u8](https://www.youtube.com/watch?v=bz16BURH_u8)

zkEVM also uses two very important features of Plonkish, namely "custom gate" and "lookup table argument"

These two features of Plonk (halo2 inherited) allow us to write highly customized constraints, which are very helpful to reduce the overhead of the circuit (you will find that we frequently use these two features in the native zkEVM architecture later)

[https://www.youtube.com/watch?v=Vdlc1CmRYRY](https://www.youtube.com/watch?v=Vdlc1CmRYRY)

# Architecture of native zkevm

As we mentioned earlier, the native zkEVM will not only be used in the zk-Rollup but also will replace our current L1 EVM and become the L1 zkEVM So its design/code and architecture are very worth learning (the most cutting-edge innovation!)

The well-known EVM is essentially a state machine, which drives state1 to state2 through transactions So it can be understood that the operation that drives the smallest state change is a transaction (actually trace)

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414834273-104f613e-6b2e-4b3b-aa0e-6ede2756e251.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414834273-104f613e-6b2e-4b3b-aa0e-6ede2756e251.png)

If we can get transactions and constrain/prove them, in fact, it can constrain/prove the entire state machine

The basic idea of zkEVM is to create an evm circuit to constrain the EVM (the state machine) and prove that all the execution logic of the EVM is correct

This EVM circuit can get all transactions, and each specific opcode called by this transaction Then prove that each transaction, as well as all opcodes called by each transaction, the operation logic of opcodes, and even the sequence of operations, are completely correct

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414880457-ed80c4c3-a682-4485-8e72-1085670e2609.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414880457-ed80c4c3-a682-4485-8e72-1085670e2609.png)

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414890224-b7086b82-0108-4a74-9ed9-d4485849aad1.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414890224-b7086b82-0108-4a74-9ed9-d4485849aad1.png)

But in our practice, we found that if only one circuit (EVM circuit) is used to constrain the EVM, this circuit will become very huge, and finally it will increase unnecessary complexity and overhead

So we designed many different sub-circuits/tables according to different modules in EVM. When proving, we only need to query the corresponding table (a table probably looks like this, fill in different variables according to requirements)

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414950116-4d931ff8-65d4-4b02-b304-d758af3beb09.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414950116-4d931ff8-65d4-4b02-b304-d758af3beb09.png)

For example, if it is the logic of memory/stack/stoarge read & write, the EVM circuit will query the state table If it is some operation involving opcode, the EVM circuit will query the bytecode table. Similarly, tx and block will query tx table and block table respectively

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414973167-4c4c3922-140e-4469-90d8-05337d7009a7.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656414973167-4c4c3922-140e-4469-90d8-05337d7009a7.png)

Here, the state circuit needs to operate MPT when constraining storage-related operations (state table), so the corresponding MPT table is queried The Tx circuit also needs to query the corresponding Keccak and Sig table when calculating the hash and transaction sig verification

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656415000381-4d24cad0-1cd2-4ba7-b0ca-d95aa4f44a07.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656415000381-4d24cad0-1cd2-4ba7-b0ca-d95aa4f44a07.png)

This table is not fixed, but filled in with different values according to different operations (this is one of the reasons why zkEVM can become universal) So prover has the ability to fill in false values to forge an invalid table

Therefore, in order to ensure the correctness of the table, we design a circuit for each table, and each circuit has some special polynomial constraints on the table to ensure that the table is completely correct

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656415038620-3ac33846-1e92-4e0f-b522-41f820c981f6.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656415038620-3ac33846-1e92-4e0f-b522-41f820c981f6.png)

When a transaction/trace enters the EVM circuit, all operations (opcode, stack/storage, etc.) involved in it will be reordered and then assigned to different sub-circuits These sub-circuits will prove the correctness of these operations and generate a proof

Finally, the proofs generated by these sub-circuits will be input into an aggregation circuit as public input, and the aggregation circuit will aggregate these single proofs into an aggregate proof

![https://cdn.nlark.com/yuque/0/2022/png/25416421/1656415070575-a57de8ae-5799-4ea7-b6c6-97d047441db6.png](https://cdn.nlark.com/yuque/0/2022/png/25416421/1656415070575-a57de8ae-5799-4ea7-b6c6-97d047441db6.png)

Ending and the beginning zkEVM is a milestone of "zk everything" and innovation that can only appear after the practical zk-proving systems are mature

While researching zkEVM, I was deeply impressed by the mathematical mechanism behind it, I believe zk is a huge innovation and we are at the forefront of this innovation
