---
Title : "[ELK_문서검색시스템] 추가된 문서만 Elasticsearch로 보내기"
Date : "2019-12-13"
Categorization :
---

문서 검색 시스템을 구축하기 위해서 여태까지 한 작업을 간단히 나열해보면,  
1. ELK 설치 및 환경설정  
2. 다양한 문서 (pdf, csv, hwp 등)의 텍스트 추출 및 json 형태로 변환을 위한 python 코드 작성  
3. Logstash를 활용해 변환된 문서의 json 파일을 Elasticsearch로 전송 및 색인 작업

여기까지 완료했음.  
마지막 3번을 하긴 했는데 logstash를 실행할 때마다 폴더에 있는 문서 전체를 다시 색인을 하고 있음.  
logstash 를 실행시킨 상태에서 .conf 파일에 설정해 둔 폴더 안으로 문서가 추가되면  
딱 추가된 문서만 es에 업로드되는데,  
logstash를 껐다가 다시 키면 .conf파일 안의 모든 문서가 es로 한번 다시 업로드 됨.....  
이때 추가된 문서의 _id 값은 또 달라서 같은 문서인데 다른 _id로 Elasticsearch에 추가 저장이 된댜  
중복 없이 새로 추가된 문서만 전송하고 저장하고 싶었음!  
찾아보니까 logstash *.conf 파일에서 File input plugins에서 start_position 설정옵션을 바꾸면  
쉽게 추가된 문서만 전송할 수 있다고 함 야호~  

```bash
# name.conf 파일을 열어서
vi my_conf_file_directory/name.conf
# logstash name.conf 파일 input 설정 추가
input{
  file{
   path => "보낼 파일 path"
   # start_position 을 "end"로 설정
   start_position => "end"  
```
  
# logstash input plugin 
start_position => "beginning" or "end"  
start_position은 logstash를 시작할 때 file의 어느 부분부터 읽을지를 결정하는 설정. file을 최초 읽을 때만 적용  
  
sincedb_path => "/var/lib/logstash/plugins/inputs/file/.sincedb_d95dab0ed656526494a9672a508c1315"  
sincedb라는 이름의 파일로 현재까지 logstash가 전송한 데이터의 line 포지션을 트랙킹할 수 있음  
참고로 sincedb 파일은 /var/lib/logstash/plugins/inputs/file 에 위치함!  
Sincedb 파일을 열면 4~6개의 컬럼으로 구성이 되는데 각 컬럼은  
1. The inode number (or equivalent).  
2. The major device number of the file system (or equivalent).  
3. The minor device number of the file system (or equivalent).  
4. The current byte offset within the file.  
5. The last active timestamp (a floating point number)  
6. The last known path that this record was matched to (for old sincedb records converted to the new format, this is blank.  
를 뜻함.  
logstash가 처음 시작될 때는 start_position 옵션에 따라서 결정되고, 그 이후로는 sincedb에 저장된 offset부터 읽기 시작함.
   
stat_interval => "1 second"  
파일 갱신 확인 주기 (default:1초)  
sincedb_write_interval => "15 seconds"  
처리 중인 offset을 얼마의 주기로 sincedb에 기록할지 (default: 15초). 중복 유입에 관련된 설정  
  
아무튼 start-position을 end로 해줬더니 추가된 문서만 Elasticsearch에 잘 저장됨!
근데 여기서 또 문제 발생ㅠㅠ  
logstash를 실행시킨 후에 새로운 doc를 넣으면 걔만 잘 저장되는데,  
새로운 doc를 폴더에 먼저 넣고 logstash를 키면 미리 넣어둔 새 doc는 es에 저장이 안됨....반응이 없다  
sincedb 열어봐도 변화가 없음..  
일단 여기까지 보류. logstash가 꺼지지만 않으면 문제는 없을거야......
