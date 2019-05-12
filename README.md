# hw5-team-19

ISA525700 Computer Vision for Visual Effects<br/>Assignment 5: Generate the multi-view 3D visual effects.<br/>Team 19 
===

# Abstract
本文延續HW4以ORB模型作為影像特徵提取(Feature extration)與尋找配對(Matching)最佳化，進行三種前後影像景動態實驗，觀察3D影像效果。

Keyword: 動態, 3D視覺, ORB, SIFT, SURF.


# Table of Contents


# Introduction
在上次作業中，採用OpenCV執行影像之間的特徵提取與匹配。在本文的實驗中，我們延續此技術生成多視圖3D立體效果，例如運動視差與停止動作效果。本文實作目標為
-自拍多視圖、顯示不同影像間的對齊結果。
-產生多視圖3D效果。
-創意發想方式，輔以後製軟體進形影像處理增強影像效果。
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


## SIFT與SURF

在比較模型方法上，我們採用SIFT與SURF[2]，

在SIFT模型中，Lowe此採用高斯差分近似高斯的發普拉斯轉換，尋找尺度空間。SURF更進一步，用Box Filter趨近LoG。如下圖表示，這種近似法最大優點為借助積分影像可輕易運算出Box Filter的卷積，同時亦可針對不同尺度平行完成。此外SURF仰賴Hessian矩陣的尺度和位置的行列式。
 
 

對於方向分配，SURF在水平與垂直方向係採用小波響應，對於大小在6s範圍內的鄰近區域，足夠高斯量使用它。然後繪製下圖中的空間中，透過計算角度60度的滑動定向窗口內所有響應總和來估計主導定向。有趣的是，小波響應可以容易使用積分影像找到任何影像縮放。對於不需要旋轉影像處理，無須找到影像定位，加速影像處理過程。SURF提供Upright-SURF與U-SURF兩項功能，可以提高速度，並且可以達到±15。OpenCV則支援兩者，取決於旗標(flag)與直立(upright)。如果為0，則計算方向。如果為1，則不計算方向並且速度更快。

 



## Motion Stills – Create beautiful GIFs from Live Photos

Motion Stills[3]是一種虛擬相機鏡頭技術，將攝影穩定技術，讓背景凝結成靜態方式，前景動態，後以製作gif檔，影片變得生動。

將流水剪輯與組合，產生身臨其境於水流體驗。可以使用演算法以線性程式計算虛擬鏡頭路徑，路鏡經過優化，重新製作影片，產生背景靜止不動與電影平移消除抖動影片效果。這項技術將挑戰手機影片如同資料中心運算速度。透過時間採樣、運動參數，以及Google Research的custom linear solver與GLOP等技術，實現40倍的加速。低分辨率扭曲紋理技術執行GPU渲染，可以加速與節省儲存空間，如同影像遊戲。

循環播放，短片非常適合建立循環播放，方法是確定最佳起始與終點。為保持循環播放背景穩定，將前景元素遮住重要的部份。方法為時間一致下，將運動矢量分為離出前後景，運動模型由簡單到複雜。


# Example#1. Motion parallax

## Concept
	 

## Experiment


## Analysis 


## Discussion


# Example#2. Stop motion

## Concept


## xperiment


## Analysis 

## discussion


# Example#3. Live photo

## Concept


## Experiment


## Analysis 

## discussion


# Conclusion

Item	Ex1	Ex2	Ex3
時間(Time)			
照明(Illumination)			
縮放比例(Scale)			
轉譯(Translation)			
模糊(Blur)			
旋轉(Rotation)			
仿射(Affine)			


# Reference
- [1] ORB, https://docs.opencv.org/3.1.0/d1/d89/tutorial_py_orb.html
- [2] SIFT與SURF, https://docs.opencv.org/3.4.0/df/dd2/tutorial_py_surf_intro.html
- [3] Motion Stills – Create beautiful GIFs from Live Photos, https://ai.googleblog.com/2016/06/motion-stills-create-beautiful-gifs.html


