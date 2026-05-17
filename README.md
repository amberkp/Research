# Research
Amber + Mike coding around

# `simulate_beam` — Parameter Definitions

The function `simulate_beam` accepts a set of keyword arguments grouped into named tuples.  
All required parameters are described below. Units are SI.

---

## 1. `beam_params` (material and cross‑section)
- **`E`** : Young’s modulus of the beam material.
- **`ρ`** : Mass density of the beam material (kg/m³).
- **`width`** : Width of the rectangular cross‑section (m).  
  *The beam is assumed to have a uniform rectangular cross‑section.*
- **`height`** : Height (thickness) of the cross‑section in the direction of bending (m).

*Example:* `beam_params = (E=70e9, ρ=2700, width=0.02, height=0.005)`

---

## 2. `mesh` (spatial discretisation)
- **`L`** : Total length of the beam (m).
- **`n_elements`** : Number of finite elements used to discretise the beam.  
  *Node numbering runs from 0 (left end) to `n_elements` (right end).*

*Example:* `mesh = (L=1.0, n_elements=50)`

---

## 3. `bc_type` (boundary condition)
A symbol that selects the mechanical boundary condition at the two ends of the beam.

| Symbol                | Left end (node 0)   | Right end (node `n_elements`) |
|-----------------------|---------------------|-------------------------------|
| `:cantilever`         | Fixed (clamped)     | Free                          |
| `:simply_supported`   | Pinned (hinged)     | Pinned (hinged)               |
| `:clamped_clamped`    | Fixed (clamped)     | Fixed (clamped)               |

*Example:* `bc_type = :cantilever`

---

## 4. `chains` (Patches)
- A list (or vector) of node indices where *chains*, i.e. patches, want to be placed. 
- A chain is an internal linear chain of masses and springs oriented in the transverse direction.
- **`node`** : Beam node index of where the chain is placed.
- **`masses`** : List of masses in the chain in top down order.
- **`springs`** : List of spring constants in the chain in top down order.  

*Example:* `chains = [
    (node=21, masses=fill(10.0, 1), springs=fill(10.0, 2))
]`  (node 21 has 1 mass at 10 kg with 2 springs on either side with stiffness of 10 N/m)

---

## 5. `force_params` (loading definition)
A named tuple that describes both a **transverse impulse** and a **static transverse load** that ramps up linearly.

- **`location_node`** : Node index where the impulsive force is applied.
- **`F0`** : Magnitude of the impulse force (N).
- **`t_imp`** : Duration of the impulse (s). The force is applied as a rectangular pulse from `t=0` to `t=t_imp`.
- **`static_node`** : Node index where the static (ramped) force is applied.
- **`F0_static`** : Final magnitude of the static force (N).
- **`ramp_duration`** : Time over which the static force increases linearly from 0 to `F0_static` (s). After `ramp_duration` the force stays constant.

*Example:*  
`force_params = (location_node=25, F0=100.0, t_imp=0.01, static_node=40, F0_static=50.0, ramp_duration=0.5)`

---

## 6. `sim_params` (simulation control)
- **`t_end`** : End time of the simulation (s). The solver integrates from `t=0` to `t_end`.
- **`n_t`** : Number of time steps (integer). Time step size is set by `t_end / n_t`.
- **`n_modes`** : Number of vibration modes to include in the modal superposition (integer).  

*Example:* `sim_params = (t_end=2.0, n_t=2000, n_modes=20)`

---

# A Validation Case For Cantilever Beam

## No Chains In Beam (Only Static Force)
```julia
simulate_beam(
    beam_params = (E=210e9, ρ=7800.0, width=0.05, height=0.05),
    mesh = (L=1.0, n_elem=1000),
    bc_type = :cantilever,
    chains = [],
    force_params = (location_node=101, F0=0.0, t_imp=0.005,static_node=1001, F_static=10.0, ramp_duration=0.2),
    sim_params = (t_end=0.5, n_t=500, n_modes=50)
)
```
## One Overly Stiff Chain With Tiny Mass at Free End (Only Static Force)
```julia
chains = [
    (node=1001, masses=fill(0.0005, 1), springs=fill(1000000.0, 2)),
]

stiff = simulate_beam(
    beam_params = (E=210e9, ρ=7800.0, width=0.05, height=0.05),
    mesh = (L=1.0, n_elem=1000),
    bc_type = :cantilever,
    chains = chains,
    force_params = (location_node=101, F0=0.0, t_imp=0.005,static_node=1001, F_static=10.0, ramp_duration=0.2),
    sim_params = (t_end=0.5, n_t=500, n_modes=50)   
)
```
## Euler Bernoulli Theory
When the beam is configured as a cantilever (`bc_type = :cantilever`) and a static force is applied at the free end (`static_node = n_elements`), the well‑known formula for the maximum static deflection is used as a cross‑check:

$$
\delta_{\text{tip}} = \frac{P L^3}{3 E I}
$$

where  
- \(P\) = static force magnitude (N), i.e., `F0_static`  
- \(L\) = beam length (m), i.e., `mesh.L`  
- \(E\) = Young’s modulus (Pa), i.e., `beam_params.E`  
- \(I\) = second moment of area of the rectangular cross‑section (m⁴), calculated as  

$$
I = \frac{\text{width} \cdot \text{height}^3}{12}
$$

This analytical deflection can be compared with the quasi‑static displacement obtained from the simulation after the ramp has settled, providing a basic verification of the model’s stiffness.

**Substituting the validation case numbers**  
(beam: width = 0.05 m, height = 0.05 m, E = $210\times10^9$ Pa, L = 1.0 m, P = 10 N):

$$
I = \frac{0.05 \times (0.05)^3}{12} = 5.20833\times10^{-7}\ \text{m}^4
$$

$$
\delta_{\text{tip}} = \frac{10 \times (1.0)^3}{3 \times (210\times10^9) \times 5.20833\times10^{-7}}
\approx \frac{10}{328125} \approx 3.0476\times10^{-5}\ \text{m}
$$

$$
\boxed{\delta_{\text{tip}} \approx 0.0305\ \text{mm}}
$$

For the pure beam simulation (no chains), the modal‑transient solution with a 0.2 s ramp yielded a maximum tip displacement of **0.029032 mm**. For the coupled beam with an over‑stiff chain at node 1001, the maximum tip displacement was **0.029098 mm**, showing the chain has only a marginal influence on the global stiffness, as expected.
