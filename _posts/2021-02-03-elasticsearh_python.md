---
layout : single
title : "python - elasticsearch 연동"
categories: elasticsearch
author_profile: true
tags : elasticsearch
---

python - elasticsearch 연동
python에서 elasticsearch 연결하기 위해 pip으로 필요한 라이브러리 설치

```linux
pip install elasticsearch
```



1.  python에서 elasticsearch 연결 

```python
from elasticsearch import Elasticsearch
```



2. elasticsearch가 설치된 서버 주소와 포트를 입력

   ```python
   if __name__ == '__main__':
       url = 'localhost'
       port = '9200'
       es = Elasticsearch(f'{url}:{port}')
       print("Elasticsearch url:port : ", es)
       print("Elasticsearch 정보 : ",es.info())
   ```

3. noritokenizer 설치 및 확인

   ```python
   #디렉토리 이동
   cd /usr/share/elasticsearch/bin/
   
   #설치
   sudo .elasticsearch-plugin install analysis-nori
   
   #확인
   sudo .elasticsearch-plugin list
   ```

   

4. 인덱스 생성 함수 

   ```python
   def create_index(index):
       body={
           "settings" : {
               "index" : {
                   "analysis": {
                       "analyzer": {
                           "my_analyzer": {
                               "type": "custom",
                               "tokenizer": "nori_tokenizer",
                               "decompound_mode" : "mixed"
                           }
                       }
                   }
               }
           },
           "mappings" : {
               "properties" : {
                   "content" : {
                       "type" : "text",
                       "analyzer" : "my_analyzer"
                   }
               }
           }
       }
       if not es.indices.exists(index = index):
           return es.indices.create(index = index, body = body)
   ```

   

5. 인덱스 삭제 함수

```python
def delete_index(index):
    if es.indices.exists(index = index):
        return es.indices.delete(index = index)
```



6. 데이터 삽입 함수

   ```python
   def insert(body, id=None):
       return es.index(index = index, body = body, id = id)
   ```

   

7. 데이터 검색 함수 

   ```python
   def search(index, data=None):
       if data is None:
           data = {"match_all" : {}}
       else:
           data = {"match" : data}
       body = {"from" : 0, "size" : 100 ,"query" : data}
       res = es.search(index = index, body = body)
       return res
   ```

   

8. 데이터 삭제 함수

   ```python
   def delete(index, data):
       if data is None :
           data = {"match_all" : {}}
       else:
           data = {"match" : data}
       body = {"query":data}
       return es.delete_by_query(index = index, body= body)
   ```

   

9. 데이터 갱신 함수

```python
def update(id, doc):
    body = {
        'doc' : doc
    }
    res = es.update(index= index, id= id, body= body)
    return res
```



10.  인덱스 생성

    ```python
    if __name__ == '__main__':
        url = 'localhost'
        port = '9200'
        es = Elasticsearch(f'{url}:{port}')
        print("Elasticsearch url, port : ", es)
        print("Elasticsearch 정보 : ",es.info())
    
        index = 'test'
    
        #인덱스 생성
        r = create_index(index)
        pprint.pprint(es.indices.get_settings(index)) 
    ```



11. 데이터 추가

    - id 파라미터를 넣지 않을경우 임의의 id 값이 생성되며 함수를 반복하면 n개의 데이터가 created된다.
    - 하지만 id 파라미터를 줄 경우 지정한 값으로 id값이 생성되며 함수를 반복하면 1회 실행시에는 created 2회차 부터는 updated로 내용이 갱신이 된다.

    ```python
    data = {'content' : '테스트용 데이터 test data'}
    ir = insert(data, 'test_id')
    pprint.pprint(ir)
    ```

    

12. 데이터 갱신

    - 11번에서 사용한 방법말고 위에서 생성한 방법으로도 데이터 갱신이 가능하다.

    ```python
    ur = update('test_id', {'content' : 'test data, 테스트 데이터'})
    pprint.pprint(ur)
    ```

13.  데이터 검색

    ```python
    sr = search(index, {'content' : '테스트'})
    for result in sr['hits']['hits']:
        print('score:', result['_score'], 'source:', result['_source'])
    ```

    

14. 데이터 삭제

    ```python
    dr = delete(index, {'content' : '테스트'})
    pprint.pprint(dr)
    ```



15. 인덱스 삭제 

```python
dr = delete_index(index)
```



끝.