---
title: "使用google sheet + api定时获取油管播放量"
datePublished: Sun Dec 15 2024 14:19:47 GMT+0000 (Coordinated Universal Time)
cuid: cm4pp02lk000009mt9k3ebuqz
slug: google-sheet-youtube-api
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734272325936/3a412e91-a703-401e-af65-8b24d0bade92.png
tags: youtube, google-apps-script, googlesheets, milklove

---

起因是想要记录一下 milklove 的二搭剧 Whale Store xoxo 在油管上的预告片的播放量的 [คุณวาฬร้านชำ (Whale Store xoxo) | GMMTV 2025 - YouTube](https://www.youtube.com/watch?v=Eia_Sh_ZTyQ)

于是先搜了一下有没有可以直接使用的开源项目或者接口，然后在 [批量统计YouTube视频播放量方法 - 杨哥的出海营销笔记](https://marketingyang.com/%E6%89%B9%E9%87%8F%E7%BB%9F%E8%AE%A1youtube%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%E9%87%8F%E6%96%B9%E6%B3%95/) 这篇文章中了解到了可以使用 google sheet 的 apps 脚本 直接获取油管的播放量。

不过这篇文章中的脚本更多的获取当前的播放量，而不是定时获取，因此我在这个脚本的基础上进行了一些修改，来实现定时获取播放量，然后把时间和对应的播放量的信息存到 sheet 中。

最终的效果如图所示。[杂货铺播放量记录 - Google 表格](https://docs.google.com/spreadsheets/d/12l5v7V-lyHFXw1KkgGAFtnvbVejrKshOLGsulLf8P1A/edit?usp=sharing)

![CleanShot 2024-12-15 at 21.38.06@2x.png](https://pictures.kazoottt.top/2024/12/20241215-1d48251e5fc8c4b10db40df0be42ec44.png align="left")

## 操作教程

第一步，在 sheet 中的原始模板是这样的：

| 链接 | 视频 id | 时间 | 播放量 |
| --- | --- | --- | --- |
| [https://www.youtube.com/watch?v=Eia\_Sh\_ZTyQ](https://www.youtube.com/watch?v=Eia_Sh_ZTyQ) | \=MID(A2, FIND("v=", A2) + 2, LEN(A2) - FIND("v=", A2) - 1) |  |  |

在第一列写上要监听的油管的链接，然后视频的 id 通过公式直接计算出来。

第二步，点击 extentions - apps script 这里,跳转到脚本配置界面。

![CleanShot 2024-12-15 at 21.40.14@2x.png](https://pictures.kazoottt.top/2024/12/20241215-67500f88d9213bc751ab385fabc8ba4d.png align="left")

点击左侧的 services 的加号，添加 YouTube Data API v3，标识符、版本直接默认就可以了。

![CleanShot 2024-12-15 at 21.44.17@2x.png](https://pictures.kazoottt.top/2024/12/20241215-910db7890a903b9cd74b173ad34fa0ad.png align="left")

第三步，把代码复制到 [Code.gs](http://Code.gs) 中，效果如图：

```js
function onOpenFunc() {
  PropertiesService.getScriptProperties().setProperty("accessToken", ScriptApp.getOAuthToken());
}

function getVideoViews(videoid) {
  var accessToken = PropertiesService.getScriptProperties().getProperty('accessToken');
  
  // Check if the access token exists
  if (!accessToken) {
    Logger.log('No access token found.');
    return 'Error: No access token';
  }
  
  try {
    var videoStatsResponse = YouTube.Videos.list('statistics', {
      'id': videoid, 
      'access_token': accessToken
    });
    
    // Check if the response contains the expected data
    if (videoStatsResponse.items && videoStatsResponse.items[0] && videoStatsResponse.items[0].statistics) {
      return videoStatsResponse.items[0].statistics.viewCount;
    } else {
      Logger.log('No statistics found for video ID: ' + videoid);
      return 'Error: No statistics';
    }
  } catch (error) {
    Logger.log('Error fetching statistics for video ID ' + videoid + ': ' + error.toString());
    return 'Error: ' + error.toString();
  }
}

function recordYouTubeViewCount() {
  // 获取活动表格
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  
  // 获取第二行的 ID
  var videoId = sheet.getRange(2, 2).getValue();   
  
  // 获取当前时间
  var currentTime = new Date();
  
  try {
    // 获取播放量
    var viewCount = getVideoViews(videoId);
    
    // 获取当前表格的最后一行
    var lastRow = sheet.getLastRow();
    
    // 在最后一行之后插入新行，并填写时间和播放量
    sheet.insertRowAfter(lastRow);
    sheet.getRange(lastRow + 1, 3).setValue(currentTime); // 设置时间
    sheet.getRange(lastRow + 1, 4).setValue(viewCount); // 设置播放量
  } catch (error) {
    // 错误处理
    Logger.log('Error processing video ' + videoId + ': ' + error.toString());
    
    // 如果发生错误，记录错误信息
    var lastRow = sheet.getLastRow();
    sheet.insertRowAfter(lastRow);
    sheet.getRange(lastRow + 1, 3).setValue(currentTime); // 设置时间
    sheet.getRange(lastRow + 1, 4).setValue('Error: ' + error.toString()); // 设置错误信息
  }
}
```

![CleanShot 2024-12-15 at 21.41.33@2x.png](https://pictures.kazoottt.top/2024/12/20241215-bc187c06a7f8ff245814034294da6035.png align="left")

然后如果要验证是否正确，选择 recordYouTubeViewCount，然后点击 run 按钮。不出意外的话会在 sheet 中新增一行记录。

![CleanShot 2024-12-15 at 21.42.08@2x.png](https://pictures.kazoottt.top/2024/12/20241215-9e27a00e167349a672a56770393fb680.png align="left")

第四步，也是最后一步，我们要设置一下 trigger,来实现定时地查询播放量并且写入到 sheet 中。点击左边侧边栏的 trigger 菜单，再点击右下角的 add trigger.

![CleanShot 2024-12-15 at 21.47.04@2x.png](https://pictures.kazoottt.top/2024/12/20241215-9cc933a553b36a5f35cb1310ee5f455f.png align="left")

设置如下，配置完成后点击 save 按钮。

![image.png](https://pictures.kazoottt.top/2024/12/20241215-2158ed0c440adc3d3bae20f2bfdc5355.png align="left")

点击完之后，可能出现一个验证弹窗，需要你选择自己的 google 账号，然后点击 go to xxx project (unsave), 点击 allow 进行授权。（这里没有保留截图，但或许大概应该能直接看懂？）

然后我们就完成了所有的配置，等待一小时后看是否运行正常即可。

## 参考和鸣谢

这个脚本是在 [批量统计YouTube视频播放量方法 - 杨哥的出海营销笔记](https://marketingyang.com/%E6%89%B9%E9%87%8F%E7%BB%9F%E8%AE%A1youtube%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%E9%87%8F%E6%96%B9%E6%B3%95/) 的脚本的基础上改的，感谢作者提供了很详细的教程和代码。

同时也要感谢 google 提供的 api.