# Project Title - Rephrasing Product Descriptions using LLMs for SEO
Rephrasing 7000+ product descriptions using OpenAI API to optimize them for SEO.

# Main Libraries - 
1) Transformers - (AutoTokenizer, AutoModelForCasualLM, BitsAndBytesConfig)
2) Pandas, Scikit-learn, PyTorch
3) Google Colab
4) OpenAI
5) Pickle, Rouge Scorer, Spacy

# LLMs Used - 
1) Open Source Models - Mistral-7B-Instruct, Qwen2-7B-Instruct, Phi-3-medium-128k-instruct, DeepSeek-LLM-7B-Chat
2) Gated Models - Llama-3-7B, Nous-Hermes-2-Mistral-7B-DPO
3) Closed - GPT-4o-mini, GPT-4.1-mini

# Model Evaluation - 
1) Semantic Similarity (BERTScore, Cosine Similarity)
2) Word Overlap (ROUGE, BLEU Score)
3) Text Length Ratio
4) Numbers, Entities names preserved

# Workflow - 
1) Loading the file containing the product descriptions.
2) Clean the dataset - Removing the HTML format in the product description text.
3) Prompt engineering - generating appropriate prompt and iteratevely improving it.
5) Experimenting rephrasing with a sample of 50 product descriptions and different open-source, closed and gated LLMs.
6) Model evaluation & iteratively improving based on the output.
7) Final model selection & running the rephrasing process on the entire dataset.

# Optimizations - 
1) Model Quantization, Distributed Processing
2) Batch API - Reduces HTTP requests to OpenAI. In one request, send multiple rows to process. As the records are processed asynchronously on the OpenAI servers, this reduces the API cost by ~50%.  
3) Prompt Caching - Automatically applied by OpenAI, when it detects repeatable patterns in [System + User Prompt]

# Tech Stack - 
1) Google Colab - L4 GPU (System RAM - 53GB, GPU RAM - 22.5GB, Disk - 235GB)
2) Python
3) OpenAI API
4) LLMs
5) Hugging Face Transformers
6) Prompt Engineering
7) n8n

