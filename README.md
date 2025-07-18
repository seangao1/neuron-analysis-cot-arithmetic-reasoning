# neuron-analysis-cot-arithmetic-reasoning

This repository contains the code for ACL 2024 paper: [An Investigation of Neuron Activation as a Unified Lens to Explain Chain-of-Thought Eliciting Arithmetic Reasoning of LLMs](https://arxiv.org/abs/2406.12288).

**How to Cite:** If you find our survey useful for your research, please cite our paper:
```
@inproceedings{rai-yao-2024-investigation,
    title = "An Investigation of Neuron Activation as a Unified Lens to Explain Chain-of-Thought Eliciting Arithmetic Reasoning of {LLM}s",
    author = "Rai, Daking  and
      Yao, Ziyu",
    editor = "Ku, Lun-Wei  and
      Martins, Andre  and
      Srikumar, Vivek",
    booktitle = "Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)",
    month = aug,
    year = "2024",
    address = "Bangkok, Thailand",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2024.acl-long.387/",
    doi = "10.18653/v1/2024.acl-long.387",
    pages = "7174--7193",
    abstract = "Large language models (LLMs) have shown strong arithmetic reasoning capabilities when prompted with Chain-of-Thought (CoT) prompts. However, we have only a limited understanding of how they are processed by LLMs. To demystify it, prior work has primarily focused on ablating different components in the CoT prompt and empirically observing their resulting LLM performance change. Yet, the reason why these components are important to LLM reasoning is not explored. To fill this gap, in this work, we investigate {\textquotedblleft}neuron activation{\textquotedblright} as a lens to provide a unified explanation to observations made by prior work. Specifically, we look into neurons within the feed-forward layers of LLMs that may have activated their arithmetic reasoning capabilities, using Llama2 as an example. To facilitate this investigation, we also propose an approach based on GPT-4 to automatically identify neurons that imply arithmetic reasoning. Our analyses revealed that the activation of reasoning neurons in the feed-forward layers of an LLM can explain the importance of various components in a CoT prompt, and future research can extend it for a more complete understanding."
}
```

## Overview
Large language models (LLMs) have shown strong arithmetic reasoning capabilities when prompted with Chain-of-Thought (CoT) prompts. However, we have only a limited understanding of how they are processed by LLMs. To demystify it, prior work has primarily focused on ablating different components in the CoT prompt and empirically observing their resulting LLM performance change \cite{madaan2022text, wang-etal-2023-towards, ye-etal-2023-complementary}. Yet, the reason why these components are important to LLM reasoning is not explored. To fill this gap, in this work, we investigate ``neuron activation'' as a lens to provide a unified explanation to observations made by prior work. Specifically, we look into neurons within the feed-forward layers of LLMs that may have activated their arithmetic reasoning capabilities, using Llama2 \cite{touvron2023llama} as an example. To facilitate this investigation, we also propose an approach based on GPT-4 to automatically identify neurons that imply arithmetic reasoning. Our analyses revealed that the activation of reasoning neurons in the feed-forward layers of an LLM can explain the importance of various components in a CoT prompt, and future research can extend it for a more complete understanding. Please refer to our [paper](https://arxiv.org/pdf/2406.12288) for more details.

## Environment Setup
This project is tested in Python 3.8.5.

To get started, set up the environment:
```
python -m venv env 
source env/bin/activate
pip install -r requirements.txt
pip install torch==1.12.1+cu116 torchvision==0.13.1+cu116 torchaudio==0.12.1 --extra-index-url https://download.pytorch.org/whl/cu116
```

Now, clone the repository. 
```
git clone https://github.com/Dakingrai/ood-generalization-semantic-boundary-techniques.git
cd ood-generalization-semantic-boundary-techniques](https://github.com/Dakingrai/neuron-analysis-cot-arithmetic-reasoning)
```

## Running Llama2 on GSM8K test set
```
torchrun --nproc_per_node 1 gsm8k_inference.py --ckpt_dir <path_to_model> --tokenizer_path <path_to_model>/tokenizer.model --prompt <prompt_type_name> --few_shot True --results_dir <path_to_dir>
```
Note*: The `--prompt` argument should specify the filename of the prompt located in the data/prompts directory. For instance, to run inference using a prompt that contains only equations without any text, the argument should be set to `equation_only`.


## Neuron Discovery
This is the implementation of the Automatic Neuron Discovery approach proposed in the paper. 
![algorithms](https://github.com/user-attachments/assets/f6560d02-f0fc-465e-913c-1544e87e1302)

### Running Algorithm One.
```
torchrun --nproc_per_node 1 main.py --ckpt_dir <path_to_model> --tokenizer_path <path_to_model>/tokenizer.model --experiment algorithm_one --prompt <prompt_type_name> --data_dir <path_to_prompt_type_inference_results> --results_dir results/algorithm_one
```

Note*: The `--data_dir` argument should point to the `--results_dir` argument while running `gsm8k_inference.py` (e.g., `results/gsm8k_inference/text_only/final.json`), as we are sampling only from examples where the model made correct predictions.

### Running Algorithm Two.
```
torchrun --nproc_per_node 1 main.py --ckpt_dir ../../../downloads/huggingface/models/llama2-7b/ --tokenizer_path ../../../downloads/huggingface/models/llama2-7b/tokenizer.model --experiment algorithm_two --prompt data/prompts/equation_only.txt --data_dir results/gsm8k_inference/equation_only/final.json
```
Note*: The `--prompt` argument should specify the path to the desired prompt type (e.g., `data/prompts/equation_only.txt`). Similarly, the `--data_dir` argument should align with the `--results_dir` argument when running `gsm8k_inference.py` (e.g., `results/gsm8k_inference/text_only/final.json`).


### Run GPT-4 for automatic feature discovery.
Set the OpenAI key using the following command:
```
export OPENAI_API_KEY = <YOUR_KEY>
```
Note*: Please note that our code supports only OpenAI and Azure API calls. If you intend to use Azure API, please additionally specify the api_version, api_base, and api_version in the code.

Now, run prompt_gpt.py to automatically discover the concept neurons.

```
python main.py --experiment prompt_gpt --data_dir results/algorithm_two/<insert_prompt_type> --results_dir <insert_results_dir>
```
