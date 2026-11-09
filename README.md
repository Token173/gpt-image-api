# GPT Image API 中转：国内直连 AI 生图接口，Flare、Sunburst、2.5、2、2-d 全覆盖

做 AI 绘图的项目，最烦的不是模型效果，是接口这层。官方端点连不上、额度限制怪、报错信息还不告诉你到底哪一步错了。Token173（[https://token173.net](https://token173.net)）把 `gpt-image-2` 全系的图像接口放到国内节点上，协议照抄 OpenAI，SDK 不用换。

## 可用版本



| 模型 ID           | 场景                         |
| --------------- | -------------------------- |
| `gpt-image-2.6-flare` | 2.6 快速迭代版，低画质最快约 6 秒，排版优化 |
| `gpt-image-2.6-sunburst` | 2.6 细节版，编辑精度与一致性更好 |
| `gpt-image-2.5` | 写实与细节表现好，适合商业素材            |
| `gpt-image-2`   | 基础版，单价低，草图与批量试稿            |
| `gpt-image-2-d` | 编辑模式，改图、扩图、局部重绘            |

## 三个接口的对应关系



| 动作       | 端点                            | 说明         |
| -------- | ----------------------------- | ---------- |
| 文生图      | `POST /v1/images/generations` | prompt 出图  |
| 图生图 / 编辑 | `POST /v1/images/edits`       | 传原图 + 修改描述 |
| 尺寸变化     | 同上，改 `size`                   | 扩图靠改画布比例   |

## 最小可用示例



```
curl https://token173.net/v1/images/generations \
  -H "Authorization: Bearer sk-你的令牌" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2.6-flare",
    "prompt": "木质书桌上的机械键盘，侧逆光，浅景深，不要出现文字",
    "size": "1536x1024"
  }'
```



```
from openai import OpenAI
import base64

client = OpenAI(api_key="sk-你的令牌", base_url="https://token173.net/v1")

res = client.images.generate(
    model="gpt-image-2.6-sunburst",
    prompt="木质书桌上的机械键盘，侧逆光，浅景深，不要出现文字",
    size="1536x1024",
)
open("kb.png", "wb").write(base64.b64decode(res.data[0].b64_json))
```

## 电商图批量产出



```
import base64, time
from openai import OpenAI

client = OpenAI(api_key="sk-你的令牌", base_url="https://token173.net/v1")
products = ["保温杯", "帆布包", "桌面收纳盒"]

for name in products:
    try:
        res = client.images.generate(
            model="gpt-image-2.5",
            prompt=f"{name}，纯白背景，45度俯拍，电商主图风格，画面仅出现产品本身",
            size="1024x1024",
        )
        open(f"{name}.png", "wb").write(base64.b64decode(res.data[0].b64_json))
    except Exception as e:
        print(name, "失败", e)
    time.sleep(1.5)
```

节奏放慢一点，比堆并发更不容易失败。一批几十张的任务，1 到 2 秒一张很稳。

## 尺寸与画质怎么选



| 用途              | 建议尺寸      | 版本              |
| --------------- | --------- | --------------- |
| 电商主图、商品卡片       | 1024×1024 | `gpt-image-2.5` |
| 横版 banner、详情页头图 | 1536×1024 | `gpt-image-2.6-flare` |
| 竖版海报、社媒封面       | 1024×1536 | `gpt-image-2.6-sunburst` |
| 草稿、方案对比         | 512×512   | `gpt-image-2`   |
| 图标、可缩放素材        | 任意，输出 SVG | `gpt-image-2.6-sunburst` |

## 计费

按张数计费，分辨率越高单价越高。账户 2 元换 1 美元，注册送 0.2 美元，后台能看到每一张的实际扣费。批量前先用低分辨率跑通流程，再换高分辨率出成品。

## 常见问题

**Q：出来的图有水印吗？**

没有。返回的就是模型原始输出。

**Q：中文字在画面里总是糊？**

这是模型当前的普遍短板。需要画面里带准确文字时，把文字部分留给设计工具后期叠，别指望模型写对。

**Q：能不能固定同一个人物形象？**

可以用参考图走编辑接口，但一致性有限。要求高的话用 LoRA 类方案更靠谱。

**Q：报&#x20;**`invalid size`**？**

尺寸必须用模型支持的档位，自定义宽高容易被拒。按上表的档位填。

**Q：并发太高返回 429？**

降并发或加延迟。出图是重任务，别和文本请求抢同一个高并发池。

## 相关



* 文档站：[https://docs.token173.net](https://docs.token173.net)

* 2.6 版本详解：[gpt-image-2](https://github.com/Token173/gpt-image-2)

* 谷歌图像：[Nano-Banana-api](https://github.com/Token173/Nano-Banana-api)

* 文本模型：[gpt-api](https://github.com/Token173/gpt-api)