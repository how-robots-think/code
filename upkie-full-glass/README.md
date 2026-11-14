# Full glass: LQR vs MPC on Upkie

*How Robots Think.* One notebook, [`upkie_full_glass.ipynb`](upkie_full_glass.ipynb): the robot model, four
controllers, the simulation shown side by side, and the plots.

[Upkie](https://github.com/upkie/upkie), an open-source wheeled biped, carries a full glass and its goal jumps 2 m
ahead. Same robot, same linear model, same weights Q and R; only the controller changes:

| controller | peak command | gravity felt by the water | arrival (5 cm) | outcome |
|---|---|---|---|---|
| LQR | 6.64 m/s² | 30.9° off the glass axis | 3.03 s | spills |
| LQR capped at 0.7 m/s² | 0.70 m/s² | (the glass tips over) | — | falls at 2.34 s |
| LQR retuned (R = 1) | 0.66 m/s² | 4.4° | 6.97 s | stays dry |
| MPC, \|u\| ≤ 0.7 m/s² | 0.70 m/s² | 5.7° | 3.92 s | stays dry |

The water is not simulated: the glass and its water are one rigid body, and "spills" is a quasi-static estimate
(the water reaches the rim past 13.6° of felt-gravity tilt).

## Run it

- [Open in Colab](https://colab.research.google.com/github/how-robots-think/code/blob/main/upkie-full-glass/upkie_full_glass.ipynb)
  and run all cells (the first cell installs MuJoCo, SciPy, Matplotlib and mediapy). The simulation film needs
  OpenGL: pick a GPU runtime.
- Locally (Python 3.10+):

  ```bash
  pip install -r requirements.txt
  jupyter notebook upkie_full_glass.ipynb
  ```

On first run the notebook downloads Upkie's official model (`upkie/upkie_description`, pinned revision, 15 MB).
Everything runs in under 10 s on a laptop with a GPU: four 10 s simulations (2 s), then the film of the four
robots (1 s with EGL on the GPU; about 30 s in software OpenGL, shadows off).

## What is inside

- MuJoCo model built from Upkie's URDF: free base, legs held by position servos, wheels on velocity servos with
  Upkie's own law, a rigid glass of 0.35 kg on the torso. Physics at 1 kHz, balancers at 200 Hz.
- The wheeled inverted pendulum `l θ'' = g sin θ − u cos θ`, linearised and discretised exactly (T = 20 ms).
- LQR from the discrete Riccati equation; the same LQR capped after the fact; a retuned LQR.
- A condensed MPC (1 s horizon, Riccati terminal cost, |u_k| ≤ 0.7 m/s², |v_k| ≤ 3 m/s) solved every 5 ms by
  ADMM, the iteration of OSQP (Stellato et al., 2020), in 30 lines of NumPy.
- The gravity the water feels in the glass frame, `g_eff = Rᵀ(g − a)`, and a check of the MPC against the LQR law.

## Credits and licence

- Upkie by Stéphane Caron and contributors: [upkie/upkie](https://github.com/upkie/upkie) and
  [upkie/upkie_description](https://github.com/upkie/upkie_description) (Apache-2.0; wheel tyre mesh by bramesh501,
  CC BY 4.0). The balancers follow Upkie's own, reimplemented here.
- OSQP: B. Stellato, G. Banjac, P. Goulart, A. Bemporad, S. Boyd, *OSQP: an operator splitting solver for quadratic
  programs*, Mathematical Programming Computation, 2020.
- This code: Apache-2.0.
