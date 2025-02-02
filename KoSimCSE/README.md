# KoSimCSE
[[Github]](https://github.com/princeton-nlp/SimCSE) Official implementation of SimCSE. <br>
KoSimCSE : Korean Sentence Embeddings using contrastive learning.

## Quick start
- If you want to do inference quickly, download the pre-trained models and then you can start some downstream tasks.
```
bash get_model_checkpoint.sh
python SemanticSearch.py
```

## Training 
- Before training or evaluation, please download the datasets by running
```
bash get_model_dataset.sh
```
### Train KoSimCSE (Supervised Only)
  ```
  python main.py \
    --model klue/bert-base \
    --test False \
    --max_len 50 \
    --batch_size 256 \
    --epochs 2 \
    --eval_steps 125 \
    --lr 0.0001 \
    --warmup_ratio 0.1 \
    --temperature 0.05 \
    --path_to_data ../Dataset/ \
    --train_data train_nli.tsv \
    --valid_data valid_sts.tsv
  ```
### Evaluation
  ```
  python main.py \
    --model klue/bert-base \
    --train False \
    --test True \
    --max_len 50 \
    --batch_size 256 \
    --temperature 0.05 \
    --path_to_data ../Dataset/ \
    --test_data test_sts.tsv \
    --path_to_saved_model output/kosimcse-klue-bert-base.pt
  ```

### Run Examples
```
bash run_example.sh
```
### Hyperparameters
- Train KoSimCSE (BERT BASE)
  1. Pooling Method: [CLS] strategy
  2. Batch Size: 256
  3. Evaluation Steps: 125
  4. Epochs: 2
  5. Token Max Length: 128
  6. Learning Rate: 0.0001
  7. Warmup Ratio: 0.1
  8. Temperature: 0.05
  
- Train KoSimCSE (RoBERTa BASE)
  1. Pooling Method: [CLS] strategy
  2. Batch Size: 256
  3. Evaluation Steps: 125
  4. Epochs: 2
  5. Token Max Length: 128
  6. Learning Rate: 0.0001
  7. Warmup Ratio: 0.05
  8. Temperature: 0.05

### Semantic Search
```
python SemanticSearch.py
```
```python
from model.simcse.bert import BERT
from transformers import AutoModel, AutoTokenizer

def main():
    model = BERT(AutoModel.from_pretrained('BM-K/KoSimCSE-roberta'))
    tokenizer = AutoTokenizer.from_pretrained('BM-K/KoSimCSE-roberta')

    model.to(device)
    model.eval()
   
    model, tokenizer, device = example_model_setting(model_name)

    # Corpus with example sentences
    corpus = ['한 남자가 음식을 먹는다.',
              '한 남자가 빵 한 조각을 먹는다.',
              '그 여자가 아이를 돌본다.',
              '한 남자가 말을 탄다.',
              '한 여자가 바이올린을 연주한다.',
              '두 남자가 수레를 숲 속으로 밀었다.',
              '한 남자가 담으로 싸인 땅에서 백마를 타고 있다.',
              '원숭이 한 마리가 드럼을 연주한다.',
              '치타 한 마리가 먹이 뒤에서 달리고 있다.']

    inputs_corpus = convert_to_tensor(corpus, tokenizer, device)

    corpus_embeddings = model.encode(inputs_corpus, device)

    # Query sentences:
    queries = ['한 남자가 파스타를 먹는다.',
               '고릴라 의상을 입은 누군가가 드럼을 연주하고 있다.',
               '치타가 들판을 가로 질러 먹이를 쫓는다.']

    # Find the closest 5 sentences of the corpus for each query sentence based on cosine similarity
    top_k = 5
    for query in queries:
        query_embedding = model.encode(convert_to_tensor([query], tokenizer, device), device)

        cos_scores = pytorch_cos_sim(query_embedding, corpus_embeddings)[0]
        cos_scores = cos_scores.cpu().detach().numpy()

        top_results = np.argpartition(-cos_scores, range(top_k))[0:top_k]

        print("\n\n======================\n\n")
        print("Query:", query)
        print("\nTop 5 most similar sentences in corpus:")

        for idx in top_results[0:top_k]:
            print(corpus[idx].strip(), "(Score: %.4f)" % (cos_scores[idx]))
```

- Results are as follows :

```

Query: 한 남자가 파스타를 먹는다.

Top 5 most similar sentences in corpus:
한 남자가 음식을 먹는다. (Score: 0.6141)
한 남자가 빵 한 조각을 먹는다. (Score: 0.5952)
한 남자가 말을 탄다. (Score: 0.1231)
한 남자가 담으로 싸인 땅에서 백마를 타고 있다. (Score: 0.0752)
두 남자가 수레를 숲 솦으로 밀었다. (Score: 0.0486)


======================


Query: 고릴라 의상을 입은 누군가가 드럼을 연주하고 있다.

Top 5 most similar sentences in corpus:
원숭이 한 마리가 드럼을 연주한다. (Score: 0.6656)
치타 한 마리가 먹이 뒤에서 달리고 있다. (Score: 0.2988)
한 여자가 바이올린을 연주한다. (Score: 0.1566)
한 남자가 말을 탄다. (Score: 0.1112)
한 남자가 담으로 싸인 땅에서 백마를 타고 있다. (Score: 0.0262)


======================


Query: 치타가 들판을 가로 질러 먹이를 쫓는다.

Top 5 most similar sentences in corpus:
치타 한 마리가 먹이 뒤에서 달리고 있다. (Score: 0.7570)
두 남자가 수레를 숲 솦으로 밀었다. (Score: 0.3658)
원숭이 한 마리가 드럼을 연주한다. (Score: 0.3583)
한 남자가 말을 탄다. (Score: 0.0505)
그 여자가 아이를 돌본다. (Score: -0.0087)
```
