---
title: Java生成验证码的工具类
comments: true
date: 2018-7-16 15:53:47
updated: 2018-7-16 15:53:47
tags:
- Verification Code
categories:
- java
---

生成验证码的工具类,可以生成简单的验证码
<!--more-->
``` java VerificationCodeUtils
import java.awt.*;
import java.awt.image.BufferedImage;
import java.util.Arrays;
import java.util.Random;

/**
 * @author nicenan
 */
public class VerificationCodeUtils {
    private int width = 100;
    private int height = 50;
    private int length = 4;
    private int interLine = 10;
    private float noiseRate = 0.1f;
    private String verificationCode;

    public int getInterLine() {
        return interLine;
    }

    public void setInterLine(int interLine) {
        this.interLine = interLine;
    }

    public float getNoiseRate() {
        return noiseRate;
    }

    public void setNoiseRate(float noiseRate) {
        this.noiseRate = noiseRate;
    }

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getLength() {
        return length;
    }

    public void setLength(int length) {
        this.length = length;
    }

    public String getVerificationCode() {
        return verificationCode;
    }

    public char[] getVerificationCodeArrary() {
        return verificationCodeArrary;
    }

    public void setVerificationCodeArrary(char[] verificationCodeArrary) {
        this.verificationCodeArrary = verificationCodeArrary;
    }

    private char[] verificationCodeArrary = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
            'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z',
            'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'
    };

    //饿汉单例模式
    private VerificationCodeUtils() {
    }

    private static VerificationCodeUtils verificationCodeUtils;

    public static VerificationCodeUtils getInstance() {
        if (verificationCodeUtils == null) {
            synchronized (VerificationCodeUtils.class) {
                if (verificationCodeUtils == null) {
                    verificationCodeUtils = new VerificationCodeUtils();
                    return verificationCodeUtils;
                }
            }
        }
        return verificationCodeUtils;
    }

    @Override
    public String toString() {
        return "VerificationCodeUtils{" +
                "width=" + width +
                ", height=" + height +
                ", length=" + length +
                ", interLine=" + interLine +
                ", noiseRate=" + noiseRate +
                ", verificationCode='" + verificationCode + '\'' +
                ", verificationCodeArrary=" + Arrays.toString(verificationCodeArrary) +
                '}';
    }

    public BufferedImage createVerificationCodeImage() {
        BufferedImage bufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        Graphics graphics = bufferedImage.getGraphics();
        Random random = new Random();

        graphics.setColor(getRandomColor());
        graphics.fillRect(0, 0, width, height);
        //设置乱七八糟横线
        if (interLine > 0) {
            for (int i = 0; i < interLine; i++) {
                int x1 = random.nextInt(width);
                int y1 = random.nextInt(height);
                int x2 = random.nextInt(width);
                int y2 = random.nextInt(height);
                graphics.setColor(getRandomColor());
                graphics.drawLine(x1, y1, x2, y2);
            }

        }

        //最终的验证码
        StringBuffer sb = new StringBuffer();
        //写入验证码
        int fontsize = (int) (height * 0.9);
        graphics.setFont(new Font(Font.SANS_SERIF, Font.BOLD, fontsize));
        //要画的字符的X坐标
        int codex = 0;
        int codey = 0;
        for (int i = 0; i < length; i++) {
            codey = (int) ((Math.random() * 0.3 + 0.7) * height);
            graphics.setColor(getRandomColor());
            int anInt = random.nextInt(verificationCodeArrary.length - 1);
            sb.append(verificationCodeArrary[anInt]);
            graphics.drawChars(verificationCodeArrary, anInt, 1, codex, codey);
            codex += (width / length) * (Math.random() * 0.3 + 0.9);
        }

        //插入噪点
        int noiseNum = (int) (width * height * noiseRate);
        for (int i = 0; i < noiseNum; i++) {
            int x = random.nextInt(width);
            int y = random.nextInt(height);
            graphics.setColor(getRandomColor());
            graphics.drawLine(x, y, x, y);
        }
        graphics.dispose();
        verificationCode = sb.toString();
        return bufferedImage;
    }

    private Color getRandomColor() {
        Random random = new Random();
        return new Color(random.nextInt(255), random.nextInt(255), random.nextInt(255));
    }
}
```