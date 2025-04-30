# draw.io

a: 文本
d: 方框
c: 箭头
f: 圆
r: 菱形
x: 自由绘制

draw.io默认配置修改：

[把 draw.io 装修为简单且现代的白板应用](https://www.sansui233.com/posts/2024-11-12-%E6%8A%8Adrawio%E8%A3%85%E4%BF%AE%E4%B8%BA%E7%AE%80%E5%8D%95%E7%BE%8E%E8%A7%82%E7%9A%84%E7%99%BD%E6%9D%BF%E5%BA%94%E7%94%A8)

相关配置项说明，参考官网：

* [Configure the draw.io editor](https://www.drawio.com/doc/faq/configure-diagram-editor)
* [Work with default styles](https://www.drawio.com/blog/default-styles)

博主的配置项：（自己改完设置默认后，新页面又不生效了，导入博主的配置后是生效的）

可以加`flowAnimation`，流动箭头，也可需要时手动启用部分箭头动画
"defaultEdgeStyle": {
  "flowAnimation": 1
},

以及 `"enableLightDarkColors": false,` 避免导出svg时light-dark模式影响手机端的外观显示（背景会显示黑色）

```json
{
  "defaultGridEnabled": false,
  "defaultPageVisible": false,
  "enableLightDarkColors": false,
  "customFonts": [
    "Inter",
    "pingfang sc"
  ],
  "defaultVertexStyle": {
    "strokeWidth": 2,
    "fontSize": 14,
    "fontFamily": "Inter",
    "fontColor": "#1f1f1f",
    "strokeColor": "#295FF3",
    "fillColor": "#dae8fc",
    "rounded": 1,
    "fontStyle": "1"
  },
  "defaultEdgeStyle": {
    "strokeWidth": 2,
    "fontSize": 13,
    "fontFamily": "Inter",
    "fontColor": "#1f1f1f",
    "strokeColor": "#1f1f1f",
    "rounded": 1,
    "curved": 1,
    "fontStyle": "1"
  },
  "customPresetColors": [
    "1F1F1F",
    "BFBFBF",
    "FFD1E1",
    "EB539E",
    "FED9D7",
    "EA4D3E",
    "FFE6CC",
    "F08C00",
    "FFF2CC",
    "F5CE45",
    "E1FEFD",
    "6DE4D0",
    "DFF7DE",
    "65C466",
    "DAE8FC",
    "295FF3",
    "F6E0FF",
    "792CF6"
  ],
  "customColorSchemes": [
    [
      {
        "fill": "#DAE8FC",
        "stroke": "#295FF3",
        "font": "#1f1f1f"
      },
      {
        "fill": "#295FF3",
        "stroke": "none",
        "font": "#ffffff"
      },
      {
        "fill": "#FFF2CC",
        "stroke": "#F5CE45",
        "font": "#1f1f1f"
      },
      {
        "fill": "#F5CE45",
        "stroke": "none",
        "font": "#1f1f1f"
      },
      {
        "fill": "#ffffff",
        "stroke": "#1F1F1F",
        "font": "#1f1f1f"
      },
      {
        "fill": "#1F1F1F",
        "stroke": "none",
        "font": "#ffffff"
      },
      {
        "fill": "#EEEEEE",
        "stroke": "#BFBFBF",
        "font": "#1f1f1f"
      },
      {
        "fill": "#BFBFBF",
        "stroke": "none",
        "font": "#ffffff"
      }
    ],
    [
      {
        "fill": "#F6E0FF",
        "stroke": "#792CF6",
        "font": "#1f1f1f"
      },
      {
        "fill": "#792CF6",
        "stroke": "none",
        "font": "#ffffff"
      },
      {
        "fill": "#FFD1E1",
        "stroke": "#EB539E",
        "font": "#1f1f1f"
      },
      {
        "fill": "#EB539E",
        "stroke": "none",
        "font": "#ffffff"
      },
      {
        "fill": "#FED9D7",
        "stroke": "#EA4D3E",
        "font": "#1f1f1f"
      },
      {
        "fill": "#EA4D3E",
        "stroke": "none",
        "font": "#ffffff"
      },
      {
        "fill": "#FFE6CC",
        "stroke": "#F08C00",
        "font": "#1f1f1f"
      },
      {
        "fill": "#F08C00",
        "stroke": "none",
        "font": "#1f1f1f"
      }
    ],
    [
      {
        "fill": "#DFF7DE",
        "stroke": "#65C466",
        "font": "#1f1f1f"
      },
      {
        "fill": "#65C466",
        "stroke": "none",
        "font": "#ffffff"
      },
      {
        "fill": "#E1FEFD",
        "stroke": "#6DE4D0",
        "font": "#1f1f1f"
      },
      {
        "fill": "#6DE4D0",
        "stroke": "none",
        "font": "#1f1f1f"
      }
    ]
  ]
}
```