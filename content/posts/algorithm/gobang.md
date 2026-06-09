+++
title = "五子棋判断输赢"
date = "2025-07-09T17:37:54+08:00"
summary = "五子棋判断输赢"
categories = ["算法相关"]
tags = ["五子棋"]
draft = false
+++

## 输赢关键点

首先在玩家决出胜负前，棋盘上，不可能存在五子连珠的情况。当决出胜负时，下的那个棋子，肯定是五子连珠中的一颗。所以判断五子棋输赢问题的关键点是检测最新一颗棋子4个方向能不能形成五子连珠，如果形成五子连珠，则可以判断出输赢。

## 暴力遍历法

基于上面的思想，可以使用暴力遍历的方法，判断出五子棋输赢。

```rust
/// 检测五子棋坐标是否在棋盘
fn check_coordinate(row: i8, col: i8) -> bool {
    row >= 0 && row < 15 && col >= 0 && col < 15
}

/// 检查五子棋输赢
fn check_win(board: &[[i8; 15]; 15], row: i8, col: i8, player: i8) -> bool {
    // 检查方向
    let directions = [(1, 0), (0, 1), (1, 1), (1, -1)];
    for (row_d, col_d) in directions {
        // 记录玩家棋子数量
        let mut count = 1;
        // 正向检测
        for i in 1..5 {
            let (r, c) = (row + row_d * i, col + col_d * i);
            if check_coordinate(r, c) && board[r as usize][c as usize] == player {
                count += 1;
            } else {
                // 边界越界，遇到其他棋子，停止检测
                break;
            }
        }
        if count >= 5 {
            return true;
        }
        // 反向检测
        for i in 1..5 {
            let (r, c) = (row + row_d * -i, col + col_d * -i);
            if check_coordinate(r, c) && board[r as usize][c as usize] == player {
                count += 1;
            } else {
                // 边界越界，遇到其他棋子，停止检测
                break;
            }
        }
        if count >= 5 {
            return true;
        }
    }
    false
}
```

## 总结

上面的算法，虽然是暴力遍历，但是，我认为时间复杂度是O(1)，当检测的是n子棋时，算法时间复杂度上升到O(n)。
