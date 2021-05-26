---
layout: post
title: "Golang：time包"
date:  2020-05-17 10:01:08
author: "Viletyy"
header-style: text
tags:
  - Golang
  - 包
---
在标准库中，为我们提供了time包，里面都是关于日期时间的操作，都比较简单。 
### 1. 获取时间对象
获取当前时间
```go 
func Now() Time 
// Now returns the current local time

t := time.Now()
```
获取指定时间
```go 
func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time 
/*
Date returns the Time corresponding to 
yyyy-mm-dd hh:mm:ss + nsec nanoseconds
in the appropriate zone for that time in the given location.

The month, day, hour, min, sec, and nsec values may be outside their usual ranges and will be normalized during the conversion. For example, October 32 converts to November 1.

A daylight savings time transition skips or repeats times. For example, in the United States, March 13, 2011 2:15am never occured, while November 6, 2011 1:15am occurred twice. In such cases, the choice of time zone, and therefore the time, is not well-defined. Date returns a time that is correct in one of the two zones involved in the transition, but it does not guarantee which.

Date panics if loc is nil.
*/
t := time.Date(2008, 7, 15, 16, 30, 28, 0, time.Local)
```
### 2. 日期和文本之间的转换
日期转为文本：
```go 
func(t Time) Format(layout string) string 

t := time.Now()
s := time.Format("2006年1月2日 15:04:05")
```
文本转为日期：
```go 
func Parse(layout, value string) (Time, error) 

s := "1999年10月10日"
t, err := time.Parse("2006年1月2日", s)
```
### 3. 时间戳
时间戳：指定的日期(当前日期，指定的某个日期)，距离1970年1月1日0点0时0分0秒的时间差值。有秒，纳秒两种函数。 
获取单位为秒的时间戳：
```go 
func (t Time) Unix() int64 

t := time.Now()
i := t.Unix()
```
获取单位为纳秒的时间戳
```go 
func(t Time) UnixNano() int64

t := time.Now() 
i := t.UnixNano()
```
### 4. 获取年月日时间秒等
根据time对象，可以获取对应等年、月、日、时、分、秒等等 
```go 
func (t Time) Year() int 
func (t Time) Month() Month 
func (t TIme) Day() int 
func (t Time) Hour() int 
func (t Time) Minute() int 
func (t Time) Second() int 
func (t Time) Date() (year int, month Month, day int)
func (t Time) Clock() (hour, min, sec int)  
```
### 5. Sleep()
```go
func Sleep(d Duration) 

time.Sleep(3*time.Second)
```