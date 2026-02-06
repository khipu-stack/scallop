---
layout: ../../layouts/Bare.astro
title: "From RAVE to Chaos: Neural Audio Synthesis as Instrument Design"
date: 2026-02-05
---

# From RAVE to Chaos: Neural Audio Synthesis as Instrument Design

(prsnt draft)

## Intro

In this text, we will explore two contrasting approaches to neural audio synthesis – one rooted in training a model on a dataset (using the RAVE model), and another born from hand-writing a neural network (using a TorchScript loaded via the nn~ audio plugin). Both approaches harness neural networks to create sound, but they differ fundamentally in process and philosophy. By examining these side by side, we’ll uncover what it means to treat a neural network as an instrument and how this challenges our understanding of AI in music.

- RAVE model (Training approach): A Realtime Audio Variational autoEncoder developed at IRCAM. This approach involves preparing a dataset of sounds, training a deep network to learn that data’s patterns, and then using the trained model to generate or process audio. The motto here could be “data is composition” – the artist composes a dataset, and the network learns to mimic it.

- Hand-Written ConvNet (No-training approach): Instead of training on data, we directly code a neural network architecture (convolution layers with non-linearities) and use it as an audio processor. This is a more experimental, “hacking” approach: we leverage the same tools that normally host trained models (the nn~ external) but feed it a custom TorchScript model that has never seen data. The architecture itself becomes the instrument, and its randomly initialized weights become the parameters shaping its sound.

We’ll delve into the technical structure of the custom TorchScript (named ChaosEffect), verify our understanding of how RAVE and nn~ relate, and discuss the artistic implications of training vs. hand-coding and the agency of these neural networks in a creative context. Along the way, I’ll share code snippets, SuperCollider patch examples, and personal observations from my experiments.

## Understanding RAVE: Training a Neural Audio Model

What is RAVE? RAVE (Realtime Audio Variational autoEncoder) is a neural audio synthesis framework that learns to compress and reconstruct audio in real-time. Technically, it’s a variational autoencoder (VAE) tailored for audio: it has an encoder that converts audio to a low-dimensional latent representation and a decoder that reconstructs audio from that latent code. By training on a large dataset of sounds, a RAVE model learns a latent space of that audio domain, enabling tasks like generating new sounds, transforming input audio, or doing style transfer between sounds.

- Training Process: Training RAVE involves three main steps: (1) preparing a dataset, (2) running the training to optimize the model’s weights on that data, and (3) exporting the trained model for real-time use. In practice, one must preprocess audio into a dataset and possibly apply augmentations. For example, RAVE v2 models include an augmentation called “mute” that randomly silences portions of training audio to force the model to learn silence – an interesting detail highlighting that even silence is something a model must learn to reproduce properly. Training is typically computationally intensive and can run for many hours or days, passing through the data repeatedly (epochs) until the model converges. RAVE’s authors provide various preset architectures (v1, v2, v2_small, etc.), each suited for different tasks or GPU budgets.

- My RAVE Experiments: Last summer, with the help of Mengtian Sun, we trained a RAVE model on 1 hour of my own noise recordings. We used the v2_small configuration, which, as documented, is an architecture with a smaller receptive field and adapted for more stationary signals like noise. The training process itself was an exploratory composition: I curated an hour of diverse noise, and the neural network gradually induced a representation of that noise universe. After training, I exported the model to a TorchScript .ts file (with the streaming option for real-time to avoid artifacts) and auditioned the results.

- Results & Observations: The output of the noise-trained model was not as dramatically varied as one might hope. In fact, different input sounds processed through this RAVE model came out sounding “flattened” or homogenized – the model seemed to average out nuances and produce a somewhat uniform noise texture. This makes sense in hindsight: the VAE had learned a compressed latent representation of the entire noise dataset. It found a kind of statistical middle-ground of those noise recordings. While it could reproduce the general texture (and did capture some spectral characteristics of my noise data), it lacked the extreme dynamics or distinct identities of individual recordings. The outcome was a smooth, almost polite noise – interesting but less chaotic than the raw source material. It was as if the model ironed out the quirks in the data, giving an almost *Hi-Fi yet somewhat characterless rendition of noise.

    - Even when feeding the model very different inputs (or even just letting it generate from random latent codes), the sound stayed within a narrow band of what it had learned. This underscored a key idea: a trained model will always reflect its training data’s bias. RAVE, acting as a mimesis machine, was intelligent only in so far as it could regurgitate the essence of its training sounds. Its creativity was bounded by the recordings I gave it.

- Second Training & Artifacts: Earlier this year, We started a second RAVE training, this time using the full v2 architecture (a larger model) on a human voice dataset. Due to GPU limits, I tweaked some batch-related parameters to get it running. The training succeeded, and I got to observe the model’s evolution across checkpoints – from rough output in early epochs to more refined reconstructions later. An intriguing find was the presence of certain artifacts in the generated audio. Some sounds had a harsh, resonant edge – a kind of high-frequency buzzing or metallic ringing that wasn’t explicitly in the dataset. These artifacts, while considered “imperfections” from a fidelity standpoint, became signatures of the model’s neural nature. I noticed that these neural artifacts are reminiscent of those heard in other AI audio works (for example, the subtle digital distortion or aliasing often heard in neural vocoders). They might arise from the model’s convolutional layers or the adversarial training process. Importantly, I would later encounter similar artifacts in my hand-written network, hinting that certain sonic fingerprints are innate to convolutional neural networks, whether trained or not.

In summary, using RAVE taught me that training a neural network is as much an art as a science. The artist’s influence comes through in data curation (what sounds you choose to train on, how you augment them, etc.) and in tweaking training parameters. The result is a model that embodies the training data – it’s an instrument that knows those sounds. This brings us to an ontological point: when you train a model on a dataset, you are in effect saying “this collection of sounds is my composition, and I want an instrument that can play in that style”.

Before contrasting this with the hand-written approach, let’s introduce the tool that allows these models to sing inside our synthesizer setups: nn~.

## nn~ as a Host: Loading Models into Max/Pd/SC

nn~ is an external object (developed by the RAVE authors) that acts as a bridge between deep learning models and audio environments. It wraps LibTorch (the C++ back-end of PyTorch) for use in real-time audio software like Max/MSP, Pure Data, and SuperCollider. Essentially, nn~ is an empty shell that becomes useful only when you give it a pretrained TorchScript model.

- A TorchScript model (with .ts extension) is a serialized neural network that can run independently of Python – perfect for deployment. RAVE provides a command to export models to TorchScript, which is what I used to get my .ts files.

- nn~ loads such a model and exposes its methods as audio processing units (UGens in SuperCollider, or objects in Max/Pd). In fact, many TorchScript models can have multiple entry-point methods. For instance, a RAVE model might have separate methods for encode, decode, and an all-in-one forward (encode+decode). With nn~, you can select which method to run and it will create the appropriate number of audio inlets/outlets for that method. This multi-method capability is key to the hack.

The Hack: **While nn~ was conceived to host trained models, it doesn’t actually verify what the model is or how it was obtained.**  As long as the .ts file is a valid TorchScript network that accepts and produces audio tensors, nn~ will run it. This opens the door to a mischievously creative idea: what if we write our own neural network (in PyTorch), don’t train it on any data, but still export it as TorchScript? We could then load this untrained network into nn~ and use it like a bizarre audio effect or synth. This is exactly what I did for the second part of my exploration.

Why do this? In an AI and art context, it’s a way of questioning the standard pipeline. Instead of “collect data → train model → use model”, I tried “imagine a model → skip training → use model”. It’s a bit like circuit bending but in software: taking the neural network paradigm and bending it to serve as a direct, unpredictable sound generator. Conceptually, it shifts the focus from training (learning from the world) to topology (designing your own little world). I often think of it this way:

- Training a model is like teaching an instrument to mimic existing music (the instrument is blank initially, but after training it can play something resembling the training sounds).

- Hand-writing a model is like building a new instrument from scratch – you assemble oscillators, filters, nonlinearities (here realized as neural network layers/functions) in whatever configuration you fancy, and see what sounds it makes. There’s no prior knowledge in it; you give it form rather than it learning form from data.

Let’s dive into the specific hand-written model I created and see how it works internally.

## Hand-Writing a Neural Network Instrument (ChaosEffect)

In a Jupyter notebook (Chaos_export.ipynb), I defined a PyTorch nn.Module called ChaosEffect. This module contains a small convolutional neural network, and I gave it three different forward methods – essentially three modes of operation – named forward, topo, and chaos. After defining it, I scripted the module and saved it as a TorchScript file (e.g. test_env_topo_chaos_3.ts). The plan was to load this single .ts file in SuperCollider and be able to call any of those three methods via nn~.

Here’s a breakdown of the network architecture and the methods:

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

##### Unpack:

- Layers: I use four convolution layers. `conv_in`, `conv`, and `conv_out` all have a kernel size of 1. A 1-sample kernel essentially means these act like fully connected layers applied at each time-step – they mix channels but have no memory across time (no window). conv_in expands the input from 1 channel to operators channels. conv is an intermediate hidden layer (mapping operators → operators channels). conv_out collapses back to 1 channel output. These form a basic feedforward network operating on each audio sample (though implemented as conv1d for convenience).

- Temporal Convolution (conv_chaos): In addition, there’s conv_chaos, which has a kernel size of 5 with padding. This layer does look at a window of 5 samples (centered on the current sample) and produces operators channels output. It’s not used in the simple forward paths, but as we’ll see, it’s crucial in the chaos method to introduce a temporal feedback element.

- Weights (No Training): Normally, these conv layers would be initialized with small random weights (PyTorch’s default initialization). Since we are not going to train them, those initial weights will remain as-is during usage. I introduced a parameter w_mult (weight multiplier) to globally scale all weights at initialization time. In the code above, layer.weight *= w_mult multiplies each weight tensor by some factor. This is a hacky way to adjust the gain or influence of the layers. Think of it as if we built an analog circuit with random components; w_mult is like turning up all the interaction strengths at once. In my experiments, I ended up using operators=14 and w_mult=2 for the exported model – meaning 14 parallel channels in the network, and all weights doubled in magnitude from their initial random values. Increasing operators tends to make the network more complex (more internal signals interacting), and increasing w_mult makes the nonlinear effects more pronounced (if too high, it might blow up; if too low, the network might do almost nothing). Finding these values was a bit of trial and error, a manual “sound check” process since there’s no training to automatically tune anything.

So, before any method is called, we have essentially a random, fixed micro neural DSP: 14 channels of random linear filters (1-sample and 5-sample kernels) and we will inject nonlinearities via code in the methods.

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
The forward method takes an input tensor buffer with 3 channels: [audio, mod1, mod2]. This corresponds to feeding the network one audio signal and two control/modulation signals. Here’s the step-by-step:

1. Input splitting: x is the main audio (first channel), mod1 and mod2 are the two modulators (second and third channels). In practice, these modulators could be any audio-rate or control-rate signals we supply from SuperCollider (e.g., an LFO, an envelope, etc.).

2. Linear processing: We pass x through conv_in and then conv. These two layers mix and transform the signal in the 14-dimensional hidden space. At this point, y is a tensor of shape (batch, 14, frames). Without training, y is some quirky linear combination of the input’s recent samples – effectively a colored version of the input in 14 channels.

3. Nonlinear modulation (Sine): y = sin(y * π * mod1). Here each element of y is multiplied by π * mod1 and then we take the sine. Because mod1 is broadcasted (it’s shape [batch,1,frames] so it multiplies each of the 14 channels equally at a given time), this line is essentially a waveshaping operation. If mod1 is 1.0, this is just sin(y * π). If mod1 varies, it’s scaling the phase of the sine transformation, which can act like a kind of modulation of harmonics or an FM-like effect. For example, a larger mod1 (>1) will cause the sine to go through multiple periods even when y changes slowly, injecting higher-frequency content. So mod1 can be seen as controlling the degree of nonlinearity or harmonic intensity.

4. Nonlinear modulation (Tanh attenuation): y = 1 - tanh(|y| * mod2). This second nonlinearity uses mod2 to scale the absolute value of y, then applies a hyperbolic tangent. tanh gives an output between -1 and 1; by using 1 - tanh(...), we effectively invert it and bias it into the 0 to 2 range. But note tanh(|y| * mod2) will always be between 0 and 1 (since |y| * mod2 is non-negative), so 1 - tanh(...) will be between 0 and 1. This operation acts like a dynamic attenuation: when |y| * mod2 is large (meaning the signal is strong and mod2 is high), tanh → 1, so output goes to ~0 (silence); when y is small or mod2 is small, tanh output is small, so we get close to 1 (passing the signal). In other words, mod2 in combination with the signal’s amplitude controls a form of nonlinear gain. If mod2 is an envelope or LFO, it can gate or shape the amplitude in a nonlinear way. If mod2 is constant, say 1, then this just does 1 - tanh(|y|), a soft clipping effect that squashes loud values more than quiet ones.

5. Output layer: After these modulations, y goes through conv_out to collapse back to 1 channel. Then we divide by (operators * w_mult) as a normalization. Since we had 14 channels and scaled weights by 2, dividing by 28 keeps the output from blowing up in amplitude. It’s a rough heuristic to ensure the energy stays in a reasonable range (very much like how one might scale down a mix of signals in a modular synth).

The forward method therefore implements a neural audio effect where:

- You input a sound x and two controls mod1, mod2.

- The sound is filtered in a random linear way (conv layers), then subjected to a sine distortion controlled by mod1, and a tanh-based dynamic shaping controlled by mod2.

- The result is a processed sound output.

We can anticipate the sound: since sin and tanh are both nonlinear, the output can be rich in harmonics or in dynamic variation. With appropriate mod1 and mod2 signals, this can do things like distortion, tremolo/gating, or even weird AM/FM hybrid effects. Without any training, it’s not “intelligent” – it won’t, for example, automatically make musical decisions – but it will respond in potentially complex ways to the mod inputs. In usage, one has to play with those modulators to find sweet spots (much like turning knobs on a synth effect you don’t fully understand yet).

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

The topo method is a simplified variant of forward. It takes only 2 channels of input: one audio x and one modulator mod1. The processing is the same up to the sine nonlinearity controlled by mod1, but it omits the tanh-based shaping and goes straight to conv_out. In essence, topo is like a leaner topology:

- It does the conv_in → conv (random linear filtering),

- then one nonlinearity (sin) with a modulator,

- then outputs via conv_out.

Why have this mode? From an experimental perspective, I wanted to see what the sound is like when we remove one layer of complexity. The name topo hints at “topology” – as in, trying a different network topology. One can imagine forward as two nonlinear stages (sin then tanh) in series, whereas topo is just one nonlinear stage (sin). This might sound more stable or less chaotic, since there’s one fewer modulation happening. topo still allows mod1 to warp the harmonic content of the sound, but it doesn’t do the dynamic gating that mod2 did in forward.

Anticipating its sound: perhaps topo acts more like a strange wavefolder or FM oscillator depending on mod1. With an appropriate mod1 (say an LFO or another audio oscillator), it could produce rich spectra. But likely it’s a bit more tame than forward since loud parts aren’t being squashed by a tanh; meaning it might output hotter levels or even get more distorted if mod1 drives it crazy. That’s where the next method comes in to really push boundaries.

#### Method 3: chaos (Feedback Chaos Oscillator)
```python
    @torch.jit.export
    def chaos(self, buffer):
        x       = buffer[:,0:1]
        control = buffer[:,1:2]

        y = self.conv_in(x)
        # First feedback loop
        shift1 = self.conv_chaos(y)
        y = torch.sin(y * torch.pi + shift1 * control * 0.5)
        # Second feedback loop
        shift2 = self.conv_chaos(y)
        y = torch.sin(y * torch.pi + torch.tanh(shift2) * control * 2.0)
        y = self.conv_out(y)
        y = y / (self.operators * 0.6)
        return y.squeeze(1)
```

The chaos method is where things get wild. It expects 2 input channels: one main audio input x and one control signal. Its internal process creates a kind of feedback-feedforward hybrid using the conv_chaos layer:

It begins similarly: y = conv_in(x). So we expand the input into 14 channels.

First feedback loop: We compute shift1 = conv_chaos(y). Here conv_chaos (kernel size 5) looks at a small window of the current y (which itself is derived from the recent x around the current time, given the conv_in is just per-sample, but conv_chaos will introduce a dependency on neighbor samples). shift1 is a transformed version of y – you could think of shift1 as a kind of delayed or phase-shifted copy of y (hence the name). We then do y = sin(y * π + shift1 * control * 0.5). This means we add the shift1 signal into the phase of the sine, scaled by the control input (halved). If control is 0, this reduces to y = sin(y * π), no chaos influence (just like before). If control is nonzero, shift1 perturbs the phase. Because shift1 is derived from y itself (via conv_chaos), we effectively have a feedback: the output of the conv_chaos influences the next output through the sine. This is not a recurrent loop in the usual sense (it’s still within one time-step calculation), but since conv_chaos can incorporate slightly past samples (due to its kernel), this construction can create oscillatory or recursive behavior across time. It’s akin to a feedforward comb filter combined with a nonlinear oscillator – a recipe for chaotic dynamics.

Second feedback loop: We take the new y after that first sin and again apply conv_chaos: shift2 = conv_chaos(y). Now shift2 is another transformed version of the signal (further down the line). We feed it into another sine: y = sin(y * π + tanh(shift2) * control * 2.0). Another phase perturbation, this time using tanh(shift2) scaled by control*2. The tanh here is likely to tame the values of shift2 (keeping them in -1 to 1 range) before multiplying by control. This second loop adds yet more complexity and nonlinearity. We are essentially running two iterations of a nonlinear recurrence:

$ 
y←sin(yπ+f(y)⋅control) 
$

where $f(y)$ is some convolution of the current $y$ (the conv_chaos results). Iterating this twice makes it very sensitive to initial conditions and parameter values – hallmark of chaotic systems.

Finally, we pass y through conv_out and scale by (operators * 0.6) (note: we used 0.6 here instead of w_mult; this was an empirical adjustment for proper output level in chaos mode).

The chaos method essentially treats the network as a self-exciting oscillator or complex filter. Rather than just processing input through once, it uses the conv_chaos layer to inject a kind of memory or state from the recent past of the signal back into the present calculation. The control input acts as a knob to dial in how much of this chaotic feedback affects the output. At low control values, chaos might behave only mildly nonlinearly (maybe similar to forward or topo but with some extra grit). At high control values, we expect it to produce oscillations, possibly even on its own without a meaningful input.

It’s worth noting: if we input silence (x = 0) into a purely feedforward network like forward or topo, we’d just get silence out (assuming no bias in conv layers, which PyTorch conv layers do have a bias term by default, though we didn’t explicitly null them – something to consider: biases could make even zero input produce some output, but anyway). However, in chaos, even with x = 0, the feedback loops might start oscillating due to the conv_chaos’s internal state and the nonlinear function. In other words, chaos can potentially generate sound from silence if the internal weights cause an instability. It effectively turns the network into a strange attractor system; control might push it into oscillation.

From a design perspective, these three methods (forward, topo, chaos) were my attempt to package multiple sonic behaviors in one neural instrument. By exporting them together, I can use the same weight initialization but tap into different “circuits” within the module. It’s like having three related guitar pedals in one box: one might be distortion (forward), one a wave shaper (topo), and one a chaotic drone generator (chaos). All share the same underlying components (the conv layers and weights), but they’re wired differently in each mode.

No Training, but Not Random Forever: One might wonder, without training, do these weights just stay random? Yes – they’re fixed after initialization. However, one could imagine manually tuning some weights or applying some rule to set them. In this project, I left them as random (aside from the uniform scaling by w_mult). Part of the charm is that the network is a bit of a black box; I as the creator set up its skeleton, but the actual numeric values inside are somewhat arbitrary. This means I, too, have to discover what the network does by experimenting, similarly to how one might discover the sound of a newly built DIY synth module by plugging it in and turning knobs.

Now that we have the TorchScript ChaosEffect model ready, the next step was to incorporate it into a SuperCollider session and play it. This is where nn~ in SuperCollider (via the SC plugin SuperCollider Neural Toolkit which provides the NN class) comes into play.


## Playing the Network in SuperCollider

With the model exported (test_env_topo_chaos_3.ts), using it in SuperCollider is straightforward. First, I load the model file into SC’s NN system:

```supercollider
s.boot;
NN.load(\abc, "/path/to/test_env_topo_chaos_3.ts");
NN(\abc).methods;    // List available methods (forward, topo, chaos)
NN(\abc).describe;   // Describe I/O configuration of the model
```

The NN.load(\abc, "...ts") call assigns the model to a key \abc (an arbitrary name I chose). By querying NN(\abc).methods, I can see the three exported methods (forward, topo, chaos), confirming the model loaded correctly. Each method has its own expected number of inputs (inlets) and outputs (outlets) – for example, forward should show 3 inlets, 1 outlet; topo 2 inlets, 1 outlet; chaos 2 inlets, 1 outlet. The describe might also show the shapes or any attributes (if we had registered any buffers or constants, which in the code I did register some buffers like forward_params but those were placeholders not actively used in processing).

Now, how to make sound with it? In SC, I use NN(\abc, \method).ar(inputs, blockSize) as a UGen. That creates an audio-rate UGen running the specified method of my model, taking the given inputs. The blockSize (2048 here) is how many samples to process per block – a higher number trades more latency for efficiency (2048 samples is ~46ms at 44.1kHz, which is acceptable for non-interactive sound generation). Using a block is important because running a complex network sample-by-sample would be too slow; `nn~` uses an internal circular buffer to manage chunked processing for performance.

### Building a Modular Patch with NN~ in SC

I approached this like building a small modular synth where each neural method is a module. Here are a few configurations I tried, as reflected in my SuperCollider code:

- Topo as a standalone synth: First, I made a SynthDef that uses the topo method as an effect on a simple oscillator:

```supercollider
SynthDef(\topo, { |freq = 100, amp = 0.5, out = 0, modBus = 0|
    var input, mod, nnOutput, final, energy;
    input = Saw.ar(freq);
    mod   = K2A.ar(MouseX.kr(1, 18).lag(0.2));  // control signal from mouse X (1 to 18)
    nnOutput = NN(\abc, \topo).ar([ input, mod ], 2048);
    // measure amplitude and send to a control bus for cross-modulation
    energy = Amplitude.kr(nnOutput);
    Out.kr(modBus, energy);
    // filter and output
    final = HPF.ar(nnOutput, 20);
    final = LPF.ar(final, 15000);
    Out.ar(out, Limiter.ar(final * amp, 0.9) ! 2);
}).add;
```
This SynthDef feeds a sawtooth wave into topo. The mod for topo is derived from the horizontal mouse position (scaled 1 to 18). So as I move the mouse, I change mod1 which alters the sine distortion inside the network. The output nnOutput is then high-pass filtered (to remove any DC or subsonic rumble the net might generate) and low-pass filtered (to tame any ultra-high artifacts, essentially focusing the sound in a nice band). Finally, it goes through a limiter and out to stereo (same signal copied to both channels). I also took the amplitude of the output (Amplitude.kr) and sent that to a control bus (modBus). This is to allow dynamic interconnection – specifically, I planned to use topo’s amplitude to modulate something in the chaos synth.

Sound of topo: Depending on the freq of the saw input and the mod value, the topo network gave a range of sounds. For example, with a low freq (sub-audio rate) and varying mod, it acted somewhat like a weird distortion unit with a tremolo; with higher freq (audio rate), it produced rich, gritty tones full of sidebands (due to the sin wavefolding effect). At some settings, it even sounded percussive – e.g., if freq was modulated or if an envelope was applied, you’d get an aggressive FM drum or metallic hit kind of sound. Indeed, later I used an envelope to drive mod for percussive hits (more on that soon).

#### Chaos as a standalone synth: Similarly, I made a SynthDef for the chaos method:

```supercollider
SynthDef(\Chaos, { |baseFreq = 60, amp = 0.5, out = 0, modBus = 0|
    var input, control, nnOutput, final, follower, dynamicFreq;
    // read the modBus (amplitude from topo synth) as a control signal
    follower = In.kr(modBus, 1);
    dynamicFreq = baseFreq + (follower * 740).lag(0.5);
    input   = Saw.ar(dynamicFreq);
    control = K2A.ar(MouseY.kr(1, 38).lag(0.2));  // control from mouse Y
    nnOutput = NN(\abc, \chaos).ar([ input, control ], 2048);
    final = HPF.ar(nnOutput, 20);
    final = LPF.ar(final, 15000);
    Out.ar(out, Limiter.ar(final * amp, 0.9) ! 2);
}).add;

```

In this \Chaos synth, I did something interesting: I let the amplitude of the topo synth (from the modBus) modulate the frequency of the chaos synth’s input oscillator. dynamicFreq = baseFreq + (follower * 740) means if the topo output is loud, the chaos input pitch will sweep up by as much as ~740 Hz. This creates a cross-coupling: the more energy topo produces, the higher the pitch feeding into chaos. This was an artistic choice to make the two networks influence each other in a dynamic way (imagine a scenario where a loud burst from one network “excites” the other into a higher pitch range). The control for chaos is taken from vertical mouse (1 to 38 range). With control high, the chaos network goes into wild oscillations; with it low, chaos network behaves more gently or might even go quiet (if the input doesn’t excite it enough).

Sound of chaos: On its own, chaos can create droning, evolving textures. For instance, with a steady input tone (like a saw wave at 60 Hz) and a high control value, I got a swirling, noisy drone that sometimes locked onto unexpected tones or burst into noisy crackles – very chaotic indeed. With control at lower values, chaos produced more stable but still richly distorted tones, almost like a fuzzed-out oscillator. Because of the feedback nature, certain frequencies would get emphasized – it reminded me of a resonator where some tone would ring out strongly. With the cross-modulation from the topo synth (when they’re used together), if topo played a rhythmic pattern, chaos would kind of howl or wail in response, then settle, giving an almost call-and-response feel between the two networks.

Interconnecting topo and chaos: As described, one way I interconnected them was by the amplitude→frequency modulation via modBus. Another way is to directly chain audio: output of topo → input of chaos. I set up an audio bus for that:

```supercollier
~audioBus = Bus.audio(s, 1);
SynthDef(\topo_sender, { |freq=100, out=0, audioBus=0|
    var input = Saw.ar(freq);
    var mod   = K2A.ar(MouseX.kr(1, 28).lag(0.2));
    var nnOut = NN(\abc, \topo).ar([ input, mod ], 2048);
    Out.ar(audioBus, nnOut);             // send topo output into audioBus
    Out.ar(out, nnOut * 0.5 ! 2);        // also directly to speakers (stereo)
}).add;
SynthDef(\chaos_processor, { |amp=0.5, out=0, audioBus=0|
    var input   = In.ar(audioBus, 1);    // read from the audio bus (topo's output)
    var control = K2A.ar(MouseY.kr(1, 118).lag(0.3));
    var nnOut   = NN(\abc, \chaos).ar([ input, control ], 2048);
    var final   = LPF.ar( HPF.ar(nnOut, 20), 15000);
    Out.ar(out, Limiter.ar(final * amp, 0.9) ! 2);
}).add;
// Then play them:
~src = Synth(\topo_sender, [\freq, 110, \audioBus, ~audioBus]);
~proc = Synth(\chaos_processor, [\audioBus, ~audioBus, \amp, 0.5], target: ~src, addAction: \addAfter);

```
In this patch, topo_sender outputs its sound to both the speakers and an internal bus. chaos_processor then reads that bus as input. This configuration means the full audio output of topo is being fed through the chaos network as if the chaos module were an effect box after topo. The control for chaos is still user-controlled (MouseY here), but I gave it a larger range up to 118 to be able to really push it (since now the input is a complex signal, maybe needed stronger control for extreme effects).

Result of chaining: This was perhaps the most unpredictable experiment. Feeding one neural net’s output into another creates an opaque combined transfer function. The sound could range from heavy, almost angry distortion to sudden tonal feedback loops. I had to be cautious with levels and filtering (hence the limiter and filters) to prevent runaway loudness or speaker-damaging noise. But at moderate settings, it yielded an organic, evolving timbre – you could hear the first network’s motion (say a rhythmic pattern or a sweep) being refracted and diffused by the second network. It’s almost like two chaotic systems feeding each other: the outcome had a life of its own, often surprising me with new textures.

Musical Context – Percussive and Pad Instruments: Finally, thinking in musical terms, I wrapped these neural processes into more musical instrument definitions:

- A Percussive SynthDef using topo (\perc): I trigger a quick envelope (Env.perc) to both amplitude-modulate a saw wave and feed that envelope as mod to the topo network. Essentially, each percussion hit sends a burst of sound into topo and simultaneously tells topo to rapidly change its internal sin modulation via the envelope. The result is a short, noisy hit whose texture is shaped by the network. By adjusting the envelope length and how much it modulates topo, I could get sounds reminiscent of a kick drum (short, low thud with extra harmonics), snare-like snaps (if freq is higher and mod amount is high, introducing chaos), or even bass notes (with longer decay, it could resonate a bit).

- A Pad SynthDef using chaos (\pad): This one uses a slower ADSR envelope for amplitude, a continuous oscillating input (a sine oscillator with a slight random wobble), and a very slow random LFO to modulate the control of chaos. This effectively makes chaos drift in and out of chaotic regimes, producing an evolving drone. I also added a touch of reverb (FreeVerb) to smooth it into a pad-like texture. The pad might start as a relatively clean tone but then the chaos control rises and you hear the sound fracturing into noisy harmonics, then it settles back – giving a sense of movement. It’s unpredictable yet can be musically harnessed by controlling the ranges of the LFO and base frequency.

With these instruments, I could imagine a performance where neural networks replace oscillators and filters in a little synthesizer ensemble. One can sequence the \perc for rhythmic patterns and play the \pad for atmospheres, and even these two can interact (e.g., the pad’s chaos amount could be tied to the percussion’s activity, etc.). It's like building a band out of neural modules.

### Reflections on Using the Hand-Written Network

Working with this hand-written neural network in SC felt very different from using the trained RAVE model:

With RAVE (say I had a trained model on certain sounds), using it typically means transforming or generating audio in a somewhat controlled manner. For example, feed it a sound and it outputs a variant of that sound as learned from data, or interpolate between known sound timbres. It’s somewhat like working with a smart sampler or a very fancy effect that has a sense of an instrument (the training data genre).

With the ChaosEffect network, it was more like dealing with an unknown electronic circuit. There was a lot of tweaking and listening involved. Small changes in input or control signals could push it from a stable state to total chaos. It demanded an improvisational mindset: I would move a knob (mouse X or Y controlling mod parameters) and listen to how the network responded, then adjust accordingly. At times, it felt alive – in that it would start oscillating by itself, almost as if the network had its own will. Of course, this is just the complex deterministic math at work, but as an artist it’s easy to personify it as the network having some agency.

This leads us into the more conceptual discussion: What is the ontology of these two approaches? And do these networks have agency, or are they just black boxes we fiddle with?

## Training vs. Hand-Coding: Ontological Differences
We have seen two paradigms:

1. Training a RAVE model: where the sound emerges from data. The artist’s role is to choose/produce the right data and possibly fine-tune the training process. The result is a network whose weights encode information about the real world (or at least the recordings provided). The network, in a sense, stands on the shoulders of that dataset – it cannot create something truly outside that distribution (it interpolates or recombines what it has learned). If the dataset is considered a form of composition, then training is an act of bringing that composition to life inside the network. After training, using the model is often about exploring the latent space of the learned sounds or applying the model in contexts (like processing new audio through it). There’s a hierarchy of creation: first you have the real-world sounds (the inspiration), then the model (the abstraction), and then the output (the model’s imitation or extension of those sounds).

2. Hand-coding a network (no training): here, sound emerges from the architecture itself. There is no external dataset; the “training data” is essentially just random noise implicitly (since weights are random). The artist’s role shifts to a design process – deciding how many layers, how they connect, what nonlinear functions to include. It’s akin to designing a synthesizer or an effects chain. The network’s weights don’t represent any external knowledge; they are more like fixed hardware components. In ontological terms, this network is a self-contained entity – it doesn’t refer to or rely on any reality outside itself (no recordings of instruments, no human performances, etc.). The sounds it produces are products of its internal mathematical structure. If the trained model is an “automaton that learned from nature”, the hand-written model is an “automaton sprung purely from mind and chance”.

Another way to compare:

- In the trained scenario, the meaning of each neuron or layer is tied to the data (e.g., one part of a RAVE decoder might correspond to bass drum timbre if it learned drums, etc.). There’s a sort of semantic or at least correspondence aspect. In the hand-coded scenario, there is no semantic meaning to any neuron – they are literally just part of a signal processing graph we created. It is ontologically closer to a synthesizer or effect than to a “model of something”. We might even question: is it accurate to call an untrained network “AI” at all? Perhaps not in the strict sense, since no intelligence was obtained via learning. Yet, it uses the form of AI (the neural net) and thus invites us to treat it with the same curiosity.

From a music ontology perspective:

- Training a model can be seen as composing with samples and statistics – the composition is latent, and the performance is the model playing with those learned stats.

- Hand-coding a model is composing with circuits and algorithms – the composition is explicit in the code structure, and the performance is you tweaking that algorithm live.

Both are “neural synthesis” but one is data-driven and the other is design-driven. This distinction echoes long-standing dualities in electronic music: musique concrète (working with recorded sounds, transforming them – parallel to data-driven) versus synthesis (working with abstract generators like oscillators – parallel to design-driven). In a sense, using RAVE is like a futuristic extension of musique concrète (the network “learns the tape music” and replays a collage of it), whereas building a Chaos net is like building a new analog synth module (just with neural parts).

Neither approach is “better” – they offer different creative affordances. Training a model gives you the power of mimesis: the network can sound like a thousand birds or a symphony of gongs if that’s what you fed it, but it may also be constrained by that and hard to force into novel behaviors not in the data. Hand-writing gives you the power of algorithmic creation: you can make something truly new that doesn’t exist in nature, but you might struggle to control its behavior or make it musical, since you’re essentially dealing with raw chaos.

## Agency of the Network: Black Box or Instrument?

When dealing with these neural networks in a creative setup, a question arises: who (or what) is in control of the sound? Is the network just a tool we fully control, or does it exhibit behavior complex enough that it feels like it has its own agency?

In my SuperCollider patch, especially when linking the topo and chaos modules, I often felt like I was playing an ecosystem rather than a set of simple deterministic effects. For instance, I’d set some initial conditions (oscillator frequencies, an envelope pattern, values for the control knobs) and then trigger the synths – and the resulting sound would sometimes take on a trajectory of its own, like a feedback system finding a groove or spiraling out. This is reminiscent of how one interacts with analog modular synths: you might set up a complex patch and let it run, and it surprises you with emergent patterns. The neural networks here behaved similarly – not because they learned to (no training, remember) but because the system we built is nonlinear and complex.

So, does the network have agency? In a literal sense, no – it’s just executing code deterministically. But from a phenomenological perspective, it feels like it at times. This relates to the concept of the network as a black box vs. as a knowable filter:

- Black Box: We call a system a black box when its internal workings are opaque or too complex to easily understand. A large trained neural network is often deemed a black box because even though we have its weights, we don’t truly know what each part is doing in human terms. In our hand-written case, we do know the structure (we wrote the code!), but due to the random weights and feedback loops, predicting exactly what output will come from a given input is nontrivial. One could analytically work out small cases, but with 14 channels and nonlinear interactions, it’s practically inscrutable. Thus, while it’s not a black box by design (we have the blueprint), it behaves like one for the user. I certainly didn’t predict that certain knob combinations would produce bird-like chirps or motor-like drones – I discovered those by experimentation. To me as the artist, the network is a partially unknowable collaborator. I provoke it with inputs, it responds with sound, I adjust, and so forth. In that sense, it exhibits a form of agency in the creative process – it leads me to ideas I wouldn’t have had alone.

- Instrument or Filter: On the other hand, we can choose to view the network more mechanistically – as a complicated digital filter/resonator. After all, at its core, a convnet is doing filtering (convolutions are filters) and the nonlinears are like distortions. One could measure its frequency response, impulse response, etc., just like any effects unit. For example, if I send an impulse into topo or chaos and record the output, that is effectively the network’s impulse response (though it might be dynamic due to nonlinearities). Likely, that impulse response would show some ringing (resonances introduced by conv layers and sin feedback). In fact, the presence of sustained tones or artifacts indicates the network has resonant modes – frequencies it naturally emphasizes or even oscillates at. This is similar to how a physical resonator (like a string or a room reverb) has modes. Here those modes come from the random weight initialization: the conv filters might accidentally form a feedback loop (especially in chaos with the conv_chaos layer) that rings at a certain frequency. We did see that the chaos network, for instance, would sometimes lock into a tone – that’s a resonance of the system.

So in a technical sense, these networks are just complex filters with static coefficients (the weights) and static non-linear functions. They don’t adapt (since no learning online), so they are completely determined by their design. In that regard, they are instruments or effects, not mysterious thinking machines.

However, one important observation: Our perception and interaction make them feel agentive. When a system is sufficiently complex, we might attribute creativity or agency to it. This happens often in music tech – consider algorithmic composition systems or even just a random arpeggiator: we know it’s algorithmic, but it can surprise us and we might say “the machine came up with that riff, not me”. In the context of AI in music, this blurs the line of authorship and control.

In my patch, I gave the networks some degree of autonomy by design: e.g., the chaos network could free-run if conditions allowed. I embraced that by letting the pad instrument’s LFO drive the chaos control – basically giving the network time-varying parameters that I’m not manually controlling in real-time. The network “performed” within those bounds, and I listened and adjusted high-level things.

Black box vs. instrument also connects to how much we want to open it up. In a formal setting, one might try to analyze the network (open the black box) – for example, inspect the weights or measure how output changes with specific inputs. I didn’t do a deep analysis like that for this project (though it’s possible – e.g., looking at the conv weight spectra might reveal if it’s basically acting like a random EQ, etc.). Instead, I took an artist’s approach: treat it like a new instrument whose playing technique I must learn by doing. In doing so, I’m implicitly treating it as a somewhat black box instrument: I know how to play it (e.g., “if I increase mod1 it gets brighter, if control too high it self-oscillates, etc.”) but I’m not explicitly calculating those relationships.

So, to answer the question: Are these networks black boxes or resonant filters? They are both:

- To our ears and fingers, they can be black boxes — unpredictable, lively, requiring exploration.

- Under the hood, they are just networks of weighted sums and nonlinear functions — one could call them “digital resonators” with certain spectra, or bizarre wave-shaping filters.

For an artist, the dichotomy is not troublesome; in fact, it’s exciting. It means we can invoke a sense of the unknown (the AI magic) while still crafting the setup in a deterministic way. It’s similar to generative art practices: you write a rule system (deterministic), but the outputs can still surprise you, so it feels like the system has a say (agency) in the final creation.

## Conclusion: Reflections on AI’s Role in Music Creation

This journey from training a RAVE model to hand-coding a chaotic neural synth has highlighted a spectrum of ways AI technology can be embedded in music:

AI as Imitator vs. AI as Material: With RAVE, AI was used to imitate and reconstitute existing sounds (the intelligence lay in how well it learned the training data’s structure). With the Chaos network, AI (or rather the neural net concept) was used as raw material to build an instrument, without any prior knowledge. This expands the notion of what “AI music” can be. It’s not always about the AI learning; sometimes it’s about the artist learning the AI’s language. In one case, the AI absorbs our music; in the other, we absorb the AI’s quirks.

The Role of the Artist: In training-based work, there’s a lot of upfront work (dataset curation, training, tweaking hyperparameters). The creative gesture is in that preparatory phase and later in how you exploit the trained model. In the hand-written approach, the artist’s work is more akin to instrument crafting and immediate improvisation – you build it, and right away you’re turning knobs to make sound. It’s more direct, though potentially more chaotic. There’s also a philosophical inversion: typically we use AI to automate or generate content for us, but here I was using AI tech (neural nets) to manually craft a synth, which is almost anti-automation.

Error and Surprise: Interestingly, the so-called “errors” or artifacts in AI became creative features. The harsh artifacts from the RAVE model training – usually undesirable in a fidelity context – made me appreciate the texture that neural networks add. Similarly, the unpredictability of the hand-made net (which could be seen as a flaw if one wanted stable results) is precisely its charm in an art context. Embracing these artifacts leads to a unique aesthetic: you start to recognize a “neural sound” that isn’t analog or typical digital, but a peculiar third kind (some describe it as glitchy, spectrally complex, or organically digital). This aesthetic might be the fingerprint of neural audio synthesis, whether trained or not.

Collaborating with the Machine: Both approaches made me feel like I was collaborating with the AI. With RAVE, the collaboration was in curating the training – I feed it, it regurgitates something, and I react. With the chaos net, the collaboration was more spontaneous – I tweak, it screams or whispers, I respond. This resonates with a broader theme in AI art: the artist often cedes some control and becomes an observer or improviser alongside the algorithm. In doing so, the process can lead to outcomes neither the artist nor a fully deterministic traditional method would have produced alone.

From a learning perspective, implementing the network by hand deepened my understanding of what these layers actually do. Without the crutch of training, one must confront questions like “What does a Conv1d layer actually sound like if you don’t train it? What does a random filter bank do to audio? What does a sine nonlinearity do spectrally?” These are fascinating questions that connect signal processing with machine learning. It demystifies AI a bit: a neural net can be seen as just a complex audio effect. Conversely, it also mystifies signal processing in a new way: even simple DSP building blocks, when randomly combined, can yield strange emergent behavior. There’s a unity here between AI and traditional DSP – they’re not as separate as we think.

Moving Forward: This exploration is just a start. It opens up more ideas:

Could we hybridize the approaches? For example, take a hand-designed architecture and then train it slightly (maybe not from scratch but fine-tune it on some data) to get a mix of wild design and learned realism.

Could we make the weights dynamic or interactive? Perhaps expose some weight values as knobs to turn (imagine changing a conv layer’s weights live – essentially morphing the filter – that could be a new performance parameter).

How might we visualize or better understand the behavior of these neural instruments? Maybe use spectrum analyzers or plots to see what frequencies they emphasize, to tame or exploit those resonances intentionally.

From a conceptual art standpoint, this project challenges the notion that “AI = training on big data”. It suggests an alternate narrative: AI as raw potential, with or without data. This could be a talking point on the role of AI in creativity beyond pattern-learning – AI as an exploratory medium even without learning.

 We can compose not only with sounds, but with the very architectures that produce sounds. Whether we let the network learn from our music or we handcraft the network and learn from its output, we are effectively engaging in a dialogue with a new kind of instrument – one that straddles the line between code and sound, between control and surprise.

Neural networks not just as models to be trained, but also as playful, malleable materials for artistic creation. After all, the frontier of AI in music is not just making the computer imitate humans, but also discovering alien musics through these non-human algorithms – and sometimes, you don’t need 1000 GPU-hours to do that, just a curiosity to hack.


(References: RAVE official implementation, nn~ documentation, and code experiments.)