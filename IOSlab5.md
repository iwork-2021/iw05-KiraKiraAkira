### IOS智能应用开发lab5: Object Detection App.

202220013徐简

#### 实验结果：

基于模板工程(ObjectDetection)，运用CoreML开发一个利用TinyYOLO进行目标检测的iOS App。

功能要求如下：

1. 通过TuriCreate，基于snacks数据集训练目标检测模型
2. 运用摄像头功能，利用训练好的模型进行实时的目标检测，支持多目标检测
3. 在屏幕上展示神经网络模型的分类结果和目标框(bounding box)

#### 实现过程

1. 在服务器配置训练环境

- 在conda环境中，并不能通过conda install来实现turicreate的安装。
- 而在使用pip install后，发现会安装在base环境中，解决方案是找到当前环境的pip程序所在地址，指定使用当前环境的pip

- 按照apple_turicreate_guide安装了tensor flow gpu，并在训练脚本中指定了enable_gpu，但是还是在用CPU训练。Anyway，经过了30个小时，还是完成了训练。

2. 训练脚本

- 首先完成数据集的加载
- YOLO模型的输入是一张图片，输入是一系列bounding box的坐标和label
- 因此csv文件中，图片文件名，会对应多行（有多个bounding box）,遍历即可

```python
img_annotations = []
for i in rows.itertuples():
	img_annotations.append({"coordinates": {"height": (i[5] - i[4]) * img_height, 
                                                    "width": (i[3] - i[2]) * img_width, 
                                                    "x": (i[3] + i[2]) / 2 * img_width, 
                                                    "y": (i[5] + i[4]) / 2 * img_height}, 
                                    "label": i[6]})
```

3.导入Xcode工程

- 基本步骤和之前一样
- 完成在show函数中可视化模型的结果(计算bounding Box View.show所需要的参数即可)

```swift
func show(predictions: [VNRecognizedObjectObservation]) {
    //process the results, call show function in BoundingBoxView
    for boxViewCount in 0..<boundingBoxViews.count {
        guard boxViewCount < predictions.count else {
            boundingBoxViews[boxViewCount].hide()
            return
        }
				if prediction.labels[0].confidence>0.8{
        let prediction = predictions[boxViewCount]
        let width = view.bounds.width
        let height = width * 1280 / 720
        let offsetY = (view.bounds.height - height) / 2
        let scale = CGAffineTransform.identity.scaledBy(x: width, y: height)
        let transform = CGAffineTransform(scaleX: 1, y: -1).translatedBy(x: 0, y: -height - offsetY)
        let rect = prediction.boundingBox.applying(scale).applying(transform)

        let label: String = prediction.labels[0].identifier + "\(prediction.labels[0].confidence * 100)%"

        let color = colors[prediction.labels[0].identifier] ?? UIColor.yellow
        boundingBoxViews[boxViewCount].show(frame: rect, label: label, color: color)
        }
    }
}

```



#### 功能展示

见录屏

#### 感想与体会

- 学习了如何在服务器上配置环境与训练模型
- GPU YYDS
- 期待以后，可以训练自己的模型（bushi