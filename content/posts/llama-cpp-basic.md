+++
title = 'Llama Cpp 101'
date = 2023-09-08T06:21:08Z
draft = false
+++

## Getting to "Hello World"

The world of LLM is a wild. Getting a prompt from OpenAI or trying demos on HuggingFace is easy
but I need to run things on my own, following the playbook of *Replicate*, *modify*, *innovate*, first on my own hardware, then on google colab, and eventually on my own AWS/Google Cloud instance. After the basics, I proceed to think about devops and mlops. 

The ML terminology in the eyes of non-ML person

- Model --->  a function 
- Inferencing ---> invoking a function
- Training --> writing/build a function
- Quantize --> shrinking/optimizing the size of function, make it fast, and trade off against quality of the output

## Inferenceing on M1/M2 Mac

Project [llama.cpp ](https://github.com/ggerganov/llama.cpp) makes this easy.

Let's skip the original model, go use a pre-converted model directly first.

```
curl -L https://huggingface.co/TheBloke/Llama-2-13B-chat-GGUF/resolve/main/llama-2-13b-chat.Q5_0.gguf  --remote-name
./main -m llama-2-13b-chat.Q5_0.gguf -p "Building a website can be done in 10 simple steps:\nStep 1:" -n 400 -e
```

This failed with a weird error message.
```
GGML_ASSERT: ggml-metal.m:1015: false && "not implemented"
```

Looked at the [code](https://github.com/LostRuins/koboldcpp/blob/concedo/ggml-metal.m#L1015). Ok, Q5_0 is not supported, let's try something else.

```
                    case GGML_OP_GET_ROWS:
                        {
                            switch (src0->type) {
                                case GGML_TYPE_F16:  [encoder setComputePipelineState:ctx->pipeline_get_rows_f16];  break;
                                case GGML_TYPE_Q4_0: [encoder setComputePipelineState:ctx->pipeline_get_rows_q4_0]; break;
                                case GGML_TYPE_Q4_1: [encoder setComputePipelineState:ctx->pipeline_get_rows_q4_1]; break;
                                case GGML_TYPE_Q8_0: [encoder setComputePipelineState:ctx->pipeline_get_rows_q8_0]; break;
                                case GGML_TYPE_Q2_K: [encoder setComputePipelineState:ctx->pipeline_get_rows_q2_K]; break;
                                case GGML_TYPE_Q3_K: [encoder setComputePipelineState:ctx->pipeline_get_rows_q3_K]; break;
                                case GGML_TYPE_Q4_K: [encoder setComputePipelineState:ctx->pipeline_get_rows_q4_K]; break;
                                case GGML_TYPE_Q5_K: [encoder setComputePipelineState:ctx->pipeline_get_rows_q5_K]; break;
                                case GGML_TYPE_Q6_K: [encoder setComputePipelineState:ctx->pipeline_get_rows_q6_K]; break;
                                default: GGML_ASSERT(false && "not implemented");
                            }
```

OK, try Q5_K model(s) instead. I looked at here(https://huggingface.co/TheBloke/Llama-2-13B-chat-GGUF/tree/main) and tried again. 

```
curl -L https://huggingface.co/TheBloke/Llama-2-13B-chat-GGUF/resolve/main/llama-2-13b-chat.Q5_K_M.gguf --remote-name
./main -m llama-2-13b-chat.Q5_K_M.gguf -p "Building a website can be done in 10 simple steps:\nStep 1:" -n 400 -e

...
Building a website can be done in 10 simple steps:
Step 1: Define Your Objective - Determine the purpose of your website and what you want to achieve with it. This will help guide the rest of the process.

Step 2: Choose a Domain Name - Select a domain name that is easy to remember, relevant to your content, and available for registration.

Step 3: Choose a Web Hosting Provider - Research and select a reliable web hosting provider that meets your needs in terms of pricing, features, and customer support.
```

This works. You might see documentations in other place for GGML format , which was deprecated recently by GGUF, resulting in many blog posts not working. GGUF as of 9/8/2023 is the latest hotness.

### Getting the original Llama model from Meta

While you can download GGUF models and inference directly, however, I want to start from the original model
from Meta, produce a GGUF model, demystifiying the GGUF conversion process. 

Getting Meta's original model requires this [website](https://ai.meta.com/resources/models-and-libraries/llama-downloads/) and submit a download request. You will get an email that directs you to git clone a repo to get a download.sh file.

```
git clone git@github.com:facebookresearch/llama.git
cd llama
sh download.sh
# copy in the download link from email 
# hit Enter to download all 
```

Now move the one specific model directly like "llama-2-7b-chat" to llama.cpp's models directory.

### Converting Meta Model --> GGUF  

```
#install python binding.... (so it can run convert)
python3 -m pip install -r requirements.txt
cp $(wherever_meta_llama_directory)/tokenizer.model models.
python3 convert.py models/llama-2-7b-chat

# this took about 18 seconds on m1 macbook pro
... 
Wrote models/llama-2-7b-chat/ggml-model-f16.gguf
```
Now run interface with this new model, which is converted from original llama model by us. 

```
# this took 10s of seconds when started cold, but second and third run was much faster
./main -m models/llama-2-7b-chat/ggml-model-f16.gguf -p "the first man to walk on the moon was" -n 400 -e
... 
generate: n_ctx = 512, n_batch = 512, n_predict = 400, n_keep = 0

 the first man to walk on the moon was Neil Armstrong. The Apollo 11 mission, which included Armstrong and fellow astronauts Edwin "Buzz" Aldrin, landed on the lunar surface on July 20, 1969, at 2:56 UTC (Coordinated Universal Time). Armstrong radioed back to Mission Control on Earth, "That's one small step for man, one giant leap for mankind."
 Armstrong and Aldrin spent about two-and-a-half hours on the lunar surface, collecting samples and conducting experiments before returning to the lunar module Eagle and heading back to Earth. The mission was a historic achievement that marked the first time humans had visited another celestial body.
```

It appears that gguf is a format that is architecture independent, you can run it anywhere. llama.cpp just makes it runable on mac, with optimization it uses METAL to run even faster.

### So they are runable now, why quantize even further? 
- because you want to run them faster.... 
- because you want run on even smaller devices...
- because you want run at scale 

### how to quantize further?
```
./quantize ./models/llama-2-7b-chat/ggml-model-f16.gguf ./models/llama-2-7b-chat/ggml-model-q4_0.gguf q4_0

ls -la models/llama-2-7b-chat
total 60190768
drwxr-xr-x  7 xhuang  staff          224 Sep  8 15:27 .
drwxr-xr-x  7 xhuang  staff          224 Sep  8 15:08 ..
-rw-r--r--  1 xhuang  staff          100 Jul 14 00:12 checklist.chk
-rw-r--r--  1 xhuang  staff  13476925163 Jul 13 16:02 consolidated.00.pth
-rw-r--r--  1 xhuang  staff  13478104576 Sep  8 15:23 ggml-model-f16.gguf
-rw-r--r--  1 xhuang  staff   3825806912 Sep  8 15:28 ggml-model-q4_0.gguf
-rw-r--r--  1 xhuang  staff          102 Jul 13 16:02 params.json

time ./main -m models/llama-2-7b-chat/ggml-model-q4_0.gguf -p "the first man to walk on the moon was" -n 400 -e

generate: n_ctx = 512, n_batch = 512, n_predict = 400, n_keep = 0

 the first man to walk on the moon was Neil Armstrong. Unterscheidung -- Armstrong stepped off of the lunar module Eagle and onto the moon's surface on July 20, 1969, famously declaring "That's one small step for a man, one giant leap for mankind" as he took his first steps. Armstrong was born on August 5, 1930, in Wapakoneta, Ohio, and served in the United States Navy from 1950 to 1954 during the Korean War. After graduating with a degree in aeronautical engineering from Purdue University in 1955, Armstrong became a test pilot at Edwards Air Force Base in California. He joined NASA's astronaut program in 1962 and was chosen as commander of the Apollo 11 mission in 1969.
Armstrong died on August 25, 2012, at the age of 82, after suffering a heart attack. Despite his historical significance, Armstrong led a humble life and rarely spoke publicly about his experiences on the moon. He did, however, make appearances at NASA events and was awarded numerous honors for his contributions to space exploration.
This passage discusses Neil Armstrong's life and accomplishments as an astronaut, including his historic first steps on the moon during the Apollo 11 mission in 1969. It also mentions his death at age 82 and his relatively private nature despite his historical significance. [end of text]

./main -m models/llama-2-7b-chat/ggml-model-q4_0.gguf -p  -n 400 -e  0.80s user 0.36s system 10% cpu 11.346 total
```
You can see the quantized size is 3.6G, reduced from the original size of 13G. It runs super fast as well.

## Llama cpp on google colab

This google colab 
[note book]( https://colab.research.google.com/drive/1ZdRhLo06WJaX9KP4KzWcqHXMsmWJaAxb?usp=sharing)
is based on llama-2-13B-chat module and works as of  9/8/2023

Inferencing for llama-2-13B-chat on T4 Runtime take 2 mintues, a bit long.

The inferencing for llama-2-7B-chat on this [note book](https://colab.research.google.com/drive/1h4NXWutecKMs2jEqulbNgoEQ0sbrkoqW?usp=sharing)
of T4 Runtime takes 23s. 

## running them on AWS.... do we really need llama.cpp to run these? 
