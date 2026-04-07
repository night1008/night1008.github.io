和代码一样，提示词和设计文档都是项目资产，应该放在 Git 仓库里统一管理。

期望按照如下目录结构进行组织
```
/docs
  README.md                    - 说明文档
  /designs
    /2026-03-01-operation-task - 需求标题, 格式 YYYY-MM-DD-requirement-title
      prompt.md                - 需求提示词
      design.md                - AI 设计文档
    /2026-03-02-ai-analyse
      prompt.md
      design.md
```

如何使用
```
1. 根据 @prompt.md，输出设计文档到 design.md
2. 根据 @design.md 中的评论，调整设计文档，格式为 <!-- review: 需要加分页参数 -->
3. 根据 @design.md，开始实现代码
```