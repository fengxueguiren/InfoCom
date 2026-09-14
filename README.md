# InfoCom (AAAI 2026)

InfoCom: Kilobyte-Scale Communication-Efficient Collaborative Perception with Information Bottleneck

[Paper](https://arxiv.org/abs/2512.10305) | [Project Page](https://weiquanmin.github.io/infocom/)

![Original1](images/Framework.png)

## Highlights
- InfoCom requires less than 10KB of communication per collaboration (vs. ~1MB+ for existing communication-efficient methods) 🚀
- Provides a principled information-theoretic analysis based on the Information Bottleneck framework 📚
- Plug-and-play: can be easily integrated into BEV intermediate collaborative perception pipelines 🔌  
- Designed for kilobyte-scale, communication-efficient collaborative perception in autonomous driving scenarios 🚗

## Abstract
Precise environmental perception is critical for the reliability of autonomous driving systems. While collaborative perception mitigates the limitations of single-agent perception through information sharing, it encounters a fundamental communication-performance trade-off. Existing communication-efficient approaches typically assume MB-level data transmission per collaboration, which may fail due to practical network constraints. To address these issues, we propose InfoCom, an information-aware framework establishing the pioneering theoretical foundation for communication-efficient collaborative perception via extended Information Bottleneck principles. Departing from mainstream feature manipulation, InfoCom introduces a novel information purification paradigm that theoretically optimizes the extraction of minimal sufficient task-critical information under Information Bottleneck constraints. Its core innovations include: i) An Information-Aware Encoding condensing features into minimal messages while preserving perception-relevant information; ii) A Sparse Mask Generation identifying spatial cues with negligible communication cost; and iii) A Multi-Scale Decoding that progressively recovers perceptual information through mask-guided mechanisms rather than simple feature reconstruction. Comprehensive experiments across multiple datasets demonstrate that InfoCom achieves near-lossless perception while reducing communication overhead from megabyte to kilobyte-scale, representing 440-fold and 90-fold reductions per agent compared to Where2comm and ERMVP, respectively.


## Installation

You can refer to the CoAlign Installation Guide [Chinese Ver.](https://udtkdfu8mk.feishu.cn/docx/LlMpdu3pNoCS94xxhjMcOWIynie) or [English Ver.](https://udtkdfu8mk.feishu.cn/docx/SZNVd0S7UoD6mVxUM6Wc8If6ncc) to learn how to install this repo. 


## Data Preparation

The data preparation is also the same as that of CoAlign and [OpenCOOD](https://opencood.readthedocs.io/en/latest/md_files/data_intro.html). For the DAIR-V2X dataset, please use the [supplemented annotations](https://siheng-chen.github.io/dataset/dair-v2x-c-complemented/).

## Training Procedure

To ensure stable training and fast convergence for communication-efficient collaborative perception models, we use pre-trained base collaborative perception weights (including AttFuse, CoAlign, and MKDCooper) instead of training from scratch with the communication-efficient components attached.

### Quick Training

Download the provided pre-trained base weights `all_models_base` and move them to the `opencood/logs` directory. Note that these weights do not include the communication-efficient modules and follow the default OpenCOOD or CoAlign training strategies and parameters.

Run the following command to train InfoCom when using CoAlign as the base model on DAIR-V2X:
```bash
python opencood/tools/train_infocom.py --model_dir opencood/logs/all_models_base/coib/coalign/dairv2x_coalign_coib
```
> You may need to update dataset paths such as `data_dir` to match your local environment.

After training, run inference to evaluate InfoCom (CoAlign base on DAIR-V2X):
```bash
python opencood/tools/inference.py --model_dir opencood/logs/all_models_base/coib/coalign/dairv2x_coalign_coib
```


### Complete Training

If you do not use our pre-trained weights, the complete training procedure is straightforward. In general, for communication-efficient frameworks such as InfoCom, Where2comm, and ERMVP, the training workflow can be divided into two steps: training the base model and training the communication-efficient model.

**Train the base collaborative perception model without communication-efficient components**

Assume you want to train CoAlign, your current working directory is `opencood/`, and the absolute path to the config file is `/opencood/hypes_yaml/opv2v/pointpillar_coalign.yaml`. You can train the base collaborative perception model with the following command:
```bash
python /home/wqm/data/infocom_dev/opencood/tools/train.py --hypes_yaml /opencood/hypes_yaml/opv2v/pointpillar_coalign.yaml
```
> This step is identical to training a collaborative perception model from scratch with OpenCOOD, including the parameters in the config file.

**Train the collaborative perception model including communication-efficient components**

After step one completes, model weights, configs, and logs can be found under `opencood/logs`, for example the `opv2v_coalign_2025_04_01_13_47_50` directory. Keep only the config file `config.yaml` and the best model weights `net_epoch_bestval_atxx.pth`, and remove the other files.

For `config.yaml`, add the InfoCom-required parameters and modify `core_method`. Specifically, add the following at the top of the file:
```yaml
ib_params:
  beta: 0.001
  begin_epoch: 15
```
Then change `core_method` to the correct class to invoke our communication-efficient method InfoCom; for example, replace:
`core_method: point_pillar_baseline` with `core_method: point_pillar_baseline_ib`.

Next, rename the best weights `net_epoch_bestval_atxx.pth` to `net_epoch_bestval_at1.pth`.

Finally, train InfoCom by running:
```bash
python opencood/tools/train_infocom.py --model_dir opv2v_coalign_2025_04_01_13_47_50
```

## Checkpoints

The main checkpoints can be downloaded [here](https://drive.google.com/drive/folders/1CoZ5rN4hXlBg78G1NNVZ6BrbUpJFXh3j?usp=sharing), and then save them in the `opencood/logs` directory. Note that our checkpoints rely on spconv=1.2.1.

We provide two types of checkpoints: pre-trained base models and the full communication-efficient InfoCom models.


## Other
If you want to implement your own communication-efficient method based on this code, simply replace the InfoCom-related code under `opencood/extensions`; it is plug-and-play.

## Citation
```
@inproceedings{wei2026infocom,
  title={Infocom: kilobyte-scale communication-efficient collaborative perception with information bottleneck},
  author={Wei, Quanmin and Dai, Penglin and Li, Wei and Liu, Bingyi and Wu, Xiao},
  booktitle={Proceedings of the AAAI Conference on Artificial Intelligence},
  volume={40},
  number={35},
  pages={29731--29739},
  year={2026}
}
```

## Acknowledgements

Thank for the excellent collaborative perception codebases [OpenCOOD](https://github.com/DerrickXuNu/OpenCOOD) and [CoAlign](https://github.com/yifanlu0227/CoAlign).
