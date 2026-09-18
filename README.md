# Heartbeat-evoked responses and self-reported meditative depth

EEG (64 channels), electrocardiography, respiration, electromyography and
electrodermal activity recorded from experienced Vipassana meditators during two
sessions of meditation with repeated self-reports of meditative depth. The
dataset derives from the experiment described in Reggente et al. (2024) and
accompanies an article analysing heartbeat-evoked responses across reported
depth (reference to be added on publication). The dataset contains the raw
recordings only; no preprocessed data are included.

## Participants

Participants were experienced Vipassana meditators who self-confirmed
proficiency in Vipassana as taught by S.N. Goenka, with at least five years of
meditation experience and sessions of at least ten minutes practiced at least
five times per week. They reported no history of hypotension, neurological or
psychiatric disorders, or use of central nervous system medications. All study
materials were approved by the Advarra Institutional Review Board (Columbia, MD;
Protocol No. Pro00063946) and informed consent was obtained from each
participant through Castor eConsent.

`participants.tsv` lists the participants in this dataset. The `in_analysis`
column marks those included in the analyses of the associated article, and the
other columns give the age, sex and meditation practice reported at screening
(`participants.json`). Participant labels are sequential; study record
identifiers and session dates are not included.

## Design

Each session (`ses-1`, `ses-2`, at least one week apart) consisted of six blocks
performed with eyes closed: resting state (5 min), pre-up meditation (5 min),
meditation with a sustained attention passive oddball sequence (SAPO; 35 min),
an active oddball task (5 min), meditation without the sequence (SILENT; 35 min)
and a cool-down meditation (5 min). The two 35-minute blocks were interchanged
between a participant's two sessions. During these blocks participants
indicated the deepest level of meditation since their last report (1 to 5)
followed by their confidence (1 to 5) with a clicker, either spontaneously
(emerge) or when prompted (probe). See `task-meditation_eeg.json` for the full
task description.

## Recording

- EEG: BrainAmp DC 64 (Brain Products; two stacked 32-channel units, hence the
  "dual-amp" system of Reggente et al.) with a 64-channel BrainCap TMS fitted with
  sintered Ag/AgCl multitrodes plus one ground and one reference, 500 Hz, online
  reference FCz, ground AFz, impedances below 10 kOhm. The cap was used unmodified:
  the holes it allows were never cut, and no transcranial magnetic stimulation was
  part of this study. The scalp was exfoliated with NuPrep (Weaver and Co.) and the
  electrodes filled with Abralyt 2000, a high-chloride abrasive electrolyte gel
  (Neurospec; EASYCAP).
- Physiology: Cognionics Aim II system, recording electromyography from the left
  and right sternocleidomastoid muscles, respiration by bio-impedance on the left
  and right pectoralis major (the same leads recorded the ECG), and electrodermal
  activity from the non-dominant palm.
- All streams were acquired and synchronized with NeuroPype Experiment Recorder
  (Intheon) over the Lab Streaming Layer and saved as XDF files.

## Conversion to BIDS

The conversion code is in `code/` (`step0_convert_to_bids.m` for the recordings,
`step0_convert_questionnaires.m` for the questionnaires). For each session,
the XDF streams were read with clock synchronization and jitter removal (EEGLAB
xdfimport plugin) and cropped to the time span common to the EEG, physiological
and marker streams. The physiological channels were resampled onto the EEG
sample times by linear interpolation, and each marker was placed at the nearest
EEG sample. Intervals of more than 0.1 s between consecutive time stamps were
treated as missing data (see Known issues). No filtering or re-referencing was
applied.

Channels: 64 EEG channels, `EMG1` to `EMG4` (the ExG channels of the Aim II),
`ECG`, `RESP`, `GSR` and `PPG`. The photoplethysmography channel is not used in
the associated article and is flat in 36 of the 78 recordings. The Aim II oxygen
saturation, heart rate, temperature and trigger channels held constant values in
nearly all recordings and no physiological signal in any, and the packet counter
is a device counter; these channels are not included. The device labels every
channel in microvolts; the respiration, electrodermal and photoplethysmography
channels are left in the raw units of the device (`n/a` in `channels.tsv`).

## Events

Each `events.tsv` keeps one row per marker, with the original marker text in
`value`, its category in `trial_type`, the block in `block`, and the depth and
confidence ratings, oddball response times, durations logged at the end of
sub-blocks and blocks, and clicker keys in their own columns. All columns and
levels are described in `task-meditation_events.json`, which also carries the
HED annotations (HED 8.4.0). Ratings are given as registered: values of 0 and
above 5 occur. In the earliest recordings (sub-005 and sub-006) the rating
markers do not name the block; the block given is that of the preceding report or
prompt window. A free-text note written by the experiment software in every file
is omitted.

## Questionnaires

Questionnaires were completed with Castor ePRO and are in `phenotype/`, in one
file per questionnaire and session (for example `maas_ses1.tsv` and
`maas_ses2.tsv`) with one row per participant, each described by its JSON
sidecar:

- Before the session: Brief Profile of Mood States (`poms_pre`), Mindful
  Attention Awareness Scale (`maas`), Cognitive and Affective Mindfulness
  Scale-Revised (`camsr`), Five Facet Mindfulness Questionnaire (`ffmq`) and
  Freiburg Mindfulness Inventory (`fmi`).
- After the session: Brief Profile of Mood States (`poms_post`), Meditation Depth
  Index (`medi`), Toronto Mindfulness Scale (`tms`) and task ratings
  (`session_ratings`).
- After the first and the second 35-minute meditation block: ratings in
  `block1_ratings` and `block2_ratings`.

Each file gives the scores followed by the item responses. Free-text answers are
not included, and the item wording of the Brief Profile of Mood States is not
reproduced. sub-007 has no questionnaires for session 2, and a questionnaire
missing for a session has no row.

## Known issues

- sub-021, ses-1: the EEG amplifier stopped mid-recording. The file keeps the
  31.8 minutes common to all streams.
- sub-005, ses-2: the EEG amplifier stopped at about 87 minutes, during the SAPO
  block. The file keeps the 86.6 minutes common to all streams.
- sub-006, ses-1: the marker stream ends at the break after the SILENT block, so
  the post-meditation block is not included. The file keeps the 103.7 minutes
  common to all streams.
- Gaps in the EEG acquisition. Where no EEG sample was recorded for more than
  0.1 s, the samples on either side are stored contiguously, a `boundary` event
  marks the gap, and markers registered during the gap are omitted. Times are
  approximate, from the start of the recording:
  - sub-004, ses-2: one 330.6 s gap at about 97 min (128 markers omitted,
    including two depth and two confidence ratings of the SAPO block).
  - sub-014, ses-2: one 330.6 s gap at about 77 min (38 markers omitted,
    including two depth and two confidence ratings of the SILENT block).
  - sub-009, ses-2: two gaps of 46.9 s and 421.1 s at about 58 and 59 min
    (no markers).
  - sub-002, ses-1: six gaps totalling 16.0 s at about 44 min (no markers);
    ses-2: six gaps totalling 8.3 s at about 29 min (8 markers omitted), and about
    500 interruptions of one or a few samples that are not marked.
- sub-028, ses-2: a 43.7 s gap in the physiological stream at about 52 min. The
  physiological channels are empty (NaN) during this gap.

## References

Reggente, N., Kothe, C., Brandmeyer, T., Hanada, G., Simonian, N., Mullen, S.,
et al. (2024). Decoding Depth of Meditation: EEG Insights from Expert Vipassana
Practitioners. Biological Psychiatry Global Open Science, 100402.
doi: 10.1016/j.bpsgos.2024.100402
