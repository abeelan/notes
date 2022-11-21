> python 连接数据库操作



```python
import pymysql

def get_connect():
    connect = pymysql.connect(
    	host="xxx.com",
        port=3306,
        user="test",
        password="test1234",
        database="database_name",
        charset="utf8mb4"
    )
    return connect

def execute_sql(sql):
    connect = get_connect()
    cursor = connect.cursor()
    cursor.execute(sql)  # 执行 sql
    record = cursor.fetchone()  # 查询记录
    return record

if __name__ == '__main__':
    print(execute_sql("select * from table_name"))
```

