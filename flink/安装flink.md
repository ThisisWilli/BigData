# 安装flink

## 在flink官网上下载flink

* 官网上下载flink安装包，在conf文件夹配置jobmanager的地址，以及slave的地址

* 在bash_profile文件夹中配置flink环境变量，配置完source一下

* 运行一下命令, 测试flink是否安装成功

  ```
  flink run examples/batch/WordCount.jar
  ```

  