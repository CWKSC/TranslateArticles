---
date: 2020-08-06 16:00:00
layout: post
title: "Canvas 組件（屬性、靜態方法、公共方法、繼承成員、Operators）"
subtitle: 
description: 
image: 
optimized_image: 
category: Unity
tags:
  - Unity
  - C#
author: CWKSC
paginate: false
---

只是筆記，搬運自：

[Untiy 笔记 - Canvas 組件（属性、静态方法、公共方法、继承成员、Operators） - 知乎](https://zhuanlan.zhihu.com/p/76748731)

___

這只是一個整合和翻譯，利用寫筆記的方式去理解 Unity

翻譯至 [Unity - Scripting API: Canvas](https://docs.unity3d.com/ScriptReference/Canvas.html) ，有修改/添加額外的內容

## ▌Overview 一下

### Properties 屬性：

- [additionalShaderChannels](https://docs.unity3d.com/ScriptReference/Canvas-additionalShaderChannels.html) 獲取或設定 additionalShaderChannels 成員
- [cachedSortingLayerValue](https://docs.unity3d.com/ScriptReference/Canvas-cachedSortingLayerValue.html) 基於 SortingLayerID 的緩存計算值
- [isRootCanvas](https://docs.unity3d.com/ScriptReference/Canvas-isRootCanvas.html) 是否根 Canvas 
- [normalizedSortingGridSize](https://docs.unity3d.com/ScriptReference/Canvas-normalizedSortingGridSize.html) Canvas 可渲染區域拆分為的標準化網格的大小。
- [overridePixelPerfect](https://docs.unity3d.com/ScriptReference/Canvas-overridePixelPerfect.html) 允許嵌套 Canvas 覆蓋從父 Canvas 繼承的 pixelPerfect 設置
- [overrideSorting](https://docs.unity3d.com/ScriptReference/Canvas-overrideSorting.html) 允許嵌套Canvas 忽略父繪製順序並在頂部或下面繪製
- [pixelPerfect](https://docs.unity3d.com/ScriptReference/Canvas-pixelPerfect.html) 是否為了精確，在沒有抗鋸齒的情況下呈現UI ？
- [pixelRect](https://docs.unity3d.com/ScriptReference/Canvas-pixelRect.html) 獲取 Canvas 的渲染矩形(render rect)
- [planeDistance](https://docs.unity3d.com/ScriptReference/Canvas-planeDistance.html) 相機與UI 元件的距離，Canvas 生成的距離相機多遠
- [referencePixelsPerUnit](https://docs.unity3d.com/ScriptReference/Canvas-referencePixelsPerUnit.html) 每被認為是默認的單位像素的數量
- [renderMode](https://docs.unity3d.com/ScriptReference/Canvas-renderMode.html) Canvas 的 Render Mode 成員
- [renderOrder](https://docs.unity3d.com/ScriptReference/Canvas-renderOrder.html) 將畫布發送到場景的渲染順序 （只讀）
- [rootCanvas](https://docs.unity3d.com/ScriptReference/Canvas-rootCanvas.html) 返回最接近 root 的 Canvas
- [scaleFactor](https://docs.unity3d.com/ScriptReference/Canvas-scaleFactor.html) 用於縮放整個畫布，同時仍然使其適合屏幕
- [sortingLayerID](https://docs.unity3d.com/ScriptReference/Canvas-sortingLayerID.html) Canvas 排序圖層的唯一 ID
- [sortingLayerName](https://docs.unity3d.com/ScriptReference/Canvas-sortingLayerName.html) Canvas 排序圖層的名稱
- [sortingOrder](https://docs.unity3d.com/ScriptReference/Canvas-sortingOrder.html) Canvas 在排序圖層中的順序
- [targetDisplay](https://docs.unity3d.com/ScriptReference/Canvas-targetDisplay.html) 選擇渲染到指定的顯示(Display)的目標
- [worldCamera](https://docs.unity3d.com/ScriptReference/Canvas-worldCamera.html) 渲染/事件相機

### Static Methods 靜態方法：

- [ForceUpdateCanvases](https://docs.unity3d.com/ScriptReference/Canvas.ForceUpdateCanvases.html) 強制所有 Canvas 更新其內容
- [GetDefaultCanvasMaterial](https://docs.unity3d.com/ScriptReference/Canvas.GetDefaultCanvasMaterial.html) 返回可用於在 Canvas 上渲染常規元素的默認材質
- [GetETC1SupportedCanvasMaterial](https://docs.unity3d.com/ScriptReference/Canvas.GetETC1SupportedCanvasMaterial.html) 獲取或生成 ETC1 材質

### Events：

- [willRenderCanvases](https://docs.unity3d.com/ScriptReference/Canvas-willRenderCanvases.html) 在 Canvas 渲染髮生之前調用的事件

### Inherited Members：

Properties, Public Methods, Static Methods, Operators :

（不是本文核心，略去。）

## ▌以下正文，有一些較詳細的解釋

## ▌Properties 屬性

- [additionalShaderChannels](https://docs.unity3d.com/ScriptReference/Canvas-additionalShaderChannels.html)

獲取或設定 additionalShaderChannels 成員

- [cachedSortingLayerValue](https://docs.unity3d.com/ScriptReference/Canvas-cachedSortingLayerValue.html)

基於 SortingLayerID 的緩存計算值。

- [isRootCanvas](https://docs.unity3d.com/ScriptReference/Canvas-isRootCanvas.html)

是否根 Canvas ？

- [normalizedSortingGridSize](https://docs.unity3d.com/ScriptReference/Canvas-normalizedSortingGridSize.html)

Canvas 可渲染區域拆分為的標準化網格的大小

在渲染期間，Canvas 將可渲染區域（所有 UI 元素的邊界）拆分為網格
這是該網格的標準化大小

例如，如果您的可渲染區域為 100 個單位且 sortedGridNormalizedSize 為 0.1f，
則每個網格單元格將為 10 個單位

注意：值 0 將默認為 0.1f

- [overridePixelPerfect](https://docs.unity3d.com/ScriptReference/Canvas-overridePixelPerfect.html)

允許嵌套 Canvas 覆蓋從父 Canvas 繼承的 pixelPerfect 設置

- [overrideSorting](https://docs.unity3d.com/ScriptReference/Canvas-overrideSorting.html)

允許嵌套 Canvas 忽略父繪製順序並在頂部或下面繪製

- [pixelPerfect](https://docs.unity3d.com/ScriptReference/Canvas-pixelPerfect.html)

Canvas 中的元素與像素對齊，僅適用於 renderMode 是 Screen Space

是否為了精確，在沒有抗鋸齒的情況下呈現 UI ？

啟用 Pixel Perfect 可以使元素看起來更清晰並防止模糊
但是，如果有許多縮放或旋轉的元件，或細微的動畫位置或縮放
禁用 Pixel Perfect 可能會比較好，因為移動將會更平滑

有額外的性能開銷。

- [pixelRect](https://docs.unity3d.com/ScriptReference/Canvas-pixelRect.html)

獲取 Canvas 的渲染矩形(render rect)

如果處於疊加模式(Screen Space - Overlay)，則這將是屏幕尺寸。

如果處於世界模式(World Space )下，這將是相機屏幕視口 rect。

- [planeDistance](https://docs.unity3d.com/ScriptReference/Canvas-planeDistance.html)

相機與 UI 元件的距離，Canvas 生成的距離相機多遠。

- [referencePixelsPerUnit](https://docs.unity3d.com/ScriptReference/Canvas-referencePixelsPerUnit.html)

每被認為是默認的單位像素的數量。

- [renderMode](https://docs.unity3d.com/ScriptReference/Canvas-renderMode.html)

Canvas 的 Render Mode 成員。

配合
[RenderMode.ScreenSpaceCamera](https://docs.unity3d.com/ScriptReference/RenderMode.ScreenSpaceCamera.html)
[RenderMode.ScreenSpaceOverlay](https://docs.unity3d.com/ScriptReference/RenderMode.ScreenSpaceOverlay.html)
[RenderMode.WorldSpace](https://docs.unity3d.com/ScriptReference/RenderMode.WorldSpace.html)
使用

- [renderOrder](https://docs.unity3d.com/ScriptReference/Canvas-renderOrder.html)

將畫布發送到場景的渲染順序 （只讀）

注意：目前只有 Screen Space - Overlay 是正確地排序
Screen Space - Camera 和 World Space 是根據與攝像機的距離進行發射和排序

- [rootCanvas](https://docs.unity3d.com/ScriptReference/Canvas-rootCanvas.html)

返回最接近 root 的 Canvas，方法是檢查每個父級並返回找到的最後一個canvas
如果沒有找到其他畫布，則畫布將自行返回

另請參見：[isRootCanvas](https://docs.unity3d.com/ScriptReference/Canvas-isRootCanvas.html)。

- [scaleFactor](https://docs.unity3d.com/ScriptReference/Canvas-scaleFactor.html)

用於縮放整個畫布，同時仍然使其適合屏幕

僅適用於 renderMode 是屏幕空間(Screen Space)

- [sortingLayerID](https://docs.unity3d.com/ScriptReference/Canvas-sortingLayerID.html)

Canvas 排序圖層的唯一 ID

- [sortingLayerName](https://docs.unity3d.com/ScriptReference/Canvas-sortingLayerName.html)

Canvas 排序圖層的名稱

- [sortingOrder](https://docs.unity3d.com/ScriptReference/Canvas-sortingOrder.html)

Canvas 在排序圖層中的順序

- [targetDisplay](https://docs.unity3d.com/ScriptReference/Canvas-targetDisplay.html)

選擇渲染到指定的顯示 (Display) 的目標，e.g. Display1, Display4, Display8

此設置使 Canvas 渲染到指定的顯示中。支持的輔助顯示器（例如監視器）的最大數量為 8

- [worldCamera](https://docs.unity3d.com/ScriptReference/Canvas-worldCamera.html)

在 Screen Space - Camera，作為 Render Camera 渲染相機

在 World Space ，作為 Event Camera 事件相機

### ▌Static Methods 靜態方法

- [ForceUpdateCanvases](https://docs.unity3d.com/ScriptReference/Canvas.ForceUpdateCanvases.html)

強制所有 Canvas 更新其內容

Canvas 在渲染之前在幀的末尾執行其佈局和內容生成計算
以確保它基於該幀期間可能發生的所有最新更改
這意味著在 Start 回調和第一個 Update 回調中，Canvas 下的佈局和內容可能不是最新的

依賴於最新佈局或內容的代碼可以在執行依賴於它的代碼之前調用此方法以確保它

- [GetDefaultCanvasMaterial](https://docs.unity3d.com/ScriptReference/Canvas.GetDefaultCanvasMaterial.html)

返回可用於在 Canvas 上渲染常規元素的默認材質

- [GetETC1SupportedCanvasMaterial](https://docs.unity3d.com/ScriptReference/Canvas.GetETC1SupportedCanvasMaterial.html)

獲取或生成 ETC1 材質

### ▌Events

- [willRenderCanvases](https://docs.unity3d.com/ScriptReference/Canvas-willRenderCanvases.html)

在 Canvas 渲染髮生之前調用的事件

這允許您延遲處理/更新基於畫布的元素，直到它們呈現之前

## ▌Inherited Members

### ▌Properties

- [enabled](https://docs.unity3d.com/ScriptReference/Behaviour-enabled.html)

啟用

已啟用會更新，禁用則相反（停止更新）

- [isActiveAndEnabled](https://docs.unity3d.com/ScriptReference/Behaviour-isActiveAndEnabled.html)

行為是否已激活且已啟用？

GameObject 可以是活動的或不活動的
同樣，可以啟用或禁用腳本

如果GameObject處於活動狀態且具有啟用的腳本
則 [isActiveAndEnabled](https://docs.unity3d.com/ScriptReference/Behaviour-isActiveAndEnabled.html) 將返回`true`。否則`false`返回

- [gameObject](https://docs.unity3d.com/ScriptReference/Component-gameObject.html)

此組件附加到的遊戲對象，組件總會有一個被附加的遊戲對象

- [tag](https://docs.unity3d.com/ScriptReference/Component-tag.html)

這個遊戲對象的標籤

- [transform](https://docs.unity3d.com/ScriptReference/Component-transform.html)

附加到此 GameObject 的 Transform

- [hideFlags](https://docs.unity3d.com/ScriptReference/Object-hideFlags.html)

隱藏對象，使用場景保存還是用戶可修改？

配合 [HideFlags.HideAndDontSave](https://docs.unity3d.com/ScriptReference/HideFlags.HideAndDontSave.html) 使用

[HideFlags.HideAndDontSave](https://docs.unity3d.com/ScriptReference/HideFlags.HideAndDontSave.html) ：

> GameObject 不會顯示在層次結構中，也不會保存到場景中，
> 也不會被 [Resources.UnloadUnusedAssets](https://docs.unity3d.com/ScriptReference/Resources.UnloadUnusedAssets.html) 卸載。
> 這最常用於由腳本創建的遊戲對象，並且完全由腳本控制。

- [name](https://docs.unity3d.com/ScriptReference/Object-name.html)

對象的名稱

組件與遊戲對象和所有附加組件共享相同的名稱

如果一個類從 MonoBehaviour 派生，它會繼承了 name 字段
如果此類也附加到 GameObject，則 name 字段將設置為該 GameObject 的名稱

### ▌Public Methods

- [BroadcastMessage](https://docs.unity3d.com/ScriptReference/Component.BroadcastMessage.html)

在此遊戲對像或其任何子對象的每個 MonoBehaviour 上調用名為 methodName 的方法

- [CompareTag](https://docs.unity3d.com/ScriptReference/Component.CompareTag.html)

這個遊戲對像被這個標籤標記的嗎？

- [GetComponent](https://docs.unity3d.com/ScriptReference/Component.GetComponent.html)

如果遊戲對像有附加這個組件，則返回 Type 類型的組件，如果沒有，則返回 null

- [GetComponentInChildren](https://docs.unity3d.com/ScriptReference/Component.GetComponentInChildren.html)

使用深度優先搜索返回 GameObject 或其任何子節點中 Type 類型的組件

- [GetComponentInParent](https://docs.unity3d.com/ScriptReference/Component.GetComponentInParent.html)

返回 GameObject 或其任何父項中的Type類型的組件

- [GetComponents](https://docs.unity3d.com/ScriptReference/Component.GetComponents.html)

返回 GameObject 中 Type 類型的所有組件

- [GetComponentsInChildren](https://docs.unity3d.com/ScriptReference/Component.GetComponentsInChildren.html)

返回 GameObject 或其任何子項中Type類型的所有組件

- [GetComponentsInParent](https://docs.unity3d.com/ScriptReference/Component.GetComponentsInParent.html)

返回 GameObject 或其任何父項中Type類型的所有組件

- [SendMessage](https://docs.unity3d.com/ScriptReference/Component.SendMessage.html)

在此遊戲對像中的每個 MonoBehaviour 上調用名為 methodName 的方法

- [SendMessageUpwards](https://docs.unity3d.com/ScriptReference/Component.SendMessageUpwards.html)

調用名為 methodName 的方法
在此遊戲對像中的每個 MonoBehaviour 以及在每個祖先上

- [TryGetComponent](https://docs.unity3d.com/ScriptReference/Component.TryGetComponent.html)

獲取指定類型的組件（如果存在）

- [GetInstanceID](https://docs.unity3d.com/ScriptReference/Object.GetInstanceID.html)

返回對象的實例 ID

- [ToString](https://docs.unity3d.com/ScriptReference/Object.ToString.html)

返回 GameObject 的名稱

### ▌Static Methods

- [Destroy](https://docs.unity3d.com/ScriptReference/Object.Destroy.html)

刪除遊戲對象，組件或資產

void Destroy(Object obj, float t = 0.0F);

該對象 `obj` 現在將被銷毀，或者 `t` 從現在開始指定時間

如果`obj` 是[Component](https://docs.unity3d.com/ScriptReference/Component.html) ，它將從[GameObject](https:/ /link.zhihu.com/?target=https://docs.unity3d.com/ScriptReference/GameObject.html) 中刪除該組件並將其銷毀

如果`obj`是[GameObject](https://docs.unity3d.com/ScriptReference/GameObject.html) 會破壞[GameObject](https://link .zhihu.com/?target=https://docs.unity3d.com/ScriptReference/GameObject.html)
它的所有組件和所有轉換 [GameObject](https://docs.unity3d.com/ScriptReference/GameObject.html) 的子節點

實際對象破壞始終延遲到當前 Update 循環之後，但始終在渲染之前完成

- [DestroyImmediate](https://docs.unity3d.com/ScriptReference/Object.DestroyImmediate.html)

立即銷毀對象 obj。強烈建議您使用 Destroy

- [DontDestroyOnLoad](https://docs.unity3d.com/ScriptReference/Object.DontDestroyOnLoad.html)

加載新場景時，請勿銷毀目標對象

- [FindObjectOfType](https://docs.unity3d.com/ScriptReference/Object.FindObjectOfType.html)

此方法調用 [Object.FindObjectOfType](https://docs.unity3d.com/ScriptReference/Object.FindObjectOfType.html) 並返回與該類型匹配的對象
如果沒有對象與該類型匹配，則返回 null

返回 Type 類型的第一個活動加載 Object

FindObjectOfType 將不返回 Assets（網格，紋理，預製件......）或非活動對象
它用於定位 GameObject 

這不會返回具有 [HideFlags.DontSave](https://docs.unity3d.com/ScriptReference/HideFlags.DontSave.html) 設置的 Object 

請注意，此功能非常慢。不建議每幀使用此功能。在大多數情況下，您可以使用單例模式

- [FindObjectsOfType](https://docs.unity3d.com/ScriptReference/Object.FindObjectsOfType.html)

返回與該類型匹配的 Object []

它不會返回任何資產（網格，紋理，預製件......）或非活動對象

不會返回具有 [HideFlags.DontSave](https://docs.unity3d.com/ScriptReference/HideFlags.DontSave.html) 設置的對象

使用 [Resources.FindObjectsOfTypeAll](https://docs.unity3d.com/ScriptReference/Resources.FindObjectsOfTypeAll.html) 可以避免這些限制

請注意，此功能非常慢。不建議每幀使用此功能。在大多數情況下，您可以使用單例模式

- [Instantiate](https://docs.unity3d.com/ScriptReference/Object.Instantiate.html)

克隆對象原始並返回克隆對象

此函數以與編輯器中的“複製”命令類似的方式創建對象的副本

如果您正在克隆 [GameObject](https://docs.unity3d.com/ScriptReference/GameObject.html) 
那麼您也可以選擇指定其位置和旋轉（否則默認為原始 GameObject 的位置和旋轉）

如果您正在克隆一個 [Component](https://docs.unity3d.com/ScriptReference/Component.html) 
那麼它附加的 GameObject 也將被克隆，再次使用可選的位置和旋轉

克隆[GameObject](https://docs.unity3d.com/ScriptReference/GameObject.html) 或[Component](https://link.zhihu.com/ ?target=https://docs.unity3d.com/ScriptReference/Component.html) 時，所有子對象和組件也將被克隆
其屬性設置與原始對象的屬性相同

默認情況下*為父級*新對象的值將為 null，因此它不會是原始對象的 "兄弟"（sibling）
但是，您仍然可以使用重載方法設置父級

如果指定了父級並且未指定位置和旋轉
則如果 instantiateInWorldSpace 參數為 true
則原始位置和旋轉將用於克隆對象的本地位置和旋轉，或其世界位置和旋轉

如果指定了位置和旋轉，它們將用作對像在世界空間中的位置和旋轉

克隆時 GameObject 的活動狀態將被傳遞
因此如果原始文件處於非活動狀態，則克隆也將在非活動狀態下創建

### ▌Operators

- [bool](https://docs.unity3d.com/ScriptReference/Object-operator_Object.html)

對象存在嗎？

- [operator !=](https://docs.unity3d.com/ScriptReference/Object-operator_ne.html)

比較兩個對像是否引用不同的對象

- [operator ==](https://docs.unity3d.com/ScriptReference/Object-operator_eq.html)

比較兩個對象引用以查看它們是否引用同一對象