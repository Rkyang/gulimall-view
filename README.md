#### 为了更好的服务国内用户，renren-fast-vue仓库已迁移到Gitee，新的仓库地址 https://gitee.com/renrenio/renren-ui ，以后会在 https://gitee.com/renrenio/renren-ui 里面进行更新维护，感谢支持！ :smiley:
<br>

## renren-fast-vue
- renren-fast-vue基于vue、element-ui构建开发，实现[renren-fast](https://gitee.com/renrenio/renren-fast)后台管理前端功能，提供一套更优的前端解决方案
- 前后端分离，通过token进行数据交互，可独立部署
- 主题定制，通过scss变量统一一站式定制
- 动态菜单，通过菜单管理统一管理访问路由
- 数据切换，通过mock配置对接口数据／mock模拟数据进行切换
- 发布时，可动态配置CDN静态资源／切换新旧版本
- 演示地址：[http://demo.open.renren.io/renren-fast](http://demo.open.renren.io/renren-fast) (账号密码：admin/admin)

![输入图片说明](https://images.gitee.com/uploads/images/2019/0305/133529_ff15f192_63154.png "01.png")
![输入图片说明](https://images.gitee.com/uploads/images/2019/0305/133537_7a1b2d85_63154.png "02.png")


## 说明文档
项目开发、部署等说明都在[wiki](https://github.com/renrenio/renren-fast-vue/wiki)中。


## 更新日志
每个版本的详细更改都记录在[release notes](https://github.com/renrenio/renren-fast-vue/releases)中。

## 代码download后到运行需要的配置，谨以此做个人的学习备忘（2022.11.17 by Rkyang）
1. 推荐使用node.js版本：12.18.2
2. 配置npm使用淘宝源：npm config set registry http://registry.npm.taobao.org/
3. 先单独下载node-sass（会有renrenfast的警告或者错误，我忽略了）：npm i node-sass --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
4. 执行cnpm install，错误和警告我也忽略了（cnpm版本：npm install cnpm@7.1.0 -g --registry=https://registry.npm.taobao.org）
   1. 后续安装pubsub-js也使用cnpm
5. 执行npm run dev启动项目
