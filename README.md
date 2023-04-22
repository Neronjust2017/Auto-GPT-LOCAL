# Auto-GPT: An Autonomous GPT-4 Experiment
## This is a fork of Auto-GPT (https://github.com/Significant-Gravitas/Auto-GPT) with support for local LLaMA models. At present, this is more of an experimental idea.

## Setup Guide

### Install [gpt-llama.cpp](https://github.com/keldenl/gpt-llama.cpp) 


### Setup local API server

#### [`gpt-llama.cpp`](https://github.com/keldenl/gpt-llama.cpp) is an API wrapper around llama.cpp. It runs a local API server that simulates OpenAI's API GPT endpoints but uses local llama-based models to process requests.

1.  Setup [`llama.cpp`](https://github.com/ggerganov/llama.cpp) by the following instructions based on this [README](https://github.com/ggerganov/llama.cpp#usage).

```bash
git clone https://github.com/ggerganov/llama.cpp

cd llama.cpp
mkdir build
cd build
cmake ..
cmake --build . --config Release

# install Python dependencies
python3 -m pip install -r requirements.txt
```

2. Install and run `gpt-llama.cpp` locally
```
git clone https://github.com/keldenl/gpt-llama.cpp.git
cd gpt-llama.cpp

# install the required dependencies
npm install

# start the server
npm start
```
For more details, please refer to [`gpt-llama.cpp README`](https://github.com/keldenl/gpt-llama.cpp/blob/master/README.md
)

### Download Models
* LLaMA-7B-q4
* Vicuna-7B-q4

I have tried above models so far. You can download original LLaMA following the instructions [here](https://huggingface.co/docs/transformers/main/model_doc/llama). Or you can download in other ways : https://github.com/facebookresearch/llama/issues/149

For Vicuna weights, you can add its delta to the original LLaMA weights to obtain the Vicuna weights, instructions [here](https://github.com/lm-sys/FastChat).

1. Convert your downloaded LLaMa-7B weights to Hugging Face Transformers foramt using the following script [(source)](https://github.com/huggingface/transformers/blob/main/src/transformers/models/llama/convert_llama_weights_to_hf.py):
```bash
python src/transformers/models/llama/convert_llama_weights_to_hf.py \
    --input_dir /path/to/downloaded/llama/weights --model_size 7B --output_dir /output/path
```

2. Get Vicuna-7B weights by applying the delta[(detailed instructions)](https://github.com/lm-sys/FastChat/blob/main/README.md)
```bash
python3 -m fastchat.model.apply_delta \
    --base /path/to/llama-7b \
    --target /output/path/to/vicuna-7b \
    --delta path/to/vicuna-7b-delta-v1.1
```

3. Quantize Vicuna-7B model to 4-bits using [`llama.cpp`](https://github.com/ggerganov/llama.cpp)
```bash
# obtain the original LLaMA model weights and place them in ./models
ls ./models
65B 30B 13B 7B Vicuna-7B tokenizer_checklist.chk tokenizer.model

# install Python dependencies
python3 -m pip install -r requirements.txt

# convert the 7B model to ggml FP16 format
python3 convert.py models/Vicuna-7B/

# quantize the model to 4-bits (using method 2 = q4_0)
./quantize ./models/Vicuna-7B/ggml-model-f16.bin ./models/Vicuna-7B/ggml-model-q4_0.bin 2

# run the inference
./main -m ./models/Vicuna-7B/ggml-model-q4_0.bin -n 128
```
Now, the local model is ready to go!

### Install `Auto-GPT` [(Guide)](https://github.com/keldenl/gpt-llama.cpp/blob/master/docs/Auto-GPT-setup-guide.md)

1. Install this `Auto-GPT` based on the DGdev91's [PR #2594](https://github.com/Significant-Gravitas/Auto-GPT/pull/2594).
```bash
git clone https://github.com/Neronjust2017/Auto-GPT-LOCAL
``` 

2. Install requirement, and make a .env file
```bash 
    pip install -r requirements.txt
    cp .env.template .env
```

3. Edit .env file
```bash 
OPENAI_API_BASE_URL=http://localhost:443/v1
    
# you can find proper value for different LLaMA model at https://huggingface.co/shalomma/llama-7b-embeddings#quantitative-analysis
EMBED_DIM=4096 
OPENAI_API_KEY= ../llama.cpp/models/vicuna/13B/ggml-vicuna-unfiltered-13b-4bit.bin
```
4. Run Auto-GPT
```
# On Linux or Mac:
./run.sh start
# On Windows:
.\run.bat

# or with python
python -m autogpt
```