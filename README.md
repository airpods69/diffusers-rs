# diffusers-rs: A Diffusers API in Rust/Torch

[![Build Status](https://github.com/LaurentMazare/diffusers-rs/workflows/Continuous%20integration/badge.svg)](https://github.com/LaurentMazare/diffusers-rs/actions)
[![Latest version](https://img.shields.io/crates/v/diffusers.svg)](https://crates.io/crates/diffusers)
[![Documentation](https://docs.rs/diffusers/badge.svg)](https://docs.rs/diffusers)
![License](https://img.shields.io/crates/l/diffusers.svg)

![rusty robot holding a torch](media/robot13.jpg)

_A rusty robot holding a fire torch_, generated by stable diffusion using Rust and libtorch.

The `diffusers` crate is a Rust equivalent to Huggingface's amazing
[diffusers](https://github.com/huggingface/diffusers) Python library.
It is based on the [tch crate](https://github.com/LaurentMazare/tch-rs/).
The implementation supports running Stable Diffusion v1.5 and v2.1.

## Getting the weights

The weight files can be retrieved from the HuggingFace model repos and should be
moved in the `data/` directory.
- For Stable Diffusion v2.1, get the `bpe_simple_vocab_16e6.txt`,
  `clip_v2.1.safetensors`, `unet_v2.1.safetensors`, and `vae_v2.1.safetensors`
  files from the
  [v2.1 repo](https://huggingface.co/lmz/rust-stable-diffusion-v2-1/tree/main/weights).
- For Stable Diffusion v1.5, get the `bpe_simple_vocab_16e6.txt`,
  `pytorch_model.safetensors`, `unet.safetensors`, and `vae.safetensors`
  files from this
  [v1.5 repo](https://huggingface.co/lmz/rust-stable-diffusion-v1-5/tree/main/weights).
- Alternatively, you can run the following python script.
```bash
# Add --sd_version 1.5 to get the v1.5 weights rather than the v2.1.
python3 ./scripts/get_weights.py
```

## Running some example.

```bash
cargo run --example stable-diffusion --features clap -- --prompt "A rusty robot holding a fire torch."
```

The final image is named `sd_final.png` by default.
The default scheduler is the Denoising Diffusion Implicit Model scheduler (DDIM). The
original paper and some code can be found in the [associated repo](https://github.com/ermongroup/ddim).

This generates some images of rusty robots holding some torches!

<img src="media/robot3.jpg" width=256><img src="media/robot4.jpg" width=256><img src="media/robot7.jpg" width=256>

<img src="media/robot8.jpg" width=256><img src="media/robot11.jpg" width=256><img src="media/robot13.jpg" width=256>

## Image to Image Pipeline

The stable diffusion model can also be used to generate an image based on
another image. The following command runs this image to image pipeline:

```bash
cargo run --example stable-diffusion-img2img --features clap -- --input-image media/in_img2img.jpg
```

The default prompt is "A fantasy landscape, trending on artstation.", but can
be changed via the `-prompt` flag.

![img2img input](media/in_img2img.jpg)
![img2img output](media/out_img2img.jpg)

## Inpainting Pipeline

Inpainting can be used to modify an existing image based on a prompt and modifying the part of the
initial image specified by a mask.
This requires different unet weights `unet-inpaint.safetensors` that could also be retrieved from this
[repo](https://huggingface.co/lmz/rust-stable-diffusion-v1-5) and should also be
placed in the `data/` directory.

The following command runs this image to image pipeline:

```bash
wget https://raw.githubusercontent.com/CompVis/latent-diffusion/main/data/inpainting_examples/overture-creations-5sI6fQgYIuo.png -O sd_input.png
wget https://raw.githubusercontent.com/CompVis/latent-diffusion/main/data/inpainting_examples/overture-creations-5sI6fQgYIuo_mask.png -O sd_mask.png
cargo run --example stable-diffusion-inpaint --features clap --input-image sd_input.png --mask-image sd_mask.png
```

The default prompt is "Face of a yellow cat, high resolution, sitting on a park bench.", but can
be changed via the `-prompt` flag.

<img src="https://raw.githubusercontent.com/CompVis/latent-diffusion/main/data/inpainting_examples/overture-creations-5sI6fQgYIuo.png" width=256><img src="https://raw.githubusercontent.com/CompVis/latent-diffusion/main/data/inpainting_examples/overture-creations-5sI6fQgYIuo_mask.png" width=256>

![inpaint output](media/out_inpaint.jpg)

## ControlNet Pipeline

The [ControlNet](https://github.com/lllyasviel/ControlNet) architecture can be
used to control how stable diffusion generate images. This is to be used with
the weights for stable diffusion 1.5 (see how to get these above). Additional
weights have to be retrieved from this [HuggingFace
repo](https://huggingface.co/lllyasviel/sd-controlnet-canny/blob/main/diffusion_pytorch_model.safetensors)
and copied in `data/controlnet.safetensors`.

The ControlNet pipeline takes as input a sample image, in the default mode it
will perform edge detection on this image using the [Canny edge
detector](https://en.wikipedia.org/wiki/Canny_edge_detector) and will use the
resulting edge image as a guide.

```bash
cargo run --example controlnet --features clap,image,imageproc -- \
  --prompt "a rusty robot, lit by a fire torch, hd, very detailed" \
  --input-image media/vermeer.jpg
```
The `media/vermeer.jpg` image is the well known painting on the left hand side,
this results in the right hand side image after performing edge detection.
<img src="https://raw.githubusercontent.com/LaurentMazare/diffusers-rs/main/media/vermeer.jpg" width=256><img src="https://raw.githubusercontent.com/LaurentMazare/diffusers-rs/main/media/vermeer-edges.png" width=256>

## FAQ

### Memory Issues

This requires a GPU with more than 8GB of memory, as a fallback the CPU version can be used
but is slower.

```bash
cargo run --example stable-diffusion --features clap -- --prompt "A very rusty robot holding a fire torch." --cpu all
```

For a GPU with 8GB, one can use the [fp16 weights for the UNet](https://huggingface.co/runwayml/stable-diffusion-v1-5/tree/fp16/unet) and put only the UNet on the GPU.

```bash
PYTORCH_CUDA_ALLOC_CONF=garbage_collection_threshold:0.6,max_split_size_mb:128 RUST_BACKTRACE=1 CARGO_TARGET_DIR=target2 cargo run \
    --example stable-diffusion --features clap -- --cpu vae --cpu clip \
    --unet-weights data/unet-fp16.safetensors
```
