
### 사용방법

1. FSQLiteDatabase instance 만들고 db file로 Open
2. FSQLitePreparedStatement를 만들기, Create() 함수를 이용해서, db랑 tie시킴
3. SQL과 파라미터를 Binding
4. Execute를 통해 INSERT or Update or Delete 같은 쿼리 실행
5. 다 사용했으면 FSQLitePreparedStatement를 메모리에서 내리기.
6. FSQLiteDatabase Cloas() 하기