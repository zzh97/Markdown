<template>
  <div class="markdown">
    <!-- 菜单 -->
    <div class="menu" :class="menuClass">
      <div class="btn" @click="menuBtn">
        <div class="line"></div>
        <div class="line"></div>
        <div class="line"></div>
      </div>
      <div class="index">
        <div class="caption" v-for="(o, i) in captionArr" :key="i" :class="'caption' + o.level">
          <a :href="'#' + o.name" @click="aJump">{{ o.name }}</a>
        </div>
      </div>
    </div>
    <div class="main">
      <!-- 编辑区 -->
      <div ref="edit" class="edit">
        <div class="lineNum">
          <div class="num" v-for="i in lineNum" :key="i">{{ i }}</div>
        </div>
        <textarea ref="text" v-model="mdText"></textarea>
      </div>
      <!-- 视图区 -->
      <div ref="view" class="view">
        <div class="result" v-html="mdHtml"></div>
      </div>
    </div>
  </div>
</template>

<script>
// https://katex.org/docs/node.html
// npm install katex
import katex from 'katex' // 解析latex
import 'katex/dist/katex.css';
let value = `# 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题

这是**粗体**字

来一首诗

##### 门前

——顾城

我多么希望，有一个门口

早晨，阳光照在草上

我们站着

扶着自己的门扇

门很低，但太阳是明亮的

**草在结它的种子**

**风在摇它的叶子**

**我们站着，不说话**

**就十分美好**

* 选项一

* 选项二

[这是超链接：在线Latex（by妈咪叔）](https://www.latexlive.com/)

`+ '```' + `
这是
  多行
    代码
` + '```' + `

这是$y = ax^2 + bx + c$单行Latex

以下是是多行Latex
$$
\\begin{pmatrix}
a_{11} & a_{12}\\\\
a_{21} & a_{22}
\\end{pmatrix}
\\begin{pmatrix}
x_1\\\\
x_2
\\end{pmatrix}
=
\\begin{pmatrix}
a_{11}x_1 + a_{12}x_2\\\\
a_{21}x_1 + a_{22}x_2
\\end{pmatrix}
$$

***

![这是图片](https://img1.baidu.com/it/u=1855658919,843246122&fm=26&fmt=auto)`

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      mdText: value,
      mdHtml: '',
      lineNum: 5,
      captionArr: [],
      isBack: true,
      menuClass: 'menuBack',
    }
  },
  methods: {
    // 编译markdown代码，为html代码
    md_to_html(md) {
      this.captionArr = []
      let log = console.log.bind(console)
      log = () => { }
      // 全部的<和>变成&lt;和&gt;
      let md_reg = /</g
      md = md.replace(md_reg, '&lt;')
      md_reg = />/g
      md = md.replace(md_reg, '&gt;')

      // 换行
      let arr_warp = md.split(`
`)
      // 匹配string与正则
      let match_reg = function (str, reg, success = function () { }, fail = function () { }) {
        // str是否符合reg正则
        if (str.search(reg) != -1) {
          success()
        } else {
          fail()
        }
      }
      // 开始匹配
      let match = function (arr) {
        // 遍历每行，进行匹配
        arr.forEach((el, i) => {
          // 匹配代码是多行，故放前面
          match_code(el, (line) => {
            arr[i] = line
            log(i + 1, el, '是代码')
          }, () => {
            match_latex(el, (line) => {
              arr[i] = line
              log(i + 1, el, '是公式')
            }, () => {
              match_title(el, (line) => {
                log(i + 1, el, '是标题')
                arr[i] = line
              }, () => {
                match_img(el, (line) => {
                  log(i + 1, el, '是图片')
                  arr[i] = line
                }, () => {
                  // 因为***包含*，故要放在前面先匹配
                  match_line(el, (line) => {
                    log(i + 1, el, '是分割线')
                    arr[i] = line
                  }, () => {
                    match_weight(el, (line) => {
                      log(i + 1, el, '是粗体')
                      arr[i] = line
                    }, () => {
                      match_http(el, (line) => {
                        log(i + 1, el, '是超链接')
                        arr[i] = line
                      }, () => {
                        match_li(el, (line) => {
                          log(i + 1, el, '是选项')
                          arr[i] = line
                        }, () => {
                          match_tex(el, (line) => {
                            log(i + 1, el, '是Latex')
                            arr[i] = line
                          }, () => {
                            log(i + 1, el, '是其他')
                            arr[i] = '<p>' + el + '</p>'
                          })
                        })
                      })
                    })
                  })
                })
              })
            })

          })
        })
      }
      let addCaption = (e, l) => {
        if (e != '') {
          this.captionArr.push({
            name: e,
            level: l,
          })
        }
      }
      // 会改变原数组：任意空格 + 1-5个# + 0/1空格 + 任意
      let match_title = function (line, success = function () { }, fail = function () { }) {
        let isFind = true
        // 开头 任意空格 + ### + 零或一个空格
        let reg = /^\s*#####\s?/g
        match_reg(line, reg, () => {
          let text = line.replace(reg, '')
          addCaption(text, 5)
          // 直接改变el是不会改变原数组的，故使用下标
          line = line.replace(reg, '<h5 id="' + text + '">') + '</h5>'
        }, () => {
          // 四级标题
          reg = /^\s*####\s?/g
          match_reg(line, reg, () => {
            let text = line.replace(reg, '')
            addCaption(text, 4)
            line = line.replace(reg, '<h4 id="' + text + '">') + '</h4>'
          }, () => {
            // 三级标题
            reg = /^\s*###\s?/g
            match_reg(line, reg, () => {
              let text = line.replace(reg, '')
              addCaption(text, 3)
              line = line.replace(reg, '<h3 id="' + text + '">') + '</h3>'
            }, () => {
              // 二级标题
              reg = /^\s*##\s?/g
              match_reg(line, reg, () => {
                let text = line.replace(reg, '')
                addCaption(text, 2)
                line = line.replace(reg, '<h2 id="' + text + '">') + '</h2>'
              }, () => {
                // 一级标题
                reg = /^\s*#\s?/g
                match_reg(line, reg, () => {
                  let text = line.replace(reg, '')
                  addCaption(text, 1)
                  line = line.replace(reg, '<h1 id="' + text + '">') + '</h1>'
                }, () => {
                  isFind = false
                })
              })
            })
          })
        })
        // 虽然上面用了大量的回调函数，但回调函数不一定是异步的
        // 因为上述的回调函数不在I/O里面，故为同步
        // return语句，会在上述回调函数都执行完后运行
        if (isFind) {
          success(line)
        } else {
          // 不是标题
          fail()
        }

      }
      // 匹配代码块：任意空格 + ``` + 任意 + 换行 + ```
      let isEnd = true // 用于奇偶次
      let match_code = function (line, success = function () { }, fail = function () { }) {
        let isFind = true
        let reg = /^\s*```/g
        match_reg(line, reg, () => {
          if (isEnd) {
            // 第奇数次匹配成功
            line = line.replace(reg, '<code>')
          } else {
            // 第偶数次匹配成功
            line = line.replace(reg, '</code>')
          }
          isEnd = !isEnd
        }, () => {
          if (isEnd) {
            isFind = false
          }

        })
        if (isFind) {
          success(line)
        } else {
          fail()
        }
      }
      // 匹配图片：0/1空格 + ![+ 任意 +](+ 任意 +)
      let match_img = function (line, success = function () { }, fail = function () { }) {
        let not_is_img = true
        let reg = /^\s?!\[/
        match_reg(line, reg, () => {
          line = line.replace(reg, '<img alt="')
          reg = /\]\(/
          match_reg(line, reg, () => {
            line = line.replace(reg, '" src="')
            reg = /\)/
            match_reg(line, reg, () => {
              not_is_img = false
              line = line.replace(reg, '" />')
              log(line)
              success(line)
            })
          })
        })
        if (not_is_img) {
          fail()
        }
      }
      // 匹配选项
      let match_li = function (line, success = function () { }, fail = function () { }) {
        let reg = /^\s*\*\s?/
        match_reg(line, reg, () => {
          line = line.replace(reg, '<li>') + '</li>'
          success(line)
        }, () => {
          fail()
        })
      }
      // 分割线
      let match_line = function (line, success = function () { }, fail = function () { }) {
        let not_is_line = true
        let reg = /^\s*\*\*\*\s*/
        match_reg(line, reg, () => {
          not_is_line = false
          line = line.replace(reg, '<hr />')
          success(line)
        })
        reg = /^\s*---\s*/
        match_reg(line, reg, () => {
          not_is_line = false
          line = line.replace(reg, '<hr />')
          success(line)
        })
        if (not_is_line) {
          fail()
        }
      }
      // 超链接
      let match_http = function (line, success = function () { }, fail = function () { }) {
        let not_is_a = true
        let reg = /^\s?\[/
        match_reg(line, reg, () => {
          line = line.split(reg)[1] // xxx]
          reg = /\]\(/
          match_reg(line, reg, () => {
            let text = line.split(reg)[0] // xxx)
            line = line.split(reg)[1] // xxx)
            log('text', text)
            reg = /\)/
            match_reg(line, reg, () => {
              not_is_a = false
              let url = line.split(reg)[0]
              log('url', url)
              line = '<a href="' + url + '" target="_blank">' + text + '</a>'
              success(line)
            })
          })
        })
        if (not_is_a) {
          fail()
        }
      }
      // 粗体
      let is_end = false // 使其只调用一次success
      let match_weight = function (line, success = function () { }, fail = function () { }) {
        let reg = /\*\*/
        match_reg(line, reg, () => {
          line = line.replace(reg, '<b>')
          match_reg(line, reg, () => {
            line = line.replace(reg, '</b>')
            match_weight(line, success)
            if (!is_end) {
              is_end = true
              success(line)
            }
          })
        }, () => {
          fail()
          is_end = false
        })
      }
      // latex
      let match_tex = function (line, success = function () { }, fail = function () { }) {
        let reg = /\$/
        match_reg(line, reg, () => {
          // 切开
          line = line.split(reg)
          line.forEach((el, i) => {
            if (i % 2 == 1) {
              line[i] = katex.renderToString(line[i], {
                // 无效输入，呈现为红色
                throwOnError: false
              });
            }
          })
          // 拼起来
          line = line.join()
          // 去除join后的逗号
          line = line.replace(/,/g, ' ')
          success(line)
        }, () => {
          fail()
        })

        // let not_is_tex = true
        // let reg = /\$/
        // match_reg (line, reg, ()=>{
        //   line = line.replace(reg, ' ')
        //   match_reg (line, reg, ()=>{
        //     line = line.replace(reg, ' ')
        //     not_is_tex = false
        //     let la_html = katex.renderToString(line, {
        //         // 无效输入，呈现为红色
        //         throwOnError: false
        //     });
        //     success(la_html)
        //   })
        // })
        // if (not_is_tex) {
        //   fail()
        // }
      }
      // 多行Latex
      let is_tex_end = true
      let match_latex = function (line, success = function () { }, fail = function () { }) {
        let reg = /\$\$/
        match_reg(line, reg, () => {
          success(line)
          is_tex_end = !is_tex_end
        }, () => {
          if (is_tex_end) {
            fail()
          }
        })
      }
      match(arr_warp)
      // 删除<cade>后的空回车
      let html = ''
      arr_warp.forEach((el) => {
        if (el.indexOf('<code>') == -1) {
          // html代码换行（美观一点）
          html += el + `
`
        } else {
          // <code>后不要换行
          html += el
        }
      })
      // 渲染多行公式
      // log ('html', html)
      let a = html.split('$$')
      let new_html = ''
      a.forEach((el, i) => {
        if (i % 2 == 1) {
          let tex = katex.renderToString(el, {
            // 无效输入，呈现为红色
            throwOnError: false
          })
          new_html += '<div class="zh_tex">' + tex + '</div>'
        } else {
          new_html += el
        }
      })

      log(new_html)
      return new_html
    },
    // 根据edit更新view
    update() {
      let html = this.md_to_html(this.mdText)
      this.mdHtml = html
      let text = this.$refs.text
      let n = text.scrollHeight / 20
      setTimeout(() => {
        // 不初始化height，scrollHeight会不断叠加
        text.style.height = '100px'
        if (text.scrollHeight > 20) {
          text.style.height = text.scrollHeight + 'px'
        }
        text.style.width = 'calc(100% - 30px)';
      }, 100)
      // 行数
      this.lineNum = n
    },
    // edit区域的锚点跳转
    aJump() {
      // let url = window.location.href
      let edit = this.$refs.edit
      let view = this.$refs.view
      setTimeout(() => {
        edit.scrollTop = view.scrollTop
        // window.location.href = url
      }, 100)
    },
    // 菜单缩进
    menuBtn() {
      if (this.isBack) {
        this.menuClass = ''
      } else {
        this.menuClass = 'menuBack'
      }
      this.isBack = !this.isBack
    },
  },
  watch: {
    mdText(val, old) {
      this.update()
    }
  },
  mounted() {
    let edit = this.$refs.edit
    let view = this.$refs.view
    // 当输入框的滚动条变化后，视图的也变化
    let old = edit.scrollTop
    this.timer = setInterval(() => {
      if (edit.scrollTop != old) {
        view.scrollTop = edit.scrollTop
        old = edit.scrollTop
      }
    }, 50)
    this.update()
  },
  destroyed() {
    clearInterval(this.timer)
  },
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
h1 {
  line-height: 50px;
  font-size: 26px;
}
h2 {
  line-height: 45px;
  font-size: 24px;
}
h3 {
  line-height: 40px;
  font-size: 22px;
}
h4 {
  line-height: 35px;
  font-size: 20px;
}
h5 {
  line-height: 30px;
  font-size: 18px;
}
p {
  line-height: 20px;
  font-size: 16px;
}
code {
  display: block;
  white-space: pre-wrap;
  margin: 5px 0;
  padding: 5px;
  width: calc(100% - 10px);
  height: auto;
  line-height: 20px;
  background: #555;
  color: #5d5;
  font-size: 14px;
}
img {
  width: 100%;
}
.zh_tex {
  text-align: center;
}
</style>

<style scoped>
.markdown {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

/* 菜单 */
.menu {
  position: fixed;
  top: 0;
  right: 0;
  display: flex;
  width: 200px;
  height: 100%;
  z-index: 2;
  transition: right 0.5s;
}
.menuBack {
  right: -150px;
}
.btn {
  margin-top: 10px;
  margin-right: 20px;
  width: 30px;
  height: 24px;
  cursor: pointer;
}
.line {
  margin-bottom: 4px;
  width: 100%;
  height: 4px;
  background: #555;
}
.index {
  width: calc(100% - 50px);
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
}
.caption {
  width: 100%;
  text-align: left;
  text-indent: 10px;
}
.caption:hover {
  background: rgba(255, 255, 255, 0.2);
}

/*包含以下四种的链接*/
.caption a {
  text-decoration: none;
  display: block;
  width: 100%;
  color: #fff;
}

/*正常的未被访问过的链接*/
.caption a:link {
  text-decoration: none;
}

/*已经访问过的链接*/
.caption a:visited {
  text-decoration: none;
}

/*鼠标划过(停留)的链接*/
.caption a:hover {
  text-decoration: none;
}

/* 正在点击的链接*/
.caption a:active {
  text-decoration: none;
}
.caption1 {
  font-size: 20px;
  line-height: 26px;
}

.caption2 {
  padding-left: 5px;
  font-size: 19px;
  line-height: 25px;
}

.caption3 {
  padding-left: 10px;
  font-size: 18px;
  line-height: 24px;
}

.caption4 {
  padding-left: 15px;
  font-size: 17px;
  line-height: 23px;
}

.caption5 {
  padding-left: 20px;
  font-size: 16px;
  line-height: 22px;
}

.main {
  display: flex;
  width: calc(100% - 0);
  height: 100%;
}
/* 编辑区 */
.edit {
  display: flex;
  width: 50%;
  height: 100%;
  overflow-y: scroll;
  background: #777;
}
.lineNum {
  width: 30px;
}
.num {
  color: #fff;
  font-size: 15px;
  line-height: 20px;
  text-align: center;
}
textarea {
  overflow-y: hidden;
  width: calc(100% - 30px);
  height: 20px;
  background: #555;
  color: #fff;
  border: none;
  outline: none;
  font-size: 15px;
  line-height: 20px;
}

/* 视图区 */
.view {
  width: 50%;
  height: 100%;
  overflow-y: scroll;
}
.result {
  text-align: left;
}
</style>
