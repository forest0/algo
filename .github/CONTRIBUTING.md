# 贡献指南

无论你是热爱算法，还是需要刷题准备面试，都欢迎来参与贡献！


## 快速开始

0. [安装Hugo](https://gohugo.io/getting-started/installing/)。
    - [Windows用户建议直接下载二进制](https://github.com/gohugoio/hugo/releases)，
      然后添加到环境变量即可。

1. fork本仓库

2. clone你的仓库到本地
    ```sh
    git clone --recurse-submodules https://github.com/your-username/algo.git
    ```

3. 实时预览效果
    ```sh
    cd algo
    hugo server
    # 在浏览器里边打开最后给出的地址
    ```

4. 新建一篇笔记，假设你用的oj为LeetCode，题号为123
    ```sh
    cd algo
    hugo new posts/leetcode/123.md
    # 用你喜欢的编辑器编辑content/posts/leetcode/123.md
    # 保存文件就可以在浏览器里边看到markdown渲染后的效果
    ```
- 请将`title`修改为`LeetCode 123`，`author`修改为你
- 如果该题明显属于某个分类如动态规划，请在顶部打上标签
```yaml
tags:
  - DP
```

5. 使用git提交你的修改，并推送到你自己的仓库

6. 向本仓库提交PR


## 参考资料

- [LoveIt主题文档](https://hugoloveit.com/zh-cn/posts/)
    - 如果你不熟悉markdown，可以查看这里的教程
    - 建议浏览该主题支持的扩展shortcode，提高你的笔记颜值


## 笔记内容

- 希望大家能分享算法思路分析和可读性良好的代码
- 如果非原创请在参考部分给出原出处
