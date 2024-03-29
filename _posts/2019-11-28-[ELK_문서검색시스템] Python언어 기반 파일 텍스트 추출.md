---
Title: "TIKA 라이브러리를 사용해 다양한 파일 텍스트 추출하기"
Date: 2019-11-28 15:21:00 +0900
Categories: TIL Study
---
Elasticsearch를 통해 다양한 파일의 본문검색을 하고자 함.  
pdf, doc, xsls 등의 확장자를 가지는 문서의 text를 추출한 후 json 형태로 재구성한 뒤에  
logstash를 이용해서 파싱된 파일을 elasticsearch 서버로 전송하려고 함.  
가장 먼저해야할 일은 다양한 문서파일의 text를 추출하는 것!  
이 작업을 쉽게 하기 위해서 tika 라이브러리를 사용하고자 함.  
사용 언어는 python.  
추가로 변환한 문서원본의 폴더경로를 같이 json 형태로 넘겨주도록 짰음. 

```python
import tika
from tika import parser
from pprint import pprint as pp
import json
import glob
import os
import sys
#print(sys.argv[0])
input_path = "tika-demos/data/"
#input_path의 모든 파일을 하나씩 불러옴
for input_file in glob.glob(os.path.join(input_path, '*')):
    #파일 이름([ex]tika-demos/data/filename)과 파일의 확장자([ex].docx)를 분리
    filename, fileExtension = os.path.splitext(input_file)
    #파일 경로([ex]tika-demos/data/)와 파일 이름 및 확장자([ex]filename.docx)를 분리
    filepath, filenameExtens = os.path.split(input_file)
  #if 'json'==False in input_file:
    #row_counter = 1
    #with open(input_file, 'r', newline='') as doc_in_file:
    #pp(doc_in_file)
    #pp(input_file)
    #file = "tika-demos/data/1.docx"
    pp([filepath,filename])
    outputpath = filename
    fileExtension = ".json"
    outputpath += fileExtension
    #pp(outputpath)
    #tika라이브러리 기반 문서의 text 추출 (metadata와 contents 파싱)
    parsed = parser.from_file(input_file)
    #print(parsed["content"])
    parsedTotal = {}
    parsedCont = {}
    addPath = {}
    parsedTotal= parsed["metadata"]
    parsedCont["content"] = parsed["content"]
    #문서원본의 경로
    addPath["src_path"] = filepath
    #pp(addPath)
    #pp(parsedCont)
    parsedTotal.update(parsedCont)
    parsedTotal.update(addPath)
    #pp(parsedTotal)
    #json라이브러리를 사용해 dictionary -> json 
    result = json.dumps(parsedTotal, ensure_ascii=False)
    outputFile = open(outputpath,'w',encoding='utf-8')
    print(result, file=outputFile)
    outputFile.close

```
[python os.path 모듈 참고]   
os.path.split(filename) --> 디렉토리와 파일을 분리한다.(리스트로 나타낸다)  
os.path.splitext(filename) --> 확장자만 따로 분류한다.(리스트로 나타낸다)  
os.path.dirname(filename) --> 디렉토리만 출력을 한다.  
os.path.basename(filename) --> 파일이름만 출력을 한다.  
os.path.isdir(파일이름)은 파일이 디렉토리인지에 대해서 검사를 하는 함수이다.  
os.path.walk  
os.path.walk(시작 상위 디렉토리, 각 디렉토리마다 실행할 함수, 함수에 대한 인수)  
os.path.exists() – 파일의 존재 여부 검사  

os.path.join --> 디렉토리 경로명 연결입니다.  
>>> os.path.join('a', 'b', 'c')
      'a\\b\\c'
      
[python os.path 모듈 참고]: https://m.blog.naver.com/PostView.nhn?blogId=jinu5561&logNo=30042778945&proxyReferer=https%3A%2F%2Fwww.google.com%2F
