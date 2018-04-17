---
title: 浅谈线程中的wait()和notify()
date: 2018-04-03 16:43:45
tags: 
	- love
categories: private
---

{% asset_img  my.jpg 图片 %}

> 示例代码如下：

<!-- more -->
``` java
package com.baizhi;

import java.time.LocalDate;
import java.time.Month;
import java.time.temporal.ChronoUnit;

/**
 * @author gaozhy
 * @date 2018/4/8.15:27
 */
public class MyHeart {

    private static final Object MY_LOVE = "❤";

    public static void main(String[] args) throws InterruptedException {

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (MY_LOVE){
                    try {
                        System.out.println("----------LOST-----------");
                        MY_LOVE.wait(Integer.MAX_VALUE);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("-----------THE MOST PRECIOUS THING IN THE WORLD IS NOT \"GET\" AND \"LOST\", BUT THE HAPPINESS THAT CAN BE GRASPED NOW----------");
            }
        }).start();

        Thread.sleep( ChronoUnit.DAYS.between(LocalDate.of(2017, Month.AUGUST, 28),LocalDate.now()) * 24 * 3600 * 1000);

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (MY_LOVE){
                    System.out.println("-----------UNTIL I MET YOU YY----------");
                    System.out.println("-----------THE DEAD HEART WAS REKINDLED----------");
                    MY_LOVE.notify();
                }
            }
        }).start();
    }
}

```