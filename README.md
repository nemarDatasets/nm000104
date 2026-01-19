# emg2qwerty: Touch Typing from Surface Electromyography

## Overview

**Dataset**: emg2qwerty - Touch typing from wrist-based surface electromyography
**Task**: Touch typing on QWERTY keyboard
**Participants**: 108 subjects
**Sessions**: 1,135 total (average 10 per subject, range 1-18)
**Duration**: 346.4 hours total (9.5-47.5 min per session)
**Publication**: Sivakumar et al., 2024 - "emg2qwerty: A Large Dataset with Baselines for Touch Typing using Surface Electromyography"

### Purpose

This dataset captures wrist-based sEMG signals during touch typing on a physical keyboard. The goal is to enable keyboard-free text input by decoding typing intent directly from neuromuscular activity, with applications in AR/VR, mobile computing, and brain-computer interfaces.

This is the largest public sEMG dataset to date, specifically designed to study:
- Cross-user generalization
- Cross-session adaptation (domain shift from electrode placement)
- Sequence-to-sequence learning (analogous to automatic speech recognition)
- High-bandwidth neuromotor interfaces

## Dataset Details

### Participants

**Sample size**: 108 participants
**Demographics**: Not available (age, sex, handedness marked as n/a)
**Screening**: Touch typists with >90% correct finger-to-key mapping
**Typing speed**: 130-439 keys/min (mean: 265 keys/min, ~4.4 keys/sec)

### Hardware

**Device**: sEMG Research Device (sEMG-RD)
**Configuration**: Two wristbands (left and right wrists)
**Channels**: 32 total (16 per wrist)
**Sampling rate**: 2000 Hz
**Bit depth**: 12 bits
**Dynamic range**: ±6.6 mV
**Bandwidth**: 20-850 Hz
**Connectivity**: Bluetooth
**Electrode type**: Dry gold-plated differential pairs

### Recording Setup

**Keyboard**: Apple Magic Keyboard (US English)
**Text prompts**:
- Random words from dictionary
- Sentences from English Wikipedia
- Filtered for offensive terms
- Lowercase with basic punctuation only

**Ground truth**: Keylogger recording key-down and key-up timestamps (±0.5 ms precision)
**Backspace usage**: Allowed (natural typing behavior)

### Session Protocol

1. Participant dons two sEMG-RDs (one per wrist)
2. Types prompted text on physical keyboard
3. Keylogger records all keystrokes with timestamps
4. sEMG signals streamed via Bluetooth
5. Between sessions: Bands doffed and re-donned (realistic electrode placement variability)

**Session duration**: 9.5-47.5 minutes (depends on typing speed)
**Inter-session protocol**: Complete band removal and replacement to simulate real-world usage

## Data Contents

### Files per Session

```
sub-XXXXXXXX/ses-YYYYYYYYYY/emg/
├── sub-XXXXXXXX_ses-YYYYYYYYYY_task-typing_emg.edf
├── sub-XXXXXXXX_ses-YYYYYYYYYY_task-typing_emg.json
├── sub-XXXXXXXX_ses-YYYYYYYYYY_task-typing_channels.tsv
├── sub-XXXXXXXX_ses-YYYYYYYYYY_task-typing_events.tsv
└── sub-XXXXXXXX_ses-YYYYYYYYYY_electrodes.tsv
```

### Channel Configuration

**Total channels**: 32
- EMG0-EMG15: Left wrist
- EMG16-EMG31: Right wrist

**Channel naming**: Unique across entire dataset (EMG0-EMG31)
**Electrode naming**: E0-E15 (reused for left and right wrists)
**Reference**: Bipolar (differential sensing)

**channels.tsv columns**:
- `name`: Channel identifier (EMG0-EMG31)
- `type`: EMG
- `units`: V
- `signal_electrode`: Physical electrode name (E0-E15)
- `reference`: bipolar
- `group`: left or right (wrist)
- `target_muscle`: forearm muscles

**electrodes.tsv columns**:
- `name`: Electrode identifier (E0-E15)
- `x`, `y`, `z`: 3D coordinates (percent units, no decimals)
- `coordinate_system`: leftForearm or rightForearm
- `group`: left or right

### Events

**events.tsv contains**:
- **Keystroke events**: Individual key-press and key-release
  - `type`: keystroke_X (where X is the key character)
  - `latency`: Sample index of keystroke
  - `duration`: Samples from press to release
  - `key`: Character typed
- **Prompt events**: Text prompts shown to participant
  - `type`: prompt
  - `prompt_text`: Displayed text

**Total keystrokes**: 5,262,671 across all sessions

### Coordinate Systems

**Two separate coordinate systems** (space entities):

**Left Forearm** (`space-leftForearm_coordsystem.json`):
```
EMGCoordinateSystem: Other
EMGCoordinateUnits: percent
X: USP → RSP (0-100%)
Y: Right-hand rule perpendicular (limits: Olecranon Process → Cubital Fossa)
Z: Midpoint RSP-USP → Lateral Humeral Epicondyle
```

**Right Forearm** (`space-rightForearm_coordsystem.json`):
```
EMGCoordinateSystem: Other
EMGCoordinateUnits: percent
X: RSP → USP (0-100%, reversed from left)
Y: Right-hand rule perpendicular (limits: Olecranon Process → Cubital Fossa)
Z: Midpoint RSP-USP → Lateral Humeral Epicondyle
```

**Anatomical landmarks**:
- RSP: Radial Styloid Process
- USP: Ulnar Styloid Process
- LHE: Lateral Humeral Epicondyle

**Note**: Same physical device worn on both wrists with reversed differential polarity

## Signal Processing

### Preprocessing Applied

1. **High-pass filtering**: 40 Hz cutoff (removes DC drift, motion artifacts)
2. **Clock drift correction**: Synchronization between devices and laptop
3. **Temporal alignment**: Left/right wristband sample alignment (±0.5 ms)
4. **Irregular sampling handling**: Resampling applied when deviation >1%

### Signal Characteristics

**Typical features**:
- Muscle activation precedes keystroke by ~tens of milliseconds
- Different muscles activate for different fingers
- "Co-articulation" effects: sEMG affected by adjacent keystrokes
- Bigram/trigram context important for fast typists

**Receptive field**: Models typically need ~1 second context

## Baseline Performance

### Published Results (Sivakumar et al., 2024)

**Generic Model** (100 training users):
- Validation CER: 52.10 ± 5.54% (with 6-gram LM)
- Test CER: 51.78 ± 4.61% (with 6-gram LM)
- **Interpretation**: Unusable without personalization

**Personalized Model** (finetuned from generic):
- Validation CER: 8.31 ± 3.19% (with 6-gram LM)
- Test CER: 6.95 ± 3.61% (with 6-gram LM)
- **Best user**: 3.16% CER
- **Usability threshold**: ~10% CER

**Model architecture**: Time Depth Separable ConvNets (TDS)
**Loss function**: Connectionist Temporal Classification (CTC)
**Language model**: 6-gram modified Kneser-Ney (trained on WikiText-103)

### Key Findings

1. **Generalization emerges at scale**: 100+ users needed for meaningful representations
2. **Personalization essential**: Generic model alone has >50% CER
3. **Domain shift is severe**: Cross-user variation much larger than cross-session
4. **No obvious user clusters**: Every user requires individual adaptation

## Data Splits

### Benchmark Setup (from paper)

**Training set**: 100 users (all sessions except 2 validation per user)
**Validation set**: 2 sessions from each of 100 training users
**Test set**: 8 held-out users
  - Each test user: Multiple sessions split into train/val/test
  - Used for personalization experiments

**Note**: This split ensures test users don't influence generic model hyperparameters

## Use Cases

### Machine Learning

- **Sequence-to-sequence learning**: Similar to ASR but with different generative process
- **Domain adaptation**: Cross-user, cross-session generalization
- **Transfer learning**: Generic models with user-specific fine-tuning
- **Few-shot learning**: Data-efficient personalization
- **Language modeling**: Backspace-aware beam search decoding

### Neuroscience

- **Motor control**: Understand muscle coordination during fine motor tasks
- **Motor learning**: Track typing skill changes across sessions
- **Neuromuscular variability**: Study individual differences in muscle recruitment

### Applications

- **Keyboard-free typing**: Text entry without physical keyboard
- **AR/VR interfaces**: Text input for head-mounted displays
- **Silent communication**: Private text entry in public spaces
- **Accessibility**: Alternative input for users with limited mobility

## Known Issues and Limitations

### By Design

- **Touch typing required**: Not representative of hunt-and-peck typists
- **English only**: Language-specific
- **Physical keyboard**: Not actual keyboard-free typing
- **Typing style variation**: Individual strategies differ (especially non-fluent typists)
- **No demographic data**: Age, sex, handedness not collected

### Technical

- **Domain shift**: Large variations across users and sessions
- **Signal amplitude**: Varies with typing force (not normalized)
- **Backspace handling**: More complex than speech (can modify history)
- **Hardware unavailable**: sEMG-RD not commercially available

### Data Quality

- **Irregular sampling**: Some sessions required resampling (up to 9290% deviation detected)
- **Electrode placement**: Intentionally varies across sessions (creates realistic challenge)
- **Session length**: Varies by typing speed (9.5-47.5 min)

## Access and Contact

**Original data**: https://github.com/facebookresearch/emg2qwerty
**BIDS conversion**: Custom MATLAB tools using EEGLAB BIDS plugin
**Data curator**: Yahya Shirazi, SCCN, INC, UCSD
**Contact**: See original publication for corresponding author

## License

Non-Commercial, Share Alike CC-BY-NC-SA 4.0

## Citation
```
Sivakumar, V., Seely, J., Du, A., Bittner, S.R., Berenzweig, A.,
Bolarinwa, A., Gramfort, A., & Mandel, M.I. (2024).
emg2qwerty: A Large Dataset with Baselines for Touch Typing using Surface Electromyography.
arXiv:2410.20081. https://github.com/facebookresearch/emg2qwerty
```

## Data Curator
**Yahya Shirazi**
SCCN (Swartz Center for Computational Neuroscience)
INC (Institute for Neural Computation)
University of California San Diego

## Version History
**v1.0** (2025-10-01): Initial BIDS conversion

---
**BIDS Version**: 1.11 | **EMG-BIDS**: BEP-042 | **Updated**: Oct 1, 2025
