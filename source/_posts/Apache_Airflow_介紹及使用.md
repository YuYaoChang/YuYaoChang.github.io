------
title: Apache_Airflow_介紹及使用
date: 2023-01-12
categories: Airflow
tags: 
 - Airflow
 - Python
 - docker
------

隨著工作上資料量、流程複雜度持續的增加，使用 Crontab 管理自動更新排程，已無法順利進行資料更新。
容易遇到前一個工作尚未進行完，下一個工作又要開始執行，且下一個工作需要使用上一個工作所產生的資料；
若繼續使用 Crontab 則會遇到資料量持續增加，而打亂自動更新排程，要常常調整排程時間設定來緩解這個情形，
因此，這次透過導入 Airflow 來解決此問題。

## Airflow 介紹
Airflow 是一個工作流程管理系統(Workflow Management System)，將有相關的工作整合為一個有向無循環圖 DAG (Directed Acyclic Graph)，並提供多種 Operator，例如 Bash Operator、Python Operator 等，甚至可直接對 GCP、S3、Slack 等進行操作；DAG 是一個 Python 程式，可達到 Infrastructure as code，減少維運上的複雜度。
<img src="/images/airflow/airflow_structure.png">
<center><a  href="https://3lexw.medium.com/apache-airflow-2-1-%E5%9F%BA%E7%A4%8E%E6%95%99%E5%AD%B8-2-%E5%AE%89%E8%A3%9D%E5%8F%8A%E5%9F%BA%E7%A4%8E%E8%A8%AD%E5%AE%9A-5622f7cff10d">圖片來源：3LexW</a></center>
<!--more-->

### Airflow 主要的元件
1. Airflow Webserver
提供圖形化介面，可快速看到排程執行的狀態、Log 或是手動執行等。
2. Airflow Scheduler
負責排程，從 Metadata Database 中找尋 DAG 跟 Task 的狀態，並判斷要將哪些 Task 傳送給 Executor 安排執行。
3. Airflow Executor
Executor 是一個 Queue Process，從 Scheduler 接收要執行的 Task，並將這些資訊存進 Queue，並從 Queue 中取出 Task 安排給閒置的 Worker 執行。
4. Airflow Worker
實際的排程工作就交由 Worker 來執行，同一個 Airflow cluster 中可以有多個 Worker，並且可透過指定 worker queue 使工作能在特定的資源上運作，而所有資料都被存在 Metadata Database 裡。
5. Metadata Database
儲存 DAG 執行的資訊、狀態 以及 Airflow 本身的設定如用戶、連線等設定。Web Server 顯示的資訊就是從 Metadata DB 來的，而 Scheduler 也會更新這些資訊到 DB 讓 Web 和 Scheduler 可以同步。

### Airflow 優缺點
- 優點
1. 清晰易懂的管理介面
2. 現成的 Operator 方便串接各式系統
3. 較容易管理複雜的工作流程
4. Workflow as Code
5. 各個工作失敗時自動重試
6. 可設置失敗提醒(Email、Slack等)
- 缺點
1. 多一套系統需維護
2. 需設定一些相依服務（資料庫、RabbitMQ 之類的）
3. 部署多組 Worker 時較麻煩
4. 只能使用 pools 去限制同時有多少的 DAG 在運作，無法直接分配相應的 CPU 數量和 memory 大小

------------------------------------------------------------------------
## Airflow 安裝

### pip 安裝
1. Airflow 最簡單的安裝方法就是使用 pip 安裝
```linux
# 安裝完整相關套件
$pip3 install apache-airflow

# 只要安裝部分相關套件(只安裝 celery, slack, redis)
$pip3 install apache-airflow[celery,slack,redis]
```
2. 安裝後可執行初始化資料庫的指令，即可使用 sqlite 來儲存設定與 log 等
```linux
$airflow initdb
```
3. 可透過編輯預設路徑中 `$home/airflow/airflow.cfg` 的設定，改為使用 MySQL 等資料庫
```linux
# airflow.cfg
sql_alchemy_conn = mysql://DB_USERNAME:DB_PASSWORD@DB_HOST:DB_PORT/DB_database

# 更改完再初始化一次資料庫
$airflow initdb
```
4. 可用 command line 指令來啟動，也可以使用 systemd 等方式來管理
```linux
$airflow webserver
$airflow scheduler
$airflow worker
```

### 使用 docker compose 安裝 (推)
1. 建置 Airflow 環境
- 新建資料夾
```linux
$mkdir airflow
$cd airflow
```
- 下載官方提供 docker-compose 文件
```linux
# 以 Airflow 2.5.0 為例
$curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.5.0/docker-compose.yaml'
```
> 在 docker-compose.yaml 設定環境變數
> Airflow 提供了透過環境變數傳遞設定值的做法，格式上會像 `AIRFLOW__<which part>__<which key>`
> 若設定 core 裡面的 sql_alchemy_conn，就會像是`export AIRFLOW__CORE__SQL_ALCHEMY_CONN=your database`
> 調整時區則是在 environment 部分加入 `AIRFLOW__CORE__DEFAULT_TIMEZONE: 'Asia/Taipei'`
- 產生相依目錄並設定 Airflow 使用者權限
```linux
$mkdir -p ./dags ./logs ./plugins
$echo -e "AIRFLOW_UID=$(id -u)" > .env
```
2. 資料庫初始化
```linux
$docker-compose up airflow-init
```
3. 啟動 Airflow 服務
```linux
$docker-compose up -d
$docker-compose ps # check it
```
4. 下載 airflow 命令工具
```linux
$curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.5.0/airflow.sh'
$chmod +x airflow.sh
```
5. 撰寫 dags 程式
將 dags 程式放置於 Container 掛載目錄 dags，後面會再談到如何撰寫 dags 程式。
6. 透過 airflow.sh 執行 DAG 程式
建立運行 python container 
```linux
$./airflow.sh bash
```
執行 python 檔案
```linux
$python dags/file.py
```
7. 網頁中查看新建立的 DAG
Airflow Web Server
http://localhost:8080
8. REST API
使用 [Airflow REST API](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html) 可以取代在 WebUI 上的工作，進行 DAG 的抓取執行刪除等工作
```linux
$curl -X GET 'http://<主機>:8080/api/v1/dags' --user "airflow:airflow"
```
回傳
```json
{
  "dag_id": "file",
  "description": null,
  "file_token": "",
  "fileloc": "/opt/airflow/dags/file.py",
  "is_active": true,
  "is_paused": false,
  "is_subdag": false,
  "owners": [],
  "root_dag_id": null,
  "schedule_interval": {
    "__type": "TimeDelta",
    "days": 1,
    "microseconds": 0,
    "seconds": 0
  },
  "tags": []
}
```

------------------------------------------------------------------------
## DAG (Directed Acyclic Graph)

Airflow 只需使用最基本的 Bash Operator (其他的 Operator 後續將會介紹)，即可執行工作排程，而 Dag 建立的部分，除了直接拉變數出來建立，也可以透過以下兩種方式建立：
1. 透過 with 建立 Dag
```python
dag = DAG(dag_id="first_dag", tags=['user'], start_date=datetime.today())
with dag:
    start_task = EmptyOperator(task_id="start_task")
    end_task = EmptyOperator(task_id="end_task")
    first_task = BashOperator(task_id="first_task",
                              bash_command=f"echo execute time: {datetime.now()}")
```
2. 透過 decorator 建立 Dag
需要於外部將該 Dag 的函式丟給一個變數，airflow 才可以抓到該 Dag
```python
from airflow.decorators import dag

@dag(tags=['user'], start_date=datetime.today())
def simple_dag():
    start_task = EmptyOperator(task_id="start_task")
    end_task = EmptyOperator(task_id="end_task")
    first_task = BashOperator(task_id="first_task",
                              bash_command=f"echo execute time: {datetime.now()}")

my_dag = simple_dag()
```

- [dag 參數](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/models/dag/index.html)、[operator 參數](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/models/baseoperator/index.html)

完成 DAG 程式，可直接放入 airflow.cfg 指定的 DAG 資料夾，並且在介面上點選啟用即可

- 範例
完整 DAG 程式：
```python
from datetime import datetime, timedelta

from airflow import DAG
from airflow.operators.bash_operator import BashOperator

# default setting
default_args = {
    'owner': 'OWNER_NAME', # dag 管理者名稱
    'depends_on_past': False,
    'start_date': airflow.utils.dates.days_ago(3), # 開始時間可使用相對時間，也可使用絕對時間
    'email': ['OWNER_EMAIL'], # 要寄的 email
    'email_on_failure': False, # 失敗時寄 email
    'email_on_retry': False, # 重試時寄 email
    'retries': 1, # 執行失敗時，要重試多少次
    'retry_delay': timedelta(minutes=5), # 執行失敗時，過多久才重試
}

# dag setting
dag = DAG(
    'example',
    default_args=default_args,
    description='Example',
    schedule_interval='10 * * * *') # 排程週期：直接按照 cron 的時間格式

# 第一次的執行 DAG 的時間是 start_date + schedule_interval，但執行的 execution_date 將會是前一天

# define tasks
start_task = EmptyOperator(task_id="start_task")

task1 = BashOperator(
    task_id='task_1',
    bash_command='/path/to/script/one.sh',
    dag=dag)

task2 = BashOperator(
    task_id='task_1',
    bash_command='/path/to/script/two.sh',
    dag=dag)

task3 = BashOperator(
    task_id='task_1',
    bash_command='/path/to/script/three.sh',
    dag=dag)

end_task = EmptyOperator(task_id="end_task")

start_task >> task1 >> [task2,task3] >> end_task
```

> 'start_date' : datetime(2022, 1, 1, 0, 0)，執行時間將在 2022-01-02 做第一次執行
> 邏輯：在 2022-01-01 23:59 結束以後，也就是 2022-01-02 00:00 的時候，將 2022-01-01 所有的使用者資料做彙總

### task 狀態

![](https://i.imgur.com/gW6GGrd.png)
圖片來源：[FrankYang0529](https://tw.coderbridge.com/series/c012cc1c8f9846359bb9b8940d4c10a8/posts/3d1f5ea33f5f4a3bac33f95099e376e8)

- No status
當手動 Trigger DAG 或是 Scheduler 排程 DAG 後，DAG 的 Task 會先被創造成 Task Instance 並寫進 Database
- Scheduled
當 Scheduler 確認某個 Task 需要被執行時，這時 Task 的狀態就會變成 Scheduled
- Queued
當 Scheduler 把確定要執行的 Task 發送給 Executor 時，相當於把 Task 放入 Queue 裡
- Running
Executor 將 Task 發送給閒置的 Worker
- 執行的結果
透過 Executor 將 Task 標示成 Success 或是 Failed

### Operator
[Basic Operator](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/index.html)

- Pools
資源控管用的變數，能夠限定執行的 operator 數量，在 UI 上新增一個 Pool
在 operator 設定 pool 參數
```python
extract_product = MyPostgresOperator(
    sql='select_product.sql'
    task_id='extract_product',
    pool='pool_name')
```

#### BranchPythonOperator
可以透過回傳的文字，決定下一個 Task 要執行什麼
```python
branching = BranchPythonOperator(
    task_id='branching',
    python_callable=lambda **context: 'store_in_redis' if int(context['task_instance'].xcom_pull(task_ids='get_timestamp')) % 2 == 0 else 'skip',
    provide_context=True,
    dag=dag,
)
```

#### 自創 Operator
Airflow 提供[自定義 Operator](https://airflow.apache.org/docs/apache-airflow/stable/howto/custom-operator.html)

Operator 路徑：預設在 `{AIRFLOW_HOME}/plugins`

自定義的 Operator 會繼承 BaseOperator，只需要在 Constructor 中定義要傳入的參數，並實作 execute 函式就能完成 Operator 的基礎功能

##### 範例
目標：讓 Operator 可以傳入使用者的名字，並且在執行時紀錄跟使用者打招呼的時間

在 Constructor 中，可使用 Airflow 定義好的裝飾器 `@apply_defaults`；透過 `@apply_defaults` 可幫 Operator 載入 Default Arguments，並存放在 kwargs['params']，讓我們可以在 constructor 中調用。
```python
class GreetOperator(BaseOperator):

    @apply_defaults
    def __init__(
            self,
            name: str,
            *args, **kwargs) -> None:
        super().__init__(*args, **kwargs)
        self.name = name
```

execute 函式帶有 context 變數，context 帶有一些功能，像是 ti 或 task_instance 代表 TaskInstance，透過 TaskInstance 可以跟 XCom 互動

- params
透過 params 可以傳遞自定義的參數進去 Task，在 Task 中，我們可以從 context.params 拿到這些參數
```python
dag = DAG(
    ...,
    params={
        "s3_bucket": "test"
    }
)
```
- Variables
在 WebServer 的 Admin → Variables 頁面看到，使用 key value 形式創造 Variable；可以是 json 的格式，因此可以用一個 Variable 就設定好一個 DAG 所需的全部參數，並在 DAG 裡面，透過 `Variable.get("some key")` 拿到 Variable。
```python
dag = DAG(
    ...,
    params=Variable.get("test", deserialize_json=True)
)
```
- Connections
```python
from airflow.hooks.base_hook import BaseHook
redis_password = BaseHook.get_connection('redis_default').password
```

在 Operator 中，執行結束回傳的值會被記錄在 XCom 的 return_value 裡，可以透過 TaskInstance 的 xcom_push 功能，存入其他的值
```python
    def execute(self, context):
        message = "Hello {}".format(self.name)
        context['ti'].xcom_push(key='time', value=datetime.now().timestamp())
        print(message)
        return message
```

定義 Airflow Plugin，讓我們待會再可以在 DAG 中引入我們的 Operator
```python
class AirflowTestPlugin(AirflowPlugin):
    name = 'greet_operator'
    operators = [GreetOperator]
```

完整 code
```python
from datetime import datetime

from airflow.plugins_manager import AirflowPlugin
from airflow.models.baseoperator import BaseOperator
from airflow.utils.decorators import apply_defaults

class GreetOperator(BaseOperator):

    @apply_defaults
    def __init__(
            self,
            name: str,
            *args, **kwargs) -> None:
        super().__init__(*args, **kwargs)
        self.name = name

    def execute(self, context):
        message = "Hello {}".format(self.name)
        context['ti'].xcom_push(key='time', value=datetime.now().timestamp())
        print(message)
        return message

class AirflowTestPlugin(AirflowPlugin):
    name = 'greet_operator'
    operators = [GreetOperator]
```

### Executor

Airflow Executor [(不同 Executor 比較)](https://docs.astronomer.io/learn/airflow-executors-explained)

- Local Executor：單機版的 Executor，可透過 MultiProcess 同時運行多個 Worker
- Celery Executor：使用 Celery 分散式的 Task Queue 當作 Executor，所以可以在多台電腦同時運行多個 Worker
![](https://i.imgur.com/RtTHUB4.png)
- Kubernetes Executor：透過 Kubernetes API 為每個工作開啟一個 Pod，工作完成後就會砍掉，可以跟據每個工作分配不同的 CPU、memory、設定等，達到資源利用最大化
![](https://i.imgur.com/AB1V8pg.png)
![](https://i.imgur.com/Cznu44B.png)

***這部分目前筆者著墨不多，後續有碰到再寫篇文章說明***

### XCOM
task_1 會將執行結果存到database，task_2 再從database來取得其執行結果，這便是 Xcom 的使用方式(以PythonOperator為例):
1. 設定operator的參數 `provide_context=True`
```python
my_operator = PythonOperator(
    task_id='my_task',
    python_callable=func,
    provide_context=True
)
```
2. 使用方式
- 在 function 使用
```python
def func(**context):
    ti = context['task_instance']
    previous_result = ti.xcom_pull(task_ids='previous_work') # 得到上游的執行結果(key的default值為'return_value')
    new_result = previous_result + 100 # 得到的新結果
    ti.xcom_push(key='xcom_push', value=new_result) # 存入Xcom(key='xcom_push')
    return new_result # return 也會存到Xcom(key='return_value')
```
- 使用其他 Operator
```python
t2 = BashOperator(
    task_id='t2', 
    bash_command='echo "{{ ti.xcom_pull("t1") }}"')
```

> XCom 的所有資料在 pickle 之後會被存到 Airflow 的 Metadata Database，因此不適合交換太大的數據

### 重複利用 Python function

利用一個 Python 函式 process_metadata 專門負責讀 / 寫使用者的閱讀紀錄，兩個 Airflow 工作的 python_callable 都呼叫 process_metadata，利用不同的 op_args 來使用 process_metadata 函式的不同功能：
- get_read_history 負責讀取閱讀紀錄
- update_read_history 負責更新閱讀紀錄

```python
def process_metadata(mode, **context):
    if mode == 'read':
        ...
    elif mode == 'write':
        ...

with DAG('comic_app_v3', default_args=default_args) as dag:

    get_read_history = PythonOperator(
        task_id='get_read_history',
        python_callable=process_metadata,
        op_args=['read'],
        provide_context=True
		)       

    update_read_history = PythonOperator(
        task_id='update_read_history',
        python_callable=process_metadata,
        op_args=['write'],
        provide_context=True
		)
```

### Trigger Rules
可以透過前面 Tasks 的狀態，來判斷後面的 Tasks 要不要執行，讓 Scheduler 判斷要不要將某個 Task 放入排程

- all_success
某個 Task 的上游 Tasks 的狀態都要是成功，才會執行這個 Task
- all_failed
上游 Tasks 的狀態都是失敗時執行，這可以用於處理 exception 狀態
- all_done
只要上游 Tasks 完成，不管它們的狀態是成功、失敗或是 skipped，都會執行
- one_failed
上游 Tasks 其中一個失敗就執行，這個 Trigger Rule 不會等上游 Tasks 都完成才執行，而是只要有失敗就立即執行
- one_success
上游 Tasks 其中一個成功就立即執行
- none_failed
上游 Tasks 的狀態都是成功或是 skipped
- none_skipped
上游 Tasks 的狀態是成功或是失敗時執行

------------------------------------------------------------------------
## Airflow.cfg

使用 docker-compose 建立 airflow 的話，可以跳過這篇，
docker-compose 是透過"環境變數傳遞設定值"的方法設定，即 `AIRFLOW__<...>__<...>: True`

### core
Airflow 設定參數的核心，會記錄 Metadata Database、DAG 資料夾位置、Plugins 資料夾位置等

- dags_folder
存放 DAG 的資料夾，在 local 一般會是 `~/airflow/dags` 對應的絕對路徑
- plugins_folder
存放 Operator 的資料夾，在 local 一般會是 `~/airflow/plugins` 對應的絕對路徑
- sql_alchemy_conn
Database 的連線位置，預設用 SQLite 連到 `~/airflow/airflow.db` 的位置；SQLite 一般只用於開發，到了線上環境將換成其他 Database
- Remote Logging
  - remote_logging
    是否開啟遠端紀錄 Log，目前支援 AWS S3、Google Cloud Storage、Elastic Search
  - remote_log_conn_id
    若要開啟，要記得先到 Connections 頁面創建一個 conn_id
    ```cfg
    Conn Id: s3_conn
    Conn Type: S3
    Extra: {"aws_access_key_id":"your_aws_key_id", "aws_secret_access_key": "your_aws_secret_key"}
    ```
  - remote_base_log_folder
    與 base_log_folder 類似，也就是在遠端 log 要存放的相對位置
- executor
預設的 Executor 是 SequentialExecutor，也可換成 LocalExecutor、CeleryExecutor、KubernetesExecutor等
- load_examples
在線上環境，如果不想看到 Airflow 預設的 Example DAG，可以透過這個參數關掉
```cfg
load_examples = False
```

### cli
透過 cli 跟線上環境溝通

- api_client
1. airflow.api.client.local_client
Airflow 會直接跟設定檔裡設定的 Database 溝通
2. airflow.api.client.json_client
Airflow 會跟 endpoint_url 所設定的 WebServer 溝通
- endpoint_url
Airflow WebServer 的 url

### debug
只有一個設定值 fail_fast
若要啟用，要將 executor 改成 DebugExecutor，這個設定值會讓 DAG 裡面只要有一個 Task 狀態是 failed，整個 DAG 的狀態也會變成 failed

### webserver
- base_url
跟 email 比較有關係，Default Args 裡面的 email_on_failure 可以在 Task 失敗時自動寄信，Airflow 信中的內容會包括 WebServer 的位置，而這個位置就是 base_url
- SSL
可以透過 `web_server_ssl_cert` 及 `web_server_ssl_key` 設定 SSL 憑證
- secret_key
在線上環境，也要記得指定 `secret_key`，因為 Airflow 的 WebServer 是使用 Flask 實現的，Flask 會透過 `secret_key` 來對 cookies 簽名

### smtp
設定 Airflow 寄信的相關設定
```cfg
[smtp]
smtp_host = smtp.gmail.com
smtp_starttls = True
smtp_ssl = False
smtp_user = YOUR_EMAIL_ADDRESS
smtp_password = 16_DIGIT_APP_PASSWORD
smtp_port = 587
smtp_mail_from = YOUR_EMAIL_ADDRESS
```

[GCP SMTP 設定](https://stackoverflow.com/questions/70681089/gcp-command-to-retrieve-an-smtp-password-for-airflow)

------------------------------------------------------------------------
## Command Line Interface (CLI)
若不需要使用 GUI 介面，也可直接使用 CLI，附上 command
### Dags
#### 查詢 dags 列表
```linux
$airflow dags list
```
#### 執行一次測試 dags
```linux
$airflow dags test <DAG_ID> <EXECUTION_TIME>
```
#### 刪除 dags
```linux
$airflow dags delete <DAG_ID>
```
#### 顯示 dags 結構
```linux
$airflow dags show <DAG_ID>
# save
$airflow dags show <DAG_ID> --save <FILE_NAME.png>
```
### Database
#### 初始化資料庫
```linux
$airflow db init
```
#### 查詢資料庫狀態
```linux
$airflow db check
```
#### 更新資料庫
```linux
$airflow db upgrade
```
#### 訪問資料庫
```linux
$airflow db shell
```
### Tasks
#### 查詢 dags 中的 tasks
```linux
$airflow tasks list <DAG_ID>
```
#### 執行 dags 中的其中一個 task 
```linux
$airflow tasks test <DAG_ID> <TASK_ID> <EXECUTION_TIME>
```
### Users
#### 查詢 users
```linux
$airflow users list
```
#### 創建 user
- For Airflow >=2.0.0:
```linux
$airflow users  create --role <user_role> --username <username> --email <email> --firstname <firstname> --lastname <lastname> --password <password>
# role includes admin, user, op, viewer, public
```
- For Airflow <1.10.14:
```linux
$airflow create_user -r <user_role> -u <username> -e <email> -f <firstname> -l <lastname> -p <password>
# role includes admin, user, op, viewer, public
```
#### 刪除 user
```linux
$airflow users delete -u <USERNAME>
```

------------------------------------------------------------------------
## Airflow 的擴充性

1. Airflow 有提供 Operators 就使用 Airflow
2. 如果沒有提供那就透過繼承使用 Hooks 連接外部服務自制 Operators
3. Hooks 沒辦法解決的話則視問題的分類
   - 資料處理或統計可以匯入資料到 SQL/BigQuery 解決
   - 執行指令可以製作 Docker image 透過 DockerOperator/KubernetesPodOperator 執行

------------------------------------------------------------------------
## ref
- [Airflow Documentation](http://apache-airflow-docs.s3-website.eu-central-1.amazonaws.com/index.html)
- [Create username and password Apache Airflow](https://stackoverflow.com/questions/66160780/first-time-login-to-apache-airflow-asks-for-username-and-password-what-is-the-u)
- [How to remove default example dags in airflow](https://stackoverflow.com/questions/43410836/how-to-remove-default-example-dags-in-airflow)
- [使用 docker-compose 建立 airflow](https://dev.to/hyperredstart/airflow-chu-ti-yan-4a6a)
- [資料工程以及 Airflow](https://leemeng.tw/a-story-about-airflow-and-data-engineering-using-how-to-use-python-to-catch-up-with-latest-comics-as-an-example.html)
- [Airflow 動手玩](https://tw.coderbridge.com/series/c012cc1c8f9846359bb9b8940d4c10a8)
- [Airflow 安裝及基礎設定](https://3lexw.medium.com/apache-airflow-2-1-%E5%9F%BA%E7%A4%8E%E6%95%99%E5%AD%B8-2-%E5%AE%89%E8%A3%9D%E5%8F%8A%E5%9F%BA%E7%A4%8E%E8%A8%AD%E5%AE%9A-5622f7cff10d)

