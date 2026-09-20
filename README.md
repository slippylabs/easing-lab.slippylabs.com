# Easing & Tween Lab

Design a motion curve — a draggable cubic-bezier editor, the 31 classic Penner easings, and a real spring — then export it as JavaScript, GLSL, C#, GDScript or CSS. Runs entirely in your browser.

**Live:** <https://easing-lab.slippylabs.com/>

## What it does

- A cubic-bezier editor with both handles draggable, ten starting points (CSS `ease`, Material, anticipate, overshoot) and live numbers.
- All 31 classic easings, grouped by family, each with a note on what it is for.
- A spring model with stiffness, damping, mass and initial velocity — under-, critically and overdamped, with the damping ratio named on screen.
- Four motion previews (position, scale, rotation, fade) driven by the same curve, because a curve is judged by eye and not on a graph.
- Export as real code in five languages, plus a 64-sample table.

## How it works

A CSS-style easing curve is a bezier in (x, y) where x is *time*, so evaluating it means solving X(s) = t for the parameter s and then reading Y(s). That solve is the only hard part: Newton's method converges in a few steps but wanders off a flat spot, so it is trusted only while the slope is meaningful and bisection finishes the job.

The classic easings are written once per family as the "in" curve; "out" is the mirror through (0.5, 0.5) and "inOut" is the two halves stitched. A hand-written "out" is the classic place for a sign error nobody notices.

The spring is the closed-form solution of `m·x'' + c·x' + k·x = 0`, split on the damping ratio into three regimes — the critically damped case is a genuinely separate formula, not a limit, because at ζ = 1 the underdamped form divides by a zero frequency. With no damping at all the spring never settles, and the page says so rather than inventing a duration.

## Verification

`verify_easing.py` checks three independent things:

- The bezier solve against **bisection to 1e-12** on the exact Bernstein polynomial, over 73 control-point sets × 201 samples. Worst deviation where the derivative is meaningful: **1.4e-11**. Where X′(s) = 0 — `cubic-bezier(1,0,0,1)` has X′ = 3(2s−1)², exactly zero at s = 0.5 — the parameter is not recoverable by any method, so the test asserts the defining property instead: the returned value must be Y(s) for *some* s whose X(s) is t to within float noise.
- The 31 easings against **Penner's published equations**, rewritten from the definitions rather than ported. Worst deviation: **2.3e-15**.
- The spring against **scipy integrating the same ODE** (DOP853, rtol 1e-11) over nine parameter sets. Worst deviation: **3.0e-09**. A control runs explicit Euler at 60 Hz on the same system and deviates by 0.071, so "it matches an integrator" is not a claim about nothing.

The CSS `linear()` export is resampled and compared to the curve it approximates, and every exported JavaScript body is executed in Node and checked against the reference. **15,417 checks.**
