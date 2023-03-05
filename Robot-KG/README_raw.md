#基于ResearchOps平台建设社区问答&资源推荐机器人

#### 介绍
本项目通过使用知识图谱技术，结合ES数据库，neo4j图数据库，实现了社区文档资源推荐的对话机器人，可以根据用户自然语言描述进行推荐对应文档链接。该项目同时包括前端、后端和构建过程三部分代码。

#### 安装教程

1. 进入项目主目录，授予脚本权限:   
   `chmod +x ./env_setup.sh`
2. 通过脚本安装环境:   
   `./env_setup.sh`

#### 使用说明

1. 数据准备  
   (1)准备好需要被搜索的文档，打包放入一个文件夹，将该文件夹命名为自己ES数据库想要创建的索引名称。
   (2)将该文件夹放于路径es_create/file_data/目录下，参考已有数据文件夹格式es_create/file_data/new-open,同时将文件名称与链接对应的同名.csv文件放于同文件夹下，具体格式参考es_create/file_data/文件夹下数据
2. ES数据库创建索引并且导入数据   
    数据准备好后，增加conf/conf.yaml（样例请参考conf/conf_demo.yaml)中对应的ES数据库相关信息为自己数据库，conf的ES数据库信息使用位置为es_create/Index_create.py，运行es_create/create_main.py文件，在data_file文件夹下生成与conf_demo.yaml文件中index_name字段同名json文件，并同时在自己的ES数据库中生成同名索引，利用curl命令将json文件导入自己的ES的数据库。
    ```sh
    # curl命令参考链接为：https://cloud.tencent.com/developer/article/1402401
    # 格式命令如下：
    curl -H 'Content-Type: application/x-ndjson' -XPOST 'ip:9200/索引名称/_doc/_bulk?pretty' --data-binary @文件位置
    ```
3. 模型训练   
   (1)命名实体模型训练(NER)  
    根据自己的部分数据进行标注，标注格式为BIO标注方式。在标注之前需要自行设计好实体类型和相互之间的关系，格式参照data_process/design_data/type.txt和data_process/design_data/relation.csv，训练数据放入Model_train_predict/example/ner/standard/data,参考train.txt,type.txt,valid.txt,运行上级目录下的run.py进行训练（训练前需确保在checkpoints里不可有model.pth同名文件）,可以进入conf文件夹修改参数  
   (2)关系抽取模型训练(RE)  
    根据自己的部分数据进行标注，标注格式参照Model_train_predict/example/re/standard/data下的origin文件夹下的几个csv文件,运行run.py进行训练
4. 模型预测  
   (1)NER:复制所有文档到Model_train_predict/example/ner/standard/data/pre_data_dir/下，运行predict.py，在当前目录生成ner_data.json文件，运行Entity_data.py文件，当前目录生成ner_entity.csv文件  
   (2)复制上述ner_data.json文件到re/standard/目录下，运行re_predict.py，生成neo4j_relation.csv文件,修改re/standard/conf/predict.yaml文件中模型路径为从项目主目录开始的存放路径
5. 导入neo4j数据库  
   将上一步产生的两个csv文件放入到我们neo4j安装目录下面的import目录下，分别作为实体文件和关系文件导入自己的neo4j数据库后端部署:    
   (1)ES后端部署:修改es_search/Search_api.py文件中host、port、auth等信息，在服务器后端运行。   
   (2)neo4j后端部署:修改Search_neo4j/Neo_data_search.py文件中的url、usr、key、table_name等信息，在服务器后端运行。  
   (3)neo4j数据导入请参考以下博客中实体导入和动态关系导入相关内容，链接如下:  
                                   https://blog.csdn.net/TT_QY/article/details/123793457

6. 后端部署   
   (1)ES后端部署:修改es_search/Search_api.py文件中host、port、auth等信息，在服务器后端运行  
   (2)neo4j后端部署:修改Search_neo4j/Neo_data_search.py文件中的url、usr、key、table_name等信息，在服务器后端运行
7. 前端查询  
   修改Send_post.py中的相关参数，修改请求发送目标数据库，运行Search_gui.py启动前端




#### 参与贡献

1.  Fork 本仓库
2.  新建 Feat_xxx 分支
3.  提交代码
4.  新建 Pull Request


#### 特技

1.  使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2.  Gitee 官方博客 [blog.gitee.com](https://blog.gitee.com)
3.  你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解 Gitee 上的优秀开源项目
4.  [GVP](https://gitee.com/gvp) 全称是 Gitee 最有价值开源项目，是综合评定出的优秀开源项目
5.  Gitee 官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6.  Gitee 封面人物是一档用来展示 Gitee 会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)
