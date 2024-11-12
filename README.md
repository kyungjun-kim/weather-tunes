# 프로젝트 최종 보고서
### **날씨와 음악 선호도간의 상관관계 및 추세 파악 대시보드 구성**

- 7igma 팀
- 프로젝트 기간 : 2024/11/01 - 2024/11/12

## 1. 프로젝트 개요

### 프로젝트 배경

음악은 우리의 감정과 밀접하게 연결되어 있으며, 날씨는 우리의 감정에 큰 영향을 미치는 주요 요소 중 하나입니다. 많은 사람들이 특정 날씨나 기후 조건에 따라 다른 종류의 음악을 즐기며, 이로 인해 음악 취향이 달라질 수 있습니다.

본 프로젝트의 배경은 날씨와 음악 선호도 사이의 연관성을 더 깊이 이해함으로써, 개인의 기분과 현재 환경에 맞춘 더 나은 음악 추천 서비스를 제공하고자 하는 데 있습니다.

### 목표 및 기대효과

본 프로젝트의 목표는 다양한 국가의 음악 스트리밍 데이터를 수집하고, 기후 데이터와 비교하여 특정 날씨 조건이 사람들이 선호하는 음악 장르와 곡에 미치는 영향을 이해하는 것입니다.

이를 통해 사용자의 기분과 현재 환경에 맞는 음악 추천을 제공할 수 있는 방법을 모색하고, 사용자 경험을 향상시킬 수 있는 맞춤형 음악 추천 시스템의 발전에 기여할 수 있을 것으로 기대됩니다.


## 2. 프로젝트 세부 내용

### 시스템 아키텍처
![image](https://github.com/user-attachments/assets/749ecc32-3e92-4cc3-a307-fec027623132)


### 데이터 수집 및 전처리

### 날씨 데이터 수집

- 데이터 수집 목표
    - 6개국(한국, 일본, 미국, 영국, 호주, 브라질)의 최근 3개월 날씨 데이터 수집
    - `weather_data_extract.py`
- 데이터 수집 과정
    
    `get_weather_data(*past_days*, *country*, *latitude*, *longitude*)`
    
    - `past_days`: 수집 기간을 설정하는 파라미터 (오늘로부터 n일 전까지)
    - `country`: 수집할 국가 이름
    - `latitude`: 수집할 국가의 위도
    - `longtitude`: 수집할 국가의 경도
    1. Open Meteo API로부터 `past_days` 간의 각 나라 별 날씨 데이터 호출
    2. 응답 데이터를 NumPy 배열로 변환해 일별로 처리
    3. Pandas DataFrame을 이용해 각 날짜에 대해 국가 정보와 기상 변수 추가(칼럼 추가)
        1. `convert_weather_code_to_wmo(code)` 함수 이용해 wmo 코드 변환
- 수집 데이터 예시 및 링크
    - 구글스프레드시트 : https://docs.google.com/spreadsheets/d/12RwjYgWCsUrVHekVQGMxix5M379Kj-yQ5bPM-G-6Ei8/edit?gid=0#gid=0
    - weather_csv_description.pdf
    - ![image](https://github.com/user-attachments/assets/6f7e5b7a-bf3f-462a-9708-4ba4e5d6be7a)
    
    - 2024-11-03_weather.csv
    - ![image](https://github.com/user-attachments/assets/ea6bd21c-41b6-4ab9-aaa1-7852ebe024b0)

    

---

### 음악 데이터 수집

- 데이터 수집 목표
    - 6개국(한국, 일본, 미국, 영국, 호주, 브라질)의 최근 3개월 일간 차트 데이터 수집
- 데이터 수집 과정
    1. (Amazon S3에 업로드 해놓은 기존 track_data.csv가 있을 경우) track_data.csv 다운로드
        
        → `download_from_s3(bucket_name, s3_path, local_path)`
        
    2. Selenium을 활용하여 [Spotify](https://charts.spotify.com/charts/view/regional-global-daily/latest) 의 일간 차트 csv 파일 다운로드
        
        → `download_csv_files(sp_email, sp_password, countries, dates, download_directory)`
        
    3. Spotify API를 호출하여 상세 트랙 정보가 담긴 track_data.csv 파일 업데이트 (없을 시 생성) 
        
        → `fetch_track_info_and_update_db(track_id, headers, track_data)`
        
    4. 일간 차트 csv 파일을 일자별로 통합 및 country, data 컬럼 추가 후, 원본 파일 삭제  
        
        → `process_csv_files(directory, track_db, headers, output_directory, country_map)`
        
    5. Amazon S3 버킷에 일간 차트 데이터 및 track_data.csv 업로드
- 수집 데이터 예시 및 칼럼 문서
    
    [music_csv_description.pdf](https://prod-files-secure.s3.us-west-2.amazonaws.com/29dd0948-7b38-443b-af5a-c6bc908226ab/1f42e1ec-85f1-45ac-a637-b68687e520a0/music_csv_description.pdf)
    
    [daily_2024-11-03.csv](https://prod-files-secure.s3.us-west-2.amazonaws.com/29dd0948-7b38-443b-af5a-c6bc908226ab/807362c1-7c93-4234-953a-b3822e0e9473/daily_2024-11-03.csv)
    
    [track_data.csv](https://prod-files-secure.s3.us-west-2.amazonaws.com/29dd0948-7b38-443b-af5a-c6bc908226ab/78441935-0777-467d-b3b3-0f95bf8f9c51/track_data.csv)
    

---

### ETL 파이프라인 초기화 모듈

- `data_initialize.py` 파일을 통해 위 기능 함수들을 모아 한 번에 처리
- API KEY 등 민감한 개인정보는 `secrets.json` 파일을 통해 개별 관리
    - .gitignore를 통해 Github repository에는 노출되지 않도록 처리함
- `config.json` 파일을 통해 데이터 수집에 필요한 국가 목록, 좌표 등의 정보를 가져옴
- 수집한 데이터를 로컬에 임시 저장한 뒤, S3 버킷에 업로드
    - 임시 저장한 데이터는 업로드 완료 시 자동 삭제
- argparse를 이용해 데이터 수집 기간 설정 가능 (default=89(일), min = 2(일))

-  데이터 웨어하우스 구조
-  ![image](https://github.com/user-attachments/assets/cd2d5ba7-4fe2-483e-9b12-2a97e71882d0)
- **데이터베이스**
    - 2개로 구분하여 `FEAT1` 은 개발용 DB로 `DEV`는 실제 운영용 DB로 사용될 수 있게 구성.
- **SCHEMA**
    - ‘RAW_DATA’, ‘ANLYTICS’, ‘REPORTING’, ‘TEST’ 로 구분
    - `RAW_DATA`  Amazon S3에 저장되어 있는 COPY 데이터
    - `ANALYTICS`  분석 기반 가공 데이터
    - `REPORTING`  시각화를 위한 데이터
    - `TEST` 테스트를 위한 임시 스키마
- **TABLE** (ANALYTICS 기준)
    - `FACT_MUSIC_DAILY` 스트리밍 차트와 트랙 ID 등의 정보
    - `FACT_WEATHER_DAILY` 최고 기온, 최저 기온, 강수량 등의 날씨 정보
    - `DIM_ARTIST` 아티스트에 대한 정보 (아티스트명에 따른 uuid, 아티스트명)
    - `DIM_DATE`  각 날짜에 대한 연도, 월, 일, 요일 번호, 요일 이름에 대한 정보
    - `DIM_LOCATION` 발매 국가에 대한 코드, 이름, 위/경도, 좌표 등에 대한 정보
    - `DIM_TRACK` 트랙의 장르나 분위기를 파악할 수 있는 메타 데이터(BPM, 에너지 등)
    - `DIM_WEATHER_CODE` WMO ID와 설명에 대한 정보

### Docker Container 이용해 일일 데이터 수집 자동화 (+Github Secrets 적용)

- 아래 `Dockerfile` 을 이용해 이미지 생성, LInux 서버에서 Crontab을 이용해 매일 데이터 수집
- Github CLI를 이용해 PAT 로그인을 통해 민감한 정보 보호

### 테이블 구조 (ERD)

- `analytics` 스키마
    - **`Star Schema`** 를 사용하여 `Fact` 테이블과 `Dimension` 테이블로 나누고 구조화
    - Fact 테이블은 여러 Dimenstion 테이블과 연결되어 있음
    - `Fact 테이블`은 수치 정보를 포함하는 중앙 테이블로 음악과 날씨데이터 저장
    
           (fact_music_daily, fact_weather_daily)
    
    - `Dimension 테이블`은 음악과 날씨 데이터의 메타 데이터
    
          (dim_location, dim_artist, dim_track, dim_date, dim_weather_code)
      ![image](https://github.com/user-attachments/assets/4f2ef0f7-b9d5-469e-b82a-e90e51d46019)

 ### 대시보드 시각화 및 결과
 ![image](https://github.com/user-attachments/assets/1d360070-df75-409f-b1b8-76e15dbcd5af)
 ### **요일별 감정 비교**
 ![image](https://github.com/user-attachments/assets/928618fe-5b20-4812-8590-53bfeeaebf67)
#### → 금, 토, 일요일의 평균 감정 점수가 월~목요일보다 높은 경향이 관찰되었습니다. 이는 주말이 다가올수록 사람들의 음악 선호도가 보다 밝고 긍정적인 음악으로 바뀌는 것을 보며 사람들이 주말에 긍정적인 감정을 느끼며 더 활기찬 음악을 선택하는 경향이 있는 것 같아 흥미롭게 느꼈습니다.

### **날씨 상태와 댄스 가능성**
![image](https://github.com/user-attachments/assets/7f7288ef-1db6-42d5-81df-e4a77e93983c)
#### → 맑은 날(Clear)에는 다른 흐린 날씨 상태보다 댄스 가능성이 높은 음악이 선호되는 것으로 나타났습니다. 이는 맑은 날씨가 활발한 분위기나 긍적적인 음악을 선택하는 경향을 있다고 볼 수 있습니다.

## 3. 프로젝트 결론

### 대시보드를 통해 도출한 주요 인사이트

**날씨와 BPM의 관계**

- 날씨 상태에 따라 평균적으로 선호되는 BPM이 다르게 나타났습니다. 예를 들어, 맑은 날에는 댄스 음악의 평균이 높아 활발한 분위기의 음악을 선택하는 경향이 있다고 보고 있습니다.

**국가별 선호 장르 차이**

- 국가별로 선호하는 음악 장르와 BPM에 차이가 있음을 확인하였으며, 특정 아티스트와 장르가 날씨에 따라 재생 수가 크게 달라지는 것을 파악할 수 있었습니다. 이를 통해 각국의 문화적 특성이 음악 선호도에 영향을 미친다는 것을 알 수 있었습니다.

**시각적 통합**

- OpenStreetMap 기반의 지도 시각화는 국가별 주요 아티스트를 한눈에 보여주어, 특정 국가에서 활동하는 아티스트와 그들의 인기도를 쉽게 비교할 수 있었습니다.

### 성취점

**데이터 수집 및 통합**

- 6개의 국가(한국, 일본, 미국, 영국, 호주, 브라질)의 음악 스트리밍 데이터와 기후 데이터를 성공적으로 수집하고 통합하여, 특정 날씨 조건이 음악 선호도에 미치는 영향을 분석할 수 있었습니다.

### 개선점

**데이터의 다양성 확대**

- 날씨 데이터는 다양한 문화권과 기후 유형을 포함하도록 확대하고, 음악 데이터는 여러 음악 스트리밍 서비스의 API를 추가적으로 활용하여, 각 지역의 음악 선호도를 더 폭넓게 반영할 필요가 있습니다.

**대시보드 활용성** 

- 이 대시보드는 기업이 다양한 기후 조건에서의 음악 소비 패턴을 이해하고, 이를 마케팅 전략에 활용할 수 있는 유용한 도구로 사용될 수 있습니다. 예를 들어, 날씨에 따라 특정 장르나 아티스트를 추천하는 맞춤형 서비스로의 확장 가능성을 시사합니다.
- 기상 데이터와 음악 데이터의 융합이 새로운 인사이트를 제공할 수 있음을 확인하였으며, 향후 더 많은 변수와 지역을 추가하여 보다 깊이 있는 분석을 진행할 수 있을 것입니다.

**클라우드 환경에서의 데이터 수집**

- 클라우드에서 Selenium을 사용하기 위한 환경 구축이 복잡하고, 사용 및 이슈 분석이 어렵습니다.
- 클라우드 환경의 IP에서 Selenium을 통해 웹 페이지에 접근할 경우, 봇으로 인식하는 경우가 매우 많아 수집에 제약이 있었습니다.
- 위 이유들로 인해 초기 기획과 달리 로컬 서버(컴퓨터)에서 데이터를 수집할 수 밖에 없었습니다. 클라우드 환경에서도 잘 동작할 수 있도록 할 수 있는 방안을 모색할 필요가 있습니다.

### 한계점

**데이터의 제약**

- 특정 국가 및 지역에 한정된 데이터 수집으로 인해 모든 기후와 문화적 차이를 충분히 반영하지 못했습니다. 특히 날씨 데이터의 경우 최대 3개월 전까지의 정보만 제공되어, 장기적인 기후 변화를 반영한 분석에는 제한이 있었습니다.

**Spotify의 클라우드 IP 차단 문제**

- Spotify의 Charts페이지에서 얻을 수 있는 csv파일을 Selenium을 통해 자동화했으나, 클라우드 환경에서 IP 또는 계정이 차단되거나 봇으로 판단해 데이터를 수집에 제한이 있었습니다.

