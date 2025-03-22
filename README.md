

# 分层
## 整体项目结构： 
- backend_xx 后端 
- frontend 前端 
- sqllite_server 服务器

## 后端项目结构
- my_agent 具体实现agent 
- langgraph.json 启动配置

## 后端实现结构
- DatabaseManager.py 数据库管理模块
- DataFormatter.py 数据格式化模块
- LLMManager.py llm管理模块
- SQLAgent.py SQL agent 模块
- State.py  全局状态模块
- WorkflowManger.py 数据流模块


# 实现
## 输出结构化如何实现 ？
- 提示词直接写入结构
- langchain的包JsonOutputParser
```
prompt = ChatPromptTemplate.from_messages([
            ("system", '''You are a data analyst that can help summarize SQL tables and parse user questions about a database. 
Given the question and database schema, identify the relevant tables and columns. 
If the question is not relevant to the database or if there is not enough information to answer the question, set is_relevant to false.

Your response should be in the following JSON format:
{{
    "is_relevant": boolean,
    "relevant_tables": [
        {{
            "table_name": string,
            "columns": [string],
            "noun_columns": [string]
        }}
    ]
}}

The "noun_columns" field should contain only the columns that are relevant to the question and contain nouns or names, for example, the column "Artist name" contains nouns relevant to the question "What are the top selling artists?", but the column "Artist ID" is not relevant because it does not contain a noun. Do not include columns that contain numbers.
'''),
            ("human", "===Database schema:\n{schema}\n\n===User question:\n{question}\n\nIdentify relevant tables and columns:")
        ])

        output_parser = JsonOutputParser()
```


2. 如何transform图配置结构
这个实现有点神
- 直接写函数format ，比如 _format_bar_data等
- 通过大模型进行渲染， 给出例子， 然后让大模型将我给定的数据进行渲染

3. 前端如何实现
- 直接通过词典
- 定义data类型和词典， 然后通过词典key获取
```

export const graphDictionary = {
  bar: {
    component: BarGraph,
    description:
      'Requires data of type BarGraphProps. Best for comparing categorical data or showing changes over time when categories are discrete. Use for questions like "What are the sales figures for each product?" or "How does the population of cities compare?"',
    exampleData: barExampleData,
  },
graphDictionary[graphState.visualization as keyof typeof graphDictionary]
                .component as React.ComponentType<any>,
              {
                data: graphState.formatted_data_for_visualization as InputType,
              },
```
## 其他
1. 命名通常是大写模块名称，要么是manager， 要么是er 作为名词提供服务， 更加容易理解
2. 配置文件通过  _ 分割使用， 让层次更加清晰
3. WorkflowManager依赖SQLAgent和DataFormatter, 设计的很好，并且清晰的表达出来
```
class WorkflowManager:
    def __init__(self):
        self.sql_agent = SQLAgent()
        self.data_formatter = DataFormatter()

```

4. SQLAgent 依赖 DatabaseManager 和 LLMManager
```
class SQLAgent:
    def __init__(self):
        self.db_manager = DatabaseManager()
        self.llm_manager = LLMManager()
```
5. 上面两条我也想到了， 但是我并没有清晰的表达出来， 尤其是3， 在我的表达中就是一个类来实现了， 应该单独拆分出来类，在通过manager进行管理
6. 我在AI上，偷懒太多了，很多还是需要自己实现的
7. 后续需要类似的图，才能进行实现，想清楚哪些需要哪些不需要
8. 刚开始不考虑设计模式， 直接通过if 语句来实现 
9. 接口设计比较简单， 都是基于state进行的


# 图
1. png图
2. 说清楚业务交互流程
3. 图上的模块作为业务模块，必须在项目中，尽可能明显的存在， 比如workflow下面包含sqlagent 和 format data
4. 图先行，没有图不开工
5. 线条表示业务流程


# 问题
1. 目录结构是不是应该更深？
2. 后续更加复杂如何重构 ？