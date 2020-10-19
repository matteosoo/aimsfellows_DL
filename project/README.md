# Project
Goal: 使用自行蒐集(爬下)的dataset, 進行quick draw model的CNN classifier應用。
- Assignment: 根據TAs所給之ptt熱門文章標題分配，訓練一個繁體中文手寫辨識模型，並於11/11上課日進行簡單的網頁成果展示。
- Score assessment: 
    - - [x] 30%: 能夠將model訓練並存為(.h5)模型
    - - [x] 20%: 能夠從(.h5)模型轉為tensorflow.js支持格式並在網頁demo
    - - [x] 10%: 精準度至少85%(?)以上
    - - [ ] 40%: written report
        - 2頁(±0.5頁)a4, 形式不拘
        - 除了模型架構和精準度等客觀評比外，可以寫下project或課堂所學、嘗試改進部分、所遇到問題)
        - 2020/11/11 (Wed.) 23:59 written report deadline, 請上傳到[此連結](https://drive.google.com/drive/folders/1DXw0QjBwAjUhwJH7aiJ03cdqHct3nhsJ?usp=sharing).

    
## Pipeline
![](https://i.imgur.com/Y3y0lMR.png)

## sketcher project 補充說明
### main.js 中修正路徑
main.js中必須修正2個部分才能成功將你訓練的新模型放入(請找“**//**”註解部分)
1. load the class names
```javascript
/*
load the class names
*/
async function loadDict() {
    if (mode == 'ar')
        loc = 'model2/class_names_ar.txt' // 這段可以不理他 
    else
        loc = 'model3/class_names.txt' // 你可以把新分的classed放在名為model3資料夾下

    await $.ajax({
        url: loc,
        dataType: 'text',
    }).done(success);
}
```
2. load the model
```javascript
/*
load the model
*/
async function start(cur_mode) {
    //arabic or english
    mode = cur_mode

    //load the model
    model = await tf.loadLayersModel('model3/model.json') // 把新model放在名為model3的資料夾下

    //warm up
    model.predict(tf.zeros([1, 28, 28, 1]))

    //allow drawing on the canvas
    allowDrawing()

    //load the class names
    await loadDict()
}
```
### main.js修正shape of input data
由於這個專案的資料是28\*28，要讓我們新訓練的資料用300\*300在js上也必須同時調整 
> [name=TAs] 可以直接control (or command) F 搜尋28比較快找到

1. 第10行, 28\*28改300\*300
```javascript
/*
preprocess the data
*/
function preprocess(imgData) {
    return tf.tidy(() => {
        //convert to a tensor
        let tensor = tf.browser.fromPixels(imgData, numChannels = 1)

        //resize
        const resized = tf.image.resizeBilinear(tensor, [28, 28]).toFloat()

        //normalize
        const offset = tf.scalar(255.0);
        const normalized = tf.scalar(1.0).sub(resized.div(offset));

        //We add a dimension to get a batch shape
        const batched = normalized.expandDims(0)
        return batched
    })
}
```
2. 第12行, 28\*28改300\*300
```javascript
/*
load the model
*/
async function start(cur_mode) {
    //arabic or english
    mode = cur_mode

    //load the model
    model = await tf.loadLayersModel('model3/model.json')

    //warm up
    model.predict(tf.zeros([1, 28, 28, 1]))

    //allow drawing on the canvas
    allowDrawing()

    //load the class names
    await loadDict()
}
```

## FYI
- Quick draw project reference
    - step by step: https://medium.com/tensorflow/train-on-google-colab-and-run-on-the-browser-a-case-study-8a45f9b1474e
    - github code: https://github.com/zaidalyafeai/zaidalyafeai.github.io/tree/master/sketcher
    - demo page: https://zaidalyafeai.github.io/sketcher/\:https://www.tensorflow.org/js/tutorials/setup

###### tags: `AIMS`
