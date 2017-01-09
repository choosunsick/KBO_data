# KBO_data

안녕하세요. 외대 철학과 스터디 그룹 LOPES의 추선식 입니다.


스터디를 하면서 R에 대한 여러 책을 접하고 해당 책들의 예제 데이터로 학습하면서 느낀 점은 책에서 제시되는 잘 정리되고 학습에 맞게 구성된 데이터가 적고 이런 형태의 자료를 현실적으로 찾아보기 어렵다는 점입니다. 물론 캐글(Kaggle)이나 서울시 통계자료같이 오픈된 곳들도 있습니다. 하지만 특정 자료들에 대한 동기 부여가 잘 안 되며 자료를 실제로 어떠한 방식으로 분석해야 할지 접근하기가 쉽지 않았습니다. 그래서 대중적으로도 인기가 있는 한국의 프로야구 자료를 이용해 보기로 하였습니다.   


위와 관련된 것을 자세히 말하자면 KBO홈페이지에 등록된 2010년도부터 현재 2016년도까지의 야구 자료를 파싱해서 한 번에 모으고 보기 좋게 정리한 자료를 만드는 것입니다. KBO사이트와 더불어 네이버의 야구 자료가 2010년 이전의 자료도 있기에 그쪽으로도 파싱을 하여 데이터를 추가하려 했지만, 네이버의 자료가 결측값이나 잘못되거나 빠진 경우가 많아서 KBO사이트만을 참조하여 자료를 만들었습니다.


자료를 만드는 과정에 대해서는 이후 코드를 통해서 보일 것이지만 미리 간략하게 소개하면 다음과 같습니다. 자료에 기초가 되는 데이터인 경기 기록들을 드래그하여 엑셀 파일로 저장합니다. 이 부분은 크롤링 능력이 부족하여서 수작업으로 진행하였으며 이후 만들어진 자료에서 눈으로 빠진 부분 있는지 확인하였습니다. 기초 엑셀 작업이 7년 치가 완료되면 csv파일로 만든 다음 R에서 경기에 관한 정보들을 원정팀, 홈팀, 점수 등의 구분 열로 나누어서 만듭니다. 이러한 과정을 거쳐 최종적으로 7년 치를 한번에 모아 단일한 자료로 만들어 주면 과정은 종료 됩니다.


저희가 수집하고 가공한 자료는 약 7년 치의 한국 프로야구 데이터가 있기에 특정팀의 연도에 따른 연속적인 변화나 7년의 기간의 최대 득점 경기 혹은 포스트 시즌에 올라간 팀 목록 등을 확인할 수 있습니다. 또한, 점수와 홈팀, 원정팀 구분하여 경기정보를 쉽게 확인해 볼 수 있으며 다양한 기록들을 시각화해볼 수 있습니다.

시각화에 대하여 예를 들어 보자면 9개의 팀들이 홈에서 어떤 원정팀을 만났을 때 평균적으로 낸 점수들을 그려 볼 수 있습니다.

```
temp <- subset(KBO_Total,KBO_Total$비고=="정규시즌") ## 정규시즌만 뽑아줍니다

home.agg <- aggregate(홈팀점수 ~ 홈팀+원정팀,temp,mean) ## 원정팀 별 홈팀들의 평균 점수를 만듭니다.

theme_set(theme_gray(base_family='NanumGothic')) ##  ggplot에서 한글을 보이게 하기 위함입니다.

## 마지막으로 각 원정팀에 대하여 홈팀들이 어느정도의 평균점수를 냈는지 시각화 해봅니다.

ggplot(home.agg,aes(x=홈팀,y=홈팀점수,fill=원정팀))+geom_bar(stat="identity",colour = "black")+
theme(axis.text.x= element_text(angle=90, hjust=1))+
facet_wrap(~원정팀)

```

다음 글에서는 이 자료를 통해 팀별 승률에 대한 함수로 찾아뵙겠습니다. 읽어 주셔서 감사합니다.

아래는 위에서 설명한 파싱 과정에 대한 코드입니다.

```
## 2016년도를 예시로 들겠습니다.

library(data.table)
temp_2016 <- read.csv(file = "2016_kbo_homepage.csv", header=TRUE, stringsAsFactors=FALSE) ## 여는 파일은 해당 년도의 경기 자료를 드래그해서 엑셀로 만든 파일입니다.
temp_2016.table <-data.table(temp_2016) ## 보기 편하게 하기 위하여 변경
temp_2016.table$경기[nchar(temp_2016.table$경기)==0] <- NA
complete.cases(temp_2016.table)
all(complete.cases(temp_2016.table))
temp_2016.table <- na.omit(temp_2016.table) ## 자료를 만드는 과정에서 생성될 수 있는 빈곳을 지웁니다.
all(complete.cases(temp_2016.table))
ifelse(all(complete.cases(temp_2016.table)) == TRUE, print("잘 되었음!"), print("잘못되었습니다.")) ## NA값이 없음을 확인합니다.

## 경기 날자 정보열을 만드는 과정입니다.
temp_2016.table$경기날짜및시간 <- temp_2016.table$날짜

for (i in 1:NROW(temp_2016.table$경기날짜및시간)) {
  if(temp_2016.table$경기날짜및시간[i] == "") { ##만약 i행의 날짜가 여백이라면
    temp_2016.table$경기날짜및시간[i] <- temp_2016.table$경기날짜및시간[i-1] ##i-1번째 행의 데이터를 i행에 입력하라
  }
}

temp <- do.call(rbind,(lapply(temp_2016.table$경기날짜및시간, function(x) paste("2016-", sub("\\.", "-", strsplit(x[1], "\\(" )[[1]][1]),sep=""))))
temp <- as.data.frame(temp)
temp_2016.table$경기날짜및시간 <- temp

temp_2016.table$경기날짜및시간 <- paste(temp_2016.table$경기날짜및시간, temp_2016.table$시간)

temp_2016.table$경기 <- gsub("\n"," ",temp_2016.table$경기) ## 경기 항목에서 \n 을 제거합니다.

## 경기 정보열을 만드는 과정입니다.

kBO.test <- function(x){
  temp <- strsplit(x, " ")
  print(temp[1])
  temp2 <- strsplit(temp[[1]][2], "vs")
  print(temp2[[1]][1])
  print(temp2[[1]][2])
  ifelse(is.na(temp2[[1]][2]) == TRUE, temp3 <- data.frame(원정팀 = temp[[1]][1], 원정팀_점수 = "", 홈팀_점수 = "", 홈팀 = temp[[1]][3], 비고 = "경기 없는 날"), temp3 <- data.frame(원정팀 = temp[[1]][1], 원정팀_점수 = temp2[[1]][1], 홈팀_점수 = temp2[[1]][2], 홈팀 = temp[[1]][3], 비고 = "" ))  
  ## 두번째 값이 없으면, 즉 특 홈팀 점수가 없으면, "경기가 없는 날"을 넣어줍니다. 그렇지 않은 결과는 그대로 리턴해줍니다.
  return(temp3)  
}
## 앞의 함수를 이용하여 자료를 합쳐줍니다.
temp_2016.table <- cbind(temp_2016.table, do.call(rbind,(lapply(temp_2016.table$경기, kBO.test))))
temp_2016.table$비고 <- as.character(temp_2016.table$비고)

## 경기 비고열을 만드는 과정입니다.

temp <- read.csv(file = "kBO_2016_PostSeason.csv", stringsAsFactors = FALSE) ## 포스트 시즌 및 시범경기의 날짜에 대한 자료를 열어줍니다.
kBO_2016_시범경기 <- as.Date(as.character(temp$시범경기), format="%Y-%m-%d", origin="1970-01-01")
kBO_2016_시범경기 <- na.omit(kBO_2016_시범경기)
kBO_2016_올스타 <- as.Date(as.character(temp$올스타), format="%Y-%m-%d", origin="1970-01-01")
kBO_2016_올스타 <- na.omit(kBO_2016_올스타)
kBO_2016_와일드카드 <- as.Date(as.character(temp$와일드카드), format="%Y-%m-%d", origin="1970-01-01")
kBO_2016_와일드카드 <- na.omit(kBO_2016_와일드카드)
kBO_2016_준플레이오프 <- as.Date(as.character(temp$준플레이오프), format="%Y-%m-%d", origin="1970-01-01")
kBO_2016_준플레이오프 <- na.omit(kBO_2016_준플레이오프)
kBO_2016_플레이오프 <- as.Date(as.character(temp$플레이오프), format="%Y-%m-%d", origin="1970-01-01")
kBO_2016_플레이오프 <- na.omit(kBO_2016_플레이오프)
kBO_2016_한국시리즈 <- as.Date(as.character(temp$한국시리즈), format="%Y-%m-%d", origin="1970-01-01")
kBO_2016_한국시리즈 <- na.omit(kBO_2016_한국시리즈)

test <- function(x){
  temp <- c("")
  ifelse(any(x == kBO_2016_올스타) == TRUE, temp <- "올스타","올스타경기아님")
  ifelse(any(x == kBO_2016_시범경기) == TRUE, temp <- "시범경기","시범경기아님")
  ifelse(any(x == kBO_2016_와일드카드) == TRUE, temp <- "와일드카드","와일드카드경기아님")
  ifelse(any(x == kBO_2016_준플레이오프) == TRUE, temp <- "준플레이오프","준플레이오프아님")
  ifelse(any(x == kBO_2016_플레이오프) == TRUE, temp <- "플레이오프","플레이오프아님")
  ifelse(any(x == kBO_2016_한국시리즈) == TRUE, temp <- "한국시리즈","한국시리즈아님")
  ifelse((temp == "" ) == TRUE,  temp <- "정규시즌", "error")
  return(temp)
}

시즌구분 <- do.call(rbind,(lapply(as.Date(as.character(temp_2016.table$경기날짜및시간)), test)))
temp_2016.table$비고 <- 시즌구분
write.csv(temp_2016.table, file="KBO_Home_2016.csv",row.names = FALSE)
## 윗 줄에서 파일을 저장하는 이유는 여기서 만든 데이터 프래임이 팩터형으로 되어 그것을 단순하게 처리하기 위해서임. 아래서 바로 읽음.

## 승,패열을 만드는 과정입니다.

temp_2016.table <- read.csv(file = "KBO_Home_2016.csv", header=TRUE, stringsAsFactors=FALSE)
temp_2016.table$홈팀_결과<- ifelse(temp_2016.table$홈팀_점수>temp_2016.table$원정팀_점수,"승",ifelse(temp_2016.table$홈팀_점수<temp_2016.table$원정팀_점수,"패","무"))
temp_2016.table$원정팀_결과<- ifelse(temp_2016.table$원정팀_점수>temp_2016.table$홈팀_점수,"승",ifelse(temp_2016.table$원정팀_점수<temp_2016.table$홈팀_점수,"패","무"))
na.omit(temp_2016.table) ## 경기가 취소된 날짜들을 제거해줍니다.
write.csv(temp_2016.table, file="KBO_2016.csv",row.names = FALSE)

```
