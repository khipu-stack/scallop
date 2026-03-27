---
layout: ../../layouts/DetailLayout.astro
title: "torch.nn"
date: 2026-02-01

---

# TorchScript Export — Source Notebooks

Full source for the neural network experiments referenced in [From RAVE to Chaos](/notes/rave2chaos).

---

## ModelSynth (prototype)

`TorchScript_export-test.ipynb` : the first version, used to verify the basic pipeline: define a module, freeze random weights, export as TorchScript, load into nn~.

```python
import torch

class ModelSynth(torch.nn.Module):
    def __init__(self, operators=7, w_mult=1, kernel=1):
        super().__init__()
        self.operators = operators
        self.w_mult = w_mult
        self.conv_in  = torch.nn.Conv1d(1, operators, kernel_size=kernel)
        self.conv     = torch.nn.Conv1d(operators, operators, kernel_size=kernel)
        self.conv_out = torch.nn.Conv1d(operators, 1, kernel_size=kernel)

        for l in [self.conv_in, self.conv, self.conv_out]:
            l.weight = torch.nn.Parameter(l.weight * w_mult)

    @torch.jit.export
    def forward(self, buffer):
        x    = buffer[:, 0]
        mod1 = buffer[:, 1]
        mod2 = buffer[:, 2]
        y = self.conv_in(x)
        y = self.conv(y)
        y = torch.sin(y * torch.pi * mod1)
        y = 1 - torch.tanh(abs(y) * mod2)
        y = self.conv_out(y)
        return y


synth = ModelSynth(w_mult=7)
synth.register_buffer("forward_params", torch.tensor([3, 1, 1, 1]))

scripted_model = torch.jit.script(synth)
torch.jit.save(scripted_model, "test3-3.ts")
```

---

## ChaosEffect

`Chaos_export.ipynb` : the developed version, adding `conv_chaos` (kernel_size=5) for local temporal correlation, three methods for different signal paths, and output normalization.

```python
import torch

class ChaosEffect(torch.nn.Module):
    def __init__(self, operators=4, w_mult=1, kernel=1):
        super().__init__()
        self.operators = operators
        self.w_mult = w_mult

        self.conv_in    = torch.nn.Conv1d(1, operators, kernel_size=kernel)
        self.conv       = torch.nn.Conv1d(operators, operators, kernel_size=kernel)
        self.conv_out   = torch.nn.Conv1d(operators, 1, kernel_size=kernel)
        self.conv_chaos = torch.nn.Conv1d(operators, operators, kernel_size=5, padding=2)

        for l in [self.conv_in, self.conv, self.conv_out, self.conv_chaos]:
            with torch.no_grad():
                l.weight = torch.nn.Parameter(l.weight * w_mult)

    @torch.jit.export
    def forward(self, buffer):
        x    = buffer[:, 0:1]
        mod1 = buffer[:, 1:2]
        mod2 = buffer[:, 2:3]
        y = self.conv_in(x)
        y = self.conv(y)
        y = torch.sin(y * torch.pi * mod1)
        y = 1 - torch.tanh(abs(y) * mod2)
        y = self.conv_out(y)
        y /= (self.operators * self.w_mult)
        return y.squeeze(1)

    @torch.jit.export
    def topo(self, buffer):
        x    = buffer[:, 0:1]
        mod1 = buffer[:, 1:2]
        y = self.conv_in(x)
        y = self.conv(y)
        y = torch.sin(y * torch.pi * mod1)
        y = self.conv_out(y)
        y /= (self.operators * self.w_mult)
        return y.squeeze(1)

    @torch.jit.export
    def chaos(self, buffer):
        x       = buffer[:, 0:1]
        control = buffer[:, 1:2]
        y = self.conv_in(x)
        shift1 = self.conv_chaos(y)
        y = torch.sin(y * torch.pi + shift1 * control * 0.5)
        shift2 = self.conv_chaos(y)
        y = torch.sin(y * torch.pi + torch.tanh(shift2) * control * 2.0)
        y = self.conv_out(y)
        y = y / (self.operators * 0.6)
        return y.squeeze(1)


chaos = ChaosEffect(operators=14, w_mult=2)

chaos.register_buffer("forward_params", torch.tensor([3, 1, 1, 1]))
chaos.register_buffer("topo_params",    torch.tensor([2, 1, 1, 1]))
chaos.register_buffer("chaos_params",   torch.tensor([2, 1, 1, 1]))

scripted_model = torch.jit.script(chaos)
torch.jit.save(scripted_model, "test_env_topo_chaos_3.ts")
```
