# 3.1 Hello World

We are finally ready to create our first Aleo program. Let's see how to proceed.

The Leo command-line tool offers a simple way to start a new project from scratch:

```bash
leo new hello_world
```

This will create a new program project named `hello_world`. This project is instantiated in a new directory. Let's start by moving into this directory:

```bash
cd hello_world
```

You should have the following file structure:

```
hello_world/
├── .gitignore
├── .env
├── program.json
└── src/
  └── main.leo
```

`.gitignore` is just a default file related to git. The environment file `.env` contains the `NETWORK` and `PRIVATE_KEY` variables. Let's keep the default generated values for now.

The `program.json` file is a manifest containing all the metadata for your project:

```json
{
  "program": "hello_world.aleo",
  "version": "0.1.0",
  "description": "",
  "license": "MIT",
  "dependencies": null
}
```

No need to modify this information for now.

The `src/main.leo` file contains the source code for a "hello world" example program:

```rust
// The 'hello_world' program.
program hello_world.aleo {
    transition main(public a: u32, b: u32) -> u32 {
        let c: u32 = a + b;
        return c;
    }
}
```

Leo's syntax looks very similar to Rust's, if you are familiar with that language. Let's break down this first program line by line to try to understand what it does:

```rust
// The 'hello_world' program.
```

This line is a comment, you can just ignore it. Like in Rust, single-line comments start with `//` and multi-line comments start with `/*` and end with `*/`.

```rust
program hello_world.aleo {
```

All the code for your program will be included within this program structure. `hello_world.aleo` is the identifier for your program; it's a unique identifier for your program on the blockchain. Once deployed, you will be able to call your program using this identifier. If a program with this identifier is already deployed, you will need to deploy your program with a different identifier, updating the value here, as well as in the `program` field of the `program.json` file.

```rust
transition main(public a: u32, b: u32) -> u32 {
```

This is where things get interesting. A transition is a function within your program that can be called by users and other programs, directly on the blockchain. Using its program identifier and name, any user can call a transition in a program, for example, using the snarkOS command-line interface.

Here, the name of the transition is `main`, but it could be anything else.

A transition has inputs and outputs. These inputs must be provided by the caller of the transition. Here, the inputs are defined as `public a: u32, b: u32`, there are two inputs `a` and `b`. They are both 32-bit unsigned integers, but `a` is public while `b` is private; we will see exactly what this means in the next chapter. The output is a single 32-bit unsigned integer, as defined by: `-> u32`. The next line is:

```rust
let c: u32 = a + b;
```

Here, we simply define a new variable `c` which is the sum of `a` and `b`.

All statements in Leo must end with a semicolon, just like in C or Rust.

```rust
return c;
```

The sum is then returned as the output in the form of a 32-bit unsigned integer.

There you have your first Leo program, although it's not very useful yet, of course.

Now, let's try to execute the `main` transition (locally off-chain of course, your program is not yet deployed on the blockchain). We will use `a = 1u32` and `b = 2u32` as test values:

```bash
leo run main 1u32 2u32
```

Here is what the console should display:

```
       Leo ✅ Compiled 'hello_world.aleo' into Aleo instructions

⛓  Constraints

 •  'hello_world.aleo/main' - 33 constraints (called 1 time)

➡️  Output

 • 3u32

       Leo ✅ Finished 'hello_world.aleo/main' (in "/path/to/hello_world/build")
```
*(Note: The exact path in the output will vary)*

This command performs several steps in the background. First, it transcribes your high-level Leo code into Aleo instructions. This is the code that can be deployed on the blockchain as a program. Aleo instructions can then be compiled into AVM bytecode, a lower-level language that is not human-readable, but can be executed by the Aleo zkVM. To draw an analogy with traditional programming, you can think of Leo as Rust or C, Aleo instructions as assembly, and AVM bytecodes as machine code.

When transforming Aleo instructions into AVM bytecode, the compiler transforms each instruction of each function into an R1CS circuit representation. In the command output shown earlier, the number of constraints in the circuit for the main function is displayed in the "Constraint" section: here, it's 33. The higher the constraints, the longer it will take to prove a transition.

In the output section, the program's output is displayed: here `c` from our previous program.

If you actually want to generate a proof for the execution of a function on certain inputs, you should use the `execute` command instead:

```bash
leo execute main 1u32 2u32
```

In addition to the information displayed previously, this should produce a JSON output. This JSON contains all the information necessary to verify your execution of the function. It includes the Proof (at the "proof" key), but no knowledge about the private input and outputs of the said function. You could send this JSON to any verifier. Once your program is deployed, this is what users will broadcast to the blockchain nodes so that the function can be verified and executed on-chain.

After executing this command, if you list your files again, you should see something like:

```
hello_world/
├── .gitignore
├── .env
├── leo.lock
├── program.json
├── build/
│   ├── build/             <-- (Note: Leo structure might slightly change, this inner 'build' might not exist)
│   │   ├── main.avm
│   │   ├── main.prover
│   │   └── main.verifier
│   ├── program.json       <-- (Note: This might be at the outer 'build' level)
│   └── main.aleo
└── src/
    └── main.leo
```
*(File structure might vary slightly depending on the Leo version)*

`leo.lock` keeps track of the resolved dependencies for your program, we will come back to this in the next module. `main.aleo` is the transcribed file containing Aleo instructions, while `main.avm` is the compiled file containing AVM bytecode. The prover and verifier files contain serialized Proving and Verifying keys for the corresponding program. They can be used respectively to generate and verify proofs of executions.

In the next chapter, we will see what the private and public keywords mean. 