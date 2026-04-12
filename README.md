# Project Title - Rephrasing Product Descriptions using LLMs for SEO
Rephrasing 2000+ product descriptions using OpenAI API to optimize them for SEO

# Main Libraries - 
AutoTokenizer - Used to load the appropriate tokenizer for a given pre-trained model, converting text into a format that the model can understand.
AutoModelForCausalLM - Used to load pre-trained causal language models (models that predict the next token in a sequence, often used for text generation).
BitsAndBytesConfig - Used to configure quantization, a technique to reduce the memory footprint and computational cost of large models by representing their weights with fewer bits.
accelerate - Helps to train and use Transformers models more efficiently on different hardware setups, including distributed training and mixed precision.

# LLMs Used - 
1) GPT-4o-mini
2) Mistral-7B-Instruct
3) Qwen2-7B-Instruct
4) Phi-3-medium-128k-instruct
5) DeepSeek-LLM-7B-Chat

# Model Evaluation - 
1) Semantic Similarity (BERTScore, Cosine Similarity)
2) Word Overlap (ROUGE, BLEU Score)
3) Text Length Ratio
4) Numbers, Entities names preserved

# Optimizations - 
1) Batch API - Reduces HTTP requests to OpenAI. In one request, send multiple rows to process. As the records are processed asynchronously on the OpenAI servers, this reduces the API cost by ~50%.  
2) Prompt Caching - Automatically applied by OpenAI, when it detects repeatable patterns in [System + User Prompt]

# Important Functions - 
1) tokenizer.apply_chat_template() - Take a list of message dictionaries and format them into a single string (or a sequence of tokens/token IDs) that adheres to the specific chat format expected by the model.

# Tech Stack - 
1) Google Colab - L4 GPU (System RAM - 53GB, GPU RAM - 22.5GB, Disk - 235GB)
2) Python
3) OpenAI API
4) LLMs
5) Hugging Face Transformers
6) Prompt Engineering
7) Make.ai

