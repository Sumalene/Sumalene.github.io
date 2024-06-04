---
layout: single
title: "[Web]从Gin到gofiber"
categories: [web]
last_modified_at: 2024-05-28
excerpt: "迁移学习"
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---

会用到的, 这里简单记录一下笔记.

### GoFiber 框架
- **简介**：GoFiber是一个基于Fasthttp的Web框架，旨在提供类似Express.js的体验，同时充分利用Go的性能优势。
- **特性**：
  - 高性能。
  - 简洁易用的API设计。
  - 中间件支持。
  - 路由处理。
  - 静态文件服务。
  - 支持WebSocket。

### 从Gin迁移到GoFiber

1. **项目结构迁移**：
   - 保持项目目录结构一致，逐步替换Gin相关代码。

2. **替换依赖**：
   - 将`gin-gonic/gin`替换为`gofiber/fiber`。

3. **路由迁移**：
   - 将Gin的路由定义转换为Fiber的格式。
   
```go
   // Gin
   r.GET("/ping", func(c *gin.Context) {
       c.JSON(200, gin.H{
           "message": "pong",
       })
   })

   // Fiber
   app.Get("/ping", func(c *fiber.Ctx) error {
       return c.JSON(fiber.Map{"message": "pong"})
   })
```

4. **中间件迁移**：
   - 将Gin的中间件替换为Fiber的中间件。
  
```go
   // Gin
   r.Use(gin.Logger())

   // Fiber
   app.Use(logger.New())
```

5. **请求处理**：
   - 将Gin的上下文处理器转换为Fiber的上下文处理器。
   
```go
   // Gin
   func handler(c *gin.Context) {
       name := c.Query("name")
       c.JSON(200, gin.H{"name": name})
   }

   // Fiber
   func handler(c *fiber.Ctx) error {
       name := c.Query("name")
       return c.JSON(fiber.Map{"name": name})
   }
 ```

> ...

