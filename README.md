# 使用方法
## 
在components创建一个PuzzleVerification文件夹，然后index.vue 放到里面即可
## 引入组建
```
import PuzzleVerification from '@/components/PuzzleVerification/index.vue'
```
## 使用组建
```
 <PuzzleVerification width="260" height="120" :verificationShow="verificationShow"
                                    blockType="puzzle" :onSuccess="sliderOnSuccess" :onError="sliderOnFail" />
```

## 参数说明

 参数  |  是否必需  |  类型  |  可选值  |  默认值  |  说明
 ---- | ---------- | ------ |  -----  |  ------ | ----
 `blockType` | 否 | string | square, puzzle | puzzle | 滑块的形状
 `blockSize` | 否 | string, number | 0 ~ | 40 | 滑块的大小（正方形），不能大于画布尺寸
 `width` | 否 | string, number | 0 ~ | 260 | 画布图片的宽度
 `height` | 否 | string, number | 0 ~ | 120 | 画布图片的高度
 `puzzleImgList` | 否 | array | --- | 三张引用图片 | 传入的图片
 `blockRadius` | 否 | string, number | 0 ~ | 4 | 滑块圆角的大小（仅当其形状是square有效）
 `deviation` | 否 | string, number | 0 ~ | 4 | 滑块吻合的误差
 `wraperPadding` | 否 | string, number | 0 ~ | 20 | 滑块随机出现的范围，数字越大，范围越大（不能大于画布尺寸）
 `onSuccess` | 是 | function | --- | 打印成功信息 | 拼接成功时的回调
 `onError` | 否 | function | --- | 打印失败信息 | 拼接失败时的回调

