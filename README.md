# tvbox-api-backup-mirror

上游 [`lubin776/tvbox-api-backup`](https://github.com/lubin776/tvbox-api-backup) 的**自动检测 + 整仓同步镜像**。

本仓库不承载任何采集逻辑，全部内容都由 GitHub Actions 从上游同步而来；上游更新后，这里会在当天自动跟进。

## 同步机制

- 工作流：[`.github/workflows/sync-upstream.yml`](.github/workflows/sync-upstream.yml)
- 触发方式：每天 `21:00 UTC`（北京时间次日 `05:00`）定时执行一次，也支持在 Actions 页面手动 `Run workflow`
- 选定北京时间 05:00 的原因：上游 `api下载.yml`（北京 00:05）、`ry下载.yml`（北京 01:00）、`接口处理扫描工具.yml`（北京 03:00）三个采集任务此时都已跑完，一次同步即可拿全当天产物
- 检测逻辑：用 `git ls-remote` 读取上游 `main` 分支最新 commit SHA，与本仓库 `.github/upstream-sha` 记录的上次同步指纹比对
  - 指纹相同 → 判定「上游无变化」，直接结束，**不产生空提交**
  - 指纹不同 → `git clone --depth 1` 上游，`rsync -a --delete` 整仓覆盖，并更新指纹、提交推送

## 镜像范围

整仓跟随上游，但由于镜像仓库自身需要保留配置文件，以下几项**不参与同步**：

| 路径 | 说明 |
| --- | --- |
| `.git/` | 本仓库自身的版本库 |
| `.github/` | 本仓库自身的同步工作流与指纹文件 |
| `README.md` | 本文件（上游同名文件不覆盖） |
| `CNAME` | 上游的 GitHub Pages 自定义域名指向其自有域名，镜像过来会导致 Pages 无法验证，故不同步 |

其余全部内容都按上游当前状态整仓覆盖。以首次同步时的上游 `main` 为例，镜像包含各页面（`index.html`、`app.html`、`firstpage.html`、`整合版.html`、`空壳板.html` 等）、`config.json`、`list.txt`、`livelist.txt`、`LICENSE`、`SUMMARY.txt`、以及 `cy/`、`ry/`、`vip/`、`python/` 四个目录下的全部文件。**上游已删除的文件在本仓库也会被同步删除**，因此是真正的镜像而非增量叠加。

## 使用方式

本仓库内容与上游一致，可作为 TVBox 类客户端的接口订阅源或静态站点直接托管：

- 直链订阅示例：`https://raw.githubusercontent.com/vgmr-99/tvbox-api-backup-mirror/main/config.json`
- 也可以直接开启 GitHub Pages（Settings → Pages → Deploy from a branch → `main` / `root`）当作静态站使用；如需绑定自己的域名，建议在 Pages 设置里配置而不要提交 `CNAME` 文件，避免与同步逻辑冲突
- 提醒：`config.json` 里的多线路域名（`apiDomains`）与接口地址一律指向上游自建域名和第三方源，镜像不会改写它们；截至首次同步（上游 `main@78deb73`），上游仓库内并没有 `tvbox/` 目录，接口数据本身也不存放在本仓库——镜像只保证「上游仓库里有的，这里一模一样」

## 版权与免责声明

- 本仓库是对上游仓库的**逐字镜像**，上游以 **GPL-3.0** 许可发布，原始 `LICENSE` 文件随镜像保留，署名归原作者
- 仓库内的接口、直播源、书源等资源均来自互联网，版权归各自原作者所有
- 仅供**测试、学习与研究**使用，请勿用于商业用途，请在下载后 24 小时内删除
- 镜像仓库不对内容的可用性、合法性作任何担保，使用风险自负