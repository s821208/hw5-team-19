# hw5-team-19

ISA525700 Computer Vision for Visual Effects<br/>Assignment 5: Generate the multi-view 3D visual effects.<br/>Team 19 
===

# Abstract
本文延續HW4以ORB模型作為影像特徵提取(Feature extration)與尋找配對(Matching)最佳化，進行三種前後影像多視景動靜態實驗，觀察3D影像效果。

Keyword: 多視景, ORB


# Table of Contents
1. [Introduction](#Introduction)
2. [Method](#Method)
3. [Result](#Result) 
4. [Conclusion](#Conclusion)
5. [Reference](#Reference)

# Introduction
在上次作業中，採用OpenCV執行影像之間的特徵提取與匹配。在本文的實驗中，我們延續此技術生成多視圖3D立體效果，例如運動視差與停止動作效果。本文實作目標為
- 自拍多視圖、顯示不同影像間的對齊結果。
- 產生多視圖3D效果。
- 創意發想方式，輔以後製軟體進形影像處理增強影像效果。
本實驗在影像之「辨識、旋轉、照明與合理縮放比例」等為基礎條件下，進行評估效能。本實驗提出可視化呈現成果，顯示辨識到影像特徵與其它影像最佳配對，可解決程式上的問題。並採用ORB模型技術取樣特徵檔，並比較不同結果。我們在室內外拍攝一系列影像，在雲端採用GPU運算執行ORB模型，顯示兩個影像間的特徵辨識與配對結果，並且執行影像對齊，依作業目的產生前後景變化3D視覺效果。


# Mothod

## ORB

ORB[1]是計算SIFT和SURF很好的替代方案，因為SIFT和SURF以申請專利，且Cost與matching是主要專利。

ORB基本上是FAST關鍵辨識和Brief descriptor的結合，具備許多改善，以增強效能。首先，它使用FAST搜尋關鍵點，然後應用Harris角點測量來搜尋其中的前N個點。它還使用金字塔方式來產生多尺度特徵。但有一個問題是，FAST不計算方向。那麼影像旋轉不變性方面呢？作者提出了以下改善。

計算貼片強度與加權質心，位於中心的角落。從該角點到質心的矢量方向給出方向。為了改善旋轉不變性，用x和y計算矩，其應該在半徑為r的圓形區域中，其中r是此貼片的大小。

ORB使用簡易descriptor，但Brief在輪換方面表現不佳。因此，ORB所做的是根據關鍵點的方向「引導」Brief。對於位置（xi，yi）處的n個二進制測試的任何影像特徵集，定義2×n矩陣，其包含這些像素的坐標。然後使用貼片的方向θ，找到其旋轉矩陣並旋轉S以獲得轉向（旋轉）版本Sθ。

ORB將角度離散為2π/ 30（12度）增量，並構建預先計算的簡要模式的查詢表。只要關鍵點方向θ在視圖之間是相同的，將使用正確的點集Sθ來計算其descriptor。

BRIEF具有一個重要特性，即每個位特徵具有較大的方差，平均值接近0.5。但是一旦它沿著關鍵點方向定向，它就會失去這個屬性並變得更加分散。高差異使得特徵更具辨別力，因為它對輸入有不同的響應。另一個理想的特性是使測試不相關，因為每次測試都會對結果產生影響。為了解決所有這些問題，ORB在所有可能的二進制測試中執行貪婪式搜索，以找到具有高方差和意味著近似0.5的那些，以及不相關的。結果稱為rBRIEF。

對於描述符匹配(descriptor matching)，採用改進傳統LSH的多探測LSH。使得ORB比SURF快的多，SIFT和ORB描述符比SURF更好。結論為ORB是用於全景拼接等的低功率設備的不錯選擇。


## Motion Stills – Create beautiful GIFs from Live Photos

Motion Stills[2]是一種虛擬相機鏡頭技術，將攝影穩定技術，讓背景凝結成靜態方式，前景動態，後以製作gif檔，影片變得生動。

將流水剪輯與組合，產生身臨其境於水流體驗。可以使用演算法以線性程式計算虛擬鏡頭路徑，路鏡經過優化，重新製作影片，產生背景靜止不動與電影平移消除抖動影片效果。這項技術將挑戰手機影片如同資料中心運算速度。透過時間採樣、運動參數，以及Google Research的custom linear solver與GLOP等技術，實現40倍的加速。低分辨率扭曲紋理技術執行GPU渲染，可以加速與節省儲存空間，如同影像遊戲。

循環播放，短片非常適合建立循環播放，方法是確定最佳起始與終點。為保持循環播放背景穩定，將前景元素遮住重要的部份。方法為時間一致下，將運動矢量分為離出前後景，運動模型由簡單到複雜。

# Result

## Example 1:Motion parallax

### Concept

實驗1為探討運動視差(Motion parallax)，產生3D視覺效果。前景靜態，背景動態實驗，觀察對於電腦視覺3D效果是否顯著影響。

### Experiment

我們在戶外一處景處最佳景點，拍攝花圃中，風動視景作為背景。前景以知名人物小小兵為主角，嘗試建立3D動態視景圖。如下圖，以ORB找出最佳Matching，在室外光亮處，Matching效果非常佳。但小小兵照明亮度非常好，Smart的ORB辨識到小小兵的眼睛處，為最佳Matching，同時應為最佳旋動軸心。

|ORB|![](https://i.imgur.com/hXBbUjx.png)

經實驗後，確認如下圖３張影格，以最少資源與最完美呈現。

|Frame 1|Frame 2|Frame 3|
|--|--|--|
|![](https://i.imgur.com/rsQEqUz.jpg)|![](https://i.imgur.com/TNqeqPz.jpg)|![](https://i.imgur.com/RnTogFg.jpg)|


經ORB確認好最佳Matchig與最佳旋動軸心後，如下三張影格。
以軸心為背景作動態選旋動，產生最佳3D立體動態效果。但是如下圖所示，小小兵卻呈現漂浮於空中。
![](https://i.imgur.com/OZz0xHp.gif)

經過多次實驗後，決定增加背景靜態部份。
![](https://i.imgur.com/kDuz5rv.gif)


### Analysis 
實驗一共實驗兩種，第一為前景靜態不動，背景動態風動，產生運動視差。第二為深入探討背景之動態與靜態同時呈現，是否有不同效果。

### Discussion
經過觀察實驗結果、討論與他人客觀分享評比後，背景動態與前景靜態，真的產生視覺的運動視差(Motion parallax)達到3D立體效果完美呈現。


## Example 2-1:Stop motion

### Concept
實驗2-1，目的探討視覺暫留(Stop motion)，產生3D視覺效果。本次實驗共計三項:

- 1B_2_1F:背景旋動對單一前景物(Object)，是否對３Ｄ視覺效果有顯著影響。 。 
- 1B_2_2F:背景旋動對雙前景物(Object)，模擬多前景物，是否對３Ｄ視覺效果有顯著影響。 。 
- 1B_2_2F:背景旋動對雙前景物(Object)，增加中視景，觀察空間中移動與自旋融合運動模型，是否對３Ｄ視覺效果有顯著影響。 


### Experiment

我們在NTHU校園內一角，拍攝投影影布為背景。前景以知名人物小小兵為主角，嘗試建立3D動態視景圖。如下圖，以ORB找出最佳Matching，小小兵照明亮度非常好，Smart的ORB辨識到小小兵的眼睛與身體處，為最佳Matching，同時應為最佳旋動軸心。
|ORB|![](https://i.imgur.com/lS1gSlV.png)

經實驗後，確認如下圖５張影格，以最少資源與最完美呈現。
|Frame 1|Frame 2|Frame 3|
|--|--|--|
|![](https://i.imgur.com/0sQ7Exk.jpg)|![](https://i.imgur.com/UcZ8a0V.jpg)|![](https://i.imgur.com/zIWFIEQ.jpg)|

|Frame 4|Frame 5|
|--|--|
|![](https://i.imgur.com/0YdGfAD.jpg)|![](https://i.imgur.com/7pRSV7M.jpg)|

這是我們製作單背景對單前景物的影片。

|1B_2_1F|
|--|
|![](https://i.imgur.com/D9g3Bnk.gif)|

這是我們製作單背景對雙前景物的影片。

|1B_2_2F|
|--|
|![](https://i.imgur.com/Dt61S7f.gif)|

這是我們製作單背景對雙前景物，以及增加中視景的影片。中視景為酷酷的足球，它打中小小兵，搞笑一下！也觀察３Ｄ立體效果。

|1B_2_2F|
|--|
|![](https://i.imgur.com/cbBvrea.gif)|

### Analysis 
本次實驗進行三項後，研究發現如下：
- 發現增加前景物數目，對３Ｄ立體效果顯著。
- 發現增加光環，對３Ｄ立體效果顯著。
- 發現背景黑色，對於背景旋動效果非常顯著！
- 發現背景物，面積越大，圖形運算效能資源需求越大，需要ＧＰＵ運算支援。
- 發現前景越清晰，背景物越模糊，對於視覺暫留之３Ｄ立體效果顯著。

### discussion
經過如上結果分析，可得小結：
- 前景物增加，運動模型越複雜，３Ｄ立體效果呈現正顯著。
- 增加光環，並調節尺寸下，３Ｄ立體效果呈現正顯著。
- 背景照明光度越暗，對於視覺暫留之３Ｄ立體效果顯著。
- 背景物之面積越大，３Ｄ運算效能需求越高，是否有演算法模型可以最佳化處理。
- 前景清晰，背景模糊，對於視覺暫留之３Ｄ立體效果顯著。
如上多項珍貴的研究發現，為本次實驗重要心血，分享給各位。


## Example 2-2:Stop motion
### Concept
實驗2-2為延續Stop motion研究。本次實驗為在室外之前景與背景旋動，中視景揮動之運動模型，觀察對產生3D視覺效果是否顯著。

### Experiment
我們在一處湖畔景處取景為本實驗背景，前景以知名人物小小兵為主角，嘗試建立3D動態視景圖。如下圖，以ORB找出最佳Matching，小小兵照明亮度非常好，Smart的ORB辨識到小小兵的頭部處與背景的落地窗框架，為最佳Matching，同時應為最佳旋動軸心。

![](https://i.imgur.com/YvIznsg.png)

經實驗後，確認如下圖３張影格，以最少資源與最完美呈現。

|Frame 1|Frame 2|Frame 3|
|--|--|--|
|![](https://i.imgur.com/50WxRU8.jpg)|![](https://i.imgur.com/m3MiYPG.jpg)|![](https://i.imgur.com/Lvugu1m.jpg)|

影片是前景與背景同步連動之模型，配合鋁管的揮動。
![](https://i.imgur.com/4VLXL0j.gif)


### Analysis 
本次實驗之研究發現如下：

- 背景深度越高，對３Ｄ立體效果越顯著。
- 前景物越清晰，對３Ｄ立體效果越顯著。
- 前景物鋁棒揮動，長短不一時，對３Ｄ立體效果越顯著。

### discussion
經過如上結果分析，可得小結：

- 背景深度越高，確實對３Ｄ立體效果越顯著。
- 前景物越清晰，確實對３Ｄ立體效果越顯著。
- 前景物之尺寸不同時，確實對３Ｄ立體效果越顯著。

如上多項珍貴的研究發現，為本次實驗重要心血，分享給各位。

## Example 3_1:Live photo

### Concept

實驗3-1，Live photo是製作動態桌布與拍攝酷炫動態照片的影像科技，研究探討是否對產生3D視覺效果顯著。我們以動態背景與背景物與否，觀察3D視覺效果。

### Experiment
我們在一處湖畔景處取景為本實驗背景，前景以情侶為男女主角，嘗試建立3D動態視景圖。如下圖，以ORB找出最佳Matching，背景照明亮度非常好，Smart的ORB辨識到多處背景物，如樹與落地燈，為最佳Matching。
![](https://i.imgur.com/y7s1R3j.png)

經實驗後，確認如下圖3張影格，以最少資源與最完美呈現。
|Frame 1|Frame 2|Frame 3|
|--|--|--|
|![](https://i.imgur.com/zJ5WFWp.jpg)|![](https://i.imgur.com/6Dcs43x.jpg)|![](https://i.imgur.com/znGc4be.jpg)|

這是我們製作靜態背景對雙前景物，主題為什麼風把我們頭髮吹亂了!惡搞情侶頭髮，希望呈現娛樂效果，也觀察３Ｄ立體效果。

![](https://i.imgur.com/yXHcp26.gif)

### Analysis
由於本實驗3-1為背景不動，經過ORB的Matching確認，疊圖對焦效果絕佳。經觀察後，所得研究發現:

- 中視景使用最少運算效能，呈現最佳3D效果。
- 前後景凝結靜態，傳達出近中遠，因此呈現絕佳電腦視覺3D效果。

### discussion
本實驗後，所得結果如下:
- 中視景使用最少運算效能資源，呈現3D視覺效果顯著。
- 前後景凝結靜態，傳達出近中遠，呈現3D視覺效果顯著。

## Example 3_2:Live photo

### Concept
實驗3-2，延續Live photo科技，探討前中後三視景連動下，對於生3D視覺效果是否顯著?

### Experiment

本次實驗素材以海岸小港口為背景，前景為正妹為主角，搭配背景海水波浪與長榮客機，建立三視景連動的3D立體視覺效果。如下圖，以ORB找出最佳Matching，背景照明亮度非常好，Smart的ORB辨識到多處背景物，如女主角的腳與港口地板，為最佳Matching。

![](https://i.imgur.com/foWszh2.png)

經實驗後，確認如下圖3張影格，以最少資源與最完美呈現。

|Frame 1|Frame 2|Frame 3|
|--|--|--|
|![](https://i.imgur.com/6BmVEAo.jpg)|![](https://i.imgur.com/WIia2Ne.jpg)|![](https://i.imgur.com/8iid5QO.jpg)|

下列影片是海景波浪與美少女長髮風吹飄逸。
![](https://i.imgur.com/R7jujKp.gif)

下列影片是加入飛機飛過上空。
![](https://i.imgur.com/SvS5bSr.gif)

### Analysis 
由於本實驗3-2之兩項小實驗為:
- 在前景、背景連動下，觀察3D呈現效果。
- 在前景、背景與中視景三者連動下，觀察3D呈現效果。。

經過ORB的Matching確認，疊圖對焦效果絕佳。經觀察後，所得研究發現:
- 在前景、背景連動下，因前景與背景距離為同區域，呈現3D效果不顯著。
- 在前景、背景與中視景三者連動下，因產生視差，呈現絕佳電腦視覺3D效果。

### discussion
本實驗後，所得結果如下:
- 在前景、背景連動下，呈現3D視覺效果不顯著。
- 在前景、背景與中視景三者連動下，傳達出近遠的視差，呈現3D視覺效果顯著。

# Conclusion
本次實驗後，提出結論如下:
- 本文在ORB模型對應Motion parallax、Stop motion、Live photo等三項研究方法，觀察得出視覺效果評比如下:

|Item|Motion parallax|Stop motion|Live photo|
|---|---|---|---|
|照明(Illumination)|高|低|高|				
|縮放比例(Scale)|無縮放比例|高|低|			
|轉譯(Translation)|高|低|高|			
|模糊(Blur)|高|低|中|			
|旋轉(Rotation)|無旋轉|高|中|			
|仿射(Affine)|低|高|中|	

- 本文經過實驗後，提出一種多視軸建立3D視覺運動模型最佳化，期以解決在影像中，前景、中視景、背景的多物體空間真實呈現問題。

# Reference
- [1] ORB, https://docs.opencv.org/3.1.0/d1/d89/tutorial_py_orb.html
- [2] Motion Stills – Create beautiful GIFs from Live Photos, https://ai.googleblog.com/2016/06/motion-stills-create-beautiful-gifs.html
