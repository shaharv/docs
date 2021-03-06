# Glow Compiler Overview (Slides)

## The Glow Compiler

- Glow is a retargetable neural network compiler, open sourced by Facebook in 2018

- Glow stands for “Graph LOWering”

- At the heart of Glow are two IR levels – the High Level IR (Graph) and the Low Level IR (Instructions) - allowing for high level graph optimizations and low level instruction based optimizations, respectively

- Target specific optimizations are performed by the respective Backend (which is free to introduce additional IR levels)
  - A Glow backend takes either Graph IR or Low Level IR, depending on the backend implementation
  - Code generation is done by an LLVM backend (CPU JIT LLVM backend is bundled)

- Supports JIT and AOT compilation modes, in addition to the Glow interpreter

- Supports loading ONNX, pytorch and Caffe2 models (through ONNX conversion)

## Glow Intermediate Representations

- Similar to traditional C/C++ compilers, Glow introduces several IR levels

- Both IR forms are strongly typed – tensor dimensions and types provided at compile time

- High Level IR – dataflow node-based graph representation:
  - Classic optimizations (CSE, DCE)
  - NN and Linear Algebra optimizations (merge concat/transpose nodes, identities to reduce tensor dims)
  - Quantization

- Low Level IR – Instruction based representation:
  - Memory optimizations (DSE and dead allocation removal, shorten allocation life ranges, buffer sharing)
  - Peephole optimizations

## Glow High Level IR

- Structured as a module containing functions, each containing multiple nodes

- Operator nodes – Convolution, MatrixMultiply, MaxPool, etc.

- Storage nodes:
  - Constant nodes – concrete compile time known tensors (e.g. weights)
  - Placeholder nodes – tensor variables

## Glow High Level IR Life Cycle

- Constructed by translating each node of the input model
  - Each operator to one or more graph nodes
  - Some ONNX nodes (e.g. GEMM) are lowered during translation
  
- The generated IR is optimized by the [Graph Optimizer](https://github.com/pytorch/glow/blob/master/docs/GraphOptimizationPipeline.md)

- Then, Linear Algebra lowering is performed:
  - Saves the need to implement every single operator
  - Glow backends can exclude or defer operators from being lowered (shouldLower, transformPostLowering methods)

- Then, additional graph optimizations are performed

## Glow Low Level IR

- Generated by the IRGen phase

- Each IR function consists of two sections:
  - `declare` – global memory regions
  - `program` – list of instructions

- Operands are annotated with `@in`, `@out`, `@inout`

- Function local memory is denoted by alloc

- Instructions could have additional attributes

- Following IRGen, the LL IR is optimized as well

## Code Generation

- The Glow backend receives the low level IR (or, the graph IR, for some backends)

- Concrete memory buffers are allocated

- The IR is compiled into LLVM-IR

- Optimized by the LLVM optimizer
  - Powerful vectorization
  - IPO - Linked with standard library LLVM bitcode

- Target code is generated

## IR/CG Life Cycle

![Glow IR Levels](https://github.com/shaharv/glow/blob/master/docs/3LevelIR.png)

## Glow Runtime

- Supports partitioning, queueing and running models across multiple devices

![Glow Runtime](https://github.com/shaharv/glow/blob/master/docs/glow_runtime.svg)

## Links

- Glow: Graph Lowering Compiler Techniques for Neural Networks  
  https://arxiv.org/pdf/1805.00907.pdf

- Glow talk at 2018 LLVM Developers’ Meeting  
  https://www.youtube.com/watch?v=cTz7c5dn5Gc

# Glow Internals

## model-compiler

- The `model-compiler` is one of several wrappers around the `Loader` tool. It takes an .onnx model and compiles it to machine code with the desired backend.

- Flow:
  - [ModelCompiler.cpp](https://github.com/pytorch/glow/blob/master/tools/loader/ModelCompiler.cpp) `main` function
  - Loader object is initialized
  - Model is loaded: `loader.loadModel()`
    - `ONNXModelLoader` is instantiated - see [ONNXModelLoader.cpp](https://github.com/pytorch/glow/blob/master/lib/Importer/ONNXModelLoader.cpp):  
      `new ONNXModelLoader(getOnnxModelFilename(), inputNameRefs, inputTypeRefs, *getFunction()))`
  - Model is compiled with default options: `loader.compile(cctx)`
    - Dump DAG (graph IR) - optional
    - Optimizes the graph IR: `glow::optimizeFunction(F_, *backend_, cctx)`  
    - Calls `backend_->save` for saving the bundle  
      - In `save`, DAG is transformed to IR: `generateAndOptimizeIR(F, *this /*backend*/, shouldShareBuffers())`  
    - Dump the optimized DAG - optional

## ClassGen

- The code for graph nodes and IR instructions is mostly generated automatically in the [ClassGen](https://github.com/pytorch/glow/tree/master/tools/ClassGen) project.  
- Code is generated during build and written to the build folder, under `glow`.
- `NodeGen` executable:
  - Used for automatic code generation for nodes.
  - Generates required boilerplate code: methods (getters and setters, visit, isEqual, clone, verify), definitions, etc.
  - The actual node creations functions, `createNodeName`, are in [Graph.cpp](https://github.com/pytorch/glow/blob/master/lib/Graph/Graph.cpp), and are called from ONNXModelLoader during ONNX model traversal.
  - In [`NodeGen.cpp`](https://github.com/pytorch/glow/blob/master/tools/ClassGen/NodeGen.cpp), the nodes are defined, each by calling `BB.newNode("NodeName")`, where BB is a builder object. There are currently 89 node types (not including backend specific ones).  
  - Running manually:  
    `$GLOW_BIN/NodeGen ./AutoGenNodes.h ./AutoGenNodes.cpp ./AutoGenNodes.def ./AutoGenNodesImport.h ./AutoGenNodesExport.h`
- `InstrGen` executable:  
  - Used for automatic code generation for instructions.
  - Generates instructions boilerplate code. As opposed to `NodeGen`, it also generates the actual IR creation functions, which are defined in `AutoGenIRBuilder.cpp`.
  - In [`InstrGen.cpp`](https://github.com/pytorch/glow/blob/master/tools/ClassGen/InstrGen.cpp), the instructions are defined, each by calling `BB.newInstr("InstrName")`. There are currently 79 instruction types (not including backend specific ones).  
  - Running manually:  
    `$GLOW_BIN/InstrGen ./AutoGenInstr.h ./AutoGenInstr.cpp ./AutoGenInstr.def ./AutoGenIRBuilder.h ./AutoGenIRBuilder.cpp ./AutoGenIRGen.h`
