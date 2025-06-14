# News-Classify-in-NaiveBayes
 
## 目录  
1. [项目概述](#项目概述)  
2. [文件结构](#文件结构)  
3. [文件详细说明](#文件详细说明)  
   - 3.1 [app.py](#1-apppy)  
   - 3.2 [train_model.py](#2-train_modelpy)  
   - 3.3 [toutiao_cat_data.txt](#3-toutiao_cat_datatxt)  
   - 3.4 [templates/index.html](#4-templatesindexhtml)  
   - 3.5 [static/script.js](#5-staticscriptjs)  
   - 3.6 [news_classifier_model.joblib 和 tfidf_vectorizer.joblib](#6-news_classifier_modeljoblib-和-tfidf_vectorizerjoblib)  
   - 3.7 [confusion_matrix.png](#7-confusion_matrixpng)  
4. [总结](#总结)  


## 项目概述  
本项目旨在实现一个新闻文本分类系统，包括数据预处理、模型训练、模型评估和 Web 应用部署。以下是各个文件的具体作用介绍。  


## 文件结构  
```
web_app/
├── app.py                      # Flask Web 应用主程序
├── train_model.py              # 模型训练脚本
├── toutiao_cat_data.txt        # 原始新闻数据文件
├── templates/                  # HTML 模板目录
│   └── index.html             # 主页模板
├── static/                     # 静态资源目录
│   └── script.js              # JavaScript 脚本
├── news_classifier_model.joblib  # 训练好的模型文件
└── tfidf_vectorizer.joblib     # TF-IDF 向量化器文件
```


## 文件详细说明  


### 1. app.py  
**作用**：Flask Web 应用主程序，负责加载模型和向量化器，并提供预测接口。  

**内容要点**：  
- **加载模型和向量化器**：  
  ```python
  model = load('news_classifier_model.joblib')
  vectorizer = load('tfidf_vectorizer.joblib')
  ```
- **定义预测函数**：  
  ```python
  @app.route('/predict', methods=['POST'])
  def predict():
      # 处理输入文本并进行预测
      ...
  ```
- **返回预测结果**：  
  ```python
  return jsonify({
      "prediction": prediction,
      "history": history
  })
  ```


### 2. train_model.py  
**作用**：模型训练脚本，用于从原始数据中提取特征并训练分类模型。  

**内容要点**：  
- **读取数据**：  
  ```python
  def load_text_data(file_path):
      data = []
      with open(file_path, 'r', encoding='utf-8') as f:
          for line in f:
              if line.strip():
                  parts = line.strip().split('_!_')
                  if len(parts) >= 5:
                      data.append(parts)
      df = pd.DataFrame(data, columns=['ID', 'Category Code', 'News Category', 'Title', 'Tags'])
      return df
  ```
- **数据预处理**：  
  ```python
  X = df['Title']
  y = df['News Category']
  ```
- **TF-IDF 向量化**：  
  ```python
  vectorizer = TfidfVectorizer(max_features=5000)
  X_vec = vectorizer.fit_transform(X_tokenized)
  ```
- **模型训练与保存**：  
  ```python
  model = MultinomialNB()
  model.fit(X_train, y_train)

  joblib.dump(model, 'news_classifier_model.joblib')
  joblib.dump(vectorizer, 'tfidf_vectorizer.joblib')
  ```


### 3. toutiao_cat_data.txt  
**作用**：原始新闻数据文件，包含新闻的标题、类别等信息。

来源：https://github.com/BenDerPan/toutiao-text-classfication-dataset  

**内容示例**：  
```
6551700932705387022_!_101_!_news_culture_!_京城最值得你来场文化之旅的博物馆_!_保利集团,马未都,中国科学技术馆,博物馆,新中国
6552368441838272771_!_101_!_news_culture_!_发酵床的垫料种类有哪些？哪种更好？_!_
6552407965343678723_!_101_!_news_culture_!_上联：黄山黄河黄皮肤黄土高原。怎么对下联？_!_
```


### 4. templates/index.html  
**作用**：Flask Web 应用的主页模板，提供用户界面以输入新闻标题并显示预测结果。  

**内容要点**：  
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>新闻分类器</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans SC', sans-serif;
            background-color: #f9f9f9;
            margin: 0;
            padding: 0;
        }
        ...
    </style>
</head>
<body>
    ...
</body>
</html>
```


### 5. static/script.js  
**作用**：JavaScript 脚本，处理表单提交和 AJAX 请求，实现与后端的交互。  

**内容要点**：  
```javascript
document.addEventListener("DOMContentLoaded", function () {
    const form = document.getElementById("news-form");
    const input = document.getElementById("news-input");
    const resultDiv = document.getElementById("result");
    const clearBtn = document.getElementById("clear-btn");
    const historyList = document.getElementById("history-list");

    // 提交表单（AJAX）
    form.addEventListener("submit", function (e) {
        e.preventDefault();
        ...
    });
});
```


### 6. news_classifier_model.joblib 和 tfidf_vectorizer.joblib  
**作用**：  
- `news_classifier_model.joblib`：保存训练好的分类模型。  
- `tfidf_vectorizer.joblib`：保存 TF-IDF 向量化器，用于将文本转换为特征向量。  


### 7. confusion_matrix.png  
**作用**：混淆矩阵图，展示模型在测试集上的分类效果。  

![混淆矩阵](confusion_matrix.png)

**内容说明**：  
- **横轴（Predicted Label）**：模型预测的标签。  
- **纵轴（True Label）**：实际的标签。  
- **颜色深浅**：表示每个单元格的数值大小，颜色越深表示该类别的样本数越多。  


## 总结  
本项目通过上述文件协同工作，实现了从数据预处理、模型训练到 Web 应用部署的完整流程。具体步骤如下：  

1. **数据预处理**：`train_model.py` 读取 `toutiao_cat_data.txt` 并进行分词、向量化等操作。  
2. **模型训练与保存**：`train_model.py` 使用朴素贝叶斯算法训练模型，并保存模型和向量化器。  
3. **Web 应用开发**：`app.py` 加载模型和向量化器，提供预测接口；`templates/index.html` 和 `static/script.js` 实现用户界面和交互功能。
