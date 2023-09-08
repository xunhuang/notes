+++
title = 'Llama Cpp 101'
date = 2023-09-08T06:21:08Z
draft = false
+++

## Getting to "Hello World"

The world of LLM is a wild. Getting a prompt from OpenAI or trying demos on HuggingFace is easy
but I need to run things on my own, following the playbook of *Replicate*, *modify*, *innovate*, first on my own hardware, then on google colab, and eventually on my own AWS/Google Cloud instance. After the basics, I proceed to think about devops and mlops. 

## Inferenceing on M1/M2 Mac

Project [llama.cpp ](https://github.com/ggerganov/llama.cpp) makes this easy.

Basically original Llama model --> GGUF --> Run. Let's skip the original model download and conversion initally.

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

You might see documentations on GGML format, which was deprecated recently by GGUF, resulting in many blog posts not working. GGUF as of 9/8/2023 is the latest hotness.

### Getting the original Llama model from Meta

You can find GGUF models and run directly. However, I want to start from the original model
from Meta, produce a GGUF model myself. After this works, I feel more comfortable using other
people's GGUF model, with less mystery involved. 

Getting Meta's original model was annoying. Go this website and submit a reqeust. You will get an email that directs you to git clone a download.sh file, and some URL (don't click the URL it wont' work dirctly). Selecting individual model didn't work for me. I hit "enter" and download all, which took a while.

### Converting Meta Model --> GGUF  

TBD.

```
install python binding.... (so it can run convert)
convert model
```
### Running Llama.cpp

```
./main -m llama-2-13b-chat.Q5_K_M.gguf -p "the first man to walk on the moon was" -n 400 -e
```

Lesson: it appears that gguf is a format that is architecture independnnt, you can run it anywhere. llama.cpp just makes it runable on mac, and with some optimization it uses METAL to make run even faster.

# Llama cpp on google colab

This google colab 
[note book]( https://colab.research.google.com/drive/1ZdRhLo06WJaX9KP4KzWcqHXMsmWJaAxb?usp=sharing)
is based on llama-2-13B-chat module and works as of  9/8/2023

Inferencing for llama-2-13B-chat on T4 Runtime take 2 mintues, a bit long.

The inferencing for llama-2-7B-chat on this [note book](https://colab.research.google.com/drive/1h4NXWutecKMs2jEqulbNgoEQ0sbrkoqW?usp=sharing)
of T4 Runtime takes 23s. 

### So they are runable now, why quantize even further? 
- because you want to run them faster.... 
- because you want run on even smaller devices...
- because you want run at scale 

## running them on AWS.... do we really need llama.cpp to run these? 
