# ContextualConv

[![PyPI version](https://img.shields.io/pypi/v/contextual-conv)](https://pypi.org/project/contextual-conv/)
[![CI](https://github.com/abbassix/ContextualConv/actions/workflows/test.yml/badge.svg?branch=main)](https://github.com/abbassix/ContextualConv/actions/workflows/test.yml)
[![Docs](https://readthedocs.org/projects/contextualconv/badge/?version=latest)](https://contextualconv.readthedocs.io/en/latest/)
[![Coverage](https://img.shields.io/codecov/c/github/abbassix/ContextualConv/main.svg?style=flat)](https://codecov.io/gh/abbassix/ContextualConv)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

> **ContextualConv** – PyTorch convolutional layers with **global context conditioning**: per‑channel **bias**, **scale**, or **modulated FiLM-style scaling**.

---

## 🚀 Quick start

```python
from contextual_conv import ContextualConv2d
import torch

# FiLM‑style (scale + bias)
conv = ContextualConv2d(
    in_channels=16,
    out_channels=32,
    kernel_size=3,
    padding=1,
    context_dim=10,   # size of global vector c
    h_dim=64,         # optional MLP hidden dim
    use_scale=True,   # γ(c)
    use_bias=True,    # β(c)
    scale_mode="film" # or "scale"
)

x = torch.randn(8, 16, 32, 32)  # feature map
c = torch.randn(8, 10)          # context vector

out = conv(x, c)  # shape: (8, 32, 32, 32)
```

### Modes at a glance

| `use_scale` | `use_bias` | `scale_mode` | Behaviour |
|-------------|------------|--------------|-----------|
| `False`     | `True`     | –            | **Contextual bias** only |
| `True`      | `False`    | `"scale"`    | **Scale only**: `out * γ` |
| `True`      | `True`     | `"film"`     | **FiLM**: `out * (1 + γ) + β` |
| `True`      | `True`     | `"scale"`    | **Scale + shift**: `out * γ + β` |
| `False`     | `False`    | –            | **Plain convolution** (no modulation) |

If `context_dim` is provided, at least one of `use_scale` or `use_bias` must be `True`.

---

## 🔧 Key features

* ⚙️ **Drop‑in replacement** for `nn.Conv1d` / `nn.Conv2d`  
  → Same arguments + optional context options.
* 🧠 **Global vector conditioning** via learnable γ(c) and/or β(c)
* 🔀 **Modulation modes**:
  * `scale_mode="film"`: `out * (1 + γ)`
  * `scale_mode="scale"`: `out * γ`
* 🪶 **Lightweight** – one small MLP (or single `Linear`) per layer
* 🧑‍🔬 **FiLM ready** – reproduce Feature‑wise Linear Modulation with two lines
* 🧩 **Modular** – combine with any architecture, works on CPU / GPU
* 📤 **Infer context vectors** from unmodulated outputs with `.infer_context()`
* ✅ **Unit‑tested** and documented

---

## 📦 Installation

```bash
pip install contextual-conv  # version 0.6.3 on PyPI
```

Or install from source:

```bash
git clone https://github.com/abbassix/ContextualConv.git
cd ContextualConv
pip install -e .[dev]
```

---

## 📐 Context vector details

* Shape: **`(B, context_dim)`**  
  (one global descriptor per sample – class label embedding, latent code, etc.)
* Processed by a **`ContextProcessor`**:
  * `Linear(context_dim, out_dim)` *(bias‑only / scale‑only)*
  * Small **MLP** if `h_dim` is set.
* Output dims:
  * `out_channels` → bias **or** scale
  * `2 × out_channels` → FiLM (scale + bias)

---

## 🔎 Context inference

You can extract the context vector inferred from the output using:

```python
context = conv.infer_context(x)
```

To also get the **unmodulated output** from the convolution layer:

```python
context, raw_out = conv.infer_context(x, return_raw_output=True)
```

This is useful when you need both the input’s context and its original unmodulated features.

---

## 🧪 Running tests

Run the full test suite with coverage:

```bash
pytest --cov=contextual_conv --cov-report=term-missing
```

---

## 📘 Documentation

Full API reference & tutorials: **<https://contextualconv.readthedocs.io>**

---

## 🤝 Contributing

Bug reports, feature requests, and PRs are welcome! See `CONTRIBUTING.md`.

---

## 📄 License

GNU GPLv3 – see `LICENSE` file for details.
