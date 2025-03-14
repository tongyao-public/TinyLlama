# SkyLadder: Better and Faster Pretraining via Context Window Scheduling



## Introduction
Recent advancements in LLM pretraining have featured ever-expanding context
windows to process longer sequences. However, our pilot study reveals that models
pretrained with shorter context windows consistently outperform their long-context
counterparts under a fixed token budget. This finding motivates us to explore
an optimal **context window scheduling** strategy to better balance long-context
capability with pretraining efficiency. To this end, we propose SkyLadder, a simple
yet effective approach that implements a short-to-long context window transition.
SkyLadder preserves strong standard benchmark performance, while matching or
exceeding baseline results on long-context tasks. Through extensive experiments,
we pre-train 1B-parameter models (up to 32K context) and 3B-parameter models
(8K context) on 100B tokens, demonstrating that SkyLadder yields consistent gains
of up to 3.7% on common benchmarks, while achieving up to 22% faster training
speeds compared to baselines

![1b-8k-proweb.png](assets%2F1b-8k-proweb.png)
## Quick Start 

This project is based on the [TinyLLaMA]() project. It has been adapted to support pretraining with context window scheduling, intra-document masking, etc.

###  Installation
Please follow the instructions in the original [TinyLLaMA]() project.
```python
conda create -n ladder-pretrain python=3.8
conda activate ladder-pretrain
pip install -r requirements.txt
```

### Data preparation
The data preparation process is the same as the original tinyllama project.
First make sure that your data is in jsonl format in one directory. Then run the following:
```bash
export TEXT_DIR=YOUR_TEXT_DIR
export BIN_DIR=OUTPUT_DIR
bash scripts/pajama_processing.sh cc 8k
```
where `cc` is the dataset name and `8k` is the number of tokens in the vocabulary. 
The `TEXT_DIR` is the directory where the text data is stored and the `BIN_DIR` is the directory where the processed data will be stored.
In other words, your training text jsonl files should be in `$TEXT_DIR/cc/train` and the evaluation files should be in `$TEXT_DIR/cc/valid`, and the processed data will be stored in `$BIN_DIR/cc`

### Pretraining
Next, you can start pretraining by running the following:
```bash
export WANDB_API_KEY=<YOUR_WANDB_API_KEY>
export BINS_ROOT=$BIN_DIR # replace with your own
bash scripts/pretraining.sh tiny_LLaMA_1b_8k cc_8k cc_8k
```
First, you need to set the `WANDB_API_KEY` to your own key. The `BINS_DIR` is the directory where the processed data is stored.
The usage of `pretraining.sh` is 'bash scripts/pretraining.sh model_config_name train_dataset_name eval_dataset_name'. For instance, 
`tiny_LLaMA_1b_8k` is the model config name, `cc_8k` is the training dataset name, and `cc_8k` is the evaluation dataset name. 
Please make sure that there is a folder named $BIN_DIR/$val_dataset_name/valid in the `BINS_DIR` directory.


You can simply replace the model config name to get different models:
```
bash scripts/pretraining.sh tiny_LLaMA_1b_8k cc_8k cc_8k
bash scripts/pretraining.sh tiny_LLaMA_1b_8k_intramask cc_8k cc_8k # intradocument masking
bash scripts/pretraining.sh tiny_LLaMA_1b_8k_dm8 cc_8k cc_8k # short-to-long dynamic increase 
bash scripts/pretraining.sh tiny_LLaMA_1b_8k_intradm8 cc_8k cc_8k # intradocument masking + short-to-long dynamic increase
```
Here, dm8 means that $\alpha$ is 1/8. Therefore dm1 is the fastest and dm8 is the slowest.


### Advanced Usage

#### Multi-node Pretraining
Alternatively, if you are running on multiple nodes, you can use the following command:
```bash
export WANDB_API_KEY=xxxxxxx
export BINS_ROOT=XXXX # replace with your own
export NUM_NODES=4 # number of nodes, adjust accordingly
bash scripts/pretraining_multi.sh tiny_LLaMA_1b_8k cc_8k cc_8k
```
This will run the pretraining on multiple nodes. 

#### Intra-Document Masking
We implemented intra-document masking (which can be combined with SkyLadder). 
The model name of `tiny_LLaMA_1b_8k_intramask` means that the model will be trained with intra-document masking. 
To combine with SkyLadder, use suffices like `intradm8` ($\alpha=1/8$), `intradm4`, etc.


#### Other types of schedules 
You can also find other types of schedules we experimented with in our paper.
For instance, 'tiny_LLaMA_1b_8k_sin8' means that the schedule is a sinusoidal schedule with $\alpha$ being 1/8. 

#### Convert to HF
You can convert the model to Huggingface format by running the following:
```bash
export MODEL_DIR=YOUR_MODEL_DIR
export MODEL_NAME=YOUR_MODEL_NAME
bash scripts/convert_to_hf.sh $MODEL_DIR $MODEL_NAME
```
where `MODEL_DIR` is the directory where the model is stored and `MODEL_NAME` is the name of the model.

## Citation
If you find our paper or this code useful, we would appreciate it if you could cite our work: 
```

```

## Acknowledgement
We would like to thank the authors of the original [TinyLLaMA](https://github.com/jzhang38/TinyLlama) project for their work. 
The ladder icon in the title is made by [Freepik](https://www.flaticon.com/authors/freepik) from [www.flaticon.com](https://www.flaticon.com/).. 
