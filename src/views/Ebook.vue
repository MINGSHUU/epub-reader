<template>
    <div class="ebook">
        <title-bar
            :ifTitleAndMenuShow="ifTitleAndMenuShow"/>
        <div class="read-wrapper">
            <div></div>
            <div id="read"></div>
            <div class="mask">
                <div class="left" @click="prevPage"></div>
                <div class="center" @click="toggleTitleAndMenu"></div>
                <div class="right" @click="nextPage"></div>
            </div>
        </div>
        <menu-bar
             :ifTitleAndMenuShow="ifTitleAndMenuShow"
             :fontSizeList="fontSizeList"
             :defaultFontSize="defaultFontSize"
             @setFontSize="setFontSize"
             :themeList="themeList"
             :defaultTheme="defaultTheme"
             @setTheme="setTheme"
             :bookAvailable="bookAvailable"
             @onProgressChange="onProgressChange"
             :navigation="navigation"
             @jumpTo="jumpTo"
             ref="menuBar"/>
    </div>
</template>

<script>
  import TitleBar from '@/components/TitleBar'
  import MenuBar from '@/components/MenuBar'
  import Epub from 'epubjs'
  const DOWNLOAD_URL = '/book/2018_Book_AgileProcessesInSoftwareEngine.epub';
  // const DOWNLOAD_URL = '/book/66485.epub';
  export default {
    components: {
      TitleBar,
      MenuBar
    },
    data() {
      return {
        // 是否显示标题栏和菜单栏
        ifTitleAndMenuShow: false,
        // 字体大小列表
        fontSizeList: [
          { fontSize: 12 },
          { fontSize: 14 },
          { fontSize: 16 },
          { fontSize: 18 },
          { fontSize: 20 },
          { fontSize: 22 },
          { fontSize: 24 }
        ],
        defaultFontSize: 16,
        // 主题列表
        themeList: [
          {
            name: "default",
            styles: {
              body: {
                color: "#000", background: "#fff"
              }
            }
          },
          {
            name: "eye",
            styles: {
              body: {
                color: "#000", background: "#ceeaba"
              }
            }
          },
          {
            name: "night",
            styles: {
              body: {
                color: "#fff", background: "#000"
              }
            }
          },
          {
            name: "gold",
            styles: {
              body: {
                color: "#000", background: "rgb(241,236,226)"
              }
            }
          }
        ],
        // 当前主题索引号
        defaultTheme: 0,
        // 图书是否处于可用状态
        bookAvailable: false,
        navigation: null
      }
    },
    mounted() {
      this.showEpub();
    },
    methods: {
      // 根据链接跳转到指定位置
      jumpTo(href) {
        this.rendition.display(href);
        this.hideTitleAndMenu();
      },
      hideTitleAndMenu() {
        // 隐藏标题栏和菜单栏
        this.ifTitleAndMenuShow = false;
        // 隐藏菜单栏弹出的设置栏
        this.$refs.menuBar.hideSetting();
        // 隐藏目录
        this.$refs.menuBar.hideContent();
      },
      // progress 进度条的数值（0-100）
      onProgressChange(progress) {
        const percentage = progress / 100;
        const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
        this.rendition.display(location)
      },
      setTheme(index) {
        this.themes.select(this.themeList[index].name);
        this.defaultTheme = index;
      },
      registerTheme() {
        this.themeList.forEach(theme => {
          this.themes.register(theme.name, theme.styles)
        })
      },
      setFontSize(fontSize) {
        this.defaultFontSize = fontSize;
        if (this.themes) {
          this.themes.fontSize(fontSize + 'px');
        }
      },
      toggleTitleAndMenu() {
        this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow;
        if(!this.ifTitleAndMenuShow) {
          this.$refs.menuBar.hideSetting();
        }
      },
      // 上一页
      prevPage() {
        if (this.rendition) {
          this.rendition.prev();
          this.hideTitleAndMenu();
        }
      },
      // 下一页
      nextPage() {
        if (this.rendition) {
          this.rendition.next();
          this.hideTitleAndMenu();
        }
      },
      // 显示电子书
      showEpub() {
        this.book = new Epub(DOWNLOAD_URL);
        // 生成Rendition, 通过Book.renderTo
        this.rendition = this.book.renderTo('read', {
          width: window.innerWidth,
          height: window.innerHeight
        });
        // 通过Rendition.display渲染电子书
        this.rendition.display();
        // 获取Theme对象
        this.themes = this.rendition.themes;
        // 设置默认字体
        this.setFontSize(this.defaultFontSize);
        //设置主题
        // this.themes.register(name, styles)
        // this.themes.select(name)
        this.registerTheme();
        this.setTheme(this.defaultTheme);
        // 设置进度条&目录
        // 获取Locatios对象
        // 通过epubjs的钩子函数来实现
        this.book.ready.then(() => {
          this.navigation = this.book.navigation;
          return this.book.locations.generate()
        }).then(result => {
            this.locations = this.book.locations;
            this.bookAvailable = true
        })
      }
    }
  }
</script>
<style lang="scss" scoped>
    @import '../assets/styles/global';

    .ebook {
        position: relative;
        .read-wrapper {
            .mask {
                position: absolute;
                top: 0;
                left: 0;
                display: flex;
                height: 100%;
                width: 100%;
                z-index: 100;
                .left, .right {
                    flex: 0 0 px2rem(100);
                }
                .center {
                    flex: 1;
                }
            }
        }
    }
</style>