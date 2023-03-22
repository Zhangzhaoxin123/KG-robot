# 基于ResearchOps平台建设社区问答&资源推荐机器人

#### 介绍

本项目通过使用知识图谱技术，结合ES数据库，neo4j图数据库，实现了社区文档资源推荐的对话机器人，可以根据用户自然语言描述进行推荐对应文档链接。该项目同时包括前端、后端和构建过程三部分代码。

#### 安装/启动教程

1. 克隆项目
   `git clone repo_url`

2. 启动ElasticSearch

   ```sh
   cd tools/elasticsearch # 在项目路径下，进入ElasticSearch Dockerfile地址
   
   docker build -t user_name/elasticsearch:7.14.0 ./ # 构建镜像, 自动安装ik分词器
   
   # 启动容器，修改local_port端口信息，修改docker的user_name信息
   docker run -d --name elasticsearch \
   -p local_port_1:9200 -p local_port_2:9300 \
   -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
   user_name/elasticsearch:7.14.0
   ```

3. 启动neo4j

   ```sh
   cd tools/neo4j # 在项目路径下，进入neo4j Dockerfile地址
   
   docker build -t user_name/neo4j:4.1.12 ./ # 构建镜像, 安装jar
   
   # 启动容器，修改local_port端口信息，修改local_path为正确地址，修改docker的user_name信息，可在NEO4J_AUTH中更换用户名和密码
   docker run -d --name neo4j \
   -p local_port_1:7474 -p local_port_2:7687 \
   -v local_path/resource/neo4j/data:/data \
   -v local_path/resource/neo4j/logs:/logs \
   -v local_path/resource/neo4j/conf:/var/lib/neo4j/conf \
   -v local_path/resource/neo4j/import:/var/lib/neo4j/import \
   --env NEO4J_AUTH=neo4j/password \
   user_name/neo4j:4.1.12
   ```

#### 使用说明

1. 挂载文件说明（创建文件夹resource, 挂载文件样例请参考[这里](./sample_resource)）

   ```shell
   # 创建文件夹resource - 训练、预测结合
   resource/conf => conf/ # 输入, 配置文件
   resource/raw_data_input => es_create/file_data # 输入, 原始文件和记录每个文件链接
   resource/train_ner_input => /kg/Model_train_predict/example/ner/standard/data # 输入, 训练ner模型所需的数据, 可使用pip install label-studio构造输入数据
   resource/train_re_input=> /kg/Model_train_predict/example/re/standard/data/origin # 输入, 训练re模型所需的数据, 可使用pip install label-studio构造输入数据
   resource/neo4j/import => /kg/Model_train_predict/example/predict_output # 输出, 实体和关系最终的csv文件
   ```

2. 知识图谱社区问答机器人主服务 - 训练预测结合（脚本详细介绍请参考[这里](./train_predict.sh)，若想了解train和predict的详细信息，请参考[这里](./README_train_predict.md)）：

- 脚本内容

  - 调用train.sh
  - 调用predict.sh

- 配置输入

  ```txt
  resource/conf：需用户自行根据Elasticsearch、NEO4J和服务端口等情况自行配置。
  其余输入文件已默认配置好，在不改变数据源的情况下，无需更换。
  ```

- 启动docker

  ```shell
  docker build -t user_name/kg_qabot:0.1 ./ # 进入项目主目录，通过Dockerfile创建镜像
  
  # 启动容器，修改local_port端口信息，修改local_path为正确地址，修改docker的user_name信息
  docker run -it --name kg_qabot \
  -p local_port:8000 \
  -v local_path/resource/conf:/kg/conf \
  -v local_path/resource/raw_data_input:/kg/es_create/file_data \
  -v local_path/resource/train_ner_input:/kg/Model_train_predict/example/ner/standard/data \
  -v local_path/resource/train_re_input:/kg/Model_train_predict/example/re/standard/data/origin \
  -v local_path/resource/neo4j/import:/kg/Model_train_predict/example/predict_output \
  user_name/kg_qabot:0.1 \
  /bin/bash
  ```

- 运行docker

  ```shell
  # 确保已经进入该容器内部，若未进入请执行下列命令
  docker exec -it kg_qabot /bin/bash
  # 在进入该容器内部后，授予train_predict.sh脚本权限并运行
  chmod +x ./train_predict.sh
  ./train_predict.sh

  # 如果已经训练和预测过，本次只想启动后端查询服务，不需要运行训练模型和预测新实体和关系识别，则可直接使用该service_run.sh
  chmod +x ./service_run.sh
  ./service_run.sh
  ```

- 获取训练、预测后生成的csv文件

  ```txt
  在脚本predict.sh运行结束后，会自动将NER和RE的预训练模型model_ner.pth和model_re.pth预测的结果csv保存至local_path/resource/neo4j/import
  ```

3. 导入neo4j数据库

  ```shell
  # 在浏览器中打开neo4j，将上一步产生的两个csv文件，分别作为实体文件和关系文件导入自己的neo4j数据库

  # 写入实体
  LOAD CSV WITH HEADERS  FROM "file:///var/lib/neo4j/import/ner_entity.csv" AS line
  MERGE (z:实体{name:line.name,name_type:line.name_type})
  return *;

  # 写入实体关系
  LOAD CSV WITH HEADERS FROM "file:///var/lib/neo4j/import/neo4j_relation.csv" AS line  
  MATCH (from:实体{name:line.head}),(to:实体{name:line.tail})  
  call apoc.merge.relationship(from, line.relation, {}, {}, to) yield rel
  return *;

  # 若需删除该实体，请使用下列命令
  MATCH (r:实体)
  DETACH DELETE r
  ```

4. 后端请求

   向**知识图谱社区问答机器人主服务**，或**predict.sh部分**所在**ip**和**local_port**发送**post**请求，输入自然语言，获取查询信息。
