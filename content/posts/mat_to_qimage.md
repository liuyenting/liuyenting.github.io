---
title: "Mat 轉 QImage"
date: 2016-01-01T10:47:21+08:00
categories:
  - "Language"
tags:
  - "C/C++"
---
在嘗試把 Mat 印到 OpenGL 的元件上面時，通常會使用原生的影像元件進行操作而不再是原始的像素資料（`cv::Mat`）。也因為這樣，有了轉換函式的需求 :p
<!--more-->

在 Stack Overflow 上面逛的時候，[有人](http://stackoverflow.com/questions/5026965/how-to-convert-an-opencv-cvmat-to-qimage/8866986#8866986)早就處理好了這件事情。
```cpp
QImage Mat2QImage(const cv::Mat3b &src) {
    QImage dest(src.cols, src.rows, QImage::Format_ARGB32);
    for (int y = 0; y < src.rows; ++y) {
        const cv::Vec3b *srcrow = src[y];
        QRgb *destrow = (QRgb*)dest.scanLine(y);
        for (int x = 0; x < src.cols; ++x) {
            destrow[x] = qRgba(srcrow[x][2], srcrow[x][1], srcrow[x][0], 255);
        }
    }
    return dest;
}
```
不過討論串裡面其實有很多方法，我最後選擇了這個。

手邊在處理的影像，像素的格式是 RGB888，也就是一個像素由 24-bit 構成，三原色各佔用 8-bit 的空間。如果直接使用 `QImage` 的建構式創造出新的 `QImage`，由於 bits 排列的方式不同於 `imread` 在 Mat 當中的排列方式，會產生亂掉的圖像（應該直接說雜訊也可以⋯⋯）

這邊使用 `Vec3b`，一次抽出來該像素三個維度當中的資料（R、G、B 串接在一起，整份資料成為一個二維的巨大陣列），然後拆散到三個元素中（當作 vector 當中的不同 element）。最後，再逐步把像素用 qRGB 建構成 `QImage` 開心的格式、塞回去。

文件當中說 `cv::Mat` 在操作上，會盡可能使用指標來處理資料。這份 snippet 當中，最有可能影響效能的地方在
```cpp
QRgb *destrow = (QRgb*)dest.scanLine(y);
```
如果沒有使用指標來抽取 Mat 的內部資料結構的資料，將會在 loop-through 所有像素時，產生不必要的 copy。

誠如作者所說，要把它改給不同影像格式也很輕鬆，只要修改
```cpp
QImage dest(src.cols, src.rows, QImage::Format_ARGB32);
```
與
```cpp
destrow[x] = qRgba(srcrow[x][2], srcrow[x][1], srcrow[x][0], 255);
```
讓程式針對輸入的影像格式即可。以這份 snippet 為例，新的 QImage 的像素格式為 ARGB，其中 alpha 因為用不到所以被寫死為 0xFF，而我們已知 Mat 個顏色通道順序為 BGR，這邊直接調換元素的 indices，即可輕鬆轉回 RGB 的形式。
