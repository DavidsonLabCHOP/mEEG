Welcome to mEEG README file!! 

**mEEG** is a MATLAB GUI-based pipeline for EEG preprocessing, qEEG analysis, spike analysis, spike-wave-discharge (SWD) analysis, event detection, and connectivity/coherence visualization. This README is intended to serve as the central place to document version-specific changes, known fixes, output behavior, and future updates.


Current tracked version: **mEEG v1.103**  
Last README update: **2026-06-24**

---

## 1. Main analysis modules

mEEG currently includes the following major analysis areas:

- **Project setup / manifest building**: defines animals, groups, input files, electrode/brain-area labels, sampling information, and recording start time.
- **Preprocessing**: performs channel/epoch quality control, bad-epoch detection, and saves QC summaries and figures.
- **qEEG**: computes normalized band power, PSD/AUC summaries, correlation, coherence, and coherence pair-spectrum outputs.
- **Spike Analysis**: detects spike-like events and summarizes single-lead and multilead spike activity.
- **Spike Viewer**: visualizes detected spikes for selected animals/leads.
- **SWD Analysis**: detects spike-wave-discharge-like events using the SWD Identifier Lite workflow.
- **SWD Viewer**: supports review of SWD candidate events.
- **Event Detection**: beta event-detection workflow for candidate event detection and summary plotting.
- **Connectivity / Coherence visualizations**: includes matrix and 3D/region-based visualization tools for correlation and coherence outputs.

---

## 2. Supported data layouts

### 10-channel mEEG/Intan-style projects

The original/default workflow supports fixed 10-lead projects with canonical brain-region labels such as motor, barrel, hippocampus, auditory, and visual, including left/right variants.

### EDF projects with variable electrode counts

The EDF pipeline is intended to support **any EDF project with 1-10 electrodes**. User-assigned labels should be respected throughout the app, including labels such as:

- `Motor`
- `Motor (L)`
- `Motor (R)`
- `Auditory (L)`
- `Visual (R)`
- `Area A`
- `Area B`
- and any other region options offered during project setup.

Important behavior added in recent versions: GUI dropdowns, plots, qEEG region summaries, Spike Viewer lead lists, and Event/Spike plotting should use the current project's actual assigned labels rather than assuming the original 10-channel template.

---

## 3. Standard qEEG frequency bands

As of v98+, qEEG band boundaries are standardized to:

| Band | Frequency range |
|---|---:|
| Delta | 0.4-4 Hz |
| Theta | 4-8 Hz |
| Alpha | 8-13 Hz |
| Beta | 13-30 Hz |
| Gamma | 30-50 Hz |
| Fast gamma | 50-100 Hz |

These boundaries should be used consistently across normalized band power, PSD/AUC, correlation, coherence band summaries, ratio lookups, and PSD GUI shading.

---

## 4. Day/night timing rules

Day/night classification should never assume that the recording starts at midnight. Event clock time should be calculated using the recording `StartTime` stored in the project/manifest:

```text
clock hour = StartTime + event time from recording start
```

For example, if `StartTime = 11` and the detected event occurs 2 hours after recording start, the event should be treated as occurring at 13:00. With day settings of 6.5-18.5, that event is classified as day.

The qEEG Settings GUI day/night values should be used for qEEG postprocessing and coherence pair-spectrum outputs. Spike, SWD, and Event Detection should also use StartTime-aware day/night annotation.

---

## 5. Parallel processing behavior

Recent versions use animal-level parallelization for qEEG, Spike Analysis, SWD Analysis, and Event Detection where possible. Because several MATLAB functions are not supported on thread-based workers, Spike/SWD/Event parallel runs should use a **process-based parallel pool**, not a thread-based pool.

This avoids known MATLAB worker errors involving:

- `designfilt`
- `ishandle`
- saving `-v7.3` MAT-files

---

## 6. Logging behavior

Recent versions add text logs for major processes. Logs should record selected settings, output folders, run progress, and error/warning messages.

Typical log file patterns include:

- `qEEG_log_[DATETIME].txt`
- `SpikeAnalysis_log_[DATETIME].txt`
- `SWDAnalysis_log_[DATETIME].txt`
- `EventDetection_log_[DATETIME].txt`

Future updates should continue adding important settings to these logs, especially parameters that affect manuscript reproducibility.

---

## 7. Version changelog

### mEEG v103 patch notes

Base: v102 spike/event EDF viewer plot fix.

Changes:
1. Event Viewer now has an Event type filter, including mixed abnormality.
2. Event Detection now labels events as mixed abnormality when two or more feature families exceed their thresholds within the merged event: RMS, line length, spike-rate, and rhythmicity.
3. Event Detection event tables now include PrimaryReason and FeatureCount columns for mixed events.
4. Event Analysis mixed abnormality filtering now works for newly generated mixed events and also for older event tables when multiple z-score feature columns exceed the current GUI thresholds.
5. Event Viewer plots now show only the raw trace. The blue filtered display trace was removed.
6. Fixed the Spike Analysis Save plot and data syntax error in core/meeg_plot_spike_group_counts.m caused by the invalid MATLAB operator !=.

Toolbar note:
MATLAB's built-in UIAxes toolbar is top-right anchored. R2026a exposes ToolbarLocation for inside/outside/container placement, but not left-vs-right anchoring, so this patch does not attempt an unsupported left-corner move.


### v102 - Spike/Event EDF viewer and one-electrode plot fix

Built on v101.

Changes:

- Spike Viewer lead dropdown now uses the current project or saved spike-output lead labels instead of the fixed 10-lead template.
- For one-electrode EDF projects assigned to `Area A`, the Spike Viewer should show `Area A` rather than legacy labels such as `motor_L` or `motor_R`.
- Spike Viewer maps displayed lead labels back to the correct channel index dynamically.
- Supported labels include `Area A`, `Area B`, `Motor`, `Motor (L)`, `Motor (R)`, and other EDF-assigned labels.
- Spike Analysis GUI preview no longer uses fragile MATLAB grouped-bar handles that can collapse to `GraphicsPlaceholder` objects in one-region datasets.
- Spike Analysis open/save plotting was updated to use the manual grouped-bar helper.
- Event Analysis preview/open/save plotting was updated to use the same manual grouped-bar helper.
- Fixes the EDF plotting error: `Unrecognized property 'FaceColor' for class 'matlab.graphics.GraphicsPlaceholder'`.
- No detection or analysis math was changed in this version.

### v101 - SWD settings layout adjustment

Built on v100.

Changes:

- Reduced the height of the **Advanced detection settings** box in the Spike-Wave-Discharge Settings subtab by about 25%.
- Moved the **Run SWD Identifier Lite** button higher and kept it directly below the Advanced detection settings box.
- Reduced vertical spacing in the SWD settings panel so the run button remains visible on smaller screens.
- Layout-only patch; no analysis logic was changed.

### v100 - Variable-electrode EDF support and process-pool fix

Built on v99.

Changes:

- Improved EDF support for projects with 1-10 electrodes.
- Replaced fragile MATLAB grouped-bar plotting in normalized band-power plots with a manual grouped-bar helper.
- Fixed cases where one-region EDF plots showed two groups but colored/labeled them incorrectly.
- Applied plotting improvements to auto-saved `NormBandPower_RegionComparison_*`, `PerRegionNormBandPower_*`, Normalized Band Power GUI preview, opened/saved band comparison plots, and Power Spectra fractional mode.
- Added `core/meeg_pretty_region_label.m` for user-friendly display of assigned electrode labels.
- Preserved labels such as `Motor`, `Motor (L)`, `Motor (R)`, `Area A`, and `Area B` across qEEG and plotting workflows.
- Added `core/meeg_ensure_process_pool.m`.
- Spike Analysis, SWD Analysis, and Event Detection now switch away from thread-based pools and use process-based workers when parallelized.
- Fixed MATLAB thread-worker errors involving `designfilt`, `ishandle`, and saving `-v7.3` MAT-files.
- Added explicit group color handling for WT/Wildtype and Disease/KO groups.

### v99 - Coherence day/night boundary and one-electrode qEEG postprocessing

Built on v98.

Changes:

- `_cohSpec.mat` pair-spectrum coherence output no longer uses a hardcoded 6.5-18.5 day boundary.
- qEEG GUI `dayStart` and `dayEnd` settings are passed into coherence processing.
- `_cohSpec.mat` now stores `cohSpecDayStart` and `cohSpecDayEnd` metadata.
- `_coh_daynight.xlsx` uses the same GUI-selected day/night boundaries.
- qEEG StartTime parsing uses `meeg_manifest_get_start_hour` for consistency.
- Restored visibility of the pair-spectrum coherence checkbox for the Pair Explorer workflow.
- Improved qEEG postprocessing for one-electrode EDF datasets.
- PSD postprocessing preserves singleton region dimensions.
- PSD overview figures support variable electrode counts, including one-electrode/one-region EDF projects.
- BandAnalysis auto-figures avoid `GraphicsPlaceholder`/`FaceColor` errors in one-region or one-group datasets.
- `Area A-E` labels are handled more cleanly in PSD region collection.

### v98 - Correlation bad-epoch handling and standardized qEEG bands

Built on v97.

Changes:

- Correlation now uses a separate NaN-masked signal copy.
- Bad 5-second epochs are set to `NaN` for the affected channel instead of contributing zero-filled samples.
- Pairwise correlations now use only samples where both channels are valid.
- Correlation bandpass filtering preserves the bad-epoch NaN mask after filtering.
- qEEG frequency bands were standardized to delta 0.4-4 Hz, theta 4-8 Hz, alpha 8-13 Hz, beta 13-30 Hz, gamma 30-50 Hz, and fast gamma 50-100 Hz.
- Updated normalized band power, correlation bandpass, coherence band means, PSD AUC/fractional AUC, ratio AUC lookup, PSD GUI shading, and autosaved PSD overview shading.
- Lower PSD overview filename changed from `PowerSpecDens_0to25Hz_*` to `PowerSpecDens_0to30Hz_*`.

### v97 - GUI busy lock and StartTime-aware day/night annotation

Built on v96.

Changes:

- Added a GUI busy lock for Spike Analysis, SWD Analysis, and Event Detection.
- During long runs, visible GUI controls are disabled/greyed out and restored after completion or error.
- Added/used `core/meeg_manifest_get_start_hour.m` as the shared recording-start-time parser.
- Numeric `StartTime` values are treated as hours of day; for example, `StartTime = 11` means 11:00 AM.
- Fractional numeric values can represent fraction-of-day times.
- Excel-style datetime serials use their fractional time component when applicable.
- Spike, SWD, and Event Detection use `clock hour = manifest StartTime + event seconds/3600`, modulo 24.
- Spike browser/list helpers use the same parser so existing spike files can be interpreted relative to manifest StartTime.

### v96 - qEEG tab behavior, logs, correlation stability, and parallelization

Built on v95.

Changes:

- qEEG main tab no longer automatically loads Normalized Band Power data just by being clicked.
- Normalized Band Power waits for the user to click **Load project data**.
- GUI preview loading no longer regenerates or re-saves BandAnalysis auto-plots.
- BandAnalysis auto-saved plots are generated during actual qEEG run, not during tab navigation.
- Added process logs for qEEG, preprocessing, Spike Analysis, SWD Analysis, and Event Detection.
- Replaced unstable direct-form `butter + filtfilt` bandpass used for correlation with a more stable FFT-domain bandpass.
- Correlation remains time-domain waveform correlation after bandpass filtering.
- Added pairwise-safe correlation calculations so one bad/flat lead does not blank the whole matrix.
- Improved SWD/Event plot legend handling so multiple groups appear correctly.
- Added animal-level parallelization support for Spike, SWD, and Event Detection.

### v95 - MATLAB-safe lead-label sanitization

Built on v94.

Changes:

- Added `core/meeg_sanitize_lead_label.m`.
- Fixed Mac MATLAB errors caused by labels such as `auditory.R` being used as MATLAB struct field names.
- Labels such as `auditory.R`, `auditory-R`, `auditory (R)`, and `auditory R` are normalized to MATLAB-safe underscore forms such as `auditory_R` internally.
- Legacy compact labels such as `Rhip`, `Lhip`, `Rmot`, and `Lmot` are mapped to descriptive labels such as `hippocampus_R` and `motor_L`.
- qEEG sanitizes channel labels before creating PSD structs or writing sheet names.
- Saved/loaded projects with old dot/dash labels are migrated automatically.
- EDF label sanitization now uses the same global sanitizer.

### v94 - Baseline version for this changelog

v94 is the baseline version immediately before the v95-v102 patch series documented here. It ran on Windows/MATLAB R2026a but exposed Mac/MATLAB R2025b label issues and several variable-EDF plotting/processing edge cases that were fixed in later versions.

### Raw Data Viewer / Annotator v75

Adds a new main Workspace module: Raw Viewer.

Features:
- Loads animals from the current project.animals list.
- Displays raw/cache .dat traces one animal at a time.
- Supports multiple visible leads and vertical trace offsets.
- Supports start time, window length, notch option, and display downsampling for faster plotting.
- Allows manual event annotation with event type, start/end time, lead label, and notes.
- Saves annotations per animal under Results qEEG/Raw Data Viewer as XLSX and CSV.
- Annotations are not deleted when excluded; they use UserIncluded=true/false.
- Current-window annotations are overlaid on the trace view.

This is the first scaffolding pass and is intentionally independent from the Spike and SWD event tables. Later patches can add import/export into curated Spike/SWD lists.


### v30 Added and refined the Connectivity Plots style: 3D mouse brain network.

Highlights:
- approximate 3D mouse brain shell
- region nodes placed for motor, visual, auditory, barrel, hippocampus, medial prefrontal, retrosplenial, parietal, cingulate, thalamus, striatum, and Area A-E
- left/right versions now supported for cortical labels and for medial prefrontal, retrosplenial, parietal, cingulate, thalamus, and striatum
- negative x coordinates are used for left hemisphere labels and positive x coordinates for right hemisphere labels
- only labels present in the current dataset are plotted
- network edges drawn as curved 3D lines so inter-hemisphere connections can be visualized
- the default view is now a slightly tilted dorsal view so left/right hemisphere placement is easier to interpret


v24 refinement:
- changed default 3D camera to dorsal view so left/right hemisphere separation is visible immediately
- added small hemisphere-aware label offsets so L/R labels do not overlap as much


v25 refinement:
- fixed the 3D helper from resetting the camera to an unhelpful default
- default 3D view now uses an oblique perspective so left/right hemisphere separation is visible
- increased left/right coordinate spread for bilateral regions
- increased label offsets for L/R nodes


v27 fix:
- repaired syntax in the bilateral 3D layout/parser block


v28 refinement:
- moved barrel cortex L/R nodes farther laterally and slightly dorsally/anteriorly for clearer bilateral separation

### v10 mEEG EDF pipeline patch notes (best-effort implementation)

What was added
- Standard EDF system button now launches the app.
- Add animal flow for EDF:
  1) choose EDF
  2) assign usable EDF channels
  3) relabel each selected channel to a brain region
  4) optionally average channels that share the same assigned region
  5) cache imported channels as per-animal int16 DAT files for downstream reuse
- Project profile is updated from the first EDF animal so pre-processing and qEEG follow the chosen labels.
- Core qEEG runner no longer assumes exactly 10 channels for correlation/coherence/lead-level PSD outputs.

---

## 8. Known important output files

This section is a brief orientation only. A separate full data dictionary can be maintained if desired.

### Preprocessing / QC

- `qc_badEpochHeatmap_[GROUPID_ANIMALID]`: QC heatmap of bad epochs by channel and time.
- `qc_goodQChannelPct_[GROUPID_ANIMALID]`: percentage of recording where each channel passed QC.
- `qc_lowQEpochPercent_[GROUPID_ANIMALID]`: percentage of epochs flagged as low quality per channel.
- `qc_summary`: per-animal/channel preprocessing summary table.

### qEEG

- `[GROUPID_ANIMALID]_bands`: per-animal normalized band power by band, channel/region, and time bin.
- `[GROUPID_ANIMALID]_bands_DN`: day/night summarized normalized band power.
- `BandAnalysis_RegionPerAnimal`: tidy per-animal normalized band-power table.
- `BandAnalysis_RegionSummary`: group-level normalized band-power summary table.
- `[GROUPID_ANIMALID]_ps_avg`: per-animal average PSD by region/lead and time bin.
- `[GROUPID_ANIMALID]_ps_std`: per-animal PSD variability by region/lead and time bin.
- `[GROUPID_ANIMALID]_corr`: per-animal band-specific and broadband correlation matrices.
- `[GROUPID_ANIMALID]_coh`: per-animal coherence data.
- `[GROUPID_ANIMALID]_coh_daynight`: day/night-averaged coherence matrices.
- `[GROUPID_ANIMALID]_cohSpec`: full-spectrum pair-level coherence data for Pair Explorer when enabled.

### Spike / SWD / Event Detection

- `[GROUPID_ANIMALID]_spikes`: per-animal spike detection outputs.
- `[GROUPID_ANIMALID]_swd`: per-animal SWD `.mat` output.
- `[GROUPID_ANIMALID]_swd_events`: per-event SWD spreadsheet.
- `[GROUPID_ANIMALID]_swd_summary`: per-animal SWD summary spreadsheet.
- `SWD_summary_all_animals`: combined SWD summary across animals.
- `[GROUPID_ANIMALID]_event_detection`: per-animal event-detection `.mat` output.
- `[GROUPID_ANIMALID]_event_detection_events`: per-event detection spreadsheet.
- `[GROUPID_ANIMALID]_event_detection_summary`: per-animal event-detection summary spreadsheet.

---


