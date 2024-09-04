# FirstCircuit on Sepolia Testnet 🐱

## Introduction

This project involves the deployment of a custom circuit written in Circom on the Ethereum Sepolia testnet. The circuit includes basic logic gates (AND, OR, NOT) and combines them to form a simple logical operation. The primary functionality of the circuit is encapsulated in the `FirstCircuit` template.

## Circuit Description

### Templates
```
pragma circom 2.0.0;


template FirstCircuit () {  
   // input signals
   signal input a;
   signal input b;

   // Internal input signals

   signal x;
   signal y;

   // output signals
   signal output Q;

   // component
   component andGate = AND();
   component orGate = OR();
   component notGate = NOT();

   // logic

   andGate.a <== a;
   andGate.b <== b;
   x <== andGate.y;

   notGate.in <== b;
   y <== notGate.out;

   orGate.a <== x;
   orGate.b <== y;
   Q <== orGate.y;
}

template AND(){
   signal input a;
   signal input b;
   signal output y;
   y <== a*b;
}

template OR(){
   signal input a;
   signal input b;
   signal output y;
   y <== a + b - a*b;
}

template NOT() {
    signal input in;
    signal output out;

    out <== 1 + in - 2*in;
}

component main = FirstCircuit();
```

### Install
`npm i`

### Compile
`npx hardhat circom` 
This will generate the **out** file with circuit intermediaries and geneate the **MultiplierVerifier.sol** contract

### Prove and Deploy
`npx hardhat run scripts/deploy.ts --network sepolia`
This script does 4 things  
1. Deploys the MultiplierVerifier.sol contract
2. Generates a proof from circuit intermediaries with `generateProof()`
3. Generates calldata with `generateCallData()`
4. Calls `verifyProof()` on the verifier contract with calldata

With two commands you can compile a ZKP, generate a proof, deploy a verifier, and verify the proof 🎉

## Configuration
### Directory Structure
**circuits**
```
├── multiplier
│   ├── circuit.circom
│   ├── input.json
│   └── out
│       ├── circuit.wasm
│       ├── multiplier.r1cs
│       ├── multiplier.vkey
│       └── multiplier.zkey
├── new-circuit
└── powersOfTau28_hez_final_12.ptau
```
Each new circuit lives in it's own directory. At the top level of each circuit directory lives the circom circuit and input to the circuit.
The **out** directory will be autogenerated and store the compiled outputs, keys, and proofs. The Powers of Tau file comes from the Polygon Hermez ceremony, which saves time by not needing a new ceremony. 


**contracts**
```
contracts
└── MultiplierVerifier.sol
```
Verifier contracts are autogenerated and prefixed by the circuit name, in this example **Multiplier**

## hardhat.config.ts
```
  circom: {
    // (optional) Base path for input files, defaults to `./circuits/`
    inputBasePath: "./circuits",
    // (required) The final ptau file, relative to inputBasePath, from a Phase 1 ceremony
    ptau: "powersOfTau28_hez_final_12.ptau",
    // (required) Each object in this array refers to a separate circuit
    circuits: JSON.parse(JSON.stringify(circuits))
  },
```
### circuits.config.json
circuits configuation is separated from hardhat.config.ts for **autogenerated** purposes (see next section)
```
[
  {
    "name": "multiplier",
    "protocol": "groth16",
    "circuit": "multiplier/circuit.circom",
    "input": "multiplier/input.json",
    "wasm": "multiplier/out/circuit.wasm",
    "zkey": "multiplier/out/multiplier.zkey",
    "vkey": "multiplier/out/multiplier.vkey",
    "r1cs": "multiplier/out/multiplier.r1cs",
    "beacon": "0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f"
  }
]
```

**adding circuits**   
To add a new circuit, you can run the `newcircuit` hardhat task to autogenerate configuration and directories i.e  
```
npx hardhat newcircuit --name newcircuit
```
## Contact 
For any questions or inquiries, feel free to reach out via Mail(harshdeepsingh2809@gmail.com) or open an issue in the GitHub repository.

