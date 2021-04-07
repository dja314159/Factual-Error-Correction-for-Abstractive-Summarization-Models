# Factual-Error-Correction-for-Abstractive-Summarization-Models
Paper rewiew_Factual Error Correction for Abstractive Summarization Models
Link : https://arxiv.org/abs/2010.08712

## 서론

### 배경

- Self-supervised 방법이 NLP 분야에서 많은 성공을 거두었고 automatic summarization에도 도입1).
- 성능이 좋은 abstractive summarization(이하 추상요약) 모델은 transformer-based 요약 모델을 finetune 한 것이 대부분2).
- ROUGE score3) 측면에서는 이전보다 성능이 향상되었지만 source와 summary 간의 factual consistency는 아직 부족.
- 추상요약 모델은 30%정도 factual error를 포함4)
- Fact extraction과 fact triple에 attention을 적용한 방식5), QA모델을 적용한 방식6), 그리고 artificial dataset으로 모델을 학습시킴7).

  => 이러한 방식들의 대부분은 높은 수준의 fact-extraction model도 필요로 함. 또는 fact evaluation에만 집중한다. 그래서 틀린 부분을 수정하는 것은 많이 개발 되지 않음. 



1) Multi-News: a Large-Scale Multi-Document Summarization Dataset and Abstractive Hierarchical Model 
2) Attention Is All You Need
3) ROUGE-N: unigram, bigram, trigram 등 문장 간 중복되는 n-gram을 비교하는 지표
4) Faithful to the Original: Fact Aware Neural Abstractive Summarization
5) Optimizing the factual correctness of a summary: A study of summarizing radiology reports, Assessing the factual accuracy of generated text.
6) Ranking generated summaries by correctness: An interesting but challenging application for natural language inference, Ensure the correctness of the summary: Incorporate entailment knowledge into abstractive sentence summarization, Asking and answering questions to evaluate the factual consistency of summaries.
7) Evaluating the factual consistency of abstractive text summarization


### 제안 및 모델 예시

- 따라서, 본 논문에서는 post-editing correction을 통한 system summary의 factual correction을 향상하는 모델을 제안한다.
![image](https://user-images.githubusercontent.com/62656584/113813605-2ae84900-97ab-11eb-881e-8e921396c497.png)
- 요약이 모델을 통과해서 나온 결과이고 France’s 라는 오류가 Israel’s 로 고쳐 짐
- 추상 요약 모델의 결과물과 source document를 통해 최종 요약을 생성함
- 뿐만 아니라, 추상 요약에 대한 fact evaluation model로도 사용이 가능함.
  -  추상 요약을 수정한다면 틀린 문장, 수정하지 않는다면 맞는 문장

- 모델 학습에는 다른 논문에 사용된 artificial data를 사용함.
  -  Paraphrasing, Entity and Number swapping, Pronoun swapping 등이 인위적으로 만들어진 요약들

- 그러나, 모델 결과물의 recall은 좀 낮은데 이는 artificial data와 추상 요약의 에러 분포 간 차이가 원인인 것으로 추정됨




## 관련 연구

### Factual consistency를 위한 추상 요약 모델 연구가 진행중

- Cao et al., 2018(원문과 원문에서 추출한 relation triple을 사용)
- Zhang et al., 2019b(방사선과에서 얻은 데이터로 강화 학습)
- Li et al., 2018(Encoder를 summarization과 NLI tasks 모두에 대하여 학습)
- Guo et al.,2018(추상 요약 모델을 질문과 함의 생성으로도 학습시킴)
- Kumar and Cheung., 2019(추상 요약이 뒷 내용에 더 weight를 두는 현상 발견)
- Dong et al., 2020(QA모델을 활용한 사실 교정 모델을 제안함)


### 추상 요약의 Factual consistency 여부를 평가하는 연구가 진행중

- Goodrich et al., 2019(원문,요약문 fact triple에 대해 Wikidata와 비교)
- Falke et al., 2019(entailment model을 적용)
- Wang et al., 2020(요약문과 원문에 question을 통해 측정)
- Kry´sci´nski et al., 2019(임의로 생성된 틀린 요약문으로 BERT모델을 finetune 시켜 factual checking)




## 제안방법


### Dataset

- 기존 논문1)은 원문에서 가져온 문장에 error를 발생시켰지만 여기는 정답 요약에 error를 추가.
- 원문을 𝑑, 정답 요약을 𝑠라 함.
- 그리고 일정한 확률 𝛼에 따라 𝑠를 수정해 사실인지 알 수 없는𝑠′ 를 만들어 냄.
- 기존 논문2)에 근거하여 실제 틀린요약 비율과 같은 𝛼 = 0.3으로 하고 𝑠′의 70%는 수정을 거치지 않은 요약. 
- 학습데이터는 (𝑠′,𝑠,𝑑)의 triplet을 포함.




1) Kry´sci´nski et al., 2019
2) Cao et al., 2018
  


### Error Corruption

- Entity, Number, Date, Pronoun에 error를 발생시킴. 
  - Entity, Number, Date 같은 경우에는 요약문의 entity와 똑같은 type의 무작위 entity를 원문에서 찾아 바꿈
  - Pronoun의 경우 syntactic적으로 일치하는 다른 pronoun으로 대체됨

![image](https://user-images.githubusercontent.com/62656584/113815030-83b8e100-97ad-11eb-8c22-43a0e501b125.png)
(Number가 corruption된 example)


### 학습목표

- 학습목표는 (𝑠′,𝑠,𝑑)의 triplet에서 사실인지 알 수 없는 𝑠′와 원문 𝑑를 바탕으로  𝑠를 생성하는 것.
- 𝑃(𝑠|𝑠′,𝑑)의 likelihood를 maximizing하는 문제로 표현 할 수 있다.
- Encoder 에는 𝑠′,𝑑를 concatenate 하여서 input하고 decoder가 s를 생성하도록 학습 시킨다. 


### 모델

- noise function으로 손상된 text를 복구할 수 있도록 pretrain된, BART를 사용.
  - BART는 text transformation이나 token deletion과 같은 input이 주어졌을때 original sentence를 출력하도록 학습 됨

- BART가  본 논문의 task와 유사하게 pretrained 되었기 때문에 corrupted 혹은 generated summary 를 noisy input이라 여기고 여기서 inconsistent 한 content를 noise라고 볼 수 있음




## 실험

- 실험 데이터는 2개를 이용함

1. CNN/DailyMail dataset을 artificially corrupt함
  
  - 총 287,227개의 샘플 중 30%(85,583)을 corrupt 했고 date/entity/number/pronoun은 각각 16,858/35,113/13,408/20,204 만큼 변형.
  - 학습데이터는 평가에 사용할 corrupted sample 5,780개, clean sample 5,710개를 제외한 275,737개를 사용함(약 96%)

2. Kry´sci´nski et al., 2019에서 썼던 K2019데이터를 사용
  
  - 추상 요약 모델들이 생성한 503개의 data를 사용
  - Consistency 여부는 각 요약들에 label되어 있음


- 실험 과정

1. Factual consistency checking

  - 원문에 빗대어 input summary가 consist한지 판별
  - Precision, Recall, F1을 이용해서 Accuracy라는 척도로 성능 측정
  - 추상 요약을 수정한다면 틀린 문장, 수정하지 않는다면 맞는 문장

2. Error correction
  
  - 해당 task에서 모델은 input summary의 inconsistency를 무조건 correct 해야함
  - 해당 모델이 알맞게 고친 비율을 correction accuracy라고 정의함
  - Artificial dataset같은 경우 input summary(𝑠^′)가 정답 요약(𝑠)과 일치한다면 맞다고 간주함
  - K2019 dataset은 정답 요약이 따로 없으므로 human evaluation으로 평가함.

- 실험 환경
  
  - pre-trained BART model 이 training dataset에 10 epochs 만큼 fine-tuned 됨
  - learning rate 는 3e-5
  - 4 NVIDIA Tesla V100 GPUs
  - takes about 12 hours.




## 결과

### Artificial Dataset

- Consistency checking 결과 
![image](https://user-images.githubusercontent.com/62656584/113817866-f6c45680-97b1-11eb-872e-d45a04b42f5c.png)

- Error correction 결과
  - Test set의 5,780개의 corrupted summary중 62.13%가 정확히 수정됨
  - Test set의 5,710개의 clean summary중 26.27%가 수정되었고 73.73%가 알맞게 수정됨



### K2019

![image](https://user-images.githubusercontent.com/62656584/113817951-0fcd0780-97b2-11eb-9111-f0c0019bb87a.png)

- Consistency checking 결과, BERT 모델에 비해 성능은 좋지만, Fact CC에 비해서 좋지못하다.

* 2021.04.01 : 해당 Table 수정된 논문 re-submission됨.(반영완료)


![image](https://user-images.githubusercontent.com/62656584/113818060-368b3e00-97b2-11eb-9b91-89eeed1a8e5f.png)
(Human evaluation 결과.)

- Inconsistent sample중 19개를 수정했으나 11개만 알맞게 수정되었다.
- Consistent sample중 39개를 수정했는데 5개의 의미가 변화되었다.
  =>  17.74%를 알맞게 수정했으나 1.13%로 맞는 문장을 틀리게 바꾸었다.
  =>  Artificial dataset 실험에서 62.13%를 correction하는 수치에는 못 미치는걸 확인할 수 있고 이는 설정 문제로 추정.
  =>  Summarization system의 다양한 error type을 represent 할 수 없는 것 같음.




![image](https://user-images.githubusercontent.com/62656584/113818176-620e2880-97b2-11eb-8563-d11ff65ca6a1.png)
- 첫번째 예시를 잘 수정됨을 알 수 있고 두번째 예시를 보면 147은 19로 알맞게 바뀐 것을 알 수 있지만 “including 142 students”라는 오류는 수정되지 않았음을 볼 수있다. 세번째, 네번째 예시는 consistent한 요약도 수정시키는 모습을 볼 수 있다.



## 결론

- 본 논문에서는 추상 요약의 오류를 수정하는 방법을 제안함
- 이전 모델들에 비해 성능 향상을 보임

- Future Work
  =>  Inconsistent summary에서의 낮은 Recall 수치와 false positive sample을 해결할 방안이 필요











