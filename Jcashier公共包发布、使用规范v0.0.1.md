## Jcashier公共包协同开发指南v0.0.1
### SNAPSHOT VS RELEASE
`SNAPSHOT`版本不保证用户下载使用的包代码是不变的，属于开发阶段的不稳定版本。`RELEASE`（不带`SNAPSHOT`后缀的都属于`RELEASE`版本）版本是稳定版本，一旦发布之后不可修改或者覆盖。

#### 发布
`SNAPSHOT`相同版本的包可重复发布到远程`SNAPSHOT`仓库，既有版本会被加上时间后缀作为历史版本。而`RELEASE`相同版本的包只允许发布一次，所以`RELEASE`仓库相同版本的包不允许被修改，从而保证包的稳定性和不变性。

#### 下载安装
当使用的包是`SNAPSHOT`版本时，maven将根据`SNAPSHOT`包的更新策略决定是否从**远程仓库**下载最新的包到**本地仓库**，所以如果更新策略是"always"，理论上用户使用的包永远是最新的，即使这个包没有被正确验证。当使用的包时`RELEASE`版本时，maven也会根据更新策略尝试更新，但在版本固定的情况下（<version>配置为固定的版本号），尽管是从远程仓库重新拉取的，这个包也是不变的。

>Snapshot本地仓库更新策略[updatePolicy]: "always", "daily" (default), "interval:XXX" (in minutes) or "never" (only if it doesn't exist locally).

### 当前问题
当前收银重构的Jcashier项目中,公共模块的发布都是`SNAPSHOT`版本，这也意味着依赖公共模块上线的各个微服务处于不可控的不稳定状态。而截至目前依赖公共模块的微服务多达8个（未来可能有衍生更多），因此当公共模块同时协同开发的情况下，开发阶段定会造成互相干扰，甚至因为公共包被覆盖导致上线失败、出现bug的隐患也非常大。

举个例子，Jcashier的项目结构简述如下
ibt-cashier-parent
| —— ibt-cashier-commmon-1.0.12-SNAPSHOT //工具模块
| —— ibt-cahier-data //数据模块
| —— ibt-cashier-settle // 结算服务
| —— ibt-cashier-wakanda // 通用收银服务
| —— ...

分支x的**ibt-cashier-settl**e和分支y的**ibt-cashier-wakanda**模块都在使用**ibt-cashier-commmon-1.0.12-SNAPSHOT**这一个版本，如果分支x修改了common包某个方法的逻辑（未被验证存在重大bug），并上传到远程仓库。而此后分支y打包上线，那么就预定了线上问题同步群的消息位。

### 尝试解决
#### 统一版本管理
既然`SNAPSHOT`版本会互相覆盖，是否可以通过提前约定各自的版本然后维护起来，从而达到互不干扰的效果。收银当前就是在使用类似的方式