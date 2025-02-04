---
date: 2025-02-05T02:49:01+08:00
tags:
  - Swift
title: 开发日志 | 收藏导出功能开发记录 Part1 基础绘制实现
slug: "1738694941"
share: true
searchHidden: false
disableShare: true
canonicalURL: ""
keywords: 
description: ""
series: 系列
lastmod: 
lang: cn
cover:
    image: ""
author: xy0v0
dir: posts
---
## 开发构思

收藏时内容包含html标签，这是在完成阅读器界面时留下的问题。现在有两个解决方案：
- 修改阅读器，但是现在运行正常，且效率足够
- 添加收藏时增加一步去除html标签的步骤，这一方法之前有过实现

故更倾向后者

swift如何绘制/渲染图形？查阅到 `UIGraphicsImageRenderer` 可以使用

## 开发过程

### 预处理收藏内容

去除html标签的方法之前已经在添加收藏时候写过了，所以不用写了

### 生成图片

这个功能的关键所在，我对这个软件的定位主要在于“阅读”，所以我想在导出的图片中体现复古风格，故选取米黄色 `UIColor(red: 253/255, green: 246/255, blue: 227/255, alpha: 1.0)` 作为导出的背景，接下来就是元素的绘制：

``` swift
class ExportManager {
	...
	// MARK: - 图片导出
    
    /// 生成报纸风格的图片
    /// - Parameters:
    ///   - quotes: 要导出的收藏数组
    ///   - completion: 完成回调，返回生成的图片或错误
    func generateNewspaperImage(
        from quotes: [Quote],
        completion: @escaping (Result<UIImage, Error>) -> Void
    ) {
        // 检查是否有内容
        guard !quotes.isEmpty else {
            completion(.failure(ExportError.emptyContent))
            return
        }
        
        // 计算内容总高度
        let contentWidth: CGFloat = 800 // 基础宽度
        let titleHeight: CGFloat = 120 // 标题区域高度
        let dateHeight: CGFloat = 30 // 日期区域高度
        let topPadding: CGFloat = 40 // 顶部边距
        let bottomPadding: CGFloat = 40 // 底部边距
        let quoteSpacing: CGFloat = 20 // 引用之间的间距
        
        // 计算每个引用视图的高度
        var totalHeight: CGFloat = titleHeight + dateHeight + topPadding + bottomPadding
        let quoteViews = quotes.map { createQuoteView($0, width: contentWidth - 120) }
        totalHeight += quoteViews.reduce(0) { $0 + $1.bounds.height }
        totalHeight += CGFloat(quotes.count - 1) * quoteSpacing
        
        // 创建容器视图
        let containerView = UIView(frame: CGRect(x: 0, y: 0, width: contentWidth, height: totalHeight))
        containerView.backgroundColor = UIColor(red: 253/255, green: 246/255, blue: 227/255, alpha: 1.0) // 复古米黄色背景
        
        // 添加纸张纹理效果
        let noiseLayer = CALayer()
        noiseLayer.frame = containerView.bounds
        noiseLayer.backgroundColor = UIColor.black.cgColor
        noiseLayer.opacity = 0.03
        containerView.layer.addSublayer(noiseLayer)
        
        // 创建主标题容器
        let titleContainer = UIView(frame: CGRect(x: 60, y: topPadding, width: containerView.bounds.width - 120, height: titleHeight))
        titleContainer.backgroundColor = .clear
        
        // 添加装饰性边框
        let borderLayer = CAShapeLayer()
        borderLayer.strokeColor = UIColor(red: 0.2, green: 0.2, blue: 0.2, alpha: 0.8).cgColor
        borderLayer.fillColor = nil
        borderLayer.lineWidth = 2
        borderLayer.path = UIBezierPath(roundedRect: titleContainer.bounds, cornerRadius: 4).cgPath
        
        // 添加双线边框效果
        let innerBorderLayer = CAShapeLayer()
        innerBorderLayer.strokeColor = borderLayer.strokeColor
        innerBorderLayer.fillColor = nil
        innerBorderLayer.lineWidth = 1
        innerBorderLayer.path = UIBezierPath(roundedRect: titleContainer.bounds.insetBy(dx: 6, dy: 6), cornerRadius: 2).cgPath
        
        titleContainer.layer.addSublayer(borderLayer)
        titleContainer.layer.addSublayer(innerBorderLayer)
        
        // 创建主标题
        let titleLabel = UILabel()
        titleLabel.text = NSLocalizedString("export.newspaper.title", comment: "Newspaper title") // i18n
        titleLabel.font = UIFont(name: "TimesNewRomanPS-BoldMT", size: 48)
        titleLabel.textAlignment = .center
        titleLabel.frame = titleContainer.bounds.insetBy(dx: 20, dy: 20)
        titleLabel.backgroundColor = .clear
        titleContainer.addSubview(titleLabel)
        containerView.addSubview(titleContainer)
        
        // 添加日期
        let dateLabel = UILabel()
        dateLabel.text = String(format: NSLocalizedString("export.newspaper.date", comment: "Publish date"),
                              DateFormatter.yyyyMMdd.string(from: Date()))
        dateLabel.font = UIFont(name: "TimesNewRomanPS-ItalicMT", size: 16)
        dateLabel.textAlignment = .center
        dateLabel.textColor = UIColor(red: 0.4, green: 0.4, blue: 0.4, alpha: 1.0)
        dateLabel.frame = CGRect(x: 60, y: titleContainer.frame.maxY + 16, width: containerView.bounds.width - 120, height: dateHeight)
        dateLabel.backgroundColor = .clear
        containerView.addSubview(dateLabel)
        
        // 添加引用视图
        var currentY = dateLabel.frame.maxY + 30
        for (index, quoteView) in quoteViews.enumerated() {
            quoteView.frame.origin = CGPoint(x: 60, y: currentY)
            containerView.addSubview(quoteView)
            currentY = quoteView.frame.maxY + (index < quoteViews.count - 1 ? quoteSpacing : 0)
        }
        
        // 生成图片
        DispatchQueue.main.async {
            // 确保所有子视图都已布局
            containerView.layoutIfNeeded()
            
            // 创建图片上下文，背景透明
            UIGraphicsBeginImageContextWithOptions(containerView.bounds.size, false, 0.0)
            defer { UIGraphicsEndImageContext() }
            
            guard let context = UIGraphicsGetCurrentContext() else {
                completion(.failure(ExportError.imageGenerationFailed))
                return
            }
            
            // 渲染视图层级
            containerView.layer.render(in: context)
            
            guard let image = UIGraphicsGetImageFromCurrentImageContext() else {
                completion(.failure(ExportError.imageGenerationFailed))
                return
            }
            
            completion(.success(image))
        }
    }

	/// 创建单条引用视图
    /// - Parameters:
    ///   - quote: 引用内容
    ///   - width: 视图宽度
    /// - Returns: 包含引用内容的视图
    private func createQuoteView(_ quote: Quote, width: CGFloat) -> UIView {
        let container = UIView()
        container.backgroundColor = .clear
        
        // 添加内容
        let contentLabel = UILabel()
        contentLabel.text = quote.content.removingHTMLTags()
        contentLabel.font = UIFont(name: "TimesNewRomanPS-BoldMT", size: 24)
        contentLabel.numberOfLines = 0
        contentLabel.textColor = UIColor(red: 0.2, green: 0.2, blue: 0.2, alpha: 1.0)
        
        // 计算内容高度
        let contentSize = contentLabel.sizeThatFits(CGSize(width: width, height: .greatestFiniteMagnitude))
        contentLabel.frame = CGRect(x: 0, y: 0, width: width, height: contentSize.height)
        container.addSubview(contentLabel)
        
        // 添加来源信息
        let metaLabel = UILabel()
        metaLabel.text = "—— " + quote.articleTitle
        metaLabel.font = UIFont(name: "TimesNewRomanPS-ItalicMT", size: 14)
        metaLabel.textColor = UIColor(red: 0.5, green: 0.5, blue: 0.5, alpha: 1.0)
        metaLabel.textAlignment = .right
        metaLabel.frame = CGRect(x: 0, y: contentLabel.frame.maxY + 12, width: width, height: 20)
        container.addSubview(metaLabel)
        
        // 添加装饰性分隔线
        let separatorView = UIView(frame: CGRect(x: width * 0.1,
                                               y: metaLabel.frame.maxY + 24,
                                               width: width * 0.8,
                                               height: 1))
        separatorView.backgroundColor = UIColor(red: 0.8, green: 0.8, blue: 0.8, alpha: 0.5)
        container.addSubview(separatorView)
        
        // 设置容器大小
        container.frame = CGRect(x: 0, y: 0,
                               width: width,
                               height: separatorView.frame.maxY)
        
        return container
    }
}
```

最终效果姑且有了部分我想要的效果，但中文字体的选取可能后续还要更改：

<div align=center><img src = https://cn-sy1.rains3.com/pic/pic/2025/02/cc0007c2d90dedf3ef1b992a9beb844d.png width = 80%></div>

但是仍然有几个问题：
- 导出界面并不是图片类型的导出，需要修改
- 长文导出图片过大... 如何解决？或许省略部分或者阻止导出图片？又或许分割为若干张图片，限制单张图片最大长度
- 动画还是有点卡顿

放在后续版本慢慢修复吧，计划能够在3个版本内完全完善这个功能实现
