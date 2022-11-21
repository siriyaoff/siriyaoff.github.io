---
layout: single
title: "psd-tools 간단 사용법"
categories:
  - Dev
tags:
  - Python
  - psd-tools
  - psd
  - layer
---

```python
from psd_tools import PSDImage
from matplotlib import pyplot as plt
import json

psd=PSDImage.open('tpsd.psd')
for layer in psd:
    if layer.kind=='group':
        for tl in layer:
            if '23' in tl.name and '4' in tl.name:
                print('found')
                limg=tl.composite()
                limg.save('teyelash.png')
                plt.imshow(limg)

print('end')

from IPython.display import Image
Image('/home/ec2-user/teyelash.png')
```

- `psd`로 파일을 받으면 `for layer in psd`와 같이 layer를 탐색할 수 있음
  - 이때 `layer.kind`는 `'pixel'`, `'group'` 두 가지가 존재함
    - `'pixel'` : 한 장짜리 레이어
    - `'group'` : 레이어 그룹
      `’pixel’` 레이어의 리스트이기 때문에 `psd`와 마찬가지로 탐색 가능
- 레이어 객체의 attribute
  - `layer.name` : 레이어 이름
  - `layer.kind` : 레이어 종류
  - `layer.composite()` : `layer`에 속하는 레이어들을 합쳐서 `PIL.Image` 타입으로 반환
