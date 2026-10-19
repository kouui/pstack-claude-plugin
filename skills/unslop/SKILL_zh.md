---
name: unslop
description: 去除任何文字中的 AI 痕迹。
disable-model-invocation: true
---

# Unslop

编辑文本，去除 AI 句式。先扫描下面的句式，再改写。保留原意，并符合目标语气。

## 需要检测并修复的句式

规则编号是其他 skill 引用的稳定 ID。删除某条规则后会留下编号空缺。

### 内容

3. **表面化的 -ing 短语。** "highlighting...""ensuring...""reflecting...""showcasing...""fostering..."。删除，或用真实来源展开。
5. **含糊的归因。** "Experts believe""Industry reports suggest""Some critics argue"。指明来源，或删除。

### 语言

7. **AI 词汇。** Additionally、crucial、delve、enduring、enhance、fostering、garner、interplay、intricate、landscape（抽象用法）、pivotal、showcase、tapestry（抽象用法）、testament、underscore、vibrant。换成朴素的词。
8. **"is"的花哨替代说法。** "serves as""stands as""boasts""features"。改用 "is" 或 "has"。
9. **"Not just X, but Y."** 直接陈述观点。
10. **三段式排比。** 不要把观点硬凑成三个一组。使用实际的数量。
11. **同义词轮换。** 同一段中出现 Protagonist、main character、central figure、hero。选定一个，重复使用。
12. **虚假范围。** "from X to Y"中，X 和 Y 不在有意义的同一尺度上。直接列出主题。

### 风格

13. **破折号过度使用。** 完全避免破折号。只用句号或逗号（不用括号、不用 en dash、不用连字符充当破折号）。如果需要分隔想法，就结束句子，或使用逗号。
14. **冒号过度使用。** 冒号可用于列表或举例之前，但不要作为句中的连接词。"If you're coming from traditional automation: instead of registering event handlers, you describe conditions"中的冒号没有增加信息。改写时，让观点独立成立，不要用对比框架。"Describing when the scheduler should fire works best as plain English." 意思相同，不依赖这种标点拐杖。
15. **粗体过度使用。** 不要给每个专有名词或缩写都加粗。
16. **行内标题式列表。** 特征是粗体标签加冒号，重复了本行内容："**Performance:** Performance improved..."。改成正文。如果粗体引导词以句号结尾、点明条目名称，且后面跟着真正的新信息（"**Schema in TypeScript.** Tables live in one file."），则可以保留，这不是问题。
17. **标题使用 Title Case。** 改用 sentence case。
18. **装饰性 emoji。** 从标题和列表项中删除。
19. **弯引号。** 使用直引号。

### 交流类痕迹

20. **聊天机器人式用语。** "I hope this helps!""Let me know if...""Of course!""Certainly!""Found the smoking gun!"。删除。
22. **谄媚语气。** "Great question! You're absolutely right!"。直接回应。

### 填充语

23. **填充短语。** "In order to"改为"To"。"Due to the fact that"改为"Because"。删除 "It is important to note that"。
24. **过度保留。** "could potentially possibly be argued that it might"改为 "may"。
25. **空泛的结论。** "The future looks bright."。改为具体的计划或事实。

### 术语

26. **抽象隐喻名词。** Substrate（基础）、wedge、wedge in（加入）、vector（方式、方法）、locus、vantage、nexus、作名词的 primitive、作隐喻的 harness、"API surface"中的 surface、bedrock、作隐喻的 scaffolding、modality、paradigm、gold-plating（超出工作所需）、作隐喻的 ratchet（机制的真实名称，或只能收紧的限制）、把代码迁出说成 evacuate（应说 move out）、endgame（最后阶段）、north star、flywheel。这些词读起来很专业，但通常有更朴素、具体的词。选具体的词。

### 朴素表达

27. **说明它做什么，而不是它有什么感觉。** "the database stays close at hand""SQL you can read""types that follow your schema"描述的是感受。修改应写出机制或数字："`.toSQL()` returns the exact string sent to the database""a column rename fails the build"。问自己这句话告诉读者要做什么或要知道什么，然后直接写出来。如果无法改写成具体的指令、事实或数字，就删掉。另一个检验方法：如果这句话原封不动地出现在别的项目文档里也成立，那它就没有说明本项目的情况，删掉。
28. **缩短或拆分复杂句。** 如果读者需要回头才能理解一个句子，就把它拆成两句，或删掉从句。每句只表达一个观点。
29. **使用主动语态。** 优先使用。注意 "is/are/was/were + 过去分词" 结构，并点明执行者："queries are validated"改为"the compiler validates queries"，"the file is parsed by the loader"改为"the loader parses the file"。只有在执行者未知或确实无关紧要时，才可以使用被动语态。
30. **删去副词，或使用更有力的动词。** "runs quickly"改为"is fast"或具体数字，"significantly improves"改为测得的具体变化量。如果副词在支撑一个弱动词，说明动词选错了。
31. **优先使用朴素的词。** "utilize"和"leverage"改为"use"，"facilitate"改为"help"，"numerous"改为"many"，"in the event that"改为"if"。更花哨的同义词通常并不更清楚。
32. **做作的文风。** 在已有字面表达的地方使用比喻或修饰：警句（"wire it or delete it"）、为了效果而用的修辞性断句、拟人化的代码（"the plan holds it"）、比喻性动词（"rides along""stands on"）、套话式框架。"A dial worth turning"改为"a parameter worth varying"。直接说出你的意思。隐喻名词由第 26 条处理。
33. **过度压缩。** 省略冠词、无动词的短语、符号式表达，以及需要读者解码才能理解的缩写。"Parser rejects bad date → exit 2, no write"改为"The parser rejects a bad date, exits with code 2, and writes nothing."。写完整的句子，保留冠词和动词，箭头和缩写要写清楚。
