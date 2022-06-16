# Music-Recommender-System

Content-based Music Recommender System (Lyric and Audio Combined)

### Why
기술이 발전하면서 시간과 공간의 제약없이 콘텐츠 감상하는 것이 가능해지면서 사용자가 좋아할만한 항목을 선별해주는 추천 시스템에 대한 요구가 높아지고 있다. 음악의 경우 하루 평균 발매되는 음원의 수가 수만 곡에 이르기 때문에 모두 듣고 선호하는 음악을 탐색하는 것이 불가능해졌다. 이에 희소성 문제와 콜드 스타트 문제라는 한계점을 극복하기 위해 음악 신호를 활용하여 사용자에게 음악을 추천해주는 내용기반 추천 방법이 등장하게 되었다. 음악을 들을 때 멜로디를 중점으로 듣는 사용자와 가사를 중점으로 듣는 사용자 모두에게 맞춤 추천을 해주는 가사 기반 추천시스템과 오디오 기반 추천시스템을 결합해보았다.

## Dataset
* Music Summary    
  곡 제목, 가수, 앨범명 등    
  Million Song Dataset(subset of 10,000 songs) http://millionsongdataset.com/
* User Data    
  사용자의 음악 사용기록    
  The Echo Nest Taste Profile Subset http://millionsongdataset.com/tasteprofile/
* Lyric    
  MusixMatch API 이용 https://www.musixmatch.com/
* Audio    
  MFCC feature    
  http://www.ifs.tuwien.ac.at/mir/msd/download.html
  
## Lyric Recommendation
### 1. tf-idf, Word2Vec, BERT
각 모델을 이용하여 가사 임베딩을 구한 뒤 코사인 유사도로 음악 간의 유사도를 구해 추천시스템을 구축하였다.

**추천시스템 평가**
   || Recall@10 | Recall@50 | nDCG_100|
   |-|-|-|-|
   |tf-idf | 0.01140 | 0.05302 |0.17884|
   |PCA_tf-idf | 0.01342 |0.05771 |0.15640|
   |Word2Vec|0.00202|0.20269|0.06956|
   |BERT|0.00335|0.09530|0.10969|
   
   **가사 기반 추천시스템 결과 tf-idf가 가장 성능이 좋았는데 모델을 결합하여 더 성능이 좋은 추천시스템을 구축하였다.**  

### 2. 가사 기반 추천시스템 결합
위의 PCA를 적용한 tf-idf, Word2Vec, BERT를 통해 구한 코사인 유사도를 비율을 달리해 결합해 추천시스템을 구축하였다.

**추천시스템 평가**
||Recall@10|Recall@50|nDCG_100|
|-|-|-|-|
|tf-idf + bert|0.00872|0.08322|0.12107|
|Word2Vec 7 + bert 3|0.00067|0.13670|0.14639|
|Word2Vec 8 + bert 2|0|0.16026|0.14743|

=> 가장 높은 성능을 가진 것은 Word2Vec과 BERT를 7:3으로 결합한 모델로 이를 가사 기반 추천시스템 모델로 결정하였다.  

---

## Audio Recommendation
사용한 데이터는 음악의 MFCC 특징과 템포를 사용하였다.  
### CNN
사용한 모델은 CNN으로 장르별로 분류하는 CNN 모델을 훈련시킨 후 마지막 레이어인 softmax layer를 제외한 모델을 통과시킨 output이 latent representation이 되도록 하였다.  

**CNN parameter별 추천시스템 평가**
||Recall@10|Recall@50|nDCG_100|
|-|-|-|-|
|fil16 + ker7|0.00202|0.13198|0.06333|
|fil16 + ker4|0.00471|0.10437|0.06437|
|fil32 + ker4|0.00404|0.11245|0.07891|

=> filter = 32, kernel_size = 4인 CNN 모델을 최종 오디오 기반 추천시스템 모델로 결정하였다.

성능을 높이기 위해 parameter를 바꾼 결과 최종적으로 사용한 CNN 모델의 구조
```
model = Sequential()
model.add(Conv1D(filters=32, kernel_size=4, kernel_initializer = initializers.he_normal(seed=1), activation="relu", input_shape = (27, 1)))
model.add(Flatten())
model.add(Dense(64, activation="relu", kernel_initializer=initializers.he_normal(seed=1)))
model.add(Dense(n_classes, activation="softmax", kernel_initializer=initializers.he_normal(seed=1)))
model.compile(loss="categorical_crossentropy", optimizer=optimizers.Adam(lr=0.0001), metrics=['accuracy'])
```

---
## Content-based Music Recommendation
가사 중점과 오디오 중점으로 듣는 사용자에게 맞춤 음악을 추천해주기 위해 가사 기반 추천시스템과 오디오 기반 추천시스템을 결합하였다.  

||Recall@10|Recall@50|nDCG_100|
|-|-|-|-|
|lyr5 + aud5|0.00128|0.14401|0.11684|
|lyr3 + aud7|0.00264|0.14334|0.08937|
|lyr2 + aud8|0.00302|0.15145|0.09304|

=> 최종적으로 가사 기반 추천시스템의 유사도와 오디오 기반 추천시스템의 유사도를 2:8로 결합한 추천시스템의 성능이 가장 높았다.

**최종 추천시스템 추천 결과 예시**


---
## Conclusion
개인별 맞춤 추천시스템을 제공하는 플랫폼이 많아지면서 이러한 음악 추천 시스템은 곡을 들을 때에 가사를 중점으로 보는 사람과 멜로디를 중점으로 보는 사람 모두에게 맞춤 추천시스템을 제공할 수 있다. 여기서 더 나아가 장르별 혹은 계절별 맞춤 곡들을 미리 군집화한 후 군집별 추천시스템을 적용한다면 상황에 맞는 곡들을 추천해줄 수 있을 것이다.
