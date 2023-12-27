---
layout: single
title: "[Java]大作业添加一个评论区"
categories:
  - Java
last_modified_at: 2023-12-27
excerpt: 一个意见反馈界面
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---

>意见反馈界面,存储留言内容

## 相关知识点：  

文件读写,UI

只需要修改MainFrame.java这个UI文件

---

确保包导入

```java
package com.ascent.ui;  
  
import javax.swing.*;  
import java.awt.*;  
import java.awt.event.*;  
import java.io.BufferedWriter;  
import java.io.FileWriter;  
import java.io.IOException;  
import javax.swing.JTabbedPane;  
import javax.swing.JFrame;  
import javax.swing.event.ChangeListener;
```


在构造函数(....public class MainFrame extends JFrame {....)里添加:

```java
// 创建"意见反馈"选项卡  
JPanel feedbackPanel = new JPanel(new BorderLayout());  
JTextArea feedbackArea = new JTextArea();  
feedbackArea.setEditable(false);  
  
JScrollPane feedbackScrollPane = new JScrollPane(feedbackArea);  
feedbackPanel.add(feedbackScrollPane, BorderLayout.CENTER);  
  
JPanel inputPanel = new JPanel();  
JTextField nameField = new JTextField(10);  
JTextField commentField = new JTextField(20);  
JButton submitButton = new JButton("提交");  
inputPanel.add(new JLabel("昵称:"));  
inputPanel.add(nameField);  
inputPanel.add(new JLabel("评论:"));  
inputPanel.add(commentField);  
inputPanel.add(submitButton);  
feedbackPanel.add(inputPanel, BorderLayout.SOUTH);  
  
tabbedPane.addTab("意见反馈", feedbackPanel);  
  
submitButton.addActionListener(new ActionListener() {  
    @Override  
    public void actionPerformed(ActionEvent e) {  
        String name = nameField.getText();  
        String comment = commentField.getText();  
        if (!name.isEmpty() && !comment.isEmpty()) {  
            feedbackArea.append(name + ": " + comment + "\n");  
            nameField.setText("");  
            commentField.setText("");  
            saveFeedback(name, comment);  
        }  
    }  
  
    private void saveFeedback(String name, String comment) {  
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("feedback.txt", true))) {  
            writer.write(name + ": " + comment);  
            writer.newLine();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }  
});
```

效果:

留言板会收集昵称和评论内容

目录会自动生成feedback文件,形成一个简易的留言板.