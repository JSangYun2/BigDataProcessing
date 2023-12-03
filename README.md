# 대한민국 국민 지역별 GDP 비교 

## 1️⃣  목적 

 - ☑️ 각 지역별 어떤 종목의 산업이 두드러지게 발달하였는가? 
 - ☑️ 스타트업 창업 시 그 종목에 따라 어떤 지역이 유리한지?
 - ☑️ 대한민국 시도별 GDP 생산량의 차이는 어떻게 되는가?
 - ☑️ 각 지역의 산업의 발달 이유는 무엇인가?

## 2️⃣ 진행 기준

 - 데이터 수집 : 국가교통DB -> 국내 총생산 및 국민 총 소득 (2021 기준)
 - 사용 라이브러리 :  pandas / matplotlib.pyplopt / numpy / drive / seaborn / requests / json / re / folium
 - 진행 중 자료 내에 결측치 존재 x
 - 자료 내 행 : 시도별 / 경제활동별 / 명목 / 실질 / 실질기여도 / -> 측정치는 <b>'실질'</b> 통계 사용

## 3️⃣ 진행 사항

<b>※ 한글 폰트로 진행하였습니다!! </b>

### 3-1) 
본인 구글 드라이브에 파일을 저장하여 불러오는 형식을 사용하였습니다. df로 파일명을 지정후
'시도별','경제활동별'로 멀티인덱스 지정 후 제가 실질적으로 사용할 파일인 multi_index가 생성되었습니다.


```
 
import pandas as pd
import matplotlib.pyplot as plt
from google.colab import drive
import seaborn as sns

drive.mount("/gdrive")
df = pd.read_csv('/gdrive/My Drive/시도별_경제활동별_지역내총생산_20231110004110.csv', encoding = 'CP949')

result = df.groupby('시도별')

df_multi = df.groupby(['시도별','경제활동별'])

multi_index = df_multi.first()
```


### 3-2 ) 
전국 시도별 GDP 비교를 위한 df_WGDP 데이터프레임 생성 후 자료 내 '지역내총생산(시장가격)' 으로 사용할 column 값을 지정합니다.
후에 파이 차트로 17개 시도별 GDP 금액 차이를 데이터 시각화 하였습니다.

```

df_WGDP = multi_index.xs('지역내총생산(시장가격)', level = '경제활동별')
df_WGDP.drop(labels ='전국', axis = 0, inplace = True)

print(df_WGDP)

wed={"width": 0.8}
colors = sns.color_palette('Paired',len(df_WGDP.index))
plt.title('시도별 지역내총생산(시장가격)')
plt.pie(df_WGDP['실질'], labels = df_WGDP.index, autopct='%.1f%%', wedgeprops=wed , radius = 2.7, colors = colors)
plt.legend(loc = 'center')
plt.show()

```

![지역내 총생산 비교 (전국)](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/31c35d6a-7a59-403d-9114-cec3b3dcca1e)

### 3-3)
3-2와 내용은 동일하나, 단계구분도로 표현 하였습니다.

```
import requests
import json
import re
import folium

# 서울 행정구역 json raw파일(githubcontent)https://raw.githubusercontent.com/southkorea/seoul-maps/master/kostat/2013/json/seoul_municipalities_geo_simple.json
s_geo='/gdrive/My Drive/SIDO_MAP_2022.json'

s_map = folium.Map(location=[37.559984,126.9753071],  # 숭례문 위도, 경도
                   zoom_start=7)

df_WGDP_2 = df_WGDP.reset_index()

folium.Choropleth(geo_data = s_geo,
                  data = df_WGDP_2,
                  columns = ['시도별','실질'],
                  fill_color = 'OrRd', fill_opacity = 0.7, line_opacity=0.3,
                  threshold_scale=[10000000, 30000000, 50000000, 80000000, 100000000, 150000000, 200000000, 250000000, 300000000, 350000000, 400000000,450000000,500000000],
                  key_on='feature.properties.CTP_KOR_NM',).add_to(s_map)

s_map

```
![image-removebg-preview__4_-removebg-preview](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/d5c52c49-9ee8-4735-ac30-d3ac01f08a90)

### 3-4) 
17개 시도 중 서울특별시의 산업 발달표를 시각화 하였습니다. 마찬가지로 파이차트로 구현 하였고, 
범주는 5가지로 분류 하였습니다.

```
df_SEOUL = multi_index.loc['서울특별시']
print(df_SEOUL)

Ind_S = ['농업, 임업 및 어업','광제조업', '전기, 가스, 증기 및 공기 조절 공급업', '건설업', '서비스업']

df_SEOUL_D = df_SEOUL.loc[Ind_S]

wed={"width": 0.8}
plt.title('서울특별시 산업 발달 실질 차트')
plt.pie(df_SEOUL_D['실질'], labels = df_SEOUL_D.index, autopct='%.1f%%', wedgeprops=wed , radius = 2.5,  colors = colors)
plt.legend(loc = 'center')
plt.show()
```
![서울](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/d70470ce-8c81-41b5-9814-ff13afc545be)

<b> ※ 서울은 GDP 비율의 90% 이상이 서비스업입니다. 추가로 이러한 그래프가 이러한 형태를 띄는 이유를 조사해 보았습니다. </b>

### 1. 경제적 중심지:
서울은 대한민국의 경제적 중심지로서, 국내 외 대부분의 기업 본사가 위치하고 있습니다. 이는 서비스업이 발달하는데 중요한 역할을 합니다. 금융, 컨설팅, 법률, 교육, 의료, IT 등 다양한 서비스 산업이 다른 지역보다 집중되어 있습니다.

### 2. 국제적으로 경쟁력 있는 도시:
서울은 국제적으로 경쟁력 있는 도시로서 글로벌 기업들의 사무소와 지사가 밀집되어 있습니다. 이로 인해 국제적인 비즈니스, 금융, 무역, 컨설팅 등의 서비스 업무가 활발히 이루어지고 있습니다.

### 3. 문화, 쇼핑, 엔터테인먼트 중심:
서울은 문화와 쇼핑, 엔터테인먼트 산업이 잘 발달한 도시입니다. 다양한 문화 이벤트, 영화, 연극, 음악, 패션 등이 집중되어 있어 관련 서비스 산업이 번성합니다.

### 4. 교육 및 의료 서비스:
서울은 국내 최고 수준의 대학, 연구소, 의료 시설이 위치해 있어 교육 및 의료 서비스 산업도 발달하고 있습니다. 다양한 교육 및 의료 서비스가 제공되면서 해당 분야의 서비스 업무도 활발합니다.

### 5. 산업 및 기술의 집중:
서울은 산업 분야와 기술 분야에서도 많은 기업이 집중되어 있습니다. 이에 따라 기술적인 지식과 노하우를 바탕으로 하는 서비스 산업이 발전하고 있습니다.

# 3-5) 
서울특별시의 서비스업에 대해 조금 더 세부적으로 분석해보았습니다. 

```
Service_Ind = ['　도매 및 소매업','　운수 및 창고업','　숙박 및 음식점업','　정보통신업','　금융 및 보험업','　부동산업','　사업서비스업','　공공 행정, 국방 및 사회보장 행정','　교육 서비스업','　보건업 및 사회복지 서비스업','　문화 및 기타서비스업']
df_SEOUL_S = df_SEOUL.loc[Service_Ind]
#print(df_SEOUL_S)
wed={"width": 0.8}
plt.title('서울특별시 서비스업 세분화 차트')
plt.pie(df_SEOUL_S['실질'], labels = df_SEOUL_S.index, autopct='%.1f%%', wedgeprops=wed , radius = 3, colors = colors)
plt.legend(loc = 'center')
plt.show()
```

### 3-6) 
17개 시도 중 울산광역시의 산업 발달표를 시각화 하였습니다. 마찬가지로 파이차트로 구현 하였고, 범주는 5가지로 분류 하였습니다.

```
df_ULSAN = multi_index.loc['울산광역시']
print(df_ULSAN)

df_ULSAN_D = df_ULSAN.loc[Ind_S]

wed={"width": 0.8}
plt.title('울산광역시 산업 발달 실질 차트')
plt.pie(df_ULSAN_D['실질'], labels = df_ULSAN_D.index, autopct='%.1f%%', wedgeprops=wed , radius = 3.0, colors = colors)
plt.legend(loc = 'center')
plt.show()
```
![울산](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/6de003b5-7ea9-401b-90bb-ef9c5924dd4a)

<b> ※ 울산광역시는 GDP의 60% 이상이 광제조업에서 나타납니다. 그 이유에 대해서 추가적으로 조사해보았습니다. </b>

### 1. 자원과 지리적 위치:
울산은 석유화학 산업을 비롯한 광업과 제조업이 발달한 지역으로, 석유화학 산업의 중심지로 꼽힙니다. 이는 대한민국 내에서 석유화학 및 석유정제 산업이 발달한 지역이기 때문입니다.
울산은 한국만의 가장 큰 외항과 제3의 국제항으로서 국내외 무역의 중심지이기도 합니다. 또한, 중국과 가까운 지리적 위치로 인해 수출입이 용이하며 외국과의 경제 교류가 활발합니다.

### 2. 석유화학 산업:
울산은 석유화학 산업의 중심지로, 한국 내에서 가장 큰 석유화학 단지인 대우석유화학과 GS칼텍스 등의 대규모 석유화학 단지가 있습니다. 석유화학 산업은 광제조업에서 중요한 부분을 차지하며, 석유정제와 화학공업이 발달하면서 이 지역의 광제조업도 함께 성장하였습니다.

### 3. 산업 인프라와 정부 정책:
대한민국 정부의 산업화 정책과 투자, 인프라 구축 등이 울산 지역의 광제조업 발전에 큰 역할을 했습니다. 특히, 울산은 1970년대 이후 경제성장을 위해 대규모 산업단지와 인프라를 조성하는 등의 정책적 지원을 받았습니다.

### 4. 노동력과 기술력:
울산은 고령의 석유화학 산업을 기반으로 하고 있으며, 이를 유지하고 성장시키기 위해 높은 기술력과 첨단 기술을 필요로 합니다. 이를 위해 울산은 첨단 기술력을 보유한 노동력을 확보하고 이를 육성하는데 많은 투자를 하고 있습니다.

### 3-7) 
서울시와 마찬가지로 울산광역시는 광제조업을 좀 더 세부적으로 분석해보았습니다. 

```
Service_Ind = ['　광업','　　음식료품 및 담배제조업','　　섬유 의복 및 가죽 제품 제조업','　　목재종이인쇄 및 복제업','　　석탄 및 석유 화학제품 제조업','　　비금속광물 및 금속제품 제조업','　　전기 전자 및 정밀기기 제조업','　　기계 운송장비 및 기타 제품 제조업']
df_ULSAN_S = df_ULSAN.loc[Service_Ind]

wed={"width": 0.8}
plt.title('울산광역시 광제조업 세분화 차트')
plt.pie(df_ULSAN_S['실질'], labels = df_ULSAN_S.index, autopct='%.1f%%', wedgeprops=wed , radius = 3.0, colors = colors)
plt.legend(loc='center')
plt.show()
```
![울산(광제조업)](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/93f5c193-76ee-443a-896c-982a88214395)

### 3-8) 
산업의 범주를 위에서 언급했던 것과 같이 5개의 범주로 나누어 각 산업 종목 별 우세한 도시를 분석해보았습니다. 

```
df_1 = multi_index.xs('농업, 임업 및 어업', level = '경제활동별')
df_1.drop(labels ='전국', axis = 0, inplace = True)

df_2 = multi_index.xs('광제조업', level = '경제활동별')
df_2.drop(labels ='전국', axis = 0, inplace = True)

df_3 = multi_index.xs('전기, 가스, 증기 및 공기 조절 공급업', level = '경제활동별')
df_3.drop(labels ='전국', axis = 0, inplace = True)

df_4 = multi_index.xs('건설업', level = '경제활동별')
df_4.drop(labels ='전국', axis = 0, inplace = True)

df_5 = multi_index.xs('서비스업', level = '경제활동별')
df_5.drop(labels ='전국', axis = 0, inplace = True)
```
### -> 데이터프레임 나누기

```
import numpy as np

x = np.arange(17)

df_1.reset_index(inplace=True)
plt.figure(figsize=(24, 5))
plt.title('전국 "농업, 임업 및 어업" 산업 발달 비교 ')
plt.bar(x, df_1['실질'], color = colors)
plt.xticks(x, df_1['시도별'])
```
![농업-removebg-preview (1)](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/df453938-b506-49b8-9dc4-a05c33dd14ac)

```
plt.figure(figsize=(24, 5))
df_2.reset_index(inplace=True)
plt.title('전국 "광제조업" 산업 발달 비교 ')
plt.bar(x, df_2['실질'], color = colors)
plt.xticks(x, df_2['시도별'])
```
![광제조업-removebg-preview (1)](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/795f452e-86bf-48ed-a87f-f43afe7e7d6d)


```
plt.figure(figsize=(24, 5))
df_3.reset_index(inplace=True)
plt.title('전국 "전기, 가스 증기 및 공기 조절 공급업" 산업 발달 비교 ')
plt.bar(x, df_3['실질'], color = colors)
plt.xticks(x, df_3['시도별'])
```
![전기-removebg-preview](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/148c35b7-1c79-4de1-b6ff-53c65d8d3324)


```
plt.figure(figsize=(24, 5))
df_4.reset_index(inplace=True)
plt.title('전국 "건설업" 산업 발달 비교 ')
plt.bar(x, df_4['실질'], color = colors)
plt.xticks(x, df_4['시도별'])
```
![건설업-removebg-preview](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/faa1e1c1-0a8a-4df7-bf2f-d0a0f8e1b01c)

```
plt.figure(figsize=(24, 5))
df_5.reset_index(inplace=True)
plt.title('전국 "서비스업" 산업 발달 비교 ')
plt.bar(x, df_5['실질'], color = colors)
plt.xticks(x, df_5['시도별'])
```
![서비스업-removebg-preview (1)](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/14e48cb4-7685-4f66-8463-9fc528be2fb7)

### 3 - etc) 
추가로 근 5년간 국내총생산(GDP) 변화를 그래프로 분석해보았습니다. 
csv 파일은 동일한 출처에서 연별 GDP 파일은 받아와 진행하였습니다. (결측치 x)

```
df_Second = pd.read_csv('/gdrive/My Drive/경제활동별_GDP_및_GNI_원계열__실질__분기_및_연간__20231202235634.csv', encoding = 'CP949')

df_Second.set_index('계정항목별' , inplace = True)

print(df_Second.index)

x = df_Second.columns

plt.title('국내 5년간 GDP 변화')
plt.plot(x, df_Second.loc['국내총생산(시장가격, GDP)'],'bo--')
```
![GDP변화](https://github.com/JSangYun2/BigDataProcessing/assets/116622873/8f69cd1c-fdf0-4b3e-992b-bd788b8e2d58)


### 4) 결론

- 지역 간 종목 별 산업 발달 정도가 생각보다 편차가 심하고, 17개 시도 중 수도권 지역의 GDP 비율이 상당히 크다는 것을 알 수 있다. (50% 이상의 비율을 차지)
- 1차 산업 -> 경기도 유리 / 2차 산업 ->  경기도 유리 / 3차 산업 -> 수도권 유리
- 각 지역의 지리적 위치와 역사를 확인해보면 그 도시에서 발달 한 산업 종목이 무엇인지 대략적으로 예측 할 수있다.
