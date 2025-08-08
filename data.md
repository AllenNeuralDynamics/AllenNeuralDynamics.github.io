---
layout: page
title: Data Access
---

The Allen Institute for Neural Dynamics (AIND) is committed to FAIR, Open, and Reproducible science. We therefore share all of the data we collect publicly with rich metadata as near to the time of collection as possible. We share data at all stages of the data lifecycle, including preliminary data collected during methods development, processed data that we are actively improving, or highly curated data used in a publication.

## aind-open-data

In addition to sharing curated datasets with modality-specific NIH data archives like DANDI and BIL, we are also excited to share all of our data in one public S3 bucket generously hosted by the Registry of Open Data on AWS. 

All data is stored here: `s3://aind-open-data`

### Bucket Organization
AIND’s mission to share data early with full reproducibility has significant implications on how data and metadata should be organized and represented. Organizational principles include:

* **Immutability**: once data is collected it should not be touched to ensure reproducibility.
* **Metadata Accessibility**: metadata must be trivial to find and read for humans and machines.
* **Cloud Compatibility**: data should be stored in cloud-friendly formats.

Based on these principles, `s3://aind-open-data` is organized as a flat list of data assets, where a data asset is simply a logical collection of files. Derived data assets are separate from their source assets, meaning they are not stored in the same path or directory.

Inspired by [BIDS](https://bids-specification.readthedocs.io/en/stable/) and the [HCA metadata schema](https://data.humancellatlas.org/metadata/structure), metadata describing a data asset is stored as sidecar JSON files that live adjacent to the data they describe. These JSON files conform to the schemas defined in [aind-data-schema](https://github.com/allenNeuralDynamics/aind-data-schema/).

As soon as possible, data in this bucket are stored in cloud-friendly formats, including NWB-Zarr for physiology and OME-Zarr for imaging. We aspire to produce data in these formats at the time of acquisition.

### Naming Conventions
Raw data assets are named:

`<modality>_<subject-id>_<acquisition-date>_<acquisition-time>`

Derived data assets are named:

`<source-data-asset-name>_<label>_<acquisition-date>_<acquisition-time>`

A raw extracellular ephys asset would look like this:

```
    ecephys_123456_2022-12-12_05-06-07/
        ecephys/
            <ephys data>
        behavior/
            <video data>
        data_description.json
        subject.json
        procedures.json
        processing.json
        rig.json
        acquisition.json
```

A spike-sorting result asset would look like this:

```
    ecephys_123456_2022-12-12_05-06-07_sorted-ks25_2022-12-12_06-07-08/
        sorted/
            <sorted ephys data>
        data_description.json
        subject.json
        procedures.json
        processing.json
        rig.json
        acquisition.json
```

## Benchmark Data

### aind-benchmark-data/ephys-compression

Extracellular electrophysiology data is growing at a remarkable pace. This data, collected neuropixels probes by the Allen Institute for Neural Dynamics (AIND) and the International Brain Lab (IBL) can be used to benchmark throughput rates and storage ratios of various data compression algorithms.

All data is available within the `aind-benchmark-data` bucket at `s3://aind-benchmark-data/ephys-compression`.

The bucket is organized in subfolders containing datasets from different sources.

#### Experimental data

The following folders include experimental data sources:

- `aind-np2`: Neuropixels 2.0 data collected by AIND (8 sessions)
- `aind-np1`: Neuropixels 1.0 data collected by AIND (4 sessions)
- `ibl-np1`: Neuropixels 1.0 data collected by IBL (4 sessions)

Each session (e.g. `aind-np2/612962_2022-04-13_19-18-04_ProbeB`) contains the raw traces in binary format as saved by [SpikeInterface](https://github.com/SpikeInterface/spikeinterface).

To load one session, you can use the `spikeinterface` library (see [installation instructions](https://spikeinterface.readthedocs.io/en/latest/installation.html)):

```
import spikeinterface as si

recording = si.load_extractor("{local-path-to-aind-benchmark-data/ephys-compression}/aind-np2/612962_2022-04-13_19-18-04_ProbeB")
```

`recording` is a [`spikeinterface.BaseRecording`](https://spikeinterface.readthedocs.io/en/latest/api.html#spikeinterface.core.BaseRecording) object.

#### Simulated data

The `mearec` subfolder contains two simulated datasets using the [MEArec](https://github.com/SpikeInterface/MEArec) simulator.

You can load a simulated session using `spikeinterface` (after [installing MEArec](https://mearec.readthedocs.io/en/latest/installation.html)):

```
import spikeinterface.extractors as se

recording_gt, sorting_gt = se.read_mearec("{local-path-to-aind-benchmark-data/ephys-compression}/mearec/mearec_NP1.h5")
```

`recording_gt` is also [`spikeinterface.BaseRecording`](https://spikeinterface.readthedocs.io/en/latest/api.html#spikeinterface.core.BaseRecording) object. `sorting_gt` is a [`spikeinterface.BaseSorting`](https://spikeinterface.readthedocs.io/en/latest/api.html#spikeinterface.core.BaseSorting) containing the ground-truth spiking data.

### aind-benchmark-data/ephys-hybrid-evaluation

Evaluation of spike sorting methods is a challenging task, as it requires both ground-truth data and a variety of sorting algorithms to compare against. The `aind-benchmark-data/ephys-hybrid-evaluation` bucket contains a set of hybrid datasets specifically designed for benchmarking spike sorting methods.

All data is available within the `aind-benchmark-data` bucket at `s3://aind-benchmark-data/ephys-hybrid-evaluation`.

The datasets are used in the [Cloud-compatible pipelines for processing and evaluating large-scale electrophysiology data](). There are two subfolders, corresponding to the two applications used in the paper:

- `sorters`: used for comparison between `Kilosort2.5` and `Kilosort4`
- `lossy`: used for comparison between lossless and lossy compression (using [WavPack](www.wavpack.com))

For each application, there are two subfolders, one for Neuropixels 1.0 (`np1`) data and one for Neuropixels 2.0 - 4 shank (`np2`) data. Each subfolder contains hybrid datasets from multiple sessions, with the following structure:

```
    <application>/<np1|np2>/<session-id>/<recording-id>/
        recording.zarr/
        sorting.zarr/
```

For Neuropixels 2.0, the different shanks (or groups) are split into separate recording ids.
The last part of the path is the recording id, `_{i}` is the hybrid case: for each dataset and probe,
5 cases with randomly sampled hybrid units are available.

Recordings and sortings can be easily loaded using the `spikeinterface` library (version 0.102.3 or later).

The following code loads session id: `ecephys_644864_2023-01-31_15-27-43`, recording id: `experiment1_Record Node 110#Neuropix-PXI-107.ProbeA-AP_recording1_0/`:

```
import spikeinterface as si

session_id = "ecephys_644864_2023-01-31_15-27-43"
recording_id = "experiment1_Record Node 110#Neuropix-PXI-107.ProbeA-AP_recording1_0"

sorting = si.load(
    f"s3://aind-benchmark-data/ephys-hybrid-evaluation/sorters/np1/{session_id}/{recording_id}/sorting.zarr"
)
recording = si.load(
    f"s3://aind-benchmark-data/ephys-hybrid-evaluation/sorters/np1/{session_id}/{recording_id}/recording.zarr"
)
```
