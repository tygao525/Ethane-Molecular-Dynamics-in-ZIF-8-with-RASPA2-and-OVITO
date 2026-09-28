# Ethane Molecular Dynamics in ZIF-8 with RASPA2 and OVITO

This repository documents an exercise: choosing a molecular loading from GCMC, running NVT molecular dynamics in RASPA2, and combining the fixed framework with the ethane trajectory in OVITO Basic for visualization and animation export.

**This is a preliminary learning example; diffusion coefficients and quantitative validation against the literature have not been completed.**

[View or download the animation](ethane_in_zif8.mp4)

## 1. Software and files

Software used: RASPA 2.0.50, and OVITO Basic 3.16.1. The RASPA version was confirmed from the simulation output.

- [RASPA2](https://github.com/iRASPA/RASPA2)
- [OVITO downloads](https://www.ovito.org/download/)

```text
.
├── README.md
├── SOURCES.md
├── inputs/                 # Six simulation inputs
├── results/                # Recorded RASPA output
├── trajectory/             # Framework and ethane trajectory
└── media/                  # OVITO screenshot and MP4
```

## 2. From GCMC to MD

The preceding GCMC exercise calculated adsorption isotherms with fluctuating molecule counts. Here, 45 ethane molecules were chosen as a fixed loading, based on the approximate average occupancy at 1 bar. A new initial configuration was generated; the final GCMC configuration was not imported.

NVT fixes molecule count, volume, and target temperature without a pressure reservoir. This is therefore not MD maintained at 1 bar. A default external pressure of zero in the output does not mean vacuum. MC configurations have no physical time axis; this MD trajectory represents time-dependent molecular motion.

## 3. Parameters and time

| Item | Setting |
|---|---|
| Framework | Fixed ZIF-8 |
| Supercell | 2 × 2 × 2 |
| Box lengths | 33.982 × 33.982 × 33.982 Å |
| Ensemble | NVT |
| Ethane molecules | 45 |
| Ethane model | Flexible two-site CH₃ united-atom model |
| Target temperature | 303 K |
| Time step | 0.0005 ps = 0.5 fs |
| MC initialization | 5,000 cycles |
| MD equilibration | 10,000 steps = 5 ps |
| MD production | 20,000 steps = 10 ps |
| Trajectory interval | 100 steps = 0.05 ps |
| Saved trajectory | 200 frames, 90 ethane sites per frame |
| van der Waals cutoff | 12 Å |

The 5,000 MC initialization cycles cannot be converted to ps. The 200 production frames are spaced by 0.05 ps; the first-to-last frame span is 199 intervals, or 9.95 ps, within a 10 ps production stage.

## 4. Prepare the inputs

The following six files in `inputs/` must be present in the simulation working directory.

| File |
|---|
| `simulation.input` |
| `ZIF_08.cif` |
| `ethane.def` |
| `pseudo_atoms.def` |
| `force_field.def` |
| `force_field_mixing_rules.def` |

The actual `simulation.input` used in this exercise:

```text
SimulationType                MolecularDynamics
NumberOfInitializationCycles  5000
NumberOfEquilibrationCycles   10000
NumberOfCycles                20000
PrintEvery                    1000
RestartFile                   no

Ensemble                      NVT
TimeStep                      0.0005

Forcefield                    Local
RemoveAtomNumberCodeFromLabel no
CutOffVDW                     12.0
ChargeMethod                  Ewald
EwaldPrecision                1e-6

Framework 0
FrameworkName                 ZIF_08
UnitCells                     2 2 2
ExternalTemperature           303.0

Movies                        yes
WriteMoviesEvery               100

Component 0 MoleculeName       ethane
            MoleculeDefinition       Local
            TranslationProbability   0.5
            RotationProbability      0.5
            ReinsertionProbability   0.5
            CreateNumberOfMolecules  45
```

The translation, rotation, and reinsertion probabilities are retained as used. They are MC move settings, not MD time steps or molecular speeds. `RestartFile no` means the run does not resume from a saved restart state; it does not protect existing output from being overwritten.

## 5. Run and check

```bash
conda create -n raspa2 --override-channels -c conda-forge raspa2
```

**Use a new directory for every new run.** In Finder, create `NVT_303K_45molecules_run02` and copy the six files from `inputs/` into it. 

Open Terminal and run the following two lines. Then type `cd ` (including the trailing space), drag the new folder from Finder into Terminal, and press Enter.

```bash
conda activate raspa2
export RASPA_DIR="$CONDA_PREFIX"
```

After confirming you are in the new directory, launch the simulation once:

```bash
simulate > run.log 2>&1
```

A quiet terminal is expected because output is redirected. Do not launch another run for this reason. The main progress and results are in the `.data` file under `Output/System_0/`; `run.log` may be short. This run ended with `Simulation finished, 0 warnings`. Control+C stops a foreground run, but cannot restore previously overwritten results.

## 6. Import and combine in OVITO Basic

### 6.1 Import ethane first

Use `File → Load File` to open the following file. It is in `trajectory/` for this example, or in `Movies/System_0/` for a new simulation.

```text
Movie_ZIF_08_2.2.2_303.000000_0.000000_component_ethane_0.pdb
```

Expect 90 sites and a timeline of `0 / 199` (200 frames). Each ethane molecule is represented by two CH₃ sites, not an all-atom model.

### 6.2 Add the fixed framework

Choose `Add modification… → Combine datasets`. Use the folder button under the modifier's `Secondary Source: External file` to load `Framework_0_initial.pdb`.

**Keep ethane as the primary source and the framework as the secondary source.** Do not replace the primary trajectory to add the framework.

Expected message: `Merged 90 existing particles with 2208 particles from frame 0 of second dataset.` The combined dataset contains 2,298 particles and retains 200 trajectory frames. The framework remains at its static frame 0.

## 7. Colors and Radius

Add the following modifiers at the top of the pipeline in order. The expressions rely on this example's ethane-first, framework-second ordering; recheck the ordering for other datasets.

1. 
   **Expression selection**: operate on `Particles`, enter `ParticleIndex < 90`, and confirm 90 / 2298 particles are selected.
2. 
   **Assign color**: color the selected ethane sites red; enable the selected-particles-only option if shown.
3. 
   **Clear selection**: keep enabled to remove selection highlighting while retaining the assigned color.
4.   
   **Compute property**: operate on `Particles`, set `Property name` to `Radius`, leave `Compute only for selected particles` unchecked, and leave `Neighbor particle expression` empty.

Enter the following in `Central expression`:

```text
ParticleIndex < 90 ? 0.7 : 0.25
```

The display radii are 0.7 Å for ethane and 0.25 Å for the framework. These are visualization properties, not changes to coordinates, bonds, or force-field parameters. The final pipeline, from top to bottom, is:

```text
Compute property (Radius)
Clear selection
Assign color
Expression selection
Combine datasets
Ethane trajectory data source
```

## 8. Playback, saving, and rendering

Use the playback button to inspect motion: red ethane sites move while the fixed framework remains stationary. Each frame represents 0.05 ps of simulation time; playback speed and video frame rate only affect viewing speed.

Use `File → Save State…` to save `ZIF8_ethane_MD_view.ovito`. This stores the pipeline and display settings but still depends on the PDB files; moving them may require relinking both sources. This repository provides pipeline reconstruction instructions rather than a machine-specific state file.

For animation export, select the desired viewport (for example, `Perspective`), open rendering settings with the camera icon, select the complete animation range (0–199), set image dimensions, background, frame rate, and the video output file, then render. Select the current frame for a still image. Exact control labels can vary by version.

A suggested first export is 1600 × 1600 pixels at 20 fps, giving approximately 10 seconds for 200 frames. **These are suggested settings, not verified encoding parameters of the included MP4.** The actual exported video is `media/ethane_in_zif8.mp4`. Relative MP4 links generally do not play inline in GitHub READMEs; use the link above to view or download it.

## 9. Recorded results and limitations

| Metric | Recorded value |
|---|---|
| Completion | `Simulation finished, 0 warnings` |
| Mean temperature | 299.75755 K |
| RASPA-reported temperature error | ±6.60112 K |
| Random seed recorded in output | 1790581044 |
| Framework atoms | 2,208 |
| Ethane sites | 90 |
| Combined particles | 2,298 |

The error is quoted as reported by RASPA, without relabeling it as a standard deviation or standard error. Individual frames need not be exactly at the 303 K target. The input does not explicitly fix the random seed, so new runs are not expected to match frame by frame.

The 10 ps production trajectory is useful for visualization practice and initial inspection, but does not by itself establish converged diffusion. Apparent boundary jumps may be periodic wrapping. MSD and diffusion analysis require correct unwrapping, molecular centers of mass, and longer sampling; playback speed is not a diffusion coefficient.
