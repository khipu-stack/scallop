---
layout: ../../layouts/RawLayout.astro
title: "torch.nn"
date: 2026-02-01

---





```python
class ModelSynth(torch.nn.Module):
    def __init__(self, operators=16, w_mult=1, kernel=1):
        super().__init__()
        self.operators = operators
        self.w_mult = w_mult
        self.conv_in = torch.nn.Conv1d(1, operators, kernel_size=kernel)
        self.conv = torch.nn.Conv1d(operators, operators, kernel_size=kernel)
        self.conv_out = torch.nn.Conv1d(operators, 1, kernel_size=kernel)
        layers = [self.conv_in, self.conv, self.conv_out]
        for l in layers:
            l.weight = torch.nn.Parameter(l.weight*w_mult)
    @torch.jit.export
    def forward(self, buffer):
        x = buffer[:,0]
        mod1 = buffer[:,1]
        mod2 = buffer[:,2]
        y = self.conv_in(x)
        
        y = self.conv(y)
        y = torch.sin(y*torch.pi*mod1)
        y =1-torch.tanh(abs(y)*mod2)
        
        # mod2 = self.conv_in(mod2)
        # mod2 = torch.sin(torch.pi*mod2)
        # y *=mod2
        
        y = self.conv_out(y)
        y /=(self.operators * self.w_mult)
    
        return y

```
