---
layout: ../../layouts/Bare.astro
title: "From RAVE to Chaos: Neural Audio Synthesis as Instrument Design"
date: 2026-02-05
---

# From RAVE to Chaos: Neural Audio Synthesis as Instrument Design

(draft feb 2026)

## Intro

In this text, we will explore two contrasting approaches to neural audio synthesis: one rooted in training a model on a dataset (using the RAVE model), and another born from hand-writing a neural network (using a TorchScript loaded via the nn~ audio plugin). Both approaches harness neural networks to create sound, but they differ fundamentally in process and philosophy. By examining these side by side, we’ll uncover what it means to treat a neural network as an instrument and how this challenges our understanding of AI in music.

- RAVE model (Training approach): A Realtime Audio Variational autoEncoder developed at IRCAM. This approach involves preparing a dataset of sounds, training a deep network to learn that data’s patterns, and then using the trained model to generate or process audio. In this sense, the dataset itself becomes a compositional act and the network learns to mimic it.

- Architechturally composed ConvNet (No-training approach): Instead of training on data, we directly code a neural network architecture ([convolution](https://en.wikipedia.org/wiki/Convolution) layers with non-linearities) and use it as an audio processor. This is a more experimental, “hacking” approach: we leverage the same tools that normally host trained models (the nn~ external) but feed it a custom TorchScript model that initializede without any training data. The architecture itself becomes the instrument, and its randomly initialized weights become the parameters shaping its sound.

We’ll delve into the technical structure of the custom TorchScript, verify the understanding of how RAVE and nn~ relate, and discuss the artistic implications of training vs. hand-coding and the agency of these neural networks in a creative context. Along the way, I’ll share code snippets, SuperCollider patch examples, and personal observations from my experiments.

## Understanding RAVE: Training a Neural Audio Model

What is [RAVE](https://github.com/acids-ircam/RAVE)? RAVE (Realtime Audio Variational autoEncoder) is a neural audio synthesis framework that learns to compress and reconstruct audio in real-time. Technically, it’s a variational autoencoder (VAE) tailored for audio: it has an encoder that converts audio to a low-dimensional latent representation and a decoder that reconstructs audio from that latent code. By training on a large dataset of sounds, a RAVE model learns a latent space of that audio domain, enabling tasks like generating new sounds, transforming input audio, or doing style transfer between sounds.

Training Process: Training RAVE involves three main steps: (1) preparing a dataset, (2) running the training to optimize the model’s weights on that data, and (3) exporting the trained model for real-time use. In practice, one must preprocess audio into a dataset and possibly apply augmentations. For example, RAVE v2 models include an augmentation called “mute” that randomly silences portions of training audio to force the model to learn silence, an interesting detail highlighting that even silence is something a model must learn to reproduce properly. Training is typically computationally intensive and can run for many hours or days, passing through the data repeatedly (epochs) until the model converges. RAVE’s authors provide various preset architectures (v1, v2, v2_small, etc.), each suited for different tasks or GPU budgets.

My RAVE Experiments: Last summer, in collaboration with [Mengtian Sun](https://eat1glueberrypie.com/), we trained a RAVE model on 1 hour of my own noise music recordings. We used the v2_small configuration, which, as documented, is an architecture with a smaller receptive field and adapted for more stationary signals like noise. The training process itself was an exploratory composition: I curated an hour of diverse noise, and the neural network gradually learned a representation of that noise universe. After training, we exported the model to a TorchScript .ts file (with the streaming option for real-time to avoid artifacts) and auditioned the results.

Results & Observations: The output of the noise-trained model was not as dramatically varied as one might hope. In fact, different input sounds processed through this RAVE model came out sounding “flattened” or homogenized. The model seemed to average out nuances and produce a somewhat uniform noise texture. This makes sense in hindsight: the VAE had learned a compressed latent representation of the entire noise dataset. It found a kind of statistical middle-ground of those noise recordings. While it could reproduce the general texture (and did capture some spectral characteristics of my noise data), it lacked the extreme dynamics or distinct identities of individual recordings. The outcome was a smooth, almost polite noise – interesting but less chaotic than the raw source material. It was as if the model ironed out the quirks in the data, giving an almost noisy yet somewhat characterless rendition of noise.

Even when feeding the model very different inputs (or even just letting it generate from random latent codes), the sound stayed within a narrow band of what it had learned. This underscored a key idea: a trained model will always reflect its training data’s bias. RAVE, acting as a mimesis machine, was intelligent only in so far as it could regurgitate the essence of its training sounds. Its creativity was bounded by the recordings I gave it.

Curious about how a different dataset might change the model's character, I began a second training.

Second Training & Artifacts: This time we used the full v2 architecture (a larger model) on a human voice dataset(recording of our voices passages in Chinese and English). Due to GPU limits, I tweaked some batch-related parameters to get it running. The training succeeded, and I got to observe the model’s evolution across checkpoints, from rough output in early epochs to more refined reconstructions later. An intriguing find was the presence of certain artifacts in the generated audio. Some sounds had a harsh, resonant edge, a kind of high-frequency buzzing or metallic ringing that wasn’t explicitly in the dataset. These artifacts, while considered “imperfections” from a fidelity standpoint, became signatures of the model’s neural nature. I noticed that these neural artifacts are reminiscent of those heard in other AI audio works (for example, the subtle digital distortion or aliasing often heard in neural audio synthesis). They might arise from the model’s convolutional layers or the adversarial training process. Importantly, I would later encounter similar artifacts in my hand-written network, hinting that certain sonic fingerprints are innate to convolutional neural networks, whether trained or not.

In summary, using RAVE taught me that training a neural network is as much an art as a science. The artist’s influence comes through in data curation (what sounds you choose to train on, how you augment them, etc.) and in tweaking training parameters. The result is a model that embodies the training data. It’s an instrument that knows those sounds. This brings us to an ontological point: when you train a model on a dataset, you are in effect saying “this collection of sounds is my composition, and I want an instrument that can play in that style”.

Before contrasting this with the hand-written approach, let’s introduce the tool that allows these models to come alive inside the synthesis setups: nn~.

## nn~ as a Host: Loading Models into SuperCollider/Max/Pd

[nn~](https://github.com/acids-ircam/nn_tilde) is an external object originally developed by the RAVE authors (ACIDS-IRCAM) for Max/MSP and Pure Data, and adapted for SuperCollider by elgiano as [nn.ar](https://github.com/elgiano/nn.ar). It acts as a bridge between deep learning models and audio environments wrapping LibTorch (the C++ back-end of PyTorch) for use in real-time audio software. Essentially, nn~ is an empty shell that becomes useful only when you give it a pretrained TorchScript model.

- A TorchScript model (with .ts extension) is a serialized neural network that can run independently of Python – perfect for deployment. RAVE provides a command to export models to TorchScript, which is what I used to get my .ts files.

- nn~ loads such a model and exposes its methods as audio processing units. In fact, many TorchScript models can have multiple entry-point methods. For instance, a RAVE model might have separate methods for encode, decode, and an all-in-one forward (encode+decode). With nn~, you can select which method to run and it will create the appropriate number of audio inlets/outlets for that method. This multi-method capability is key to the hack.

The Hack: **While nn~ was conceived to host trained models, it doesn’t actually verify what the model is or how it was obtained.**  As long as the .ts file is a valid TorchScript network that accepts and produces audio tensors, nn~ will run it. This opens the door to a mischievously creative idea: what if we write our own neural network (in PyTorch), don’t train it on any data, but still export it as TorchScript? We could then load this untrained network into nn~ and use it like a bizarre audio effect or synth. This approach was first shown by [George Bagdasarov](https://monoskop.org/George_Bagdasarov) in a block seminar at Universität der Künste Berlin, whose code experiments clearly demonstrated that nn~ could host hand-written networks. Building on that, I developed my own version as the second part of this exploration.

Why do this? It is, in part, a way of questioning the standard pipeline. Instead of “collect data → train model → use model”, this approach proposes “imagine a model → skip training → use model”. It’s a bit like circuit bending but in software: taking the neural network paradigm and bending it to serve as a direct, unpredictable sound generator. Conceptually, it shifts the focus from training (learning from the world) to topology (designing your own little world), from teaching an instrument what to play, to building the instrument itself: you assemble oscillators, filters, nonlinearities (here realized as neural network layers/functions) in any configuration you design, and see what sounds it makes. There’s no prior knowledge in it; you impose form on it, rather than letting it learn form from data.

Let’s take a closer look into the specific scratch-built model and see how it works internally.

## Hand-Writing a Neural Network Instrument (ChaosEffect)

In a [Jupyter Notebook](https://jupyter.org/), I defined a PyTorch `nn.Module` called `ChaosEffect`: a small [convolutional neural network](https://en.wikipedia.org/wiki/Convolutional_neural_network) with three distinct methods — `forward`, `topo`, and `chaos` — each representing a different mode of operation. Think of them as three differently wired circuits sharing the same underlying components: one a dual-modulator effect, one a leaner wave-shaper, and one a nonlinear oscillator driven by intra-buffer phase perturbation. The module is exported as a single TorchScript file (`test_env_topo_chaos_3.ts`), loadable in SuperCollider via nn~, with each method accessible as a separate processing mode.

In PyTorch, any neural network is defined as a subclass of `nn.Module`. You implement an `__init__` method to define the layers, and one or more `forward`-style methods to define how audio flows through them. Once defined, the module can be serialized to TorchScript with `torch.jit.script()`, making it runnable outside of Python, which is what nn~ requires.

#### Network Architecture and Weight Initialization
```python
class ChaosEffect(torch.nn.Module):
    def __init__(self, operators=4, w_mult=1, kernel=1):
        super().__init__()
        self.operators = operators
        self.w_mult = w_mult

        # Convolutional layers (kernel=1 means no time context, just per-sample)
        self.conv_in   = torch.nn.Conv1d(1, operators, kernel_size=kernel)
        self.conv      = torch.nn.Conv1d(operators, operators, kernel_size=kernel)
        self.conv_out  = torch.nn.Conv1d(operators, 1, kernel_size=kernel)
        # An extra conv for chaos mode (kernel=5 introduces temporal context)
        self.conv_chaos = torch.nn.Conv1d(operators, operators, kernel_size=5, padding=2)

        # Scale weights by w_mult (no training, so this is a manual tweak)
        for layer in [self.conv_in, self.conv, self.conv_out, self.conv_chaos]:
            with torch.no_grad():
                layer.weight *= w_mult
```

The network has four [Conv1d](https://docs.pytorch.org/docs/stable/generated/torch.nn.Conv1d.html) layers. `conv_in`, `conv`, and `conv_out` all use `kernel_size=1`, meaning they operate per-sample with no temporal window — functionally equivalent to a fully connected layer applied independently at each time-step. `conv_in` expands the single input channel into `operators` channels; `conv` is a hidden layer (operators → operators); `conv_out` collapses back to 1 channel. These three form the core feedforward path shared by all methods.

`conv_chaos` uses `kernel_size=5` with `padding=2`, so it looks at a window of 5 consecutive samples within the current buffer. This gives it local temporal context — it is essentially a learned FIR filter operating across neighboring samples. It is only used in the `chaos` method.

Since there is no training, weights are frozen at their random initialization. `w_mult` scales all weights uniformly at construction — a manual way to tune overall nonlinear intensity before any audio runs through. The model is instantiated with `operators=14` and `w_mult=2`:

```python
chaos = ChaosEffect(operators=14, w_mult=2)
```

So the actual hidden dimension throughout is 14 channels, and all weights are doubled from PyTorch's default random initialization. Too high a `w_mult` causes the output to blow up; too low and the network is nearly transparent. Finding these values was trial and error, since there is no loss function to optimize.

#### Registering Method Signatures for nn~

Before exporting, each method needs its input/output channel count registered as a buffer so nn~ can configure the correct number of inlets and outlets:

```python
chaos.register_buffer("forward_params", torch.tensor([3, 1, 1, 1]))
chaos.register_buffer("topo_params",    torch.tensor([2, 1, 1, 1]))
chaos.register_buffer("chaos_params",   torch.tensor([2, 1, 1, 1]))

scripted_model = torch.jit.script(chaos)
torch.jit.save(scripted_model, "test_env_topo_chaos_3.ts") # you can choose other names
```

The tensor `[3, 1, 1, 1]` tells nn~ that `forward` expects 3 audio inputs and produces 1 output; `[2, 1, 1, 1]` means 2 inputs and 1 output for `topo` and `chaos`. Without these buffers, nn~ cannot automatically determine the method's audio I/O layout. All three methods here run at full audio rate. This convention is documented in the RAVE team's [cached_conv](https://acids-ircam.github.io/cached_conv/) library, which is the official framework for building streamable models compatible with nn~.

<details>
<summary>One distinction</summary>  
One distinction worth noting: `cached_conv` replaces standard `torch.nn.Conv1d` layers with its own streaming-aware versions, which use an internal cache to handle real-time buffer-by-buffer processing without latency artifacts. ChaosEffect uses standard `torch.nn.Conv1d` layers instead, which means it is not streamable in the `cached_conv` sense. It works in real time because nn.ar handles the buffering externally, but the convolutions themselves are not designed with streaming in mind.
</details>

#### Method 1: forward (Base Input with Two Modulators)
```python
    @torch.jit.export
    def forward(self, buffer):
        x    = buffer[:, 0:1]    # main audio input
        mod1 = buffer[:, 1:2]    # first modulator input
        mod2 = buffer[:, 2:3]    # second modulator input

        y = self.conv_in(x)
        y = self.conv(y)
        # Nonlinear transformations:
        y = torch.sin(y * torch.pi * mod1)
        y = 1 - torch.tanh(torch.abs(y) * mod2)
        y = self.conv_out(y)
        y = y / (self.operators * self.w_mult)
        return y.squeeze(1)
```

`forward` takes 3 input channels: main audio `x`, and two modulators. After `conv_in` and `conv` expand and mix `x` into 14 channels of random linear combinations, two nonlinearities are applied in series:

**Sine waveshaping**: `y = sin(y * π * mod1)`. `mod1` scales the argument of the sine — when `mod1 = 1`, this is `sin(y * π)`; larger values cause the sine to cycle faster relative to the signal amplitude, folding in more harmonics. It behaves like a wavefolder whose fold intensity is modulated in real time.

**Tanh-based amplitude shaping**: `y = 1 - tanh(abs(y) * mod2)`. Using `abs(y)` makes this operation symmetric around zero. When the signal amplitude is large or `mod2` is high, `tanh` saturates toward 1 and `1 - tanh(...)` approaches 0 — effectively attenuating loud signals. When the signal is quiet or `mod2` is near zero, the output stays close to 1. Note that if `mod2` is a bipolar audio signal (i.e. can go negative), `abs(y) * mod2` can also be negative, and `tanh` will output in (-1, 0), making `1 - tanh(...)` exceed 1 — an edge case worth being aware of when patching. In practice, using a unipolar envelope or LFO for `mod2` gives the most predictable shaping behavior.

`conv_out` then collapses the 14 channels back to 1, and dividing by `(operators * w_mult) = 28` keeps the output level roughly normalized.


#### Method 2: topo (Alternate Topology, One Modulator)
```python
    @torch.jit.export
    def topo(self, buffer):
        x    = buffer[:,0:1]
        mod1 = buffer[:,1:2]

        y = self.conv_in(x)
        y = self.conv(y)
        y = torch.sin(y * torch.pi * mod1)
        y = self.conv_out(y)
        y = y / (self.operators * self.w_mult)
        return y.squeeze(1)
``` 

The `topo` method is a simplified variant of `forward`. It takes only 2 channels of input. one audio x and one modulator `mod1`. The processing is the same up to the sine nonlinearity controlled by `mod1`, but it omits the tanh-based shaping and goes straight to conv_out. The `topo` is a different topology as the name relects: one fewer stage, to compare the sonic result.

Why have this mode? From an experimental perspective, I wanted to see what the sound is like when we remove one layer of complexity. One can imagine forward as two nonlinear stages (sin then tanh) in series, whereas topo is just one nonlinear stage (sin). This might sound more stable or less chaotic, since there’s one fewer modulation happening, `topo` still allows `mod1` to warp the harmonic content of the sound, but it doesn’t do the dynamic gating that `mod2` did in `forward`.

Perhaps topo acts more like a strange wavefolder or FM oscillator depending on `mod1`. With an appropriate `mod1` (say an LFO or another audio oscillator), it could produce rich spectra. But likely it’s a bit more tame than `forward` since loud parts aren’t being squashed by a tanh; meaning it might output hotter levels or even get more distorted if mod1 drives it crazy. 

#### Method 3: chaos (Intra-Buffer Phase Perturbation)
```python
    @torch.jit.export
    def chaos(self, buffer):
        x       = buffer[:,0:1]
        control = buffer[:,1:2]

        y = self.conv_in(x)
        
        shift1 = self.conv_chaos(y)
        y = torch.sin(y * torch.pi + shift1 * control * 0.5)
        
        shift2 = self.conv_chaos(y)
        y = torch.sin(y * torch.pi + torch.tanh(shift2) * control * 2.0)

        y = self.conv_out(y)
        y = y / (self.operators * 0.6)
        return y.squeeze(1)
```

`chaos` introduces a qualitatively different structure. After `conv_in(x)`, it applies `conv_chaos` twice, each time using its output to perturb the phase of a sine:

**First perturbation**: `shift1 = conv_chaos(y)`. Because `conv_chaos` has a kernel of 5 samples with symmetric padding, `shift1` is a locally-weighted combination of the current sample and its 2 immediate neighbors on each side — a learned FIR filter. This `shift1` is then added into the phase argument: `y = sin(y * π + shift1 * control * 0.5)`. At `control = 0`, this is just `sin(y * π)`. As `control` increases, the local temporal structure of `y` starts bending its own phase — the signal begins to interact with itself within the buffer window.

**Second perturbation**: the updated `y` is passed through `conv_chaos` again to get `shift2`, and applied to another sine: `y = sin(y * π + tanh(shift2) * control * 2.0)`. The `tanh` clamps `shift2` into (-1, 1) before scaling, preventing the phase offset from blowing up. The second pass doubles the effect and is scaled more aggressively (`* 2.0`), making the output highly sensitive to `control`.

A clarification on terminology: this is not feedback in the recurrent or cross-buffer sense: no state is carried from one audio buffer to the next. What `conv_chaos` introduces is *intra-buffer self-interaction*: the phase of each sample is perturbed by a locally weighted mix of its neighbors, and after a nonlinearity this creates complex, nonlinear spatiotemporal structure within each processed block. The emergent behavior can resemble chaos because the sine-of-sine-of-convolution chain is highly sensitive to the initial signal and `control` value, but it is deterministic and stateless.

The output scaling uses `0.6` instead of `w_mult`, an empirical adjustment since chaos mode tends to produce larger amplitude values than the other methods.

Because the conv layers carry a bias term (PyTorch default), even a silent input (`x = 0`) can produce non-zero output in `chaos` mode if the bias values and control signal combine to push the sine functions into a non-zero region.

### One Module, Three Circuits

The three methods share the same weights but wire them differently. `forward` offers two modulation handles with amplitude control; `topo` strips it down to pure harmonic shaping; `chaos` replaces clean modulation with self-perturbing phase structures. The progression traces a path from predictable to increasingly opaque behavior, all without any training.

Since the weights are fixed at random initialization, their actual values are somewhat arbitrary: the network's character has to be discovered through use rather than designed in advance, not unlike finding the sound of a newly built DIY synth module by plugging it in and turning knobs. That said, w_mult is the main lever available without rewriting the architecture: re-instantiating the model with a different w_mult (or operators) and re-exporting is the simplest way to explore a different sonic territory:

```python
chaos = ChaosEffect(operators=5, w_mult=4) # try different values
scripted_model = torch.jit.script(chaos)
```

Now that the TorchScript model is ready (the full Jupyter notebook is available [here](torch-nn)) , the next step was to load it into SuperCollider via nn~ and build a playable patch around it.



## Playing the Network in SuperCollider

With the model exported (test_env_topo_chaos_3.ts), using it in SuperCollider is straightforward. 

```js
s.boot;
NN.load(\void, "/path/to/test_env_topo_chaos_3.ts");
NN(\void).methods;    // List available methods (forward, topo, chaos)
NN(\void).describe;   // Describe I/O configuration of the model
```

`NN.load` assigns the model to a symbol key — `\void` here, though the name is arbitrary. `NN(\void).methods` confirms the three exported methods loaded correctly and shows their inlet/outlet counts: `forward` has 3 inlets and 1 outlet; `topo` and `chaos` each have 2 inlets and 1 outlet. `describe` prints the full I/O layout, confirming these match what we registered with `register_buffer`.

To generate sound, `NN(\void, \method).ar(inputs, blockSize)` creates an audio-rate UGen running the specified method. nn~ uses an internal circular buffer to accumulate samples and runs the network on a separate thread once a full block is ready.


### Building a Modular Patch with NN~ in SC

I approached this like building a small modular synth where each neural method is a module. Here are a few configurations I tried:

#### topo as a standalone synth

```js
SynthDef(\topo, { |freq = 100, amp = 0.5, out = 0, modBus = 0|
    var input, mod, nnOutput, final, energy;
    input = Saw.ar(freq);
    mod   = K2A.ar(MouseX.kr(1, 18).lag(0.2));  // control signal from mouse X (1 to 18)
    nnOutput = NN(\void, \topo).ar([ input, mod ], 2048);
    // measure amplitude and send to a control bus for cross-modulation
    energy = Amplitude.kr(nnOutput);
    Out.kr(modBus, energy);
    // filter and output
    final = HPF.ar(nnOutput, 20);
    final = LPF.ar(final, 15000);
    Out.ar(out, Limiter.ar(final * amp, 0.9) ! 2);
}).add;
```
A sawtooth wave feeds into `topo`, with `mod1` mapped to horizontal mouse position (range 1–18). The HPF removes DC offset and subsonic content the network may produce; the LPF trims high-frequency artifacts; the limiter prevents clipping. The signal is then copied to stereo.

The `Amplitude.kr` line measures the output level and writes it to `modBus`: a control bus that will later drive the chaos synth's pitch, creating cross-modulation between the two networks.

Sonically, `mod1` range has a large effect: at low values the network produces gritty, harmonically dense tones; at higher values the sine wavefolder cycles faster and the output becomes noisier and more unpredictable. With a sub-audio `freq`, it can behave like a slow distortion unit with tremolo-like fluctuation; at audio-rate `freq` it produces dense sidebands. This range made it useful as both a tonal and percussive source, with an envelope driving `mod`, it could produce short aggressive hits, which I later used in the percussive SynthDef.

#### Chaos as a standalone synth

```js
SynthDef(\Chaos, { |baseFreq = 60, amp = 0.5, out = 0, modBus = 0|
    var input, control, nnOutput, final, follower, dynamicFreq;
    // read the modBus (amplitude from topo synth) as a control signal
    follower = In.kr(modBus, 1);
    dynamicFreq = baseFreq + (follower * 740).lag(0.5);
    input   = Saw.ar(dynamicFreq);
    control = K2A.ar(MouseY.kr(1, 38).lag(0.2));  // control from mouse Y
    nnOutput = NN(\void, \chaos).ar([ input, control ], 2048);
    final = HPF.ar(nnOutput, 20);
    final = LPF.ar(final, 15000);
    Out.ar(out, Limiter.ar(final * amp, 0.9) ! 2);
}).add;

```

The design choice here is `dynamicFreq`: the amplitude follower from `modBus` (written by the `\topo` synth) is scaled by 740 and added to `baseFreq`. When `topo` produces a loud burst, the pitch feeding into `chaos` sweeps upward by up to ~740 Hz, then settles back with a 0.5s lag. This makes the two networks cross-coupled: energy in one shifts the tonal territory of the other.

`control` (mouse Y, range 1–38) determines how strongly the intra-buffer phase perturbation operates. At low values, `chaos` produces stable, richly distorted tones; at high values it pushes into dense, unpredictable oscillation. Combined with the pitch modulation from `topo`, the two synths develop a kind of call-and-response character, where a rhythmic burst in `topo` excites `chaos` into a higher register, which then settles as the energy dissipates.

#### Chaining topo into chaos via audio bus

A second interconnection is direct audio chaining: `topo`'s output becomes `chaos`'s input.

```js
~audioBus = Bus.audio(s, 1);
SynthDef(\topo_sender, { |freq=100, out=0, audioBus=0|
    var input = Saw.ar(freq);
    var mod   = K2A.ar(MouseX.kr(1, 28).lag(0.2));
    var nnOut = NN(\void, \topo).ar([ input, mod ], 2048);
    Out.ar(audioBus, nnOut);             // send topo output into audioBus
    Out.ar(out, nnOut * 0.5 ! 2);        // also directly to speakers (stereo)
}).add;
SynthDef(\chaos_processor, { |amp=0.5, out=0, audioBus=0|
    var input   = In.ar(audioBus, 1);    // read from the audio bus (topo's output)
    var control = K2A.ar(MouseY.kr(1, 118).lag(0.3));
    var nnOut   = NN(\void, \chaos).ar([ input, control ], 2048);
    var final   = LPF.ar( HPF.ar(nnOut, 20), 15000);
    Out.ar(out, Limiter.ar(final * amp, 0.9) ! 2);
}).add;
// Then play them:
~src = Synth(\topo_sender, [\freq, 110, \audioBus, ~audioBus]);
~proc = Synth(\chaos_processor, [\audioBus, ~audioBus, \amp, 0.5], target: ~src, addAction: \addAfter);

```

`\topo_sender` writes its output to both the speakers and `~audioBus`. `\chaos_processor` reads from that bus, making `chaos` act as a post-processing stage on `topo`'s full audio output.

The result is an opaque combined transfer function: the motion of the first network (sweeps, rhythmic patterns) gets refracted and diffused by the second. 

These neural processes can also be wrapped into conventional instrument shapes:

**\perc** (topo-based): A `Env.perc` envelope drives both the amplitude of a saw wave and `mod1` of `topo` simultaneously. Each trigger sends a burst of audio into the network while rapidly sweeping its wavefolder intensity, producing a short, spectrally dense hit. Adjusting `freq` and envelope duration shifts the character from kick-like thuds (low freq, short decay) to snare-like snaps (higher freq, fast mod sweep) to resonant bass hits (longer decay).

**\pad** (chaos-based): A slow random LFO modulates `control`, causing `chaos` to drift in and out of its unstable regime over time. A sine oscillator with slight pitch wobble provides the input, and a `FreeVerb` smooths the output into a pad texture. The result is an evolving drone that periodically fractures into noisy harmonics as `control` climbs, then stabilizes as it falls back.

Both instruments demonstrate that neural network layers, even untrained ones, can function as the core processing element in conventional synthesizer roles.

### Reflections on Using the Hand-Written Network

Working with ChaosEffect felt very different from working with the trained RAVE model.

With RAVE, using a trained model typically means transforming or generating audio within a learned territory, feed it a sound and it outputs a variant shaped by the training data, or interpolate between known timbres. It behaves like a smart effect that carries a sense of its source material: the dataset genre is always audible somewhere in the output.

With ChaosEffect, it was more like dealing with an unknown electronic circuit. Small changes in input or control signals could push it from a stable state to total noise. It demanded an improvisational mindset: move a knob, listen to how the network responded, adjust. At times it felt alive:  it would start oscillating without a meaningful input, almost as if it had its own will. This is a useful fiction: the math is deterministic, but the opacity of the system makes authorship feel genuinely shared. That feeling points toward a real question worth asking: one about agency, and about what it means to compose with a process you cannot fully predict or explain.

This leads into the broader conceptual discussion: what is the ontology of these two approaches, and do these networks have agency, or are they just black boxes we fiddle with?

## Training vs. Hand-Coding: Ontological Differences

The two approaches described in this text differ not just technically but in what kind of object they produce.

When you train a RAVE model, the network's weights become a compressed encoding of the dataset. The sounds it generates or transforms are always in dialogue with that source material: they interpolate within a learned distribution, recombine its statistical patterns, or extend it slightly at the edges. The artist's primary act is curatorial: choosing what to record, how to augment it, what to leave out. In my noise training, this became literal. An hour of curated recordings became the boundary of what the model could express. The network knows that territory intimately but cannot leave it. In this sense, using a trained model is always a conversation with the data it came from, and the dataset itself is a kind of latent composition.

Hand-coding a network inverts this entirely. There is no dataset, no external reference; the weights are random and stay that way. The artist's act is architectural: deciding how many layers, how they connect, what nonlinearities to introduce. The network doesn't encode any knowledge about the world; it is a self-contained signal processing structure whose behavior emerges entirely from its mathematical form. ChaosEffect doesn't "know" anything about noise or voices or any sound. It only knows its own topology. This makes it ontologically closer to a synthesizer than to a model: not a representation of something, but a thing in itself.

This distinction echoes a long-standing duality in electronic music. Training a model on recordings has structural similarities to musique concrète: the source material is real-world sound, and the work consists of transforming and recombining it, except here the transformation is learned rather than manually spliced. The difference is that RAVE doesn't replay or collage the recordings; it learns their underlying distribution and generates from it, which is a more abstract operation. Hand-coding a network, by contrast, is closer to synthesis in the traditional sense: you design an abstract generator from scratch, and the sound is a product of the system's internal logic rather than any external reference. The parallel isn't perfect, but the creative posture is recognizably similar.

One question worth sitting with: is an untrained network "AI" at all? Strictly speaking, no. Nothing was learned, no intelligence was acquired through optimization. It is a neural network in form only, borrowing the architecture of machine learning without the learning. And yet, working with ChaosEffect felt meaningfully different from working with a conventional DSP effect. The opacity of its behavior, the way small parameter changes produced unpredictable responses, the sense that it had a character I had to discover rather than design: these qualities felt closer to working with a trained model than with a filter or oscillator. Perhaps what makes something feel like AI in practice is not whether it learned, but whether it resists full understanding by its operator.

The two approaches offer different creative affordances rather than different levels of quality. The trained RAVE model gave me mimesis: a network that could produce textures recognizably descended from my noise recordings or from our voice dataset, but constrained to that territory. ChaosEffect gave me something genuinely outside any existing sound world, but harder to direct. Its musical usefulness depended entirely on how well I could navigate its behavior through patching and parameter control. Both are generative, but one generates from memory and the other from structure.

## Agency of the Network: Black Box or Instrument?

The ontological question from the previous section has a practical counterpart: in the moment of working with these networks, who is actually in control?

With the trained RAVE model, the sense of control was relatively clear. I chose the dataset, set the training parameters, and the model's behavior stayed within a recognizable territory. Exploring it felt like navigating a space I had partly designed, even if I couldn't predict every corner of it.

With ChaosEffect, the experience was different. Setting initial conditions (oscillator frequencies, envelope shapes, control ranges) and triggering the synths would sometimes produce a trajectory I hadn't anticipated. Certain combinations of `mod1` and `control` values produced bird-like chirps or motor-like drones that I didn't design so much as discover. When the `topo` and `chaos` modules were linked, the system would occasionally find a groove or spiral out in ways that felt less like I was operating a tool and more like I was negotiating with something. This is not unique to neural networks: any sufficiently complex nonlinear system, whether an analog modular patch, a feedback loop, or a physical resonator, can produce this feeling. But the neural network form amplifies it, partly because of cultural associations with AI, and partly because the opacity is real.

Technically, ChaosEffect is completely determined. It has static weights, static nonlinear functions, and no online learning. Given the same input and the same parameters, it will always produce the same output. In that sense it is no more agentive than a filter or a reverb. But predictability in principle is not the same as predictability in practice. With 14 channels of randomly initialized weights interacting through sine and tanh nonlinearities, working out what a given input will produce requires either simulation or listening. I did the latter. I learned its behavior empirically: increasing `mod1` makes it brighter; pushing `control` past a certain threshold causes self-oscillation; feeding the output of `topo` into `chaos` produces textures neither generates alone. This is how one learns to play an instrument, not how one operates a calculator.

The impulse response analogy is worth noting here, with a caveat. For a linear system, the impulse response fully characterizes behavior. These networks are not linear; the sine and tanh functions mean the response changes with input amplitude. But at small amplitudes, the behavior approximates a linear filter, and even at larger amplitudes, the conv layers give the system something like resonant modes: frequencies it naturally emphasizes or oscillates at, determined by the random weight initialization. The sustained tones and artifacts that emerged in both the trained and hand-written networks are evidence of this, with the conv filters accidentally forming feedback paths that ring at particular frequencies.

Whether to call this a black box or an instrument is partly a question of stance. Analytically, one could inspect the weight spectra, measure frequency responses at different amplitudes, or trace signal paths through the layers. I didn't do that here; I took an artist's approach and treated it as a new instrument whose technique I had to develop through practice. That choice kept the system partly opaque by design, not because I couldn't open it, but because the opacity was generative. The network led me to sounds I wouldn't have arrived at through explicit design, and that felt like a meaningful contribution to the work, regardless of whether we call it agency.


## Conclusion

This text has traced two approaches to neural audio synthesis that differ not just in technique but in fundamental orientation. Training a RAVE model positions the network as a learning system: the artist curates data, the network absorbs it, and the result is an instrument that knows a particular sonic territory. Hand-coding ChaosEffect positions the network as raw material: no data, no learning, just architecture and randomness assembled into something playable. Both are legitimate uses of neural network technology, but they make different demands on the artist and produce different kinds of results.

One thing that surprised me in working through both was how much the hand-written approach clarified the trained one. Without a loss function to optimize toward, I had to actually listen to what each layer was doing: what a Conv1d with random weights sounds like on its own, what a sine nonlinearity does to a spectrum, how a kernel size of 5 differs from 1 in practice. These were questions I hadn't thought to ask while working with RAVE, where a well-trained model tends to hide its internal mechanisms behind coherent output. Writing the network by hand made the two feel less like separate paradigms and more like points on a continuum. At some level, a neural network is just a complex audio effect. The training is what gives it memory of the world.

The artifacts mattered too. The harsh resonances in the RAVE voice model and the unpredictable oscillations in ChaosEffect were both, in different ways, the network asserting its own character. I started to notice something that might be called a "neural sound" appearing in both trained and untrained contexts: spectrally complex, neither analog nor cleanly digital, shaped by the fingerprint of convolution and nonlinearity. I found myself drawn to that character rather than trying to work around it.

What this experience suggested to me, as someone still early in working with these tools, is that "AI in music" does not have to mean training on large datasets and generating plausible output. It can also mean taking the architectural form of a neural network and using it the way one might use any other building material: to construct something whose behavior you partially design and partially discover. The network does not need to have learned anything to be interesting. Sometimes what it does when left to its own random initialization is the most surprising thing it has to offer.

Several questions remain open. The two approaches could be hybridized: take a hand-designed architecture and fine-tune it on a small dataset, mixing the structural wildness of random initialization with a degree of learned realism. The weights themselves could become performance parameters, exposed as live controls that morph the filter characteristics in real time rather than staying fixed after export. And the behavior of these networks could be studied more systematically, using spectral analysis to map which frequencies they naturally emphasize or suppress, turning accidental resonances into intentional design choices rather than discovered surprises. Underlying all of this is a broader provocation: if a neural network can be a creative instrument without ever having learned anything, then the interesting question is not what the AI knows, but what the architecture affords.

---

*References: 
[RAVE: A variational autoencoder for fast and high-quality neural audio synthesis](https://arxiv.org/abs/2111.05011)
[export.py](https://github.com/acids-ircam/RAVE/blob/master/scripts/export.py) 
