# candle
[![discord server](https://dcbadge.vercel.app/api/server/hugging-face-879548962464493619)](https://discord.com/channels/879548962464493619/1136218819447238726)
[![Latest version](https://img.shields.io/crates/v/candle-core.svg)](https://crates.io/crates/candle-core)
[![Documentation](https://docs.rs/candle-core/badge.svg)](https://docs.rs/candle-core)
![License](https://img.shields.io/crates/l/candle-core.svg)

Candle is a minimalist ML framework for Rust with a focus on performance (including GPU support) 
and ease of use. Try our online demos: 
[whisper](https://huggingface.co/spaces/lmz/candle-whisper),
[llama2](https://huggingface.co/spaces/lmz/candle-llama2).

```rust
let a = Tensor::randn(0f32, 1., (2, 3), &Device::Cpu)?;
let b = Tensor::randn(0f32, 1., (3, 4), &Device::Cpu)?;

let c = a.matmul(&b)?;
println!("{c}");
```

## Check out our examples

Check out our [examples](./candle-examples/examples/):

- [Whisper](./candle-examples/examples/whisper/): speech recognition model.
- [Llama and Llama-v2](./candle-examples/examples/llama/): general LLM.
- [Falcon](./candle-examples/examples/falcon/): general LLM.
- [Bert](./candle-examples/examples/bert/): useful for sentence embeddings.
- [StarCoder](./candle-examples/examples/bigcode/): LLM specialized to code
  generation.
- [Stable Diffusion](./candle-examples/examples/stable-diffusion/): text to
  image generative model, only cpu support at the moment and on the slow side.

Run them using the following commands:
```
cargo run --example whisper --release
cargo run --example llama --release
cargo run --example falcon --release
cargo run --example bert --release
cargo run --example bigcode --release
cargo run --example stable-diffusion --release --features image -- --prompt "a rusty robot holding a fire torch"
```

In order to use **CUDA** add `--features cuda` to the example command line.

There are also some wasm examples for whisper and
[llama2.c](https://github.com/karpathy/llama2.c). You can either build them with
`trunk` or try them online:
[whisper](https://huggingface.co/spaces/lmz/candle-whisper),
[llama2](https://huggingface.co/spaces/lmz/candle-llama2).

For llama2, run the following command to retrieve the weight files and start a
test server:
```bash
cd candle-wasm-examples/llama2-c
wget https://huggingface.co/spaces/lmz/candle-llama2/resolve/main/model.bin
wget https://huggingface.co/spaces/lmz/candle-llama2/resolve/main/tokenizer.json
trunk serve --release --public-url /candle-llama2/ --port 8081
```
And then head over to
[http://localhost:8081/candle-llama2](http://localhost:8081/candle-llama2).

<!--- ANCHOR: features --->

## Features

- Simple syntax, looks and feels like PyTorch.
- CPU and Cuda backends, m1, f16, bf16.
- Serverless (on CPU), small and fast deployments
- WASM support, run your models in a browser.
- Model training.
- Distributed computing using NCCL.
- Model support out of the box: Llama, Whisper, Falcon, StarCoder...
- Embed user-defined ops/kernels, such as [flash-attention
  v2](https://github.com/huggingface/candle/blob/89ba005962495f2bfbda286e185e9c3c7f5300a3/candle-flash-attn/src/lib.rs#L152).

<!--- ANCHOR_END: features --->

## How to use

<!--- ANCHOR: cheatsheet --->
Cheatsheet:

|            | Using PyTorch                            | Using Candle                                                     |
|------------|------------------------------------------|------------------------------------------------------------------|
| Creation   | `torch.Tensor([[1, 2], [3, 4]])`         | `Tensor::new(&[[1f32, 2.], [3., 4.]], &Device::Cpu)?`           |
| Creation   | `torch.zeros((2, 2))`                    | `Tensor::zeros((2, 2), DType::F32, &Device::Cpu)?`               |
| Indexing   | `tensor[:, :4]`                          | `tensor.i((.., ..4))?`                                           |
| Operations | `tensor.view((2, 2))`                    | `tensor.reshape((2, 2))?`                                        |
| Operations | `a.matmul(b)`                            | `a.matmul(&b)?`                                                  |
| Arithmetic | `a + b`                                  | `&a + &b`                                                        |
| Device     | `tensor.to(device="cuda")`               | `tensor.to_device(&Device::Cuda(0))?`                            |
| Dtype      | `tensor.to(dtype=torch.float16)`         | `tensor.to_dtype(&DType::F16)?`                                  |
| Saving     | `torch.save({"A": A}, "model.bin")`      | `candle::safetensors::save(&HashMap::from([("A", A)]), "model.safetensors")?` |
| Loading    | `weights = torch.load("model.bin")`      | `candle::safetensors::load("model.safetensors", &device)`        |

<!--- ANCHOR_END: cheatsheet --->


## Structure

- [candle-core](./candle-core): Core ops, devices, and `Tensor` struct definition
- [candle-nn](./candle-nn/): Tools to build real models
- [candle-examples](./candle-examples/): Examples of using the library in realistic settings
- [candle-kernels](./candle-kernels/): CUDA custom kernels
- [candle-datasets](./candle-datasets/): Datasets and data loaders.
- [candle-transformers](./candle-transformers): transformers-related utilities.
- [candle-flash-attn](./candle-flash-attn): Flash attention v2 layer.

## FAQ

### Why should I use Candle?

Candle's core goal is to *make serverless inference possible*. Full machine learning frameworks like PyTorch
are very large, which makes creating instances on a cluster slow. Candle allows deployment of lightweight
binaries.

Secondly, Candle lets you *remove Python* from production workloads. Python overhead can seriously hurt performance,
and the [GIL](https://www.backblaze.com/blog/the-python-gil-past-present-and-future/) is a notorious source of headaches.

Finally, Rust is cool! A lot of the HF ecosystem already has Rust crates, like [safetensors](https://github.com/huggingface/safetensors) and [tokenizers](https://github.com/huggingface/tokenizers).


### Other ML frameworks

- [dfdx](https://github.com/coreylowman/dfdx) is a formidable crate, with shapes being included
  in types. This prevents a lot of headaches by getting the compiler to complain about shape mismatches right off the bat.
  However, we found that some features still require nightly, and writing code can be a bit daunting for non rust experts.

  We're leveraging and contributing to other core crates for the runtime so hopefully both crates can benefit from each
  other.

- [burn](https://github.com/burn-rs/burn) is a general crate that can leverage multiple backends so you can choose the best
  engine for your workload.

- [tch-rs](https://github.com/LaurentMazare/tch-rs.git) Bindings to the torch library in Rust. Extremely versatile, but they 
  bring in the entire torch library into the runtime. The main contributor of `tch-rs` is also involved in the development
  of `candle`.

### Common Errors

#### Missing symbols when compiling with the mkl feature.

If you get some missing symbols when compiling binaries/tests using the mkl
features, e.g.:
```
  = note: /usr/bin/ld: (....o): in function `blas::sgemm':
          .../blas-0.22.0/src/lib.rs:1944: undefined reference to `sgemm_' collect2: error: ld returned 1 exit status

  = note: some `extern` functions couldn't be found; some native libraries may need to be installed or have their path specified
  = note: use the `-l` flag to specify native libraries to link
  = note: use the `cargo:rustc-link-lib` directive to specify the native libraries to link with Cargo (see https://doc.rust-lang.org/cargo/reference/build-scripts.html#cargorustc-link-libkindname)
```

This is likely due to a missing linker flag that was needed to enable the mkl library. You
can try adding the following at the top of your binary:
```
extern crate intel_mkl_src;
```

#### Cannot run llama example : access to source requires login credentials

```
Error: request error: https://huggingface.co/meta-llama/Llama-2-7b-hf/resolve/main/tokenizer.json: status code 401
```

This is likely because you're not permissioned for the llama-v2 model. To fix
this, you have to register on the huggingface-hub, accept the [llama-v2 model
conditions](https://huggingface.co/meta-llama/Llama-2-7b-hf), and set up your
authentication token. See issue
[#350](https://github.com/huggingface/candle/issues/350) for more details.

#### Tracking down errors

You can set `RUST_BACKTRACE=1` to be provided with backtraces when a candle
error is generated.
