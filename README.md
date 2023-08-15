# WebArena: A Realistic Web Environment for Building Autonomous Agents
<p align="center">
<b>WebArena is a standalone, self-hostable web environment for building autonomous agents</b>
</p>

<p align="center">
<a href="https://www.python.org/downloads/release/python-3109/"><img src="https://img.shields.io/badge/python-3.10-blue.svg" alt="Python 3.10"></a>
<a href="https://pre-commit.com/"><img src="https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white" alt="pre-commit"></a>
<a href="https://github.com/psf/black"><img src="https://img.shields.io/badge/code%20style-black-000000.svg" alt="Code style: black"></a>
<a href="https://mypy-lang.org/"><img src="https://www.mypy-lang.org/static/mypy_badge.svg" alt="Checked with mypy"></a>
<a href="https://beartype.readthedocs.io"><img src="https://raw.githubusercontent.com/beartype/beartype-assets/main/badge/bear-ified.svg" alt="bear-ified"></a>
</p>

<p align="center">
<a href="https://webarena.dev/">Website</a> •
<a href="https://arxiv.org/2307.13854">Paper</a>
</p>

![Overview](media/overview.png)

## News
* [8/4/2023] Added the instructions and the docker resources to host your own WebArena Environment. Check out [this page](environment_docker/README.md) for details.
* [7/29/2023] Added [a well commented script](minimal_example.py) to walk through the environment setup.
## Install
```bash
# Python 3.10+
conda create -n webarena python=3.10; conda activate webarena
pip install -r requirements.txt
playwright install
pip install -e .

# optional, dev only
pip install -e ".[dev]"
mypy --install-types --non-interactive browser_env agents evaluation_harness
pip install pre-commit
pre-commit install
```
## Quick Walkthrough
Check out [this script](minimal_example.py) for a quick walkthrough on how to set up the environment and interact with it.

## To Reproduce Our Results
1. Configurate the urls for each website, in the following example, we use the demo websites we host as an example. You can replace the URLs with your own websites if you [host your own WebArena environment](./environment_docker/).
```bash
export SHOPPING="http://ec2-3-131-244-37.us-east-2.compute.amazonaws.com:7770"
export SHOPPING_ADMIN="http://ec2-3-131-244-37.us-east-2.compute.amazonaws.com:7780/admin"
export REDDIT="http://ec2-3-131-244-37.us-east-2.compute.amazonaws.com:9999"
export GITLAB="http://ec2-3-131-244-37.us-east-2.compute.amazonaws.com:8023"
export MAP="http://ec2-3-131-244-37.us-east-2.compute.amazonaws.com:3000"
export WIKIPEDIA="http://ec2-3-131-244-37.us-east-2.compute.amazonaws.com:8888/wikipedia_en_all_maxi_2022-05/A/User:The_other_Kiwix_guy/Landing"
export HOMEPAGE="PASS" # this is a placeholder
```

2. Generate config file for each test example
```bash
python scripts/generate_test_data.py
```
You will see `*.json` files generated in [config_files](./config_files) folder. Each file contains the configuration for one test example.

3. Obtain the auto-login cookies for all websites
```
bash prepare.sh
```
4. export `OPENAI_API_KEY=your_key`, a valid OpenAI API key starts with `sk-`

5. Launch the evaluation
```bash
python run.py \
  --instruction_path agent/prompts/jsons/p_cot_id_actree_2s.json \ # this is the reasoning agent prompt we used in the paper
  --test_start_idx 0 \
  --test_end_idx 1 \
  --model gpt-3.5-turbo \
  --result_dir <your_result_dir>
```
This script will run the first example with GPT-3.5 reasoning agent. The trajectory will be saved in `<your_result_dir>/0.html`

## To Develop Your Prompt-based Agent
1. Define the prompts. We provide two baseline agents whose correrponding prompts are listed [here](./agent/prompts/raw). Each prompt is a dictionary with the following keys:
```python
prompt = {
  "intro": <The overall guideline which includes the task description, available action, hint and others>,
  "examples": [
    (
      example_1_observation,
      example_1_response
    ),
    (
      example_2_observation,
      example_2_response
    ),
    ...
  ],
  "template": <How to organize different information such as observation, previous action, instruction, url>,
  "meta_data": {
    "observation": <Which observation space the agent uses>,
    "action_type": <Which action space the agent uses>,
    "keywords": <The keywords used in the template, the program will later enumerate all keywords in the template to see if all of them are correctly replaced with the content>,
    "prompt_constructor": <Which prompt construtor is in used, the prompt constructor will construct the input feed to an LLM and extract the action from the generation, more details below>,
    "action_splitter": <Inside which splitter can we extract the action, used by the prompt constructor>
    }
  }
```

2. Implement the prompt constructor. An example prompt constructor using Chain-of-thought/ReAct style reasoning is [here](./agent/prompts/prompt_constructor.py#L184). The prompt constructor is a class with the following methods:
* `construct`: construct the input feed to an LLM
* `_extract_action`: given the generation from an LLM, how to extract the phrase that corresponds to the action

## Citation
If you use our environment or data, please cite our paper:
```
@article{zhou2023webarena,
  title={WebArena: A Realistic Web Environment for Building Autonomous Agents},
  author={Zhou, Shuyan and Xu, Frank F and Zhu, Hao and Zhou, Xuhui and Lo, Robert and Sridhar, Abishek and Cheng, Xianyi and Bisk, Yonatan and Fried, Daniel and Alon, Uri and others},
  journal={arXiv preprint arXiv:2307.13854},
  year={2023}
}
```
