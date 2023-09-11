+++
title = 'Llama 2 Inferencing Basics'
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

## Inferencing on M1/M2 Mac

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
from Meta, produce a GGUF model, demystifying the GGUF conversion process. 

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

It appears that gguf is a format that is architecture independent, you can run it anywhere. llama.cpp just makes it runnable on mac, with optimization it uses METAL to run even faster.


### So they are runnable now, why quantize even further?  
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

## Llama.cpp on google Colab

This google Colab 
[note book]( https://colab.research.google.com/drive/1ZdRhLo06WJaX9KP4KzWcqHXMsmWJaAxb?usp=sharing)
is based on llama-2-13B-chat module and works as of  9/8/2023

Inferencing for llama-2-13B-chat on T4 Runtime take 2 minutes, a bit long.

The inferencing for llama-2-7B-chat on this [note book](https://colab.research.google.com/drive/1h4NXWutecKMs2jEqulbNgoEQ0sbrkoqW?usp=sharing)
of T4 Runtime takes 23s. 


## Google Colab without Llama.cpp

Llama.cpp performs quantization and allows such model to be run in lower performing GPU. While great, I still want to run the
Llama 2 model on pytorch directly, without the *magic* for Llama.cpp, to get the ultimate inference quality and experience the slow down
so I can appreciate the llama.cpp magic.

It was difficult to find a notebook that does inferencing without llama.cpp (perhaps for good reasons).  Here is one that find and [copied](https://colab.research.google.com/drive/16uLq1YJM0A4sJwJgf19Cf15RNP1qbStQ?usp=sharing) and it does the following: 

- It downloads the llama 2 (7B-chat) model, directly from Meta, instead of using someone else's like TheBloke converted model. The process is a bit awkward as you first have to go to Meta website to request a download link (which expires in 24 hours), and you copy the download link into the notebook, then it can perform the download for you. 
- It runs torchrun and a simply python loop to ask user for a prompt and run inference on the prompt.

I wish it was this simple. :) this didn't work, and it's because Google Colab by default runs on the T4 GPU, which can't run even the smallest Llama 2 (7B) model and it gives
a cryptic error message

```
error:torch.distributed.elastic.multiprocessing.api:failed
```

In order to run it with larger runtime, I had to switch to a paid plan ("Pay as you go" for $9.99 for 100 Compute Unit, whatever that means). I tried to choose a V100 GPU but I go same error. I then selected the A100 GPU but google told me they don't have A100 GPU right now and then default back to V100 (which I didn't notice until failing a few times). Another random tries later, I got a A100 by chance. Then my run was successful.

```
> initializing model parallel with size 1
> initializing ddp with size 1
> initializing pipeline with size 1
Loaded in 16.90 seconds
chat prompt (or 'exit' to quit): whats your name
User: whats your namen
> Assistant:  Hello! My name is LLaMA, I'm a large language model trained by a team of researcher at Meta AI. 😊
n==================================n
chat prompt (or 'exit' to quit): who created you
User: who created youn
> Assistant:  I was created by a team of researcher at Meta AI. I'm a large language model, my knowledge was built from a massive corpus of text, including books, articles, and websites, and I was trained using a variety of machine learning algorithms. My creators are a group of researcher at Meta AI, who are constantly working to improve my performance and add new features. 😊
n==================================n
chat prompt (or 'exit' to quit): the first man on the moon was
User: the first man on the moon wasn
> Assistant:  The first man to walk on the moon was Neil Armstrong. He stepped onto the moon's surface on July 20, 1969, during the Apollo 11 mission. Armstrong famously declared, "That's one small step for man, one giant leap for mankind," as he became the first person to set foot on the lunar surface.
n==================================n
chat prompt (or 'exit' to quit): tell me more
User: tell me moren
```

Now I'm paying more attention the pane on the right on System RAM,  GPU RAM, Disk space  for A100

![A100](/a100-google-colab.png 'A100') 

I repeated the run with a V100, it looks like the _System RAM_ is what got me. 

![V100](/v100-google-colab.png 'V100') 

This is why you don't see google colab notebooks for running Llama 2 models directly and instead rely on other quantized models 

- you have to download the model, with expiring links, from Meta (Meta, please make this less friction)
- More importantly, Google Colab's free GPU can run the inferencing directly, and even the paid version you have to fight with other for availability of A100 GPUs. 

For now, the solution is get quantized model (small/and fast) and run them on free version of the google Colab.

## Running them on AWS.... do we really need llama.cpp to run these? 

Amazon SageMaker has a number of notebook that's well-configured. You can deploy a server with a notebook and you can directly run inference. It's nice and simple,
but you can probably get to see the wrapping code on how it is run. It is a little bit too well-package for my taste.

Many youtube videos online for if you search for 'llama 2 amazon sagemaker"
