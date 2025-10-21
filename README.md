# Exact Coset Sampling for Quantum Lattice Algorithms 

[![arXiv](https://img.shields.io/badge/arXiv-2509.12341-b31b1b.svg)](https://arxiv.org/abs/2509.12341)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/license/apache-2-0)

**Author:** [Yifan Zhang](https://yifzhang.com) (Princeton University) 

This repository accompanies the paper **“Exact Coset Sampling for Quantum Lattice Algorithms”**.  
It presents a **drop-in, provably correct** replacement for the contested **Step 9** in a recent windowed-QFT lattice algorithm with complex-Gaussian windows ([ePrint 2024/555](https://eprint.iacr.org/2024/555)). Our subroutine, **Step $9^{\dagger}$**, uses a *pair-shift difference* to cancel offsets exactly, synthesize a uniform cyclic CRT-coset of size $P$, and recover the intended modular linear relation by plain character orthogonality after a QFT.

### Updates

- [10/21/2025] Uploaded `A Note on Apon (2025)’s Comment on
Quantum Lattice Algorithms`: [Note_on_Daniel_Apon.pdf](https://yifanzhang-pro.github.io/quantum-lattice/Note_on_Daniel_Apon.pdf).

## Abstract

We replace the “domain-extension” in Step 9 of a windowed-QFT lattice algorithm with a **pair-shift difference** that (i) **coherently cancels unknown offsets**, (ii) **produces an exactly uniform** subgroup-coset of size $P$ inside $(\mathbb Z_{M_2})^n$ (with $M_2=D^2P$, $P$ odd, $\gcd(D,P)=1$), and (iii) uses the QFT to enforce $\langle \mathbf b^*, \mathbf u\rangle\equiv 0\pmod P$ **exactly** by character orthogonality.  
The unitary is reversible, uses only poly $(\log M\_2)$ gates, and preserves upstream asymptotics. A mild **residue-accessibility** condition is required only for coherent cleanup; no amplitude periodicity is invoked.

## Problem Statement

The targeted step must take an affine-structured state (with unknown offsets $\mathbf v^*$) and output a random $\mathbf u\in(\mathbb Z_{M_2})^n$ such that

$$
\langle \mathbf b^*, \mathbf u \rangle \equiv 0 \pmod P.
$$

The original "domain-extension on one coordinate" assumes global $P$-periodicity of amplitudes; **offsets violate this**, causing a **support mismatch** and breaking correctness when offsets are present.

## Our Replacement: Step $9^{\dagger}$ via Pair-Shift Difference

**Idea.** Prepare a uniform $T\in\mathbb Z_P$, implement a coherent *difference* that removes offsets, and obtain an **exact uniform superposition** over the cyclic subgroup

$$
\{ -2D^2 T \mathbf b^* \ (\bmod\ M_2) : T \in \mathbb Z_P  \} \subseteq (\mathbb Z_{M_2})^n.
$$

A subsequent $\mathrm{QFT}\_{\mathbb Z\_{M\_2}}^{\otimes n}$ enforces the constraint by annihilator orthogonality.

**Default (J-free) realization.** Harvest once (on **basis** inputs) the finite difference

$$
\Delta := \mathbf X(1)-\mathbf X(0) \equiv 2D^2 \mathbf b^* \pmod{M_2}.
$$

Then **without** touching the state-preparation on superpositions:

$$
\mathbf Z \leftarrow - T\cdot \Delta \pmod{M_2},
$$

cleanly producing the cyclic subgroup state indexed by $T$. After **coherent cleanup** that erases $T$ as a function of $(\mathbf Z \bmod P)$, the state **factors** and the QFT yields $\langle \mathbf b^*, \mathbf u\rangle\equiv 0\pmod P$ with certainty.

### High-level flow (J-free default)

```mermaid
graph TD
    A["Input registers X (post Step 8)"] --> B["Harvest Δ := X(1)-X(0) on basis calls"];
    B --> C["Prepare uniform |T⟩ over Z_P"];
    C --> D["Compute Z := - T · Δ  (mod M₂)  via reversible arithmetic"];
    D --> E["Compute T' from (Z,Δ)  per-prime inversions + CRT  (cleanup)"];
    E --> F["Zero T using T' and uncompute T'  (state now factors)"];
    F --> G["Apply QFT_{Z_{M₂}}^{⊗ n} to Z and measure"];
    G --> H["Output u with ⟨b*,u⟩ ≡ 0 (mod P)"];
````

### Why it works

* **Offset cancellation:** Differences depend only on $T$ and $\mathbf{b^{\*}}$; offsets $\mathbf{v^{\*}}$ vanish identically.
* **Exact coset:** The map $T \mapsto -2D^2T \mathbf{b^*}$ embeds $\mathbb{Z\_P}$ into $(\mathbb{Z\_{M\_2})^n}$, producing a **uniform cyclic subgroup** of order $P$.
* **Fourier support:** QFT of a uniform coset is supported on its **annihilator**; here exactly those $\mathbf u$ with $\langle \mathbf b^*,\mathbf u\rangle\equiv 0\pmod P$.
  (Geometric-series collapse uses $M\_2=D^2P\Rightarrow \frac{-2D^2}{M\_2}\equiv -\frac{2}{P}\pmod 1$.)

## Formal Interface (pre-/post-conditions)

**Preconditions**

* $M\_2=D^2 P$ with **odd** $P=\prod\_\eta p\_\eta$ and $\gcd(D,P)=1$.
* Registers realize an affine law $\mathbf X(j)\equiv 2D^2 j,\mathbf b^* + \mathbf v^* (\bmod M\_2)$.
* **Deterministic reproducible state preparation** within a run; harvest **basis** outputs $\mathbf X(0)$, $\mathbf X(1)$ once to cache $V,\Delta$.
* **Phase discipline:** All superposition-time arithmetic uses **classical reversible** adders/multipliers (no QFT-based adders); never re-run the state-preparation on superposed inputs.
* **Residue accessibility** (for cleanup only):
  for each prime $p\_\eta\mid P$, $\exists$ coordinate $i(\eta)$ with $b\_{i(\eta)}^\*\not\equiv 0\pmod{p\_\eta}$ (equivalently, $\Delta\_{i(\eta)}\not\equiv 0\pmod{p\_\eta}$).

**Postcondition**

* After cleanup and $\mathrm{QFT}\_{\mathbb{Z}_{M\_2}}^{\otimes n}$ on $\mathbf{Z}$, the measurement outcome $\mathbf u$ is **uniform** over
 ${\mathbf u: \langle \mathbf b^*,\mathbf u\rangle\equiv 0\pmod P}$ and zero elsewhere.

## Step $9^{\dagger}$: Algorithmic Summary

> **Default (J-free) path — recommended**

```
Input: affine-register block X (mod M₂); cached Δ := X(1) - X(0) (basis-harvested)

1. Prepare T ← (1/√P) ∑_{t∈Z_P} |t⟩  (e.g., per-prime QFTs and CRT wiring).
2. Compute Z ← - T · Δ  (mod M₂) via reversible double-and-add (Δ is read-only basis data).
3. Cleanup (required for interference):
   For each p_η | P:
     - Choose index i(η) with Δ_{i(η)} ≠ 0 (mod p_η) (reversible priority encoder).
     - Compute c_η ← Δ_{i(η)}^{-1} (mod p_η) by reversible extended Euclid.
     - Set T_η' ← - c_η · Z_{i(η)} (mod p_η).
   CRT-recombine T' from (T_η'), then:
     - Update T ← T - T'  (so T = 0),
     - Uncompute T' by inverting its construction from (Z,Δ).
4. Apply QFT_{Z_{M₂}}^{⊗ n} to Z; measure u ∈ (Z_{M₂})^n.

Output: u satisfies ⟨b*, u⟩ ≡ 0 (mod P), uniformly over the solution set.
```

> **Optional re-evaluation variant**. 
> Retain a label $J\equiv j\pmod P$, copy $\mathbf X(j)$ to $\mathbf Y$, coherently re-evaluate $\mathbf X(j+T)$ into $\mathbf Y$ via the *arithmetic evaluator* $U\_{\text{prep}}$ built from $(V,\Delta)$ (no phases), set $\mathbf Z:=\mathbf X-\mathbf Y$, then perform the same cleanup (recover $T$ from $(\mathbf Z,\Delta)$, zero it, uncompute).
> Both routes yield the identical $\mathbf Z$.

## Correctness Guarantee (one-paragraph proof sketch) 

Let $H=\langle -2D^2 \mathbf{b^*}\rangle \le(\mathbb{Z}\_{M\_2})^n$. 

After cleanup, the $\mathbf{Z}$-register is the **uniform superposition** over $H$ of size $P$ (injectivity from residue accessibility). The $n$-fold QFT maps a uniform coset to uniform support on the **annihilator** $H^\perp={\mathbf{u}:\langle \mathbf{b}^*,\mathbf{u}\rangle\equiv 0\pmod P}$. The quadratic envelope and offsets are confined to disjoint registers and **do not** affect support. Hence, measurement yields exactly the desired constraint, uniformly.

## Complexity & Resources

* Reversible arithmetic over $\mathbb Z\_{M\_2}$: **poly $(\log M\_2)$** gates.
* Double-and-add for $T \cdot \Delta$: $O (\log P)$ modular additions **per coordinate**.
* Cleanup per prime $p\_\eta$: one reversible inversion $O((\log{p\_\eta})^2)$ (half-GCD yields $\tilde{O} (log{p\_{\eta}})$), 
 plus reversible CRT (Garner $O({\kappa}^2)$ or product-tree $O(\kappa \log{\\kappa})$).
* QFT over $(\mathbb Z\_{M\_2})^n$: $O(n\cdot \text{poly}(\log M\_2))$.
* With an $\varepsilon\_1$-approximate single-register QFT, total-variation leakage is $O(n\varepsilon\_1)$.

## Practical Notes & Failure Modes

* **Copying basis registers is legal.** Use modular-add copy; no-cloning is not violated.
* **Do not** call the upstream state-preparation unitary on superpositions; it can imprint unwanted phases.
* **Residue accessibility fails for some primes?**
  Enforce the constraint modulo the accessible product $P'\mid P$ and handle missing primes by a change of basis or additional directions; or postselect via $\mathrm{QFT}^{-1}$ on $T$ (success $1/P$).
* **Why one-coordinate domain extension fails:** it presumes globally $P$-periodic amplitudes. Offsets break that premise; extending only one coordinate misaligns the support and cannot enforce the hyperplane.

## “Quick-Start” Checklist for Implementers

* [ ] Cache $\Delta=\mathbf X(1)-\mathbf X(0)$ from the exact **same** circuit instance used to prepare the live superposition.
* [ ] Ensure all superposition-time arithmetic is **classical reversible** (no QFT adders).
* [ ] Prepare $T$ **exactly** uniform over $\mathbb Z\_P$ (e.g., per-prime QFTs + CRT).
* [ ] Use **J-free** path unless you specifically need re-evaluation.
* [ ] Perform cleanup (recover $T$ from $(\mathbf Z,\Delta)$, zero it, uncompute).
* [ ] QFT on $\mathbf Z$ and measure $\mathbf u$; solve CRT downstream as usual.

## Glossary of Symbols

* $P=\prod\_{\eta=1}^{\kappa} p\_\eta$: product of distinct **odd** primes.
* $M\_2=D^2P$ with $\gcd(D,P)=1$.
* $\mathbf b^* \in \mathbb{Z}^n$: known direction modulo $P$ (structural vector).
* $\mathbf v^* \in \mathbb{Z}^n$: **unknown** offsets.
* $\mathbf X(j)\equiv 2D^2 j, \mathbf{b}^* + \mathbf v^* (\bmod M\_2)$.
* $\Delta := \mathbf X(1)-\mathbf X(0)\equiv 2D^2,\mathbf b^*\ (\bmod M\_2)$.
* Residue accessibility: for each $p\_\eta \mid P$, some coordinate of $\mathbf b^*$ is nonzero mod $p\_\eta$.

## Citation

If this work is useful, please cite:

```bibtex
@article{zhang2025exact,
  title   = {Exact Coset Sampling for Quantum Lattice Algorithms},
  author  = {Yifan Zhang},
  journal = {arXiv preprint arXiv:2509.12341},
  year    = {2025}
}
```

## License

This project is licensed under the **Apache 2.0** License. See the [LICENSE](https://opensource.org/license/apache-2-0) file for details.
