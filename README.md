# PeakNetFP

Official repository of _PeakNetFP: Peak-based Neural Audio Fingerprinting Robust to Time Stretching_ by Guillem Cortès-Sebastià, Benjamin Martin, Emilio Molina, Xavier Serra, and Romain Hennequin. To be presented at ISMIR 2025. Preprint is available in arXiv.

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.17811353.svg)](https://doi.org/10.5281/zenodo.17811353)
 

<p align="left">
<img src="https://github.com/user-attachments/assets/b95847aa-3188-491a-a5fe-2e8486d98ace" width="390">
</p>

## Getting started
PeakNetFP runs in a docker container. The same container can be used to run PeakNetFP and NeuralFP.

1. Clone the repository.

2. Build docker image.
Modify the `docker-compose.yml` if we want to change the image or containers names, for instance.
   ```bash
   docker-compose build
   ```
3. Build and run the container.
   ```bash
   docker-compose up -d
   ```
4. Open an interactive shell
   ```bash
   docker exec -it peaknetfp /bin/bash
   ```

## Getting the data
To reproduce the experiments in this publication two data sets are needed:

### Train and validation

We use the dataset from Neural-Audio-FP (https://github.com/mimbres/neural-audio-fp?tab=readme-ov-file#dataset) for training and validation. The scalability of the approach is tested by adding the tracks of `test-dummy-db-100k-full` to the reference DB. Hence the importance to download the full neural-audio-fp dataset, available in IEEEDataPort (https://ieee-dataport.org/open-access/neural-audio-fingerprint-dataset). We train with `train-10k-30s` subset while `val-query-db-500-30s` is used for validation.

### Test
The test dataset is created in this publication from the test set of neural-audio-fp test set and it is available in [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15646861.svg)](https://doi.org/10.5281/zenodo.15646861)
. The test set contains the tracks of `test-query-db-500-30s` stretched by the following factors: 0.5, 0.6, 0.7, 0.8, 0.9, 0.95, 0.975, 1 (no stretch), 1.05, 1.1, 1.2, 1.4, 1.6, 1.8, and 2. using the `tempo` functionality of SOX (https://sourceforge.net/projects/sox/). The downloaded `query_stretch` folder must be placed inside the `test-query-db-500-30s` folder of the neural-audio-fp dataset. So the tree structure of the dataset directory should look like this

```
neural-audio-fp-dataset
├── aug
├── extras
├── LICENSE
├── music
│   ├── LICENSE-fma
│   ├── others
│   ├── test-dummy-db-100k-full
│   ├── test-query-db-500-30s
│   │   ├── db
│   │   ├── query
│   │   ├── query_stretch   --> PeakNetFP stretched test set
│   │   ├── test_ids_icassp2021.npy
│   ├── train-10k-30s
│   └── val-query-db-500-30s
└── README.md
```

## Model weights

Model checkpoints are available in [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15782389.svg)](https://doi.org/10.5281/zenodo.15782389). Checkpoints have to be stored in the root folder of this repo under `logs/checkpoint`.

## Train

  - Check batch-size that fits on your device first.
  - Check the config file and adjust the parameters accordingly.

```bash
python run.py train -c config/<config_file.yml>
```

## Generate 
example with stretching 0.9

```bash
python run.py generate <exp_name> <ckpt_index> \
     -c config/<config_file.yaml> \
     -o logs/emb/<exp_name>/tempo-0_900/<ckpt_index> \
     --query_dir /datasets/neural-audio-fp-dataset/music/test-query-db-500-30s/query_stretch/tempo-0_900
```

It will generate the following files:
  ```sh
  .
  └──logs
     └── emb
         └── CHECKPOINT_NAME
             └── CHECKPOINT_INDEX
                 ├── db.mm
                 ├── db_shape.npy
                 ├── dummy_db.mm
                 ├── dummy_db_shape.npy
                 ├── query.mm
                 ├── query_shape.npy
                 ├── query_segments.csv
                 ├── db_segments.csv
                 └── dummy_db_segments.csv 
  ```
  By `default` config, `generate` will generate embeddings (or fingerprints)
   from 'dummy_db', `test_query` and `test_db`. The generated embeddings will 
   be located in `logs/emb/<exp_name>/<ckpt_index>/**.mm` and 
   `**.npy`.

  - `dummy_db` is generated from the test set specified in the config file.
  - In the `DATASEL` section of config, you can select options for a pair of
   `db`  and `query` generation. The default is `unseen_icassp`, which uses a
    pre-defined test set.
  - It is possilbe to generate only the `db` and `query` pairs by 
  `--skip_dummy` option. This is a frequently used option to avoid overwriting 
    the most time-consuming `dummy_db` fingerprints in every experiment.   
  - It is also possilbe to generate embeddings (or fingreprints) from your
   custom source using `-s` or `--source` argument during generation.



## Evaluate
example with stretching 0.9

- PeakNetFP performs song-level search, but we keep the segment level search code from NeuralFP for backwards compatibility.


```bash
python run.py evaluate <exp_name> <ckpt_index> \
     -c config/<config_file.yaml> \
     --emb_dir logs/emb/neuralfp/tempo-0_900/<ckpt_index> \
     --stretch_factor 0.9
```

## Other 
### Tensorboard

Run a dedicated docker container to check tensorboard:

```bash
docker run --network=host -ti --rm --name tensorboard -v <vol_path>:<vol_path> tensorflow/tensorflow
```

then run:

```bash
tensorboard --logdir <vol_path>/neuralfp/logs/fit --port <port> --bind_all
```


## Notes
Main developments with respect to the [original NeuralFP repository](https://github.com/mimbres/neural-audio-fp):

* Fix some parts of the code such as dataset generation loader.
* Correct the data normalization (now each melspec is normalized, before it was normalize at batch level).
* Upgrade packages to newer versions. This was motivated by the fact that the original code with tensorflow==2.4.1 cannot be run in newer gpu cards, so we upgraded to the last version 2.15.
* ADD peak spectrogram, gaussian spectrogram transformations to input data.
* ADD stretching augmentation 
* Include PointNet++ code to build PeakNetFP


## Acknowledgements

This research is part of resCUE – Smart system for automatic usage reporting of musical works in audiovisual productions (SAV-20221147) funded by CDTI and the European Union - Next Generation EU, and supported by the Spanish Ministerio de Ciencia, Innovación y Universidades and the Ministerio para la Transformación Digital y de la Función Pública. Furthermore, it has received support from the Industrial Doctorates plan of the Secretaria d’Universitats i Recerca, Departament d’Empresa i Coneixement de la Generalitat de Catalunya, grant agreement No. DI46-2020.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4ff73525-1d9c-4f38-bae3-da5cf8067bf4" height="50" hspace="10" vspace="10" />
  <img src="https://github.com/user-attachments/assets/9181df37-11df-4fd7-bb69-ec7bc482c93c" height="40" hspace="10" vspace="10"/>
  <img src="https://github.com/user-attachments/assets/13f9a73d-288a-449e-b17f-6a26479c7a55" height="50" hspace="10" vspace="10"/>
</p>
<p align="center">
  <img src="https://user-images.githubusercontent.com/25322884/186637854-50e06004-9dc6-40ee-8ec9-701899136a6e.png" height="40" hspace="10" vspace="10"/>
  <img src="https://user-images.githubusercontent.com/25322884/186637746-e18c4517-250c-4474-b11e-58df1e1f0787.jpeg" height="50" hspace="10" vspace="10"/>
  <img src="https://user-images.githubusercontent.com/25322884/186637861-24a64957-f82b-4faa-be34-5b1221bbd05c.png" height="40" hspace="10" vspace="10"/>
</p>



## Citation

> G. Cortès-Sebastià, B. Martin, E. Molina, X. Serra, R. Hennequin, “PeakNetFP: Peak-based Neural Audio Fingerprinting Robust to Extreme Time Stretching”, in Proc. of the 26th Int. Society for Music Information Retrieval Conf., ISMIR 2025, Daejeon, South Korea, September 21-25, 2025.

```
@article{cortes2025peaknetfp,
  title        = {PeakNetFP: Peak-Based Neural Audio Fingerprinting Robust to Extreme
                  Time Stretching},
  author       = {Guillem Cort{\`{e}}s{-}Sebasti{\`{a}} and
                  Benjamin Martin and
                  Emilio Molina and
                  Xavier Serra and
                  Romain Hennequin},
  booktitle    = {Proceedings of the 26th International Society for Music Information
                  Retrieval Conference, {ISMIR} 2025, Daejeon, South Korea, September
                  21-25, 2025},
  pages        = {206--214},
  year         = {2025},
  url          = {https://doi.org/10.5281/zenodo.17811353},
  doi          = {10.5281/ZENODO.17811353},
  timestamp    = {Mon, 01 Dec 2025 23:06:58 +0100},
}
```
