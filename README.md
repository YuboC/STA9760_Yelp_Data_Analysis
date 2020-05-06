[![GitHub stars](https://img.shields.io/github/stars/YuboC/STA9760_Yelp_Data_Analysis.svg?style=flat&label=Star)](https://github.com/YuboC/STA9760_Yelp_Data_Analysis/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/YuboC/STA9760_Yelp_Data_Analysis.svg?style=flat&label=Fork)](https://github.com/YuboC/STA9760_Yelp_Data_Analysis/fork)
[![GitHub watchers](https://img.shields.io/github/watchers/YuboC/STA9760_Yelp_Data_Analysis.svg?style=flat&label=Watch)](https://github.com/YuboC/STA9760_Yelp_Data_Analysis/watchers)


# Analyzing 10Gb of Yelp Reviews Data

````
For this project, I will be tasked with provisioning a Spark Cluster on 
AWS EMR for loading and running some analysis on Yelp’s Reviews and Businesses 
dataset (about 10gb) from Kaggle. I will run my analysis via Jupyter 
Notebook and the expected output artifact is a .ipynb file  
````  
![](https://github.com/YuboC/STA9760_Yelp_Data_Analysis/blob/master/assets/2.PNG)
![](https://github.com/YuboC/STA9760_Yelp_Data_Analysis/blob/master/assets/3.PNG)
## Part I: Installation and Initial Setup
In this portion, I will import the necessary dependencies (pandas and matplotlib) and load my dataset as a pyspark dataframe.

## Part II:  Analyzing Categories
For this part, you will take a stab at denormalizing the categories that are associated with each business (there may be more than one, presented as a string of comma separated identifiers) and then running some basic analysis on the result.

## Part III: Do Yelp Reviews Skew Negative?
For this next part, you will attempt to answer the question: are the (written) reviews generally more pessimistic or more optimistic as compared to the overall business rating. There are some required questions you must answer (see the analysis.ipynb file) which is the bare minimum. But, feel free to have fun with it and take your analysis as far as you’d like. Any additional work you do will be counted for up to 5 points of extra credit on your project grade, capped at 105.

## Part IV: Should the Elite be Trusted? (Or, some other analysis of your choice)
For this final part you may choose to either answer the question posed or explore the data in some other manner of your own choosing. The only requirements are:
You must leverage the users dataset provided
You must have at least one data visualization as part of your analysis

## Note: Spark on Docker
- **EC2 Instance With Internet Connection**
````
First, we begin by configuring and building an EC2 instance that can 
connect to the internet. By “connect to the internet”, we mean that 
the EC2’s public IP address (the thing we use to SSH into the devices) 
must be accessible via the web browser.
````
- **Properly Configuring EC2**
  - **create a Ubuntu server of Amazon Machine Image**
  - **at** `Step 6. Configure Security Group` **Add Rule:**  
   `| Custom TCP | TCP | 8888 | Custom 0.0.0.0/0 | describe: allow ipw4`  
   `| Custom TCP | TCP | 8888 | Custom ::/0      | describe: allow ipw6`
  - **launch**
- **SSH into EC2 and Starting a Webserver**
````java
> chmod 0400 jupyter.pem
> ls -ahl jupyter.pem
> ssh -i ./jupyter.pem ubuntu@Public_IP
````  
- **Test the connection:**
  - create a `HTML` file:
    ````java
    :~$ cat >> index.html
    <h1>Hello, World!</h1>
    ^C
    ````
  - start a simple **web server** in instance:
    ````java
    :~$ python3 -m http.server 8888
    ````
  - visit the site in browser:  
  ` http://Public_IP:8888`
  - back to terminal and hit **Ctrl+C** to kill the server
  
- **Install Docker on EC2 to run Jupyter Notebook and Apache Spark**
  - install `Docker`:  
  `:~$ sudo apt-get update`  
  `:~$ sudo apt-get remove docker docker-engine docker.io`  
  `:~$ sudo at install docker.io -y`     the **-y** flag will not ask you for permissions halfway through the install  
  `:~$ sudo systemctl start docker`  
  `:~$ sudo systemctl enable docker`
  - test:
  ` sudo docker`
  - install `Docker-Compose`:  
  ````java
  :~$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o 
  /usr/local/bin/docker-compose
  ````
  - set permissions:
  `:~$ sudo chmod +x /usr/local/bin/docker-compose`
  - test:
  `:~$ sudo docker-compose --version`
- **Run Jupyter/pyspark-notebook Image**:  
  - pull image from dockerhub
    `:~$ docker run -it -p 8888:8888 jupyter/pyspark-notebook`
  - visit jupyter notebook in browser: `http://YOUR_EC2_PUBLIC_IP:8888?token=[TOKEN_FROM_YOUR_TERMINAL_SCREEN]`  
- **Tweaking Docker Settings**:  
  - stop the container with `Ctrl + C` and run:  
  `:~$ sudo docker run -d -v $(pwd):/home/jovyan/data -it -p 8888:8888 jupyter/pyspark-notebook`  
  `:~$ -d`: running docker in detached mode in the background of terminal  
  `:~$ -v $(pwd):/home/jovyan/data`: the **WORKDIR** of this container is **/home/joyyan**  
  mount our current working directory to it. This allows us to share datasets with the container  
  meaning we can access the datasets inside Jupyter.  
  - access logs:  
  `:~$ sudo docker logs funny_chatterjee`  
  - use **curl** to download the dataset:  
  `:~$ curl https://s3.amazonaws.com/tripdata/201908-citibike-tripdata.csv.zip >> 201908-citibike-tripdata.csv.zip`  
  - unzip the file:  
  `:~$ sudo apt install unzip` & `:~$ unzip 201908-citibike-tripdata.csv.zip`
  - test the file:  
  `:~$ head 201908-citibike-tripdata.csv`
## Spark on EMR
- **use S3 Bucket to store the dataset and access it via EMR**:
  - click **Create Cluster** in EMR  
  - go to **Advanced Options**
    - Step 1: Software Configuration: `Realease: emr-5.29.0`  
      - [x] Hadoop 2.8.5  
      - [x] Livy 0.6.0  
      - [x] Hive 2.3.6  
      - [x] Hue 4.4.0  
      - [x] Spark 2.4.4
    - Step 2: Hardware : make sure **EC2 Subset** is configured correctly that instance types are **m5.xlarge**  
    - Step 3: General Cluster Settings  
    - Step 4: Security : set the key pair and create cluster  
    - Step 5: Configuring Jupyter Notebook and Open it  
    - Step 6: Running Spark Cluster Tasks via Jupyter Notebook  
    - Kernel --> Change Kernel --> Pyspark
      ````java
      %%info
      sc.list_packages()   # Viewing and installing packages on the cluster via the notebook
      sc.install_pypi_package("pandas==1.0.3")  # install packages
      sc.install_Pypi_package("matplotlib==3.2.1")
      ````
  - Loading data into **S3** : rely on HDFS to access the data
    - in S3 click **Create bucket**  
    - in the bucket **upload** the files
  - Loading data to EMR from S3
    ````python
    df = spark.read.csv('s3://Bucket_name/file_name.csv', header='true', inferSchema = True)
    df = spark.read.csv('s3://Bucket_name/*.csv', header = 'true', inferSchema = True)
    
    #Total columns
    print(f'Total Columns: {len(df.dtypes)}')
    print(f'Total Rows: {df.count():,}')
    df.printSchema()
    
    df.createOrReplaceTempView('Citibike')
    
    sqldf = spark.sql(
    '''
    SELECT `column_name`, COUNT ( `column_name`) as new_name
    FROM Table_name
    GROUP BY `column_name`
    ORDER BY column_name2 DESC
    LIMIT 5
    '''
    )
    
    sqldf.toPandas().plot.bar()
    ````

