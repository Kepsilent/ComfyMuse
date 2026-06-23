# ComfyMuse Config

## Language
lang: zh-CN
obsidian_path: D:\wanjie\Documents\Obsidian

## Platform
platform: comfyui
comfyui_path: D:\wanjie\Documents\Local AI\ComfyUI\ComfyUI\ComfyUI

## Limits
memory_limit: 999
recommend_count: all

## Hardware
gpu: NVIDIA GeForce RTX 3060
vram: 12GB
ram: 32GB
tier: High

## Models
- name: darkBeast_dbzit9DIMRclaw.safetensors
  size: 12GB
  node_checkpoint: CheckpointLoaderSimple
  node_clip: CLIPLoader
  node_clip_name: qwen3_4b_zimage.safetensors
  node_clip_type: z_image
  node_vae: VAELoader
  node_vae_name: ae.safetensors
  node_latent: EmptyFlux2LatentImage
  node_latent_size: 1024x1024
  node_encode: CLIPTextEncode
  node_prompt_format: standard
  params: {steps: 40, cfg: 1, sampler: euler, scheduler: simple}
  style: realistic
  safety: nsfw
  use_case: portrait

- name: Asian Realism PONY v3.0
  size: 6.5GB
  node_checkpoint: CheckpointLoaderSimple
  node_clip: built-in
  node_clip_name: ""
  node_clip_type: ""
  node_vae: VAELoader
  node_vae_name: sdxl_vae.safetensors
  node_latent: EmptyLatentImage
  node_latent_size: 832x1216
  node_encode: CLIPTextEncode
  node_prompt_format: score_prefix
  params: {steps: 25, cfg: 5, sampler: euler, scheduler: normal}
  style: semi-realistic
  safety: nsfw
  use_case: general

- name: 2127ZImageAsianUtopian_v36BaseFFV.safetensors
  size: 12GB
  node_checkpoint: CheckpointLoaderSimple
  node_clip: CLIPLoader
  node_clip_name: qwen3_4b_zimage.safetensors
  node_clip_type: z_image
  node_vae: VAELoader
  node_vae_name: diffusion_pytorch_model.safetensors
  node_latent: EmptyLatentImage
  node_latent_size: 832x1216
  node_encode: CLIPTextEncode
  node_prompt_format: standard
  params: {steps: 30, cfg: 4, sampler: euler, scheduler: normal}
  style: realistic
  safety: sfw
  use_case: portrait

- name: JANKUTrainedChenkinNoobai_v777.safetensors
  size: 6.5GB
  node_checkpoint: CheckpointLoaderSimple
  node_clip: built-in
  node_clip_name: ""
  node_clip_type: ""
  node_vae: VAELoader
  node_vae_name: sdxl_vae.safetensors
  node_latent: EmptyLatentImage
  node_latent_size: 1024x1536
  node_encode: CLIPTextEncode
  node_prompt_format: standard
  params: {steps: 30, cfg: 5, sampler: euler, scheduler: normal}
  style: anime
  safety: nsfw
  use_case: general

- name: waiIllustriousSDXL_v170.safetensors
  size: 6.5GB
  node_checkpoint: CheckpointLoaderSimple
  node_clip: built-in
  node_clip_name: ""
  node_clip_type: ""
  node_vae: VAELoader
  node_vae_name: sdxl_vae.safetensors
  node_latent: EmptyLatentImage
  node_latent_size: 1024x1536
  node_encode: CLIPTextEncode
  node_prompt_format: standard
  params: {steps: 30, cfg: 5, sampler: euler, scheduler: normal}
  style: anime
  safety: nsfw
  use_case: general

- name: redcraftERNIERedmix_ernieRedmix.safetensors
  size: 10GB
  node_checkpoint: CheckpointLoaderSimple
  node_clip: CLIPLoader
  node_clip_name: ernie-image-prompt-enhancer.safetensors
  node_clip_type: lumina2
  node_vae: VAELoader
  node_vae_name: flux2-tiny-vae.safetensors
  node_latent: EmptySD3LatentImage
  node_latent_size: 1024x1024
  node_encode: CLIPTextEncode
  node_prompt_format: standard
  params: {steps: 10, cfg: 1, sampler: euler, scheduler: simple}
  style: text-render
  safety: sfw
  use_case: text
