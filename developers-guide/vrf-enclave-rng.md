### A VRF+Enclave Random Number Generator for smartBCH

The dev team of smartBCH implements and maintains the service of a manipulation-proof random number generator. As we have [described](https://read.cash/@SmartBCH/light-weighted-manipulation-proof-random-number-generator-for-smartbch-79ac7d36), it is based on the VRF+Enclave mechanism. This document describes how to get the random numbers (VRF outputs of block hashes) from the RPC server, and how to verify them with a governance contract.

On the RPC server, two processes are running: an enclaved process keeping the VRF private key and a non-enclave proxy process. The proxy process inputs block hashes to the enclaved process and then caches its outputs. The proxy process can access larger DRAM space, so it interacts with the clients. And the attestation requests sent to the enclaved process are forwarded by the proxy process.

The proxy process has the following RPC endpoints:

#### `/vrf?b=<blockhash>`

Given a block hash, it returns the corresponding VRF output, i.e., the manipulation-proof random number, as well as the output's proof.

#### `/pubkey`

The corresponding public key of the VRF private key kept by the enclave.

#### `/report`

The attestation report generated by the enclave

#### `/token`

A JWT token issued by Azure which endorses the enclave's attestation report



 The governance contract provides to functions for DApp developers:

```solidity
function pubKey() public returns (bytes);
function verify(uint blockHash, uint rdm, bytes calldata pi) public view returns (bool);
```

The `pubKey` function returns the endorsed VRF public key. A quorum of operators checks the `/pubkey` RPC's result and endorses the pubkey store in this governance contract.

The `verify` function takes the `blockHash` input, the VRF output `rdm`, and the proof `pi`, and returns whether `rdm` is valid  according to `pi`.

