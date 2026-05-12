# Research
Amber + Mike coding around

# Awesome Project

## How to Use

To set up a scenario, call the following function with these parameters:

simulate_beam = (beam_params=(E = Young's Modulus, ρ = density, width, height), mesh=(L = length, n_elements = # of elements), bc_type=(:cantilever, :simply_supported, OR :clamped_clamped), chains=[list of beam node values where chains are to be placed], force_params=(location_node = Node value of where transverse impulse force is, F0 = Magnitude of Impulse Force, t_imp = Impulse of force (units in seconds), static_node = Node value of where static transverse force is, F0_static = Magnitude of Static Force, ramp_duration = Time of linear increase to static load value), sim_params = (t_end = End time of simulation, n_t = time step size, n_modes = Number of modes to extract))
        

## Step 1: Setup

[Jump to Installation](#installation-guide)

[Jump to Step 1](#step-1-setup)
