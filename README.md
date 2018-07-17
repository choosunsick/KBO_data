# KBO_data

안녕하세요. 외대 철학과 스터디 그룹 LOPES의 추선식 입니다.

데이터 분석을 공부하면서 느낀 점은 학습에 맞게 구성된 데이터가 적고 이런 형태의 자료를 현실적으로 찾아보기 어렵다는 점입니다. 물론 [캐글(Kaggle)](https://www.kaggle.com)이나 서울시 통계자료같이 오픈된 곳들도 있습니다. 하지만 특정 자료들을 다룰 분석의 목적성이 부족했습니다. 나아가 자료를 실제로 어떤 방식으로 분석해야 하는지 접근하기가 쉽지 않았습니다. 그래서 대중적으로도 인기가 있는 한국의 프로야구 자료를 통해 공부해 보기로 했습니다.   

우선 자료를 모으기 위해 [KBO 홈페이지](http://www.koreabaseball.com)에 등록된 2010년도부터 현재 2016년도까지의 야구 자료를 모으고 보기 좋게 정리하기 시작했습니다. 네이버와 같은 포털사이트에도 야구 자료가 있더군요. 거기에 있는 야구 자료는 2010년 이전의 자료도 있기에 그쪽으로도 작업하여 데이터를 추가하려 했습니다. 그러나 네이버에 있는 자료는 결측값이 있고, 부정확하거나 부족한 경우가 많아서 공식적인 KBO 사이트만을 참조하여 자료를 만들었습니다.

## 정리 방법

크롤링 능력이 부족하여서 수작업으로 진행했습니다. 자료를 수집하고 모두 정리한 다음에는 직접 눈으로 잘못된 부분이나 빠진 부분이 있는지 확인했습니다. 최종적으로 확인한 후, csv 파일로 만들고 R에서 경기에 관한 정보들을 원정팀, 홈팀, 점수 등의 구분 열로 나누어서 만들었습니다. 이러한 과정을 거쳐 최종적으로 7년 치를 한 번에 모아 단일한 자료로 만들었습니다.

## 사용 분야

저희가 수집하고 가공한 자료는 약 7년 치의 한국 프로야구 데이터가 있습니다. 따라서 특정팀의 연도에 따른 연속적인 변화나 7년 동안의 최대 득점 경기 혹은 포스트 시즌에 올라간 팀 목록 등을 확인할 수 있습니다. 또한, 점수와 홈 & 원정팀, 시즌정보 등을 구분하여 경기정보를 쉽게 확인해 볼 수 있으며 다양한 기록들을 시각화해볼 수 있습니다.

# 응용사례

```
## 경기 결과열을 만들어 줍니다.

kbo_total$경기결과  <- ifelse(kbo_total$홈팀점수>kbo_total$원정팀점수,"홈승",ifelse(kbo_total$홈팀점수<kbo_total$원정팀점수,"원정승","무"))


## 정규시즌만 뽑아 줍니다.

normal_season <- subset(kbo_total,kbo_total$비고=="정규시즌")
doosan <- subset(normal_season,normal_season$홈팀=="두산"|normal_season$원정팀=="두산")


## 홈팀과 원정팀의 경기 수를 경기결과에 따라 분류합니다.

home <- aggregate(홈팀점수~홈팀+원정팀+경기결과,doosan,length)
doosan_home <- home[home$홈팀=="두산",]

away <- aggregate(원정팀점수~홈팀+원정팀+경기결과,doosan,length)
doosan_away <- away[away$원정팀=="두산",]


## 두산이 홈일 때와 원정일 때의 승리만 따로 뽑아줍니다.

doosan_home_win <- doosan_home[doosan_home$경기결과=="홈승",]
doosan_away_win <- doosan_away[doosan_away$경기결과=="원정승",]


## 두산의 열이 중복되므로 지워줍니다.

doosan_home_win <- doosan_home_win[,-1]
doosan_away_win <- doosan_away_win[,-2]


## 하나의 자료로 만들어 줍니다.

colnames(doosan_home_win) <- c("상대팀","경기결과","승리횟수")
colnames(doosan_away_win) <- c("상대팀","경기결과","승리횟수")

doosan_win <- rbind(doosan_home_win,doosan_away_win)  

## 이제 두산의 상대전적(승리)그래프를 그려봅시다.

p <- ggplot(doosan_win,aes(x=상대팀,y=승리횟수,fill= 경기결과))+
geom_bar(stat="identity",colour = "black")+
scale_x_discrete(limits=c("한화","KIA", "SK", "LG", "넥센", "삼성", "롯데", "NC","kt" ))+
theme(axis.text.x= element_text(angle=90, hjust=1))

p+labs(title="두산상대전적그래프")

```

## 참고

혹시 자료를 공개하는데 문제가 있을 경우에는 연락을 주시면 즉시 삭제하도록 하겠습니다. 또한 자료에 틀린 곳이 있으면 issues로 올려주시면, 확인한 다음 고치도록 하겠습니다.
