
## Installation & Setting up Airflow w/ Docker.
1. Create a src folder
2. Install all Airflow dependencies viz. apache-airflow, docker, docker compose, etc
3. Create a docker-compose file
    cmd: {curl -LfO 'https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml'}. 
    It would generate a new file: {docker-compose.yaml}
4. Review the docker-compose file
5. Create 3 folder: dags, logs and plugins
6. Run the command: {echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=0" >.env} to create a .env file
7. To start the database, it is incharge of the running all the databases: {docker-compose up [application-name]}
    {docker-compose up airflow-init} 
8. Finally, hit {docker-compose up}, runs all the services, specified in the docker-compse.yaml file.
9. To check if the containers are up and running, {docker ps}. triggerer, worker, scheduler, webserver, flower, postgres, redis
10. Run {localhost:8080}. Login with default credentials {airflow | airflow}. Later On, you can setup a profile with username and password.

*** Misc

docker-compose up - Creates all the containers specified in the .yaml file
docker-compose down - Removes the containers



## Airflow CLI
1. To know the version of the container, {docker exec [CONTAINER_ID] airflow version}

## Airflow API
1. curl -X GET "http://localhost:8080/api/v1/dags"
    Hitting this command, the title would be "Unathorized"
    To fix this, go to your docker-compose.yaml file
    In environments, add another env variable,
    {AIRFLOW__API__AUTH_BACKEND: 'airflow.api.auth.backend.basic_auth'}
2. Run docker-compose down && docker-compose up
3. Inorder to get the list of all dags, un {curl -X GET --user "airflow:airflow" "http://localhost:8080/api/v1/dags" }[airflow: airflow corresponds to the username:password]



## Coding your First DAG - Data Pipeline

1. In /dags folder, create a new .py file
    eg. my_dag.py
2. Whenever we write a DAG, do not forget to import the DAG class.
    {from airflow import DAG}. This enables the interpreter to recognize that this file is a DAG.
3. Second important import is datetime.
    {from datetime import datetime}. Enables the functionality to ensure the DAG has a start and end date.
4. Create an instance of the DAG class.
    {
        with DAG("dag_id", 
                    start_date=datetime(2021, 1, 1), schedule_interval="@daily", 
                    dag_run="", 
                    catchup=False ) as dag: 
    }
        {dag_id} : Name of the DAG
        {start_date} : From the start date, the DAG is being scheduled
        {scheduled_interval} : Defines the frequency, interval at which your DAG is triggered.
        {dag_run}:  Instance of the dag running at a given date 
        {catchup}:False : The latest non trigeered dag run will be automatically triggered by Airflow. Avoids non-triggered dag runs between the start date and the current date.
        
        eg. Every 10 mins. We can define it in CRON expression.
        eg. Daily: @daily (Its a preset)
5. Whenever you want to write a task, import the subsequent operator.
6. Follow the syntax for creating a task,
    task_name = Operator(
        task_id="unique identifier",
        python_callable=name_of_the_python_function
        
    )
7. Create a python function that performs certain operation.
8. Important: Define the sequence/ order of the task.
    eg [A, B] >> C >> [D, E]
9. Run {docker-compose down && docker-compose up} and run all the services from Docker.
10. Hurray ! The dag appears on the UI.
11. The DAG automatically runs. Examine the logs if needed. The logs also appear in the /logs folder that was manually created.


## Steps to setup an Airflow DAG
There are only 5 steps (as mentioned above) you need to remember to write an Airflow DAG or workflow:
    1. Importing the 
        from datetime import timedelta, datetime
        import airflow
        from airflow import DAG
        from airflow.operators.bash_operator import BashOperator, etc
    2. Default Arguements
        default args = {
            'owner': 'airflow',
            'start_date': airflow.utils.dates.days_ago(2),
            'email': [xyz@example.com],
            'email_on_failure': False,
            'email_on_retry': False,
            ## If a task fails, retry it once after waiting at lesat 5 minutes
            'retries': 1,
            'retry_delay': timedelta(minutes=5),
            etc, etc
        }
    3. Instantiate a DAG
        dag = DAG(
                    "dag_id", 
                    start_date=datetime(2021, 1, 1), schedule_interval="@daily/@weekly/@monthly/@yearly/", 
                    dag_run="", 
                    catchup=False
                    etc, etc 
        )
    4. Tasks
    5. Setting up Dependencies




## XComms - Cross Communication Message
A mechanism to share data between your dag.


## Airflow : 
A worklow management system which is used to programmatically author, schedule and monitor workflows.

A DAG is a collection of all the task you want to run, organized in a way that reflects their relationships and dependencies.

It is a graph has no cycles and the data in each node flows forward in only one direction.

A graph has 2 main parts:
    1. Vertices(nodes) where the data is stored
        eg Task A, Task B
    2. Edges(connections) which connect the nodes
        eg A >> B 


## Task
Task: Once the operator is instantiated, it is referred to as a "task". An operator describes a singel task in a workflow.. Instantiaitng a task requires providing a unique {task_id} and DAG container
     eg. 

     t1 = BashOperator(
         task_id='Task 1',
         bash_command='date',
         dag=dag
     )

Each task describes what you are trying to do, and DAG is just a container that is ued to organize task and set their execution context.

## Operator:
Operators:  determine what actually gets done.

A task is created by instantianting an Operator class
    eg task = PythonOperator(
        .....
        .....
    )
An Operator defines the nature of this Task and how should it be executed. When an Operator is instantiated, this task becomes a node in your DAG.

Categories:
1. Sensors -> waiting for data to arrive at a defined location
    A certain type of operator that will keep running until a certain criteria is met. 
        eg. include waiting for a certain time, external file, or upstream data source.
    Sensor operator inherit of BaseSensorOperator (BaseOperator being the superclass of BaseSensorOperator)
    They are basically long running task

2. Operators: or Action Operators -> perform an action
    Triggers a certain action
    BashOperator, PythonOperator, EmailOperator...etc
    eg. run a bash command - BashOperator
        execute a python function - PythonOperator
    Can be fouund in Airflow Official repo : airflow/contrib

3. Transfers -> that move data from one system to another
    Moves data from one location to another
    eg MysqltoHiveTransfer: Moves data from MySql to Hive.



## Dag Runs & TaskInstances 
execution_time: begins at the DAG's start_date and repeats every schedule_interval. A DagRun is simply a DAG that has a specific execution time.

TaskInstances are the task belongs to that DagRuns.

DagRun and TaskInstances is associated with an entry in Airflow's metadata database that logs their state
    queued
    running
    failed
    skipped
    up for retry


## Variables

Variables are key-value stores in Airflow's metadata database.


It is mostly used yo store static values like:
    config variables
    configuration file
    list of tables
    list of IDs to dynamically generate tasks from