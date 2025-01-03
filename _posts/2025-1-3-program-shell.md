---
layout: post
author: Dragroo
title: 一个用于后台执行的shell脚本
tag: 传家小技巧
---
```bash
#!/bin/bash

# 定义变量
PROGRAM="/usr/bin/mihomo -d /etc/mihomo"
LOG_FILE="mihomo.log"

# 检查进程是否正在运行
is_running() {
  if pgrep -x "$(basename "$PROGRAM")" > /dev/null; then
    return 0 # 进程存在
  else
    return 1 # 进程不存在
  fi
}

# 启动程序
start_program() {
  if is_running; then
    echo "错误: 程序已经启动，请不要重复启动。"
  else
    nohup $PROGRAM > $LOG_FILE 2>&1 &
    echo "程序已启动，日志记录在 $LOG_FILE 中。"
  fi
}

# 关闭程序
stop_program() {
  if is_running; then
    pkill -x "$(basename "$PROGRAM")"
    echo "程序已关闭。"
  else
    echo "错误: 未检测到运行中的程序。"
  fi
}

# 主菜单
while true; do
  echo "请选择操作:"
  echo "1) 启动程序"
  echo "2) 关闭程序"
  echo "3) 退出"
  read -p "输入选项 [1-3]: " option

  case $option in
    1)
      start_program
      ;;
    2)
      stop_program
      ;;
    3)
      echo "退出脚本。"
      exit 0
      ;;
    *)
      echo "无效选项，请重新输入！"
      ;;
  esac
done
```