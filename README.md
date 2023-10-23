# GPTFUZZER : Red Teaming Large Language Models with Auto-Generated Jailbreak Prompts

<img src="./sources/icon.png" width=500>

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This is the official repository for "GPTFUZZER : [Red Teaming Large Language Models with Auto-Generated Jailbreak Prompts](https://arxiv.org/pdf/2309.10253.pdf)" by [Jiahao Yu](https://sherdencooper.github.io/), [Xingwei Lin](https://scholar.google.com/citations?user=Zv_rC0AAAAAJ&hl=en), [Zheng Yu](http://www.dataisland.org/), [Xinyu Xing](http://xinyuxing.org/).

## Table of Contents

- [Updates](#updates)
- [Installation](#installation)
- [Datasets](#datasets)
- [Models](#models)
- [Running] (#running)
<!-- - [Experiments](#experiments) -->
- [Release](#release)
- [FQA](#fqa)

## Updates
- (2023/10/22) Our work will be presented in the [Geekcon 2023](https://geekcon.darknavy.com/2023/china/en/index.html) conference! We will give a talk about our work and give a live attack demo with our tool. We will also include the attack results for Chinese LLMs which have not be included in our paper. We are looking forward to meeting you there!
- (2023/10/21) We have updated our codebase to make it more readable and easier to use. We will continue to update the codebase and add more features and other implementations. We are looking forward to build a general black-box fuzzing framework for large language models. Stay tuned!
- (2023/9/19) Our paper is on arXiv! Check it out [here](https://arxiv.org/pdf/2309.10253.pdf)!

## Installation

Please refer to [install.ipynb](./install.ipynb)

## Datasets
The datasets for the harmful question and human-written templates are available in `datasets/questions/question_list.csv` and `datasets/prompts/GPTFuzzer.csv`. The questions are sampled from two public datasets: [llm-jailbreak-study](https://sites.google.com/view/llm-jailbreak-study) and [hh-rlhf](https://huggingface.co/datasets/Anthropic/hh-rlhf), and the templates are collected from [llm-jailbreak-study](https://sites.google.com/view/llm-jailbreak-study).

For the responses we got by querying Vicuna-7B, ChatGPT and Llama-2-7B-chat, we store them in `datasets/responses` and the labeled responses are in `datasets/responses_labeled`. You could also use `generate_responses.py` to generate responses for different models or different questions (see the scripts under `scripts` folder for examples).

We are still working on the evaluation on other question dataset and jailbreak dataset. We will update the codebase and the datasets after we have some results.
## Models

Our judgment model is a finetuned RoBERTa-large model and the training code is in `finetune_roberta.py`, and the training/evaluating data is stored in `datasets/responses_labeled`. The model we used is hosted on [Hugging Face](https://huggingface.co/hubert233/GPTFuzz). When running fuzzing experiments, the model will be automatically downloaded and cached for the first time. If you would like to download the model manually, you can run the following code:

```python
from transformers import RobertaForSequenceClassification, RobertaTokenizer
model_path = 'hubert233/GPTFuzz'
model = RobertaForSequenceClassification.from_pretrained(model_path)
tokenizer = RobertaTokenizer.from_pretrained(model_path)
```
During our experiments, we found that our trained model can also be transferred to other questions. However, we also found that it does not work well on some questions and other languages. We will add more predictor model soon.

## Running
We provide a python [example](./gptfuzz.py) to show the minimal code to run the fuzzing experiments. This example uses ChatGPT as mutate model to attack Llama-2-7B-chat with official system prompt(we did the monkey patch for Fastchat template), and you should be able to get the identical results in [example folder](./example/) (we set the random seed for reproducibility and temperature=0).


You can also refer to the [notebook](./gptfuzz.ipynb) for more details and explanations.

<!-- ## Experiments 

### Single-Model Fuzzing
To run the single-model single-question fuzzing experiments, you can run `fuzz_single_question_single_model.py`. In default, it will use ChatGPT as the mutate model and try to attack Llama-2-7B-chat. You can read the args in the script for more details. We have a script to run fuzzing on all questions that human-written templates failed for Llama-2-7B-chat, which is `scripts/run_single_question_single_model.sh`. You could edit that script for your own experiments.

To run the single-model multi-question fuzzing experiments, you can run `fuzz_multi_question_single_model.py`. In default, it will use ChatGPT as the mutate model and try to attack Llama-2-7B-chat. It will run 10000 queries against the target model. 

### Multi-Model Fuzzing
To run the multi-model fuzzing experiments, you can run `fuzz_multi_question_multi_model.py`. In default, it will use run fuzzing on Vicuna-7B, ChatGPT and Llama-2-7B-chat. It will run 30000 queries against the target models.

During fuzzing, the successful templates will be saved under `datasets/prompts_generated` and you can evaluate them with `evaluate_multi_question_multi_model.py` and `evaluate_multi_question_multi_model.py`

For more details about the experiments, please refer to our paper. -->
## Release

Due to ethical concern, we decided not to release the adversarial templates we found during our experiments openly. However, we are happy to share them with researchers who are interested in this topic. Please contact us via [email](mailto:jiahao.yu@northwestern.edu) if you would like to get access to the templates we found during the experiments. Also, you can use the code in this repository to generate your own adversarial templates.

## FQA
1. I found some labels in your labeled responses are wrong.
    - We are sorry about that. As our paper claimed, determining whether it is a jailbroken response is not a trivial task and some responses are hard to label. Also, due to the stress of labeling a large amount of potential toxic responses, we might have made some mistakes. If you find any wrong labels, please let us know and we will fix them as soon as possible.
2. The fuzz is slow, especially when I am using multiple questions for the local model.
    - We found that use batched inference can significantly speed up the fuzzing process. However, the results might be slightly different from the original results because of the padding tokens (see [here](https://github.com/tloen/alpaca-lora/issues/20)). We suggest using [vllm](https://github.com/vllm-project/vllm) inference for hyper performance
3. How could I implement my own mutator/seed selector?
    - You can implement your own mutator/seed selector by inheriting the class. You can refer to `mutator.py` and `selection.py` for examples.
    Also, as we claimed, we would like to work on a general black-box fuzzing framework for large language models. If you have some ideas or suggestions or you find other papers that are related to this topic, please let us know or leave the comment in the issue. We are happy to implement them and make this framework more powerful.