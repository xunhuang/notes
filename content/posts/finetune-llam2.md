+++
title = 'Finetuning Llama 2'
date = 2023-09-13T17:42:59-07:00
draft = false
+++

## Resource

- [Fine-tune Llama 2 with LoRA for Question Answering] (https://deci.ai/blog/fine-tune-llama-2-with-lora-for-question-answering/)
- [colab notebook](https://colab.research.google.com/drive/1vpVUT3rGWbLpOyUm3zaVM6D2EONw1xPp?usp=sharing) 
- [Similar Notebook with more details in model manipulation](https://colab.research.google.com/drive/1PEQyJO1-f6j0S_XJ8DV50NkpzasXkrzd?usp=sharing)

### how do these perform anecdotally?  

To measure the performance of a fine-tuned machine learning model, we need to make sure

- The new fine-tuned model is performing well with new dataset
- The new model doesn't regress on existing dataset (like you learn something new but forget about your old skills)

Here is the one example performed by this [Colab notebook](https://colab.research.google.com/drive/1vpVUT3rGWbLpOyUm3zaVM6D2EONw1xPp?usp=sharing) You can try this with the default T4 GPU, or better with A100 GPU(paid version) for faster performance.

Before fine-tuning, using the base pretrained model "NousResearch/Llama-2-7b-chat-hf" on HuggingFace, not that bad 
as it is already pretrained to have chat as capabilities. 

```
Xformers is not installed correctly. If you want to use memory_efficient_attention to accelerate training use the following command to install Xformers
pip install xformers.
/usr/local/lib/python3.10/dist-packages/transformers/generation/utils.py:1270: UserWarning: You have modified the pretrained model configuration to control generation. This is a deprecated strategy to control generation and will be removed soon, in a future version. Please use a generation configuration file (see https://huggingface.co/docs/transformers/main_classes/text_generation )
  warnings.warn(
<s>[INST] How do I use the OpenAI API? [/INST]  The OpenAI API is a set of RESTful APIs that provide access to various AI models and services developed by OpenAI. everybody can use the OpenAI API to build AI-powered applications and services. Here are the general steps to use the OpenAI API:

1. Sign up for an OpenAI account: To use the OpenAI API, you need to sign up for an OpenAI account. You can sign up for a free account on the OpenAI website.
2. Obtain an API key: After signing up for an OpenAI account, you will receive an API key that you can use to authenticate your API requests.
3. Choose the API endpoint: OpenAI provides several API endpoints for different AI models and services, such as the GPT-2 language model, the CLIP image recognition model, and the DALL-E text-to-image model. Choose the API endpoint that best fits your needs.
4. Send a request: Once you have chosen the API endpoint, you can send a request to the OpenAI API using your API key. The request should include the necessary parameters and data for the API endpoint.
5. Receive a response: After sending a request to the OpenAI API, you will receive a response that contains the results of the API call. The response may include data in various formats, such as JSON or XML.
6. Use the results: Once you have received the results of the API call, you can use them to build your AI-powered application or service. For example, you can use the results to generate text, create images, or classify data.

Here are some examples of how to use the OpenAI API:

* GPT-2 language model: You can use the GPT-2 API to generate text based on a given prompt or input. For example, you can send a request to the API with a prompt and receive a generated text response.
* CLIP image recognition model: You can use the CLIP API to classify images or generate image descriptions. For example, you can send a request to the API with an image URL and receive a description of the image.
* DALL-E text-to-image model: You can use the DALL-E API to generate images based on a given text description. For example, you can send a request to the API with a text description and receive a generated image response.

OpenAI provides documentation and examples for each of its API endpoints, which can help you get started with using the API. Additionally, OpenAI offers a Python library called `openai` that can simplify the process of using the API.

Keep in mind that the OpenAI API is a powerful tool that can be used for a wide range of AI applications and services. However, it is important to use the API responsibly and ethically, and to comply with any applicable laws and regulations.
```

After it's finetune with a small 1000 instruction from dataset "mlabonne/guanaco-llama2-1k" on HuggingFace.

```

<s>[INST] How do I use the OpenAI API? [/INST] The OpenAI API is a RESTful API that allows you to interact with the OpenAI model. It is built on top of the OpenAI CLI and provides a simple way to interact with the model.

To use the OpenAI API, you will need to install the OpenAI CLI and create an OpenAI account. Once you have done this, you can use the OpenAI API to interact with the model.

Here is an example of how to use the OpenAI API to get the model's predictions with max_length 400, not bad with examples


import requests

. Set the API endpoint
url = "https://api.openai.com/v1/models/<model_id>/predictions"

. Set the API headers
headers = {
    "Authorization": "Bearer <access_token>",
    "Content-Type": "application/json"
}

. Set the API payload
payload = {
    "input": "This is a test input."
}

. Make the API request
response = requests.post(url, headers=headers, data=payload)

. Check the API response
if response.status_code == 200:
    print("Predictions:")
    print(response.json())
else:
    print("Error:")
    print(response.json())


This code will send a POST request to the OpenAI API with the input "This is a test input." and will print the predictions.

You can also use the OpenAI API to get the model's parameters, which can be useful for fine-tuning the model.

Here is an example of how to use the OpenAI API to get the model's parameters:
```

It's a bit weird that the loss didn't converge

```
Step	Training Loss
25	1.345800
50	1.622300
75	1.210700
100	1.440200
125	1.175200
150	1.357600
175	1.169300
200	1.452000
225	1.150800
250	1.520100
```

Now trying max_length = 1000
![max_length 1000 ](../images/finetune-1000.png)

The result didn't seem better with longer length and it appears to be repeating itself.

Now trying max_length = 2000
![max_length 2000 ](../images/finetune-2000.png)

This offered "bert generate", which doesn't seem correct 

##  Others

- [Fine Tuning/GGML Quantization on Apple Silicon with SuperAdapter](https://www.reddit.com/r/LocalLLaMA/comments/15y9m64/fine_tuningggml_quantiziation_on_apple_silicon/?share_id=NzuooTD-GpE2r5igtN39C&utm_content=1&utm_medium=ios_app&utm_name=ioscss&utm_source=share&utm_term=1) - Not ready for prime time, one epoch takes 24 hours. 