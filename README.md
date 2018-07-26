# KBO.data

안녕하세요. 외대 철학과 스터디 그룹 LOPES의 추선식 입니다.

데이터 분석을 공부하면서 느낀 점은 학습에 맞게 구성된 데이터가 적고 이런 형태의 자료를 현실적으로 찾아보기 어렵다는 점입니다. 물론 [캐글(Kaggle)](https://www.kaggle.com)이나 서울시 통계자료같이 오픈된 곳들도 있습니다. 하지만 특정 자료들을 다룰 분석의 목적성이 부족했습니다. 나아가 자료를 실제로 어떤 방식으로 분석해야 하는지 접근하기가 쉽지 않았습니다. 그래서 대중적으로도 인기가 있는 한국의 프로야구 자료를 통해 공부해 보기로 했습니다.   

우선 자료를 모으기 위해 [KBO 홈페이지](http://www.koreabaseball.com)에 등록된 야구 자료를 모으고 보기 좋게 정리하기 시작했습니다. 네이버와 같은 포털사이트에도 야구 자료가 있더군요. 거기에 있는 야구 자료는 2010년 이전의 자료도 있기에 그쪽으로도 작업하여 데이터를 추가하려 했습니다. 그러나 네이버에 있는 자료는 결측값이 있고, 부정확하거나 부족한 경우가 많아서 공식적인 KBO 사이트만을 참조하여 자료를 만들었습니다.

## 데이터 설명 및 정리 방법

저희가 수집하고 가공한 자료는 2010년부터 2018년 올스타전까지의 KBO 프로야구 데이터가 있습니다. KBO_light 자료는 2010년부터 2018년 올스타전까지의 데이터를 합친 자료입니다. 이 자료들은 앞으로도 월마다 계속 데이트할 예정입니다. 자료는 크롤링을 통해 수집하고 모두 정리한 다음에는 직접 눈으로 잘못된 부분이나 빠진 부분이 있는지 확인했습니다. 최종적으로 확인한 후, csv 파일로 만들고 R에서 경기에 관한 정보들을 홈팀과 원정팀, 점수, 정규시즌, 시범경기의 구분 등으로 열을 나누어서 만들었습니다. 이러한 과정을 거쳐 자료를 정리했습니다.

## 간략한 데이터 사용방법

깃허브에서 데이터를 다운받거나 사용하시기 어려우신 분들은 "https://lopes.hufs.ac.kr/KBO/" 기본 주소에 열고 싶은 파일 이름을 붙여서 R에서 열어 사용하실 수 있습니다. 예를 들어 2010년부터 2018년까지의 전체자료를 읽어온다고 하면 "KBO_light.csv"를 위 주소에 붙여서 아래와 같이 읽고 사용하실 수 있습니다.

```
KBO.light.URL<-"https://lopes.hufs.ac.kr/KBO/KBO_light.csv"
KBO.light<-read.table(file = KBO.light.URL, header= TRUE, sep=",",stringsAsFactors = FALSE)
doosan_home<-KBO.light[KBO.light$비고=="정규시즌"&KBO.light$홈팀=="두산",]
doosan_away<-KBO.light[KBO.light$비고=="정규시즌"&KBO.light$원정팀=="두산",]
mean(doosan_home$원정팀점수) #10년간 두산이 홈일 때 평균 실점
mean(doosan_away$홈팀점수) # 10년간 두산이 원정팀일 때 평균 실점
```


## 사용 분야

특정팀의 연도에 따른 총득점과 총실점의 변화, 연도별 팀 간의 상대전적 혹은 연속적인 포스트시즌 진출 여부 등을 확인할 수 있습니다. 또한, 점수와 홈 & 원정팀, 시즌 정보 등을 구분하여 경기 정보를 쉽게 다룰 수 있으며 다양한 기록들을 시각화해볼 수 있습니다.

## 응용사례 코드 설명

아래의 코드는 2010년부터 2018년 전반기까지 전체가 저장된 KBO_light파일을 읽고 특정팀의 연도별 총득점과 총실점을 구하고 그것을 그려보는 코드입니다. `Runs.and.score` 함수는 데이터와 원하는 팀이름을 인자로 받아 해당 팀의 연도별 총득점과 총실점을 계산합니다. `aggregate` 함수를 통해 홈팀과 원정팀에서의 득점과 실점을 모으고, 각각의 결과를 서로 더해 연도별 총득점과 총실점을 구합니다. 이렇게 구한 연도별 총 득,실점은 팀이름과 경기 연도가 함께 반환됩니다.

이후의 코드는 전체팀을 대상으로 연도별 총실점을 구해 `ggplot2`패키지를 이용해 시각화하는 코드입니다.

## 응용사례

```r
install.package("ggplot2")
library(ggplot2)
KBO.light.URL<-"https://lopes.hufs.ac.kr/KBO/KBO_light.csv"
KBO.light<-read.table(file = KBO.light.URL, header= TRUE, sep=",",stringsAsFactors = FALSE)

#연도별 총득,실점구하는 함수

Runs.and.score<-function(x,y){#x=data,y=teamname
  x$경기연도<-substr(x$Date,1,4)
  team.data.home<-x[x$홈팀==y,]
  team.data.away<-x[x$원정팀==y,]
  Runs.score.home.yearly <- aggregate(홈팀점수~경기연도,team.data.home,sum)
  Runs.score.away.yearly <- aggregate(원정팀점수~경기연도,team.data.away,sum)
  Runs.home <- aggregate(원정팀점수~경기연도,team.data.home,sum)
  Runs.away <- aggregate(홈팀점수~경기연도,team.data.away,sum)
  Total.Runs.score<-Runs.score.home.yearly$홈팀점수+Runs.score.away.yearly$원정팀점수
  Total.Runs<-Runs.home$원정팀점수+Runs.away$홈팀점수
  runs.change<-data.frame(Total.Runs.score=Total.Runs.score,Total.Runs=Total.Runs,year=Runs.score.home.yearly$경기연도,teamname=y)
  return(runs.change)
}

allteam<-unique(KBO.light$원정팀)[c(1:8,11,12)]
allteam.runs<-do.call(rbind,lapply(1:10,FUN =function(i){Runs.and.score(KBO.light,allteam[i])}))

#한글을 보이게 하기 위한 코드
theme_set(theme_gray(base_family='NanumGothic'))

g<-ggplot(data = allteam.runs,aes(x=year,y=Total.Runs,fill= teamname,color= teamname))
g+geom_point(stat="identity")+theme(axis.text.x= element_text(angle=90, hjust=1))+facet_wrap(~teamname)


```

## 참고

혹시 자료를 공개하는데 문제가 있을 경우에는 연락을 주시면 즉시 삭제하도록 하겠습니다. 또한 자료에 틀린 곳이 있으면 issues로 올려주시면, 확인한 다음 고치도록 하겠습니다.
