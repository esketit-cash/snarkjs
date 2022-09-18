# snarkjs: JavaScript implementation of zkSNARKs.

This is a JavaScript implementation of zkSNARK schemes. It allows the original 8points protocol
and the Groth Protocol (3 point only and 3 pairings)

This library allows to do the trusted setup, generate proofs and verify the proofs.

This library uses the compiled circuits generated by the jaz compiler.

### Tutorial.

A good starting point [is this tutorial](https://github.com/iden3/circom/blob/master/TUTORIAL.md)

Also this [video](https://www.youtube.com/watch?v=-9TJa1hVsKA) is a good starting point.

## Install.

```sh
npm install snarkjs
```

## Usage from command line.


```sh
snarkjs --help
```

Will show all the info in how to use the cli.

## Usage from javascript


### Import.

```js
const zkSnark = require("snarkjs");
```

### Load a circuit.

```js
// "myCircuit.cir" is the output of the jaz compiler

const circuitDef = JSON.parse(fs.readFileSync("myCircuit.cir", "utf8"));
const circuit = new zkSnark.Circuit(circuitDef);
```

### Inspect the circuit.

```js
    // `signalId` can always be a number or an alias string

    circuit.nConstraints; // number of constraints
    circuit.nSignals; // number of signals
    circuit.nPublic; // number of public signals (nOutputs + nPublicInputs)

    // The array of signals is always sorted in this order:
    // [ 1, outputs, publicInputs, privateInputs, internalSignals, constants]

    // returns a,b and c coeficients of the `signalId` on a given `constraint`
    circuit.a(constraint, signalId)
    circuit.b(constraint, signalId)
    circuit.c(constraint, signalId)

    circuit.nOutputs           // number of public outputs
    circuit.pubInputs          // number of public inputs
    circuit.nPrvInputs         // number of private inputs
    circuit.nInputs            // number of inputs ( nPublicInputs + nPrivateInputs)
    circuit.nVars              // number of variables ( not including constants (one is a variable) )
    circuit.nSignals           // number of signals ( including constants )

    circuit.outputIdx(i)       // returns the index of the i'th output
    circuit.inputIdx(i)        // returns the index of the i'th input
    circuit.pubInputIdx(i)     // returns the index of the i'th public input
    circuit.prvInputIdx(i)     // returns the index of the i'th private input
    circuit.varIdx(i)          // returns the index of the i'th variable
    circuit.constantIdx(i)     // returns the index of the i'th constant
    circuit.signalIdx(i)       // returns the index of the i'th signal

    // returns signal Idx given a signalId
    // if the idx >= n , it is a constant
    // if the idx == -1, the signal does not exist
    circuit.getSignalIdx(name);

    // returns an array aliases names of the i'th signal
    circuit.signalNames(i)

    // input is a key value object where keys are the signal names
    //   of all the inputs (public and private)
    // returns an array of values representing the witness
    circuit.calculateWitness(input)
```

### Trusted setup.

```js
const setup = zkSnark.setup(circuit);
fs.writeFileSync("myCircuit.vk_proof", JSON.stringify(setup.vk_proof), "utf8");
fs.writeFileSync("myCircuit.vk_verifier", JSON.stringify(setup.vk_verifier), "utf8");
setup.toxic  // Must be discarded.
```

### Generate proof.

```js
const circuitDef = JSON.parse(fs.readFileSync("myCircuit.cir", "utf8"));
const circuit = new zkSnark.Circuit(circuitDef);
const input = {
    "main.pubIn1": "123",
    "main.out1": "456"
}
const witness = circuit.calculateWitness(input);
const vk_proof = JSON.parse(fs.readFileSync("myCircuit.vk_proof", "utf8"));

const {proof, publicSignals} = zkSnark.genProof(vk_proof, witness);
```

### Verifier.

```js
const vk_verifier = JSON.parse(fs.readFileSync("myCircuit.vk_verifier", "utf8"));

if (zkSnark.isValid(vk_verifier, proof, publicSignals)) {
    console.log("The proof is valid");
} else {
    console.log("The proof is not valid");
}
```

## License

snarkjs is part of the iden3 project copyright 2018 0KIMS association and published with GPL-3 license. Please check the COPYING file for more details.
