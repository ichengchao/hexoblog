---
title: 鼠须管(Squirrel)输入法
date: 2023-04-06 19:35:17
tags:

 - mac
 - tech

---



![image-20230406195300319](https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/uPic/2023_04_06_1680781985.png)

#  背景

看到好多人在安利Rime输入法,就忍不住要去试试,之前有用过几次都没坚持下来.看到这次有人整理好了词库和配置.应该会好很多

# 安装

Rime 官网: https://rime.im/

- Windows : 小狼毫 Weasel
- macOS : 鼠鬚管 Squirrel

配合雾凇拼音的配置:https://github.com/iDvel/rime-ice

```bash
cd ~/Library
# fork from rime-ice,加上了一些个性化的配置
git clone git@github.com:ichengchao/rime-ice.git
rm -rf Rime
# 或者备份原来的配置 
# mv Rime Rime.old
ln -s rime-ice Rime
# 然后点击菜单中的 Deploy
```



# 个性化配置

- 开启 “ 逗号，句号”翻页

  ```yaml
  # 文件: default.yaml
  # 需要额外注释掉方案中 recognizer/patterns 下的 url_2 选项（这个会覆盖掉句号的行为）
      - { when: paging, accept: comma, send: Page_Up }
      - { when: has_menu, accept: period, send: Page_Down }
  
  # 文件: rime_ice.schema.yaml
  # 使用句号翻页时需要注释掉
  # url_2: "^[A-Za-z]+[.].*" 
  ```

- 使用macos_dark主题,我个性化了一点配置,主题列表在这里：https://github.com/NavisLab/rime-pifu

  ```yaml
  # 文件: squirrel.yaml
  macos_dark:
        name: Mac仿原生暗色/macos_dark
        author: 一方
  
        text_color: 0xe8f3f6
        back_color: 0xee303030
        hilited_text_color: 0x82e6ca
        hilited_candidate_text_color: 0x000000
        hilited_candidate_back_color: 0x82e6ca
        comment_text_color: 0xbb82e6ca
        hilited_comment_text_color: 0xbb203d34
        horizontal: true    # 水平排列
        inline_preedit: true    # 单行显示，false双行显示
        candidate_format: "%c\u2005%@"    # 用 1/6 em 空格 U+2005 来控制编号 %c 和候选词 %@ 前后的空间。
        font_face: "PingFangSC"   # 候选词编号字体
        font_point: 64    # 候选文字大小
        label_font_point: 24    # 候选编号大小
        corner_radius: 5    # 候选条圆角
        hilited_corner_radius: 5    # 高亮圆角
        border_height: 4     # 窗口上下高度
        border_width: 4   # 窗口左右宽度
        border_color_width: 0   #输入条边框宽度
  ```

- 在 custom_phrase.txt 中增加一些常用词,一般是姓名和住址

  ```
  # 以 Tab 分割：汉字<Tab>编码<Tab>权重
  测试	cs	1
  城市	cs	2
  ```

- 默认使用英文标点符号

  ```yaml
  # 文件： rime_ice.schema.yaml
  states: [ 。，, ．， ]
  reset: 1
  ```

- 忽略翘舌音和后鼻音

  ```yaml
  # 文件： rime_ice.schema.yaml 选择需要忽略的选项
  # 拼写设定
  speller:
  ```

- 将CapsLock设置成中英文切换

  ```yaml
  # 文件: default.yaml
  good_old_caps_lock: false
  ```

- 指定App默认英文

  ```yaml
  # 文件: squirrel.yaml 使用osascript -e 'id of app "iTerm2"' 查询对应的code
  app_options:
  	com.googlecode.iterm2:
    	ascii_mode: true
    org.eclipse.platform.ide:
    	ascii_mode: true
  ```

  

# 小技巧

- 快速输入日期(rq),时间(sj),时间戳(ts)
- 强大的V模式: 节气(vjq),星座(vxz),天气(vtq),麻将(vmjvmj)



# 配置文件说明

```
.
├── default.yaml   # 一些全局设置
├── squirrel.yaml  # 鼠须管的皮肤、默认中英设置

├── rime_ice.schema.yaml  # 全拼方案
├── double_pinyin...      # 双拼方案
├── rime_ice.dict.yaml    # 挂载词库
├── cn_dicts/             # 词库目录
├── symbols_custom.yaml   # 标点符号，全拼 v 模式
├── symbols_custom.yaml   # 双拼 V 模式

├── melt_eng.schema.yaml  # 英文方案，作为次翻译器挂载到拼音方案
├── melt_eng.dict.yaml    # 挂载词库
├── en_dicts/             # 词库目录

├── liangfen.schema.yaml  # 两分方案，作为反查挂载到拼音方案
├── liangfen.dict.yaml    # 两分词库

├── custom_phrase.txt         # 自定义短语
├── opencc/                   # 词语映射，Emoji
├── rime.lua                  # lua 脚本
└── zh-hans-t-essay-bgw.gram  # 八股文
```

