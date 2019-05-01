

![](https://raw.githubusercontent.com/tanliner/tanliner.github.io/master/img/readme_preview.png)

[![Build Status](https://travis-ci.org/tanliner/tanliner.github.io.svg?branch=master)](https://travis-ci.org/tanliner/tanliner.github.io/)
[![GitHub issues](https://img.shields.io/github/issues/tanliner/tanliner.github.io.svg?style=flat)](https://github.com/tanliner/tanliner.github.io/issues)
[![License MIT](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://github.com/home-assistant/home-assistant-iOS/blob/master/LICENSE)
[![](https://img.shields.io/github/stars/tanliner/tanliner.github.io.svg?style=social&label=Star)](https://github.com/tanliner/tanliner.github.io)
[![](https://img.shields.io/github/forks/tanliner/tanliner.github.io.svg?style=social&label=Fork)](https://github.com/tanliner/tanliner.github.io)

>
### [查看博客戳这里](http://tanliner.github.io)

### 如何构建你的Jkeyll博客
[BY-Blog](https://github.com/qiubaiying/qiubaiying.github.io)

### 本地环境
> server start: jekyll server [--detach]
> hot dev: jekyll build [--watch]
> http://localhost:4000
> server stop: pkill -f jekyll


##### 优雅点的本地环境
alias jkybuild='jkybuild'<br/>
alias jkybuildw='jkybuild --watch'<br/>
alias jkyStart='server --detach'<br/>
alias jkykill='pkill -f jekyll'<br/>

```bash
$ jkyStart &
$ jkybuildw
// chrome s输入
web> http://localhost:4000
```

### 如何查找你的 blog
已集成[Blog 搜索](https://github.com/christian-fei/Simple-Jekyll-Search)，提供了搜索和呈现方式
大致原理，通过js执行post目录的查询，然后遍历模板并完成替换


## 致谢

1. 这个模板是从这里 [BY](https://github.com/qiubaiying/qiubaiying.github.io) fork 的, 感谢这个作者。 更多blog请前往[他的仓库](https://github.com/qiubaiying/)查看
2. 感谢 Jekyll、Github Pages 和 Bootstrap!

## License

遵循 MIT 许可证。有关详细,请参阅 [LICENSE](https://github.com/tanliner/tanliner.github.io/blob/master/LICENSE)。

