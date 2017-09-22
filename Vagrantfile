# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-17.04"
  config.vm.provider "virtualbox" do |vb|
      vb.memory = "9216"
  end

  config.vm.provision "dependencies", type: "shell", inline: <<-DEPENDENCIES
    apt-get update
    apt-get install -y -qq zip unzip curl bzip2 python-dev build-essential git libssl1.0.0 libssl-dev software-properties-common debconf-utils python-software-properties
  DEPENDENCIES

  config.vm.provision "java", type: "shell", privileged: false, inline: <<-JAVA
    sudo add-apt-repository -y ppa:webupd8team/java
    sudo apt-get -qq update
    echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections
    sudo apt-get install -y -qq oracle-java8-installer oracle-java8-set-default

    export JAVA_HOME=/usr/lib/jvm/java-8-oracle
    echo "export JAVA_HOME=/usr/lib/jvm/java-8-oracle" | tee -a $HOME/.bash_profile
  JAVA

  config.vm.provision "miniconda", type: "shell", privileged: false, inline: <<-MINICONDA
    echo "curl -sLko /tmp/Miniconda3-latest-Linux-x86_64.sh https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh"
    curl -sLko /tmp/Miniconda3-latest-Linux-x86_64.sh https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
    chmod +x /tmp/Miniconda3-latest-Linux-x86_64.sh
    /tmp/Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/anaconda

    export PATH=$HOME/anaconda/bin:$PATH
    echo 'export PATH=$HOME/anaconda/bin:$PATH' | tee -a $HOME/.bash_profile
  MINICONDA

  config.vm.provision "repo", type: "shell", privileged:false, inline: <<-REPO
    cd $HOME
    git clone https://github.com/rjurney/Agile_Data_Code_2
    cd $HOME/Agile_Data_Code_2
    export PROJECT_HOME=$HOME/Agile_Data_Code_2
    echo "export PROJECT_HOME=$HOME/Agile_Data_Code_2" | tee -a $HOME/.bash_profile
    conda install python=3.6.2
    conda install iso8601 numpy scipy scikit-learn matplotlib ipython jupyter
    pip install -qqq bs4 Flask beautifulsoup4 airflow frozendict geopy kafka-python py4j pymongo pyelasticsearch requests selenium tabulate tldextract wikipedia findspark

    cd $HOME
  REPO

  config.vm.provision "hadoop", type: "shell", privileged: false, inline: <<-HADOOP
    echo "curl -sLko /tmp/hadoop-2.7.4.tar.gz http://apache.osuosl.org/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz"
    curl -sLko /tmp/hadoop-2.7.4.tar.gz http://apache.osuosl.org/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz
    mkdir -p $HOME/hadoop
    cd $HOME/
    tar xzf /tmp/hadoop-2.7.4.tar.gz -C hadoop --strip-components=1

    echo '# Hadoop environment setup' | tee -a $HOME/.bash_profile
    export HADOOP_HOME=$HOME/hadoop
    echo 'export HADOOP_HOME=$HOME/hadoop' | tee -a $HOME/.bash_profile
    export PATH=$PATH:$HADOOP_HOME/bin
    echo 'export PATH=$PATH:$HADOOP_HOME/bin' | tee -a $HOME/.bash_profile
    export HADOOP_CLASSPATH=$(hadoop classpath)
    echo 'export HADOOP_CLASSPATH=$(hadoop classpath)' | tee -a $HOME/.bash_profile
    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
    echo 'export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop' | tee -a $HOME/.bash_profile
  HADOOP

  config.vm.provision "spark", type: "shell", privileged: false, inline: <<-SPARK
    echo "curl -sLko /tmp/spark-2.2.0-bin-without-hadoop.tgz http://d3kbcqa49mib13.cloudfront.net/spark-2.2.0-bin-without-hadoop.tgz"
    curl -sLko /tmp/spark-2.2.0-bin-without-hadoop.tgz http://d3kbcqa49mib13.cloudfront.net/spark-2.2.0-bin-without-hadoop.tgz
    mkdir -p $HOME/spark
    cd $HOME
    tar xzf /tmp/spark-2.2.0-bin-without-hadoop.tgz -C spark --strip-components=1

    echo "" >> $HOME/.bash_profile
    echo "# Spark environment setup" | tee -a $HOME/.bash_profile
    export SPARK_HOME=$HOME/spark
    echo 'export SPARK_HOME=$HOME/spark' | tee -a $HOME/.bash_profile
    export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop/
    echo 'export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop/' | tee -a $HOME/.bash_profile
    export SPARK_DIST_CLASSPATH=`$HADOOP_HOME/bin/hadoop classpath`
    echo 'export SPARK_DIST_CLASSPATH=`$HADOOP_HOME/bin/hadoop classpath`' | tee -a $HOME/.bash_profile
    export PATH=$PATH:$SPARK_HOME/bin
    echo 'export PATH=$PATH:$SPARK_HOME/bin' | tee -a $HOME/.bash_profile

    # Have to set spark.io.compression.codec in Spark local mode
    cp $HOME/spark/conf/spark-defaults.conf.template $HOME/spark/conf/spark-defaults.conf
    echo 'spark.io.compression.codec org.apache.spark.io.SnappyCompressionCodec' | tee -a $HOME/spark/conf/spark-defaults.conf

    # Give Spark 8GB of RAM, used Python3
    echo "spark.driver.memory 9g" | tee -a $SPARK_HOME/conf/spark-defaults.conf
    echo "PYSPARK_PYTHON=python3" | tee -a $SPARK_HOME/conf/spark-env.sh
    echo "PYSPARK_DRIVER_PYTHON=python3" | tee -a $SPARK_HOME/conf/spark-env.sh

    # Setup log4j config to reduce logging output
    cp $SPARK_HOME/conf/log4j.properties.template $SPARK_HOME/conf/log4j.properties
    sed -i 's/INFO/ERROR/g' $SPARK_HOME/conf/log4j.properties
  SPARK

  config.vm.provision "mongo", type: "shell", privileged: false, inline: <<-MONGO
    #echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
    #sudo apt-get update
    #sudo apt-get install -y -qq --allow-unauthenticated mongodb-org-shell mongodb-org-server mongodb-org-mongos mongodb-org-tools mongodb-org
    sudo apt-get install -y -qq mongodb
    sudo mkdir -p /data/db
    sudo setfacl -m user:mongodb:rwx /data/db
    sudo setfacl -m group:mongodb:rwx /data/db

    # run MongoDB as daemon
    #sudo /usr/bin/mongod --fork --logpath /var/log/mongodb.log

    # Get the MongoDB Java Driver
    echo "curl -sLko $HOME/Agile_Data_Code_2/lib/mongo-java-driver-3.4.2.jar http://central.maven.org/maven2/org/mongodb/mongo-java-driver/3.4.2/mongo-java-driver-3.4.2.jar"
    curl -sLko $HOME/Agile_Data_Code_2/lib/mongo-java-driver-3.4.2.jar http://central.maven.org/maven2/org/mongodb/mongo-java-driver/3.4.2/mongo-java-driver-3.4.2.jar

    # Install the mongo-hadoop project in the mongo-hadoop directory in the root of our project.
    echo "curl -sLko /tmp/mongo-hadoop-r2.0.2.tar.gz https://github.com/mongodb/mongo-hadoop/archive/r2.0.2.tar.gz"
    curl -sLko /tmp/mongo-hadoop-r2.0.2.tar.gz https://github.com/mongodb/mongo-hadoop/archive/r2.0.2.tar.gz
    mkdir $HOME/mongo-hadoop
    cd $HOME
    tar xzf /tmp/mongo-hadoop-r2.0.2.tar.gz -C mongo-hadoop --strip-components=1
    rm -rf /tmp/mongo-hadoop-r2.0.2.tar.gz

    # Now build the mongo-hadoop-spark jars
    cd $HOME/mongo-hadoop
    ./gradlew jar
    cp $HOME/mongo-hadoop/spark/build/libs/mongo-hadoop-spark-*.jar $HOME/Agile_Data_Code_2/lib/
    cp $HOME/mongo-hadoop/build/libs/mongo-hadoop-*.jar $HOME/Agile_Data_Code_2/lib/
    cd $HOME

    # Now build the pymongo_spark package
    cd $HOME/mongo-hadoop/spark/src/main/python
    python setup.py install
    cp $HOME/mongo-hadoop/spark/src/main/python/pymongo_spark.py $HOME/Agile_Data_Code_2/lib/
    export PYTHONPATH=$PYTHONPATH:$PROJECT_HOME/lib
    echo 'export PYTHONPATH=$PYTHONPATH:$PROJECT_HOME/lib' | sudo tee -a $HOME/.bash_profile
    cd $HOME

    rm -rf $HOME/mongo-hadoop
  MONGO

  config.vm.provision "elasticsearch", type: "shell", privileged: false, inline: <<-ELASTICSEARCH
    echo "curl -sLko /tmp/elasticsearch-5.6.1.tar.gz https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.1.tar.gz"
    curl -sLko /tmp/elasticsearch-5.6.1.tar.gz https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.1.tar.gz
    mkdir $HOME/elasticsearch
    cd $HOME
    tar xzf /tmp/elasticsearch-5.6.1.tar.gz -C elasticsearch --strip-components=1
    mkdir -p $HOME/elasticsearch/logs

    # Run elasticsearch
    #$HOME/elasticsearch/bin/elasticsearch -d # re-run if you shutdown your computer

    # Install Elasticsearch for Hadoop
    echo "curl -sLko /tmp/elasticsearch-hadoop-5.6.1.zip http://download.elastic.co/hadoop/elasticsearch-hadoop-5.6.1.zip"
    curl -sLko /tmp/elasticsearch-hadoop-5.6.1.zip http://download.elastic.co/hadoop/elasticsearch-hadoop-5.6.1.zip
    unzip -qq /tmp/elasticsearch-hadoop-5.6.1.zip
    mv $HOME/elasticsearch-hadoop-5.6.1 $HOME/elasticsearch-hadoop
    cp $HOME/elasticsearch-hadoop/dist/elasticsearch-hadoop-5.6.1.jar $HOME/Agile_Data_Code_2/lib/
    cp $HOME/elasticsearch-hadoop/dist/elasticsearch-spark-20_2.11-5.6.1.jar $HOME/Agile_Data_Code_2/lib/
    echo "spark.speculation false" | sudo tee -a $HOME/spark/conf/spark-defaults.conf
    rm -f /tmp/elasticsearch-hadoop-5.6.1.zip
    rm -rf $HOME/elasticsearch-hadoop/conf/spark-defaults.conf
  ELASTICSEARCH

  config.vm.provision "spark-jars", type: "shell", privileged: false, inline: <<-SPARKJARS
    # Install and add snappy-java and lzo-java to our classpath below via spark.jars
    cd $HOME/Agile_Data_Code_2
    echo "curl -sLko lib/snappy-java-1.1.2.6.jar http://central.maven.org/maven2/org/xerial/snappy/snappy-java/1.1.2.6/snappy-java-1.1.2.6.jar"
    curl -sLko lib/snappy-java-1.1.2.6.jar http://central.maven.org/maven2/org/xerial/snappy/snappy-java/1.1.2.6/snappy-java-1.1.2.6.jar
    echo "curl -sLko lib/lzo-hadoop-1.0.5.jar http://central.maven.org/maven2/org/anarres/lzo/lzo-hadoop/1.0.0/lzo-hadoop-1.0.0.jar"
    curl -sLko lib/lzo-hadoop-1.0.5.jar http://central.maven.org/maven2/org/anarres/lzo/lzo-hadoop/1.0.0/lzo-hadoop-1.0.0.jar
    cd $HOME

    # Set the spark.jars path
    echo "spark.jars $HOME/Agile_Data_Code_2/lib/mongo-hadoop-spark-2.0.2.jar,$HOME/Agile_Data_Code_2/lib/mongo-java-driver-3.4.2.jar,$HOME/Agile_Data_Code_2/lib/mongo-hadoop-2.0.2.jar,$HOME/Agile_Data_Code_2/lib/elasticsearch-spark-20_2.11-5.6.1.jar,$HOME/Agile_Data_Code_2/lib/snappy-java-1.1.2.6.jar,$HOME/Agile_Data_Code_2/lib/lzo-hadoop-1.0.5.jar" | sudo tee -a $HOME/spark/conf/spark-defaults.conf
  SPARKJARS

  config.vm.provision "kafka", type: "shell", privileged: false, inline: <<-KAFKA
    echo "curl -sLko /tmp/kafka_2.11-0.11.0.1.tgz http://www-us.apache.org/dist/kafka/0.11.0.1/kafka_2.11-0.11.0.1.tgz"
    curl -sLko /tmp/kafka_2.11-0.11.0.1.tgz http://www-us.apache.org/dist/kafka/0.11.0.1/kafka_2.11-0.11.0.1.tgz
    mkdir -p $HOME/kafka
    cd $HOME/
    tar xzf /tmp/kafka_2.11-0.11.0.1.tgz -C kafka --strip-components=1 && rm -f /tmp/kafka_2.11-0.11.0.1.tgz
    rm -f /tmp/kafka_2.11-0.11.0.1.tgz

    # Set the log dir to kafka/logs
    sed -i '/log.dirs=\/tmp\/kafka-logs/c\log.dirs=logs' $HOME/kafka/config/server.properties

    # Run zookeeper (which kafka depends on), then Kafka
    #$HOME/kafka/bin/zookeeper-server-start.sh -daemon $HOME/kafka/config/zookeeper.properties
    #$HOME/kafka/bin/kafka-server-start.sh -daemon $HOME/kafka/config/server.properties
  KAFKA

  config.vm.provision "airflow", type: "shell", privileged: false, inline: <<-AIRFLOW
    pip install -qqq airflow[hive]

    mkdir $HOME/airflow
    mkdir $HOME/airflow/dags
    mkdir $HOME/airflow/logs
    mkdir $HOME/airflow/plugins

    airflow initdb
    #airflow webserver -D &
    #airflow scheduler -D &
  AIRFLOW

  config.vm.provision "zeppelin", type: "shell", privileged: false, inline: <<-ZEPPELIN
    # Install Apache Zeppelin
    echo "curl -sLko /tmp/zeppelin-0.7.2-bin-all.tgz http://www-us.apache.org/dist/zeppelin/zeppelin-0.7.2/zeppelin-0.7.2-bin-all.tgz"
    curl -sLko /tmp/zeppelin-0.7.2-bin-all.tgz http://www-us.apache.org/dist/zeppelin/zeppelin-0.7.2/zeppelin-0.7.2-bin-all.tgz
    mkdir zeppelin
    tar xzf /tmp/zeppelin-0.7.2-bin-all.tgz -C zeppelin --strip-components=1

    # Configure Zeppelin
    cp zeppelin/conf/zeppelin-env.sh.template zeppelin/conf/zeppelin-env.sh
    echo "export SPARK_HOME=$PROJECT_HOME/spark" >> zeppelin/conf/zeppelin-env.sh
    echo "export SPARK_MASTER=local" >> zeppelin/conf/zeppelin-env.sh
    echo "export SPARK_CLASSPATH=" >> zeppelin/conf/zeppelin-env.sh
  ZEPPELIN

  config.vm.provision "jupyter", type: "shell", privileged: false, inline: <<-JUPYTER
    # Jupyter server setup
    echo "Jupyter"
    jupyter notebook --generate-config
    mkdir -p $HOME/.jupyter
    cp $HOME/Agile_Data_Code_2/jupyter_notebook_config.py $HOME/.jupyter/
    mkdir $HOME/certs
    openssl req -x509 -nodes -days 365 -newkey rsa:1024 -subj "/C=US" -keyout $HOME/certs/mycert.pem -out $HOME/certs/mycert.pem

    #jupyter notebook --ip=0.0.0.0 --no-browser &
  JUPYTER

  config.vm.provision "cleanup", type: "shell", inline: <<-CLEANUP
    apt-get clean
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
    echo "DONE!"
  CLEANUP

  config.vm.provision "start", type: "shell", run: "always", inline: <<-START
    echo "Starting Mongodb"
    #sudo /usr/bin/mongod --fork --logpath /var/log/mongodb.log

    echo "Starting Elasticsearch"
    $HOME/elasticsearch/bin/elasticsearch -d # re-run if you shutdown your computer

    echo "Starting Kafka"
    $HOME/kafka/bin/zookeeper-server-start.sh -daemon $HOME/kafka/config/zookeeper.properties
    $HOME/kafka/bin/kafka-server-start.sh -daemon $HOME/kafka/config/server.properties

    echo "Starting airflow"
    airflow webserver -D &
    airflow scheduler -D &

    echo "Starting Jupyter"
    jupyter notebook --ip=0.0.0.0 --no-browser &
  START

  # Map hosts ports 5000,4567,8888 to local port 5000,4567,8888
  config.vm.network :forwarded_port, guest: 5000, host: 5000
  config.vm.network :forwarded_port, guest: 4567, host: 4567
  config.vm.network :forwarded_port, guest: 8888, host: 8888
  config.vm.network :forwarded_port, guest: 8080, host: 8080
end
