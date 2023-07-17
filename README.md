# LogicLLaMA and MALLS

**LogicLLaMA:**
A language model that translates natural-language (NL) statements into first-order logic (FOL) rules. It is trained by fine-tuning the LLaMA-7B model on the [MALLS](https://huggingface.co/datasets/yuan-yang/MALLS-v0) dataset.

**MALLS**
(large language **M**odel gener**A**ted natural-**L**anguage-to-first-order-**L**ogic pair**S**):
a dataset consists of 34K pairs of real-world natural language (NL) statements and the corresponding first-order logic (FOL) rules annotations. All pairs are generated by prompting GPT-4 and processed to ensure the validity of the FOL rules.

**Harnessing the Power of Large Language Models for Natural Language to First-Order Logic Translation**

[Yuan Yang](https://gblackout.github.io/), [Siheng Xiong](https://dblp.org/pid/277/4221.html), [Ali Payani](https://scholar.google.com/citations?user=9rHwD8wAAAAJ&hl=en), [Ehsan Shareghi](https://eehsan.github.io/) and [Faramarz Fekri](https://fekri.ece.gatech.edu/)

[[Paper](https://arxiv.org/abs/2305.15541)] [[Data](https://huggingface.co/datasets/yuan-yang/MALLS-v0)] [[Model](https://huggingface.co/yuan-yang)]

## Release

- [x] [7/16/2023] We release the MALLS dataset (large language **M**odel gener**A**ted natural-**L**anguage-to-first-order-**L**ogic pair**S**) 
which consists of 34K pairs of real-world natural language (NL) statements and the corresponding first-order logic (FOL) rules annotations.
- [x] [7/16/2023] We released the LoRA delta weights for direct translation and naive correction LogicLLaMA
- [x] [7/16/2023] Support int8 loading.
- [ ] Colab demo.
- [ ] Pipeline for generating synthetically perturbed FOL rules.
- [ ] Release weights for RLHF CoT correction LogicLLaMA and the corresponding pipeline.


## Dataset and weights

Datasets:
- [MALLS-v0](https://huggingface.co/datasets/yuan-yang/MALLS-v0/blob/main/MALLS-v0.json): The file containing the 34K pairs of the MALLS dataset.
- [FOLIO-parsed](https://huggingface.co/datasets/yuan-yang/MALLS-v0/blob/main/folio_parsed.json): the file containing 2K pairs collected and processed from the [FOLIO](https://github.com/Yale-LILY/FOLIO) datset.

LoRA delta weights:
- [direct translation](https://huggingface.co/yuan-yang/LogicLLaMA-7b-direct-translate-delta-v0)
- [naive correction](https://huggingface.co/yuan-yang/LogicLLaMA-7b-naive-correction-delta-v0)

Download the dataset by running
```commandline
sh data_download.sh
```

## Install

1. Clone this repo
```commandline
git clone https://github.com/gblackout/LogicLLaMA.git
cd LogicLLaMA
```

2. Prepare environment
```
conda create -n logicllama python=3.7
conda activate logicllama
pip install -r requirements.txt
```

## Notebook Demo

Checkout out `demo.ipynb` for a quick demonstration of LogicLLaMA inference and the FOL rule parsing.


## Inference


### Direct translation

Copy the template from `scripts` to the project root
```commandline
cp scripts/eval_translation.sh ./
```
Modify `--base_model` to the folder/link of the base LLaMA-7B model ([instructions](https://huggingface.co/docs/transformers/main/model_doc/llama) for getting the base model).
Also modify `--data_path` to the path to the dataset that you want to infer or evaluate.
Set `--load_in_8bit=False` if your GPU does not support 8 bit quantization.

It can be the FOLIO or the MALLS dataset you downloaded with `data_download.sh`.
Or it can be your own dataset as long as it follows the following format:
```commandline
[
    {
        'NL': <your NL statement>,
        'FOL': <optional FOL rule>
    }, 
    ...
]
```
The `FOL` field is optional: if provided, we will evaluate the BLEU and
the logical equivalence (LE) score of the predicted FOL rule with respect to this field, otherwise only inference is performed.

Finally, run `sh eval_translation.sh`.


### Naive correction

In this mode, LogicLLaMA serves as a downstream model that corrects the predicted FOL rule by ChatGPT,
which generally leads to better performance than ChatGPT alone or the direct translation mode.

That said, you need to first collect the response from ChatGPT and then correct it:

1. Collect ChatGPT predictions by copying the template from `scripts` to the project root
```commandline
cp scripts/gpt_translation.sh ./
``` 
Modify the following field:
- `--api_key`: your OpenAi api key or the path to the key file
- `--dataset`: the path to the dataset
- `--save_path`: the path to save the dataset
- Set `--load_in_8bit=False` if your GPU does not support 8 bit quantization.

Finally, run `sh gpt_translation.sh`.

You can also use your own methods to get the response, as long as the final dataset has the following format:
```commandline
[
    {
        'NL': <your NL statement>,
        'Pred FOL': <GPT predicted FOL rule>,
        'FOL': <optional FOL rule>        
    }, 
    ...
]
```

2. Correct the output with LogicLLaMA. Copy the template from `scripts` to the project root
```commandline
cp scripts/eval_correction.sh ./
```
Modify the `--base_model` to the LLaMA-7B base model and `--data_path` to the output dataset from step 1.

Finally, run `sh eval_correction.sh`.

## Training

You can also train LogicLLaMA from scratch on MALLS or your own dataset.

### Direct translation

Copy the template from `scripts` to the project root
```commandline
cp scripts/sft_translation.sh ./
``` 
Modify the following field:
- `--base_model`: path/link to the base model 
- `--data_path`: the path to the dataset with both `NL` and `FOL` fields
- `--output_dir`: the path for saving the `peft` model
- `--use_wandb`: whether to use wandb for logging
- Set `--load_in_8bit=False` if your GPU does not support 8 bit quantization.

Finally, run `sh sft_translation.sh`

### Naive correction

Copy the template from `scripts` to the project root
```commandline
cp scripts/sft_correction.sh ./
``` 
Modify the same fields as above.
Note that the dataset here needs to have `NL`, `FOL` (ground-truth FOL) and `Pred FOL` (GPT predicted FOL) available.
You can obtain `Pred FOL` by running `gpt_translation.sh` following  the instruction in the inference section.

Finally, run `sh sft_correction.sh`

## License

The data, code and weights are released under Apache 2.0 license and are intended for research use only. 
Additionally, the usage of the LogicLLaMA model should follow the license agreement of LLaMA and Alpaca. 
The dataset is CC BY NC 4.0 and should follow the [policy of OpenAI](https://openai.com/policies/terms-of-use),
as it is collected from GPT-4.

## Acknowledgement

This project is developed based on the following repos:
- [alpaca-lora](https://github.com/tloen/alpaca-lora): the sft and inference modules are mainly built upon this repo.
- [trl](https://github.com/lvwerra/trl): we refer to the RLHF scripts in this repo when implementing the RLHF CoT correction.


## Citation

```commandline
@article{yang2023harnessing,
      title={Harnessing the Power of Large Language Models for Natural Language to First-Order Logic Translation}, 
      author={Yuan Yang and Siheng Xiong and Ali Payani and Ehsan Shareghi and Faramarz Fekri},
      journal={arXiv preprint arXiv:2305.15541},
      year={2023}
}
```