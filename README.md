# multimodality-brain-image-segmentation
A practical guide to scaling neurological AI with multimodality brain image segmentation across CT, MRI, DSA and beyond. Maintained by Pariedolia Systems LLP.
Why this repository exists

A neuroradiologist rarely trusts a single image. CT shows acute bleeding quickly, MRI shows fine soft-tissue detail, and DSA shows cerebral vessels in real time. Diagnosis comes from combining them.

Much of neurological AI, however, is still built around one scan type at a time. That works in a notebook and often struggles in a real hospital.

Multimodality brain image segmentation means training systems to outline the same structures or findings (tumour, infarct, haemorrhage, vessel, aneurysm) across more than one imaging type, so results stay consistent whichever scan arrives first.

This repository collects what we have learned about doing that at scale.

Contents
The modalities at a glance
Why scaling is hard
A practical pipeline checklist
Example: multi-channel input with nnU-Net
Evaluation checklist
Tools and references
Roadmap
Contributing
License
The modalities at a glance
Modality	Strengths	Challenges
CT	Fast, widely available, strong for acute haemorrhage and bone	Lower soft-tissue contrast, subtle early ischaemia
MRI	Rich contrast across T1, T2, FLAIR, DWI, SWI	Large variation between scanners, protocols and sites
DSA	Gold standard for cerebral vessels, time-resolved	2D frame series, small datasets, hard to segment
CTA / MRA / Perfusion	Vessel and flow context	Registration and timing complexity
Why scaling is hard
Annotation cost. Voxel-wise labelling of 3D volumes is slow, and every extra modality multiplies the effort.
Misalignment. CT and MRI of the same patient differ in timing, orientation and resolution. Poor registration quietly corrupts labels.
Domain shift. Models that perform well on one site can drop sharply on another scanner or protocol.
Missing modalities. Real patients often lack one or more scans, so models must tolerate absent inputs.
Trust. Boundary errors in clinically critical regions matter far more than average overlap scores suggest.
A practical pipeline checklist

Data

 Convert and verify DICOM to NIfTI consistently
 De-identify before data leaves its source
 Standardise orientation and voxel spacing
 Normalise intensities per modality
 Apply skull stripping where appropriate
 Check and visually review registration between modalities

Modelling

 Start with a strong, self-configuring baseline before custom architectures
 Pretrain with self-supervised learning on unlabelled CT and MRI where possible
 Train with modality dropout so the model tolerates missing inputs
 Use active learning to focus expert review on uncertain cases

Deployment

 Validate externally on unseen sites and scanners
 Review failure cases with clinicians
 Monitor performance after deployment for drift
Example: multi-channel input with nnU-Net

In nnU-Net v2, each modality is supplied as its own channel by adding a numeric suffix to the filename. This is a layout example, not a trained model.

text
nnUNet_raw/
└── Dataset001_BrainMultimodal/
    ├── dataset.json
    ├── imagesTr/
    │   ├── case_001_0000.nii.gz   # channel 0, e.g. CT
    │   ├── case_001_0001.nii.gz   # channel 1, e.g. MRI (registered to CT)
    │   └── case_002_0000.nii.gz
    └── labelsTr/
        ├── case_001.nii.gz
        └── case_002.nii.gz

Typical command flow:

bash
nnUNetv2_plan_and_preprocess -d 001 --verify_dataset_integrity
nnUNetv2_train 001 3d_fullres 0
nnUNetv2_predict -i INPUT_FOLDER -o OUTPUT_FOLDER -d 001 -c 3d_fullres

Note: channels must be spatially aligned. Register all modalities to a common space first.

Evaluation checklist
Report overlap (for example Dice) and boundary-based metrics (for example Hausdorff distance).
Report results per modality and per site, not only pooled averages.
Test with missing modalities on purpose.
Include external validation data the model has never seen.
Review a sample of outputs visually with a clinician.
Tools and references

Frameworks and libraries

nnU-Net: self-configuring segmentation framework
MONAI: open-source framework for deep learning in healthcare imaging
SimpleITK, nibabel, ANTs: image I/O and registration
3D Slicer: visualisation and annotation

Public challenges and datasets worth knowing

BraTS: multimodal MRI brain tumour segmentation
ISLES: ischaemic stroke lesion segmentation

Please check each dataset's own licence and data use terms before use.

Roadmap
 Expand the guide on DSA vessel segmentation
 Add a section on handling missing modalities
 Add notes on federated and privacy-preserving training
 Add a glossary for non-clinical engineers
 Add a reading list of key papers
Contributing

Corrections, improvements and new sections are welcome.

Fork the repository
Create a branch for your change
Open a pull request with a short description

If you work in neuroimaging, medical AI or clinical neurology, we would love to hear how you handle multimodal data. Open an issue to start a discussion.

About Pariedolia Systems LLP

The name comes from pareidolia, the human habit of seeing meaningful patterns, like faces in clouds, in ambiguous images. In medical imaging the goal is the opposite: systems that find the patterns that are really there and do not invent the ones that are not.

Our focus is neurological AI that treats CT, MRI, DSA and other modalities as one connected picture, with careful data preparation, rigorous validation and clinical usefulness as the goal.

License

Documentation in this repository is released under the CC BY 4.0 license unless stated otherwise.
