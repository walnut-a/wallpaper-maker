# Wallpaper Maker

一个单文件、纯前端的参数化手机壁纸生成工具。

## 在线体验

[https://walnut-a.github.io/wallpaper-maker/](https://walnut-a.github.io/wallpaper-maker/)

## 特点

- 打开页面就能直接看到一张成品壁纸
- 内置四种几何风格预设
- 支持常用参数和高级参数两层控制
- 支持自定义尺寸和常用手机尺寸预设
- 支持本地导出 `PNG`
- 不依赖后端，可直接托管到 GitHub Pages

## 文件结构

- `index.html`：主应用，包含结构、样式和脚本
- `PRINCIPLES.md`：这个项目的设计与实现原则
- `docs/superpowers/specs/`：设计文档
- `docs/superpowers/plans/`：实现计划

## 本地使用

直接双击打开 `index.html` 就能使用。

如果你想用本地静态服务预览，可以运行：

```bash
python3 -m http.server 8123
```

然后访问：

```text
http://127.0.0.1:8123
```

## 部署到 GitHub Pages

这个项目是静态页面，不需要构建步骤。把仓库发布到 GitHub Pages 后，直接访问生成的页面地址即可。

当前公开地址：

[https://walnut-a.github.io/wallpaper-maker/](https://walnut-a.github.io/wallpaper-maker/)
