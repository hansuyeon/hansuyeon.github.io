---
Title: "Kibana create a index pattern 에러 수정"
date: 2019-12-11 16:00:00 +0900
categories: Error
---
평소처럼 Kibana에서 index pattern을 생성하는 중이었는데,  
갑자기 뜬금없이 error:Forbidden 메세지를 받았음  
당황해서 구글 검색을 하니까
[이 블로그]를 발견했고, 그대로 따라하니까 진짜 에러가 사라짐!  
블로그 내용을 보니 아래와 같은 상황이 문제라는 것 같다....  
# elasticsearch's index is made read-only and never set back when disk watermark reached high  
해결방법은 아래와 같음! 밑에 컬을 날려주니까 됐다.  

```bash
curl -XPUT -H "Content-Type: application/json" https://[YOUR_ELASTICSEARCH_ENDPOINT]:9200/_all/_settings -d '{"index.blocks.read_only_allow_delete": null}'
```
정확한 의미를 모르지만 수정을 하긴 했다 깔깔  

[이 블로그]: https://blog.mimimi.fun/ki/
