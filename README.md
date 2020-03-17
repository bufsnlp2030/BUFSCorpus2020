# 다중 언어분석기 기반 대용량 형태, 구문 말뭉치 반자동 구축
```
    - 다중 언어분석기 기반 반자동 형태, 구문 대용량 말뭉치 구축
    - 수작업  형태, 구문 말뭉치 평가
    - GitHub를 통한 형태, 구문 말뭉치 공개
```
- 본 연구는 대규모 말뭉치를 반자동으로 구축하는 플랫폼을 개발하는 방법을 제안한다.   

- 말뭉치는 자연언어처리에 쓰이는 여러 기계학습 알고리즘을 위한 학습데이터로써 알고리즘의 성능을 높이기 위해서는 많은 양이 필요하며, Universal Dependencies에서 제시하는 가이드라인을 따라 한국어 말뭉치를 구축하면 언어 간 구조 변환 연구와 다양한 수준의 언어처리 알고리즘에서 일관된 방법을 적용할 수 있어서 효율적이다.   

- 다중 언어분석기를 기반으로 하여 한국어 원시 말뭉치를 반자동으로 대용량 형태/구문 대용량 말뭉치를 구축하였으며, 구축과정에서 형태소 분석기의 특징으로 인해 발생하는 서로 다른 분석결과를 예제를 기반으로 정규화하였다.   

- 수작업으로 구문 말뭉치를 평가한 결과 기존의 규칙기반 정규화 방식보다 토큰 단위와 문장 단위의 일치율을 향상시켰으며, 원시말뭉치중 하나인 KCC150의 성능 평가는 아래와 같이 형태소분석은 토큰 기준으로 99.12%, 구문분석은 토큰 기준으로 99.58%의 정확도를 가진다.

|||문장|어절|
|:------:|:------:|:------:|:------:|
|구축 규모|형태|14,617,767|196,683,135|
||구문|3,889,038|38,521,608|
|성능 평가|형태|92.00%|99.12%|
||구문|97.00%|99.58%|

---------------------------------------

# 목차

<details>
<summary>펼치기/접기</summary>
<div markdown="1">

### [1. 원시말뭉치](#원시말뭉치)
#### [1.1. 원시말뭉치 전처리](#원시말뭉치-전처리)
    
### [2. 언어분석기](#언어분석기)
#### [2.1. 자동 형태/구문 분석](#자동-형태소-및-구문-분석)
#### [2.2. 반자동 형태소 분석/구문 분석 말뭉치 구축](#반자동-형태소-및-구문-분석)

### [3. 오류 및 불일치 수정](#오류-및-불일치-수정)
#### [3.1. 특정 토큰 일관성 오류 수정](#특정-토큰-일관성-오류-수정)
#### [3.2. 연결어미(EC), 종결어미(EF) 일관성 오류 수정](#연결어미-및-종결어미-일관성-오류-수정)
#### [3.3. Fixed expression에서 의존관계 일관성 수정](#fixed-expression에서-의존관계-일관성-수정)
#### [3.4. "본용언(의존소) <- 보조용언(지배소)" 규칙 적용](#본용언-및-보조용언-규칙-적용)

### [4. 형태소 분석](#형태소-분석)
#### [4.1. 규칙기반 형태소 분석 결과 정규화](#규칙기반-형태소-분석-결과-정규화)
#### [4.2. 예제기반 형태소 분석 결과 정규화](#예제기반-형태소-분석-결과-정규화)
#### [4.3. 수작업 평가 및 예제](#수작업-평가-및-예제)

### [5. 구문 분석](#구문-분석)
#### [5.1. 수작업 평가 및 예제](#수작업-평가-및-예제)

</div>
</details>

-------------------------------------------

## 원시말뭉치

### KCC(Korean Contemporary Corpus) : 원시 말뭉치
- Written raw sentences of the Korean language
- <http://nlp.kookmin.ac.kr/kcc/>

#### 1) KCC150_Korean_sentence_EUCKR.txt
```
150,705,457 words (11,961,347 sentences)
Sentences with quotes are not included.
```

#### 2) KCCq28_Korean_sentence_EUCKR.txt
```
All the sentences include a quote.
28,782,776 words (1,337,721 sentences with double qoutes)
```

#### 3) KCC940_Korean_sentence_EUCKR.txt
```
All the sentences are no more than 30 words.
93,210,332 words (6,263,454 sentences)
```


## 원시말뭉치 전처리
#### 1) 문장부호(. , ? !)를 별도의 토큰으로 분리함
![그림2](https://user-images.githubusercontent.com/61721751/75887286-e930f480-5e6c-11ea-98f9-df0b0c72411d.png)

#### 2) 인용문에서 내포문장을 추출하여 단문으로 변경 (KCCq28, KCC940 대상)
||문장 수|토큰 수|
|:------:|:------:|:------:|
|KCC150|11,961,347|166,904,618|
|KCCq28|1,355,280|18,113,979|
|KCC940|5,607,899|81,123,651|
|계|__18,924,526__|__266,142,248__|

#### 3) 문장 길이 별 전처리 말뭉치 분포
<img width="550" alt="그림3" src="https://user-images.githubusercontent.com/61721751/75896241-20f26900-5e7a-11ea-97e9-2cc68b1b4bea.PNG">


-------------------------------------------

## 언어분석기

### 자동 형태소 및 구문 분석
* 국민대, ETRI, 울산대 언어분석기를 이용하여 분석함
* 국민대 언어분석 결과는 ETRI, 울산대 언어분석 결과와 형태소 분할 기준 및 태그가 상이하여, 실험에서 ETRI, 울산대 언어분석 결과만 비교함   
<img width="550" alt="그림4" src="https://user-images.githubusercontent.com/61721751/75897523-091be480-5e7c-11ea-8911-f02e5149d199.PNG">

### 반자동 형태소 및 구문 분석
* 복수의 자동 형태소/구문 결과를 비교하여, 일치하는 분석결과를 정답이라고 가정함   
![그림5](https://user-images.githubusercontent.com/61721751/75897869-965f3900-5e7c-11ea-92c0-898a2694cf53.png)

-------------------------------------------

## 오류 및 불일치 수정

### 특정 토큰 일관성 오류 수정
```
및/MAG -> MAJ
혹은/MAG -> MAJ
또는/MAG -> MAJ
즉/MAG -> MAJ
```

### 연결어미 및 종결어미 일관성 오류 수정
* 문장 중간(1~마지막-2) token이 종결어미(EF) 인 경우 연결어미(EC) 로 변환   
단, 다음 token이 symbol인 경우는 제외함

* 문장 마지막-1 token이 연결어미(EC)인 경우 종결어미(EF)로 변환   
단, 다음 token이 SF인 경우만 적용

* 마지막 token이 연결어미(EC)로 끝나는 경우 종결어미(EF)로 변환

### fixed expression에서 의존관계 일관성 수정
* 뒤 토큰은 앞 토큰만 의존소로 가짐
* 뒤 토큰이 기타 토큰을 의존소로 가지면, 기타 토큰의 지배소를 수정함
![그림6](https://user-images.githubusercontent.com/61721751/75899479-efc86780-5e7e-11ea-940c-15b7fd92c152.png)
```
을/를 <- 통하+어 | 통하+어서
을/를 <- 위하+어 | 위하+어서 | 위하+ㄴ
만 <- 위하+어 | 위하+어서 | 위하+ㄴ
기 <- 위하+어 | 위하+어서 | 위하+ㄴ
에 <- 대하+어 | 대하+어서 | 대하+ㄴ
에 <- 관하+어 | 관하+어서 | 관하+ㄴ
로/으로 <- 인하+어 |  인하+ㄴ | 인하+어서
을/를 <- 필두+로
에 <- 따르+아 | 따르+면
와/과 <- 마찬가지+로
로/으로 <- 말미암+아
…
```
* “ㄹ” <- “수” <- “있/없” 
* “ㄴ” <- “것“ <- “같＂
![그림1](https://user-images.githubusercontent.com/61721751/75899623-1ab2bb80-5e7f-11ea-8742-e48b73f96edb.png)

### 본용언 및 보조용언 규칙 적용
* “본용언(의존소) <-- 보조용언(지배소)” 규칙 적용
* 종결기호 (., !, ?)가 문장 마지막에 나타나는 경우 바로 앞 token을 지배소로 지정
* 콤마(,)는 바로 앞 token을 지배소로 지정
![그림7](https://user-images.githubusercontent.com/61721751/75899945-89901480-5e7f-11ea-9b77-16265b014a32.png)

-------------------------------------------

## 형태소 분석
#### 1) 형태소 정규화 전후 일치 토큰 및 문장수
정규화 후 전체 토큰 수 188,208,139(70.71%) / 전체 문장 수 __14,083,748__(74.42%)

* KCC150[파일 다운로드](https://github.com/bufsnlp2030/BUFS-JBNUCorpus2020/blob/master/KCC150_equal_morph.egg "클릭하면 파일이 다운로드 됩니다")   
  ||토큰 수 ||문장 수||
  |:------:|:------:|:------:|:------:|:------:|
  |전체|166,904,509|-|11,961,347|-|
  |형태소 일치(정규화 전)|23,227,819|13.92%|2,242,867|18.75%|
  |형태소 일치(정규화 후)|117,849,839|__70.61%__|8,875,652|__74.20%__|

* KCC940   
  ||토큰 수 ||문장 수||
  |:------:|:------:|:------:|:------:|:------:|
  |전체|81,123,597|-|5,607,899|-|
  |형태소 일치(정규화 전)|10,195,189|12.57%|928,976|16.57%|
  |형태소 일치(정규화 후)|56,938,877|__70.19%__|4,157,230|__74.13%__|
  
* KCCq28   
  ||토큰 수 ||문장 수||
  |:------:|:------:|:------:|:------:|:------:|
  |전체|18,113,978|-|1,355,280|-|
  |형태소 일치(정규화 전)|2,875,703|15.88%|277,697|20.49%|
  |형태소 일치(정규화 후)|13,419,423|__74.08%__|1,050,866|__77.54%__|
 
#### 2) 형태소 정규화 후 일치 문장 길이 분포
- 원시 말뭉치의 문장 길이 분포와, 형태소 분석결과 일치 말뭉치의 문장 길이 분포가 유사함
- 형태소 분석결과 불일치 현상은 문장 길이와 관련이 적음
![그림8](https://user-images.githubusercontent.com/61721751/75903165-84819400-5e84-11ea-8065-de24641dd757.png)
 
### 규칙기반 형태소 분석 결과 정규화
* 규칙기반 정규화 일부 (토큰 : 어절 단위)
  |정규화 전|정규화 후|
  |:------:|:------:|
  |NNG+NNG|NNG|
  |NNP+NNP|NNP|
  |XSN+XSN|XSN|
  |NNG+XSN|NNG|
  |NNG+XSV|VV|
  
* 예시
```
NNG+XSN -> NNG 	// 위원 + 장 -> 위원장/NNG
NNG+NNG -> NNG 	// 입당 + 원서 -> 입당원서/NNG
NNG+XSV -> VV 	// 공부 + 하 -> 공부하/VV
NNG+VV -> VV 	// 존중 + 받 -> 공부하/VV
NNG+XSA -> VA 	// 적정 + 하 -> 적정하/VA
NNG+VA -> VA 	// 독기 + 어리 -> 독기어리/VA
MAG+XSA -> VA 	// 짜릿 + 하 -> 짜릿하/VA
XR+XSA -> VA 	// 치밀 + 하 -> 치밀하/VA , 심각 + 하 -> 심각하/VA
NNP+NNP -> NNP 	// 경기 + 포천 -> 경기포천/NNP
NNP+NNG -> NNP 	// 양주 + 시청 -> 양주시청/NNP
NNG+NNP -> NNP	// 더본 + 코리아 -> 더본코리아/NNP
XPN+NNP -> NNP 	// 반 + 스탈린주의 -> 반스탈린주의/NNP
NNP+SN -> NNP 	// 갤럭시노트 + 7 -> 갤럭시노트7/NNP
XPN+NNG -> NNG 	// 초+강수
SN+NR -> NR 	// 500+만
JX+JX -> JX 	// 마저+도
```

### 예제기반 형태소 분석 결과 정규화
* 예제기반 정규화 일부 (토큰 : 어절 단위 + 문장 부호)
  |ETRI 형태소|울산대 형태소|규칙(번호)|정규화 후|
  |:-------:|:-------:|:-------:|:-------:|
  |VV+EP+EC|VV+EP+EF|–EC(ETRI), -EF(울산대)로 끝나고 다음 토큰이 ./SF인 경우 울산대 분석결과로 정규화(0)|VV+EP+EF|
  |NNG+JKB|NNG+NNB+JKB|ETRI의 분석결과가 간결한 경우 ETRI 분석결과로 정규화(1)|NNG+JKB|
  |MAG+XSV+EF|VV+EF|울산대의 분석결과가 간결한 경우 울산대 분석결과로 정규화(2)|VV+EF|
  |VX+EP+EF|VV+EP+EF|두 개의 언어 분석기가 언어학적으로 서로 다른 분석결과를 제시한 경우 정규화 안함(3)|정규화 안함|
  
* 예시
```diff
0번 규칙 예) 둘이다 .
[E] 둘/NR+이/VCP+다/EC ./SF
+ [U] 둘/NR+이/VCP+다/EF ./SF
```
```diff
1번 규칙 예) 예민해짐에
+ [E] 예민하/VA+어/EC+지/VX+ㅁ/ETN+에/JKB
[U] 예민/NNG+하/XSA+여/EC+지/VX+ㅁ/ETN+에/JKB
```
```diff
2번 규칙 예) 말이라는 게 그래서 허무한 것인지도 모른다 .
[E] 말이라는/NNG+VCP+ETM 게/NNB+JKS '그래서/MAG+XSV+EC' 허무한/NNG+XSA+ETM
것인지도/NNB+VCP+EC+JX '모른다/VV+EC ./SF'
+ [U] 말이라는/NNG+VCP+ETM 게/NNB+JKS '그래서/VV+EC' 허무한/NNG+XSA+ETM
+ 것인지도/NNB+VCP+EC+JX '모른다/VV+EF ./SF'
```
```
3번 규칙 예) 아물지를
[E] 아물지/NNP+를/JX
[U] 아물/VV+지/EC+를/JX
```

### 수작업 평가 및 예제
#### 1) 임의로 100문장 선택 후 수작업 평가

* KCC150
  |토큰 길이|평가 문장 수|평가 토큰 수|정확한 문장 수|정확한 토큰 수|문장 단위 정확률|토큰 단위 정확률|
  |:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|
  |1~5|4|20|3|19|75.00%|95.00%|
  |6~10|37|299|36|298|97.30%|99.67%|
  |11~15|34|430|32|427|94.12%|99.30%|
  |16~20|15|250|13|246|86.67%|98.40%|
  |21~25|5|112|4|111|80.00%|99.11%|
  |26~30|4|109|3|108|75.00%|99.08%|
  |31~|1|31|1|31|100.00%|100.00%|
  |계|__100__|__1,251__|__92__|__1,240__|__92.00%__|__99.12%__|
  
* KCC940
  |토큰 길이|평가 문장 수|평가 토큰 수|정확한 문장 수|정확한 토큰 수|문장 단위 정확률|토큰 단위 정확률|
  |:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|
  |1~5|2|10|2|10|100.00%|100.00%|
  |6~10|31|263|27|259|87.10%|98.48%|
  |11~15|34|444|25|434|73.53%|97.75%|
  |16~20|17|300|14|297|82.35%|99.00%|
  |21~25|13|293|9|288|69.23%|98.29%|
  |26~30|2|52|2|52|100.00%|100.00%|
  |31~|1|36|0|31|0.00%|86.11%|
  |계|__100__|__1,398__|__79__|__1,371__|__79.00%__|__98.07%__|

* KCCq28
  |토큰 길이|평가 문장 수 |평가 토큰 수|정확한 문장 수|정확한 토큰 수|문장 단위 정확률|토큰 단위 정확률|
  |:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|
  |1~5|3|15|3|15|100.00%|100.00%|
  |6~10|29|227|23|221|79.31%|97.36%|
  |11~15|34|418|31|415|91.18%|99.28%|
  |16~20|26|462|23|459|88.46%|99.35%|
  |21~25|6|136|4|134|66.67%|98.53%|
  |26~30|2|54|1|53|50.00%|98.15%|
  |31~|0|0|0|0|0.00%|0.00%|
  |계|__100__|__1,312__|__85__|__1,297__|__85.00%__|__98.86%__|
 
#### 2) 형태소 일치 문장 : 오류 예제

|ID|FORM|LEMMA|UPOS|XPOS|오류 수정|
|:---:|:---:|:---:|:---:|:---:|:---:|
|__1__|__목소리는__|__목소리+는__|__NOUN__|__NNP+JX__|__NNG+JX__|
|2|늘|늘|ADV|MAG||
|3|온화하게|온화하+게|ADJ|VA+EC||
|4|유지했고요|유지하+었+고+요|VERB|VV+EP+EF+JX||
|5|.|.|PUNC|SF||   

```diff
- 목소리는 ‘목구멍에서 나는 소리’라는 일반명사이므로 1번 XPOS의 NNP가 NNG로 바뀌어야 한다.
````

|ID|FORM|LEMMA|UPOS|XPOS|오류 수정|
|:---:|:---:|:---:|:---:|:---:|:---:|
|1|강동희|강동희|PROPN|NNP||
|2|감독은|감독+은|NOUN|NNG+JX||
|3|가드인|가드+이+ㄴ|VERB|NNB+VCP+ETM||
|4|리처드|리처드|PROPN|NNP||
|__5__|__로비를__|__로비+를__|__NOUN__|__NNG+JKO__|__NNP+JKO__|
|6|뽑는|뽑+는|VERB|VV+ETM||
|7|강수까지|강수+까지|NOUN|NNG+JX||
|8|뒀다|두+었+다|VERB|VV+EP+EF||
|9|.|.|PUNC|SF||   

```diff
- 사람의 이름(人名)은 고유명사이므로 2, 6번 XPOS의 NNG가 NNP로 바뀌어야 한다.
````

-------------------------------------------

## 구문 분석
#### 1) 구문분석 일관성 수정 후 일치 토큰 및 문장수
정규화 후 전체 토큰 수 33,197,177(12.47%) / 전체 문장 수 __3,376,968__(17.84%)

* KCC150   
  ||토큰 수 ||문장 수||
  |:------:|:------:|:------:|:------:|:------:|
  |전체|166,904,509|-|11,961,347|-|
  |구문 분석 일치|20,842,946|__12.49%__|2,166,162|__18.11%__|

* KCC940   
  ||토큰 수 ||문장 수||
  |:------:|:------:|:------:|:------:|:------:|
  |전체|81,123,597|-|5,607,899|-|
  |구문 분석 일치|9,275,574|__11.43%__|901,492|__16.08%__|
  
* KCCq28   
  ||토큰 수 ||문장 수||
  |:------:|:------:|:------:|:------:|:------:|
  |전체|18,113,978|-|1,355,280|-|
  |구문 분석 일치|3,078,657|__17.00%__|309,314|__22.82%__|
  
#### 2) 구문 일치 문장 길이 분포
문장의 길이가 짧을 수록 문장 단위 의존 관계 일치 확률이 높음
![그림9](https://user-images.githubusercontent.com/61721751/75948194-22f21180-5ee6-11ea-8cb3-cd65e88c9d83.png)
  
### 수작업 평가 및 예제
#### 1) 임의로 100문장 선택 후 수작업 평가

* 구문 일치 문장에 대한 수작업 평가(UAS, Unlabeled Attachment Score)
  |토큰 길이|평가 문장 수 |평가 의존관계 수|정확한 문장 수|정확한 의존관계 수|문장 단위 정확률|의존관계 단위 정확률|
  |:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|
  |1~5|13|52|12|51|92.31%|98.08%|
  |6~10|69|465|68|464|98.55%|99.78%|
  |11~15|18|204|17|203|94.44%|99.51%|
  |16~20|0|0|0|0|0.00%|0.00%|
  |21~25|0|0|0|0|0.00%|0.00%|
  |26~30|0|0|0|0|0.00%|0.00%|
  |31~|0|0|0|0|0.00%|0.00%|
  |계|__100__|__721__|__97__|__718__|__97.00%__|__99.58%__|
  
* 구문 일치 문장에 대한 수작업 평가(LAS, Labeled Attachment Score)
  |토큰 길이|평가 문장 수 |평가 의존관계 수|정확한 문장 수|정확한 의존관계 수|문장 단위 정확률|의존관계 단위 정확률|
  |:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|
  |1~5|13|52|11|50|84.62%|96.15%|
  |6~10|69|465|66|462|95.65%|99.35%|
  |11~15|18|204|17|203|94.44%|99.51%|
  |16~20|0|0|0|0|0.00%|0.00%|
  |21~25|0|0|0|0|0.00%|0.00%|
  |26~30|0|0|0|0|0.00%|0.00%|
  |31~|0|0|0|0|0.00%|0.00%|
  |계|__100__|__721__|__94__|__715__|__94.00%__|__99.17%__|

#### 2) 구문 일치 문장 : 오류 예제

|ID|FORM|LEMMA|UPOS|XPOS|FEATS|HEAD|DEPREL|DEPS|MISC|오류 수정|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|1|서로|서로|ADV|MAG|-|3|AP|-|-||
|2|간격이|간격+이|NOUN|NNG+JKS|-|3|NP_SBJ|-|-||
|3|맞게|맞+게|VERB|VV+EC|-|5|VP|-|-||
|4|따로|따로|ADV|MAG|-|5|VP|-|-||
|__5__|__뚫으면__|__뚫+으면__|__VERB__|__VV+EC__|__-__|__6__|__VP__|__-__|__-__|VP_CMP|
|6|됩니다|되+ㅂ니다|VERB|VV+EF|-|0|ROOT|-|-||
|7|.|.|PUNC|SF|-|6|SF(punct)|-|-||

![그림10](https://user-images.githubusercontent.com/61721751/75955717-c39dfc80-5ef9-11ea-9f95-7ff708b1be6f.png)

```diff
- ID 5의 VP(DEPREL)를 그림과 같이 VP_CMP로 수정해야 한다.
```

|ID|FORM|LEMMA|UPOS|XPOS|FEATS|HEAD|DEPREL|DEPS|MISC|오류 수정|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|1|그의|그+의|PRON|NP+JKG|-|2|NP_MOD|-|-||
|2|영화|영화|NOUN|NNG|-|3|NP|-|-||
|__3__|__속__|__속__|__NOUN__|__NNG__|__-__|__4__|__NP__|__-__|__-__|6|
|4|미니멀리즘적|미니멀리즘적|NOUN|NNG|-|5|NP|-|-||
|5|표현|표현|NOUN|NNG|-|6|NP|-|-||
|6|방식처럼|방식+처럼|NOUN|NNG+JKB|-|7|NP_AJT|-|-||
|7|절제되고|절제되+고|VERB|VV+EC|-|10|VP|-|-||
|8|함축적인|함축적+이+ㄴ|VERB|NNG+VCP+ETM|-|9|VNP_MOD|-|-||
|9|미학이|미학+이|NOUN|NNG+JKS|-|10|NP_SBJ|-|-||
|10|돋보이는|돋보이+는|VERB|VV+ETM|-|11|VP_MOD|-|-||
|11|작품들이다|작품들+이+다|VERB|NNG+VCP+EF|-|0|ROOT|-|-||
|12|.|.|PUNC|SF|-|11|SF(punct)|-|-||

![그림11](https://user-images.githubusercontent.com/61721751/75958399-7d4b9c00-5eff-11ea-8898-57320236cefa.png)

```diff
- ID 3의 4(HEAD)를 그림과 같이 6으로 수정해야 한다.
```
