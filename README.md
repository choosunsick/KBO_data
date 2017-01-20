# KBO_data

안녕하세요. 외대 철학과 스터디 그룹 LOPES의 추선식 입니다.


데이터 분석을 공부하면서 느낀 점은 학습에 맞게 구성된 데이터가 적고 이런 형태의 자료를 현실적으로 찾아보기 어렵다는 점입니다. 물론 캐글(Kaggle)이나 서울시 통계자료같이 오픈된 곳들도 있습니다. 하지만 특정 자료들을 다룰 분석의 목적성이 부족했습니다. 나아가 자료를 실제로 어떤 방식으로 분석해야 하는지 접근하기가 쉽지 않았습니다. 그래서 대중적으로도 인기가 있는 한국의 프로야구 자료를 통해 공부해 보기로 했습니다.   


우선 자료를 모으기 위해 KBO 홈페이지(http://www.koreabaseball.com)에 등록된 2010년도부터 현재 2016년도까지의 야구 자료를 모으고 보기 좋게 정리하기 시작했습니다. 네이버와 같은 포털사이트에도 야구 자료가 있더군요. 거기에 있는 야구 자료는 2010년 이전의 자료도 있기에 그쪽으로도 작업하여 데이터를 추가하려 했습니다. 그러나 네이버에 있는 자료는 결측값이 있고, 부정확하거나 부족한 경우가 많아서 공식적인 KBO 사이트만을 참조하여 자료를 만들었습니다.


## 정리 방법

크롤링 능력이 부족하여서 수작업으로 진행했습니다. 자료를 수집하고 모두 정리한 다음에는 직접 눈으로 잘못된 부분이나 빠진 부분이 있는지 확인했습니다. 최종적으로 확인한 후, csv 파일로 만들고 R에서 경기에 관한 정보들을 원정팀, 홈팀, 점수 등의 구분 열로 나누어서 만들었습니다. 이러한 과정을 거쳐 최종적으로 7년 치를 한 번에 모아 단일한 자료로 만들었습니다.


## 사용 분야

저희가 수집하고 가공한 자료는 약 7년 치의 한국 프로야구 데이터가 있습니다. 따라서 특정팀의 연도에 따른 연속적인 변화나 7년 동안의 최대 득점 경기 혹은 포스트 시즌에 올라간 팀 목록 등을 확인할 수 있습니다. 또한, 점수와 홈 & 원정팀, 시즌정보 등을 구분하여 경기정보를 쉽게 확인해 볼 수 있으며 다양한 기록들을 시각화해볼 수 있습니다.


# 사용법

```
## 경기 결과열을 만들어 줍니다.

kbo_total$경기결과  <- ifelse(kbo_total$홈팀점수>kbo_total$원정팀점수,"홈팀승",ifelse(kbo_total$홈팀점수<kbo_total$원정팀점수,"원정팀승","무"))

## 정규시즌만 뽑아 줍니다.

temp <- subset(kbo_total,kbo_total$비고=="정규시즌")
doosan <- subset(temp,temp$홈팀=="두산"|temp$원정팀=="두산")


## 홈팀의 경기 수를 경기 결과별로 분류해줍니다.

temp <- aggregate(홈팀점수~홈팀+원정팀+경기결과,doosan,length)
doosan_home <- temp[temp$홈팀=="두산",]
doosan_home_win <- doosan_home[doosan_home$경기결과=="홈팀승",]


## 원정팀의 경기 수를 경기 결과별로 분류해줍니다.

temp <- aggregate(원정팀점수~홈팀+원정팀+경기결과,doosan,length)
doosan_away <- temp[temp$원정팀=="두산",]
doosan_away_win <- doosan_home[doosan_home$경기결과=="원정팀승",]


## 이긴 횟수와 진횟수가 많은 순서의 팀 배열을 찾습니다.

doosan_home_win$원정팀[order(doosan_home_win$홈팀점수,decreasing = TRUE)]
doosan_away_win$원정팀[order(doosan_away_win$홈팀점수,decreasing = TRUE)]

## 홈팀일 때 승리가 많은 순서대로 나열한 그래프를 그립니다.

p <- ggplot(doosan_home_win,aes(x=원정팀,y=홈팀점수))+
geom_bar(stat="identity",colour = "black",fill="green")+
scale_x_discrete(limits=c("KIA", "한화", "넥센", "롯데", "LG","SK", "삼성", "NC","kt"))+
theme(axis.text.x= element_text(angle=90, hjust=1))+
geom_text(aes(y=홈팀점수,label=홈팀점수),size=4,vjust=1.5)

p+labs(title="두산_상대전적그래프")+ylab("이긴횟수")


## 원정팀일 때 승리가 많은 순서대로 나열한 그래프를 그립니다.

p <- ggplot(doosan_away_win,aes(x=원정팀,y=홈팀점수))+
    geom_bar(stat="identity",colour = "black",fill="green")+
    scale_x_discrete(limits=c("삼성","SK", "롯데", "LG", "넥센", "한화", "KIA", "NC","kt" ))+
    geom_text(aes(y=홈팀점수,label=홈팀점수),size=4,vjust=1.5)

 p+labs(title="두산_상대전적그래프")+ylab("진횟수")

```


## 참고

혹시 자료를 공개하는데 문제가 있을 경우에는 연락을 주시면 즉시 삭제하도록 하겠습니다. 또한 자료에 틀린 곳이 있으면 issues로 올려주시면, 확인한 다음 고치도록 하겠습니다.
