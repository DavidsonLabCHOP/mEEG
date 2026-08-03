Welcome to mEEG README file!! 

**mEEG** is a MATLAB GUI-based pipeline for EEG preprocessing, qEEG analysis, spike analysis, spike-wave-discharge (SWD) analysis, event detection, and connectivity/coherence visualization. This README is intended to serve as the central place to document version-specific changes, known fixes, output behavior, and future updates.


Current tracked version: **mEEG v1.118  
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

### mEEG v1.115 — SWD calibration workflow, feature export, and Threshold Assistant
NEW: Calibration / Discovery mode
- Optional checkbox in SWD Settings.
- Retains a broader candidate pool for manual review while preserving the user's displayed thresholds as the original automatic SWD decision.
- Discovery candidate defaults are intentionally only moderately permissive:
  envelope z >= min(user setting, 2.0)
  duration >= min(user setting, 0.50 s)
  cycles >= min(user setting, 2)
  frequency extends 1 Hz below and 2 Hz above the selected SWD band.
- Normal mode preserves prior behavior and saves detector-passing candidates only.
- Calibration mode saves automatic passes and automatic near-misses, with AutoIncluded/AutoDecision kept separately from UserIncluded/UserDecision.

NEW: Explicit SWD review labels
- Viewer buttons now read:
  Keep current candidate (real SWD)
  Exclude current candidate (not a real SWD)
- User decisions are saved as user-kept or user-excluded.
- The viewer shows automatic versus current/manual decision separately.
- Viewer summary now displays duration, peak-to-peak amplitude, cycles, dominant frequency, envelope z-score, rhythmicity, harmonic score, artifact score, and detector score.

NEW: Export SWD feature list
- Button at the bottom of SWD Viewer.
- Excel workbook contains:
  SWD_Features (one candidate per row; split into multiple sheets only if Excel row limits require it)
  Settings_Used
  Review_Summary
  README
- Includes AnimalID, TreatmentGroup, other manifest metadata, lead, timing/day-night, automatic and manual decisions, review timestamp, duration, amplitude, cycles, envelope z-score, rhythmicity, harmonic score, artifact score, and all other saved SWD features.
- Includes individual automatic pass/fail flags and FailedCriteria for duration, frequency, cycles, IPI regularity, rhythmicity, harmonic score, artifact score, and lead count.

NEW: SWD Threshold Assistant
- Compares current Keep versus Exclude labels for each adjustable SWD threshold feature:
  minimum duration
  minimum cycles
  minimum envelope z-score
  minimum rhythmicity
  minimum harmonic score
  maximum artifact score
- Shows current and suggested thresholds, a near-optimal range, sensitivity (kept SWDs retained), specificity (excluded candidates rejected), precision, F1, and balanced accuracy.
- Displays Keep/Exclude feature distributions with TreatmentGroup colors visible.
- Provides group-specific retention and excluded-pass diagnostics in the assistant report.
- Can apply one or all suggested values to the SWD Settings controls; it never starts an analysis automatically.
- Can export Recommendations, Group_Diagnostics, Threshold_Curves, and README sheets.

SMALL/PILOT STUDIES
- No minimum number of animals is required.
- Recommendations are labeled exploratory when reviewed event/animal counts are small.
- “Weight animals equally” is enabled by default so an animal with many SWDs does not dominate threshold selection.
- No train/test split is forced; users should rerun and validate settings before full-cohort analysis.

DAY/NIGHT
- The assistant detects available event time periods.
- It works with day-only, night-only, or combined recordings and only offers time filters that are present.

TREATMENT GROUPS
- TreatmentGroup is not hidden.
- Groups remain visible in plots and exports.
- Thresholds are intentionally optimized for reviewed real-SWD versus non-SWD classification, not for maximizing treatment-group separation, to avoid circular group-biased detection.

IMPORTANT INTERPRETATION
- Each suggested threshold is evaluated one feature at a time.
- The SWD detector combines several criteria, so applied suggestions must be rerun and the complete detector output validated.
- Discovery mode may produce more candidates, longer run times, and larger review files.


### mEEG v1.114 — Safe shared-FFT coherence engine
Purpose
-------
The previous coherence implementation called mscohere separately for every
unique electrode pair in every 30-minute qEEG period. With 10 electrodes,
that is 45 pair calls per period. Each pair call repeated the Welch windowing
and FFT calculation for both electrodes.

v1.114 calculates each lead's windowed FFT once on a shared, time-aligned
Welch segment grid and reuses those FFTs for every pair. The magnitude-squared
coherence estimator remains:

    |Pxy(f)|^2 / (Pxx(f) * Pyy(f))

Implemented changes
-------------------
1. Added core/meeg_coherence_shared_fft.m
   - Uses the existing coherence-window duration.
   - Uses the existing overlap fraction.
   - Uses the same Hamming window and power-of-two FFT length.
   - Processes Welch segments in memory-bounded batches of 64.
   - Computes all electrode-pair cross spectra from shared lead FFTs.
   - Retains double-precision calculations.

2. QC-aware time-aligned segments
   - A Welch segment is used for a pair only if the complete segment is valid
     for both electrodes.
   - Rejected 5-second epochs remain at their original recording times.
   - Clean samples on opposite sides of a rejected epoch are no longer joined
     together before coherence calculation.

3. Dynamic 1-10 electrode support
   - The shared engine uses the number of leads present in the project.
   - A single-electrode project continues to have no off-diagonal coherence.
   - EDF electrode labels and separate channels remain unchanged.

4. Frequency-limited accumulation
   - The FFT is still calculated at the existing FFT length.
   - Only frequencies required for gamma/fast-gamma summaries and optional
     Pair Explorer spectra are retained in intermediate arrays.

5. No second full signal copy
   - The existing lead-by-sample 30-minute matrix is passed directly to the
     engine using MATLAB copy-on-write behavior.
   - Only the current batch of windowed segments and their retained FFT bins
     are held as additional large arrays.

6. Compatibility fallback
   - If the shared-FFT engine errors for a particular period, mEEG falls back
     to the legacy pair-by-pair mscohere calculation for that period.
   - The number of fallback periods is printed and stored in *_coh.mat as
     cohSharedFFTFallbackWindows.
   - cohEngine is stored as 'shared_fft_v1.114'.

mEEG v1.114b — Optional coherence-only 500 Hz resampling
1. Added an optional qEEG GUI checkbox:
   "Use 500 Hz when original sampling rate is higher"
   Default: off.

2. When enabled, only the coherence signal is resampled. The original EEG
   and all other mEEG modules remain at the original sampling rate.

3. MATLAB resample() is used, including anti-alias FIR filtering.

4. Lower-rate recordings are never upsampled. Examples:
   2500 Hz -> 500 Hz; 1000 Hz -> 500 Hz; 500 Hz -> unchanged;
   256 Hz EDF -> unchanged.

5. Coherence fmax remains user-defined in the GUI with limits 50-200 Hz.
   The actual calculation fmax is capped only by the processing Nyquist
   frequency. Requested and actual values are logged and saved.

6. The following metadata are added to *_coh.mat (and relevant fields to
   *_cohSpec.mat):
   cohOriginalFs, cohProcessingFs, cohResampleRequested,
   cohResampleApplied, cohResampleTargetHz, cohResampleP, cohResampleQ,
   cohRequestedFmax, cohActualFmax.

7. The shared-FFT engine identifier is now shared_fft_v1.114b.

Scientific behavior
-------------------
- Welch window duration, overlap, band definitions, QC masks, 30-minute
  periods, day/night handling, and pairwise coherence definition are
  unchanged.
- Small numerical differences can occur after resampling because the
  anti-alias filter removes frequencies above the new Nyquist limit.
- Fast-gamma coherence should be validated when enabling the 500 Hz option.


### mEEG v1.113 — Event Detector Phase 1 safe-speed patch
1. 30-minute processing blocks
   - Each lead is read and processed in 30-minute central blocks.
   - Each block includes surrounding local-baseline context.
   - The context defaults to the GUI local-baseline duration, with a small
     additional guard for the candidate window and merge interval.
   - Candidate windows remain aligned to one global recording-time grid.
   - Events are assigned to one central block by event midpoint.
   - Rare split/duplicate events at block boundaries are consolidated.

2. Reusable 0.5-80 Hz bandpass
   - The filter is designed once per animal sampling rate.
   - The preferred design uses MATLAB bandpass-compatible minimum-order IIR
     settings (0.85 steepness and 60 dB stopband attenuation), then reuses
     the returned digitalFilter with zero-phase filtfilt.
   - A stable SOS Butterworth fallback and the original bandpass fallback
     are retained for compatibility.

3. Vectorized time-domain features
   - RMS is calculated by cumulative window sums.
   - line length is calculated by cumulative absolute first differences.
   - peak-to-peak amplitude is calculated with sliding extrema.
   - candidate-window SD is vectorized for the existing peak-prominence rule.
   - These calculations preserve the previous feature definitions.

4. Block-wise rhythmicity spectrum
   - One spectrogram call calculates the 3-30 Hz spectral values for all
     candidate windows in a processing block.
   - This replaces one periodogram call per candidate window.
   - The same rectangular candidate window, overlap, DFT-bin frequencies,
     and max-power / summed-power rhythmicity definition are retained.
   - A periodogram fallback is retained if the block spectrogram call fails.

5. Existing spike-rate behavior retained
   - findpeaks is still run per candidate window.
   - MinPeakProminence, MinPeakDistance, candidate duration, and spike-rate
     thresholds are unchanged.

6. Cell-array result collection
   - Block tables are collected in cells and concatenated once per lead.
   - Repeated table growth inside the block loop was removed.

7. Intan and EDF compatibility
   - Processing uses the project's per-channel cache files and actual animal
     sampling rate.
   - It supports 1-10 electrode EDF projects and standard Intan projects.
   - EDF user-assigned lead labels remain separate; no new L/R averaging was
     introduced by this patch.


### mEEG v1.112 — SWD 30-minute chunking and adaptive parallel fallback
Problem addressed
-----------------
Long, high-sampling-rate recordings previously loaded every lead of an animal
into one full double-precision array. Running several animals simultaneously
multiplied the filtered, Hilbert-envelope, and FFT memory requirements and
could produce out-of-memory errors or process-worker access violations.
1. SWD reads one lead at a time in overlapping chunks.
   - Core chunk duration: 30 minutes (1800 seconds)
   - Overlap on each side: up to 60 seconds
   - Full recordings are never loaded as one channels-by-samples matrix.

2. Events near chunk boundaries are preserved using the overlap. Each event
   is retained by the chunk whose non-overlap core contains the event center,
   preventing duplicate event rows.

3. Adaptive animal-level process parallelism:
   - First tries the maximum local process-worker capacity, limited by the
     number of selected animals.
   - If animals fail or the parallel loop/pool crashes, mEEG releases the
     current pool and retries unresolved animals at approximately 75% of the
     maximum worker count.
   - It then retries at approximately 50%, then approximately 25%.
   - Any unresolved animals are finally run serially with the same 30-minute
     chunk size.

4. Successful animals are retained between completed worker tiers. Only
   unresolved animals are retried after an ordinary MATLAB-level error. If a
   worker access violation aborts the complete parfor attempt, the unresolved
   attempt is safely repeated at the next lower worker tier.

5. Process pools are resized exactly between adaptive tiers so worker memory
   is returned to the operating system. Thread pools are replaced by process
   pools because the filtering and MAT-file operations used by these mEEG
   analyses are not consistently supported by thread workers.

6. Full filtered/envelope debug vectors are not saved by default. Event
   tables, summaries, SWD settings, lead labels, candidate counts, recording
   duration, and chunk metadata remain in the saved outputs.

7. SWD logs report:
   - 30-minute/60-second chunk settings
   - detected maximum process-worker count
   - each adaptive worker tier
   - lead and chunk progress
   - animals retried at lower worker counts
   - final serial fallback and any remaining errors

Files changed
-------------
- mEEGApp.m
- core/meeg_swd_runall.m
- core/meeg_swd_run_animal.m
- core/meeg_swd_find_candidates.m
- core/meeg_ensure_process_pool.m

Scientific behavior
-------------------
SWD bands, thresholds, duration rules, rhythmicity scoring, harmonic scoring,
artifact scoring, multilead scoring, and manifest-based day/night timing were
not changed. Envelope normalization is calculated within each 30-minute
processing chunk rather than across the entire recording at once.


### mEEG v1.109 GUI and connectivity color-scale patch
1. Connectivity plots now use correlation-appropriate symmetric color limits for all correlation displays:
   - Slider 0.5 -> color range -0.5 to +0.5
   - Slider 1.0 -> color range -1.0 to +1.0
   This applies to Group A, Group B, and Difference when the source is Correlation.
   Coherence Group A/Group B remain 0 to the selected maximum; coherence Difference remains symmetric.
2. Reduced the widths of the action buttons by approximately 10% in:
   - Normalized Band Power
   - Power Spectral Density
   - Connectivity Plots
3. Increased the Spike Analysis Settings Advanced detection settings panel height from 140 to 180 pixels.
4. Changed the checkbox label from "Use legacy spike filtering (for compatibility)" to "Use advanced detection settings".
   This patch changes the displayed label only; the existing underlying spike filtering option is otherwise unchanged.
5. Changed the Workspace navigation label from "Spike-Wave-Discharge" to "Spike-Wave Discharge".


### mEEG v1.108 - macOS PSD post-processing performance/reliability patch
1. qEEG now writes a compact internal *_ps_avg_cache.mat alongside each
   *_ps_avg.xlsx file. PSD post-processing, Power Spectra previews, fractional
   PSD tables, and AUC ratios prefer this cache instead of repeatedly opening
   many Excel sheets. The XLSX files remain unchanged and are still produced.

2. Legacy projects without the cache are still supported. The fallback reader
   now enumerates workbook sheets once and performs one read per matched sheet,
   avoiding repeated failed readmatrix calls that are especially slow on macOS.

3. PSD AUC smoothing-spline fitting is limited to 0.4-100 Hz, the only range
   used by the defined delta through fast-gamma AUC bands. Previously it fit the
   entire spectrum up to Nyquist, which could make post-processing appear hung
   on high-sampling-rate recordings. A toolbox-free pchip fallback is used if
   the smoothing-spline fit is unavailable.

4. Detailed PSD post-processing milestones are written to qeeg_log, including
   animal loading, cache/XLSX source, group AUC export, and figure export.

5. Auto-saved PSD figures are created invisibly to reduce macOS graphics/UI
   overhead. Plot contents and saved output formats are unchanged.


### mEEG v1.107 connectivity color-scale update
1. Added a "Color scale max (0-1)" slider to the Connectivity Plots subtab.
2. Group A / Group B connectivity plots use color limits [0, selected maximum].
3. Difference connectivity plots use symmetric color limits [-selected maximum, +selected maximum].
4. Connectivity heatmaps, network edges, and colorbars now use the same MATLAB parula blue-to-yellow colormap used by Correlation and Coherence, replacing the previous red/blue difference scheme.


### mEEG v1.106 patch notes
1. Standard Intan 10-electrode projects now show only the five bilateral
   regional averages in the Power Ratios region selector:
     - Hippocampus
     - Barrel
     - Motor
     - Visual
     - Auditory
   Separate (L)/(R) choices are intentionally hidden because qEEG ratio
   source tables average the paired Intan electrodes by region.
2. EDF projects continue to show the actual user-assigned labels, including
   side-specific labels such as Hippocampus (L), Hippocampus (R), Motor (L),
   Area A, Area B, and other project-setup choices.
3. EDF ratio matching now prioritizes exact labels. Selecting Hippocampus (L)
   will no longer also match Hippocampus (R).

4. Ratio rows are reduced to one value per animal for the selected region.
   This safely averages matched bilateral legacy Intan rows if encountered,
   while leaving an exact EDF side-specific value unchanged.


### mEEG v1.105 patch notes
1) Correlation plots with variable channel counts
   - Correlation group collection no longer preallocates based only on project.profile.channelLabels.
   - It now inspects the saved *_corr.xlsx files and chooses the shared actual matrix size for the selected groups.
   - This prevents errors such as assigning a 6x6 animal matrix into a 10x10 stack.
   - Correlation stats now carry labels matched to the actual matrix size, and the qEEG GUI/Connectivity tab use those labels.
2) Connectivity plots using correlation
   - Connectivity no longer overwrites correlation labels with stale profile labels.
   - This keeps correlation/connectivity consistent for EDF projects with 1-10 electrodes and for projects where saved matrix size differs from the old 10-channel template.
3) Coherence diagnostics/warnings
   - Coherence readback now restores the diagonal to 1 when a saved matrix has missing/NaN diagonals.
   - This preserves off-diagonal values while preventing otherwise valid sparse coherence files from being reported as completely empty.
   - Coherence labels are now prettified consistently.
4) PSD auto-saved region labels for the 10-electrode template
   - The 10-electrode template is now collapsed to the expected five regional averages for PSD post-processing:
       Auditory, Visual, Hippocampus, Barrel, Motor
   - L/R lead labels are preserved for correlation and coherence, but PSD regional autosaves now show the base region labels instead of only the right-side labels.
   - Variable-electrode EDF projects still plot the actual user-assigned labels present in the project.
5) PSD autosave subplot selection
   - The PSD autosave helper no longer uses the old fixed five-subplot order for every project with five or more labels.
   - It only applies the old five-region order when the data are actually the canonical five-region set.
   - Variable EDF projects with 6, 7, 8, 9, or 10 user-assigned labels should now plot all labels instead of silently selecting the first five.


### mEEG v1.104 patch notes

1. qEEG ratio log message
   - Removed the incorrect message: "Band ratios are currently a stub (not implemented yet)."
   - Power ratios are implemented in the Power Ratios subtab through meeg_ratio_build_table.m and meeg_ratio_plot_grouped.m.
   - qEEG now logs that ratios are generated on demand from the Power Ratios subtab because ratio numerator/denominator/time/region are user-selected.
2. Optional CORE qEEG modules
   - The qEEG Settings checkbox for CORE modules is enabled again.
   - When checked, qEEG runs normalized band power and power spectral density outputs used by downstream ratio calculations.
   - When unchecked, the user can run correlation and/or coherence without regenerating core qEEG outputs.
   - qEEG now warns if no qEEG analysis module is selected.
3. qEEG subtab layout
   - Narrowed qEEG-side selection/control panels by approximately 16% to give plots more horizontal space.
   - Applied to qEEG Settings, Normalized Band Power, Power Spectra, Power Ratios, Correlation, Coherence, and Connectivity qEEG subtabs.
4. Per-region normalized band-power figure layout
   - PerRegionNormBandPower_* figures now use up to a 5-column layout.
   - A 10-region/electrode project will display as 5 x 2 instead of one long horizontal row.
   - Smaller variable-electrode EDF projects still use a compact dynamic layout.
5. Day/night clarification
   - qEEG stores the current qEEG Settings dayStart/dayEnd values in project.profile during the run.
   - The recording start clock hour is still taken from the manifest StartTime/RecordingStartHour via meeg_manifest_get_start_hour.m.
   - Calculations then use clock time = manifest StartTime + time from recording start.


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


