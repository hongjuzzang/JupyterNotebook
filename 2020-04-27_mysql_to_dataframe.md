Jupyter에서 mysql 데이터읽어서 DataFrame으로 만들기
```python
from sqlalchemy import create_engine
import pymysql
import pandas as pd
# MySQL Connection 연결
db_connection_str = 'mysql+pymysql://[db유저이름]:[db password]@[host address]/[db name]'
db_connection = create_engine(db_connection_str)
conn = db_connection.connect()
df = pd.read_sql('SELECT * FROM api_storereview', con=conn) # 여기서 sql문, 나는 api_storereview 테이블을 dataframe으로 전환
conn.close() # 커넥션 끊기
```
