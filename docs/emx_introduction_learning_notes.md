# EMX Learning Notes

Source: [Introduction to EMX](https://baikal2411.github.io/2025/04/15/Introduction-to-EMX/)

Author/site shown on source page: Liyuan Zha, "Millimeter-wave IC Design Learning Road"

Captured as study notes on 2026-07-23. This file is an attributed summary and workflow checklist, not a full mirror of the original post or its images.

Source recovery links and identifying metadata are kept in
[EMX Source Preservation Record](emx_source_preservation.md).

## Scope

The source article is a practical EMX learning record covering:

- EMX software configuration and Cadence integration
- EMX simulation setup details
- Process/substrate file preparation
- Layout EM simulation flow and black-box usage
- A lab-style inductor simulation comparison using EMX and other tools

## 1. EMX Configuration And Cadence Integration

The article uses EMX 6.1 as the reference version. The setup starts from the EMX installation/configuration directory and focuses on `emxconfig.il` plus the scripts loaded by Cadence.

Key configuration items:

- `EMX_interface_path` should point to the correct EMX interface installation path.
- Viewers for model files, mesh, current distribution, and related diagnostics should be configured explicitly.
- Temporary output and S-parameter output paths should be chosen carefully so that generated S-parameter files are not lost during cleanup.
- Black-box related configuration can be enabled when the intended flow needs black-box extraction or LVS isolation.

Cadence integration checklist:

- Add the EMX interface `.il` script load command to the Virtuoso startup `.cdsinit`.
- Load the Calibre interface when the EMX flow depends on Calibre-generated data or checking.
- Start Virtuoso and check the CIW for successful script loading and license availability.
- Confirm the process/substrate file path before running a simulation.

## 2. EMX Simulation Setup

The EMX setup form is organized around process, options, layout view, ports/signals, frequency, advanced settings, output/model generation, and cleanup.

Important setup items:

- `process`: choose the substrate/process file and inspect its cross-section. The scaled and unscaled views help catch layer-order or unit mistakes.
- `options`: set temperature, license/key related options, and black-box cell lists when needed.
- `layout view`: inspect the converted EMX layer view of the GDS/layout before trusting a run.
- `signals`: add labels or pins for ports. The source recommends pins for internal ports because label placement can create ambiguity.
- `ground`: EMX adds a ground/reference port by default; port order affects generated symbols.
- Frequency sweep: define the simulation frequency range according to the target passive device or RF block.

Advanced options noted in the article:

- Add via capacitance/inductance models if the via model only includes resistance and the accuracy target needs extra parasitic modeling.
- For boundary/internal port behavior, place ports at metal edges when possible; internal ports inject current from the top surface.
- Do not merge ports unless the intended circuit really shorts them.
- RLC fitting is commonly used when generating compact models.
- CPU count settings: `0` means a capped maximum CPU count, while `-1` requests all available server CPUs.
- Memory settings can use negative values to represent remaining memory constraints.
- Outputs can include S-parameters and Touchstone files.
- Some options can export mesh/current-distribution data for external analysis.

Post-run workflow:

- Select the correct ports and run EMX.
- Use `create model` to fit a model from the simulated S-parameters.
- Inspect the fit with a new plot before using the generated view.
- Generate symbol/schematic/s-parameter views as needed.
- The generated nport should point to the stored S-parameter data.
- Avoid cleanup settings that delete data needed by downstream Cadence testbenches.

## 3. Process And Substrate File Preparation

The article outlines how to prepare substrate/process files using PDK documentation and parasitic rule files.

Useful PDK inputs:

- Metal stack and dielectric information from process specification documents.
- Sheet resistance and thick-metal RF information from the relevant RF/process manuals.
- `.ict` files containing metal, dielectric, and via information.
- `.itf` or Calibre-related files that describe wafer/layer ordering.
- Layer map files that define layer number and purpose.

General process-file structure:

- Define scale and units first.
- Define physical layers, conductors, dielectrics, vias, and air/substrate regions.
- Keep layer definitions consistent with the layer map and PDK naming.
- Be careful with contact/via layers that may connect to different underlying layers, such as active-area contacts versus poly contacts.

Layer definition notes:

- `fill()` can filter metal dummy/fill objects. A larger fill size can ignore small dummy structures; setting it to zero keeps all layout objects.
- `merge()` can merge small vias or closely spaced features to speed simulation.
- Derived layers may be needed to separate contacts by intersection with active or poly layers.

Dielectric and conductor notes:

- Substrate material definitions include thickness, relative permittivity, permeability, resistivity, and optionally temperature coefficients.
- The vertical reference relationship between dielectric and conductor layers matters. A conductor layer and the dielectric above/below it must align with the intended stack geometry.
- Verify the generated cross-section in EMX after editing the process file.

Advanced-process notes:

- Advanced nodes may require width/spacing dependent resistance or resistivity tables.
- Metal edge-bias/width-bias effects may need to be represented.
- Via definitions can include via dimensions, resistance, and material resistivity.
- Flip-chip or inverted stack flows require special care with layer order and air-layer placement.

## 4. Layout EM Simulation And Black-Box Usage

The layout example in the source uses an inductor-like passive structure and discusses reducing unnecessary eddy-current effects by breaking or isolating unrelated AA/GT/M1 structures.

Basic passive simulation flow:

- Simplify the layout to the relevant passive structure when possible.
- Define signal ports.
- Connect substrate/reference nodes with explicit pins or labels.
- Choose process file, frequency range, and advanced parasitic options.
- Run EMX, then compare extracted inductance and Q with existing PDK/schematic models.
- Fit a compact model and inspect fit quality before using it in circuit simulation.

Using generated models in Cadence:

- Create a testbench for the passive component.
- Create a config view using Spectre as the template.
- Replace the schematic view with the generated `nport` or S-parameter view.
- Open the config view and verify that the subcircuit resolves to the nport model.

Black-box/LVS isolation flow:

- Use black-boxing to separate active circuits from passive EM simulation regions.
- When loading states, avoid overwriting or reloading model setup unless intended.
- For devices that should not be netlisted, add or edit cellview parameters carefully; the source specifically warns that the netlisting action property name must be correct.
- Preserve true differential ports. If the layout requires tying auxiliary terminals together, do not short positive and negative signal terminals directly.

## 5. Inductor Lab Takeaways

The lab portion compares EM simulation against PDK and other simulation tools for inductors around RF/mm-wave frequencies.

Practical workflow:

- Pick representative PDK inductors, such as a multi-turn inductor around the nH range and a smaller single-turn inductor around the pH range.
- Simplify/export the layout for EM simulation.
- Run EMX and export S-parameters.
- Plot inductance and Q versus frequency against PDK model simulation.
- Check both value agreement and Q trends. The source notes that inductance may match more closely than Q, and that Q discrepancy can still be credible depending on model assumptions and layout treatment.

## Quick EMX Checklist

1. Confirm EMX interface and license in Cadence.
2. Load EMX and Calibre interface scripts from `.cdsinit`.
3. Select the correct process/substrate file.
4. Inspect the scaled/unscaled cross-section.
5. Inspect the converted layout layer view.
6. Place pins for ports, especially internal ports.
7. Verify ground/reference port ordering.
8. Choose frequency sweep range and precision settings.
9. Configure via parasitics, port merge, CPU, memory, and output options.
10. Run EMX and preserve S-parameter output.
11. Fit the model and inspect fit quality.
12. Generate the Cadence view and verify nport resolution in a config testbench.
13. Use black-boxing when isolating passive EM regions from larger active circuits.

## Notes For Our PeakView/Cadence Flow

- The EMX flow is conceptually similar to the PeakView Layout EM flow we used: both need careful port definition, process stack verification, output S-parameter preservation, and Cadence nport/config integration.
- The most useful transfer into our workflow is the checklist discipline: verify stack, port order, S-parameter path, generated model view, and testbench view binding before comparing L/Q/R or circuit-level AC results.
- For generated inductors in `Codex_Lib`, keep a small metadata file next to each generated passive model recording frequency range, process/profile, port order, and S-parameter location.
