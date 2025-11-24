---
title: "Working Without a GPU"
subtitle: "Notes From a Year of Building Deep Learning Projects the Hard Way"
date: 2025-11-07
categories:
  - AI
tags:
  - Reflection
  - AI
  - AI Engineering
  - Deep Learning
  - GPU
layout: single
author_profile: true
read_time: true
comments: false
share: true
mermaid: true   
---


# Introduction

Most university labs in 2025 still look like they were frozen in time.  
A couple of office PCs, a shared server no one dares to upgrade, and a cluster that hasn’t been turned on since 2020. In my case, there wasn’t a single GPU I could reliably use — not even an old gaming card someone forgot to uninstall from a workstation. 

Especially in a European university which isn’t rich like those in the US, this is pretty common.

It happened more than once: a new project idea came up, I opened my laptop, typed `nvidia-smi`, and got the same answer every time — nothing. 

For a while I really thought this meant I couldn’t do anything serious. Everyone online seemed to fine-tune LLMs on their own 4090 at home, and here I was debugging Python on a machine that lagged when opening Chrome.

But after a few failed attempts, a few desperate experiments, and a lot of accidental learning, I realized something I genuinely didn’t expect:

**working without a GPU forced me to to think deeper about every part of the deep learning pipeline.**

Not in a motivational “constraints make you stronger” kind of way.  
More like: when you don’t have a GPU, you stop leaning on brute force and start noticing every part of the pipeline you normally ignore.

I ended up learning things I probably never would’ve bothered with otherwise — lighter training setups, CPU tricks, proper checkpointing, quantization, containerizing environments so they never break, relying on cloud machines only when absolutely necessary, and building prototypes around APIs instead of raw compute.

None of this was planned. It just happened because I had no choice.

Over the past year, I’ve managed to finish a surprising number of projects this way:  
fine-tuning a 7B model on a single T4, getting LLaMA to run locally in quantized form, shipping small RAG prototypes that needed zero GPU, dealing with cloud instances dying in the middle of training, and even explaining to my supervisor why a missing GPU wasn’t the real blocker.

This post is simply a record of what I learned.  
Not a tidy tutorial — just the things that actually worked and the things that didn’t.

Before getting into any of that, I should start with the first mistake I made:  
thinking that buying a GPU would solve everything.


# Why Buying a GPU Didn’t Solve Anything

When I realized the lab wasn’t going to magically receive new hardware, my first instinct was the obvious one:  
Fine, I’ll just buy a GPU myself.

Everyone online seemed to have a 3090 or 4090 at home, so I assumed it was the standard solution.  
I saved up, watched way too many teardown videos, compared benchmarks for weeks, and finally bought a used card that wasn’t completely overpriced.

The excitement lasted about two days.

The first problem wasn’t even the GPU itself — it was everything around it.  
Drivers, CUDA versions, PyTorch compatibility, Linux quirks… I hadn’t realized how easy it is to break your environment by simply trying to “upgrade something small”. One wrong version and PyTorch suddenly can’t see the GPU, or the driver crashes, or half the libraries need to be reinstalled.

I thought I could fix it quickly.  
Instead, I spent an entire weekend trying to revert a CUDA installation because it silently overwrote the system path.

Once the card finally worked, I discovered a more embarrassing issue:  
**a single consumer GPU doesn’t actually go very far anymore.**

I tried loading a 13B model and immediately ran out of VRAM.  
I lowered the batch size to 1, still crashed.  
Eventually I got it running by slicing everything down to the point where the experiment wasn’t even meaningful.

Then there was the heat.  
Training anything that used the GPU continuously made my room feel like a sauna.  
I stopped one training run because I genuinely thought the breaker might trip.

I also underestimated how much time you lose maintaining your own hardware:

- reinstalling CUDA after a kernel update  
- testing which driver version breaks TensorRT  
- keeping two environments because one uses PyTorch 1.13 and another uses 2.x  
- rebooting because the GPU suddenly disappeared from `nvidia-smi`  

All of this happens *before* you get to run your actual experiment.

By the time I got through my first month of “owning a GPU”, I realized something I didn’t expect to admit:  
**the GPU wasn’t the bottleneck — I was.**

Most of the things I needed to do didn’t require raw compute.  
I needed clean data, a reproducible environment, a reliable pipeline, and a place to run a model *occasionally*. Instead, I ended up with a noisy metal box under my desk and a system that broke every time I touched it.

The card didn’t make my work easier.  
It made everything around it more fragile.

So I eventually stopped relying on it.  
I kept it for occasional inference tests, but for anything serious — anything that needed to be stable or repeatable — I started looking for alternatives.

That’s when I began piecing together free compute sources that actually worked.

# How I Started Getting Free Compute to Work

Once I gave up on treating my own GPU as a magic solution, I had to figure out how to actually run things without depending on it. I didn’t plan anything systematic. I just started trying whatever was available: Colab, Kaggle, HF Spaces, even a couple of “student cloud credit” programs I didn’t know existed until someone mentioned them in a Discord thread.

The surprising part wasn’t that these services work — it was how *specific* you have to use them for them to be reliable.

I’ll start with Colab, because that’s where I first managed to get anything non-trivial to run.

## Colab: basically useless for training, perfect for debugging

The trick with Colab is simple:

**you don’t train on it — you debug on it.**

Once I stopped treating Colab like a training environment, it became extremely useful.

My process eventually settled into something like this:

1. Write the entire pipeline locally in VSCode or Something you like  
2. Run it on CPU with a tiny subset of data (batch=1,2,3 samples) until it works
3. Fix everything that breaks (usually everything)  
4. Zip the folder and upload it to Colab  
5. Use the free T4 for the final sanity check  
6. Shut it down as soon as it stops being useful

Colab isn’t fast, it isn’t stable, and it disconnects whenever it feels like it —  
but it’s a great place to catch bugs you’d never want to waste real GPU time on.

One small debugging session on a T4 saved me hours of cloud rental fees later.

I once uploaded a text-generation dataset that accidentally included several megabyte-sized samples because a PDF parser exploded. On CPU I didn't notice it — the dataloader was slow anyway.  
On Colab, the GPU memory blew up instantly.

If that bug had happened on a paid instance, I would have wasted a full hour’s rental fee just to discover the dataset was wrong.

Colab paid for itself right there.

## Kaggle: the closest thing to “free GPU that actually works”

Kaggle surprised me.  
The sessions last longer, GPUs don’t disappear randomly, and the environment is much cleaner than Colab. I ended up finishing several medium-sized experiments there — including a QLoRA fine-tune of a 7B model.

The main problem is that Kaggle sessions still die unexpectedly.  
The good news is that Kaggle doesn’t warn you; it just kills the notebook like nothing happened. So you *must* prepare for that.

My first serious run died at around 35% progress. After being annoyed for a while, I added checkpoints and resume logic. The second run died again, but this time I resumed from where I left off.

It took three separate sessions to finish the whole fine-tuning job, but it worked.

A few things that helped:

- always save checkpoints to `/kaggle/working`, not the default  
- reduce max sequence length when VRAM gets tight  
- use gradient accumulation instead of large batches  
- avoid printing too much to stdout (surprisingly expensive on Kaggle)

Slow, yes. But good enough for anything below “production-level”.

## HF Spaces: unexpectedly helpful once you treat it like a deployment playground

Hugging Face Spaces wasn’t something I expected to rely on, but I ended up using it whenever I needed to show something functioning to someone who didn’t care about the training process.

It’s not suitable for training at all.  
But for small demos — quantized models, RAG prototypes, or even a quick Gradio front-end — it’s perfect.

The biggest issue is that the “free GPU” is more of a suggestion than a guarantee. Some days you get one, some days you don’t. But the CPU mode is reliable, and that’s all I needed for LLaMA-cpp deployments or API-based models.

One small hack that saved me

HF Spaces has a fairly strict RAM limit.  
Loading even a 7B model in FP16 is impossible. But once I converted it to a GGUF file and used `llama.cpp` as the backend, the whole demo suddenly ran smoothly on CPU.

A few people thought I was doing magic.  
I wasn’t. I was just using a quantized file format designed for machines that can’t run anything heavier.

## Random free credits: more useful than expected

Most cloud providers have some form of student or trial credits. I used several of them, but the one thing they all have in common is that you must not waste the time on debugging.

That’s where the Colab + CPU-debugging workflow became essential.  
I didn’t burn credits on issues like:

- batch sizes being too large  
- tokenizers choking on weird text  
- dataloaders failing on corrupted files  
- running out of disk space  
- models not loading because I messed up a config  

By the time I touched a paid cloud machine, the script usually ran on the first try.

I wish I could say I planned it that way, but honestly, it came out of frustration.  
Once I ran out of free credits once, I never made that mistake again.

---

This was basically how I stitched together enough compute to get things done.  
It wasn’t pretty, but it was reliable, and it kept me from relying on a GPU I didn’t have.

The next part is where things got more serious: actually training large-ish models on hardware that shouldn’t have been able to run them in the first place.


# Closing Thoughts

To be honest, GPU is just a small part of the deep learning ecosystem. The data, pipeline, environment, and deployment matter just as much — if not more.

It wasn’t always fun. Some days were just messy: sessions dying, models not loading, cloud credits evaporating faster than expected. But over time, these constraints shaped my habits. I got more careful with experiments, more systematic with debugging, and much less dependent on the “ideal setup” that rarely exists in real life anyway.

And the last but not least: **always make full use of your student email** — it unlocks more free compute and credits than most people realize.
