### 自动化测试核心指标

- 测试用例数：越多越好
    - 编写、调试效率
    - 管理、维护成本
    - 学习成本、上手难度
- 执行频率：越快越好
    - 运行便捷性，各种环境下运行
    - 运行效率（分布式、并行）
- 运行成功率：越高越好
    - 框架稳定性
    - 用例稳定性
        - 数据依赖
        - hook 机制（setup、teardown）



### 借鉴优秀开源项目

![image-20210908140712711](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210908140712711.png)

![image-20210908140221266](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210908140221266.png)

字段对标 requests，保持一致，便于理解

![image-20210908140609364](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210908140609364.png)



### 如何减少用户的维护工作量？

- 约定大于配置，保障测试用例风格统一
- 接口测试三要素
    - 参数构造
    - 发起请求，获取响应
    - 校验结果
- YAML/Json 模版
- 框架中融入自动化最佳工程实践 flask & django
- 测试用例分层机制，减少重复性描述

![image-20210908141123137](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210908141123137.png)



### 如何尽可能降低测试用例的编写步骤

![image-20210908141434110](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20210908141434110.png)

